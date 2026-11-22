# Styling Philosophy

Tailwind CSS v4 integration patterns with design tokens, CVA variants, and minimal custom CSS.

Tailwind CSS v4 provides the styling foundation through CSS-first configuration, utility-first class composition, and native container queries. Custom CSS is a last resort, reserved for capabilities Tailwind utilities cannot express.

- KISS (engineering module, `principles` profile: `design/reference/kiss.md`)
- Convention over configuration (engineering module, `principles` profile: `design/reference/convention_over_configuration.md`)

## CSS-First Configuration

Tailwind v4 replaces `tailwind.config.js` with the `@theme` directive in CSS. Design tokens defined in `@theme` blocks generate both CSS variables and Tailwind utilities without JavaScript configuration. This architectural shift delivers 3.5-5x faster builds and eliminates configuration file complexity.

```css
@import "tailwindcss";

@theme {
  --color-primary: oklch(0.53 0.12 118.34);
  --color-background: oklch(0.98 0.00 0);
  --font-display: "Satoshi", sans-serif;
}
```

Use `@theme` for tokens that should generate utilities (e.g., `--color-primary` produces `bg-primary`, `text-primary`). Use standard CSS `:root` variables for runtime-only values that do not need utility generation.

## Semantic Token Naming

Name tokens by purpose rather than visual property. `--color-primary` enables theme switching without utility class changes; `--color-blue-500` embeds a specific value that requires refactoring when palettes change.

| Pattern | Example | Rationale |
|---------|---------|-----------|
| Semantic names | `--color-primary`, `--color-destructive` | Theme-flexible |
| Purpose-driven | `--color-card`, `--color-muted` | Context-clear |
| Scale-based | `--spacing-lg`, `--radius-sm` | Consistent sizing |
| Foreground pairing | `--color-card` + `--color-card-foreground` | Readable contrast |

Pair each surface token with a foreground counterpart: `--color-card` gets `--color-card-foreground`, `--color-muted` gets `--color-muted-foreground`. This pairing guarantees readable contrast across themes.

The full token-layer architecture these naming rules sit on top of -- static `@theme` generation, bridge `@theme inline` exposure, and the `@layer theme` runtime swap -- is specified in the shadcn-side companion doc (the shadcn profile's *Design Token Architecture* reference (`profiles/shadcn/packages/components/reference/design_token_architecture.md`), shadcn tier); adopt it only when the application also uses shadcn components.

## Utility-First Approach

Apply Tailwind utilities directly in JSX. Use built-in variants (`hover:`, `focus:`, `group-*`, `not-*`) for state management. Dynamic values with numeric suffixes (`grid-cols-15`, `mt-17`) work without configuration extension in v4.

Use static class strings with composition utilities (`clsx`, `cn`). Tailwind's class detection scans source files as plain text -- dynamic template expressions like `` bg-${color}-600 `` produce missing styles in production builds.

```typescript
// Correct: static, detectable
const classes = clsx("bg-primary", isActive && "ring-2 ring-primary");

// Incorrect: dynamic, undetectable
const classes = `bg-${color}-600`;
```

## The cn() Utility

The `cn()` function combines `clsx` for conditional class composition and `tailwind-merge` for conflict resolution. Use it in every component that accepts a `className` prop.

```typescript
import { clsx, type ClassValue } from "clsx";
import { twMerge } from "tailwind-merge";

export function cn(...inputs: ClassValue[]) {
  return twMerge(clsx(inputs));
}
```

Apply with CVA: `cn(variants({ variant, size }), className)`. Base variant classes apply first; the consumer's `className` overrides conflicting utilities while non-conflicting properties remain intact.

## Container Queries as Default

Container queries are built into Tailwind v4 core. Use `@container` on parent elements and `@md:`, `@lg:` variants on children for component-level responsiveness. Reserve viewport-based utilities (`md:`, `lg:`) for page-level layouts only.

```tsx
<div className="@container">
  <div className="flex flex-col @md:flex-row gap-4">
    {/* Layout adapts to container width, not viewport */}
  </div>
</div>
```

Named containers (`@container/sidebar`) prevent unintended responsive behavior in nested layouts.

## CVA for Variant Management

Class Variance Authority (CVA) manages multi-dimensional component variants. Define variant schemas mapping props to complete Tailwind class strings, eliminating conditional className logic.

```typescript
const buttonVariants = cva(
  "inline-flex items-center justify-center rounded-md font-medium",
  {
    variants: {
      variant: {
        default: "bg-primary text-primary-foreground",
        outline: "border border-input bg-background",
      },
      size: {
        default: "h-10 px-4 py-2",
        sm: "h-9 px-3",
      },
    },
  }
);
```

Components with multiple variant dimensions (size, color, state) benefit from CVA. Components with a single variant can use direct conditional classes. Extract type-safe props with `VariantProps<typeof buttonVariants>`.

## Thresholds

| Metric | Pass | Warn | Fail |
|--------|------|------|------|
| Custom CSS lines (excluding `@theme`) | <50 | 50-150 | >150 |
| `@apply` usage instances | 0-3 | 3-10 | >10 |
| Component variants without CVA | 0 | 1-2 | >2 |
| Arbitrary value usage per component | 0-1 | 2-3 | >3 |

## Dark Mode and Theming

Use bridge variables for multi-theme support: declare public API tokens with `@theme inline`, then swap underlying variables in `@layer theme` under theme selectors.

```css
@theme inline {
  --color-app-bg: var(--app-bg);
}

@layer theme {
  :root { --app-bg: var(--color-gray-50); }
  .dark { --app-bg: var(--color-slate-950); }
}
```

This separation keeps static design tokens distinct from runtime theme switching, enabling a single utility surface across multiple themes.

## CSS Module Fallback

When Tailwind utilities cannot express a style (complex animations, pseudo-element content, vendor-specific properties), use a co-located CSS module. Import and compose with cn():

```typescript
import styles from "./component-name.module.css";

<div className={cn(componentVariants({ variant }), styles.animatedBorder, className)} />
```

CSS modules automatically scope class names, preventing conflicts between components. Reserve them for the 5-10% of styles that require capabilities beyond Tailwind utilities.

## Print Integration

Add a print bridge in global CSS using `@media print` rules. Define semantic hooks (`.no-print`, `.only-print`) for visibility control. Use `@page` at-rules for margins, orientation, and page size -- properties Tailwind does not cover. Force light theme colors and expand all collapsible content in print to ensure complete output.

## Related

- [Component Composition](component_composition.md)
