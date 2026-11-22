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

## What the Fast Toolchain Covers Now

The coverage line the legacy-linter rule tests against moves, so check it
at the pinned version before keeping a second toolchain:

- **Biome 2.x** (2.0 shipped June 2025; the 2.5 line is current) adds
  pattern-matching lint plugins (GritQL), multi-file analysis behind an
  opt-in project scanner, type-aware rules that run on Biome's own type
  inference without installing the TypeScript compiler (`noFloatingPromises`
  and related rules, a partial but growing share of typescript-eslint's
  coverage), and — since 2.3 — Vue, Svelte, and Astro file support. A
  custom rule that once justified a legacy-linter plugin is often a small
  GritQL plugin now.
- **The legacy linter's typed rules depend on the compiler API.**
  typescript-eslint's type-aware rules import the TypeScript 6.0 API, which
  the native TypeScript 7 compiler does not provide yet; a scoped legacy
  config with typed rules is one of the tools that keeps the 6.0 install
  alive ([Native TypeScript Compiler](typescript_native_compiler.md)). Count
  that cost when deciding whether the niche rule family still earns its
  place.
- **Oxlint is the other fast linter.** Its type-aware mode (via tsgolint,
  built directly on TypeScript 7) is stable since July 2026, and its
  ESLint-compatible JavaScript plugin API is in alpha. It is a linter only
  (no formatter or import organizer in the same binary), so adopting it does
  not change the formatting posture above; choosing it over Biome for
  linting is the same kind of recorded toolchain decision, never a third
  layer added "just in case".

Framework CLIs no longer bundle a linter step either: Next.js 16 removed
`next lint` and `next build` no longer lints, so the workspace's lint script
is the only lint gate.

## Single Config Surface

One root config, extended per package only for annotated, local deltas
(every deviation carries an inline rationale). Per-package floating copies of
tool configuration drift silently — the same discipline the workspace applies
to dependency versions applies to tool configuration.

## Related

- [Monorepo Orchestration](monorepo_orchestration.md)
- [Module Organization](../../modules/reference/module_organization.md)

## Sources

- https://biomejs.dev/blog/biome-v2/ — Biome v2: lint plugins, compiler-free type-aware rules (`noFloatingPromises`), project scanner, import organizer (2025-06-17; accessed 2026-11-22)
- https://next.biomejs.dev/blog/biome-v2-3 — Biome 2.3: Vue, Svelte, Astro support (accessed 2026-11-22)
- https://github.com/biomejs/biome/releases — current 2.5.x CLI releases (accessed 2026-11-22)
- https://oxc.rs/blog/2026-07-22-type-aware-linting-stable — Oxlint type-aware linting stable on tsgolint v7 / TypeScript 7 (2026-07-22; accessed 2026-11-22)
- https://oxc.rs/blog/2026-03-11-oxlint-js-plugins-alpha — Oxlint ESLint-compatible JS plugins in alpha (2026-03-11; accessed 2026-11-22)
- https://devblogs.microsoft.com/typescript/announcing-typescript-7-0/ — typescript-eslint needs the 6.0 API alongside 7.0 (2026-07-08; accessed 2026-11-22)
- https://nextjs.org/blog/next-16 — `next lint` removed; `next build` no longer lints (2025-10-21; accessed 2026-11-22)
