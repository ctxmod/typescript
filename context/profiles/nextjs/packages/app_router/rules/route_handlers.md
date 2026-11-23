---
paths:
  - "**/app/**/route.ts"
---

# Route Handler Context

Route Handler conventions for route.ts files including params, body parsing, and caching.

Route handlers export named HTTP method functions (GET, POST, PUT, DELETE). `params` is a Promise and must be awaited. Body parsing is manual via `request.json()`. Non-GET handlers are never cached. GET handlers can opt into caching with `"use cache"` on extracted helper functions — the directive cannot appear directly inside a Route Handler export.

- [Route Handlers](../reference/route_handlers.md)
- [Cache Components](../reference/cache_components.md)
