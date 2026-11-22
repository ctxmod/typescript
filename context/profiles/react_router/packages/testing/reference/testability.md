# Testability

Dependency injection and container/presentation patterns for testable React components, and the testing pyramid they support.

Testable components result from applying composition principles -- dependency injection, container/presentation separation, and small interfaces -- rather than from testing-specific techniques. Components requiring extensive mocking scaffolding typically violate one of these principles. The remedy is design correction.

When tests run (hooks, CI, or explicit invocation) and where they live is the repository's test policy. A working-behavior claim about the real application is strongest when backed by a check that drives the built application from outside, as a user or client would.

- KISS (engineering module, `principles` profile: `design/reference/kiss.md`)

## Context API as Dependency Injection Container

The Context API provides React's native dependency injection mechanism. Define an injection interface, create a context, and expose a provider with a consumption hook:

```typescript
interface DIContainer {
  authService: AuthService;
  apiClient: ApiClient;
  analyticsService: AnalyticsService;
}

const InjectionContext = createContext<DIContainer | null>(null);

export function InjectionProvider({ children, ...services }: Props) {
  const value = useMemo(() => services, [/* deps */]);
  return (
    <InjectionContext.Provider value={value}>
      {children}
    </InjectionContext.Provider>
  );
}

export function useInjectedValue(): DIContainer {
  const ctx = useContext(InjectionContext);
  if (!ctx) throw new Error("Missing InjectionProvider");
  return ctx;
}
```

Components consume services through `useInjectedValue()` rather than importing concrete implementations. Tests substitute mock implementations by wrapping components in `InjectionProvider` with test doubles.

## Container/Presentation Split

Container components handle state, side effects, and dependency consumption. Presentation components receive props and render UI as pure functions.

**Container** (smart):
- Calls hooks (`useInjectedValue`, `useState`, `useEffect`)
- Orchestrates data flow
- Handles events and side effects

**Presentation** (dumb):
- Pure function of props
- No hooks beyond `useMemo` for rendering optimization
- Testable with prop assertions alone

```typescript
// Container
function UserListContainer() {
  const { apiClient } = useInjectedValue();
  const [users, setUsers] = useState<User[]>([]);

  useEffect(() => {
    apiClient.getUsers().then(setUsers);
  }, [apiClient]);

  return <UserListView users={users} />;
}

// Presentation
function UserListView({ users }: { users: User[] }) {
  return (
    <ul>
      {users.map(u => <li key={u.id}>{u.name}</li>)}
    </ul>
  );
}
```

Route modules serve as natural containers. They fetch data in loaders, handle mutations in actions, and delegate rendering to colocated presentational components.

## Infrastructure Services Pattern

Separate API clients from business logic using service classes with constructor-injected dependencies. This creates three layers:

| Layer | Responsibility | Testing |
|-------|---------------|---------|
| API clients | Network requests (single responsibility) | Integration tests with real endpoints |
| Service classes | Data transformations, business logic | Unit tests with mocked API interfaces |
| Components | UI rendering and user interaction | Unit tests with mocked services via DI |

```typescript
interface MediaApi {
  uploadImage(formData: FormData): Promise<{ data: ImageDto }>;
}

class MediaService {
  constructor(private api: MediaApi) {}

  async saveImage(file: File): Promise<Image> {
    const formData = new FormData();
    formData.append("image", file);
    const { data } = await this.api.uploadImage(formData);
    return dtoToImage(data);
  }
}
```

Tests verify service logic by injecting a mock API, without HTTP mocking or network interception.

## Testing Pyramid

| Test Type | Dependencies | Scope | Share |
|-----------|--------------|-------|------------------|
| Isolated unit | Mocked via DI | Single component or hook | Most authored checks |
| Integration | Real implementations | Component + services | Moderate |
| Route testing | Mocked loaders/actions | Full route module | Few |
| End-to-end | Full application | Complete user flows | Minimal -- executed against the real built application |

Presentation components test with prop assertions. Container components test with mocked dependencies via `InjectionProvider`. Route modules test by mocking loader and action exports. The share column guides what authored tests cover; it is not a maintained artifact.

## Testing Route Modules

Test loaders and actions as standalone async functions. They accept `Route.LoaderArgs` or `Route.ActionArgs` and return data -- no React rendering required for data layer tests.

```typescript
test("loader returns user data", async () => {
  const result = await loader({
    params: { userId: "123" },
    request: new Request("http://test/users/123"),
  } as Route.LoaderArgs);

  expect(result.user.id).toBe("123");
});
```

Test route components by rendering with mock loader data. The component receives `loaderData` as a prop through `Route.ComponentProps`.

## Testability Barriers

| Barrier | Mechanism | Resolution |
|---------|-----------|------------|
| Hardcoded imports | Direct service imports bypass DI | Consume services through `useInjectedValue` |
| Global state | Module-level singletons shared across tests | Scope state to providers or Zustand stores |
| Mixed concerns | Container logic embedded in presentation | Split into container and presentation components |
| Wide service interfaces | 10+ method contracts bloat test doubles | Define narrow consumer-specific interfaces |

## Thresholds

| Metric | Pass | Warn | Fail |
|--------|------|------|------|
| DI coverage (services consumed via context) | >80% | 60-80% | <60% |
| Presentation components as % of total | >50% | 30-50% | <30% |
| Test setup lines per test | <10 | 10-20 | >20 |

## Related

- [Component Composition](../../components/reference/component_composition.md)
- [Route Module Design](../../routes/reference/route_module_design.md)
- [State Management](../../components/reference/state_management.md)
