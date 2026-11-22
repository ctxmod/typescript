# Cloudflare Workers Deployment

Deployment workflow for React Router v7 SSR applications on Cloudflare Workers with streaming, security headers, and binding configuration.

Procedure for deploying React Router v7 SSR applications to Cloudflare Workers. This covers the Worker fetch handler, streaming SSR configuration, security headers, environment variables, Cloudflare binding integration, and deployment workflow.

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

The Worker entry point receives all incoming requests and delegates to React Router's request handler:

```typescript
// workers/app.ts
import { createRequestHandler } from "react-router";

const requestHandler = createRequestHandler(
  // @ts-expect-error virtual module provided by React Router
  () => import("virtual:react-router/server-build"),
  import.meta.env.MODE
);

export default {
  async fetch(request: Request, env: Env, ctx: ExecutionContext) {
    const response = await requestHandler(request, {
      cloudflare: { env, ctx },
    });

    return addSecurityHeaders(response);
  },
} satisfies ExportedHandler<Env>;
```

Pass `env` and `ctx` through the cloudflare context object. Loaders and actions access these via `context.cloudflare.env`.

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
  remixContext: EntryContext
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
    <ServerRouter context={remixContext} url={request.url} />,
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
  "compatibility_date": "2025-01-01",
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
  const { env } = context.cloudflare;
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

- [ ] `workers/app.ts` creates request handler with cloudflare context
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
