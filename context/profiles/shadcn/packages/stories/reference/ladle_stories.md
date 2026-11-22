# Ladle Stories

Story file patterns, controls, decorators, co-location, and MSW integration.

Stories preview components in isolation with interactive controls. Each story file exports named `Story` functions that appear as individual entries in the Ladle navigation sidebar. Co-locate story files alongside their components for discoverability.

## Story File Structure

A story file exports one or more `Story` functions and an optional default export for file-level configuration.

```tsx
import type { Story, StoryDefault } from "@ladle/react";
import { Button } from "./button";

export default {
  title: "Interaction/Button",
} satisfies StoryDefault;

export const Primary: Story = () => (
  <Button variant="primary">Action</Button>
);

export const Secondary: Story = () => (
  <Button variant="secondary">Cancel</Button>
);
```

Named exports become individual stories. The `title` in the default export controls navigation hierarchy -- use the component's category path for consistent grouping: `"Interaction/Button"`, `"Form/Select"`, `"Display/Card"`. When `title` is omitted, Ladle derives navigation from the filename. CamelCase splits into spaces, hyphens become spaces, and double-hyphens (`--`) create nesting levels.

## Controls with Args

Typed args produce interactive property controls in the Ladle sidebar.

```tsx
const ButtonStory: Story<{
  variant: "primary" | "secondary" | "ghost" | "destructive";
  size: "sm" | "md" | "lg";
  disabled: boolean;
  children: string;
}> = ({ variant, size, disabled, children }) => (
  <Button variant={variant} size={size} disabled={disabled}>
    {children}
  </Button>
);

ButtonStory.args = {
  variant: "primary",
  size: "md",
  disabled: false,
  children: "Click me",
};

ButtonStory.argTypes = {
  variant: { control: { type: "select" } },
  size: { control: { type: "radio" } },
  disabled: { control: { type: "boolean" } },
};
```

Ladle auto-detects TypeScript union types and generates appropriate controls. Explicit `argTypes` override detection when finer control is needed -- radio instead of select, custom labels, or check groups for arrays. Args cascade through three levels: global defaults (`.ladle/components.tsx`) < file-level args (default export) < per-story args. Per-story values take precedence.

## Decorators

File-level decorators wrap all stories in a file with shared context.

```tsx
export default {
  decorators: [
    (Story) => (
      <div className="flex gap-4 items-center">
        <Story />
      </div>
    ),
  ],
} satisfies StoryDefault;
```

Common decorator uses:
- **Layout wrappers** -- flexbox containers, padding, max-width constraints
- **Provider injection** -- theme, i18n, or auth context wrapping
- **Mock data setup** -- shared fixture state accessible to all stories

Ladle applies decorators file-wide. Per-story decorators are unsupported. When stories require different decorator configurations, split them into separate story files.

## Co-location Pattern

Story files live in the same directory as their component source.

```
src/components/common/interaction/button/
  button.tsx
  button.test.tsx
  button.stories.tsx
  button.module.css    (optional)
```

Ladle discovers stories via the glob configured in `.ladle/config.mjs`. The default pattern `src/**/*.stories.{ts,tsx}` finds co-located files automatically. Adjusting the glob to `src/components/**/*.stories.tsx` restricts discovery to the component tree.

the react_router profile's *Component Directory Structure* reference (`profiles/react_router/packages/components/reference/component_directory_structure.md`)

## MSW Handlers

Stories that depend on API data use Mock Service Worker handlers attached per-story. Enable MSW in `.ladle/config.mjs` before using handlers.

```tsx
import { msw } from "@ladle/react";

export const WithData: Story = () => <UserList />;

WithData.msw = [
  msw.http.get("/api/users", () =>
    msw.HttpResponse.json([
      { id: 1, name: "Alice" },
      { id: 2, name: "Bob" },
    ])
  ),
];
```

File-level handlers defined in the default export apply to all stories in that file. Per-story handlers override file-level handlers for the same endpoints. Import `msw` from `@ladle/react` directly -- Ladle bundles MSW v2 and manages the service worker lifecycle.

## Variant Coverage Strategy

For components built with CVA variant definitions, stories should cover the variant matrix systematically.

| Coverage | Stories |
|----------|---------|
| Default state | One story rendering the component with no explicit props |
| Each variant value | One named story per variant (primary, secondary, ghost, destructive) |
| Size range | One composite story displaying all sizes side-by-side |
| Interactive states | Disabled, loading, and error variants |
| Composition | Component combined with siblings (icon alongside label, badge inside card) |

Pair one interactive story with full args for ad-hoc exploration with named stories for documentation. Named stories serve as visual regression anchors -- each captures a single, predictable configuration.

## Anti-Patterns

- **Stories without args** -- prevents interactive exploration and limits the story to a static preview
- **One story per file** -- loses the grouping context that related variants provide; multiple exports per file reduce navigation noise
- **Hardcoded mock data inline** -- extract shared fixtures to a `__fixtures__` directory or a `*.fixtures.ts` co-located file for reuse across stories and tests
- **Skipping dark mode verification** -- verify both light and dark themes through the GlobalProvider theme toggle; visual regressions frequently hide in the alternate theme

- [Ladle Setup](ladle_setup.md)

## Related

- the react_router profile's *Component Testing* reference (`profiles/react_router/packages/testing/reference/component_testing.md`)
