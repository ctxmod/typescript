---
paths:
  - "*.ts"
  - "*.tsx"
  - "*.mts"
  - "*.cts"
---

# Server and Client Component Context

Server and Client Component boundaries, server-only guards, and children composition patterns.

Default to Server Components. Add `"use client"` only at leaf components that need state, event handlers, browser APIs, or lifecycle effects. Use `"use server"` only on mutation functions — it does not prevent client bundling. Use `"import 'server-only'"` on modules with sensitive server logic to create build-time errors if accidentally imported client-side. Pass Server Components as `children` or props to Client Components for interleaving. Rely on the React Compiler (`reactCompiler: true`) rather than hand-written memoization in new code.

- [Server and Client Components](../reference/server_client.md)
- [Cache Components](../reference/cache_components.md)
