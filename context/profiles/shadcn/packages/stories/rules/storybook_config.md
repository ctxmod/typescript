---
paths:
  - ".storybook/*"
  - ".storybook/*/*"
  - "*/.storybook/*"
  - "*/.storybook/*/*"
  - "*/*/.storybook/*"
  - "*/*/.storybook/*/*"
  - "*/*/*/.storybook/*"
  - "*/*/*/.storybook/*/*"
  - "*/*/*/*/.storybook/*"
  - "*/*/*/*/.storybook/*/*"
---

# Storybook Configuration Context

Storybook setup and configuration principles for `.storybook/` configuration files.

Use the React + Vite framework (`@storybook/react-vite`) so stories reuse the project's Vite configuration; import the application stylesheet in `preview.ts`, switch themes with `withThemeByClassName`, and enable the a11y and Vitest addons so every story is an executed test. Ladle is acceptable only for an existing preview-only surface, recorded as such.

- [Storybook Setup](../reference/storybook_setup.md)
