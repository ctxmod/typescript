# Server and Client Components

Server and Client Component boundaries and usage patterns.

## Server Components

**Default to Server Components**

Server Components reduce client-side JavaScript, enable direct database/secret access, and improve FCP. Add `"use client"` only when necessary — it pulls the entire subtree into the client bundle.

**Push `"use client"` to leaf components**

Place `"use client"` on the smallest possible component (buttons, forms, interactive widgets). Keep parent components as Server Components. When a file is marked `"use client"`, all its imports and directly rendered components become part of the client bundle.

**Use children/props composition for interleaving**

Pass Server Components as `children` or props to Client Components rather than importing them inside Client Component files. Server Components passed as `children` props are rendered on the server and streamed as rendered output — they are NOT included in the client module graph.

**Use `import 'server-only'` for sensitive server logic**

Add `"import 'server-only'"` to files containing database queries, API calls with secrets, or internal business logic. This creates a build-time error if the module is accidentally imported into a client component, preventing server code from leaking.

## Client Components

**Server Functions require `"use server"`**

`"use server"` marks individual functions as Server Functions (called Server Actions when used for form and mutation actions). It does NOT prevent the surrounding module from being included in the client bundle — use `"import 'server-only'"` for that.

**Server Actions for mutations only**

Use Server Actions ("use server") only for mutations (POST/PUT/DELETE). For data fetching (GET), use `fetch()` with caching or `"use cache"`. Server Actions lose the benefits of request deduplication, caching, streaming, and prefetching that fetch provides.

## React 19.x Surface in Server Components

Next.js 16.4 ships React 19.3. The parts that change how boundaries are drawn:

- **Context renders directly from a Server Component** (React 19.3) when the
  context object is imported from a `"use client"` module —
  `<UserContext value={user}>{children}</UserContext>` in a server layout,
  no wrapper provider component needed.
- **`use(browser())`** (from `react-dom`, 19.3) suspends a component on the
  server and renders it after hydration — the explicit opt-out for a
  browser-only leaf, instead of a `"use client"` + `useEffect` mount gate.
- **Mutations use Actions.** Client forms bind Server Functions through
  `<form action>`, `useActionState`, `useFormStatus`, and `useOptimistic`;
  invalidation after the write follows
  [Cache Components](cache_components.md) (`updateTag` for read-your-writes).
- **The React Compiler is stable** (1.0) and enabled per app with
  `reactCompiler: true`. In new code rely on it instead of hand-written
  `useMemo`/`useCallback`, which remain an escape hatch (for example, a
  stable effect dependency); leave existing memoization in place unless
  tests cover its removal.

## Implementation Patterns

### Composition with Children

```typescript
// components/InteractiveWrapper.tsx (Client Component)
export function InteractiveWrapper({ children }: { children: React.ReactNode }) {
  const [count, setCount] = useState(0);
  return (
    <div>
      <button onClick={() => setCount(c => c + 1)}>Count: {count}</button>
      {children} {/* Server Components work here */}
    </div>
  );
}

// app/dashboard/page.tsx (Server Component)
export default function DashboardPage() {
  return <InteractiveWrapper>Server Content</InteractiveWrapper>;
}
```

The Server Component is passed as `children`, so it's not included in the client bundle. This pattern lets you nest server-rendered content inside interactive client wrappers.

### Server-Only Modules

```typescript
// lib/db.ts (Server-only)
import 'server-only';

export async function queryDatabase() {
  // Database queries and secrets live here
  return await db.query('SELECT * FROM users');
}
```

If any file in the call chain imports this module into a client component, the build fails with an error — preventing runtime security issues.

## Thresholds

| Metric | Pass | Warn | Fail |
|--------|------|------|------|
| `"use client"` at page level | 0 | 1 | >1 |
| `"use client"` in layout level | 0 | 0 | >0 |
| `server-only` missing on DB modules | 0 | 0 | >0 |

- KISS (engineering module, `principles` profile: `design/reference/kiss.md`)

## Sources

- https://react.dev/blog/2026/09/09/react-19-3 — `<Context>` in Server Components, `browser()`, stable `<ViewTransition>` and Fragment refs (2026-09-09; accessed 2026-11-22)
- https://react.dev/blog/2025/10/07/react-compiler-1 — React Compiler 1.0 stable; memoization guidance for new and existing code (2025-10-07; accessed 2026-11-22)
- https://nextjs.org/blog/next-16-4 — Next.js 16.4 ships React 19.3 (2026-10-06; accessed 2026-11-22)
- https://nextjs.org/blog/next-16 — `reactCompiler` stable, not enabled by default (2025-10-21; accessed 2026-11-22)
