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
  - "*.tsx"
  - "*/*.tsx"
  - "*/*/*.tsx"
  - "*/*/*/*.tsx"
  - "*/*/*/*/*.tsx"
  - "*/*/*/*/*/*.tsx"
  - "*/*/*/*/*/*/*.tsx"
  - "*/*/*/*/*/*/*/*.tsx"
  - "*/*/*/*/*/*/*/*/*.tsx"
  - "*/*/*/*/*/*/*/*/*/*.tsx"
  - "*.jsx"
  - "*/*.jsx"
  - "*/*/*.jsx"
  - "*/*/*/*.jsx"
  - "*/*/*/*/*.jsx"
  - "*/*/*/*/*/*.jsx"
  - "*/*/*/*/*/*/*.jsx"
  - "*/*/*/*/*/*/*/*.jsx"
  - "*/*/*/*/*/*/*/*/*.jsx"
  - "*/*/*/*/*/*/*/*/*/*.jsx"
  - "*.html"
  - "*/*.html"
  - "*/*/*.html"
  - "*/*/*/*.html"
  - "*/*/*/*/*.html"
  - "*/*/*/*/*/*.html"
  - "*/*/*/*/*/*/*.html"
  - "*/*/*/*/*/*/*/*.html"
  - "*/*/*/*/*/*/*/*/*.html"
  - "*/*/*/*/*/*/*/*/*/*.html"
---

# Tailwind v4 Migration Context

Tailwind CSS v3 to v4 migration pitfalls and breaking changes.

Key v3 to v4 breaking changes: `theme()` dot notation is deprecated in favor of CSS variable references (`var(--color-red-500)`; `theme(--breakpoint-xl)` in media queries). `hover:` variant now only fires on devices supporting hover — restore v3 behavior with `@custom-variant hover (&:hover)`. Individual transform properties (`rotate`, `scale`, `translate`) replace the `transform` shorthand — `transform-none` does not reset them. `border-*` and `divide-*` default border color changed to `currentColor`. `@apply` in separately-bundled stylesheets requires `@reference` to import theme context. v4 requires Safari 16.4+, Chrome 111+, Firefox 128+.

- [Tailwind v4 Migration](../reference/migration.md)
