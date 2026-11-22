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
---

# ECMAScript Toolchain

Biome-first toolchain for formatting, linting, and import sorting; the legacy linter only for scoped niche plugin rules.

The fast single-toolchain posture covers formatting, linting, and import
organization in one config surface; formatting decisions are settled by the
tool, not in review. The legacy linter returns only where a niche plugin rule
family has no equivalent — scoped to the packages that need it, documented,
and deleted when coverage lands.

[Biome-First Toolchain](../reference/biome_toolchain.md)
