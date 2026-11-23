# Tailwind v4 Migration

Tailwind CSS v3 to v4 migration pitfalls.

## Plugin and Build Changes

**PostCSS plugin moved**

`@tailwindcss/postcss` replaces `tailwindcss` and `autoprefixer` (no longer needed separately). CLI moved to `@tailwindcss/cli`.

**Browser requirements**

v4 requires Safari 16.4+, Chrome 111+, Firefox 128+. It uses modern CSS features (`@property`, `color-mix()`). No fallback mode for older browsers.

**Not compatible with Sass, Less, or Stylus**

Tailwind CSS v4 replaces the need for a CSS preprocessor. Using alongside preprocessors is not supported.

## Breaking API Changes

**`theme()` dot notation gives way to CSS variables**

The `theme()` function is deprecated; reference theme variables directly. Where a variable cannot be used — media queries — `theme()` takes the variable name instead of dot notation:

```css
/* v3 */
background-color: theme(colors.red.500);
/* v4 */
background-color: var(--color-red-500);

/* Media queries */
/* v3 */ theme(screens.xl)
/* v4 */ theme(--breakpoint-xl)
```

**`resolveConfig` JS export removed**

Access theme values via `getComputedStyle` in JavaScript:
```javascript
let styles = getComputedStyle(document.documentElement);
let val = styles.getPropertyValue("--shadow-xl");
```

**Hover behavior change**

`hover:` variant now only applies on devices supporting hover (`@media (hover: hover)`). Touch devices will not trigger hover on tap. Restore v3 behavior:
```css
@custom-variant hover (&:hover);
```

**Transform property changes**

Individual transform properties (`rotate`, `scale`, `translate`) are now used instead of the `transform` shorthand. `transform-none` does NOT reset them. Transitions must use specific property names:
```css
/* v3 */
transition-[opacity,transform]
/* v4 */
transition-[opacity,scale]
```

**Border color default changed**

`border-*` and `divide-*` utilities default to `currentColor` instead of `gray-200`. This silently changes appearance for elements using these utilities without explicit colors; give each a color, or restore v3 behavior with a `@layer base` rule setting `border-color: var(--color-gray-200, currentColor)` on `*, ::after, ::before, ::backdrop, ::file-selector-button`.

**`@apply` in bundled stylesheets requires `@reference`**

For Vue SFC, Svelte, or CSS Modules:
```css
@reference "../../app.css";
```

Without it, `@apply` fails against custom class names.

## Migration Checklist

- [ ] Run `npx @tailwindcss/upgrade` (Node.js 20+) on a branch and review the diff
- [ ] Replace `@tailwind` directives with `@import "tailwindcss"`
- [ ] Move `tailwind.config.js` tokens to `@theme` blocks in CSS (or load it with `@config` temporarily)
- [ ] Load JS plugins with `@plugin`
- [ ] Update `@tailwindcss/postcss` in PostCSS config
- [ ] Remove `postcss-import` and `autoprefixer` (handled automatically)
- [ ] Fix `!` modifier placement (prefix → suffix)
- [ ] Fix CSS variable arbitrary values (`[]` → `()`)
- [ ] Fix comma-separated arbitrary values (commas → underscores)
- [ ] Replace `*-opacity-*` with slash syntax
- [ ] Update shadow/blur/radius scale names
- [ ] Replace `theme()` with CSS variable references (`theme(--breakpoint-*)` only where variables cannot be used)
- [ ] Add `@custom-variant dark` if using class-based dark mode
- [ ] Fix `hover:` behavior for touch devices if needed
- [ ] Add `@reference` to bundled stylesheets using `@apply`
- [ ] Remove Sass/Less/Stylus if present

## Thresholds

| Metric | Pass | Warn | Fail |
|--------|------|------|------|
| v3 syntax patterns remaining | 0 | 1-3 | >3 |
| `tailwind.config.js` present | 0 | 0 | >0 |

## Sources

- https://tailwindcss.com/docs/upgrade-guide — browser floor, `border-*`/`divide-*` default `currentColor`, `theme(--breakpoint-xl)` in media queries, JS config not auto-detected, `npx @tailwindcss/upgrade` on Node.js 20+ (accessed 2026-11-22)
- https://tailwindcss.com/docs/functions-and-directives — `@plugin` for JS plugins, `@config` for legacy JS config, `@source`, `@reference`, `@variant`, `@utility`, `--alpha()`, `--spacing()`, `theme()` deprecated (accessed 2026-11-22)
