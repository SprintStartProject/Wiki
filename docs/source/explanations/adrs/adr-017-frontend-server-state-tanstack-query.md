# ADR-017: TanStack Query for Frontend Server State

| Field | Value |
|---|---|
| **Status** | `Accepted` |
| **Date** | 2026-09-12 |
| **Deciders** | Frontend Team |
| **Supersedes** | - |
| **Superseded by** | - |

---

## Context

Until September 2026 every page and feature hook in `sprintstart-frontend` loaded its backend data by hand. The common pattern was the shared `useFetch` hook, a `useState` + `useEffect` pair that ran a loader on mount and whenever its `deps` changed. Several hooks went further and hand-rolled their own refs, timers and state machines (for example the sidebar badge counter in `useRateLimitedRead`).

This had three visible problems:

- **No cache.** `useFetch` only had `deps`, never a cache key. Navigating back to a page that was loaded seconds earlier fetched everything again and showed a spinner again.
- **Duplicate requests.** A dashboard widget and the page it summarizes read the same data, but each ran its own request because nothing connected them.
- **Inconsistent refresh behavior.** Forcing a reload meant adding a `refreshKey` to `deps`. Polling, focus refresh and rate limiting were implemented differently in each hook, and a background refresh that failed could replace good data with an error state.

The app also grew a global project switcher. Every project-scoped read has to be keyed by the selected project, otherwise a project switch keeps showing the previous project's data.

The frontend has no global store (see the architecture doc in the frontend repository). Client and UI state is held in React Context providers (`AuthProvider`, `ThemeProvider`, `ProjectProvider`, `ToastProvider`, and others). The question here is only about *server state*: data that is owned by the backend and mirrored in the browser.

## Decision Drivers

- Revisiting a page shows cached data immediately instead of a spinner.
- Components that read the same data share one request.
- Project and user switches never serve another project's or user's cached data.
- Refresh, polling and error behavior are defined once instead of per hook.
- Migration can happen hook by hook without changing the public return shape that pages already use.
- The roughly 230 existing test files keep working without being touched.
- Works with the existing declarative React Router setup and the native `fetch` based `apiClient`.

## Considered Options

- **Option A** - Keep hand-rolled hooks (`useFetch` and per-hook logic)
- **Option B** - TanStack Query (`@tanstack/react-query` v5)
- **Option C** - SWR
- **Option D** - React Router data APIs (`loader` / `action`)

## Decision

**Chosen option: Option B** - TanStack Query, because it provides caching, request sharing, invalidation, prefetching and mutations in one library and can be introduced behind the existing hook signatures.

## Rationale

TanStack Query solves exactly the problems listed above with a cache keyed by query keys. Request deduplication, stale-while-revalidate, refetch on focus and reconnect, polling, invalidation and prefetching are built in, so the hand-rolled versions could be deleted instead of maintained.

It does not require a new data-loading model. The app keeps its declarative `<Route element={...}>` routing and its service modules. A loader function from a service is passed to `useQuery` unchanged. This allowed the migration to be done hook by hook while each hook kept its public return shape, so pages did not have to change.

SWR covers the read side similarly, but mutations, invalidation by key prefix and prefetching are less complete, and the team needs those for the board, the admin pages and the sidebar prefetch. React Router data APIs would have required moving to the data router and rewriting every page's loading logic at once, and they do not share data between a widget and a page that are not on the same route.

## Pros and Cons of the Options

### Option A - Keep hand-rolled hooks

- ✅ No new dependency
- ✅ Every hook is fully under the team's control
- ❌ No cache, so every revisit refetches and shows a spinner
- ❌ Each hook reimplements polling, rate limiting and error handling slightly differently
- ❌ Widget and page cannot share a request

### Option B - TanStack Query

- ✅ Shared cache with request deduplication and stale-while-revalidate
- ✅ Invalidation, optimistic updates (`setQueryData`), mutations and prefetching built in
- ✅ Can be introduced behind existing hook signatures, migration step by step
- ✅ Widely used, well documented, good TypeScript support
- ❌ New concepts for the team (query keys, `staleTime`, `isLoading` vs. `isFetching`)
- ❌ Query keys become a contract that has to be kept consistent across hooks, invalidations and prefetches

### Option C - SWR

- ✅ Small API, easy to learn
- ✅ Stale-while-revalidate and focus refresh built in
- ❌ Weaker mutation and invalidation support than TanStack Query
- ❌ No equivalent to `prefetchQuery` with the same level of control

### Option D - React Router data APIs

- ✅ Data loading is tied to the route, no extra library
- ❌ Requires migrating from the declarative router to the data router and rewriting all pages at once
- ❌ Does not share data between a dashboard widget and a different page
- ❌ No built-in cache across navigations

## Consequences

