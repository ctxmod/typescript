---
paths:
  - "proxy.ts"
  - "middleware.ts"
  - "next.config.ts"
  - "next.config.js"
  - "next.config.mjs"
  - "instrumentation.ts"
  - "instrumentation-node.ts"
  - "src/proxy.ts"
  - "src/middleware.ts"
  - "src/instrumentation.ts"
  - "src/instrumentation-node.ts"
  - "*/*/proxy.ts"
  - "*/*/middleware.ts"
  - "*/*/next.config.ts"
  - "*/*/next.config.js"
  - "*/*/next.config.mjs"
  - "*/*/instrumentation.ts"
  - "*/*/instrumentation-node.ts"
---

# Proxy, Configuration, and Instrumentation Context

Proxy.ts middleware, next.config.ts configuration, and instrumentation startup conventions.

Next.js 16 uses `proxy.ts` (`middleware.ts` is deprecated, kept only for Edge-runtime cases), exporting one function as a default export or named `proxy`. Proxy runs on Node.js runtime only; no `export const runtime` is available, and auth is still checked inside every Server Function. Always configure a matcher to exclude static assets. `instrumentation.ts` runs on every runtime — guard native module imports behind `process.env.NEXT_RUNTIME === "nodejs"` with dynamic import. `next.config.ts` uses `output: "standalone"` for containerized deployments, `serverExternalPackages` for native modules, and `cacheComponents: true` for the Cache Components model.

- [Proxy, Configuration, and Instrumentation](../reference/proxy_config.md)
- [Cache Components](../reference/cache_components.md)
