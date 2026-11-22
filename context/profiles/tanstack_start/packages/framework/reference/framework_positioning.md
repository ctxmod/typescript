# Framework Positioning

Where TanStack Start sits as a deliberate framework choice — type-safe routing and server functions, the throughput profile, explicit-control-over-magic, and the framing for not-yet-supported ecosystem features.

Choosing this framework is a positioning decision worth stating in the
repository's own terms: what it buys, what it costs, and which ecosystem gaps
are consciously accepted. The framing matters — a gap documented as a
deliberate trade reads as judgment; a gap discovered later reads as an
oversight.

## What the Choice Buys

- **Type-safe routing.** The route tree is generated and typed end to end:
  links, parameters, and search params are type-checked, and a route rename
  is a compile error at every call site rather than a runtime 404. This
  composes with the routing reference's full-stack-boundary model.
- **Type-safe server functions.** The server-side operations a client invokes
  are typed from the server definition to the client call site with no
  hand-written client layer between — the same generated-binding posture the
  es tier applies to wire contracts, applied to the framework's own RPC seam.
- **Throughput profile.** A lean core with less per-request framework
  machinery delivers competitive throughput and latency characteristics;
  performance comes from doing less implicitly rather than from tuning more.
- **Explicit control over magic.** Configuration composes in typed code
  rather than accumulating in implicit conventions. Fewer framework behaviors
  activate unasked; the ones that do are findable in the project's own code.
  This is a philosophy match, not a feature list: it suits repositories that
  treat the framework as a composed dependency rather than an environment.

## The RSC Gap, Framed

TanStack Start 1.0 is stable (October 2026), and its React Server
Component support is an opt-in integration that is still experimental —
enabled per application (`rsc: { enabled: true }` in the `tanstackStart()`
plugin options, plus `@vitejs/plugin-rsc` on Vite 8+ or Rsbuild 2+) and
deliberately not the framework's core rendering model. Frameworks that
center on RSC position it as their core. Not adopting it is deliberate
when the application's rendering needs are met by the stable rendering
modes — server rendering, data-only routes, and client-only routes cover
the common shapes without RSC's streaming component model. Adopting the
experimental integration is likewise a recorded decision, scoped to the
routes that need it, with its stabilization as the revisit trigger.

The framing to carry in the repository: state what is deferred, why the
supported modes suffice for the current product shape, and the integration
path when support lands (component boundaries and data seams placed so the
RSC migration is incremental, not a rewrite). Revisit at framework
milestones, not per release candidate; a positioning note that churns with
every minor version is noise.

## Decision Record Shape

When a repository commits to the framework, its positioning note names: the
three buys above (or the ones that actually drove the decision), the
accepted gaps with their rationale, and the revisit trigger. A future
maintainer should be able to re-run the decision from the note without
archaeology.

## Related

- [Routing](routing.md)
- [Rendering Modes](rendering_modes.md)

## Sources

- https://tanstack.com/blog/tanstack-start-1-0 — TanStack Start 1.0 stable and production-ready; `createServerFn`; Vite and Nitro for hosting, Rsbuild support (2026-10-13; accessed 2026-11-22)
- https://tanstack.com/start/latest/docs/framework/react/guide/server-components — Server Components experimental "into early v1": `rsc: { enabled: true }`, `@vitejs/plugin-rsc`, React 19+, Vite 8+ or Rsbuild 2+ (accessed 2026-11-22)
