# Monorepo Orchestration

Workspace catalog as the dependency-version SSOT, graph-based versus minimal-cache orchestrators as a deliberate choice, and the task-graph ownership rules that make caching sound.

The workspace layout (which packages exist, how they are grouped) is owned by
the module-organization reference. This reference owns how tasks run and how
versions stay single-sourced across a multi-package workspace.

## The Catalog: Dependency-Version SSOT

Versions that more than one package shares are pinned once, in the
workspace's `catalog:` table, and referenced from package manifests with the
`catalog:` protocol. One library, one version row: a coordinated bump is a
one-line change, and the class of drift where two packages silently resolve
different major versions of the same library disappears. Versions unique to
one package stay in that manifest; the catalog is for shared pins, not for
centralizing everything. Generated lockfile state stays derived — the catalog
is the reviewed input.

## Orchestrator Choice: Graph-Based versus Minimal-Cache

A task orchestrator gives the workspace a dependency graph over
build/lint/test tasks and a cache keyed on task inputs. Two postures exist,
and picking one is a deliberate, stated decision:

| Posture | Shape | Fits |
|---|---|---|
| Graph-based suite | Project graph, generators, visualization, plugin/executor surface | Larger workspaces, codegen-heavy flows, teams needing enforced module boundaries and cross-package generators |
| Minimal-cache layer | One task config file, local (and optional shared) caching, little else | Smaller workspaces that want the task graph and cache without platform machinery |

Both give the load-bearing parts — ordered execution along the package graph
and input-keyed caching. The rule is to choose by workspace scale and
workflow needs, record the choice once, and not run two orchestrators. A
workspace that outgrows the minimal layer migrates once, deliberately.

## Task-Graph Ownership

- **Every task declares its inputs and outputs.** Caching is only sound when
  the input globs are complete: a missing input means a stale cache hit that
  silently serves wrong artifacts — worse than no cache. Outputs declared
  wrongly fail visibly; inputs declared wrongly fail invisibly.
- **Pipeline dependencies are declared between tasks, not encoded in shell
  sequencing.** `build` depends on upstream `build`s through the graph, so
  parallelism and caching come from the declaration, not the author's
  ordering instincts.
- **Task definitions stay package-local** (each package owns its `build`,
  `test`, `lint` scripts); the root owns only the pipeline wiring. A root
  script that re-implements a package's task duplicates it.

## Typed Go Task Runners Stay Go-Side

Where a repository also carries Go tooling, the Go tier runs its tasks with a
typed Go task runner — and the boundary between the two orchestrators is a
process boundary: the JS/TS orchestrator may invoke the Go toolchain as a
subprocess (an executor that shells out), and vice versa, but task logic is
not ported across the boundary. Each tier's runner owns its tier's tasks;
shared logic lives in the shared tool, invoked as a binary, not in
duplicated task definitions.

## Related

- [Module Organization](../../modules/reference/module_organization.md)
- [Biome-First Toolchain](biome_toolchain.md)
