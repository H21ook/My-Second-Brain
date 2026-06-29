# Redux Toolkit

[](https://github.com/reduxjs/redux-toolkit#redux-toolkit)

[![GitHub Workflow Status](https://camo.githubusercontent.com/85c2d17d6f67c91c551bea8aa7a9bb1ef2c94cc9ac98c47b167d10bf59abc906/68747470733a2f2f696d672e736869656c64732e696f2f6769746875622f616374696f6e732f776f726b666c6f772f7374617475732f72656475786a732f72656475782d746f6f6c6b69742f74657374732e796d6c3f7374796c653d666c61742d737175617265)](https://camo.githubusercontent.com/85c2d17d6f67c91c551bea8aa7a9bb1ef2c94cc9ac98c47b167d10bf59abc906/68747470733a2f2f696d672e736869656c64732e696f2f6769746875622f616374696f6e732f776f726b666c6f772f7374617475732f72656475786a732f72656475782d746f6f6c6b69742f74657374732e796d6c3f7374796c653d666c61742d737175617265) [![npm version](https://camo.githubusercontent.com/b8da9802ccc28c0f3e5b8903d9657e3d52be222e9d6f90c57f287efba363d163/68747470733a2f2f696d672e736869656c64732e696f2f6e706d2f762f4072656475786a732f746f6f6c6b69742e7376673f7374796c653d666c61742d737175617265)](https://www.npmjs.com/package/@reduxjs/toolkit) [![npm downloads](https://camo.githubusercontent.com/538f37b80b73dcf1142761f7b23257f9cf9ab62e4ee07ef6fc7d290e036e92b7/68747470733a2f2f696d672e736869656c64732e696f2f6e706d2f646d2f4072656475786a732f746f6f6c6b69742e7376673f7374796c653d666c61742d737175617265266c6162656c3d52544b2b646f776e6c6f616473)](https://www.npmjs.com/package/@reduxjs/toolkit)

**The official, opinionated, batteries-included toolset for efficient Redux development**

## Installation

[](https://github.com/reduxjs/redux-toolkit#installation)

### Create a React Redux App

[](https://github.com/reduxjs/redux-toolkit#create-a-react-redux-app)

The recommended way to start new apps with React and Redux Toolkit is by using [our official Redux Toolkit + TS template for Vite](https://github.com/reduxjs/redux-templates), or by creating a new Next.js project using [Next's `with-redux` template](https://github.com/vercel/next.js/tree/canary/examples/with-redux).

Both of these already have Redux Toolkit and React-Redux configured appropriately for that build tool, and come with a small example app that demonstrates how to use several of Redux Toolkit's features.

```shell
# Vite with our Redux+TS template
# (using the `degit` tool to clone and extract the template)
npx degit reduxjs/redux-templates/packages/vite-template-redux my-app

# Next.js using the `with-redux` template
npx create-next-app --example with-redux my-app
```

We do not currently have official React Native templates, but recommend these templates for standard React Native and for Expo:

- [https://github.com/rahsheen/react-native-template-redux-typescript](https://github.com/rahsheen/react-native-template-redux-typescript)
- [https://github.com/rahsheen/expo-template-redux-typescript](https://github.com/rahsheen/expo-template-redux-typescript)

### An Existing App

[](https://github.com/reduxjs/redux-toolkit#an-existing-app)

Redux Toolkit is available as a package on NPM for use with a module bundler or in a Node application:

```shell
# NPM
npm install @reduxjs/toolkit

# Yarn
yarn add @reduxjs/toolkit
```

If you use an AI agent, run `npx @tanstack/intent@latest install` to install agent skills.

The package includes a precompiled ESM build that can be used as a [`<script type="module">` tag](https://unpkg.com/@reduxjs/toolkit/dist/redux-toolkit.browser.mjs) directly in the browser.

## Documentation

[](https://github.com/reduxjs/redux-toolkit#documentation)

The Redux Toolkit docs are available at **[https://redux-toolkit.js.org](https://redux-toolkit.js.org/)**, including API references and usage guides for all of the APIs included in Redux Toolkit.

The Redux core docs at [https://redux.js.org](https://redux.js.org/) includes the full Redux tutorials, as well usage guides on general Redux patterns.

## Purpose

[](https://github.com/reduxjs/redux-toolkit#purpose)

The **Redux Toolkit** package is intended to be the standard way to write Redux logic. It was originally created to help address three common concerns about Redux:

- "Configuring a Redux store is too complicated"
- "I have to add a lot of packages to get Redux to do anything useful"
- "Redux requires too much boilerplate code"

We can't solve every use case, but in the spirit of [`create-react-app`](https://github.com/facebook/create-react-app), we can try to provide some tools that abstract over the setup process and handle the most common use cases, as well as include some useful utilities that will let the user simplify their application code.

Because of that, this package is deliberately limited in scope. It does _not_ address concepts like "reusable encapsulated Redux modules", folder or file structures, managing entity relationships in the store, and so on.

Redux Toolkit also includes a powerful data fetching and caching capability that we've dubbed "RTK Query". It's included in the package as a separate set of entry points. It's optional, but can eliminate the need to hand-write data fetching logic yourself.

## What's Included

[](https://github.com/reduxjs/redux-toolkit#whats-included)

Redux Toolkit includes these APIs:

- `configureStore()`: wraps `createStore` to provide simplified configuration options and good defaults. It can automatically combine your slice reducers, add whatever Redux middleware you supply, includes `redux-thunk` by default, and enables use of the Redux DevTools Extension.
- `createReducer()`: lets you supply a lookup table of action types to case reducer functions, rather than writing switch statements. In addition, it automatically uses the [`immer` library](https://github.com/mweststrate/immer) to let you write simpler immutable updates with normal mutative code, like `state.todos[3].completed = true`.
- `createAction()`: generates an action creator function for the given action type string. The function itself has `toString()` defined, so that it can be used in place of the type constant.
- `createSlice()`: combines `createReducer()` + `createAction()`. Accepts an object of reducer functions, a slice name, and an initial state value, and automatically generates a slice reducer with corresponding action creators and action types.
- `combineSlices()`: combines multiple slices into a single reducer, and allows "lazy loading" of slices after initialisation.
- `createListenerMiddleware()`: lets you define "listener" entries that contain an "effect" callback with additional logic, and a way to specify when that callback should run based on dispatched actions or state changes. A lightweight alternative to Redux async middleware like sagas and observables.
- `createAsyncThunk()`: accepts an action type string and a function that returns a promise, and generates a thunk that dispatches `pending/resolved/rejected` action types based on that promise
- `createEntityAdapter()`: generates a set of reusable reducers and selectors to manage normalized data in the store
- The `createSelector()` utility from the [Reselect](https://github.com/reduxjs/reselect) library, re-exported for ease of use.

For details, see [the Redux Toolkit API Reference section in the docs](https://redux-toolkit.js.org/api/configureStore).

## RTK Query

[](https://github.com/reduxjs/redux-toolkit#rtk-query)

**RTK Query** is provided as an optional addon within the `@reduxjs/toolkit` package. It is purpose-built to solve the use case of data fetching and caching, supplying a compact, but powerful toolset to define an API interface layer for your app. It is intended to simplify common cases for loading data in a web application, eliminating the need to hand-write data fetching & caching logic yourself.

RTK Query is built on top of the Redux Toolkit core for its implementation, using [Redux](https://redux.js.org/) internally for its architecture. Although knowledge of Redux and RTK are not required to use RTK Query, you should explore all of the additional global store management capabilities they provide, as well as installing the [Redux DevTools browser extension](https://github.com/reduxjs/redux-devtools), which works flawlessly with RTK Query to traverse and replay a timeline of your request & cache behavior.

RTK Query is included within the installation of the core Redux Toolkit package. It is available via either of the two entry points below:

```ts
import { createApi } from '@reduxjs/toolkit/query'

/* React-specific entry point that automatically generates
   hooks corresponding to the defined endpoints */
import { createApi } from '@reduxjs/toolkit/query/react'
```

### What's included

[](https://github.com/reduxjs/redux-toolkit#whats-included-1)

RTK Query includes these APIs:

- `createApi()`: The core of RTK Query's functionality. It allows you to define a set of endpoints describe how to retrieve data from a series of endpoints, including configuration of how to fetch and transform that data. In most cases, you should use this once per app, with "one API slice per base URL" as a rule of thumb.
- `fetchBaseQuery()`: A small wrapper around fetch that aims to simplify requests. Intended as the recommended baseQuery to be used in createApi for the majority of users.
- `<ApiProvider />`: Can be used as a Provider if you do not already have a Redux store.
- `setupListeners()`: A utility used to enable refetchOnMount and refetchOnReconnect behaviors.

See the [**RTK Query Overview**](https://redux-toolkit.js.org/rtk-query/overview) page for more details on what RTK Query is, what problems it solves, and how to use it.

## Contributing

[](https://github.com/reduxjs/redux-toolkit#contributing)

Please refer to our [contributing guide](https://github.com/reduxjs/redux-toolkit/blob/master/CONTRIBUTING.md) to learn about our development process, how to propose bugfixes and improvements, and how to build and test your changes to Redux Toolkit.