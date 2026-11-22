# Generated Clients

The generated browser SDK surface — typed client, query hooks, mutators, and mocks from one bundled contract; the prohibition on hand-written fetch wrappers; and the view-model allowance that scopes it.

The browser's entire transport surface is generated from the bundled API
contract. Nothing between a component and the HTTP boundary is written by
hand. This reference owns what that generated surface contains, the
prohibition it enforces, and the precise scope of that prohibition.

## The Generated Browser Surface

One bundled contract produces four coordinated outputs. They are
generated together, from the same input, in the same step — so they
cannot describe different shapes.

| Output | Responsibility | Typical tool |
|---|---|---|
| Typed client | The transport layer: request construction, response typing, error typing | Orval |
| Query hooks | Reads: caching, revalidation, loading and error states, per-operation keys | Orval generating TanStack Query hooks |
| Typed mutators | Writes: one typed function per mutating operation, including its precondition and idempotency parameters | Orval |
| Mocks | Request handlers whose shapes match the contract, for tests and local development | Orval generating MSW handlers |

Library selection is in scope for this profile, and the tools above are
the default selection. Their configuration surfaces, options, and
invocation syntax are not restated here — those belong to each tool's own
documentation, which the repository's generator configuration links to
and pins.

## Hand-Written Fetch Wrappers Are Prohibited

Two rules, stated without exception:

- **Every read goes through a generated query hook.** A component that
  needs server data calls the hook the contract produced for that
  operation.
- **Every mutation goes through a generated mutator.** A component that
  changes server state calls the typed mutator for that operation, and
  passes the idempotency key and precondition parameters the mutator
  declares.

A hand-written fetch wrapper — even a thin one, even one that only adds a
base path or a header — reintroduces every failure the contract exists to
prevent: an untyped response shape, a URL that no longer matches the
contract after an operation is renamed, an error branch that does not
match the error model, and a call site that the drift check cannot see.
The convenience the wrapper adds is one generator configuration option
away from being generated instead.

## The View-Model Allowance

The no-handwritten rule is scoped, not blanket. The precise rule is:

> Never hand-author models that duplicate an external transport contract.

The browser legitimately needs models the contract does not describe, and
these remain handwritten code:

- **view composition** — the shape a screen or region assembles from
  several sources;
- **local state** — anything owned by the browser and never sent;
- **form state** — draft values, dirty tracking, per-field validation
  status;
- **derived UI state** — values computed from server data for display;
- **domain-specific presentation models** — a shape that exists to be
  rendered, not to be transmitted.

None of these restate the wire shape; they present or assemble
information. The test is semantic, not structural: a handwritten model
that happens to have the same fields as a generated type is fine when it
exists for a different reason, and a violation when it exists because
someone wanted the generated type with one field changed. In the second
case the change belongs in the contract.

## Consume, Never Re-Wrap

The generated surface is consumed directly by the code that needs it. It
is never wrapped in a parallel hand-authored abstraction layer that
re-declares the contract — a "client service", a "repository", or an "API
layer" whose methods mirror the generated operations one for one.

Such a layer is contract duplication with extra steps: it must be updated
by hand on every contract change, it obscures which operation a component
actually calls, and it becomes the place where a stray untyped shape
survives regeneration. Where cross-cutting behavior is genuinely needed —
authentication, correlation identifiers, retry policy — it belongs in the
generator's configuration or the client's single injected transport
configuration, applied once, not in a per-operation wrapper.

Composition code that assembles generated hooks into a view is not a
wrapper layer: it produces a view model, it does not re-declare an
operation.

## Placement

The generated client tree sits outside application source, under the
repository's generated output directory, and is imported by application
code (see the contract-first repository layout in the contract-first
architecture reference). It is never edited, and it is never relocated
into an application's source tree to make imports shorter — a generated
tree inside source is a generated tree someone will eventually edit.
Import paths into the generated tree follow the same module conventions
as any other dependency in the workspace
([Module Organization](../../modules/reference/module_organization.md)).

## Where the Rest Lives

- The **OpenAPI consumption reference** owns the contract source that
  produces this surface, including the operation identifiers the client's
  function names derive from.
- The **generated-code discipline reference** owns the properties of the
  generated tree, the editing prohibitions, and the pinned toolchain.
- The **contract-first architecture reference** owns the handwritten-
  versus-generated split and boundary ownership.

## Related

- [Contract-First Architecture](contract_first_architecture.md)
- [Generated-Code Discipline](generated_code_discipline.md)
- [OpenAPI Consumption](openapi_consumption.md)
