# Component Styling

Workflow for styling React components with Tailwind CSS v4 design tokens, CVA variants, cn() utility, and container queries.

Procedure for styling React components using Tailwind CSS v4's CSS-first architecture. This workflow ensures consistent token usage, component-level responsiveness, and minimal custom CSS.

## Prerequisites

- Design tokens defined in the project's `@theme` block
- Tailwind v4 configured with `@tailwindcss/vite` plugin
- Component planning completed (composition pattern selected)
- cn() utility available at project utility path

## Styling Workflow

### 1. Check Existing Design Tokens

Before creating custom values, verify the `@theme` block for existing tokens. Tailwind v4 uses CSS-first configuration where all tokens live in a single stylesheet.

```css
@theme {
  --color-primary: oklch(0.53 0.12 118.34);
  --font-display: "Satoshi", sans-serif;
  --spacing-section: 4rem;
}
```

Tokens declared in `@theme` automatically generate utilities: `--color-primary` produces `bg-primary`, `text-primary`, `border-primary`. Reference semantic token names (`--color-primary`) rather than literal values (`--color-blue-500`). Semantic names survive palette changes without class refactoring.

Common semantic token mappings:

| Use Case | Token | Utility |
|----------|-------|---------|
| Page background | `--color-background` | `bg-background` |
| Card surface | `--color-card` | `bg-card` |
| Primary action | `--color-primary` | `bg-primary` |
| Muted surface | `--color-muted` | `bg-muted` |
| Text (primary) | `--color-foreground` | `text-foreground` |
| Text (secondary) | `--color-muted-foreground` | `text-muted-foreground` |
| Borders | `--color-border` | `border-border` |
| Focus ring | `--color-ring` | `ring-ring` |

- [Styling Philosophy](styling_philosophy.md)

Deeper token-layer mechanics -- the full static/bridge/theme token pipeline and shadcn-specific theming -- live in the shadcn-side companion doc (the shadcn profile's *Design Token Architecture* reference (`profiles/shadcn/packages/components/reference/design_token_architecture.md`), shadcn tier); consult it only when the application also uses shadcn components.

### 2. Apply Utility-First Classes

Write component markup with Tailwind utilities before considering custom CSS. Use built-in variants for interactive states:

| State | Variant | Example |
|-------|---------|---------|
| Hover | `hover:` | `hover:bg-primary/90` |
| Focus | `focus-visible:` | `focus-visible:ring-2` |
| Active | `active:` | `active:scale-95` |
| Disabled | `disabled:` | `disabled:opacity-50` |
| Group hover | `group-hover:` | `group-hover:translate-x-1` |
| Peer state | `peer-checked:` | `peer-checked:bg-primary` |

Chain variants for compound states: `hover:focus-visible:ring-primary`. Use `not-*` for negation: `not-disabled:hover:bg-primary`.

### 3. Define Component Variants with CVA

When a component has multiple visual dimensions (size, color, state), use Class Variance Authority to manage variant combinations.

```typescript
import { cva, type VariantProps } from "class-variance-authority";

const buttonVariants = cva(
  "inline-flex items-center justify-center rounded-md font-medium transition-colors",
  {
    variants: {
      variant: {
        default: "bg-primary text-primary-foreground hover:bg-primary/90",
        outline: "border border-input bg-background hover:bg-accent",
        ghost: "hover:bg-accent hover:text-accent-foreground",
      },
      size: {
        default: "h-10 px-4 py-2",
        sm: "h-9 px-3 text-sm",
        lg: "h-11 px-8 text-lg",
      },
    },
    defaultVariants: {
      variant: "default",
      size: "default",
    },
  }
);

type ButtonProps = VariantProps<typeof buttonVariants> & {
  children: React.ReactNode;
};
```

CVA is warranted when a component has 2+ variant dimensions. For single-dimension variants, a simple ternary or `clsx` call suffices. Use `compoundVariants` sparingly for combinations needing special treatment.

### 4. Integrate cn() Utility

Every component accepting a `className` prop merges it with cn():

```typescript
export function Button({ className, variant, size, ...props }: ButtonProps) {
  return (
    <button
      className={cn(buttonVariants({ variant, size }), className)}
      {...props}
    />
  );
}
```

cn() resolves Tailwind conflicts so that a consumer's `className` overrides component defaults for conflicting utilities. The component's base classes remain intact for non-conflicting properties.

### 5. Implement Container Queries

Use container queries as the default responsive pattern for reusable components. Container queries respond to the component's container width, not the viewport.

```tsx
<div className="@container">
  <div className="flex flex-col @md:flex-row gap-4">
    <div className="@lg:w-1/3">Sidebar</div>
    <div className="@lg:w-2/3">Content</div>
  </div>
</div>
```

For nested layouts, use named containers (`@container/sidebar`) to control which ancestor a component responds to. Reserve viewport-based utilities (`md:`, `lg:`, `xl:`) for page-level layouts only.

### 6. Handle Dark Mode

Use the project's dark mode mechanism (typically a `.dark` class or `data-theme` attribute on the root element). Components styled with semantic tokens adapt automatically when the theme switches.

For properties needing theme-specific adjustments beyond token switching, use Tailwind's `dark:` variant. Verify dark mode by toggling the theme in the component preview tool.

### 7. Handle CSS Module Fallback

When Tailwind utilities cannot express a style (complex animations, pseudo-element content, vendor-specific properties), use a co-located CSS module:

```css
/* component-name.module.css */
.animatedBorder {
  animation: borderPulse 2s ease-in-out infinite;
}

@keyframes borderPulse {
  0%, 100% { border-color: var(--color-border); }
  50% { border-color: var(--color-primary); }
}
```

Import and apply with cn():

```typescript
import styles from "./component-name.module.css";

<div className={cn(buttonVariants({ variant }), styles.animatedBorder, className)} />
```

CSS modules automatically scope class names, preventing conflicts between components.

## Threshold Guidance

| Metric | Acceptable | Review | Refactor |
|--------|-----------|--------|----------|
| Custom CSS lines (excluding @theme) | <50 | 50-150 | >150 |
| `@apply` usage instances | 0-3 | 3-10 | >10 |
| Component variants without CVA | 0-1 | 2-3 | >3 |
| Arbitrary bracket values per component | 0-2 | 3-5 | >5 |

## Styling Checklist

- [ ] Tokens reference semantic names from `@theme`
- [ ] Utility classes applied before any custom CSS
- [ ] CVA applied for multi-dimensional variants
- [ ] cn() merges component classes with consumer className
- [ ] Container queries used for component responsiveness
- [ ] Viewport utilities reserved for page layouts
- [ ] Dark mode tokens verified
- [ ] CSS module used only for styles Tailwind cannot express
- [ ] Static class strings used (no dynamic interpolation like `` bg-${color}-500 ``)

## Related

- [Component Planning](component_planning.md)
