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

**`params` is a Promise in Next.js 15+**

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

If a page reads `searchParams`, it cannot be statically prerendered at build time. For content pages that should be static, move search/filter functionality to a separate dynamic route.

## Thresholds

| Metric | Pass | Warn | Fail |
|--------|------|------|------|
| Module-level singletons used | 0% | 0% | >0% |
| `params` accessed without awaiting | 0% | 0% | >0% |
| Native modules imported in Edge bundle | 0% | 0% | >0% |
