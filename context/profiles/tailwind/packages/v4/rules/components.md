---
paths:
  - "*.css"
---

# Tailwind v4 Component Styling Context

Tailwind CSS v4 component styling with `@utility`, `@layer components`, and `@variant`.

Prefer extracting framework components over `@apply`. Use `@utility` for classes that need variant support (responsive, state, dark mode). Use `@layer components` for overridable component classes — they have lower specificity than utilities. Use `@variant` to apply Tailwind variants inside custom CSS instead of `@apply hover:bg-black`. Classes in `@layer components` reference theme variables directly (`var(--color-...)`) — do not use `@apply` within them.

- [Tailwind v4 Component Styling](../reference/components.md)
