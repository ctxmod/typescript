# Route Error Handling

Workflow for implementing route-granular error boundaries, fallback UIs, and server-side error logging.

Procedure for implementing error boundaries in React Router framework-mode route modules. Error boundaries catch failures at the nearest route level, preventing full-page crashes and providing contextual error UIs. This procedure covers route-level boundaries, root fallbacks, error type discrimination, and server-side logging.

## Error Architecture

React Router propagates errors to the nearest route that exports an `ErrorBoundary`. If no ancestor route exports one, the root error boundary catches it. Each route should define its own boundary to keep error UIs contextual: a missing product shows "Product not found," not a generic application error.

### Error Flow

```
Route loader/action throws
  -> Nearest ErrorBoundary in route hierarchy
    -> Parent route ErrorBoundary (if current route lacks one)
      -> Root ErrorBoundary (last resort)
```

Errors from a route's `loader`, `action`, or `Component` all propagate to that route's `ErrorBoundary`. Layout routes can also export boundaries, catching errors from nested child routes while preserving the surrounding layout.

## Implementation Workflow

### 1. Export ErrorBoundary from Route Modules

Every route that fetches data or handles mutations should export an `ErrorBoundary`:

```typescript
import type { Route } from "./+types/route";

export function ErrorBoundary({ error }: Route.ErrorBoundaryProps) {
  if (error instanceof Response) {
    return <ResponseError response={error} />;
  }

  return <UnexpectedError error={error} />;
}
```

### 2. Distinguish Error Types

React Router delivers two error categories to boundaries. Handle each differently:

| Error Type | Source | Detection | Response |
|------------|--------|-----------|----------|
| Response errors | `throw data("msg", { status })` or `throw redirect()` | `error instanceof Response` | Status-specific UI |
| Runtime errors | Unhandled exceptions | Default case | Generic error with retry |

**Response errors** represent expected conditions (missing resource, unauthorized access). Display specific messages matching the status code:

```typescript
function ResponseError({ response }: { response: Response }) {
  switch (response.status) {
    case 404:
      return (
        <div>
          <h2>Not Found</h2>
          <p>The resource you requested does not exist.</p>
        </div>
      );
    case 403:
      return (
        <div>
          <h2>Access Denied</h2>
          <p>You do not have permission to view this content.</p>
        </div>
      );
    default:
      return (
        <div>
          <h2>Error {response.status}</h2>
          <p>Something went wrong. Please try again.</p>
        </div>
      );
  }
}
```

**Runtime errors** represent unexpected failures. Show a generic error message with a retry mechanism:

```typescript
function UnexpectedError({ error }: { error: Error }) {
  return (
    <div>
      <h2>Something went wrong</h2>
      <p>An unexpected error occurred. Please try again later.</p>
      {process.env.NODE_ENV === "development" && (
        <pre>{error.stack}</pre>
      )}
    </div>
  );
}
```

Display stack traces only in development. Production error boundaries show user-friendly messages without implementation details.

### 3. Implement Root Error Boundary

The root error boundary in `root.tsx` catches errors that escape all route boundaries. It must render a complete HTML document because the root layout may not be available:

```typescript
export function ErrorBoundary() {
  const error = useRouteError();

  return (
    <html lang="en">
      <head>
        <meta charSet="utf-8" />
        <meta name="viewport" content="width=device-width, initial-scale=1" />
        <title>Error</title>
      </head>
      <body>
        <h1>Application Error</h1>
        <p>We encountered an unexpected problem. Please refresh the page.</p>
      </body>
    </html>
  );
}
```

The root boundary renders without `<Links />`, `<Meta />`, or `<Scripts />` from the normal layout, since those components may have caused the error.

### 4. Throw Appropriate Error Responses

In loaders and actions, throw typed responses for expected error conditions:

```typescript
export async function loader({ params }: Route.LoaderArgs) {
  const product = await db.product.findUnique({ where: { id: params.pid } });

  if (!product) {
    throw data("Not Found", { status: 404 });
  }

  return product;
}
```

Map error conditions to HTTP status codes consistently:

| Condition | Status | Example |
|-----------|--------|---------|
| Resource missing | 404 | Product not in database |
| Unauthorized | 401 | No valid session |
| Forbidden | 403 | Insufficient permissions |
| Invalid input | 422 | Zod validation failure |
| Rate limited | 429 | Too many requests |
| Server failure | 500 | Unhandled exception |

For authentication failures, prefer `throw redirect("/login")` over error responses. Redirects skip error boundaries and navigate the user directly.

### 5. Implement Server-Side Error Logging

Log errors in the server entry point with contextual information:

```typescript
// entry.server.tsx
const body = await renderToReadableStream(
  <I18nextProvider i18n={i18n}>
    <ServerRouter context={routerContext} url={request.url} />
  </I18nextProvider>,
  {
    onError(error: unknown) {
      responseStatusCode = 500;
      console.error({
        url: request.url,
        isBot: isbot(request.headers.get("user-agent")),
        error: error instanceof Error ? error.message : String(error),
        stack: error instanceof Error ? error.stack : undefined,
      });
    },
  }
);
```

Include URL, user-agent classification (bot vs. human), error message, and stack trace. Structured logging enables filtering and aggregation in production monitoring.

### 6. Handle Bot Timeouts

Bots require complete HTML for indexing. Implement different timeout strategies:

```typescript
const controller = new AbortController();
const isBot = isBotRequest(request.headers.get("user-agent"));

const timeout = setTimeout(
  () => controller.abort(),
  isBot ? 15_000 : 5_000
);

if (isBot) {
  await body.allReady; // Wait for complete HTML
}
```

When a bot request times out, return 504 status. Incomplete HTML harms SEO worse than an explicit server error. For human requests, stream content progressively without waiting for completion.

## Error Boundary Placement Guide

| Route Type | Boundary Needed | Rationale |
|------------|----------------|-----------|
| Data-fetching route | Yes | Loader failures need contextual UI |
| Mutation route | Yes | Action failures need inline feedback |
| Layout route | Optional | Preserves layout when child errors |
| Static route | No | No failure modes to catch |
| Root route | Required | Last-resort catch-all |

## Validation Checklist

- [ ] Every data-fetching route exports `ErrorBoundary`
- [ ] Response errors distinguished from runtime errors
- [ ] Root boundary renders complete HTML document
- [ ] Missing resources throw `data()` with 404 status
- [ ] Auth failures use `redirect()`, not error responses
- [ ] Server errors logged with URL, user-agent, and stack
- [ ] Bot timeouts return 504 status
- [ ] Development mode shows stack traces; production hides them

## Related

- [Route Module Design](route_module_design.md)
- [Route Planning](route_planning.md)
- [Loader and Action Implementation](loader_action.md)
