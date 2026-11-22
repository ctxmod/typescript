# ctxmod/typescript

A [ctxmod](https://github.com/ludvigalden/ctxmod) context module for
contract-first TypeScript and JavaScript (ECMAScript) applications: the
Biome toolchain, deep modules and enforced boundaries, generated clients
and drift gates, problem details, localization, and testing.

Module path: `github.com/ctxmod/typescript/context` (requires
`github.com/ctxmod/engineering/context` and `github.com/ctxmod/design/context`).

## Profiles

| Profile | Selected | Covers |
| --- | --- | --- |
| `core` | by default | Architecture overview and glossary, toolchain, modules, contracts, frontend authority, localization, testing |
| `nextjs` | opt-in | Next.js App Router |
| `react_router` | opt-in | React Router v7: components, routes, i18n, testing, Cloudflare Workers deployment |
| `tanstack_start` | opt-in | TanStack Start: positioning, routing, rendering modes, Vite configuration |
| `shadcn` | opt-in | shadcn/ui components, design tokens, Ladle stories |
| `tailwind` | opt-in | Tailwind CSS v4 |

Every opt-in profile extends `core`. Select one by naming it, for example
`ctxmod render --profile github.com/ctxmod/typescript/context/profiles/nextjs`,
or list it in a consuming module's profile `extends`. Browser automation and
scraping context lives in the browser module; select it alongside when needed.
