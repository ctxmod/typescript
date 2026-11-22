# App Router

App Router file conventions and rendering order for Next.js 16.

## File Conventions

**Root layout must define `<html>` and `<body>`**

The root `app/layout.tsx` is the only layout that should render HTML elements. Any route groups that remove the top-level layout must include `<html>` and `<body>` tags themselves.

**Rendering order is fixed**

The rendering order determines which boundary handles which state:

```
layout.tsx ──> template.tsx ──> error.tsx ──> loading.tsx ──> not-found.tsx ──> page.tsx
```

Each file creates a separate boundary with different semantics:
- `template.tsx` remounts on every navigation (unlike layout which persists)
- `loading.tsx` creates a Suspense boundary for page content below the layout
- `error.tsx` creates an Error Boundary for route-granular errors
- `not-found.tsx` creates an error boundary for 404s

**Route handlers and pages cannot coexist**

Never place `route.ts` and `page.tsx` in the same directory. Next.js treats `route.ts` as the lowest-level routing primitive — it owns all HTTP verbs for that segment, and the page will not be served. Route handlers must live in a sub-path (e.g., `app/api/users/route.ts`, not `app/users/route.ts`).

**Error boundaries require client components**

`error.tsx` files must include `"use client"` at the top. React Error Boundaries require client-side rendering to catch failures.

**Global error boundary replaces root layout**

`global-error.tsx` handles errors that bubble past the root layout. Unlike `error.tsx` which renders within the layout tree, `global-error.tsx` replaces everything and must define its own `<html>` and `<body>` tags.

**`params` and `searchParams` are Promises**

Pages, layouts, and route handlers receive `params` (and pages `searchParams`) as Promises; Next.js 16 removed synchronous access, so every read is `await params`. The generated `PageProps<'/route'>`, `LayoutProps<'/route'>`, and `RouteContext<'/route'>` helpers type them.

**Loading only suspends descendants**

`loading.tsx` suspends the page and nested content below the layout, not the layout's own data fetching. If `layout.tsx` fetches expensive data, `loading.tsx` will not show during that fetch. Move expensive layout data into a parallel route slot or use a separate client-side fetch.

## Implementation Patterns

### Parallel Route Slots

Every parallel route slot (e.g., `@dashboard`) must have a `default.tsx` fallback. When no content matches the slot, the fallback renders. Since Next.js 16 the build fails without it; a `default.tsx` that calls `notFound()` or returns `null` restores the earlier behavior explicitly.

### Intercepting Routes

Intercepting routes (`(.)folder`, `(..)folder`) only intercept when navigating via `next/link`. Direct URL access bypasses interception — design the actual route to work standalone as a fallback.

## Thresholds

| Metric | Pass | Warn | Fail |
|--------|------|------|------|
| Root layout without `<html>`/`<body>` | 0 | 1 | >1 |
| Route handler + page in same directory | 0 | 0 | >0 |
| Error boundary missing `"use client"` | 0 | 0 | >0 |
| Parallel route slot without `default.tsx` | 0 | 0 | >0 |

- KISS (engineering module, `principles` profile: `design/reference/kiss.md`)
- DRY (engineering module, `principles` profile: `design/reference/dry.md`)

## Sources

- https://nextjs.org/blog/next-16 — parallel route slots require `default.js` (build fails), sync `params`/`searchParams` access removed (2025-10-21; accessed 2026-11-22)
- https://nextjs.org/docs/app/api-reference/file-conventions — App Router file conventions (accessed 2026-11-22)
