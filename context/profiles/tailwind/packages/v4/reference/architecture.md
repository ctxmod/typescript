# Tailwind v4 Architecture

Globals.css structure, theme namespaces, and dark mode.

## globals.css Structure

**Standard file order**

```css
@import "tailwindcss";
@import "@tailwindcss/typography";

@theme inline {
  --color-background: var(--background);
  --color-foreground: var(--foreground);
}

@theme {
  --color-primary: #2563eb;
  --font-sans: "Inter", sans-serif;
}

@layer base {
  h1 { font-size: var(--text-2xl); }
}

@layer components {
  .card { /* ... */ }
}

@utility content-auto {
  content-visibility: auto;
}
```

Order matters: `@import "tailwindcss"` must come first, `@theme` after the import to override defaults, `@layer` blocks can appear anywhere after the import.

## Theme Variable Namespaces

**`@theme` variables generate utilities and CSS variables**

Variables defined in `@theme` produce both a CSS custom property and a corresponding Tailwind utility class. `--color-primary` generates both `var(--color-primary)` and `bg-primary`/`text-primary`/etc.

**`:root` variables are CSS-only**

Variables defined in `:root` (or under selectors) are CSS custom properties with no utility generation. Use these for values consumed by JavaScript or external CSS that doesn't need Tailwind utilities.

**Namespace convention**

Use the appropriate namespace prefix for token types:

| Prefix | Generates |
|--------|-----------|
| `--color-*` | Color utilities (bg, text, border, ring) |
| `--font-*` | Font family utilities |
| `--spacing-*` | Spacing utilities |
| `--radius-*` | Border radius utilities |
| `--shadow-*` | Box shadow utilities |
| `--breakpoint-*` | Responsive breakpoints |
| `--animate-*` | Animation utilities |

## Dark Mode

**Default: `prefers-color-scheme` media query**

The `dark:` variant works out of the box following OS preference. No configuration needed.

**Class-based toggle: `@custom-variant dark`**

```css
/* Class strategy */
@custom-variant dark (&:where(.dark, .dark *));

/* Data attribute strategy */
@custom-variant dark (&:where([data-theme=dark], [data-theme=dark] *));

/* Theme class applied to <html> */
/* <html class="dark"> */
```

This replaces the v3 `darkMode: "class"` JavaScript config option.

## Content Detection

**Automatic — no `content` configuration needed**

Tailwind v4 scans all project files except `.gitignore`d paths, `node_modules`, binary files, CSS files, and lock files.

**Register additional paths with `@source`**

```css
@source "../node_modules/@acmecorp/ui-lib";
@source not "../src/legacy";
```

**Safelist with `@source inline()`**

```css
@source inline("{hover:,}bg-red-{50,{100..900..100},950}");
```

## Thresholds

| Metric | Pass | Warn | Fail |
|--------|------|------|------|
| `content` config in v4 | 0 | 0 | >0 |
| `@theme` namespace misuse | 0 | 0 | >0 |

- KISS (engineering module, `principles` profile: `design/reference/kiss.md`)
