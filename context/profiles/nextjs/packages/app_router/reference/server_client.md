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

**Server Actions require `"use server"`**

`"use server"` marks individual functions as Server Actions. It does NOT prevent the surrounding module from being included in the client bundle — use `"import 'server-only'"` for that.

**Server Actions for mutations only**

Use Server Actions ("use server") only for mutations (POST/PUT/DELETE). For data fetching (GET), use `fetch()` with caching or `"use cache"`. Server Actions lose the benefits of request deduplication, caching, streaming, and prefetching that fetch provides.

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
