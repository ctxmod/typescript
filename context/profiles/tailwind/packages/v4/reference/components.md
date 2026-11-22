# Tailwind v4 Component Styling

Tailwind CSS v4 component styling with @utility, @layer, and @variant.

## Duplication Management Hierarchy

The officially recommended approach, in order of preference:

1. Loops and multi-cursor editing
2. Framework components / template partials
3. Custom CSS only as last resort

Extract React/Vue/Svelte components over using `@apply` or CSS component classes.

## `@utility` Directive

**Use `@utility` for variant-aware custom classes**

Classes defined with `@utility` automatically work with `hover:`, `focus:`, `dark:`, `lg:`, and other Tailwind variants:

```css
@utility content-auto {
  content-visibility: auto;
}
```

Functional `@utility` definitions support `--value()` for theme/bare/arbitrary resolution and `--modifier()` for slash-notation modifiers. Multiple `--value()` declarations coexist; failing resolutions are silently omitted.

## `@layer components`

**Use `@layer components` for overridable component classes**

The `components` layer has lower specificity than `utilities`, so any Tailwind utility class on the element will override it:

```css
@layer components {
  .card {
    background: var(--color-background);
    border-radius: var(--radius-lg);
    padding: var(--spacing-4);
  }
}
```

Classes in `@layer components` use plain CSS properties referencing theme variables — do not use `@apply` within them.

## `@variant` Directive

**Use `@variant` for Tailwind variants in custom CSS**

Instead of `@apply hover:bg-black`:

```css
.my-element {
  background: white;
  @variant dark { background: black; }
  @variant hover, focus { background: gray; }
  @variant hover:focus { background: black; }
}
```

## `@apply` Deprecation

`@apply` still works but is no longer documented in v4. Prefer:
- `@utility` for classes needing variant support
- `@layer components` for overridable component classes
- `@variant` for applying variants in custom CSS

## `@apply` in Bundled Stylesheets

When using `@apply` in separately-bundled stylesheets (Vue SFC, Svelte, CSS Modules), use `@reference` to import theme context:

```css
@reference "../../app.css";
```

Without it, `@apply` fails against custom class names.

## Container Queries

**First-class container queries in v4**

Use `@container` on parent elements. Children use `@sm:`, `@md:` (with `@` prefix) for container-query variants:

```css
.parent { @container; }
.child { @xl:grid-cols-3 @sm:grid-cols-1; }
```

Named containers: `@container/sidebar` on parent, `@xl/sidebar:` on children. `@container-size` exposes both width and height as CSS variables.

**Prefer container queries for components, media queries for page layout**

Components appearing in different contexts (sidebar, main, modals) should respond to container size. Page-level chrome uses media queries.

## Thresholds

| Metric | Pass | Warn | Fail |
|--------|------|------|------|
| `@apply` in new v4 code | 0 | 1-3 | >3 |
| Custom classes without variant support needing it | 0 | 0 | >0 |

- KISS (engineering module, `principles` profile: `design/reference/kiss.md`)
- DRY (engineering module, `principles` profile: `design/reference/dry.md`)
