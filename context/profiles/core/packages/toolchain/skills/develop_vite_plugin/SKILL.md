---
name: develop_vite_plugin
description: "Develop a Vite plugin composition-first — decide admission against configuration and upstream composition, structure the package as a preset factory the consumer spreads once, choose the narrowest hook channel, pin the architecture with ratification guard tests driven by a real fixture app, and wire it through the workspace. Use when: write a vite plugin; develop a vite plugin; new vite plugin; vite plugin factory; extend vite; build tool plugin; plugin or configuration."
---

# Develop a Vite Plugin

Build a Vite plugin in the order the composition-first discipline requires:
admission first, package structure second, hook choice third, guard tests
fourth, workspace wiring last.

## Purpose

Use when a build-time responsibility — an injection, an alias policy, a
layer-ordering contract, a preset that wires several engines together —
needs to exist as more than one application's configuration. The sequence
below operationalizes rules owned by the references linked at the end;
this skill sequences them and does not restate them.

Two failure modes are what this skill exists to prevent. The first is the
reimplementation: a plugin that re-derives what an upstream engine already
does, which forks the engine's behavior the moment upstream changes. The
second is the self-referential test: a plugin proven only against its own
mocked internals, which ratifies nothing about the build it will actually
participate in.

## Workflow

**1. Decide admission before writing code.**

| Situation | Action |
|---|---|
| An existing plugin reaches the effect through its options | Configure it; no new package |
| Several upstream plugins must be wired in a fixed order under project defaults | A composition preset that composes them, never re-derives them |
| No existing surface reaches the responsibility | A thin plugin owning exactly that responsibility, on the narrowest hook that does the job |
| The need is a one-off transform for one application | Keep it in that application's config; a package is justified only when a second consumer exists |

A plugin is justified when a responsibility cannot be expressed as
configuration — never because configuring felt verbose.

**2. Structure the package as a preset factory.** The public surface is a
factory returning a plugin array the consumer spreads into its plugin list
exactly once. Ordering contracts are documented as load-bearing: an engine
whose transform must see every stylesheet goes before the preset; a
post-ordered build hook goes last. Upstream engines are composed, never
imported at their internals. Sibling workspace packages the preset can use
declare as optional peers so it degrades gracefully without them, and
every peer is mirrored in devDependencies so the package's own tests run
without a host installing them. Framework types are imported type-only —
erased at compile time — so the config-time loading path never pulls the
engine. Node-only surfaces ride their own subpath exports so a browser
bundle never reaches them.

```
<repo>/
  packages/            # the plugin package: one directory, source in src/
  apps/
    <app>/             # a real consumer
                       # the fixture app lives here too — see step 4
```

One loading contract decides the shape: a package consumed from a config
file that Node loads cannot be served from TypeScript source through
Node's published-specifier resolution. Either register a resolver hook
that maps published specifiers onto workspace sources, or ship a compiled
config-time entry — and let the exports map carry a development condition
serving source so bundler-driven flows resolve source while Node resolves
compiled output. This posture is the package-development procedure's
compiled-versus-source decision applied to the config-time path.

**3. Choose the hook by channel, not by habit.** Enumerate the injection
channels the build pipeline actually offers, state which channel survives
which output posture, record why the rejected channel fails, and implement
exactly one. Where both apply, prefer module-graph injection over
document-transform injection: a stylesheet imported by a rendered module
is emitted as a build asset and linked from every document posture —
build-time prerendered and request-time rendered alike — while a
document-transform hook cannot reach a shell emitted after that phase.
Patch an emitted file on disk only when every document is produced at
build time, and say so in the decision record.

When inserting statements into an existing ordered chain, define a
canonical rank model over the statements already present, verify the
pairwise ordering rules, and fail the build loudly on a violation — with
the offending lines and the canonical order restated. Insert only the
missing entries, each after the last statement that canonically precedes
it, so the final top-down sequence stays canonical.

Two defaults govern contribution. Consumer wins: before contributing an
alias or default, check both configuration forms for a declaration the
consumer already made, merge after it, never clobber it — and match
specifiers exactly or by path suffix so package names never match. Fail
loudly on discovery: when an option is absent, resolution walks a
documented chain (explicit option, then the ecosystem's source-of-truth
file on disk, then the conventional default path, then conventional
fallbacks); if the feature is enabled and nothing is found, the build
fails listing the whole chain it tried and the two remedies. A silent
feature-less build is never produced.

