# Design Token Architecture

Tailwind v4 @theme directive, shadcn theming, semantic naming, and dark mode bridge variables.

Design tokens defined in Tailwind v4's `@theme` directive serve as the single styling API. shadcn components consume these tokens through CSS variables, enabling theme switching without class changes. Three distinct layers separate compile-time utility generation from runtime theme switching.

## Token Layers

Each layer handles a different concern in the token pipeline.

| Layer | Mechanism | Purpose |
|-------|-----------|---------|
| Static tokens | `@theme { }` | Generate Tailwind utilities at compile time |
| Bridge tokens | `@theme inline { }` | Expose runtime-switchable variables as utilities |
| Theme values | `@layer theme { }` | Assign per-theme values to bridge variables |

Static tokens produce fixed utilities. Bridge tokens reference CSS variables that change at runtime, and the theme layer swaps those variables under selectors like `:root` and `.dark`.

```css
@import "tailwindcss";

/* Static: generates bg-primary, text-primary utilities */
@theme {
  --color-primary: oklch(0.62 0.14 210);
  --radius-md: 0.5rem;
}

/* Bridge: runtime-switchable surface tokens */
@theme inline {
  --color-background: var(--bg);
  --color-foreground: var(--fg);
}

/* Theme assignments: swap under selectors */
@layer theme {
  :root { --bg: oklch(0.98 0.01 250); --fg: oklch(0.20 0.02 250); }
  .dark { --bg: oklch(0.13 0.02 250); --fg: oklch(0.90 0.02 250); }
}
```

The `@theme inline` directive registers variables with Tailwind's utility generator while deferring their values to runtime. Standard `@theme` bakes values into the compiled output. Choosing between them determines whether a token is fixed or switchable.

## Semantic Token Naming

Name tokens by purpose rather than visual property. Semantic names survive palette changes without refactoring utility classes across the codebase.

| Pattern | Good | Avoid |
|---------|------|-------|
| Surfaces | `--color-background`, `--color-card`, `--color-muted` | `--color-white`, `--color-gray-100` |
| Actions | `--color-primary`, `--color-destructive` | `--color-blue-500`, `--color-red-600` |
| Text | `--color-foreground`, `--color-muted-foreground` | `--color-gray-900`, `--color-gray-500` |
| Borders | `--color-border`, `--color-input` | `--color-gray-200` |
| Layout | `--radius-sm`, `--radius-md`, `--radius-lg` | `--radius-4`, `--radius-8` |

The foreground convention pairs each surface token with a text counterpart: `--color-card` gets `--color-card-foreground`, `--color-muted` gets `--color-muted-foreground`. This pairing guarantees readable contrast across themes.

## shadcn Theme Integration

shadcn uses CSS variables for all component colors. The `components.json` file configures the integration.

```json
{
  "$schema": "https://ui.shadcn.com/schema.json",
  "style": "new-york",
  "tailwind": {
    "cssVariables": true
  }
}
```

shadcn components reference variables like `--background`, `--foreground`, `--primary`, `--border`. Map project tokens to these variables in the theme layer so components receive values without source modification.

```css
@layer theme {
  :root {
    --background: var(--color-background);
    --foreground: var(--color-foreground);
    --primary: var(--color-primary);
    --border: var(--color-border);
    --ring: var(--color-ring);
    --radius: var(--radius-md);
  }
}
```

This indirection lets project-level tokens drive shadcn components. When palettes change, update the token definitions once; components adapt automatically.

## Dark Mode Architecture

The bridge variable pattern handles theme switching through four steps:

1. Define static tokens for each theme's color values
2. Declare bridge tokens in `@theme inline` referencing CSS custom properties
3. Swap underlying properties under `.dark` (or `data-theme`) selector
4. Tailwind utilities adapt automatically -- no conditional classes needed

Components styled with `bg-background text-foreground` render correctly in both themes. Adding a third theme (high contrast, for example) requires only a new selector block in `@layer theme` with the corresponding variable assignments.

## Color Space

Use OKLCH for all color definitions. OKLCH provides perceptually uniform lightness, which means adjusting the first parameter creates predictable hover and active states. A darker variant of `oklch(0.62 0.14 210)` is `oklch(0.55 0.14 210)` -- the chroma and hue remain stable while lightness decreases proportionally. The `oklch()` function has full support in modern browsers and produces more consistent results than HSL across the hue spectrum.

## Thresholds

| Metric | Pass | Warn | Fail |
|--------|------|------|------|
| Non-semantic color references | 0 | 1-3 | >3 |
| Token layers defined | 3 (static, bridge, theme) | 2 | 1 |
| Themes supported | 2+ (light, dark) | 1 | 0 |
| Arbitrary color values per component | 0 | 1-2 | >2 |

- Convention over configuration (engineering module, `principles` profile: `design/reference/convention_over_configuration.md`)
the react_router profile's *Styling Philosophy* reference (`profiles/react_router/packages/components/reference/styling_philosophy.md`)
[shadcn/ui Integration](shadcn_integration.md)
