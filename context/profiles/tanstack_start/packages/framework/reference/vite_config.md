# Vite Configuration

Vite configuration composition — configure-never-reimplement with a naming glossary, the workspace-local foundation package owning one typed declaration, declarations-as-data with config-load tripwires, a configure-only factory with a build-time agreement gate, first-class degradation, and custom plugin development through the es skill.

The Vite configuration is composed from declarations, not hand-assembled.
This reference owns the composition discipline: what a consuming
application's configuration file contains, the foundation package that
turns declarations into option fragments, and the tripwires that make a
regression fail at configuration load rather than in a deployed build.

## Configure, Never Reimplement

An application — and any foundation package it composes — declares and
configures the upstream framework's options. It never reimplements
prerendering, sitemap generation, robots serving, or rendering. A
configure-only posture is pinned by a source guard in the owning
package, so the mandate is enforced rather than remembered.

The supporting discipline is a naming glossary: where one overloaded
word carries several senses (a rendering term that names both the
served shell and the pending fallback and the route-content behavior),
resolve the polysemy into one term per concept before naming any
export, and derive every exported symbol name from the glossary. Every
name a consumer types should carry exactly one meaning.

## The Foundation Package

Where more than one application in a workspace shares a full-stack
posture, the composition machinery lives in a workspace-local
foundation package — a peer package under the workspace protocol, never
a file-path reference. It owns the application-profile declaration
type, the compilers that turn one declaration into per-surface option
fragments, and the factory below. Application-specific values never
enter it; it carries the mechanism, and each application carries its
own declarations.

Package mechanics — subpath exports so browser bundles never pull
node-only surfaces, type-only framework imports, peers mirrored in
development dependencies — are the TypeScript core's:
[Package Development](../../../../core/packages/modules/reference/package_development.md) and
[Module Organization](../../../../core/packages/modules/reference/module_organization.md).

## Declarations as Data, with a Composition Seam

A consuming application keeps its platform configuration as small typed
data modules — the application profile, the indexability declaration,
the locale set, the site origin — imported by its Vite configuration
file. Because the configuration file is loaded by a runtime that cannot
always execute the framework-facing barrel, a composition-seam module
re-exports exactly the configuration-time surface the file needs,
keeping the heavy surface out of the load path.

The configuration file carries **tripwires that run at every load** —
build and serve alike. A tripwire asserts the application's own
contract over its route and locale matrix: a degradation that should be
a no-op producing output, an emission appearing under a declaration
that forbids it, a route matrix drifting from the declaration — each
throws at configuration load with the contract reference in the
message. A regression that fails at config load never reaches CI as a
mystery build artifact.

The same declarations feed how the application is exercised, not only
how it builds. A workspace-owned preview vehicle — a private workspace
application rendering the surfaces under development — composes its
configuration from the application's own declaration modules, so the
composition being exercised is the composition that ships. An
application is exercised through that vehicle, never through a bespoke
per-application development script. The vehicle's construction and
verification posture are owned by the plugin-development skill:

- [Develop a Vite Plugin](../../../../core/packages/toolchain/skills/develop_vite_plugin/SKILL.md)

## One Factory, Every Fragment, One Gate

A single factory call takes the application's declarations and returns
typed option fragments for every consumption surface — the framework
plugin options, the router configuration, per-route head emission
helpers, server-route options objects:

```text
factory(declarations) -> fragments:
    manifest = compile(declarations.indexability)
    locale   = compile(declarations.locales)
    gate(manifest, locale)            # agreement gate, at factory-call time
    return [pluginOptions(manifest, locale),
            routerConfig(manifest),
            headEmission(manifest),
            serverRouteOptions(manifest)]
```

The factory composes existing compilers; it re-derives nothing.
Localized paths derive from the compiled URL mapping, never from string
surgery over URL patterns. The agreement gate runs at factory-call
time — so importing the factory into a configuration file is itself the
build-time gate, and a bad declaration fails the build at configuration
load with the disagreement named.

Two commitments bind delivery:

- **One delivery channel per concern.** A concern that could ship two
  ways (head links versus alternate-link emission) commits to exactly
  one, and a dual-delivery attempt is rejected by the same checker.
- **Degradation is a first-class mode, not a failure.** A single-locale
  declaration degrades to a documented no-op — no localized entries, no
  emission, zero errors — mirroring the single-locale application
  profile the es locale-strategies reference owns.

## The Generated Route Tree Posture

The framework's generated route tree is a generated artifact under the
es discipline ([Generated-Code Discipline](../../../../core/packages/contracts/reference/generated_code_discipline.md)).
The posture choice specific to this framework: where the repository's
typecheck tier sweeps application sources, the tree is committed so a
fresh clone resolves its imports, every harness boot regenerates it,
and the regeneration ritual is stated where the tree lives. The
committed-versus-regenerate decision itself is the discipline
reference's; this reference records only the framework-specific
consequence.

## Custom Plugins

Where a build-time responsibility genuinely needs a plugin — admission
decided against configuration first, package structure second, hook
channel third, guard tests fourth — the development procedure is the es
skill, which sequences it:

- [Develop a Vite Plugin](../../../../core/packages/toolchain/skills/develop_vite_plugin/SKILL.md)

## Where the Rest Lives

- The **rendering-modes reference** owns the application profile and
  the mode axis the fragments configure.
- The **contract-first architecture reference** owns the wider
  declarations-are-source discipline this composition sits inside.
- The **package-development procedure** owns workspace package posture
  for the foundation package.

## Related

- [Contract-First Architecture](../../../../core/packages/contracts/reference/contract_first_architecture.md)
- [Rendering Modes](rendering_modes.md)
