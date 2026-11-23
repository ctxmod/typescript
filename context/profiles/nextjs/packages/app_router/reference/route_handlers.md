# Route Handlers

Route Handler conventions, params handling, and caching for Next.js 16.

## HTTP Method Handling

**Export named functions per HTTP verb**

Route handlers export named functions that map to HTTP methods: `GET`, `POST`, `PUT`, `DELETE`, `PATCH`, `HEAD`, `OPTIONS`. Unsupported methods automatically return `405 Method Not Allowed`.

```typescript
export async function GET(_req: NextRequest) {
  return Response.json({ message: 'Hello' });
}

export async function POST(_req: NextRequest) {
  return Response.json({ created: true });
}
```

## Typed Route Parameters

**Use RouteContext for typed params**

`params` is a Promise that must be awaited (synchronous access was removed in Next.js 16). Use the `RouteContext<'/users/[id]'>` type helper for typed access:

```typescript
export async function GET(_req: NextRequest, ctx: RouteContext<'/users/[id]'>) {
  const { id } = await ctx.params;
  const user = await db.users.findUnique({ where: { id } });
  return Response.json(user);
}
```

Types are generated during dev, build, or typegen.

## Body Parsing

**Manual parsing with Web Request API**

Route handlers use the Web Request API — body must be explicitly read and parsed. No automatic body parsing like the old Pages Router API routes:

```typescript
export async function POST(req: NextRequest) {
  const body = await req.json(); // JSON
  const formData = await req.formData(); // Form data
  return Response.json({ received: true });
}
```

## Caching

**Non-GET handlers are never cached**

Route handlers (POST, PUT, DELETE, etc.) run at request time by default. Under Cache Components ([Cache Components](cache_components.md)), GET handlers are dynamic by default too; opt into caching with `"use cache"` on helper functions:

```typescript
export async function GET(_req: NextRequest) {
  // NOT cached by default
}

async function getCachedUsers() {
  'use cache';
  return await db.users.findMany();
}

export async function GET(_req: NextRequest) {
  return Response.json(await getCachedUsers()); // Cached
}
```

**`"use cache"` cannot appear directly in exports**

The directive must be at the top of a function or file. Route Handler exports (GET, POST, etc.) are special — wrap cached data fetching in a separate async function.

**Caching in Route Handlers requires helpers**

```typescript
export async function GET(_req: NextRequest) {
  // ❌ WRONG: directive at file level, not in function
  'use cache';
  return Response.json(await getUsers());
}

async function getUsers() {
  'use cache'; // ✅ CORRECT: inside helper function
  return await db.users.findMany();
}

export async function GET(_req: NextRequest) {
  return Response.json(await getUsers());
}
```

## Thresholds

| Metric | Pass | Warn | Fail |
|--------|------|------|------|
| `params` awaited in route handlers | 100% | 0% | 0% |
| Body parsing manual in route handlers | 100% | 0% | 0% |
| `"use cache"` in route handler export | 0% | 0% | >0% |

## Sources

- https://nextjs.org/blog/next-16 — async `params`, Cache Components dynamic-by-default model (2025-10-21; accessed 2026-11-22)
- https://nextjs.org/docs/app/api-reference/file-conventions/route — route handler conventions, `RouteContext` (accessed 2026-11-22)
