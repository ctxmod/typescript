# Cloudflare Workers Deployment

Deployment workflow for React Router v8 (framework mode) SSR applications on Cloudflare Workers with streaming, security headers, and binding configuration.

Procedure for deploying React Router v8 (framework mode) SSR applications to Cloudflare Workers. This covers the Worker fetch handler, streaming SSR configuration, security headers, environment variables, Cloudflare binding integration, and deployment workflow.

## Deployment Architecture

The deployment stack consists of four layers:

| Layer | File | Responsibility |
|-------|------|---------------|
| Worker entry | `workers/app.ts` | Request handling, security headers |
| SSR entry | `src/entry.server.tsx` | Streaming HTML rendering |
| Route modules | `src/routes/**/*.tsx` | Data loading, mutations, UI |
| Configuration | `wrangler.jsonc` | Bindings, variables, compatibility |

Cloudflare Workers use the Web Streams API (`renderToReadableStream`), not Node.js streams. React Router configures the correct streaming API based on the deployment environment.

## Deployment Workflow

### 1. Create Worker Fetch Handler

The Worker entry point receives all incoming requests and delegates to React Router's request handler. Since React Router v8, middleware is always on and the server request context is a `RouterContextProvider` — a plain `{ cloudflare: { env, ctx } }` object is no longer accepted. Declare a typed context once:

```typescript
// src/cloudflare-context.ts — one module, imported by the worker entry and by routes
import { createContext } from "react-router";

export const cloudflareContext = createContext<{
  env: Env;
  ctx: ExecutionContext;
}>();
```

Seed it per request in the worker:

```typescript
// workers/app.ts
import { RouterContextProvider, createRequestHandler } from "react-router";
import { cloudflareContext } from "../src/cloudflare-context";

const requestHandler = createRequestHandler(
  () => import("virtual:react-router/server-build"),
  import.meta.env.MODE
);

export default {
  async fetch(request, env, ctx) {
    const routerContext = new RouterContextProvider();
    routerContext.set(cloudflareContext, { env, ctx });
    const response = await requestHandler(request, routerContext);
    return addSecurityHeaders(response);
  },
} satisfies ExportedHandler<Env>;
```

Loaders, actions, and middleware read the bindings with `context.get(cloudflareContext).env`. A v7 application still on `context.cloudflare.env` migrates by enabling `future.v8_middleware` first, switching the worker to this shape, and then upgrading, the future-flag path React Router majors are designed around.

### 2. Configure Streaming SSR Entry

Set up server-side rendering with `renderToReadableStream` and bot-aware timeout handling:

```typescript
// src/entry.server.tsx
import { renderToReadableStream } from "react-dom/server";
import { ServerRouter } from "react-router";
import { isbot } from "isbot";

export default async function handleRequest(
  request: Request,
  responseStatusCode: number,
  responseHeaders: Headers,
  routerContext: EntryContext
) {
  const userAgent = request.headers.get("user-agent");
  const bot = isbot(userAgent);

  const controller = new AbortController();
  const timeout = setTimeout(
    () => controller.abort(),
    bot ? 15_000 : 5_000
  );

  let status = responseStatusCode;

  const body = await renderToReadableStream(
    <ServerRouter context={routerContext} url={request.url} />,
    {
      signal: controller.signal,
      onError(error: unknown) {
        status = 500;
        console.error({ url: request.url, isBot: bot, error });
      },
    }
  );

  clearTimeout(timeout);

  if (bot) {
    await body.allReady;
  }

  responseHeaders.set("Content-Type", "text/html");

  return new Response(body, {
    headers: responseHeaders,
    status,
  });
}
```

**Bot handling**: Bots receive complete HTML (`allReady`) because search engines penalize incomplete responses. Humans receive streaming HTML for faster perceived performance. If a bot request exceeds 15 seconds, return 504 rather than incomplete HTML.

### 3. Add Security Headers

Implement security headers in the Worker fetch handler:

```typescript
function addSecurityHeaders(response: Response): Response {
  const secure = new Response(response.body, response);

  secure.headers.set("X-Content-Type-Options", "nosniff");
  secure.headers.set("X-Frame-Options", "DENY");
  secure.headers.set("Referrer-Policy", "strict-origin-when-cross-origin");
  secure.headers.set(
    "Permissions-Policy",
    "geolocation=(), microphone=(), camera=()"
  );
  secure.headers.set(
    "Strict-Transport-Security",
    "max-age=31536000; includeSubDomains; preload"
  );

  return secure;
}
```

For applications requiring Content Security Policy with nonces, generate the nonce in the SSR entry and pass it to `<ServerRouter>`, `<Scripts>`, and `<ScrollRestoration>`.

### 4. Configure Cloudflare Bindings

Declare bindings in `wrangler.jsonc` for Cloudflare services:

