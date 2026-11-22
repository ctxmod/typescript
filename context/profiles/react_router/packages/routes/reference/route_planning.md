# Route Planning

Workflow for planning route module structure, data requirements, code splitting, and error boundaries.

Procedure for designing route modules before implementation. Route modules in React Router framework mode serve as full-stack boundaries encompassing data loading, mutations, UI composition, and error handling. Planning upfront prevents loader waterfalls, missing error boundaries, and oversized route files.

## Prerequisites

Gather before starting:

- **URL structure**: The path pattern this route handles, including dynamic segments.
- **Data requirements**: Database queries, API calls, or external services the route needs.
- **User actions**: Mutations (create, update, delete) the route supports.
- **Error scenarios**: Expected failures (404, validation) and unexpected failures (timeouts, server errors).
- **Layout context**: Whether the route nests inside a layout route or stands alone.

## Planning Workflow

### 1. Define Route Boundaries

Each route module owns one URL segment and its associated data. Determine what this route is responsible for:

| Export | Purpose | Runs On |
|--------|---------|---------|
| `loader` | Data fetching for GET requests | Server only |
| `action` | Mutation handling for POST/PUT/DELETE | Server only |
| `Component` | UI rendering | Server + Client |
| `ErrorBoundary` | Error UI for this route | Server + Client |
| `meta` | Document metadata | Server + Client |
| `links` | Stylesheet and preload links | Server + Client |

Server-only exports (`loader`, `action`) are stripped from client bundles automatically. Route modules act as glue: they wire data to components. Push presentational and feature logic into colocated components.

- [Route Module Design](route_module_design.md)

### 2. Identify Data Dependencies

List every data source the route needs. Classify each as independent or dependent:

| Data Source | Type | Dependency |
|-------------|------|------------|
| User profile | DB query | Independent |
| User's products | DB query | Independent (same user ID) |
| Product reviews | DB query | Depends on product IDs |
| External pricing API | HTTP call | Independent |

Independent sources parallelize with `Promise.all()`. Dependent sources require sequential execution or streaming.

### 3. Plan Fetching Strategy

Apply fetching patterns based on data classification:

**Parallel fetching** for independent sources:
```typescript
const [user, products] = await Promise.all([
  db.user.findUnique({ where: { id: params.userId } }),
  db.product.findMany({ where: { userId: params.userId } }),
]);
```

**Streaming** for slow secondary data:
```typescript
const product = await db.product.findUnique({ where: { id: params.pid } });
const reviewsPromise = db.review.findMany({ where: { productId: params.pid } });
return { product, reviewsPromise }; // Router streams the promise
```

Use streaming when secondary data takes >100ms and the primary content renders meaningfully without it. The component wraps streamed data in `<Suspense>` with `<Await>`.

### 4. Plan Code Splitting

React Router framework mode automatically splits code by route; v8 also splits each route module's exports (`splitRouteModules`, on by default) so `clientLoader` and the component load independently. Additional splitting decisions:

| Content | Approach |
|---------|----------|
| Route module | Automatic split (framework mode) |
| Heavy UI (charts, editors) | Place in leaf routes for natural split boundaries |
| Shared components | Bundled with first importing route, cached for subsequent |
| Server utilities | `.server.ts` suffix excludes from client bundle |

For granular lazy loading (v7.5+), route exports split individually:

```typescript
{
  path: "/products",
  lazy: {
    Component: () => import("./products/Component"),
    loader: () => import("./products/loader"),
    ErrorBoundary: () => import("./products/ErrorBoundary"),
  }
}
```

`HydrateFallback` loads only during initial render, not during client navigations.

### 5. Define Error Boundaries

Plan error handling at two levels:

**Route-level boundary**: Catches errors from this route's loader, action, and component. Every route with data fetching should export an `ErrorBoundary`.

**Root boundary**: Catches errors that escape all route boundaries. Lives in `root.tsx` and renders a full HTML document since layout may not be available.

Map error types to responses:

| Error Type | Source | Status Code | UI |
|------------|--------|-------------|-----|
| Missing resource | `throw data("Not Found", { status: 404 })` | 404 | Specific "not found" message |
| Validation failure | `return { errors: result.error }` | 200 | Inline form errors |
| Auth failure | `throw redirect("/login")` | 302 | Redirect, no boundary |
| Server error | Unhandled exception | 500 | Generic error with retry |
| Bot timeout | AbortController timeout | 504 | Minimal error HTML |

- [Route Error Handling](error_handling.md)

### 6. Plan File Organization

Structure route files based on complexity:

**Simple route** (single file):
```
routes/
  about.tsx              # Loader + Component + ErrorBoundary in one file
```

**Complex route** (folder with colocated files):
```
routes/
  products.$pid/
    route.tsx            # Route module (loader, action, Component, ErrorBoundary)
    product-card.tsx     # Presentation component
    product-form.tsx     # Form component
    get-product.server.ts  # Server utility
```

Route modules under 150 lines stay in a single file. Beyond 150 lines, extract components into colocated files. Beyond 250 lines, the route likely handles multiple concerns and should be split.

### 7. Generate Types

React Router generates type-safe route types. Configure TypeScript to include generated types:

```json
{
  "include": [".react-router/types/**/*"],
  "compilerOptions": {
    "rootDirs": [".", "./.react-router/types"]
  }
}
```

Use `Route.ComponentProps`, `Route.LoaderArgs`, and `Route.ActionArgs` instead of manual type definitions. Generated types ensure loader return types flow through to component props without manual synchronization.

## Performance Checklist

- [ ] Independent data sources parallelized with `Promise.all()`
- [ ] Single queries with joins preferred over multiple round-trips
- [ ] Slow secondary data returned as promises for streaming
- [ ] Infrastructure co-located with deployment (sub-30ms DB latency target)
- [ ] Caching headers set for static or slowly-changing data
- [ ] Heavy UI components placed in leaf routes
- [ ] Server utilities use `.server.ts` suffix

## Validation Checklist

- [ ] Route module owns one URL segment
- [ ] Every data-fetching route exports `ErrorBoundary`
- [ ] Error types mapped to status codes and UI
- [ ] Route module under 150 lines (extract if larger)
- [ ] Generated `Route.*` types used throughout
- [ ] Layout nesting matches URL hierarchy

- [Loader and Action Implementation](loader_action.md)

## Sources

- https://remix.run/blog/react-router-v8 — React Router v8: Node 22.22+/React 19.2.7+/Vite 7+ baselines, ESM-only, `v8_*` flags adopted, `splitRouteModules` top-level and on by default, `react-router-dom` removed, Cloudflare dev proxy removed for `@cloudflare/vite-plugin`, RSC still unstable (2026-06-17; accessed 2026-11-22)
