---
paths:
  - "*.css"
  - "*/*.css"
  - "*/*/*.css"
  - "*/*/*/*.css"
  - "*/*/*/*/*.css"
  - "*/*/*/*/*/*.css"
  - "*/*/*/*/*/*/*.css"
  - "*/*/*/*/*/*/*/*.css"
  - "*/*/*/*/*/*/*/*/*.css"
  - "*/*/*/*/*/*/*/*/*/*.css"
---

# Tailwind v4 Architecture Context

Globals.css structure, theme namespaces, and dark mode configuration.

Structure `globals.css`: `@import "tailwindcss"` first, then `@theme` blocks, then `@layer` blocks, then custom styles. Variables in `@theme` generate utility classes and CSS variables; variables in `:root` are CSS variables only. Use `@theme static` to force emission of all variables regardless of usage. Content detection is automatic — no `content` config needed. Use `@source` to register additional scan paths. Dark mode defaults to `prefers-color-scheme`; override with `@custom-variant dark (&:where(.dark, .dark *))`.

- [Tailwind v4 Architecture](../reference/architecture.md)
