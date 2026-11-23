---
paths:
  - "*.ts"
  - "*.tsx"
  - "*.mts"
  - "*.cts"
---

# App Router Context

App Router file conventions, rendering order, and route segment coexistence rules.

App Router file conventions define rendering boundaries. Rendering order is: layout > template > error > loading > not-found > page. `route.ts` and `page.tsx` must not coexist in the same directory — route handlers own the entire HTTP surface for that segment. `error.tsx` must be a Client Component. Root layout is the sole owner of `<html>` and `<body>`. Every parallel route slot carries a `default.tsx` (builds fail without one), `params` is always awaited, and caching is explicit through Cache Components (`"use cache"`).

- [App Router](../reference/app_router.md)
- [Cache Components](../reference/cache_components.md)
