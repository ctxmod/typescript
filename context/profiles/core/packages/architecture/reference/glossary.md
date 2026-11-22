# Architecture Glossary

The typescript and python modules share one capitalized architecture
vocabulary. The references use these terms without restating them; this
glossary is where they are defined. Capitalization marks the term of art:
"a Command" is the defined concept, "a command" is ordinary English. The
python module carries the same glossary, so a term means the same thing
on both sides of a contract.

## Writes, reads, and state

**Command** — an externally issued request that something be performed:
the write side of the API contract. A Command is submitted by a
principal (a client, an agent, a browser) and may be accepted, rejected,
or replayed; it never asserts that its effect happened. Mutating
endpoints accept an `Idempotency-Key` so a resubmitted Command is
recognized (see *Command idempotency*).

**Query** — a read against the API contract or the read model. Queries
never mutate state; interface and client state come from Queries or the
read model, never from reconstruction.

**Workflow State** — the authoritative state of a business workflow, held
in the authoritative state store and transitioned only by committed
transitions. It is never reconstructed from Domain Event or Audit Record
ordering, agent messages, local state, inferred transitions, or
animation. Its optimistic-concurrency precondition is the
`workflow_revision`.

**Domain Event** — an immutable record that something happened in the
domain, published to the Domain Event transport (the message bus) and
consumed by separately deployed runtimes. Domain Event contracts (for
example one `.proto` definition per event) are reviewed source, like the
API contract. Each event carries an immutable `domain_event_id`, the key
consumers deduplicate on.

**Projection** — a derived, non-authoritative read model built from
backend truth for a consumer to read. A Projection may lag and may be
rebuilt; where a Projection and a Command result disagree, the
authoritative read model wins and is refetched. Projection updates carry
a `projection_revision`; a stale (older) update is discarded, never
rendered. Frontend state is a Projection, never a second workflow
engine.

**Read model** — the published, query-side representation clients read
(through Queries and subscriptions), of which Projections are the
derived documents.

**Action** (proposed action) — a consequential side effect a workflow may
authorize, often proposed by an agent and approved by a principal. Each
has an `action_id` (its execution identity) and an `action_revision`
(the identity of one specific proposed action; where computed from the
action's content, the computation is the generic *precondition hash*).
The workflow owns when a side effect is allowed, the agent may propose
it, and the adapter reports whether it actually occurred.

**Receipt** — the durable record an adapter creates after validating an
external call's response. Success is established by response validation
and recorded as a receipt that reconciles back into Workflow State and
the Audit Record history — never inferred from having issued the call.

## Records and observation

**Audit Record** — one entry in the append-only Audit Record history:
what the system committed or attempted to commit — transition identity,
actor, timestamp, correlation and revision identifiers, receipts, and
approval references. It is written by the code path that commits the
transition. The history is the immutable record of committed
transitions; it is never Workflow State, never the operational source
for concurrency or queries, and never rebuilt or sampled.

**Telemetry** — observational records of execution. *Technical
Telemetry* covers latency, retries, errors, queue depth, and resource
use (structured log entries, tracing spans, metric series);
*agent-execution Telemetry* covers model and tool invocations, steps,
evaluation results, and decision outcomes. Telemetry never carries
business state and is never evidence that a transition committed.

**The three observability layers** — the Audit Record history, technical
Telemetry, and agent-execution Telemetry. None substitutes for another.

**Correlation identifier** (`correlation_id`) — the identifier that ties
one business transaction together across the HTTP request, Domain Event
processing in other runtimes, Telemetry, and the Audit Record that ends
it.

## Idempotency and preconditions

Idempotency is three distinct rules with three distinct identity
formulas, never collapsed into one concept:

| Level | Guards | Identity formula |
|---|---|---|
| **Command idempotency** | Resubmission of an externally issued Command | principal + operation + idempotency key |
| **Domain Event delivery deduplication** | Redelivery of a Domain Event to a consumer | the immutable `domain_event_id`, scoped to its consumer |
| **Action idempotency** | Double execution of a consequential action | `action_id`, valid against its `workflow_revision` precondition |

The delivery guarantee built from them is application-level
effectively-once side-effect semantics where required, through durable
deduplication records and idempotent adapters. No transport provides
exactly-once delivery.

**Revisions** — `workflow_revision` is the optimistic-concurrency
precondition on Workflow State; `action_revision` additionally pins a
specific proposed action; `projection_revision` orders Projection updates
and protects rendering only, never Command correctness. Revision
identifiers are preconditions, never deduplication keys. A stale
`workflow_revision` or `action_revision` fails with a typed 409 Conflict
problem-details response; the remedy is to reload and resubmit against a
fresh precondition, never to retry unchanged.

## Boundaries named but not owned here

**Cloud-platform doctrine** — deployment, infrastructure, state-store
mechanics (where deduplication records and approval state live), the
outbox/inbox delivery pipeline, retry and backoff dispatch, subscription
transport, and the platform side of the observability model. The
language modules point to it and restate none of it; it belongs to the
infrastructure and AI modules (`github.com/ctxmod/infra/context`,
`github.com/ctxmod/ai/context`) and to the platform a repository runs on.

**Agent-framework doctrine** — agent-facing schema design and
agent-execution Telemetry: what an agent run records, where, and with
what semantics. It belongs to the AI module.

**Server-side doctrine** — the services that publish the contracts the
browser consumes (typed errors, contract sources, the conformance suite,
the test-tier model): the python module (`github.com/ctxmod/python/context`).
