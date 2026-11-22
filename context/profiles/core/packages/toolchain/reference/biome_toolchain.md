# Biome-First Toolchain

Biome-first positioning — one fast toolchain for formatting, linting, and import sorting; the legacy linter returns only for niche plugin rules, scoped and temporary.

The ES tier standardizes on one high-performance toolchain for formatting,
linting, and import organization, replacing the separate formatter + linter
stack that preceded it. One binary, one config surface, one lockfile-pinned
version — the same consolidation the other language tiers make with their own
toolchains.

## What Biome Owns

- **Formatting** — the formatter decides style; formatting debates are settled
  by tool choice, not review comments. Code is always formatted before it is
  reviewed.
- **Linting** — the rule set covers the correctness, style, and suspicious-code
  families that drove the legacy linter's default config.
- **Import organization** — import sorting runs in the same pass, removing the
  dedicated import-sort plugin from the toolchain.

The speed difference is not a benchmark curiosity — it is what makes
format-plus-lint-on-every-save and check-on-every-commit affordable on large
workspaces, which in turn is what keeps violations out of review.

## The Legacy Linter's Remaining Role

The legacy linter (ESLint) returns only for **niche plugin rules not yet
covered** — framework-specific rule families whose plugins have no equivalent
in the fast toolchain. When that need is real:

- scope it to the packages that carry the framework, never repo-wide by
  default;
- document, beside the config, which rule families justify the second
  toolchain and which fast-toolchain gap each fills;
- treat it as a debt position with a migration path — when the fast toolchain
  covers the family, the scoped config is deleted in the same change that
  adopts the coverage.

Running both toolchains repo-wide "just in case" forfeits the speed
consolidation and recreates the two-config drift the single toolchain exists
to remove.

## Single Config Surface

One root config, extended per package only for annotated, local deltas
(every deviation carries an inline rationale). Per-package floating copies of
tool configuration drift silently — the same discipline the workspace applies
to dependency versions applies to tool configuration.

## Related

- [Monorepo Orchestration](monorepo_orchestration.md)
- [Module Organization](../../modules/reference/module_organization.md)
