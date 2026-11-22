# Component Testing

UI verification workflow for React components — dependency injection, container/presentation split, and route module exercises; behavior claims through checks against the built application.

Procedure for writing and organizing UI checks for React components in a React Router v7 SSR application. Implementation-coupled questions (does this container pass the right props, does this loader map errors correctly) are answered by component and route tests. A working-behavior claim about the real application requires a check that drives the built application from outside, as a user would; component tests alone never establish it. When these checks run (hooks, CI, or explicit invocation) is the repository's test policy.

## Testing Scope

### What to Test

Concentrate testing effort on components where defects carry real cost:

| Category | Examples |
|----------|----------|
| Conditional rendering logic | Role-based UI, feature flags, empty states |
| User interaction paths | Form submissions, navigation triggers, toggle states |
| Data transformation | Formatting, filtering, sorting before display |
| Error states | Missing data, API failures, validation errors |
| Accessibility contracts | ARIA attributes, keyboard navigation, focus management |

### What to Skip

Simple pass-through components, one-prop wrappers, and layout-only components with no logic offer low return on testing investment. If a component renders children without transformation, testing the parent provides coverage.

## Test Organization

### File Placement

Test files are not part of the component directory's required layout. Place them where the repository's test-placement policy says — beside the component, or in a separate test tree such as:

```
tests/
  button.test.tsx
  route-products.test.tsx
```

Story files (`.stories.tsx`) still co-locate with their component. Stories serve as visual fixtures for verifying variant rendering and dark mode, complementing the executed checks.

- [Component Directory Structure](../../components/reference/component_directory_structure.md)

### Test Types and Scope

| Type | Dependencies | Scope | Speed |
|------|-------------|-------|-------|
| Unit | Mocked via DI | Single component or hook | Fast (<50ms) |
| Integration | Real implementations | Component tree with services | Moderate (<500ms) |
| Route | Mocked loader/action | Full route module | Moderate |
| E2E | Full application | User flows across routes | Slow (seconds) |

## Testing Workflow

### 1. Identify Dependencies

List external dependencies the component consumes: API clients, auth services, analytics, i18n. These flow through the `InjectionProvider` context and require test doubles for isolation.

```typescript
// Component consuming injected dependencies
function UserProfile() {
  const { authService } = useInjectedValue();
  const [user, setUser] = useState<User | null>(null);

  useEffect(() => {
    authService.getCurrentUser().then(setUser);
  }, [authService]);

  if (!user) return <Skeleton />;
  return <ProfileCard user={user} />;
}
```

- [Testability](testability.md)

### 2. Create Test Doubles

Build test doubles based on verification needs, not tool availability.

| Double Type | When | Verification |
|-------------|------|--------------|
| Fake | Domain logic, state matters | Check end result |
| Stub | Predictable responses needed | Check caller behavior |
| Mock | External service boundaries | Check call sequences |
| Spy | Observation concerns (analytics) | Check what was recorded |

Prefer state verification ("profile renders user name") over interaction verification ("getCurrentUser was called once"). State verification survives refactoring.

```typescript
const mockAuthService = {
  getCurrentUser: vi.fn().mockResolvedValue({
    id: "1",
    name: "Test User",
    email: "test@example.com",
  }),
  login: vi.fn(),
  logout: vi.fn(),
};
```

### 3. Write Presentation Component Tests

Presentation components receive props and render UI. Test them with props alone, without DI wrappers.

```typescript
describe("ProfileCard", () => {
  it("renders user name and email", () => {
    const user = { id: "1", name: "Alice", email: "alice@example.com" };

    render(<ProfileCard user={user} />);

    expect(screen.getByText("Alice")).toBeInTheDocument();
    expect(screen.getByText("alice@example.com")).toBeInTheDocument();
  });

  it("shows edit button for admin users", () => {
    const admin = { id: "1", name: "Admin", email: "a@b.com", role: "admin" };

    render(<ProfileCard user={admin} />);

    expect(screen.getByRole("button", { name: /edit/i })).toBeInTheDocument();
  });
});
```

### 4. Write Container Component Tests

Container components manage state and side effects. Wrap them with `InjectionProvider` supplying test doubles.

```typescript
describe("UserProfile", () => {
  it("fetches and displays current user", async () => {
    render(
      <InjectionProvider authService={mockAuthService} apiClient={mockApi}>
        <UserProfile />
      </InjectionProvider>
    );

    expect(await screen.findByText("Test User")).toBeInTheDocument();
    expect(mockAuthService.getCurrentUser).toHaveBeenCalledOnce();
  });
});
```

### 5. Test Custom Hooks

Test hooks independently using `renderHook` from `@testing-library/react`:

```typescript
describe("useAuth", () => {
  it("returns current user after initialization", async () => {
    const wrapper = ({ children }) => (
      <InjectionProvider authService={mockAuthService} apiClient={mockApi}>
        {children}
      </InjectionProvider>
    );

    const { result } = renderHook(() => useAuth(), { wrapper });

    await waitFor(() => {
      expect(result.current.user).toEqual({ id: "1", name: "Test User" });
      expect(result.current.loading).toBe(false);
    });
  });
});
```

### 6. Test Route Modules

Test route modules by mocking loader and action exports. Use `createRoutesStub` (or equivalent test utility) to render routes with controlled data.

```typescript
describe("products/$pid route", () => {
  it("renders product from loader data", () => {
    const product = { id: "1", name: "Widget", price: 9.99 };

    render(
      <MemoryRouter initialEntries={["/products/1"]}>
        <Route
          path="/products/:pid"
          element={<Product />}
          loader={() => product}
        />
      </MemoryRouter>
    );

    expect(screen.getByText("Widget")).toBeInTheDocument();
  });
});
```

Test actions by simulating form submissions and verifying revalidation behavior.

## Coverage Philosophy

Apply coverage proportionally to defect impact:

| Risk Level | Coverage Target | Examples |
|------------|----------------|----------|
| Critical | Thorough | Auth flows, payment, data mutations |
| Standard | Baseline (80-90%) | Business logic, form handling |
| Low | Minimal | Layout wrappers, pass-through components |

Use coverage as a minimum floor catching gross omissions, not a target to maximize. Stop increasing coverage when defect discovery plateaus.

## Validation Checklist

- [ ] Presentation components tested with props alone
- [ ] Container components tested with `InjectionProvider`
- [ ] Custom hooks tested independently with `renderHook`
- [ ] Route modules tested with mocked loader/action data
- [ ] State verification preferred over interaction verification
- [ ] Critical paths have thorough coverage
- [ ] Test doubles match verification needs (fake, stub, mock, spy)
- [ ] Checks placed according to the repository's test-placement policy

## Related

- [Component Planning](../../components/reference/component_planning.md)
- [Route Planning](../../routes/reference/route_planning.md)
