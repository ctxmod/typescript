# Contract-First Architecture

Contract-first architecture for the browser side — contracts as source code, the handwritten-versus-generated split, contract-first monorepo placement, and boundary ownership.

Write the durable browser code once and generate everything that crosses
the HTTP boundary: types, the client, query hooks, mutators, and mocks.
The API contract is source code — it is reviewed before implementation
code, and a change flows contract first, then regeneration, then
implementation. The contract source is the single source of truth for its
boundary, and no component deserializes untyped JSON at a process
boundary.

## Contracts Are Source Code

- The API contract lives in the repository, is code-reviewed like code,
  and changes through the same review path as implementation.
- A needed behavior change starts in the contract: edit the contract
  source, regenerate the derived output, then write the small amount of
  handwritten code the change requires. Never the reverse.
- Every contract carries a `contract_revision` — the compatibility and
  schema-evolution marker that consumers and Audit Records reference.
- A component may not deserialize untyped JSON at a process boundary.
  Payloads cross as generated types, never as untyped objects shaped by
  hand at the call site.
- **Generated directories are never manually edited.** The full rule for
  generated output — its four properties and the two prohibitions that
  follow from them — is owned by the generated-code discipline reference
  ([Generated-Code Discipline](generated_code_discipline.md)).
  This reference states the prohibition and links; it does not restate
  the rule.

## Handwritten versus Generated on the Browser Side

Keep handwritten browser code intentionally small and clearly scoped.

**Handwritten:**

- view composition and visual components;
- local UI state, form state, and derived UI state;
- domain-specific presentation models;
- composition code that assembles the application from its parts.

**Never handwritten:**

- browser fetch calls;
- request and response interfaces duplicated across languages;
- JSON parsing at the HTTP boundary;
- mock data shapes;
- API reference documentation.

The rule is precise, not blanket: never hand-author models that duplicate
an external transport contract. View models, local and form state,
derived UI state, and presentation models are not contract duplication
and remain legitimate handwritten code. The generated surface itself —
client, query hooks, mutators, mocks — is owned by the generated-clients
reference.

## Contract-First Repository Layout

The layout below applies when a repository hosts a contract surface
shared by more than one consumer. It composes with — and never replaces —
the package and workspace conventions this profile already owns.

```
<repo>/
  contracts/
    api/                   # modular OpenAPI 3.2 source graph
  gen/                     # generated output; never hand-edit
    api/
      bundle/              # bundled, reviewed contract artifact
      typescript/          # client, types, query hooks, mocks
  apps/
    <app>/
      src/                 # view composition, local state, composition code
  packages/                # shared source packages, one directory per domain
```

Three placement rules hold:

1. **The contract source and the generated output are separate trees.**
   The contract source is reviewed and edited; the generated tree is
   written only by the generator.
2. **The application imports the generated tree; the generated tree
   imports nothing from the application.** The dependency runs one way,
   so regeneration can never be blocked by application code.
3. **One bundled artifact feeds every consumer.** The browser output and
   every other language's output are generated from the same reviewed
   bundle, so the languages cannot drift apart.

Package boundaries, the exports field, and workspace wiring stay
authoritative in the module-organization reference
([Module Organization](../../modules/reference/module_organization.md)); how
modules inside an application refer to each other stays authoritative in
the import and export conventions reference
([Import and Export Conventions](../../modules/reference/import_export_conventions.md)).
Neither is restated here.

## Boundary Ownership

- **The browser-facing HTTP command/query boundary is OpenAPI 3.2's.**
  It owns endpoint definitions, operation identifiers, request and
  response schemas, the error model, compatibility rules, and security
  requirements. How this profile consumes that contract is owned by the
  OpenAPI consumption reference.
- **Asynchronous Domain Event contracts on the message bus are a
  different boundary**, and Protobuf is the canonical contract
  technology there. Its envelope, topic-schema, and evolution specifics
  are owned by the cloud-platform doctrine; browser code does not
  consume that boundary directly, and this profile does not specify it.
- **Internal typed interfaces** — application-internal Commands, view
  models, tool and configuration schemas — are owned by the layer that
  defines the responsibility and are not automatically expressed as
  OpenAPI models.
- **Cross-language duplication is resolved by generation, not parallel
  authorship.** A second hand-maintained copy of a contract shape in
  another language is two contracts claiming one boundary.

## Where the Rest Lives

- The **generated-code discipline reference** owns the properties of
  generated output, the editing prohibitions, and the committed-versus-
  regenerated posture.
- The **generated-clients reference** owns the generated browser surface,
  the prohibition on hand-written fetch wrappers, and the view-model
  allowance.
- The **OpenAPI consumption reference** owns the modular contract source,
  operation identifiers, the common schema set, evolution discipline, and
  the generation pipeline in semantics.
- The **cloud-platform doctrine** owns the asynchronous
  Domain Event boundary and platform deployment; this profile stops at
  the browser application boundary.

## Related

- SSOT (engineering module, `principles` profile: `design/reference/ssot.md`)
- [Generated Clients](generated_clients.md)
- [OpenAPI Consumption](openapi_consumption.md)
