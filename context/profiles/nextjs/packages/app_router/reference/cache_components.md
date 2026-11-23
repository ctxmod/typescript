# Cache Components

The Next.js 16 caching model — dynamic by default, explicit `"use cache"` boundaries, cache lifetimes and tags, and the invalidation API matched to the consistency a mutation needs.

Cache Components (`cacheComponents: true`) replaces the implicit caching of
earlier App Router versions. Since Next.js 16.4 it is the recommended model
for every application, new `create-next-app` projects enable it by default,
and it becomes the default in Next.js 17. Enable it together with
`partialPrefetching: true`, which 16.4 made part of the model.

## The Model

- **Everything is dynamic by default.** Pages, layouts, and route handlers
  run at request time unless a boundary says otherwise. Caching is opt-in,
  declared where it applies, never inferred from a `fetch` default.
- **`"use cache"` marks a cache boundary** at the top of a file, a component,
  or an async function. The compiler derives the cache key from the
  function's arguments and closed-over values; arguments must be
  serializable. Inside it, `cacheLife(profile)` sets the lifetime (built-in
  profiles such as `'hours'`, `'days'`, `'max'`, or custom profiles from
  `next.config`) and `cacheTag(tag)` attaches invalidation tags.
- **Static and dynamic stream in one response.** Cached subtrees prerender
  into the static shell; request-time components render behind a
  `<Suspense>` boundary and stream in. Reading request data (`cookies()`,
  `headers()`, `searchParams`, uncached I/O) outside a Suspense boundary or a
  cache boundary is a build error, which is what keeps a dynamic read from
  silently making a whole route dynamic.
- **Request APIs are async.** `params`, `searchParams`, `cookies()`,
  `headers()`, and `draftMode()` must be awaited; Next.js 16 removed the
  synchronous compatibility access.

## Invalidation: Pick the Consistency the Caller Needs

| API | Where | Semantics | Use for |
|---|---|---|---|
| `updateTag(tag)` | Server Functions only | Expires the tag and reads fresh data in the same request (read-your-writes) | Forms and settings where the user must see their own write |
| `revalidateTag(tag, profile)` | Server Functions, route handlers | Stale-while-revalidate under a `cacheLife` profile (`'max'` recommended) | Content that tolerates eventual consistency |
| `refresh()` | Server Functions only | Re-renders uncached data; touches no cache | Counters and status shown beside cached content |

The single-argument `revalidateTag(tag)` is deprecated; always pass the
profile. A mutation that matters to its author uses `updateTag`; choosing
`revalidateTag` there ships a stale read back to the person who just wrote.

## Guarding Static Routes and Prefetch Cost (16.4)

- `export const ensureStatic = 'navigation' | 'prefetch' | 'shell'` on a page
  or layout fails the build when dynamic content enters that stage — the
  guard for marketing, catalog, and documentation routes that must stay
  free of request-time compute. Set it on a layout to cover every page
  beneath.
- `await navigation()` (from `next/cache`) inside a component defers its
  cached content from prefetches to the actual navigation;
  `await prefetch()` defers content from the route shell to an explicit
  prefetch. Use them to keep `<Link prefetch>` affordable on link-heavy
  views instead of turning prefetching off.

## Rules That Follow

- One cache boundary per data concern, placed at the function that owns the
  read — not at the page — so lifetimes and tags stay legible.
- Never place `"use cache"` around code that reads per-user data without that
  data being part of the key; a user-specific read belongs in a request-time
  component behind Suspense.
- In route handlers the directive goes in an extracted helper, not in the
  exported HTTP-method function ([Route Handlers](route_handlers.md)).
- Migration from the previous model is a recorded change: the `next upgrade
  --agent` workflow and the Cache Components adoption skills exist for it;
  do not mix the removed `experimental.ppr` or route-level
  `experimental_ppr` exports back in.

## Related

- [App Router](app_router.md)
- [Server and Client Components](server_client.md)
- [Proxy, Configuration, and Instrumentation](proxy_config.md)

## Sources

- https://nextjs.org/blog/next-16 — Cache Components, `cacheComponents`, removed `experimental.ppr`/`dynamicIO`, `revalidateTag(tag, profile)`, `updateTag`, `refresh`, async request APIs (2025-10-21; accessed 2026-11-22)
- https://nextjs.org/blog/next-16-4 — Cache Components recommended for every app and default in Next.js 17, `partialPrefetching`, `ensureStatic`, `navigation()`/`prefetch()`, agent upgrade (2026-10-06; accessed 2026-11-22)
- https://nextjs.org/docs/app/getting-started/caching — caching guide for the Cache Components model (accessed 2026-11-22)
