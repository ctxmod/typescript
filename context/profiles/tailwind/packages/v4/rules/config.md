# Tailwind v4 Configuration Context

Tailwind CSS v4 CSS-first configuration via @import, @theme, and @theme inline.

Tailwind v4 uses CSS-first configuration. Replace all `@tailwind` directives with `@import "tailwindcss"`. Define design tokens with `@theme` blocks — no `tailwind.config.js` needed or supported. Use `@theme inline` when tokens reference other CSS variables that should resolve at definition time. Load first-party plugins via `@import` (e.g., `@import "@tailwindcss/typography"`). The PostCSS plugin is `@tailwindcss/postcss`; the Vite plugin is `@tailwindcss/vite`.

- [Tailwind v4 Configuration](../reference/config.md)
