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
  - "postcss.config.*"
  - "*/postcss.config.*"
  - "*/*/postcss.config.*"
  - "*/*/*/postcss.config.*"
  - "*/*/*/*/postcss.config.*"
  - "*/*/*/*/*/postcss.config.*"
  - "*/*/*/*/*/*/postcss.config.*"
  - "*/*/*/*/*/*/*/postcss.config.*"
  - "*/*/*/*/*/*/*/*/postcss.config.*"
  - "*/*/*/*/*/*/*/*/*/postcss.config.*"
  - "vite.config.*"
  - "*/vite.config.*"
  - "*/*/vite.config.*"
  - "*/*/*/vite.config.*"
  - "*/*/*/*/vite.config.*"
  - "*/*/*/*/*/vite.config.*"
  - "*/*/*/*/*/*/vite.config.*"
  - "*/*/*/*/*/*/*/vite.config.*"
  - "*/*/*/*/*/*/*/*/vite.config.*"
  - "*/*/*/*/*/*/*/*/*/vite.config.*"
---

# Tailwind v4 Configuration Context

Tailwind CSS v4 CSS-first configuration via @import, @theme, and @theme inline.

Tailwind v4 uses CSS-first configuration. Replace all `@tailwind` directives with `@import "tailwindcss"`. Define design tokens with `@theme` blocks — a `tailwind.config.js` is no longer detected, and loading one with `@config` is migration debt. Use `@theme inline` when tokens reference other CSS variables that should resolve at definition time. Load JavaScript plugins with `@plugin` (e.g., `@plugin "@tailwindcss/typography"`); `@import` is for CSS only. The PostCSS plugin is `@tailwindcss/postcss`; the Vite plugin is `@tailwindcss/vite`.

- [Tailwind v4 Configuration](../reference/config.md)
