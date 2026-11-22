# Ladle Setup

Ladle project configuration, Tailwind v4 integration, GlobalProvider, dark mode sync.

Ladle provides fast component preview using Vite. Setup requires a global provider for theming, a Tailwind CSS entry point with explicit source directives, and TypeScript inclusion of the `.ladle/` directory.

## Why Ladle

Ladle uses Vite directly, producing 10-70x faster cold starts than Storybook. Builds stay under 1MB versus Storybook's 7MB+. Zero-configuration defaults handle most projects without a config file. The React-only focus eliminates multi-framework overhead.

| Aspect | Ladle | Storybook |
|--------|-------|-----------|
| Cold start | <1s | 5-30s |
| Build size | <1MB | 7MB+ |
| Configuration | Zero-config default | Extensive |
| Framework support | React only | Multi-framework |
| Addon ecosystem | Minimal | Extensive |

Choose Ladle when component preview speed and simplicity matter more than documentation features or addon ecosystem breadth. Storybook remains appropriate for multi-framework requirements and projects that depend on a wide addon catalog.

## Installation

```bash
npm install -D @ladle/react
```

Add two scripts to `package.json`:

```json
{
  "scripts": {
    "stories": "ladle serve",
    "stories:build": "ladle build"
  }
}
```

`ladle serve` starts the development server with hot module replacement. `ladle build` produces a static preview build suitable for deployment or CI artifacts.

## Configuration

Create `.ladle/config.mjs` to override defaults:

```javascript
/** @type {import('@ladle/react').UserConfig} */
export default {
  stories: "src/components/**/*.stories.{tsx,ts}",
  defaultStory: "",
  addons: {
    msw: { enabled: false },
  },
};
```

The `stories` glob discovers story files co-located with components. Adjust the pattern to match the project's directory structure. Ladle works without this file -- create it only when defaults need overriding.

## Tailwind v4 Integration

Tailwind v4 requires explicit source configuration for Ladle because its automatic content detection does not scan Ladle's generated entry points. Create `.ladle/theme.css`:

```css
@import "tailwindcss";
@source "../src/components/";
@import "../src/styles.css";
```

The `@source` directive tells Tailwind where to scan for utility classes used in stories and components. Without it, Tailwind generates no styles in the Ladle preview. Import the project's main stylesheet after `@source` to inherit design tokens and base styles.

This workaround addresses an open compatibility issue between Ladle and Tailwind v4.0.8+ where styles fail to generate despite working in the main application.

## GlobalProvider

`.ladle/components.tsx` wraps all stories with shared context:

```tsx
import type { GlobalProvider } from "@ladle/react";
import { useEffect } from "react";
import "./theme.css";

export const Provider: GlobalProvider = ({ children, globalState }) => {
  useEffect(() => {
    document.documentElement.classList.toggle(
      "dark",
      globalState.theme === "dark",
    );
  }, [globalState.theme]);

  return <div className="p-4">{children}</div>;
};
```

The GlobalProvider performs three functions: it imports the Tailwind stylesheet so all utilities resolve, syncs Ladle's built-in theme toggle to the document's `.dark` class for dark mode, and wraps stories with consistent padding.

Add project-specific providers (i18n, auth context, theme providers) by nesting them inside the returned JSX. The `globalState` parameter exposes Ladle's addon state, and `storyMeta` provides per-story metadata defined in default exports.

## TypeScript Configuration

Include `.ladle/` in the project's TypeScript configuration to ensure type checking covers the GlobalProvider and any story utility files:

```json
{
  "include": ["src", ".ladle"]
}
```

Story files import types from `@ladle/react` directly. The package ships its own type definitions, so no separate `@types/` installation is needed.

## MSW Integration

Enable Mock Service Worker for stories that need API mocking:

```javascript
// .ladle/config.mjs
export default {
  addons: {
    msw: { enabled: true },
  },
};
```

Ladle handles service worker setup automatically. Stories define per-story handlers using the `msw` export property, with `msw.http` for REST and `msw.graphql` for GraphQL endpoints.

[Ladle Stories](ladle_stories.md)
the react_router profile's *Component Directory Structure* reference (`profiles/react_router/packages/components/reference/component_directory_structure.md`)

## Related

- the react_router profile's *Testability* reference (`profiles/react_router/packages/testing/reference/testability.md`)
