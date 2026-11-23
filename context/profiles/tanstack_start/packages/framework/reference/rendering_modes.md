# Rendering Modes

Rendering-mode discipline — the per-route mode axis with monotonic narrowing, the application profile as one typed declaration feeding every consumer, the user-non-specific render family (URL-only cacheability, rendering-off for protected routes, post-hydration chrome), byte-stable indexable renders, search and agent discoverability as sibling surfaces, and the PWA posture as a per-application decision.

Which side of the network renders a route is a declared, per-route
property — and the declaration is enforced, not hoped at. This reference
owns the rendering-mode axis, the application profile that records it,
and the cache-safety and honesty rules that follow from it. The
authority rules underneath are the TypeScript core's.

## The Mode Axis

A route's rendering mode sits on one ordered axis, from most to least
server involvement:

| Mode | What the server does | What ships |
|---|---|---|
| Server-rendered | Executes loaders and renders the document | Full HTML plus the client bundle |
| Data-only | Serves data, not documents | Client bundle; rendering in the browser |
| Client-only | Nothing route-specific | Client bundle alone |

The axis is the framework's selective-SSR `ssr` route option:
server-rendered is `ssr: true` (the default), data-only is
`ssr: 'data-only'` (loaders and `beforeLoad` run on the server, the
component renders in the browser), and client-only is `ssr: false`. The
application-wide default is `defaultSsr` in the `createStart` options
(`src/start.ts`); SPA mode turns server execution off for the whole
application. A functional `ssr` runs server-side on the initial request
only and may narrow per request from validated params and search.

**Narrowing is monotonic.** A child route may narrow its ancestors'
mode (server-rendered → data-only → client-only); it may not widen one.
Widening silently is the failure this rule exists for: upstream
tooling tends to ignore an inconsistent widening at build time and
produce the narrower output anyway, so the narrowing contract is
enforced by a build-time check that makes the attempt a hard error
naming the offending route and its ancestor — never a silent fallback.

## The Application Profile Is One Declaration

An application declares its posture as one typed record — the
application profile (what the app is), the deployment profile (where it
runs, carrying the server runtime surface as an expression of what the
app's own server code may use), and its runtime targets (which runtimes
the build must serve). The declaration is validated at construction
(unknown values and empty target lists fail with precise errors),
frozen, and kept serializable so every consumer — the Vite
configuration, CI checks, deploy wrappers — reads the same record
mechanically.

Three rules bind the declaration:

- **Expression, never narrowing.** Deployment metadata describes what
  the application's own code may use; it never narrows a library's
  contracts.
- **One declaration, many consumers, one gate.** The record is compiled
  into option fragments per consumption surface, and because different
  files consume different fragments, an agreement gate recomputes the
  expectation from the declaration and fails the build on any
  disagreement between the fragments.
- **Both halves set together.** Where a mechanism has a runtime flag
  and a build flag, the declaration sets both, explicitly on rather
  than default-inherited, with the rationale recorded — a half-set
  mechanism is the silent divergence the gate exists to catch.

The specific profile values an application picks are design decisions
recorded with the application, not laws of this profile.

## The User-Non-Specific Render Family

Three cooperating rules keep shared caches safe and shared renders
user-free:

1. **Cached server-rendered output varies by URL only.** Never by
   cookie, header, or client identity. Cache keys are canonical,
   legible strings — normalized URL, sorted parameters, sorted content
   keys — so cache logs and debug headers stay readable rather than
   opaque. Invalidation goes through a selected interface (an
   origin-owned purge or a cache-provider purge API); the application
   owns which, wired so the wrong surface is unrepresentable.
2. **Protected routes render nothing on the server.** A route serving
   user-specific protected content runs with effective rendering off.
   Both alternatives are violations, with distinct recorded rationales:
   full server rendering produces user-dependent output a shared cache
   would serve to someone else; data-only still executes loaders
   server-side per request. A build-time check over the route tree
   enforces the rule with those rationales in its messages, paired with
   the indexability declaration in both directions — a protected route
   that is also declared indexable fails the build, and so does an
   indexable route marked protected.
3. **Auth-conditional chrome mounts after hydration.** User-specific
   navigation, account surfaces, and personalized chrome render only
   once the client has hydrated, through a named mounting strategy —
   the structural difference between the served shell and the hydrated
   tree is the design, and the strategy's name appears wherever the
   mechanism appears.

The wider honesty rules — a render path never reconstructs Workflow
State, and live data follows the read model with stale
`projection_revision` updates discarded — are the TypeScript core's:
[Frontend Authority](../../../../core/packages/frontend/reference/frontend_authority.md) and
[Live Read Model](../../../../core/packages/frontend/reference/live_read_model.md). Rendering
mode changes where those rules execute, never whether they bind.

## Byte-Stable Indexable Renders

A route declared indexable returns the same bytes to the prerenderer,
the crawler, and every user — no per-header or per-identity variants.
Request-varying content belongs on a non-indexed route. The rule is
enforced by a harness that fetches the same URL under several request
shapes — plain, with session credentials, with a language preference,
with forwarding hints, under a crawler user agent — and asserts
identity. Where the framework embeds an unavoidable volatile field in
the response, the harness documents the field's span and asserts the
stronger property that every differing byte lies inside it, with
planted negatives proving the comparator catches outside-span variance.

## Discoverability Surfaces: Siblings, Not Nested

Search-engine discoverability and agent discoverability are two
audiences over one substrate, kept as sibling surfaces: the
indexability surface answers crawlers; agent-facing artifacts —
structured data in the route head, plain-language overview routes,
machine-readable listing endpoints — answer agents that read. A route
can carry agent-facing artifacts without being indexable, and the
indexability declaration says nothing about them.

Each artifact is authored as a pure renderer — validatable, freezable,
testable — plus a thin factory that wraps it into the framework's
server-route options shape. The renderer owns the content; the factory
owns the wiring; neither imports the other's concerns.

## The PWA Posture

Whether an application installs service-worker behavior is an
application-profile decision. Where it does, the mechanism is drawn
from the service-worker and asset-precaching library family — this
profile mandates neither the family nor any member of it — and the
selection is recorded with the application profile.

One consequence of a client-only posture generalizes: the build that a
PWA serves is one static client output, and where the application also
ships a native wrapper runtime, that same build is staged into the
wrapper's served directory. One build, several runtimes, named
distinctly in the application's vocabulary — the installed form and the
browser form are the same build in different runtimes, and "installed"
never implies a different artifact.

## Where the Rest Lives

- The **routing reference** owns route modules and the loader and
  server-function seams this axis renders.
- The **vite-config reference** owns the composition machinery that
  turns the application profile into build configuration.
- The **generated-code-discipline** reference owns the properties of
  anything the mode pipeline generates.

## Related

- [Generated-Code Discipline](../../../../core/packages/contracts/reference/generated_code_discipline.md)
- [Routing](routing.md)
- [Vite Configuration](vite_config.md)

## Sources

- https://tanstack.com/start/latest/docs/framework/react/guide/selective-ssr — `ssr: true | 'data-only' | false`, children only narrow, `defaultSsr` in `createStart`, functional `ssr`, SPA mode (accessed 2026-11-22)