```
factory(options) -> Plugin[]:
  resolved = walkDiscoveryChain(options)     # explicit → SSOT file → convention
  if resolved.enabled and resolved.empty:
    fail listing the chain and both remedies
  return [
    upstreamEngine(resolved.engine),          # composed verbatim
    thinInjector(resolved.tokens),            # rank-ordered, missing-only
    thinAlias(resolved.alias),                # consumer-wins merge
    optionalSibling(resolved.extra),          # optional peer
  ]
```

Hook signatures, ordering directives, and option shapes are Vite's own;
consult the Vite plugin API documentation for binding detail — this skill
never restates them.

**4. Test against a real consumer, not mocked internals.** Stand up a
fixture application inside the workspace that installs the plugin and
exercises it the way a real consumer does: build it, serve it, assert on
its emitted output. Where behavior varies by build variant, drive several
variant configurations over one fixture source rather than several
sources. Then pin the architecture with ratification guard tests, because
a mandate that is not tested is a mandate that erodes:

- A **no-reimplementation guard** source-scans the package for the
  telltale families of reimplementation — direct engine imports, engine
  internals, orchestration identifiers — and asserts the sanctioned
  delegation imports ARE present.
- **Planted negatives** write a forbidden snippet into a temporary tree
  and assert the scanner fires; a guard that has never been seen to fail
  is decoration.
- A **non-vacuousness check** asserts the scanned tree is the real
  package.
- A **package-contract test** pins the manifest posture — which peers
  exist, which are optional — and that no retired surface is referenced
  anywhere in source.

Cross-application test assets — the base end-to-end configuration,
viewport presets, harness helpers — live in a workspace-owned testing SDK
package; an application's configuration extends the base and overrides
only its own fields. Baselines follow a never-implicitly-write posture: a
missing baseline fails and is never created on the way down, and the
baseline corpus changes only through an explicit regeneration command.

**5. Wire it through the workspace.** Consumers reach the plugin package
through the workspace protocol, never file-path references; the version
discipline is the workspace's, and an internal package needs no
independent version ladder. Each consuming application keeps its platform
configuration as small typed data modules — declarations as data —
imported by its config file, and the config carries tripwires that run at
every load, asserting the application's own contract over its route and
locale matrix so a regression throws at config load with the contract
reference in the message. A standing fixture application holds its own
declarations; it never shares imports of the library's test fixture. The
preview vehicle — a private workspace application whose routes render the
surface under test — serves the production build in continuous
integration and a dev server locally, so what is verified is the shipped
artifact.

## Definition of Done

- [ ] Admission was decided against the table in step 1, and an upstream
      engine was configured or composed rather than re-implemented.
- [ ] The factory returns a plugin array the consumer spreads once, with
      ordering contracts documented and peers mirrored in devDependencies.
- [ ] The injection channel was chosen by enumerated decision record, not
      habit, and insertion is rank-ordered and missing-only.
- [ ] A fixture application drives the tests; guard tests include planted
      negatives and a non-vacuousness check.
- [ ] No consumer declaration is ever clobbered, and an enabled-but-unsatisfied
      feature fails loudly listing its discovery chain.

## Escalation

If no channel survives any output posture, that is a pipeline-level
question — an upstream contribution or a meta-framework hook — not a
reason to patch emitted files on disk as a permanent mechanism. If a
responsibility cannot be expressed without importing an engine's
internals, the boundary is wrong: raise it with the engine's public
surface or move the responsibility out of the plugin.

## Context

### Generated-code authority and the pinned toolchain behind the plugin's own output

- [Generated-Code Discipline](../../../contracts/reference/generated_code_discipline.md) (required reading)

### Package structure, exports, and publishing within the workspace

- [Package Development](../../../modules/reference/package_development.md) (required reading)

### Compose, never duplicate — the boundary rule behind composition-only presets

- [Contract-First Architecture](../../../contracts/reference/contract_first_architecture.md) (required reading)

### Workspace wiring and the exports field

- [Module Organization](../../../modules/reference/module_organization.md)
