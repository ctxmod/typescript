# Proxy, Configuration, and Instrumentation

Proxy.ts middleware, next.config.ts configuration, and instrumentation setup.

## Proxy (Next.js 16)

**Use `proxy.ts`, not `middleware.ts`**

Next.js 16 renamed the middleware file convention to `proxy.ts` (or `.js`), at the project root or inside `src/`, beside `app/`. The file exports a single function, either as a default export or named `proxy`:

```typescript
import { NextResponse } from "next/server";
import type { NextRequest } from "next/server";

export function proxy(request: NextRequest) {
  return NextResponse.redirect(new URL("/home", request.url));
}

export const config = { matcher: "/about/:path*" };
```

A library-provided handler is assigned the same way (`export const proxy = authHandler(...)` or `export default authHandler(...)`); the `NextProxy` type types an arrow-function form. The codemod `npx @next/codemod@canary middleware-to-proxy .` renames the file and the function. `middleware.ts` still works but is deprecated and kept only for Edge-runtime use cases; it will be removed in a future version.

**Proxy runs on Node.js runtime only**

Proxy runs on the Node.js runtime. The `export const runtime` segment option is NOT available in proxy files and throws an error if set. A deployment that genuinely needs Edge-runtime interception stays on the deprecated `middleware.ts` with a recorded removal plan.

**Always configure a matcher**

Auth logic or redirects in proxy can unintentionally block static assets. Always configure a matcher to exclude:
```
'/((?!api|_next/static|_next/image|favicon.ico|sitemap.xml|robots.txt).*)'
```

Without a matcher, proxy runs on every request including CSS/JS/image files.

**Server Functions bypass proxy if path is excluded**

Server Functions (Server Actions) are not separate routes: they are POST requests to the route that uses them, so a matcher that excludes a path also skips Server Function calls on that path — and a matcher change or a refactor that moves a function silently removes coverage. Always verify authentication and authorization inside each Server Function — do not rely solely on proxy.

**`_next/data` is always proxied**

Even when a negative matcher excludes `_next/data`, proxy still runs for those requests, so a protected page's data route cannot be left unprotected by accident.

**Do not import shared modules or globals**

Proxy is deployed separately from render code. Do not import shared app modules, global state, or module-level singletons into `proxy.ts`. Pass data from proxy to routes via headers, cookies, rewrites, or URL modifications — not shared memory.

## Configuration

**Use `output: "standalone"` for containerized deployments**

Set `output: "standalone"` in `next.config.ts` for Docker/container deployments. The build output is `.next/standalone` + `.next/static` + `public/`. This reduces Docker image size by up to 90%.

**Use `serverExternalPackages` for native modules**

List native/binary packages in `serverExternalPackages` to prevent bundling. They are resolved at runtime from node_modules. Required for `better-sqlite3`, `ssh2`, `sharp`, etc.

**Enable Cache Components for `"use cache"`**

To use the `"use cache"` directive in Next.js 16, add `cacheComponents: true` to `next.config.ts` (it replaced the removed `experimental.ppr` and `experimental.dynamicIO` flags). Since 16.4 it is the recommended model for every app, `create-next-app` enables it by default, and it becomes the default in Next.js 17; enable `partialPrefetching: true` with it. The model and its APIs are in [Cache Components](cache_components.md).

**Top-level `turbopack` key; webpack only by flag**

Turbopack is the default bundler for `next dev` and `next build` since Next.js 16; its configuration lives under the top-level `turbopack` key (no longer `experimental.turbopack`). A custom webpack setup opts out per command with `--webpack` — record why, since that path forgoes the Turbopack-only features.

**React Compiler is a stable option, off by default**

`reactCompiler: true` (promoted from `experimental` in Next.js 16) enables automatic memoization through `babel-plugin-react-compiler`; it raises compile times because it runs on Babel, and the experimental `experimental.turbopackRustReactCompiler` (16.3+) runs a Rust port inside Turbopack instead.

**Removed configuration**

`serverRuntimeConfig`/`publicRuntimeConfig` are removed (use environment variables), as are AMP and the `next lint` command; `images.domains` is deprecated in favor of `images.remotePatterns`, and local `next/image` sources with query strings require `images.localPatterns`.

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

## Sources

- https://nextjs.org/docs/app/api-reference/file-conventions/proxy — proxy file convention: default or named `proxy` export, `NextProxy`, Node.js runtime only, matcher, Server Functions and `_next/data` behavior, codemod (docs updated 2026-11-18; accessed 2026-11-22)
- https://nextjs.org/blog/next-16 — `proxy.ts` rename, `middleware.ts` deprecated for Edge use cases, Turbopack default and top-level `turbopack` config, stable `reactCompiler`, removals (2025-10-21; accessed 2026-11-22)
- https://nextjs.org/blog/next-16-4 — Cache Components recommended for every app, default in `create-next-app`, default in Next.js 17, `partialPrefetching`, Rust React Compiler (2026-10-06; accessed 2026-11-22)
