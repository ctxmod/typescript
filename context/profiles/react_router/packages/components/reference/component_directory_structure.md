# Component Directory Structure

Directory-per-component pattern with co-located files and functional categorization.

Every shared component lives in its own directory with co-located story and style files. Functional categorization groups directories by what components do, producing a flat hierarchy within `src/components/common/`.

- KISS (engineering module, `principles` profile: `design/reference/kiss.md`)
- Convention over configuration (engineering module, `principles` profile: `design/reference/convention_over_configuration.md`)

## Directory-Per-Component Pattern

The canonical path for a shared component follows this structure:

```
src/components/common/{category}/{component-name}/{component-name}.tsx
```

Each component directory contains a fixed set of files:

| File | Purpose | Required |
|------|---------|----------|
| `{component-name}.tsx` | Component implementation | Yes |
| `{component-name}.stories.tsx` | Ladle stories | Yes |
| `{component-name}.module.css` | CSS module | Only when Tailwind utilities cannot express the style |

No exceptions to this layout. Files that change together stay together. Moving or deleting a component means moving or deleting one directory, and nothing else breaks. Stories never drift from their component because they share the same parent folder.

Test files are not part of the component directory's required layout. Whether component tests are committed beside the component as `{component-name}.test.tsx`, kept in a separate test tree, or replaced by checks against the built application is the repository's test-placement policy; this layout neither requires nor forbids them.

## Naming Conventions

Directories use kebab-case. Component files match their directory name in kebab-case. Exported React components use PascalCase.

| Artifact | Convention | Example |
|----------|------------|---------|
| Directory | kebab-case | `date-picker/` |
| Component file | kebab-case, `.tsx` | `date-picker.tsx` |
| Component export | PascalCase | `DatePicker` |
| Story file | `{name}.stories.tsx` | `date-picker.stories.tsx` |
| CSS module | `{name}.module.css` | `date-picker.module.css` |

Kebab-case directories avoid Git case-sensitivity issues across macOS and Linux CI environments. The consistent naming removes any guesswork about file locations.

## Functional Categories

Components under `src/components/common/` organize by primary function:

| Category | Purpose | Examples |
|----------|---------|---------|
| `interaction/` | Clickable elements, triggers | Button, IconButton, Toggle, Switch |
| `display/` | Read-only content presentation | Badge, Card, Avatar, Skeleton |
| `layout/` | Spatial arrangement, containers | Stack, Grid, Separator, ScrollArea |
| `feedback/` | Status communication | Alert, Toast, Progress, Spinner |
| `navigation/` | Routing and wayfinding | Tabs, Breadcrumb, Pagination, NavLink |
| `data/` | Data display and interaction | Table, List, DataGrid, TreeView |
| `form/` | User input collection | Input, Select, Checkbox, RadioGroup, DatePicker |

Assign each component to the category matching its primary function. When a component spans two categories, place it where consumers most expect to find it. A `DropdownMenu` primarily triggers actions, so it belongs in `interaction/`. A `SearchInput` collects input, so it belongs in `form/`.

## Route-Specific vs Shared Components

Components used by a single route live alongside that route module. They stay specific, accepting concrete props tied to their route's data shape.

```
src/
  components/
    common/                # Shared (directory-per-component)
      interaction/
        button/
          button.tsx
          button.stories.tsx
  routes/
    products.$pid/
      route.tsx
      product-card.tsx     # Route-specific, stays here
```

Promote a route-specific component to `common/` only when three or more routes use it. This threshold prevents premature generalization. The component earns its abstraction through demonstrated reuse.

- [Component Composition](component_composition.md)

## Import Patterns

Use path aliases for direct imports:

```typescript
import { Button } from "@/components/common/interaction/button/button";
```

Avoid barrel exports (`index.ts` re-exports). Barrels hinder tree shaking and obscure dependency graphs, particularly in Vite-based builds where one barrel can pull an entire component set into a bundle. Direct imports make dependencies explicit, and IDE auto-import handles the longer paths without manual effort.

## Thresholds

| Metric | Pass | Warn | Fail |
|--------|------|------|------|
| Components per category | 3-15 | 15-25 | >25 (split into subcategories) |
| Files per component directory | 2-4 | 5-6 | >6 (split component) |
| Orphan files outside component dirs | 0 | 1-2 | >2 |

When a category exceeds 25 components, introduce subcategories. For example, `form/` might split into `form/input/` and `form/selection/` as the library grows. Maintain the directory-per-component pattern at every level.

- KISS (engineering module, `principles` profile: `design/reference/kiss.md`)
- Convention over configuration (engineering module, `principles` profile: `design/reference/convention_over_configuration.md`)
- [Component Composition](component_composition.md)
