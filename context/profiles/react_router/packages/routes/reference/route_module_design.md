# Route Module Design

Route modules as unified full-stack boundaries for data, mutations, UI, and error handling.

Each route module serves as a full-stack boundary encompassing data loading, mutations, UI rendering, and error handling. Route modules are the primary organizational unit in React Router framework mode (v7 and the current v8).

- SSOT (engineering module, `principles` profile: `design/reference/ssot.md`)
- KISS (engineering module, `principles` profile: `design/reference/kiss.md`)

## Module Anatomy

A route module exports a strict set of functions, each with a defined execution context:

| Export | Context | Purpose |
|--------|---------|---------|
| `loader` | Server only | Data fetching for GET requests |
| `action` | Server only | Mutation handling for POST requests |
| `Component` | Server + Client | UI rendering with typed loader data |
| `ErrorBoundary` | Server + Client | Route-granular error UI |
| `clientLoader` | Client only | Browser-specific data (IndexedDB, client SDKs) |
| `clientAction` | Client only | Browser-specific mutations |
| `HydrateFallback` | Client only | Loading UI during initial hydration |
| `middleware` | Server | Request pipeline (auth, locale) that seeds typed context for loaders and actions |
| `clientMiddleware` | Client only | Navigation pipeline for client loaders and actions |

Server-only exports are stripped from client bundles automatically. React Router generates `Route.ComponentProps`, `Route.LoaderArgs`, and `Route.ActionArgs` types for end-to-end type safety without manual hook typing.

## Loader Patterns

Loaders fetch data on the server before rendering. Two patterns dominate production implementations.

**Parallel fetching** eliminates waterfalls by batching independent operations:

```typescript
export async function loader({ params }: Route.LoaderArgs) {
  const [user, products] = await Promise.all([
    db.user.findUnique({ where: { id: params.userId } }),
    db.product.findMany({ where: { userId: params.userId } }),
  ]);

  if (!user) throw data("Not Found", { status: 404 });
  return { user, products };
}
```

**Streaming** defers slow secondary data while sending critical content immediately. Return promises from loaders and wrap deferred content in `<Suspense>` with `<Await>`. This improves Time to First Byte without blocking the entire page on slow queries.

## Action Patterns

Actions handle POST mutations with automatic revalidation. After an action completes, React Router revalidates all loader data on the page, keeping the UI synchronized without manual cache invalidation.

Use `<Form>` for navigation mutations (creates a history entry) and `useFetcher` for non-navigation mutations (inline updates, deletions). Validate all inputs server-side with schema validation (Zod) using allowlists and length bounds.

## Error Boundaries

Export `ErrorBoundary` from route modules to catch failures at the nearest route boundary. Distinguish thrown `Response` objects (expected errors like 404) from runtime errors (unexpected failures). Ship a root error boundary in `root.tsx` as the final catch-all.

Route-granular error boundaries prevent a single failure from taking down the entire page. A broken sidebar does not affect the main content area when each section has its own boundary.

## Thresholds

| Metric | Pass | Warn | Fail |
|--------|------|------|------|
| Loader execution time | <30ms | 30-100ms | >100ms |
| Independent queries per loader | 1 (with joins) | 2-3 (parallel) | >3 or sequential |
| Route module size (lines) | <150 | 150-250 | >250 |

Infrastructure co-location drives the 30ms target: databases deployed in the same region as Workers provide sub-30ms latency.

## Colocation Principle

Route modules act as glue. They wire data to components and define error boundaries, but push presentational and feature logic into colocated component files. A route folder contains the route module alongside its dedicated components and server utilities:

```
src/routes/products.$pid/
  route.tsx               # Route module (loader, action, Component, ErrorBoundary)
  product-card.tsx        # Presentational component
  product-form.tsx        # Feature component
  get-product.server.ts   # Server utility
```

When route modules grow beyond 250 lines, extract components rather than adding complexity to the module itself. The route module remains thin -- a coordination layer between data and UI.

## Anti-Patterns

- **Loader waterfalls**: Sequential queries where parallel execution is possible. Use `Promise.all()`.
- **Missing error boundaries**: Unhandled errors propagate to root, taking down the entire page.
- **Client-side auth checks**: Validate authentication in loaders/actions on the server, never in components.
- **Large route modules**: Route modules exceeding 250 lines conflate coordination with implementation.
- **State mutations in loaders**: Loaders serve GET requests. State changes belong in actions using POST.

## Related

- DRY (engineering module, `principles` profile: `design/reference/dry.md`)
- [Component Composition](../../components/reference/component_composition.md)
- [State Management](../../components/reference/state_management.md)

## Sources

- https://remix.run/blog/react-router-v8 — React Router v8: Node 22.22+/React 19.2.7+/Vite 7+ baselines, ESM-only, `v8_*` flags adopted, `splitRouteModules` top-level and on by default, `react-router-dom` removed, Cloudflare dev proxy removed for `@cloudflare/vite-plugin`, RSC still unstable (2026-06-17; accessed 2026-11-22)
- https://reactrouter.com/how-to/middleware — typed `createContext`, `context.get`/`context.set`, `getLoadContext` returning a `RouterContextProvider` (React Router 8.5.2 docs; accessed 2026-11-22)
