# Component Planning

Step-by-step workflow for designing React components with composition patterns, directory placement, and reusability assessment.

Procedure for designing React components before implementation. Planning upfront establishes composition strategy, directory placement, state ownership, and testability boundaries, preventing costly rewrites when components grow beyond their initial scope.

## Prerequisites

Gather before starting:

- **Purpose statement**: One sentence describing what the component renders or manages. Multiple unrelated responsibilities signal multiple components.
- **Consumer inventory**: Which routes or parent components use this component and what data they provide.
- **Reuse assessment**: Whether this component appears in one route or across multiple routes.
- **Data source**: Server data (loader), local state, shared state, or URL state.
- **Component library check**: Whether a UI library (e.g., shadcn/ui) provides the component or a close equivalent.

## Planning Workflow

### 1. Check Library Availability

Determine whether the project's component library provides the component or a close equivalent before planning custom work.

| Result | Action |
|--------|--------|
| Exact match exists | Add from library, customize as needed |
| Partial match (similar component) | Assess if wrapping or extending suffices |
| No match | Plan custom component from scratch |

When a match exists, examine its source to understand composition points and style tokens before deciding on customization depth.

### 2. Assess Reuse and Select Directory

Determine whether the component serves a single route or multiple consumers. This drives file placement.

| Reuse Scope | Placement |
|-------------|-----------|
| Single route | Colocated in route folder |
| 3+ routes | Shared component directory |

For shared components, assign a functional category based on primary purpose:

| Category | Primary Function |
|----------|-----------------|
| `interaction/` | User triggers an action |
| `display/` | Presents read-only content |
| `layout/` | Arranges spatial structure |
| `feedback/` | Communicates status |
| `navigation/` | Facilitates wayfinding |
| `data/` | Displays or manipulates data sets |
| `form/` | Collects user input |

Target path: `src/components/common/{category}/{component-name}/`

[Component Directory Structure](component_directory_structure.md)

### 3. Determine Composition Strategy

Select the composition pattern based on component characteristics.

| Question | Answer | Pattern |
|----------|--------|---------|
| Library component available? | Yes | Wrap or extend the library component |
| Logic reused across 3+ components? | Yes | Extract custom hook |
| Multiple visual variants? | Yes | CVA variant schema |
| Related sub-components share state? | Yes | Compound components with Context |
| Flexible content slots needed? | Yes | Slot-based API (children, named slots) |
| Server-side data fetching? | Yes | Loader in route module, not component |

[Component Composition](component_composition.md)

When no condition applies, use a standard function component with typed props. Resist premature abstraction.

### 4. Plan File Structure

Create the component directory with expected files:

```
src/components/common/{category}/{component-name}/
  {component-name}.tsx          # Component implementation
  {component-name}.stories.tsx  # Stories
  {component-name}.module.css   # CSS module (optional)
```

Test files are not part of the component directory's required layout. Whether component tests are committed beside the component as `{component-name}.test.tsx`, kept in a separate test tree, or replaced by checks against the built application is the repository's test-placement policy; this layout neither requires nor forbids them.

Route-specific components live inside the route folder without the directory-per-component pattern:

```
routes/
  products.$pid/
    route.tsx
    product-card.tsx       # Route-specific presentation
```

### 5. Identify State Ownership

Map each piece of state to its owner. State that originates from the server belongs in loaders. State that controls UI belongs in the nearest component that needs it.

| State Type | Owner | Tool |
|------------|-------|------|
| Server data | Route loader | `loaderData` via Route.ComponentProps |
| Form input | Form instance | React Hook Form or native |
| Toggle/expand | Nearest parent | `useState` |
| Theme, auth, locale | App root | Context API |
| Frequently shared UI | Store | Zustand with selectors |
| URL parameters | Router | `useSearchParams`, `useParams` |

[State Management](state_management.md)

Avoid duplicating server state into local state. Read from `loaderData` directly; mutations go through actions with automatic revalidation.

### 6. Plan Testability and Story Coverage

Determine dependency injection needs and story coverage before writing code, and plan which questions the component's checks will answer:

- **Presentation components**: Receive props, render UI. Exercisable with props alone.
- **Container components**: Fetch data, manage state. Exercisable with `InjectionProvider` wrapping mocked services.
- **Custom hooks**: Encapsulate logic. Exercisable independently with `renderHook`.

Plan story coverage for shared components:

- Default state with interactive args
- Each CVA variant as a named story
- Edge cases: empty, loading, error, disabled
- Composition examples: with icons, in forms, in layouts

[Testability](../../testing/reference/testability.md)

## Decision Support

### When to Extract a Custom Hook

Extract when: the same `useState` + `useEffect` combination appears in 3+ components, or a component's logic obscures its rendering intent.

Stay inline when: the logic is specific to one component and reads clearly alongside JSX.

### When to Use Compound Components

Use when: sub-components have a fixed relationship (Tabs + Tab, Accordion + AccordionItem) and consumers need flexible arrangement.

Use standard props when: content is dynamic from API data, or the component has no meaningful sub-components.

## Validation Checklist

- [ ] Purpose statement describes one responsibility
- [ ] Library availability assessed and decision documented
- [ ] Directory category selected (for shared components)
- [ ] Composition pattern selected and documented
- [ ] File structure planned with all co-located files
- [ ] State ownership mapped (no server state duplication)
- [ ] External dependencies identified for DI
- [ ] Story coverage planned
- [ ] Props interface drafted with types

[Component Styling](component_styling.md)
[Component Testing](../../testing/reference/component_testing.md)
