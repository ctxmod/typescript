# TanStack Start Overview

Profile for TanStack Start full-stack React applications, extends the TypeScript core profile with file-based routing, rendering-mode, and Vite composition conventions.

Scope: TanStack Start full-stack React applications — file-based routing
with route modules as full-stack boundaries, per-route rendering modes
(server-rendered, data-only, client-only, and the PWA posture), and Vite
configuration composed from typed declarations through a workspace-local
foundation package.

It targets TanStack Start 1.x — stable and production-ready since
October 2026, built on Vite (Rsbuild is also supported) with Nitro for
hosting — whose server functions (`createServerFn`) and selective SSR
(`ssr: true | 'data-only' | false`) are the seams this profile's routing
and rendering-mode references discipline. React Server Components remain
an experimental opt-in there ([Framework Positioning](framework_positioning.md)).

The profile adds TanStack Start specificity on top of the TypeScript core
references; it never restates them. The core references this profile
specializes:

- [Contract-first architecture](../../../../core/packages/contracts/reference/contract_first_architecture.md)
- [Generated clients](../../../../core/packages/contracts/reference/generated_clients.md)
- [Generated-code discipline](../../../../core/packages/contracts/reference/generated_code_discipline.md)
- [OpenAPI consumption](../../../../core/packages/contracts/reference/openapi_consumption.md)
- [Problem details](../../../../core/packages/frontend/reference/problem_details.md)
- [Frontend authority](../../../../core/packages/frontend/reference/frontend_authority.md)
- [Live read model](../../../../core/packages/frontend/reference/live_read_model.md)
- [Locale strategies](../../../../core/packages/localization/reference/locale_strategies.md)

## Sources

- https://tanstack.com/blog/tanstack-start-1-0 — TanStack Start 1.0 stable (2026-10-13; accessed 2026-11-22)
- https://tanstack.com/start/latest/docs/framework/react/guide/selective-ssr — selective SSR `ssr` values (accessed 2026-11-22)
- https://tanstack.com/start/latest/docs/framework/react/guide/server-components — experimental Server Components (accessed 2026-11-22)
