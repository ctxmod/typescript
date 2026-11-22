# Routing

File-based routing and route modules as full-stack boundaries — the generated route tree, loaders and server functions as the server-side data seam, route-granular error boundaries rendering the es error contract, and where the shape differs from the react-router sibling.

Routing is file-based: the route tree is generated from route file
conventions, and each route module is a full-stack boundary owning its
route's server-side data, mutations, rendering, and error surface. This
reference owns what is TanStack Start-specific about that shape — the
generated route tree, the loader and server-function seams, and the error
boundary placement. The data and authority rules those seams carry are
the TypeScript core's, linked throughout and never restated here.

## The Generated Route Tree

The route tree is derived from the routes directory's file conventions:
file and directory names express URL segments, nesting, and route
parameters, and the framework generates a typed route tree from them.
The generated tree is a generated artifact in the full sense owned by
the generated-code discipline reference
([Generated-Code Discipline](../../../../core/packages/contracts/reference/generated_code_discipline.md)) —
reproducible, non-authoritative, never hand-edited.

One placement consequence is specific to this framework: the route tree
is generated as an importable module that application source imports for
route links and type information. A repository whose typecheck tier
sweeps application sources therefore commits the generated tree — a
fresh clone must resolve those imports without a generation step — and
carries the regeneration ritual with it, under the committed-versus-
regenerate postures the discipline reference owns.

Handwritten route shape lives in the file conventions themselves.
Renaming a route file is renaming a URL; it is done with the same
deliberation as any other consumer-facing identifier change, never as an
incidental tidy-up.

## Route Modules as Full-Stack Boundaries

A route module co-locates the route's concerns, each with a defined
execution side:

| Concern | Execution side | Responsibility |
|---|---|---|
| Loader | Server | Data loading for the route before rendering |
| Server functions | Server | Explicit server-only operations the client can invoke outside navigation |
| Route component | Server + client | Rendering, consuming typed loader data |
| Error component | Server + client | Route-granular failure rendering |
| Head and meta | Server | Document metadata for the route |

**Loaders read; server functions act.** A loader assembles what the
route renders and belongs to the read path. A server function is the
mutation seam for interactions that are not navigations — it is the
server-side entry a client action calls. Both consume the generated
typed client from the bundled contract
([Generated Clients](../../../../core/packages/contracts/reference/generated_clients.md)); a
hand-written server-side fetch wrapper is prohibited on the same terms
as a hand-written browser one, and a mutation that matters is a Command
against the API contract, not a free-form server function.

## What Carries Over from es Core

- The browser remains a consumer and Command issuer. Server-rendered
  output is still a read of backend truth: a render path that
  reconstructs Workflow State from anything other than the API or the
  read model violates
  [Frontend Authority](../../../../core/packages/frontend/reference/frontend_authority.md) exactly as
  a client-only path would. SSR moves the read server-side; it does not
  create a second authority.
- Live data follows
  [Live Read Model](../../../../core/packages/frontend/reference/live_read_model.md): the initial
  query loads the authoritative read model, subscription updates carry
  their `projection_revision` and stale ones are discarded, and
  subscriptions are torn down when the view unmounts — in whichever
  component owns the live view, not in the route module.
- Route modules stay thin coordination layers. Presentational and
  feature logic is pushed into colocated component files, exactly as the
  react-router sibling's colocation principle holds.

## Comparison to the react-router Sibling

The sibling reference
(`react-router/reference/principles/development/react/route-module-design.md`)
owns the route-module concept for the router-lens framework. Read it for
the shared discipline; the differences that matter here:

| Dimension | React Router sibling | TanStack Start |
|---|---|---|
| Route tree origin | Configured in a routes module | Generated from file conventions |
| Data seam exports | `loader` / `action` per route | Loaders plus callable server functions |
| Mutations outside navigation | Fetcher-driven actions | Server functions |

Neither shape is the more correct one; they are two bindings of the
same boundary discipline. Content that is not in the difference table
lives in the sibling or in the TypeScript core, not here.

## Error Boundaries

Export an error component per route segment that can fail distinctly,
with a root-level boundary as the final catch-all. What an error
boundary renders is owned by the es error contract
([Problem Details](../../../../core/packages/frontend/reference/problem_details.md)): the typed
problem-details value the generated client produced, the three failure
kinds kept distinguishable, the typed stale-`workflow_revision` conflict
surfaced as a conflict with refetch and re-presentation rather than a
generic failure. A route-granular boundary localizes where a failure
renders; it never invents its own error taxonomy.

## Thresholds

| Metric | Pass | Warn | Fail |
|--------|------|------|------|
| Route module size (lines) | <150 | 150-250 | >250 |
| Business logic in route modules | None | Trivial helpers | Domain rules |
| Hand-written server-side fetch wrappers | 0 | 0 | >0 |
| Routes without an error boundary that can fail distinctly | 0 | 1-2 | >2 |

## Where the Rest Lives

- The **frontend-authority** and **live-read-model** references own the
  read path and the prohibition on reconstructing Workflow State.
- The **generated-clients** reference owns the surface loaders and
  server functions consume.
- The **problem-details** reference owns the error contract error
  components render.
- Rendering-mode selection per route is the rendering-modes reference's
  (`rendering-modes.md`); Vite composition is the vite-config
  reference's (`vite-config.md`).

## Related

- the react_router profile's *Route Module Design* reference (`profiles/react_router/packages/routes/reference/route_module_design.md`)
