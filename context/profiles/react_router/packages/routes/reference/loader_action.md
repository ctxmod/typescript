# Loader and Action Implementation

Workflow for implementing loaders and actions with parallel fetching, Zod validation, and mutation patterns.

Procedure for implementing data loading and mutation handling in React Router v7 route modules. Loaders handle GET requests with server-side data fetching. Actions handle POST mutations with automatic revalidation. Both run exclusively on the server, keeping secrets and database access out of client bundles.

## Loader Implementation

### 1. Define the Loader Signature

Use generated types for end-to-end type safety:

```typescript
import type { Route } from "./+types/route";

export async function loader({ params, request, context }: Route.LoaderArgs) {
  // Server-only: DB access, secrets, env variables
}
```

The return type flows directly to the component via `Route.ComponentProps`. No manual type synchronization required.

### 2. Fetch Data in Parallel

Identify independent data sources and batch them with `Promise.all()`. Sequential fetching creates waterfalls that multiply latency.

```typescript
export async function loader({ params, context }: Route.LoaderArgs) {
  const { env } = context.cloudflare;

  const [user, products] = await Promise.all([
    db.user.findUnique({ where: { id: params.userId } }),
    db.product.findMany({ where: { userId: params.userId } }),
  ]);

  if (!user) {
    throw data("Not Found", { status: 404 });
  }

  return { user, products };
}
```

Prefer single queries with joins over multiple round-trips when the database supports it. Each additional round-trip adds network latency.

### 3. Stream Slow Data

When secondary data takes >100ms, return promises for streaming instead of awaiting them:

```typescript
export async function loader({ params }: Route.LoaderArgs) {
  const product = await db.product.findUnique({ where: { id: params.pid } });

  if (!product) {
    throw data("Not Found", { status: 404 });
  }

  // Slow data streams progressively
  const reviewsPromise = db.review.findMany({ where: { productId: params.pid } });

  return { product, reviewsPromise };
}
```

The component renders critical data immediately and wraps streamed data in Suspense:

```tsx
export default function Product({ loaderData }: Route.ComponentProps) {
  return (
    <div>
      <ProductInfo product={loaderData.product} />
      <Suspense fallback={<ReviewsSkeleton />}>
        <Await resolve={loaderData.reviewsPromise}>
          {(reviews) => <ReviewsList reviews={reviews} />}
        </Await>
      </Suspense>
    </div>
  );
}
```

### 4. Handle Missing Resources

Throw typed responses for expected missing data. The nearest `ErrorBoundary` receives the thrown response.

```typescript
if (!product) {
  throw data("Not Found", { status: 404 });
}
```

Use `throw redirect("/login")` for authentication failures. Redirects skip error boundaries entirely.

### 5. Access Cloudflare Bindings

Access KV, R2, D1, and other Cloudflare services through the context parameter:

```typescript
export async function loader({ context }: Route.LoaderArgs) {
  const { env } = context.cloudflare;
  const cached = await env.MY_KV.get("cache-key");
  const dbResult = await env.MY_DB.prepare("SELECT * FROM items").all();
  return { cached, items: dbResult.results };
}
```

## Action Implementation

### 1. Define the Action

Actions handle POST, PUT, PATCH, and DELETE requests. After completion, React Router automatically revalidates all loader data on the page.

```typescript
export async function action({ request }: Route.ActionArgs) {
  const formData = await request.formData();
  const intent = formData.get("intent");

  switch (intent) {
    case "create":
      return handleCreate(formData);
    case "update":
      return handleUpdate(formData);
    case "delete":
      return handleDelete(formData);
    default:
      throw data("Unknown intent", { status: 400 });
  }
}
```

Use the `intent` field pattern to multiplex multiple mutation types through a single action.

### 2. Validate Input with Zod

Validate all form data server-side using Zod schemas with allowlists and length bounds:

```typescript
import { z } from "zod";

const CreateProductSchema = z.object({
  name: z.string().min(1).max(100),
  price: z.coerce.number().positive(),
  description: z.string().max(1000).optional(),
});

async function handleCreate(formData: FormData) {
  const raw = Object.fromEntries(formData);
  const result = CreateProductSchema.safeParse(raw);

  if (!result.success) {
    return data({ errors: result.error.flatten() }, { status: 422 });
  }

  await db.product.create({ data: result.data });
  return redirect("/products");
}
```

Return validation errors as data with a 422 status. The component reads errors from `actionData` to display inline feedback. Never rely on client-side validation alone.

### 3. Choose Form Submission Pattern

Two patterns serve different navigation needs:

| Pattern | Component | Behavior | Use When |
|---------|-----------|----------|----------|
| `<Form>` | Navigation form | Creates history entry, shows loading state | Page-level mutations (create, edit) |
| `useFetcher` | Non-navigation | No history entry, inline loading | Inline actions (delete, toggle, like) |

```tsx
// Navigation: creates history entry
<Form method="post">
  <input name="name" />
  <button type="submit" name="intent" value="create">Create</button>
</Form>

// Non-navigation: inline mutation
const fetcher = useFetcher();
<fetcher.Form method="post">
  <input type="hidden" name="id" value={item.id} />
  <button type="submit" name="intent" value="delete">Delete</button>
</fetcher.Form>
```

## Client Loader and Client Action

Use `clientLoader` and `clientAction` for browser-only operations (IndexedDB, client SDKs, Web APIs):

```typescript
export async function clientLoader({ params }: Route.ClientLoaderArgs) {
  const cache = await localforage.getItem(`product-${params.pid}`);
  return cache;
}

clientLoader.hydrate = true; // Force execution during SSR hydration

export function HydrateFallback() {
  return <LoadingSpinner />;
}
```

When both `clientAction` and `action` exist, `clientAction` takes priority. `HydrateFallback` renders only during initial page load hydration, not during client navigations.

## Loader Performance Thresholds

| Metric | Pass | Warn | Fail |
|--------|------|------|------|
| Loader execution time | <30ms | 30-100ms | >100ms |
| Independent DB queries per loader | 1 (with joins) | 2-3 (parallel) | >3 or sequential |
| Network round-trips | 1 | 2 | >2 |

## Validation Checklist

- [ ] Independent data sources use `Promise.all()`
- [ ] Missing resources throw `data()` with appropriate status code
- [ ] All action inputs validated server-side with Zod
- [ ] Validation errors returned as data, not thrown
- [ ] `Form` used for navigation mutations, `useFetcher` for inline
- [ ] Secrets accessed only in loader/action (server-only)
- [ ] State-changing operations use POST actions, never loaders
- [ ] Generated `Route.*` types used throughout

- [Route Error Handling](error_handling.md)

## Related

- [Route Module Design](route_module_design.md)
- [Route Planning](route_planning.md)