**Positive:**
- A revisit within 30 seconds renders from cache with no request and no spinner.
- Dashboard widgets and their pages share one cache entry and one request.
- The sidebar warms both the lazy page module and its main query on `pointerdown`, so most navigations land on data that is already there.
- Hand-rolled polling and rate-limiting code was removed.

**Negative / Trade-offs:**
- Every developer has to understand query keys and the difference between "nothing to show yet" (`isLoading`) and "refreshing data already on screen" (`isFetching`).
- A wrong or missing key part (for example a forgotten `projectId`) leads to stale or cross-project data. The key factory exists to prevent this, but it only helps if it is used.
- Data written by one feature and read by another needs an explicit `invalidateQueries` or `setQueryData` call. Forgetting it leads to stale UI, which already happened a few times with the board cache after buddy and card writes.

**Follow-up actions:**
- [x] Add `QueryClientProvider` and the shared client (`src/services/queryClient.ts`)
- [x] Add the central query key factory (`src/services/queryKeys.ts`)
- [x] Migrate all `useFetch`, `useLiveFetch`, `useRateLimitedRead` and mutation hooks
- [x] Add sidebar prefetch (`src/services/routePrefetch.ts`)
- [ ] Remove `useFetch` itself (it has no callers left, only its result type is still imported by `useQueryFetch`)
- [ ] Update `docs/FRONTEND_ARCHITECTURE.md` in the frontend repository, which does not mention TanStack Query yet

## Implementation Guidelines

### Shared client

There is exactly one `QueryClient`, defined in `src/services/queryClient.ts` and provided in `App.tsx`. Its defaults apply to every query:

| Option | Value | Why |
|---|---|---|
| `staleTime` | 30 s | A revisit shortly after the first load is served from cache without a request. |
| `gcTime` | 5 min | Unused data is kept long enough for back-and-forth navigation. |
| `refetchOnWindowFocus` | `true` | Returning to the tab picks up changes made elsewhere. |
| `refetchOnReconnect` | `true` | Same after a network drop. |
| `retry` | `1` | One flaky request is retried, a request that is really down (for example an expired session) does not stack up three retries of latency. |

The cache is cleared with `queryClient.clear()` on logout in `AuthProvider`, so a different user signing in on the same tab never sees the previous user's data.

### Query keys

All keys come from the factory in `src/services/queryKeys.ts`. Hooks, invalidations and prefetches import keys from there and never write key arrays inline.

- Every **project-scoped** key contains the `projectId`.
- Every **user-scoped** key contains the user's `profile.id`.
- Keys are hierarchical, so a prefix like `queryKeys.starterWork.pool()` can invalidate all its children (`poolByStatus(...)`) at once.

### Which hook to use

| Situation | Hook | Behavior |
|---|---|---|
| Normal page or widget read | `useQueryFetch` (`src/hooks/useQueryFetch.ts`) | Same shape as the old `useFetch` (`data`, `loading`, `error`) plus `refetch`, `isFetching` and `refetchError`. `loading` is only true when there is nothing to show yet. |
| Panel that should stay current on its own | `useLiveFetch` (`src/hooks/useLiveFetch.ts`) | Polls every 30 s while the tab is visible, at most every 10 s. Data stays on screen during a refresh (`revalidating`), a failed background refresh does not raise `error`. A key change starts from empty, because another project's data is wrong, not just old. |
| Small read in the app shell (badge, count, dot) | `useRateLimitedRead` (`src/hooks/useRateLimitedRead.ts`) | For components that never unmount, like the sidebar. Rechecks on route change and tab focus, at most every 5 s. |
| Writes, optimistic updates, custom `select` | `useQuery` / `useMutation` directly | Use `invalidateQueries` or `setQueryData` with keys from the factory after a write. |
| Anything new | not `useFetch` | `useFetch` is deprecated and has no callers. |

### Prefetching

`src/services/routePrefetch.ts` is called from `SidebarNavLink` on `pointerdown`. For every sidebar route it loads the lazy page module. For routes with one dominant query it also calls `prefetchQuery` with the same key and the same exported loader function that the page's own hook uses. When a page's hook or key changes, its entry in `routePrefetch.ts` has to change with it.

### Tests

`vite.config.ts` aliases `@testing-library/react` to `tests/unit/setup/rtl.tsx`. That wrapper gives every `render()` and `renderHook()` its own fresh `QueryClient` with `retry: false` and `gcTime: 0`, so tests are isolated from each other and error states show up immediately. Test files import `@testing-library/react` as usual and do not need to know about TanStack Query.

## Links

- [ADR Index](index.md)
- [ADR-011: Frontend Framework](adr-011-frontend-framework.md)
- [TanStack Query documentation](https://tanstack.com/query/latest)
- [sprintstart-frontend](https://github.com/SprintStartProject/sprintstart-frontend)
