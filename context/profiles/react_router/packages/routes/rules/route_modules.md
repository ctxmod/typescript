---
paths:
  - "*.ts"
  - "*/*.ts"
  - "*/*/*.ts"
  - "*/*/*/*.ts"
  - "*/*/*/*/*.ts"
  - "*/*/*/*/*/*.ts"
  - "*/*/*/*/*/*/*.ts"
  - "*/*/*/*/*/*/*/*.ts"
  - "*/*/*/*/*/*/*/*/*.ts"
  - "*/*/*/*/*/*/*/*/*/*.ts"
  - "*.tsx"
  - "*/*.tsx"
  - "*/*/*.tsx"
  - "*/*/*/*.tsx"
  - "*/*/*/*/*.tsx"
  - "*/*/*/*/*/*.tsx"
  - "*/*/*/*/*/*/*.tsx"
  - "*/*/*/*/*/*/*/*.tsx"
  - "*/*/*/*/*/*/*/*/*.tsx"
  - "*/*/*/*/*/*/*/*/*/*.tsx"
  - "*.mts"
  - "*/*.mts"
  - "*/*/*.mts"
  - "*/*/*/*.mts"
  - "*/*/*/*/*.mts"
  - "*/*/*/*/*/*.mts"
  - "*/*/*/*/*/*/*.mts"
  - "*/*/*/*/*/*/*/*.mts"
  - "*/*/*/*/*/*/*/*/*.mts"
  - "*/*/*/*/*/*/*/*/*/*.mts"
  - "*.cts"
  - "*/*.cts"
  - "*/*/*.cts"
  - "*/*/*/*.cts"
  - "*/*/*/*/*.cts"
  - "*/*/*/*/*/*.cts"
  - "*/*/*/*/*/*/*.cts"
  - "*/*/*/*/*/*/*/*.cts"
  - "*/*/*/*/*/*/*/*/*.cts"
  - "*/*/*/*/*/*/*/*/*/*.cts"
---

# Route Module Context

Route module design, state management, and route implementation procedures for React Router route files.

Route modules serve as full-stack boundaries encompassing data loading, mutations, UI, and error handling. Route design principles, state management patterns, and procedures for planning, loaders/actions, and error handling apply to all route files. Request-scoped values (auth, locale, platform bindings) arrive through middleware and typed `createContext` keys, read with `context.get`.

- [Route Module Design](../reference/route_module_design.md)
- [State Management](../../components/reference/state_management.md)
- [Route Planning](../reference/route_planning.md)
- [Loader and Action Implementation](../reference/loader_action.md)
- [Route Error Handling](../reference/error_handling.md)
