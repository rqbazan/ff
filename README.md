<p align="center">
  <img src="./logo/toggle.png" height="128" />
  <p align="center">Icons made by <a href="https://www.flaticon.com/authors/pixel-perfect" title="Pixel perfect">Pixel perfect</a> from <a href="https://www.flaticon.com/" title="Flaticon">www.flaticon.com</a></p>
</p>

<h1 align="center">
  Feature Flags
</h1>

<p align="center">
  <a href="https://npmjs.org/package/toggled">
    <img alt="NPM version" src="https://img.shields.io/npm/v/toggled.svg?style=for-the-badge">
  </a>
  <a href="https://github.com/rqbazan/toggled">
    <img alt="LICENSE" src="https://img.shields.io/github/license/rqbazan/toggled?style=for-the-badge">
  </a>
  <a href="https://github.com/rqbazan/toggled/actions/workflows/main.yml">
    <img alt="CI" src="https://img.shields.io/github/workflow/status/rqbazan/toggled/CI?label=CI&style=for-the-badge">
  </a>
  <a href="https://bundlephobia.com/package/toggled">
    <img alt="Size" src="https://img.shields.io/bundlephobia/minzip/toggled?style=for-the-badge">
  </a>
</p>

Tiny library to use [feature flags](https://martinfowler.com/articles/feature-toggles.html) in React. Get features by its slug identifier or get a binary output using flag queries.

## Installation

NPM:

```sh
npm i toggled
```

Yarn:

```sh
yarn add toggled
```

## API

### `DefaultFeature` <sup>_Interface_</sup>

Internal public interface used by default to type the `<FeatureProvider />` and `useFeature`.

#### Specification

```ts
export interface DefaultFeature {
  slug: string
}
```

#### Example

```ts
// src/types/toggled.d.ts
import 'toggled'

// extend the interface if needed
declare module 'toggled' {
  export interface DefaultFeature {
    slug: string
    settings?: any
  }
}
```

### `FlagQuery` <sup>_Type_</sup>

It could be the feature slug or an flag queries array or more powerful, an object query.

#### Specification

```ts
type FlagQuery =
  | string
  | FlagQuery[]
  | {
      $or: FlagQuery[]
    }
  | {
      $and: FlagQuery[]
    }
  | {
      [slug: string]: boolean
    }
```

#### Example

```tsx
// src/constants/domain.ts
import { Op } from 'toggled'

// Note that each entry is a `FlagQuery`
export const flagQueries: Record<string, FlagQuery> = {
  // True if the slug is in the context
  FF_1: 'ff-1',

  // True if all the slugs are in the context
  FF_2_FULL: ['ff-2.1', 'ff-2.2'],

  // True if `'ff-2.1'` is in the context and `'ff-2.2'` is not
  FF_2_1_ONLY: {
    'ff-2.1': true,
    'ff-2.2': false,
  },

  // True if `'ff-3.1'` **or** `'ff-3.2'` is in the context
  FF_3_X: {
    [Op.OR]: ['ff-3.1', 'ff-3.2'],
  },

  // True if `'ff-4.1'` **and** `'ff-4.2'` are in the context
  FF_4_FULL: {
    [Op.AND]: ['ff-4.1', 'ff-4.2'],
  },

  // True if all the previous queries are true
  COMPLEX: {
    FF_1: 'ff-1',
    FF_2_FULL: ['ff-2.1', 'ff-2.2'],
    FF_2_1_ONLY: {
      'ff-2.1': true,
      'ff-2.2': false,
    },
    FF_3_X: {
      [Op.OR]: ['ff-3.1', 'ff-3.2'],
    },
    FF_4_FULL: {
      [Op.AND]: ['ff-4.1', 'ff-4.2'],
    },
  },
}
```

### `FeatureContext`

Library context, exported for no specific reason, avoid using it and prefer the custom hooks, or open a PR to add a new one that obligates you to use the `FeatureContext`.

#### Specification

```ts
interface FeatureContextValue<F extends DefaultFeature = DefaultFeature> {
  cache: Map<string, F>
}
```

### `<FeatureProvider />`

Provider component that exposes the features in a more convenient way to get them by its own slugs.

#### Specification

```ts
interface FeatureProviderProps<F extends DefaultFeature = DefaultFeature> {
  features: F[]
  children: React.ReactNode
}
```

#### Example

```tsx
import { FeatureProvider } from 'toggled'
import apiClient from './api-client'
import App from './app'

apiClient.getAllFeatures().then(features => {
  ReactDOM.render(
    <FeatureProvider features={features}>
      <App />
    </FeatureProvider>,
    document.getElementById('root'),
  )
})
```

### `useFeature`

Hook that is used to get a feature object from the context by its slug. Notice that it could be `undefined` because the context **only should contain the features that are enabled.**

#### Specification

```ts
interface UseFeature<F extends DefaultFeature = DefaultFeature> {
  (slug: string): F | undefined
}
```

#### Example

```tsx
// src/app.tsx
import { useFeature } from 'toggled'

function App() {
  const themeFF = useFeature('theme')

  return (
    <ThemeProvider theme={themeFF ? themeFF.settings.name : 'default'}>
      <Component />
    </ThemeProvider>
  )
}
```

### `useFlagQueryFn`

Hook that is used to get the magic function that can process a _flag query_.

#### Specification

```ts
interface UseFlagQueryFn {
  (): (query: FlagQuery) => boolean
}
```

#### Example

```tsx
import { useFlagQueryFn } from 'toggled'

export default function App() {
  const flagQueryFn = useFlagQueryFn()

  return (
    <Layout designV2={flagQueryFn({ 'design-v2': true, 'design-v1': false })}>
      {flagQueryFn('chat') && <ChatWidget>}
    </Layout>
  )
}
```

> For more use cases, [please go to the tests.](./test/index.spec.tsx)

### `useFlag`

Hook that is used to get a binary output based on the existence of a feature in the context. So, if the feature is in the context then the flag will be `true`, otherwise `false`.

> The `useFlagQueryFn` hook is used internally.

#### Specification

```ts
interface UseFlag {
  (query: FlagQuery): boolean
}
```

#### Example

```tsx
import { useFlag } from 'toggled'

export default function App() {
  const hasChat = useFlag('chat')

  const hasDesignV2Only = useFlag({ 'design-v2': true, 'design-v1': false })

  return (
    <Layout designV2={hasDesignV2Only}>
      {hasChat && <ChatWidget>}
    </Layout>
  )
}
```

### `<Flag />`

Component to apply conditional rendering using a `flagQuery`

#### Specification

```ts
interface FlagProps {
  flagQuery: FlagQuery
  children: React.ReactNode
}
```

#### Example

```tsx
import { Flag } from 'toggled'

export default function App() {
  return (
    <Flag flagQuery={{ 'design-v2': true, 'design-v1': false }}>
      <Layout designV2={hasDesignV2Only}>
        <Flag flagQuery="chat">
          <ChatWidget>
        </Flag>
      </Layout>
    </Flag>
  )
}
```

### `<Feature />`

Component to consume a feature object declaratively instead of `useFeature`

#### Specification

```ts
export interface FeatureProps {
  slug: string
  children(feature: DefaultFeature): React.ReactElement
}
```

#### Example

```tsx
import { Feature } from 'toggled'

export default function App() {
  return (
    <Feature slug="chat">
      {feature => {
        return <ChatWidget settings={feature.settings}>
      }}
    </Feature>
  )
}
```

## License

MIT © [Ricardo Q. Bazan](https://rcrd.space)
