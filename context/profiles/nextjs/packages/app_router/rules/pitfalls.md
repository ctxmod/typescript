# Next.js Pitfalls Context

Common Next.js pitfalls including module-instance isolation, native modules in edge, and params gotchas.

Module-instance isolation in `output: "standalone"` builds means the same source file resolves to different module instances across entry points. Module-level singletons do not survive — use `globalThis` or a registry pattern. `params` is a Promise and must be awaited in all dynamic routes (sync access was removed in Next.js 16). Native modules in Edge bundles cause cryptic `node:crypto` errors — guard with `export const runtime = "nodejs"` or dynamic import behind runtime checks. `searchParams` access forces dynamic rendering.

- [Next.js Pitfalls](../reference/pitfalls.md)
