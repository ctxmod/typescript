# Problem Details

The browser-side error contract — RFC 9457 problem details carrying a stable application error code, the typed stale-precondition conflict and its client remedy, Command idempotency as the client sees it, error normalization at the client boundary, and the rule that the interface never implies an action occurred.

Errors are part of the contract the browser consumes. The generated client
turns a failed response into a typed value, and the interface decides what
a person sees from it — so the failure surface is designed with the same
discipline as the success surface. This reference owns the browser side of
that discipline: the error contract shape, the typed conflict a stale
precondition produces, Command idempotency as the client experiences it,
normalization at the client boundary, and the honesty rules governing what
an interface may claim happened.

## The Error Contract Shape

**RFC 9457 problem details is the error contract shape** at the
browser-facing HTTP boundary, carrying a **stable application error code**
alongside the standard members:

| Member | Meaning |
|---|---|
| `type` | A stable URI identifying the failure class, resolvable to its documentation |
| `title` | A short human-readable summary, stable per `type` |
| `status` | The HTTP status code |
| `detail` | A human-readable explanation of this occurrence |
| `instance` | The operation path this occurrence happened on |
| `code` (extension) | The stable application error code |

One shape serves every failure — request validation, authorization, a
stale precondition, a dependency failure, a business-rule rejection — so
the client parses one schema rather than one per endpoint. The schema
itself belongs to the contract's common schema set, owned by the OpenAPI
consumption reference.

**Stability is a contract property.** An error code names a failure class
and outlives refactors: it is never repurposed, never silently redefined,
and never retired without the deliberation a removed endpoint gets.
Changes to the code set follow the same additive-optional evolution
discipline as the rest of the contract, and each step advances the
`contract_revision`.

## The Typed Stale-`workflow_revision` Conflict

A Command carrying a `workflow_revision` precondition that no longer
matches authoritative state **fails with a typed 409 Conflict** rather
than producing an incorrect effect. An approval Command additionally
validates its `action_revision` against its `workflow_revision`
precondition, so an approval granted against a proposal that has since
been replaced cannot authorize the replacement.

The client's response is prescribed:

```text
submit(command):
    response = generated_mutator(command)  # idempotency key +
                                           # workflow_revision precondition
                                           # (+ action_revision on approval)
    if conflict(response):                 # typed 409
        surface(typed_conflict)            # not a generic failure
        state = refetch(authoritative_read_model)
        re_present(decision, state)        # the human decides again
```

The client **never retries blindly and never silently re-submits**. A
stale precondition is an answer, not a transient failure: resubmitting it
unchanged either fails again or, worse, succeeds against a condition
nobody reviewed. Rendering staleness is a separate concern carried by
`projection_revision`, which protects rendering only and never protects
Command correctness.

## Command Idempotency, from the Client Side

**Every mutating operation carries an idempotency key**, and repeated
submission returns the original semantic outcome. Two consequences bind
the browser:

- The key is passed through the generated mutator that declares it, never
  assembled at a hand-written call site.
- **A retried response must not be rendered as a second effect.** The
  replayed outcome is the first execution's outcome; showing it as a fresh
  one tells the user something happened twice when it happened once.

This is Command idempotency — the rule governing externally issued
Commands. The wider identity model that also governs delivery
deduplication and action execution is owned by the cloud-platform doctrine; the browser's obligation is the single rule above.

## The Interface Never Implies an Action Occurred

Connection, loading, and error states are rendered honestly, and there is
no animated fake success. After a Command succeeds, the interface surfaces
the **persisted receipt** rather than an optimistic flourish.

The underlying rule is an authority rule, not a styling preference:
**success is reported by the system that performed the side effect.** An
agent or model never infers success merely because it issued a tool call,
and the interface never renders an inference as a fact. Where no receipt
exists, the honest state is "not yet confirmed" — never "done".

## Error Normalization at the Client Boundary

- **The generated error types are consumed, not re-declared by hand.**
  A parallel hand-authored error taxonomy is contract duplication with the
  same failure mode as a hand-written fetch wrapper (the generated-clients
  reference owns that prohibition).
- **An unrecognized error code degrades to a generic, honest failure
  state.** A client that guesses at an unknown code invents semantics the
  contract never granted it. Unknown means unknown, rendered as such.
- **Three failure kinds stay distinguishable to the user.** A transport
  failure (the request never got an answer), a contract-level failure (the
  request was malformed or unauthorized), and a business-rule rejection
  (the request was understood and refused) call for different user
  actions, so collapsing them into one message destroys the only
  information the person needed.

## Where the Rest Lives

- The **OpenAPI consumption reference** owns the common schema set that
  carries this error model and the `contract_revision` evolution rules.
- The **generated-clients reference** owns the generated error types and
  the prohibition on re-wrapping them.
- The **frontend-authority** and **live-read-model** references own the
  honest-state discipline in the rendering path.
- The **cloud-platform doctrine** owns server-side mapping of typed
  application failures onto this shape, the retry, backoff, and delivery
  mechanics behind a dependency failure, and the full idempotency bucket
  model with its identity formulas. This profile consumes them and does
  not restate them.
- The validation procedures for this profile own the contract drift and
  generated-mock checks that keep the client's error handling aligned with
  the contract.

## Related

- [OpenAPI Consumption](../../contracts/reference/openapi_consumption.md)
- [Generated Clients](../../contracts/reference/generated_clients.md)
- [Contract-First Architecture](../../contracts/reference/contract_first_architecture.md)
- [Frontend Authority](frontend_authority.md)
- [Live Read Model](live_read_model.md)