```jsonc
{
  "name": "my-app",
  "main": "workers/app.ts",
  "compatibility_date": "2026-11-01", // the latest runtime date the app was tested against
  "kv_namespaces": [
    { "binding": "SESSION_KV", "id": "abc123" }
  ],
  "d1_databases": [
    { "binding": "MY_DB", "database_name": "production-db", "database_id": "def456" }
  ],
  "r2_buckets": [
    { "binding": "UPLOADS", "bucket_name": "user-uploads" }
  ]
}
```

Access bindings in loaders and actions through the context:

```typescript
export async function loader({ context }: Route.LoaderArgs) {
  const { env } = context.get(cloudflareContext);
  const session = await env.SESSION_KV.get("session-id");
  const users = await env.MY_DB.prepare("SELECT * FROM users").all();
  return { session, users: users.results };
}
```

Each binding type serves a specific purpose:

| Binding | Purpose | Access Pattern |
|---------|---------|---------------|
| KV | Key-value cache, sessions | `env.MY_KV.get()`, `env.MY_KV.put()` |
| D1 | SQL database | `env.MY_DB.prepare().all()` |
| R2 | Object storage (files) | `env.UPLOADS.get()`, `env.UPLOADS.put()` |
| Durable Objects | Stateful coordination | `env.MY_DO.get(id)` |

### 5. Configure Environment Variables

Manage variables across environments:

| Variable Type | Local | Production | Client Access |
|---------------|-------|------------|---------------|
| Public (`VITE_*`) | `.dev.vars` | `wrangler.jsonc` vars | Yes (bundled) |
| Secret | `.dev.vars` | Cloudflare Dashboard | No |
| Build-time | N/A | Dashboard > Build > Variables | At build only |

Secrets added through Cloudflare Dashboard are encrypted at rest and available only in Worker runtime. Never commit secrets to `wrangler.jsonc`.

### 6. Deploy

**Manual deployment**:

```bash
npm run build        # Build for production
npx wrangler deploy  # Deploy to Cloudflare
```

**CI/CD deployment**: Configure the build command and deploy step in your CI pipeline. Set `CLOUDFLARE_API_TOKEN` as a CI secret.

```bash
# CI pipeline steps
npm ci
npm run build
npx wrangler deploy --env production
```

### 7. Verify Deployment

After deployment, verify:

- SSR renders complete HTML (view source on deployed URL)
- Streaming works for human user-agents (check TTFB)
- Bot requests receive complete HTML (test with bot user-agent)
- Security headers present (check response headers)
- Bindings accessible (test data-fetching routes)
- Environment variables resolve (check runtime behavior)

## Monitoring

Cloudflare provides built-in observability:

| Feature | Purpose |
|---------|---------|
| Workers Analytics | Request count, CPU time, errors |
| Real-time Logs | Live request/response inspection |
| Logpush | Stream logs to external services |
| Web Analytics | Core Web Vitals (TTFB, FCP, LCP) |

Monitor loader execution times to catch performance regressions. Set alerts for error rate spikes and 504 (bot timeout) responses.

## Deployment Checklist

- [ ] `workers/app.ts` seeds a `RouterContextProvider` with the typed Cloudflare context
- [ ] `entry.server.tsx` uses `renderToReadableStream` (Web Streams API)
- [ ] Bot detection implemented with differentiated timeouts
- [ ] Security headers added in Worker fetch handler
- [ ] `wrangler.jsonc` declares all required bindings
- [ ] Secrets configured via Cloudflare Dashboard (not in config files)
- [ ] Public variables prefixed with `VITE_` for client access
- [ ] Production build tested with `npm run build`
- [ ] SSR verified on deployed URL (view source shows rendered HTML)
- [ ] Error logging includes URL, user-agent, and stack trace

## Related

- [Deployment](deployment.md)
- [Environment Setup](environment_setup.md)

## Sources

- https://remix.run/blog/react-router-v8 — React Router v8: Node 22.22+/React 19.2.7+/Vite 7+ baselines, ESM-only, `v8_*` flags adopted, `splitRouteModules` top-level and on by default, `react-router-dom` removed, Cloudflare dev proxy removed for `@cloudflare/vite-plugin`, RSC still unstable (2026-06-17; accessed 2026-11-22)
- https://reactrouter.com/changelog — v8.0.1 removed the `AppLoadContext` type (server context is a `RouterContextProvider`); latest v8.5.2 (2026-11-18); RSC APIs still `unstable_` (accessed 2026-11-22)
- https://reactrouter.com/8.3.1/api/other-api/adapter.md — official Cloudflare entry: `createContext` + `RouterContextProvider` passed to `createRequestHandler` (accessed 2026-11-22)
- https://developers.cloudflare.com/workers/framework-guides/web-apps/react-router/ — React Router on Workers with `@cloudflare/vite-plugin` (accessed 2026-11-22)
