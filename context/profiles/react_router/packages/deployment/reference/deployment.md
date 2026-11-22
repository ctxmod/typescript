# Deployment

Cloudflare Workers deployment patterns for SSR streaming applications.

React Router v7 SSR applications deploy to Cloudflare Workers using the Web Streams API for streaming server-side rendering. The deployment architecture centers on five configuration files and a worker entry point that bridges Cloudflare's runtime with React Router's request handling.

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

**react-router.config.ts** enables SSR and future flags:

```typescript
export default {
  ssr: true,
  future: {
    v8_middleware: true,
    v8_splitRouteModules: true,
  },
};
```

The `v8_middleware` flag enables the middleware API for centralized request processing (i18n, auth). The `v8_splitRouteModules` flag enables route-level code splitting for optimal navigation performance.

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
  async fetch(request: Request, env: Env, ctx: ExecutionContext) {
    const response = await handler(request, { env, ctx });
    // Add security headers
    return response;
  },
};
```

Access Cloudflare platform bindings through the `env` parameter: KV for key-value storage, R2 for object storage, D1 for SQL databases, Durable Objects for stateful coordination. Use a type-safe context getter function with a string key (not Symbol) to avoid mismatch between worker and server bundles.

## Environment Variables

| Scope | Mechanism | Access |
|-------|-----------|--------|
| Local development | `.dev.vars` file | `env.VARIABLE_NAME` in loaders |
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
