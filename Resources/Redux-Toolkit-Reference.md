---
title: "Redux Toolkit Reference"
type: resource
status: draft
created: 2026-06-30
updated: 2026-06-30
tags:
  - resource
  - programming
  - frontend
  - redux
aliases:
  - RTK
  - Redux Toolkit
  - RTK Query
source:
  - https://github.com/reduxjs/redux-toolkit
  - https://redux-toolkit.js.org/
accessed: 2026-06-30
related:
  - "[[E-Geree-v3-State-Management]]"
  - "[[E-Geree-v3-RHF-Migration-Plan]]"
---

# Redux Toolkit Reference

## Summary

Redux Toolkit is the official opinionated toolset for writing Redux logic with less setup and boilerplate.

It packages the common Redux pieces needed for store setup, reducers, actions, async logic, normalized entity management, selectors, and optional data fetching through RTK Query.

## Problems It Solves

Redux Toolkit was created to address common Redux pain points:

- manual store configuration is verbose
- useful middleware and devtools setup require extra work
- reducer and action boilerplate grows quickly
- immutable updates are easy to write incorrectly by hand
- data fetching and cache management often become repetitive

## Main APIs

| API | Purpose |
|---|---|
| `configureStore()` | Creates a Redux store with practical defaults, middleware, and DevTools support. |
| `createSlice()` | Defines slice state, reducers, generated action creators, and action types together. |
| `createReducer()` | Writes reducer logic using a lookup table instead of switch statements. |
| `createAction()` | Generates typed action creators from action type strings. |
| `createAsyncThunk()` | Creates async thunks with pending, fulfilled, and rejected lifecycle actions. |
| `createEntityAdapter()` | Provides normalized state helpers and selectors for entity collections. |
| `createListenerMiddleware()` | Runs side effects in response to actions or state changes. |
| `combineSlices()` | Combines slices and supports lazy-loaded slices. |
| `createSelector()` | Re-exported selector memoization utility from Reselect. |

## RTK Query

RTK Query is included inside `@reduxjs/toolkit` as an optional data fetching and caching layer.

Use it when the app needs:

- API endpoint definitions
- generated request hooks for React
- request caching
- refetch on reconnect or focus
- less hand-written loading and error state logic

Important entry points:

```ts
import { createApi } from "@reduxjs/toolkit/query";
import { createApi } from "@reduxjs/toolkit/query/react";
```

## Setup Notes

For new React projects, the Redux docs recommend starting from official templates when possible:

- Vite Redux TypeScript template from `reduxjs/redux-templates`
- Next.js `with-redux` example

For existing apps:

```shell
npm install @reduxjs/toolkit
```

## When to Use

Use Redux Toolkit when:

- state is shared across distant parts of an app
- state transitions benefit from explicit actions
- debugging with Redux DevTools is valuable
- async workflows need consistent lifecycle handling
- API cache behavior should be centralized with RTK Query

## When Not to Use

Do not add Redux Toolkit only because a form or small component has local state.

Prefer local state or form libraries when:

- state belongs to one component
- form input state changes on every keystroke
- server data is already handled by another cache
- global action history does not add value

## Related Notes

- [[E-Geree-v3-State-Management|E-Geree v3 State Management]]
- [[E-Geree-v3-RHF-Migration-Plan|E-Geree v3 RHF Migration Plan]]

## References

- Redux Toolkit GitHub repository.
- Redux Toolkit official documentation.

## Personal Insights

Redux Toolkit is best treated as the standard Redux implementation, not as a separate alternative to Redux.

In form-heavy applications, keep the boundary clear: Redux Toolkit is strong for global state and workflow state, while form libraries are often better for field-level input state.
