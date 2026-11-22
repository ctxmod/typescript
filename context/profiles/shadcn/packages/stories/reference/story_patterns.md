# Story Patterns

Component Story Format patterns for Storybook: story file structure, args and controls, decorators, `play`-function interaction tests, co-location, API mocking, and variant coverage.

Stories preview components in isolation with interactive controls and, run
through the Vitest addon, act as executed smoke, interaction, and
accessibility tests. Each story file default-exports its metadata and exports
named stories that appear in the Storybook sidebar. Co-locate story files
alongside their components for discoverability.

## Story File Structure (CSF 3)

```tsx
import type { Meta, StoryObj } from "@storybook/react-vite";
import { Button } from "./button";

const meta = {
  title: "Interaction/Button",
  component: Button,
  args: { children: "Action" },
} satisfies Meta<typeof Button>;

export default meta;
type Story = StoryObj<typeof meta>;

export const Primary: Story = { args: { variant: "default" } };
export const Secondary: Story = { args: { variant: "secondary", children: "Cancel" } };
```

Named exports become individual stories. The `title` controls navigation
hierarchy — use the component's category path for consistent grouping:
`"Interaction/Button"`, `"Form/Select"`, `"Display/Card"`. When `title` is
omitted, Storybook derives it from the file path. `satisfies Meta<typeof
Button>` keeps the meta's literal types so `StoryObj<typeof meta>` knows which
args are already provided.

**CSF Factories** (Preview in Storybook 10, expected default in 11) replace
the `Meta`/`StoryObj` typing with `preview.meta({ component })` and
`meta.story({ args })`, imported from the typed `.storybook/preview`
(`definePreview`). Adopt them per repository, all at once, with a
`#.storybook/preview` subpath import in `package.json` instead of deep
relative paths; mixing CSF 3 and Factories file by file splits the
conventions reviewers read against.

## Controls with Args

Args are typed from the component's props, and Storybook infers controls from
them: booleans become toggles, string unions become selects. Override only
where inference is wrong or unhelpful:

```tsx
const meta = {
  component: Button,
  args: { variant: "default", size: "default", disabled: false, children: "Click me" },
  argTypes: {
    size: { control: "radio" },
    onClick: { action: "clicked" },
  },
} satisfies Meta<typeof Button>;
```

Args cascade through three levels: global args (`.storybook/preview.ts`) <
component-level args (the default export) < per-story args. Per-story values
take precedence. Use `fn()` from `storybook/test` for callback args that
`play` functions assert on.

## Decorators

Decorators wrap stories with layout or provider context, at the story,
component (default export), or global (`preview.ts`) level:

```tsx
const meta = {
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

Common uses:
- **Layout wrappers** — flex containers, padding, max-width constraints
- **Provider injection** — theme, i18n, or auth context wrapping
- **Mock data setup** — shared fixture state accessible to all stories

## Interaction Tests with `play`

A `play` function drives the rendered story and asserts on it; the Vitest
addon runs it as a test, and the Storybook UI replays it step by step:

```tsx
import { expect, fn, userEvent, within } from "storybook/test";

export const SubmitsOnClick: Story = {
  args: { onClick: fn() },
  play: async ({ args, canvasElement }) => {
    const canvas = within(canvasElement);
    await userEvent.click(canvas.getByRole("button", { name: "Click me" }));
    await expect(args.onClick).toHaveBeenCalledOnce();
  },
};
```

Query by role and accessible name, as a user would; a story that needs a
test ID to find its element usually has an accessibility gap the a11y check
will also report.

## Co-location Pattern

```
src/components/common/interaction/button/
  button.tsx
  button.stories.tsx
  button.module.css    (optional)
```

The `stories` glob in `.storybook/main.ts` discovers co-located files;
restrict it to the component tree.

- the react_router profile's *Component Directory Structure* reference (`profiles/react_router/packages/components/reference/component_directory_structure.md`)

## API Mocking

Stories that depend on API data attach Mock Service Worker handlers through
`msw-storybook-addon`:

```tsx
import { http, HttpResponse } from "msw";

export const WithData: Story = {
  parameters: {
    msw: {
      handlers: [
        http.get("/api/users", () =>
          HttpResponse.json([
            { id: 1, name: "Alice" },
            { id: 2, name: "Bob" },
          ]),
        ),
      ],
    },
  },
};
```

Component-level handlers in the default export's `parameters.msw` apply to
every story in the file; per-story handlers override them.

## Variant Coverage Strategy

For components built with CVA variant definitions, stories should cover the
variant matrix systematically.

| Coverage | Stories |
|----------|---------|
| Default state | One story rendering the component with no explicit props |
| Each variant value | One named story per variant (default, secondary, ghost, destructive) |
| Size range | One composite story displaying all sizes side-by-side |
| Interactive states | Disabled, loading, and error variants |
| Composition | Component combined with siblings (icon alongside label, badge inside card) |

Pair one interactive story with full args for ad-hoc exploration with named
stories for documentation. Named stories serve as visual regression anchors —
each captures a single, predictable configuration — and each is also a smoke
and accessibility test under the Vitest addon.

## Anti-Patterns

- **Stories without args** — prevents interactive exploration and limits the story to a static preview
- **One story per file** — loses the grouping context that related variants provide; multiple exports per file reduce navigation noise
- **Hardcoded mock data inline** — extract shared fixtures to a `__fixtures__` directory or a `*.fixtures.ts` co-located file for reuse across stories and tests
- **Skipping dark mode verification** — verify both themes through the theme toolbar; visual regressions frequently hide in the alternate theme
- **Silencing accessibility checks** — a story-level `a11y.test: "todo"` carries the reason and the issue it waits on

- [Storybook Setup](storybook_setup.md)

## Related

- the react_router profile's *Component Testing* reference (`profiles/react_router/packages/testing/reference/component_testing.md`)

## Sources

- https://storybook.js.org/docs/api/csf — Component Story Format 3: default-export meta, named story objects, `satisfies Meta` (accessed 2026-11-22)
- https://storybook.js.org/docs/api/csf/csf-next — CSF Factories (Preview): `definePreview`, `preview.meta`, `meta.story`, `#.storybook/preview` subpath imports (accessed 2026-11-22)
- https://storybook.js.org/docs/writing-tests/interaction-testing — `play` functions with `storybook/test` (`expect`, `fn`, `userEvent`, `within`) (accessed 2026-11-22)
- https://storybook.js.org/docs/writing-tests/integrations/vitest-addon — stories run as Vitest tests with `play` assertions and a11y checks (docs v10.6; accessed 2026-11-22)
- https://www.npmjs.com/package/msw-storybook-addon — `parameters.msw.handlers` with MSW 2 (3.0.3; accessed 2026-11-22)
