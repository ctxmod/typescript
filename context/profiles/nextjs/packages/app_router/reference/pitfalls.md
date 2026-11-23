# Next.js Pitfalls

Common Next.js pitfalls and anti-patterns.

## Module-Instance Isolation

**Same file resolves to different module instances in standalone builds**

In `next build` with `output: "standalone"`, the same source file can resolve to different module instances across entry points (instrumentation vs route handlers). Module-level singletons do not survive across these instances.

**Symptom**

```typescript
// lib/db.ts (module-level)
let db: Database | null = null;

export function getDb() {
  if (!db) db = new Database();
  return db;
}

// instrumentation.ts
const db = getDb(); // First instance

// app/users/route.ts
export async function GET() {
  return Response.json(await getDb().users.findMany()); // Second instance!
}
```

**Fix: Use globalThis registry pattern**

```typescript
// lib/db.ts
const dbKey = Symbol('db');

export function getDb() {
  if (!globalThis[dbKey]) {
    globalThis[dbKey] = new Database();
  }
  return globalThis[dbKey];
}
```

## Params Promise

**`params` is a Promise (sync access removed in Next.js 16)**

```typescript
// ❌ WRONG
export async function GET(_req: NextRequest, ctx: RouteContext<'/users/[id]'>) {
  const { id } = ctx.params; // ctx.params is a Promise!
  return Response.json(await db.users.findUnique({ where: { id } }));
}

// ✅ CORRECT
export async function GET(_req: NextRequest, ctx: RouteContext<'/users/[id]'>) {
  const { id } = await ctx.params;
  return Response.json(await db.users.findUnique({ where: { id } }));
}
```

This affects all dynamic route segments in pages, layouts, and route handlers.

## Native Modules in Edge Bundles

**Importing native modules in Edge runtime causes `node:crypto` errors**

Native C++ addons cannot run in the Edge Runtime. Importing them in a file that gets bundled for Edge causes cryptic `node:crypto` errors that are hard to trace.

Route segments already default to the Node.js runtime, and Next.js 16's `proxy.ts` runs only on Node.js, so this pitfall applies to code explicitly opted into Edge (`export const runtime = "edge"`, or the deprecated `middleware.ts`).

**Fix: Guard with runtime or use dynamic import**

```typescript
// Option 1: Runtime check with export
export const runtime = "nodejs";
import 'better-sqlite3';

// Option 2: Dynamic import behind runtime check
if (process.env.NEXT_RUNTIME === 'nodejs') {
  const { Database } = await import('better-sqlite3');
  const db = new Database();
}
```

**Fix: Guard instrumentation imports**

```typescript
// instrumentation.ts
if (process.env.NEXT_RUNTIME === 'nodejs') {
  const instrumentationNode = await import('./instrumentation-node');
  instrumentationNode.register();
}
```

## Search Params Forces Dynamic Rendering

**Accessing `searchParams` forces dynamic rendering**

If a page reads `searchParams`, it cannot be statically prerendered at build time. For content pages that should be static, move search/filter functionality to a separate dynamic route. Under Cache Components, read `searchParams` in a component behind `<Suspense>` so the rest of the page stays in the static shell, and guard routes that must stay static with `ensureStatic` ([Cache Components](cache_components.md)).

## Thresholds

| Metric | Pass | Warn | Fail |
|--------|------|------|------|
| Module-level singletons used | 0% | 0% | >0% |
| `params` accessed without awaiting | 0% | 0% | >0% |
| Native modules imported in Edge bundle | 0% | 0% | >0% |

## Sources

- https://nextjs.org/blog/next-16 — async `params`/`searchParams` mandatory, `proxy.ts` on Node.js, `middleware.ts` deprecated for Edge use (2025-10-21; accessed 2026-11-22)
- https://nextjs.org/blog/next-16-4 — `ensureStatic` (2026-10-06; accessed 2026-11-22)
