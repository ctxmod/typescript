# Fast Toolchain: Biome or Oxlint

One fast, Rust-based toolchain for formatting, linting, and import organization — Biome or Oxlint (with its formatter), chosen deliberately by recorded criteria; the legacy linter returns only for niche plugin rules, scoped and temporary.

The ES tier standardizes on one high-performance toolchain for formatting,
linting, and import organization, replacing the separate formatter + linter
stack that preceded it. Two toolchains qualify as equal options — **Biome**
and **Oxlint** with the Oxc formatter — and a workspace picks exactly one,
records why, and holds it. One config surface, one lockfile-pinned version
per tool — the same consolidation the other language tiers make with their
own toolchains.

## What the Toolchain Owns

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

## Choosing Biome or Oxlint

| Criterion | Biome (2.6.x) | Oxlint (1.x) + Oxfmt |
|---|---|---|
| Surface | One binary and one `biome.json` for format, lint, and import organization | Linter (`.oxlintrc.json` or `oxlint.config.ts`) plus a separate formatter (Oxfmt, `.oxfmtrc.json`) |
| Type-aware rules | Biome's own type inference, no TypeScript install; covers a partial, growing share (`noFloatingPromises` and related) | tsgolint on the native TypeScript 7 checker — stable since July 2026, covering most of typescript-eslint's typed rules, an extra `oxlint-tsgolint` dependency |
| Migrating an ESLint setup | Rule-by-rule mapping; custom rules as GritQL pattern plugins | ESLint-v8-compatible config shape and rule names; ESLint-compatible JavaScript plugins (alpha), so existing plugins often run unchanged |
| Formatter maturity | Stable, Prettier-compatible | Oxfmt is pre-1.0 (beta: passes Prettier's JS/TS conformance suite); Prettier-plugin support still on its roadmap |
| Other file types | Vue, Svelte, Astro (2.3+), CSS, JSON, GraphQL | JS/TS-family files, including the script blocks of Vue, Svelte, and Astro files |

Choose **Biome** when one stable binary for all three jobs matters most, the
workspace has no heavy typed-lint requirement, or framework template files
must be formatted and linted by the same tool. Choose **Oxlint** when typed
lint rules are load-bearing (floating promises, unsafe `any` flows, exhaustive
switches at typescript-eslint depth), when an existing ESLint plugin set must
keep running, or when the workspace already runs the Oxc/Vite toolchain.
Record the criterion that decided it beside the config; switching later is a
deliberate migration, never a second toolchain added "just in case".

### Biome example

```json
{
  "$schema": "https://biomejs.dev/schemas/2.6.8/schema.json",
  "vcs": { "enabled": true, "clientKind": "git", "useIgnoreFile": true },
  "formatter": { "enabled": true, "indentStyle": "space" },
  "linter": {
    "enabled": true,
    "rules": { "recommended": true },
    "domains": { "project": "recommended" }
  },
  "assist": { "actions": { "source": { "organizeImports": "on" } } }
}
```

Scripts: `biome check --write` (format, lint fixes, import organization) in
development and `biome ci` in CI. The `project` domain enables the
multi-file and type-inference rules, which turn on the project scanner.

### Oxlint example

```json
{
  "$schema": "./node_modules/oxlint/configuration_schema.json",
  "plugins": ["typescript", "unicorn", "oxc", "import"],
  "categories": { "correctness": "error", "suspicious": "warn" },
  "rules": { "typescript/no-floating-promises": "error" },
  "overrides": [
    { "files": ["scripts/*.ts"], "rules": { "no-console": "off" } }
  ]
}
```

Scripts: `oxlint --type-aware` (with `oxlint-tsgolint` installed; add
`--type-check` to report compiler errors in the same pass) and
`oxfmt --check` in CI, `oxfmt` to write. `oxlint --init` writes a starter
config; `oxlint.config.ts` (wrapped in `defineConfig`) is the typed
alternative on Node.js 22.18+/24+. Because tsgolint is built on the native
TypeScript 7 checker, typed linting here needs no 6.0 API install.

## The Legacy Linter's Remaining Role

The legacy linter (ESLint) returns only for **niche plugin rules not yet
covered** — framework-specific rule families whose plugins have no equivalent
in the chosen fast toolchain. When that need is real:

- scope it to the packages that carry the framework, never repo-wide by
  default;
- document, beside the config, which rule families justify the second
  toolchain and which fast-toolchain gap each fills;
- treat it as a debt position with a migration path — when the fast toolchain
  covers the family, the scoped config is deleted in the same change that
  adopts the coverage.

Before adding it, check whether the chosen toolchain already covers the
need: a Biome GritQL plugin, or the ESLint plugin itself running under
Oxlint's JavaScript-plugin API. typescript-eslint's typed rules import the
TypeScript 6.0 API, so a scoped legacy config with typed rules is one of the
consumers of the side-by-side 6.0 install
([Native TypeScript Compiler](typescript_native_compiler.md)).

Running two toolchains repo-wide "just in case" forfeits the speed
consolidation and recreates the two-config drift the single toolchain exists
to remove. Framework CLIs no longer bundle a linter step either: Next.js 16
removed `next lint` and `next build` no longer lints, so the workspace's lint
script is the only lint gate.

## Single Config Surface

One root config per tool, extended per package only for annotated, local
deltas (every deviation carries an inline rationale) — `biome.json` nested
configs, or Oxlint `overrides` and nested configs. Per-package floating
copies of tool configuration drift silently — the same discipline the
workspace applies to dependency versions applies to tool configuration.

## Related

- [Monorepo Orchestration](monorepo_orchestration.md)
- [Module Organization](../../modules/reference/module_organization.md)

## Sources

- https://biomejs.dev/blog/biome-v2/ — Biome v2: lint plugins, compiler-free type-aware rules (`noFloatingPromises`), project scanner, import organizer (2025-06-17; accessed 2026-11-22)
- https://next.biomejs.dev/blog/biome-v2-3 — Biome 2.3: Vue, Svelte, Astro support (accessed 2026-11-22)
- https://biomejs.dev/reference/configuration/ — `linter.domains` values, `vcs.useIgnoreFile`, `plugins` (`.grit`), `overrides` (accessed 2026-11-22)
- https://www.npmjs.com/package/@biomejs/biome — current release 2.6.8 (accessed 2026-11-22)
- https://oxc.rs/blog/2026-07-22-type-aware-linting-stable — Oxlint type-aware linting stable on tsgolint v7 / TypeScript 7, `--type-aware`, `--type-check` (2026-07-22; accessed 2026-11-22)
- https://oxc.rs/blog/2026-03-11-oxlint-js-plugins-alpha — Oxlint ESLint-compatible JS plugins in alpha (2026-03-11; accessed 2026-11-22)
- https://oxc.rs/docs/guide/usage/linter/config.html — `.oxlintrc.json` fields, `oxlint.config.ts` with `defineConfig` (Node.js 22.18+/24+), `oxlint --init`, ESLint v8 config compatibility (accessed 2026-11-22)
- https://oxc.rs/blog/2026-02-24-oxfmt-beta — Oxfmt beta: Prettier JS/TS conformance, roadmap to stable (2026-02-24; accessed 2026-11-22)
- https://www.npmjs.com/package/oxlint — current releases oxlint 1.94.0, oxlint-tsgolint 7.0.x, oxfmt 0.79.0 (accessed 2026-11-22)
- https://devblogs.microsoft.com/typescript/announcing-typescript-7-0/ — typescript-eslint needs the 6.0 API alongside 7.0 (2026-07-08; accessed 2026-11-22)
- https://nextjs.org/blog/next-16 — `next lint` removed; `next build` no longer lints (2025-10-21; accessed 2026-11-22)
