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

JavaScript configuration files are not detected in v4. All configuration uses CSS directives: `@theme` for design tokens, `@utility` for custom utilities, `@custom-variant` for custom variants, `@layer` for component classes.

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

Without `inline`, the utility class emits a `var()` reference. With `inline`, the computed value is substituted. Use `inline` for tokens that reference other CSS variables.

**Use `@theme static` to force all variable emission**

By default, v4 only emits CSS variables for utilities that appear in scanned source files. `@theme static` forces all variables to be emitted — needed when variables are consumed by external CSS or JavaScript.

## Plugins

**Load plugins via `@import`**

```css
@import "tailwindcss";
@import "@tailwindcss/typography";
@import "@tailwindcss/forms";
```

First-party plugins load via `@import`. Legacy JavaScript-based plugins use the `@plugin` directive.

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

**Content detection is automatic**

No `content` configuration is needed. Tailwind v4 scans all project files except `.gitignore`d paths, `node_modules`, binary files, CSS files, and lock files. Use `@source` to register additional scan paths or `@source inline("class-name")` to safelist classes.

## Thresholds

| Metric | Pass | Warn | Fail |
|--------|------|------|------|
| `tailwind.config.js` present in v4 project | 0 | 0 | >0 |
| `@tailwind` directives in v4 CSS | 0 | 0 | >0 |
| `@theme` for design tokens | 100% | 0% | 0% |

- KISS (engineering module, `principles` profile: `design/reference/kiss.md`)
