# Story Development

Workflow for creating Ladle stories with controls, decorators, and variant coverage.

Procedure for creating Ladle stories that cover component variants, interactive controls, and edge cases. Stories serve as living documentation and visual test fixtures.

## Prerequisites

- Component implemented and styled
- Ladle configured with GlobalProvider and Tailwind integration
- Component's CVA variants identified (if applicable)

## Development Workflow

### 1. Create Story File

Create `{component-name}.stories.tsx` co-located with the component source:

```tsx
import type { Story, StoryDefault } from "@ladle/react";
import { ComponentName } from "./component-name";

export default {
  title: "Category/ComponentName",
} satisfies StoryDefault;
```

The title prefix reflects the component's category in the navigation tree: `"Interaction/Button"`, `"Form/Input"`, `"Display/Card"`. Ladle generates its sidebar hierarchy from these title paths. String literals are required -- dynamic strings cause build failures.

### 2. Write Default Story

Create an interactive story with args mapping to all variant dimensions:

```tsx
export const Default: Story<{
  variant: "primary" | "secondary" | "ghost";
  size: "sm" | "md" | "lg";
  disabled: boolean;
}> = ({ variant, size, disabled }) => (
  <ComponentName variant={variant} size={size} disabled={disabled}>
    Example content
  </ComponentName>
);

Default.args = {
  variant: "primary",
  size: "md",
  disabled: false,
};

Default.argTypes = {
  variant: { control: { type: "select" } },
  size: { control: { type: "radio" } },
};
```

This default story provides full interactive exploration through Ladle's Controls panel. Boolean args render as checkboxes automatically. String union args need explicit `control.type` to display as select dropdowns or radio buttons rather than free text.

### 3. Add Named Variant Stories

Create one story per meaningful variant combination for quick visual scanning:

```tsx
export const Primary: Story = () => (
  <ComponentName variant="primary">Primary action</ComponentName>
);

export const Secondary: Story = () => (
  <ComponentName variant="secondary">Secondary action</ComponentName>
);

export const AllSizes: Story = () => (
  <div className="flex items-center gap-2">
    <ComponentName size="sm">Small</ComponentName>
    <ComponentName size="md">Medium</ComponentName>
    <ComponentName size="lg">Large</ComponentName>
  </div>
);
```

Named stories give reviewers a direct reference for each state without toggling controls.

### 4. Cover Edge Cases

Add stories for states that consumers need to verify visually:

| State | Story Purpose |
|-------|---------------|
| Disabled | Visual differentiation when non-interactive |
| Loading | Spinner or skeleton display |
| Empty | Graceful handling of missing content |
| Error | Error state presentation |
| Long content | Overflow and truncation behavior |
| With icon | Composition with icon elements |

Each edge case warrants its own exported story. Group related edge cases (e.g., all error variations) in a single story using a flex layout when the individual states are simple.

### 5. Add Decorators

Decorators wrap all stories in a file with shared layout or provider context. Define them in the default export:

```tsx
export default {
  title: "Form/Select",
  decorators: [
    (Story) => (
      <div className="max-w-sm">
        <Story />
      </div>
    ),
  ],
} satisfies StoryDefault;
```

Common uses: max-width containers for form components, flex layouts for side-by-side comparisons, provider wrappers for context-dependent components. Ladle applies decorators file-wide -- per-story decorators are not supported. For story-specific wrapping, embed the wrapper directly in the story function.

### 6. Configure MSW Handlers

For components that fetch data, attach MSW handlers at the story level. Ladle re-exports MSW utilities from `@ladle/react`:

```tsx
import { msw } from "@ladle/react";

export const WithData: Story = () => <DataList />;

WithData.msw = [
  msw.http.get("/api/items", () =>
    msw.HttpResponse.json({ items: [{ id: 1, name: "Item" }] })
  ),
];
```

File-level handlers apply to all stories via the default export's `msw` property. Per-story handlers on the `msw` property override file-level handlers for that story. Enable MSW in `.ladle/config.mjs` before using handlers:

```javascript
export default {
  addons: { msw: { enabled: true } },
};
```

### 7. Verify in Ladle

Run `npm run stories` (or the project's configured Ladle command) and verify:

- All stories render without console errors
- Controls produce expected visual changes for interactive stories
- Dark mode toggle (via Ladle's theme addon) displays correctly
- Edge case stories behave as intended at various container widths

## Validation Checklist

- [ ] Story file co-located with component source
- [ ] Default interactive story with args covers all variant dimensions
- [ ] Named stories exist for key variant combinations
- [ ] Edge cases covered (disabled, loading, empty, error as applicable)
- [ ] Title matches component category path
- [ ] Dark mode verified through Ladle's theme toggle
- [ ] MSW handlers added for data-fetching components

- [Ladle Stories](ladle_stories.md)
- [Ladle Setup](ladle_setup.md)

## Related

- the react_router profile's *Component Testing* reference (`profiles/react_router/packages/testing/reference/component_testing.md`)
