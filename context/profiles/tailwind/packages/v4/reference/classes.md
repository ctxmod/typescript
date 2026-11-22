# Tailwind v4 Class Changes

Tailwind CSS v4 class naming changes, removed utilities, and logical utilities for bidirectional layout.

## Syntax Changes

**`!important` modifier moved to end of class name**

```css
/* v3 (deprecated) */
!flex
hover:!bg-red-600

/* v4 */
flex!
hover:bg-red-600!
```

**CSS variable arbitrary values use parentheses**

```css
/* v3 (deprecated) */
bg-[--brand-color]

/* v4 */
bg-(--brand-color)
```

The parenthetical form automatically wraps in `var()`.

**Comma-separated arbitrary values use underscores**

```css
/* v3: commas became spaces */
grid-cols-[max-content,auto] /* → grid-template-columns: max-content auto */

/* v4: underscores become spaces, commas are literal */
grid-cols-[max-content_auto] /* → grid-template-columns: max-content auto */
```

**Stacked variants apply left-to-right**

```css
/* v3: innermost applied first */
first:*:pt-0

/* v4: leftmost is outermost */
*:first:pt-0
```

## Removed Utilities

**Opacity utilities removed — use slash syntax**

All `*-opacity-*` utilities are removed (`bg-opacity-*`, `text-opacity-*`, `border-opacity-*`, `ring-opacity-*`). Use slash syntax:

```css
/* v3 (removed) */
bg-black bg-opacity-50

/* v4 */
bg-black/50
```

**Flex shorthand utilities removed**

Use `shrink-*` and `grow-*` instead of `flex-shrink-*` and `flex-grow-*`.

**Scale shifts**

Shadow/blur/radius scales shifted down by one name:

| Old Name | New Name |
|----------|----------|
| `shadow` | `shadow-sm` |
| `shadow-sm` | `shadow-xs` |
| `blur` | `blur-sm` |
| `blur-sm` | `blur-xs` |
| `rounded` | `rounded-sm` |
| `rounded-sm` | `rounded-xs` |

The same pattern applies to `backdrop-blur` and `drop-shadow`.

**Ring defaults changed**

`ring` now defaults to 1px width and `currentColor`. To get the v3 look: `ring-3 ring-blue-500`.

**Outline behavior changed**

`outline-none` now means `outline-style: none` (invisible to forced-colors tools). Use `outline-hidden` for the old v3 behavior (invisible outline, forced-colors safe).

## Transform Properties

**Individual transform properties replace the shorthand**

`rotate`, `scale`, `translate` are now independent. `transform-none` does NOT reset them. Use specific resets: `scale-none`, `rotate-none`, `translate-none`.

Transitions must use specific names: `transition-[opacity,scale]` not `transition-[opacity,transform]`.

## Hover Behavior

**`hover:` only fires on hover-capable devices**

Touch devices will no longer trigger hover on tap. Restore v3 behavior: `@custom-variant hover (&:hover)`.

## Border Color Default

**`bg-*` and `divide-*` default border color changed to `currentColor`**

This silently changes appearance. Add a base layer override to restore v3 behavior if needed.

## Logical Utilities for Bidirectional Layout

**Write layout in logical properties so a direction change mirrors the layout**

Layout that runs along the inline axis is written with logical utilities, not physical left/right ones. Under a right-to-left document direction the logical utilities flip automatically and the layout mirrors without any per-side override; physical utilities pin the layout to one direction and break under mirroring.

| Concern | Physical (direction-pinned) | Logical (preferred) |
|---------|------------------------------|---------------------|
| Inline-start margin | `ml-*` | `ms-*` |
| Inline-end margin | `mr-*` | `me-*` |
| Inline-start padding | `pl-*` | `ps-*` |
| Inline-end padding | `pr-*` | `pe-*` |
| Inline-start border | `border-l-*` | `border-s-*` |
| Inline-end border | `border-r-*` | `border-e-*` |
| Inset from inline edge | `left-*` / `right-*` | `start-*` / `end-*` |
| Text alignment | `text-left` / `text-right` | `text-start` / `text-end` |
| Corner rounding | `rounded-l-*` / `rounded-r-*` | `rounded-s-*` / `rounded-e-*` |

Physical utilities remain correct where direction-independence is the point — an offset that must stay on the same visual side regardless of writing direction is a physical concern by definition.

This is the styling-lens half of bidirectional support. Document-level direction handling, icon flipping, and bidirectional text isolation are owned by the es locale-strategies reference ([Locale Strategies](../../../../core/packages/localization/reference/locale_strategies.md)), which this styling guidance implements and does not restate.

## Thresholds

| Metric | Pass | Warn | Fail |
|--------|------|------|------|
| `!` prefix syntax | 0 | 0 | >0 |
| `*-opacity-*` utilities | 0 | 0 | >0 |
| `flex-shrink-*`/`flex-grow-*` | 0 | 0 | >0 |
| Old shadow scale names (`shadow`, `shadow-sm`) | 0 | 1-3 | >3 |

- [Locale Strategies](../../../../core/packages/localization/reference/locale_strategies.md)
