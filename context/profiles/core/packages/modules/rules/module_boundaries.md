---
paths:
  - "*.ts"
  - "*.tsx"
  - "*.mts"
  - "*.cts"
---

# Module Boundaries

Deep-module design, deepening, and enforced dependency boundaries govern how TypeScript package source exposes behaviour and where its seams sit.

Deep-module design and boundary enforcement govern TypeScript package source: packages expose behaviour through entry points, subfolders stay private, and seams are chosen deliberately rather than emergent.

- [Deep Module Design](../reference/deep_module_design.md)
- [Deepening Shallow Modules](../reference/deepening_shallow_modules.md)
- [Dependency Boundary Enforcement](../reference/dependency_boundary_enforcement.md)
- [Interface Alternatives](../reference/interface_alternatives.md)
