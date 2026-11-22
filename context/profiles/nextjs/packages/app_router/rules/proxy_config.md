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

Next.js 16 uses `proxy.ts` (not `middleware.ts`) with `export const proxy = ...` (not `export default`). Proxy runs on Node.js runtime only; no `export const runtime` is available. Always configure a matcher to exclude static assets. `instrumentation.ts` runs on every runtime — guard native module imports behind `process.env.NEXT_RUNTIME === "nodejs"` with dynamic import. `next.config.ts` uses `output: "standalone"` for containerized deployments and `serverExternalPackages` for native modules.

- [Proxy, Configuration, and Instrumentation](../reference/proxy_config.md)
