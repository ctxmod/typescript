# Proxy, Configuration, and Instrumentation

Proxy.ts middleware, next.config.ts configuration, and instrumentation setup.

## Proxy (Next.js 16)

**Use `proxy.ts`, not `middleware.ts`**

Next.js 16 renamed middleware to `proxy.ts`. The export is `export const proxy = clerkMiddleware(...)` not `export default`. The old `middleware.ts` still works but is deprecated.

**Proxy runs on Node.js runtime only**

Next.js 16's proxy defaults to Node.js runtime. The `export const runtime` option is NOT available in proxy files and will throw an error. No Edge runtime configuration is supported for proxy.

**Always configure a matcher**

Auth logic or redirects in proxy can unintentionally block static assets. Always configure a matcher to exclude:
```
'/((?!api|_next/static|_next/image|favicon.ico|sitemap.xml|robots.txt).*)'
```

Without a matcher, proxy runs on every request including CSS/JS/image files.

**Server Actions bypass proxy if path is excluded**

A proxy matcher that excludes certain paths also skips Server Actions on those paths. Always verify authentication inside each Server Function — do not rely solely on proxy.

**Do not import shared modules or globals**

Proxy is deployed separately from render code. Do not import shared app modules, global state, or module-level singletons into `proxy.ts`. Pass data from proxy to routes via headers, cookies, rewrites, or URL modifications — not shared memory.

## Configuration

**Use `output: "standalone"` for containerized deployments**

Set `output: "standalone"` in `next.config.ts` for Docker/container deployments. The build output is `.next/standalone` + `.next/static` + `public/`. This reduces Docker image size by up to 90%.

**Use `serverExternalPackages` for native modules**

List native/binary packages in `serverExternalPackages` to prevent bundling. They are resolved at runtime from node_modules. Required for `better-sqlite3`, `ssh2`, `sharp`, etc.

**Enable Cache Components for `"use cache"`**

To use the `"use cache"` directive in Next.js 16, add `cacheComponents: true` to `next.config.ts`. This is the new caching model and enables the directive.

**Image optimization requires sharp in standalone mode**

When using `output: "standalone"`, image optimization may not work out of the box. Install `sharp` as a separate dependency and ensure it is available in the production image.

## Instrumentation

**Guard native module imports behind runtime check**

`instrumentation.ts` runs on every runtime (Node.js and Edge). Guard native module imports behind `process.env.NEXT_RUNTIME === 'nodejs'` before dynamically importing `instrumentation-node.ts`:

```typescript
if (process.env.NEXT_RUNTIME === 'nodejs') {
  const instrumentationNode = await import('./instrumentation-node');
  instrumentationNode.register();
}
```

Native modules like `better-sqlite3` or `ssh2` will break the build if imported in an Edge runtime context.

**`register()` must complete before server handles requests**

The `register()` function can be async but must resolve before the server is ready. Use it for initialization (DB connections, service wiring, monitoring setup) — not for long-running tasks.

## Thresholds

| Metric | Pass | Warn | Fail |
|--------|------|------|------|
| Proxy missing matcher | 0 | 0 | >0 |
| Native modules imported without runtime check | 0% | 0% | >0% |
| Standalone output without `serverExternalPackages` for native modules | 0% | 0% | >0% |

- KISS (engineering module, `principles` profile: `design/reference/kiss.md`)
