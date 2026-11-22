# OpenAPI Consumption

How the browser consumes the OpenAPI 3.2 contract — modular source, stable operation identifiers, the common schema set, additive-optional evolution, preconditions and idempotency, and the lint-bundle-generate pipeline in semantics.

**OpenAPI 3.2** is canonical for the browser-facing HTTP command/query
boundary. The version is fixed by this module and is never
self-declared, inferred from a tool's default, or bumped because a
generator supports something newer. This reference owns how the browser
side reads and depends on that contract.

## Modular Contract Source

Organize the contract source so review scales with the contract, and so
generation has one unambiguous entry point:

- **A small root document that imports path items.** The root carries
  metadata and the path map, not schemas.
- **One file per reusable schema.** A schema that two operations share
  lives in one file that both reference.
- **Local relative references only.** No remote references and no
  registry lookups: the contract must bundle and generate offline, from
  the repository alone.
- **Paths and schemas grouped by domain**, one directory per domain, so
  ownership of a change is legible from the path.

## Operation Identifiers Are Consumer-Facing

**Every operation carries a stable operation identifier.** The generated
client's function names, hook names, mutator names, and mock handler
names all derive from it, so an operation identifier is part of the
browser's public surface.

The consequence is worth stating plainly: **changing an operation
identifier is a consumer-breaking change.** It renames every generated
symbol derived from it and breaks every call site at once. Rename an
operation identifier only with the same deliberation as removing an
endpoint, and never as an incidental tidy-up during an unrelated change.

Paths are resource-oriented. Explicit custom actions are reserved for
Commands — a request that something be performed — rather than being used
as a general escape hatch for operations that were awkward to model as
resources.

## The Common Schema Set

A shared schema set is referenced across domains rather than restated per
operation. The browser depends on all of it:

- resource identifiers;
- pagination tokens and limits;
- timestamps;
- correlation identifiers;
- the problem-details error model, carrying a stable application error
  code;
- revision-precondition objects;
- references to Audit Records and to receipts.

Command payloads set `additionalProperties: false`, so an unexpected
field is a contract violation rather than silently ignored input.

The typed error contract itself — the problem-details shape, the
application error code vocabulary, and the typed conflict response a
stale precondition produces — is owned by
[Problem Details](../../frontend/reference/problem_details.md). This reference
names the schema; it does not define it.

## Evolution Discipline

Four rules, in force for every change to the contract:

1. **Additive optional fields are the default compatible path.** A new
   optional field is the first shape to reach for.
2. **Never change a field's meaning.** A field whose semantics change is
   a new field, whatever its name suggests.
3. **Never reuse a removed field name.** Old consumers and stored
   payloads outlive the removal.
4. **Each evolution step advances the `contract_revision`.** The marker
   is how consumers and Audit Records identify which contract a payload
   was produced against.

A change that genuinely breaks a consumer is resolved through these rules
and the `contract_revision`, never by patching consumer code around the
contract.

## Preconditions and Idempotency

- **Every mutating operation carries an idempotency key**, so a retried
  request cannot produce a second effect.
- **Approval-style Commands carry their `workflow_revision` and
  `action_revision` preconditions**, so an action proposed against a
  condition that has since moved fails with a typed conflict response
  instead of committing an incorrect effect.

The browser passes both through the generated mutator for the operation;
it never constructs them at a hand-written call site. The response
handling for the conflict case is the error model's, referenced above.

## Streaming Responses

OpenAPI 3.2 can describe streaming responses: it defines sequential and
streaming media types and a per-item schema mechanism that applies to
each item of a stream independently, rather than to a complete body. The
capability exists and is available where a response is genuinely
incremental. Choosing it, and designing anything around it, is out of
scope here — this reference records only that such a response can stay
inside the contract instead of escaping it.

## The Pipeline, in Semantics

The chain from contract source to browser output has three steps and one
invariant.

```
contracts/api/  (modular OpenAPI 3.2 source)
  -> lint      # contract quality rules, before anything consumes it
  -> bundle    # one portable, reviewed artifact from the source graph
  -> generate  # browser client, query hooks, mutators, mocks
```

- **Lint and bundle** are the contract toolchain's (Redocly). The bundle
  is the reviewed artifact and the single input to every generator, so no
  two consumers can resolve the source graph differently.
- **Generation** for the browser outputs is the client generator's
  (Orval), reading the bundle — never the unbundled source graph.
- **The invariant:** every language's output is generated from that same
  bundle. Tool invocation, flags, and configuration files belong to each
  tool's documentation and the repository's pinned configuration, not to
  this reference.

## Where the Rest Lives

- The **generated-clients reference** owns the generated browser surface
  and how components consume it.
- The **generated-code discipline reference** owns the generated tree's
  properties, the pinned toolchain, and the drift principle; the drift
  gate itself is a validation procedure.
- The **contract-first architecture reference** owns boundary ownership
  and repository placement of the contract source and generated output.

## Related

- [Contract-First Architecture](contract_first_architecture.md)
- [Generated Clients](generated_clients.md)
- [Generated-Code Discipline](generated_code_discipline.md)
