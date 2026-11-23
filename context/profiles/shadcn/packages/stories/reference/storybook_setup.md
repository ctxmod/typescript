# Storybook Setup

Storybook 10 with the React + Vite framework for a shadcn/ui component library: installation, `.storybook/` configuration, Tailwind v4 styles, dark-mode switching, accessibility checks, stories as Vitest tests, and API mocking.

Storybook is the component workshop for the library: every component renders
in isolation, with controls, in both themes, and — through the Vitest addon —
every story is also an executed smoke test with accessibility checks in a real
browser. Use the React + Vite framework (`@storybook/react-vite`), which
reuses the project's own Vite configuration.

## Why Storybook

Storybook is the default because it is the actively maintained workshop that
covers the library's whole loop in one tool:

| Concern | Storybook 10.x |
|---|---|
| Maintenance | Monthly minor releases (10.6.x current, 11.0 in alpha) |
| Toolchain | Vite 5–8 and React 16.8–19 supported by `@storybook/react-vite`; ESM-only since 10.0 |
| Testing | `@storybook/addon-vitest` turns stories into Vitest browser-mode tests (Playwright Chromium): render smoke test per story plus `play`-function assertions |
| Accessibility | `@storybook/addon-a11y` runs axe checks in the UI and in the same test run |
| Typing | CSF 3 with `Meta`/`StoryObj`; CSF Factories (`preview.meta` / `meta.story`) in Preview, expected default in Storybook 11 |
| Agents | Storybook MCP (10.3+) exposes components, stories, and tests to coding agents |

**Alternative — Ladle.** Ladle (`@ladle/react`) is a lighter, React-only,
Vite-based story previewer with near-instant startup and a zero-config
default. Its latest release is 5.1.1 (November 2025); it bundles its own
Vite 6 rather than the project's Vite 8, and it has no test-runner or
accessibility integration. It remains acceptable for a small preview-only
surface that already uses it, recorded as such — not for a component library
whose stories double as tests.

## Installation

```bash
npm create storybook@latest
```

The initializer detects React + Vite, installs `storybook` and
`@storybook/react-vite`, writes `.storybook/main.ts` and
`.storybook/preview.ts`, and adds the `storybook` (dev server, port 6006) and
`build-storybook` (static build) scripts. Add the addons the library needs:

```bash
npx storybook add @storybook/addon-a11y
npx storybook add @storybook/addon-themes
npx storybook add @storybook/addon-vitest
```

Storybook 10 is ESM-only and needs a Node.js line with `require(esm)` support
(every current LTS line qualifies).

## Configuration

```typescript
// .storybook/main.ts
import type { StorybookConfig } from "@storybook/react-vite";

const config: StorybookConfig = {
  framework: "@storybook/react-vite",
  stories: ["../src/components/**/*.stories.@(ts|tsx)"],
  addons: [
    "@storybook/addon-a11y",
    "@storybook/addon-themes",
    "@storybook/addon-vitest",
  ],
};

export default config;
```

The `stories` glob discovers story files co-located with components; narrow
it to the component tree rather than all of `src/`. The React + Vite framework
merges the project's `vite.config.ts`, so the Tailwind Vite plugin and path
aliases apply to stories without being repeated here.

## Tailwind v4 and Dark Mode

```typescript
// .storybook/preview.ts
import type { Preview } from "@storybook/react-vite";
import { withThemeByClassName } from "@storybook/addon-themes";
import "../src/styles/globals.css";

const preview: Preview = {
  decorators: [
    withThemeByClassName({
      themes: { light: "light", dark: "dark" },
      defaultTheme: "light",
    }),
  ],
  parameters: {
    a11y: { test: "error" },
  },
};

export default preview;
```

Importing the application's main stylesheet gives every story the same
`@import "tailwindcss"`, `@theme` tokens, and base layer as the application;
Tailwind's automatic source detection scans the component tree, so a
separate stylesheet is needed only when stories live outside it (add an
`@source` line for that path). `withThemeByClassName` toggles the `.dark`
class on the preview root from the toolbar, matching the class-based
`@custom-variant dark` the token architecture uses. Project-wide providers
(i18n, query client) are added as further global decorators.

`a11y.test: "error"` makes accessibility violations fail the story's test
run; lower it to `"todo"` only per story, with a reason, while a known issue
is open.

## Stories as Tests

`@storybook/addon-vitest` adds a Vitest project that runs every story in
Vitest browser mode on Playwright Chromium: each story is rendered (a smoke
test), its `play` function's assertions are checked, and accessibility checks
run alongside. It requires a Vite-based Storybook framework and Vitest 3+
(current addon peers: Vitest 4 or 5 with `@vitest/browser-playwright`). Run it
from the Storybook UI's testing widget or the CLI (`vitest --project
storybook`). Whether and when the run is part of the repository's checks is
the repository's test policy.

## TypeScript Configuration

Include `.storybook/` in the TypeScript project so the preview and main files
are type-checked:

```json
{
  "include": ["src", ".storybook"]
}
```

## API Mocking

For components that fetch data, use `msw-storybook-addon` (MSW 2+): call
`initialize()` and register `mswLoader` in `.storybook/preview.ts`, then
attach handlers per story under `parameters.msw.handlers`. For module-level
dependencies, Storybook 10's `sb.mock` automocking (modeled on `vi.mock`)
replaces a module in stories without hand-written decorators.

- [Story Patterns](story_patterns.md)
- the react_router profile's *Component Directory Structure* reference (`profiles/react_router/packages/components/reference/component_directory_structure.md`)

## Related

- the react_router profile's *Testability* reference (`profiles/react_router/packages/testing/reference/testability.md`)

## Sources

- https://storybook.js.org/blog/storybook-10/ — Storybook 10: ESM-only, Node.js `require(esm)` floor, CSF Factories in Preview, `sb.mock`, Vitest 4 support (2025-10-28; accessed 2026-11-22)
- https://storybook.js.org/blog/storybook-10-3/ — 10.3: Storybook MCP for React, accessibility upgrades, Vite 8 compatibility (accessed 2026-11-22)
- https://storybook.js.org/docs/writing-tests/integrations/vitest-addon — `@storybook/addon-vitest`: stories as Vitest browser-mode tests on Playwright, a11y alongside, Vite-based framework and Vitest 3+ required (docs v10.6; accessed 2026-11-22)
- https://storybook.js.org/recipes/tailwindcss — `@storybook/addon-themes` `withThemeByClassName` (accessed 2026-11-22)
- https://www.npmjs.com/package/@storybook/react-vite — 10.6.8; peers Vite ^5–^8, React ^16.8–^19 (accessed 2026-11-22)
- https://www.npmjs.com/package/@storybook/addon-vitest — 10.6.8; peers Vitest ^4/^5, `@vitest/browser-playwright` (accessed 2026-11-22)
- https://www.npmjs.com/package/msw-storybook-addon — 3.0.3; peers MSW 2+, Storybook 9–11 (accessed 2026-11-22)
- https://www.npmjs.com/package/@ladle/react — Ladle 5.1.1, published 2025-11-04, depends on Vite ^6, peers React ≥18 (accessed 2026-11-22)
