# Story Development

Workflow for creating Storybook stories with controls, decorators, interaction tests, and variant coverage.

Procedure for creating Storybook stories that cover component variants, interactive controls, and edge cases. Stories serve as living documentation, visual fixtures, and — through the Vitest addon — executed smoke, interaction, and accessibility tests.

## Prerequisites

- Component implemented and styled
- Storybook configured with the application stylesheet, theme switching, and the a11y and Vitest addons
- Component's CVA variants identified (if applicable)

## Development Workflow

### 1. Create Story File

Create `{component-name}.stories.tsx` co-located with the component source:

```tsx
import type { Meta, StoryObj } from "@storybook/react-vite";
import { ComponentName } from "./component-name";

const meta = {
  title: "Category/ComponentName",
  component: ComponentName,
} satisfies Meta<typeof ComponentName>;

export default meta;
type Story = StoryObj<typeof meta>;
```

The title prefix reflects the component's category in the navigation tree: `"Interaction/Button"`, `"Form/Input"`, `"Display/Card"`. Storybook generates its sidebar hierarchy from these title paths; they must be string literals, since titles are read statically when stories are indexed.

### 2. Write Default Story

Create an interactive story whose args cover every variant dimension:

```tsx
export const Default: Story = {
  args: {
    variant: "default",
    size: "default",
    disabled: false,
    children: "Example content",
  },
};
```

Controls are inferred from the component's prop types — booleans as toggles, string unions as selects; add `argTypes` only to override an inferred control (`size: { control: "radio" }`).

### 3. Add Named Variant Stories

Create one story per meaningful variant combination for quick visual scanning:

```tsx
export const Secondary: Story = {
  args: { variant: "secondary", children: "Secondary action" },
};

export const AllSizes: Story = {
  render: (args) => (
    <div className="flex items-center gap-2">
      <ComponentName {...args} size="sm">Small</ComponentName>
      <ComponentName {...args} size="default">Medium</ComponentName>
      <ComponentName {...args} size="lg">Large</ComponentName>
    </div>
  ),
};
```

Named stories give reviewers a direct reference for each state without toggling controls, and each runs as its own smoke and accessibility test.

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

Decorators wrap stories with shared layout or provider context, per story or for every story in the file (default export):

```tsx
const meta = {
  title: "Form/Select",
  component: Select,
  decorators: [
    (Story) => (
      <div className="max-w-sm">
        <Story />
      </div>
    ),
  ],
} satisfies Meta<typeof Select>;
```

Common uses: max-width containers for form components, flex layouts for side-by-side comparisons, provider wrappers for context-dependent components. A decorator needed by only one story goes on that story's `decorators`.

### 6. Add Interaction Tests and API Mocks

For interactive components, add a `play` function that drives the story and asserts with `storybook/test` ([Story Patterns](story_patterns.md)). For components that fetch data, attach MSW handlers under `parameters.msw.handlers` (with `msw-storybook-addon` initialized in `.storybook/preview.ts`).

### 7. Verify in Storybook

Run `npm run storybook` and verify:

- All stories render without console errors
- Controls produce expected visual changes for interactive stories
- The theme toolbar shows both light and dark themes correctly
- The accessibility panel reports no violations
- Edge case stories behave as intended at various viewport widths

Then run the stories as tests (`vitest --project storybook`, or the Storybook testing widget): every story must render, every `play` function must pass, and accessibility checks must pass at the configured level.

## Validation Checklist

- [ ] Story file co-located with component source
- [ ] Default interactive story with args covers all variant dimensions
- [ ] Named stories exist for key variant combinations
- [ ] Edge cases covered (disabled, loading, empty, error as applicable)
- [ ] Title matches component category path
- [ ] Both themes verified through the theme toolbar
- [ ] Accessibility checks pass; interactive components have a `play` test
- [ ] MSW handlers added for data-fetching components

- [Story Patterns](story_patterns.md)
- [Storybook Setup](storybook_setup.md)

## Related

- the react_router profile's *Component Testing* reference (`profiles/react_router/packages/testing/reference/component_testing.md`)

## Sources

- https://storybook.js.org/docs/api/csf — CSF 3 meta and story objects, `render`, args, decorators (accessed 2026-11-22)
- https://storybook.js.org/docs/writing-tests/integrations/vitest-addon — running stories as Vitest tests (docs v10.6; accessed 2026-11-22)
- https://storybook.js.org/docs/writing-tests/accessibility-testing — `parameters.a11y.test` levels `off`/`todo`/`error` (accessed 2026-11-22)
