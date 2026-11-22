# Tailwind v4 Class Context

Tailwind CSS v4 class naming changes, opacity modifiers, scale renames, and removed utilities.

The `!important` modifier moved to the end: `flex!` not `!flex`. CSS variable arbitrary values use parentheses: `bg-(--brand-color)` not `bg-[--brand-color]`. Comma-separated arbitrary values use underscores: `grid-cols-[max-content_auto]`. Stacked variants apply left-to-right. `*-opacity-*` utilities are removed — use slash syntax: `bg-black/50`. Shadow/blur/radius scales shifted down by one name (`shadow` is now `shadow-sm`). `ring` defaults to 1px and `currentColor`. Use `shrink-*`/`grow-*` not `flex-shrink-*`/`flex-grow-*`.

- [Tailwind v4 Class Changes](../reference/classes.md)
