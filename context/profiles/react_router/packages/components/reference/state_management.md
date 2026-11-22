# State Management

State architecture with clear separation of local, shared, server, and URL state.

State management follows a separation principle: each type of state has a designated location and tool. Mixing state types into a single global store creates coupling between unrelated concerns and complicates testing.

- SSOT (engineering module, `principles` profile: `design/reference/ssot.md`)
the context-efficiency (two loads) guidance in the writing_context module

## State Type Separation

| State Type | Location | Tool | Example |
|------------|----------|------|---------|
| Local component | Component | `useState` / `useReducer` | Form input values, toggle states |
| Shared UI | Context or store | Context API or Zustand | Theme preference, sidebar state |
| Server | Server cache | Route loaders / TanStack Query | User data, product listings |
| Form | Form instance | React Hook Form | Validation state, field errors |
| URL | URL | React Router | Search params, pagination, filters |

This separation maintains a single source of truth for each state category. Server state lives in loaders, not in client-side global stores. URL state lives in the URL, not in component state.

## Local State

Default to `useState` for state used by a single component. Prefer `useReducer` when transitions involve multiple related values. Local state should be the default choice before reaching for shared solutions.

State flows from parent to children through props. When siblings need the same state, lift it to the nearest common parent.

## Shared UI State

Two mechanisms serve shared UI state, selected by update frequency:

**Context API** suits relatively static data: theme, authentication status, locale, feature flags. Context consumers re-render on every value change, making it unsuitable for high-frequency updates.

**Zustand** suits frequently updating shared state: real-time counters, animation states, notification queues. Zustand provides selective subscriptions where components re-render only when their specific state slice changes. Its minimal API (~1KB) avoids provider wrapping.

```typescript
// Zustand: selective subscription
const count = useStore((state) => state.count);
// Component re-renders only when count changes, not on other store updates
```

### Decision Framework

| Criterion | Context API | Zustand |
|-----------|-------------|---------|
| Update frequency | Low (theme, auth, i18n) | High (real-time data, animations) |
| Consumer count | Few (<4 unrelated consumers) | Many distant consumers |
| Provider requirement | Yes (component tree scoped) | No (works outside React tree) |
| Bundle size impact | None (built-in) | Minimal (~1KB) |
| Render optimization | Manual (memoization required) | Automatic (selective subscriptions) |

## Server State

Server state belongs in route loaders. React Router's loader/action architecture provides server-side data fetching with automatic revalidation after mutations. This eliminates the need for client-side caching libraries in most cases.

For data needing client-side caching beyond route boundaries (prefetching, background refetching, optimistic updates), TanStack Query integrates with React Router. Create the query client in component state, never at module level, to prevent SSR cache sharing between requests.

```typescript
function App() {
  // Per-request client prevents cross-request cache contamination
  const [queryClient] = useState(() => new QueryClient());

  return (
    <QueryClientProvider client={queryClient}>
      <Outlet />
    </QueryClientProvider>
  );
}
```

## Form State

Manage form state with React Hook Form for validation, field tracking, and error display. Form state is transient -- it exists during form interaction and submits to a route action for server-side processing. Client-side validation provides immediate feedback; server-side validation in actions enforces security boundaries.

## URL State

Use search parameters for state that should survive refreshes, support sharing via URL, or drive server-side data fetching. Pagination, filters, and sort order belong in URL state.

```typescript
const [searchParams, setSearchParams] = useSearchParams();
const page = Number(searchParams.get("page") ?? "1");
```

URL state synchronizes naturally with loaders -- search parameters are available in `Route.LoaderArgs` through the request URL.

## Anti-Patterns

| Anti-Pattern | Problem | Resolution |
|--------------|---------|------------|
| Global store for server data | Duplicates loader data, stale cache | Use loaders as primary server state source |
| Module-level QueryClient | Shares cache across SSR requests | Create in `useState` per request |
| Context for high-frequency updates | Re-renders all consumers on every change | Use Zustand with selective subscriptions |
| Duplicated state across components | Synchronization bugs, inconsistency | Lift state to common parent or shared store |
| Form state in global store | Couples form lifecycle to application state | Use React Hook Form for transient form state |

## Thresholds

| Metric | Pass | Warn | Fail |
|--------|------|------|------|
| Global stores in application | 0-2 | 3-4 | >4 |
| Context providers in root | <5 | 5-8 | >8 |
| State lifted beyond 2 parent levels | Rare | Occasional | Frequent |
| Server data stored in client global state | 0 | 1-2 instances | Systematic |

## Related

- [Route Module Design](../../routes/reference/route_module_design.md)
- [Component Composition](component_composition.md)
