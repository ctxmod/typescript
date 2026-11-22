# Component Composition

Composition patterns for reusable React components in SSR context.

Composition over inheritance governs all component design. Custom hooks, compound components, and slot-based APIs replace legacy patterns (HOCs, render props, class inheritance). Start simple, add structure only when complexity demands it.

- DRY (engineering module, `principles` profile: `design/reference/dry.md`)
- KISS (engineering module, `principles` profile: `design/reference/kiss.md`)

## Custom Hooks as Primary Reuse Mechanism

Custom hooks extract and share logic across components. When a behavior repeats in three or more components, encapsulate it in a hook. Hooks compose naturally with dependency injection patterns, accepting injected services through context.

```typescript
function useAuth() {
  const { authService } = useInjectedValue();
  const [user, setUser] = useState<User | null>(null);

  useEffect(() => {
    authService.getCurrentUser().then(setUser);
  }, [authService]);

  return { user };
}
```

Hooks replace Higher-Order Components and Render Props with a simpler mental model. They preserve type inference, avoid wrapper nesting, and integrate with React's rules of hooks for predictable execution.

## Compound Components

Compound components expose a family of sub-components sharing implicit state through Context. They provide declarative, intent-based APIs where consumers control arrangement without managing internal coordination.

```typescript
<Menu onChange={handleChange}>
  <Menu.Button>Toggle</Menu.Button>
  <Menu.List>
    <Menu.Item id="home">Home</Menu.Item>
    <Menu.Item id="about">About</Menu.Item>
  </Menu.List>
</Menu>
```

Use compound components for static layouts with flexible arrangement needs: radio groups, tab bars, accordions, navigation menus. Avoid them for dynamic content driven by API responses, where props-based APIs provide better data mapping.

For type safety, the factory pattern ties type parameters across the component family:

```typescript
const { RadioGroup, RadioGroupItem } = createRadioGroup<string>();
```

## Slot-Based APIs

Slots provide named content injection points within components, replacing prop-bloated APIs. Instead of `headerContent`, `footerActions`, and `sidebarContent` props, expose sub-components:

```typescript
<Card>
  <Card.Header><UserIcon /> Profile</Card.Header>
  <Card.Body>Content</Card.Body>
  <Card.Footer><Button>Edit</Button></Card.Footer>
</Card>
```

This pattern maintains visual consistency while allowing arbitrary JSX at controlled positions.

## Decision Framework

| Scenario | Approach | Rationale |
|----------|----------|-----------|
| shadcn component available | Wrap or extend the shadcn component | Leverage accessible primitives |
| Logic reused across 3+ components | Extract custom hook | Single source of truth, DRY |
| Related components with shared state | Compound components | Declarative composition |
| Component with 4-5+ optional props | Consider duplication | Focused implementations over configuration |
| Server-side data fetching | Loader in route module | SSR-first architecture |
| Named content injection points | Slot-based sub-components | Composition over prop explosion |
| Multiple visual variants | CVA variant schema | Declarative variant management |

## Container/Presentation Split

Separate logic from rendering. Container components orchestrate data fetching, state management, and side effects. Presentation components receive props and render UI as pure functions.

This split enables independent testing: presentation components test with prop assertions, container components test with mocked dependencies. Route modules naturally serve as containers, delegating rendering to colocated presentational components.

## File Organization

Organize shared components using the directory-per-component pattern with functional categorization. Each component lives in its own directory containing its implementation, tests, stories, and optional CSS module.

- [Component Directory Structure](component_directory_structure.md)

Route-specific components live alongside route modules. Extract to shared component directories only when three or more routes use the same component.

## Thresholds

| Metric | Pass | Warn | Fail |
|--------|------|------|------|
| Component prop count | <8 | 8-12 | >12 |
| Component file size (lines) | <150 | 150-250 | >250 |
| Hook dependencies (useEffect) | <4 | 4-6 | >6 |
| Nesting depth (component tree) | <4 | 4-6 | >6 |

Components exceeding these thresholds benefit from decomposition: extract hooks for logic, sub-components for rendering, and slots for flexible content injection.

## Anti-Patterns

- **Prop drilling through 3+ levels**: Use Context or compound components instead.
- **Kitchen sink components**: Components with 12+ props conflate too many concerns. Split by responsibility.
- **Premature abstraction**: Three similar lines beats a helper function until the pattern repeats clearly.
- **Inheritance hierarchies**: React's model does not support class inheritance well. Composition handles every case.

## Related

- [Route Module Design](../../routes/reference/route_module_design.md)
- [Testability](../../testing/reference/testability.md)
- [State Management](state_management.md)
