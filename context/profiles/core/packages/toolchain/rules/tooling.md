---
paths:
  - "biome.json"
  - "*/biome.json"
  - "*/*/biome.json"
  - "*/*/*/biome.json"
  - "*/*/*/*/biome.json"
  - "*/*/*/*/*/biome.json"
  - "biome.jsonc"
  - "*/biome.jsonc"
  - "*/*/biome.jsonc"
  - "*/*/*/biome.jsonc"
  - "*/*/*/*/biome.jsonc"
  - "*/*/*/*/*/biome.jsonc"
  - ".oxlintrc.json"
  - "*/.oxlintrc.json"
  - "*/*/.oxlintrc.json"
  - "*/*/*/.oxlintrc.json"
  - "*/*/*/*/.oxlintrc.json"
  - "*/*/*/*/*/.oxlintrc.json"
  - "oxlint.config.ts"
  - "*/oxlint.config.ts"
  - "*/*/oxlint.config.ts"
  - "*/*/*/oxlint.config.ts"
  - "*/*/*/*/oxlint.config.ts"
  - "*/*/*/*/*/oxlint.config.ts"
  - "oxlint.config.mts"
  - "*/oxlint.config.mts"
  - "*/*/oxlint.config.mts"
  - "*/*/*/oxlint.config.mts"
  - "*/*/*/*/oxlint.config.mts"
  - "*/*/*/*/*/oxlint.config.mts"
  - ".oxfmtrc.json"
  - "*/.oxfmtrc.json"
  - "*/*/.oxfmtrc.json"
  - "*/*/*/.oxfmtrc.json"
  - "*/*/*/*/.oxfmtrc.json"
  - "*/*/*/*/*/.oxfmtrc.json"
---

# ECMAScript Toolchain

One fast toolchain — Biome, or Oxlint with Oxfmt — for formatting, linting, and import sorting, chosen by recorded criteria; the legacy linter only for scoped niche plugin rules.

Biome and Oxlint are equal options: pick exactly one per workspace by the
decision criteria (one binary for all three jobs versus typed-lint depth and
ESLint-plugin compatibility), record the deciding criterion beside the
config, and never run both. Formatting decisions are settled by the tool,
not in review. The legacy linter returns only where a niche plugin rule
family has no equivalent — scoped to the packages that need it, documented,
and deleted when coverage lands.

- [Fast Toolchain: Biome or Oxlint](../reference/fast_toolchain.md)
