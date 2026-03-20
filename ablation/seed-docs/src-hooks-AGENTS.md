# AGENTS.md — src/hooks/

Rules and conventions for custom React hooks.

---

### Rule: Query key factory pattern
- **Rule:** Every hook file defines a `const keys = { all: ['name'] as const }` object at the top. All query and mutation operations reference keys from this object. This ensures consistency and makes cache invalidation reliable.
- **Verification:** Grep for hardcoded query key strings — they should only appear in the keys object.
- **Source:** Spec 21

### Rule: Optimistic update with rollback
- **Rule:** All mutations follow this pattern: `onMutate` saves previous data and sets optimistic cache; `onError` rolls back to previous; `onSuccess` shows toast; `onSettled` invalidates queries. This ensures the UI is responsive while staying correct on failure.
- **Verification:** Every `useMutation` must have `onMutate`, `onError`, and `onSettled` handlers.
- **Source:** Spec 21

### Rule: Array endpoints use full replacement
- **Rule:** For PUT endpoints that replace all items (focus areas, location preferences, deal breakers), the optimistic update replaces the entire array — not a merge. The mutation function receives the full update payload.
- **Verification:** `setQueryData` callback returns `newData.items`, not `{ ...old, ...newData }`.
- **Source:** Spec 21

### Rule: Profile update filters undefined values
- **Rule:** The `useUpdateProfile` optimistic update filters out `undefined` entries before spreading onto the cached `UserProfileResponse`. This prevents nullable optional fields from overwriting required fields with undefined.
- **Verification:** Code review — `Object.entries(newData).filter(([, v]) => v !== undefined)` before spread.
- **Source:** Spec 21, L004

### Rule: Infinite query cache updates use setQueriesData
- **Rule:** For optimistic updates on infinite queries (feed), use `queryClient.setQueriesData` with the `feedKeys.all` prefix to update across all focus area variants. The `updateJobInFeedCache` helper maps over all pages to find and update matching job cards.
- **Verification:** Save/pass mutations use `setQueriesData({ queryKey: feedKeys.all })`, not `setQueryData` with a specific key.
- **Source:** Spec 19

### Rule: View tracking deduplication via Set ref
- **Rule:** `useRecordView` deduplicates API calls using a `Set<string>` stored in a ref. Once a job ID is recorded, subsequent calls with the same ID are no-ops. This prevents spamming the API when cards re-enter the viewport.
- **Verification:** `viewedRef.current.has(jobId)` check before `mutate(jobId)`.
- **Source:** Spec 19

### Rule: Feed refresh is async with delayed invalidation
- **Rule:** `POST /v1/feed/refresh` returns immediately with `{"status": "queued"}`. The hook shows a toast and delays cache invalidation by 2 seconds to give the backend time to process. No polling — user pull-to-refresh or revisit later.
- **Verification:** `setTimeout(() => { invalidate... }, 2000)` in `useRefreshFeed.onSuccess`.
- **Source:** Spec 19, HANDOFF Q6

### Rule: No success toast on redirecting mutations
- **Rule:** Mutations that redirect on success (e.g., `useCompleteOnboarding`) should NOT show a success toast — the redirect provides sufficient feedback. Error toasts are still required.
- **Verification:** `useCompleteOnboarding` has `onError` toast but no `onSuccess` toast.
- **Source:** Spec 21

### Rule: Debounced mutation with flush callback
- **Rule:** Mutations with frequent updates (e.g., resume inline editing) use a timer ref for debouncing and expose both `debouncedSave` (delayed) and `flushSave` (immediate, returns Promise via `mutateAsync`). Clear the previous timer before setting a new one. The flush callback is used in approval flows to ensure edits are persisted before download.
- **Verification:** `useRef<ReturnType<typeof setTimeout>>` with `clearTimeout` before each new timer. Both `debouncedSave` and `flushSave` exported.
- **Source:** Spec 20

### Rule: Download mutation with blob handling
- **Rule:** File downloads use `api.download()` → `.blob()` → `URL.createObjectURL()` → temporary anchor element with programmatic click → `URL.revokeObjectURL()`. Sanitize filename to remove special characters. Always revoke the object URL after triggering download.
- **Verification:** Download triggers browser save dialog. No orphaned object URLs (revoke called).
- **Source:** Spec 20

### Rule: Infinite query with cursor pagination
- **Rule:** Paginated lists use `useInfiniteQuery` with `initialPageParam: undefined`, `getNextPageParam` returning `lastPage.next_cursor ?? undefined`. Flatten pages via `useMemo` + `flatMap`. Query params include `page_size` and optional `cursor`.
- **Verification:** `useInfiniteQuery` config has `initialPageParam: undefined` and `getNextPageParam`. Consumer uses `flatMap` to merge pages.
- **Source:** Spec 22

### Rule: Mutations invalidate multiple related query keys
- **Rule:** When a mutation affects both list and detail views (e.g., creating an application), invalidate both the `all` key and related keys like `stats`. When updating a detail record, invalidate detail AND list keys.
- **Verification:** `onSuccess` calls `invalidateQueries` for at least 2 query key patterns.
- **Source:** Spec 22

### Rule: staleTime for infrequently-changing data
- **Rule:** Queries for slow-changing data set explicit `staleTime`: 10 minutes for stats, 5 minutes for resume templates. This reduces unnecessary refetches and improves perceived performance.
- **Verification:** `staleTime` property present in `useQuery` config for stats and template queries.
- **Source:** Spec 22, Spec 21 Addendum

### Rule: Query key includes all filter parameters
- **Rule:** Queries with filters (status, role, search, page) include the full filter object in the query key factory. This ensures different filter combinations create separate cache entries. Factory pattern: `list: (filters) => [...base, filters] as const`.
- **Verification:** Changing a filter triggers a new fetch (different cache key), not a refetch of stale data.
- **Source:** Spec 23a, Spec 22

### Rule: Simple mutations need only mutationFn
- **Rule:** One-off mutations that don't need cache updates (e.g., trigger optimization, send chat message) can be bare `useMutation({ mutationFn })` with no `onSuccess`/`onError`/`onMutate`. The caller handles success/error in `.mutate()` callbacks. Don't add boilerplate when the hook consumer manages the side effects.
- **Verification:** Hook exports only `useMutation` with `mutationFn`. No toast or invalidation logic in the hook.
- **Source:** Spec 20

### Rule: Enabled guard for dependent queries
- **Rule:** Detail queries that depend on a parameter (user ID, application ID) use `enabled: !!paramName` to prevent API calls before the parameter is available. This avoids 404s during initial render when params are still resolving.
- **Verification:** `enabled` property checks truthiness of required param.
- **Source:** Spec 23a, Spec 22
