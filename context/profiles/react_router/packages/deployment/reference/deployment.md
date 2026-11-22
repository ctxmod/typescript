# Deployment

Cloudflare Workers deployment patterns for SSR streaming applications.

React Router framework-mode SSR applications deploy to Cloudflare Workers using the Web Streams API for streaming server-side rendering. The deployment architecture centers on five configuration files and a worker entry point that bridges Cloudflare's runtime with React Router's request handling.

- Convention over configuration (engineering module, `principles` profile: `design/reference/convention_over_configuration.md`)

## Project Structure

```
project/
  src/                        # Application code
    root.tsx                  # Root layout, middleware, loader
    routes.ts                 # Route configuration
    entry.server.tsx          # SSR entry point
    entry.client.tsx          # Hydration entry point
  workers/
    app.ts                    # Worker fetch handler
  react-router.config.ts      # Framework configuration
  vite.config.ts              # Build tooling
  wrangler.jsonc              # Worker deployment config
  .dev.vars                   # Local environment variables
```

## Configuration

**react-router.config.ts** enables SSR:

```typescript
import type { Config } from "@react-router/dev/config";

export default {
  ssr: true,
  // splitRouteModules: true is the v8 default — route exports split per module
} satisfies Config;
```

React Router v8 made the former `future.v8_*` flags the behavior: middleware is always on (centralized request processing for i18n and auth), route-module splitting is a top-level `splitRouteModules` option enabled by default, and the Vite Environment API is always used. A v7 application turns each `future.v8_*` flag on and resolves its fallout before upgrading the major.

**vite.config.ts** composes three plugins:

```typescript
import { reactRouter } from "@react-router/dev/vite";
import { cloudflare } from "@cloudflare/vite-plugin";
import tailwindcss from "@tailwindcss/vite";

export default defineConfig({
  plugins: [tailwindcss(), reactRouter(), cloudflare()],
});
```

Order matters: Tailwind processes CSS first, React Router handles routing and SSR, Cloudflare adapts the build for Workers runtime.

## Streaming SSR

The server entry point uses `renderToReadableStream` from `react-dom/server.browser` -- the Web Streams API, not Node.js streams. Cloudflare Workers run on V8 without Node.js APIs.

Bot detection drives timeout strategy:

| Client Type | Timeout | Behavior | Failure Response |
|-------------|---------|----------|-----------------|
| Human | 5s | Stream progressively | Partial content with Suspense fallbacks |
| Bot | 15s | Wait for `allReady` | 504 if timeout (incomplete HTML harms SEO) |

```typescript
const stream = await renderToReadableStream(
  <I18nextProvider i18n={i18n}>
    <ServerRouter context={entryContext} url={request.url} />
  </I18nextProvider>,
  { signal: controller.signal, onError(error) { responseStatusCode = 500; } }
);

if (isBot) await stream.allReady;
```

Bots receive complete HTML for search engine indexing. Humans receive streaming content with progressive rendering, improving perceived performance.

## Worker Entry Point

The worker fetch handler bridges Cloudflare's request model with React Router:

```typescript
import { createRequestHandler } from "react-router";

const handler = createRequestHandler(/* build */);

export default {
  async fetch(request, env, ctx) {
    const routerContext = new RouterContextProvider();
    routerContext.set(cloudflareContext, { env, ctx });
    const response = await handler(request, routerContext);
    // Add security headers
    return response;
  },
} satisfies ExportedHandler<Env>;
```

Access Cloudflare platform bindings through the typed context (`context.get(cloudflareContext).env`): KV for key-value storage, R2 for object storage, D1 for SQL databases, Durable Objects for stateful coordination. Declare the context with `createContext` in one module that both the worker entry and the route modules import, so both sides hold the same context key; the full entry is in [Cloudflare Workers Deployment](cloudflare_workers.md).

## Environment Variables

| Scope | Mechanism | Access |
|-------|-----------|--------|
| Local development | `.dev.vars` file | `context.get(cloudflareContext).env.VARIABLE_NAME` in loaders |
| Public (client-visible) | `VITE_*` prefix in `wrangler.jsonc` `vars` | Built into client bundles |
| Secret (server-only) | Cloudflare Dashboard | `env.SECRET_NAME` in loaders/actions |
| Build-time | Cloudflare Dashboard build settings | Available during `vite build` |

`.dev.vars` replaces `.env.local` for Cloudflare development. Public `VITE_*` variables embed in client bundles during build. Secrets never appear in client code.

## Security Headers

Add security headers in the worker entry point:

| Header | Value | Purpose |
|--------|-------|---------|
| `X-Content-Type-Options` | `nosniff` | Prevent MIME sniffing |
| `X-Frame-Options` | `DENY` | Prevent clickjacking |
| `Referrer-Policy` | `strict-origin-when-cross-origin` | Control referrer leakage |
| `Strict-Transport-Security` | `max-age=31536000; includeSubDomains` | Enforce HTTPS |
| `Permissions-Policy` | `geolocation=(), camera=()` | Restrict browser features |
| `Content-Security-Policy` | `script-src 'self' 'nonce-...'` | Prevent XSS |

Generate CSP nonces per request using `crypto.randomUUID()`. Pass the nonce to `<ServerRouter>`, `<Scripts>`, and `<ScrollRestoration>` for strict script policies.

## Cloudflare Integrations

| Service | Use Case | Access Pattern |
|---------|----------|---------------|
| KV | Sessions, cache, feature flags | `env.MY_KV.get(key)` |
| R2 | File uploads, static assets | `env.MY_BUCKET.put(key, body)` |
| D1 | Relational data, queries | `env.MY_DB.prepare(sql).run()` |
| Durable Objects | Real-time collaboration, counters | `env.MY_DO.get(id)` |

Configure bindings in `wrangler.jsonc`. All bindings are available in loaders and actions through the Cloudflare context.

## Thresholds

| Metric | Pass | Warn | Fail |
|--------|------|------|------|
| TTFB (streaming start) | <600ms | 600ms-1s | >1s |
| Worker cold start | <50ms | 50-100ms | >100ms |
| Bundle size (client JS) | <200KB | 200-400KB | >400KB |
| Security headers implemented | All 6 | 4-5 | <4 |

## Related

- [Route Module Design](../../routes/reference/route_module_design.md)
- [Internationalization](../../i18n/reference/internationalization.md)

## Sources

- https://remix.run/blog/react-router-v8 — React Router v8: Node 22.22+/React 19.2.7+/Vite 7+ baselines, ESM-only, `v8_*` flags adopted, `splitRouteModules` top-level and on by default, `react-router-dom` removed, Cloudflare dev proxy removed for `@cloudflare/vite-plugin`, RSC still unstable (2026-06-17; accessed 2026-11-22)
- https://reactrouter.com/8.3.1/api/other-api/adapter.md — official Cloudflare entry: `createContext` + `RouterContextProvider` passed to `createRequestHandler` (accessed 2026-11-22)
