# Tailwind v4 Configuration

Tailwind CSS v4 CSS-first configuration.

## CSS-First Setup

**Replace all `@tailwind` directives with `@import`**

The three v3 directives are replaced by one line:

```css
/* v3 (removed) */
@tailwind base;
@tailwind components;
@tailwind utilities;

/* v4 */
@import "tailwindcss";
```

**No `tailwind.config.js` — configuration lives in CSS**

JavaScript configuration files are no longer detected automatically in v4. All configuration uses CSS directives: `@theme` for design tokens, `@utility` for custom utilities, `@custom-variant` for custom variants, `@layer` for component classes. A legacy JavaScript config can still be loaded explicitly with `@config "../../tailwind.config.js";` (its `corePlugins`, `safelist`, and `separator` options are unsupported); treat that as migration debt with a removal plan, not as a configuration surface.

## Design Tokens

**Define tokens with `@theme` blocks**

```css
@theme {
  --color-primary: #2563eb;
  --font-sans: "Inter", sans-serif;
  --radius-md: 0.5rem;
}
```

Variables in `@theme` generate utility classes and become CSS variables. `bg-primary`, `text-primary`, `rounded-md` all work automatically.

**Reset a default namespace with `--*: initial`**

`--color-*: initial;` inside `@theme` removes every default color utility so only the project's own palette exists — the way to make off-palette classes like `bg-red-500` unavailable rather than merely discouraged.

**Use `@theme inline` for runtime-switchable tokens**

```css
@theme inline {
  --color-background: var(--background);
  --color-foreground: var(--foreground);
}

:root {
  --background: #ffffff;
  --foreground: #1a1a1a;
}
.dark {
  --background: #1a1a1a;
  --foreground: #ffffff;
}
```

Without `inline`, the utility references the theme variable (`var(--color-background)`), and that variable resolves where it is defined — where the referenced runtime variable may not have its value yet. With `inline`, the utility uses the theme variable's value (`var(--background)`) directly. Use `inline` for every token that references another CSS variable.

**Use `@theme static` to force all variable emission**

By default, v4 only emits CSS variables for utilities that appear in scanned source files. `@theme static` forces all variables to be emitted — needed when variables are consumed by external CSS or JavaScript.

## Plugins

**Load JavaScript plugins via `@plugin`**

```css
@import "tailwindcss";
@plugin "@tailwindcss/typography";
@plugin "@tailwindcss/forms";
```

Plugins such as `@tailwindcss/typography` and `@tailwindcss/forms` are JavaScript plugins and load with `@plugin` (a package name or a local path). `@import` inlines CSS — Tailwind itself and plain stylesheets — and does not load a JavaScript plugin.

## Build Integration

**PostCSS plugin: `@tailwindcss/postcss`**

```javascript
// postcss.config.mjs
export default {
  plugins: { "@tailwindcss/postcss": {} }
};
```

`postcss-import` and `autoprefixer` are no longer needed (handled automatically by v4).

**Vite plugin: `@tailwindcss/vite`**

Faster than PostCSS. Use when using Vite as the build tool.

**webpack loader: `@tailwindcss/webpack`** (v4.2+)

A dedicated loader that skips the PostCSS step, for webpack-based builds.

**Content detection is automatic**

No `content` configuration is needed. Tailwind v4 scans all project files except `.gitignore`d paths, `node_modules`, binary files, CSS files, and lock files. Use `@source` to register additional scan paths or `@source inline("class-name")` to safelist classes.

## Thresholds

| Metric | Pass | Warn | Fail |
|--------|------|------|------|
| `tailwind.config.js` present in v4 project | 0 | Loaded via `@config` with a removal plan | Present and unreferenced, or loaded without a plan |
| `@tailwind` directives in v4 CSS | 0 | 0 | >0 |
| `@theme` for design tokens | 100% | 0% | 0% |

- KISS (engineering module, `principles` profile: `design/reference/kiss.md`)

## Sources

- https://tailwindcss.com/docs/functions-and-directives — `@plugin` for JS plugins, `@config` for legacy JS config, `@source`, `@reference`, `@variant`, `@utility`, `--alpha()`, `--spacing()`, `theme()` deprecated (accessed 2026-11-22)
- https://tailwindcss.com/docs/upgrade-guide — browser floor, `border-*`/`divide-*` default `currentColor`, `theme(--breakpoint-xl)` in media queries, JS config not auto-detected, `npx @tailwindcss/upgrade` on Node.js 20+ (accessed 2026-11-22)
- https://tailwindcss.com/docs/theme — `@theme inline`, `@theme static`, namespace reset with `--color-*: initial` (accessed 2026-11-22)
- https://tailwindcss.com/blog/tailwindcss-v4-3 — v4.2 palettes, `@tailwindcss/webpack`, logical inset/block utilities and the `start-*`/`end-*` deprecation; v4.3 scrollbar, `zoom-*`, `tab-*`, stacked/compound `@variant`, `--default()` (2026-05-08; accessed 2026-11-22)
