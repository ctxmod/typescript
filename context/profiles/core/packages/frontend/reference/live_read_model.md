# Live Read Model

The canonical browser data model — initial query to authoritative read model to real-time Projection updates, mutations through generated mutators only, transport-neutral subscriptions with no custom realtime infrastructure, stale projection_revision discarding, receipt-backed progress rendering, and subscription lifecycle.

The browser stays visibly current without owning any of the truth it
displays. It loads an authoritative snapshot, subscribes to Projection
updates for the thing the user selected, and issues every write through
the generated transport surface. This reference owns that data model and
the rules that keep it honest.

## The Canonical Browser Data Model

```text
initial query  ->  authoritative read model  ->  real-time Projection updates
```

As a flow:

```text
on select(workflow instance):
    snapshot = initial_query()              # generated query hook
    render(snapshot)                        # authoritative read model
    subscribe(projection_channel):          # read-only, platform-provided
        on projection_update(update):
            if update.projection_revision <= rendered_projection_revision:
                discard(update)             # out-of-order or redelivered
            else:
                render(update)
on unmount:
    unsubscribe(projection_channel)

mutate(command):
    generated_mutator(command)              # never the subscription
```

## Reads and Writes Are Different Channels

**Mutations go through the generated client** — the typed mutator for the
operation, carrying its idempotency key and its `workflow_revision`
precondition, plus its `action_revision` where the operation approves or
executes a specific proposed action. They
never travel over the subscription. The subscription is a **read channel
with no write capability**: it delivers Projection updates and accepts
nothing. Keeping the two channels separate is what makes server-side
authorization, preconditions, and Audit Record creation unavoidable for
every write.

## Transport-Neutral, With No Custom Realtime Infrastructure

The subscription is a **platform-provided read-model channel**. This
profile does not specify the transport, and the consequence is the point:
**no custom realtime infrastructure**.

- no bespoke socket server;
- no hand-rolled stream protocol;
- no application-authored reconnection logic;
- no polling loop.

Each of those is transport code the application would then own, test, and
keep correct under retries and redeliveries — for a capability the
platform already provides. The concrete subscription transport, and the
authorization applied to it, are owned by the cloud-platform doctrine.

## Stale Projection Updates Are Discarded

**The browser ignores a Projection update whose `projection_revision` is
older than the one currently rendered.** This is what prevents
out-of-order visual progress during retries and redeliveries: a redelivered
earlier update would otherwise walk the interface backwards through states
the workflow has already left.

The boundary is exact: **`projection_revision` protects rendering only.**
It never protects Command correctness — that is `workflow_revision`'s job,
carried as a precondition on the Command itself and answered with a typed
conflict on mismatch (the problem-details reference).

## A Projection Is Derived and Non-Authoritative

A Projection may lag, and it may be rebuilt. The interface is designed to
stay correct without trusting it as authority. Where a rendered Projection
and a Command result disagree, **the authoritative read model wins** and is
refetched — the Projection is never patched locally to match, and the
Command result is never discarded in favor of the older rendering.

## Loading State Follows Query State

**Skeleton placeholders are tied to query or read-model state**, never to
a timer or an animation clock. A skeleton driven by elapsed time claims
that data is arriving when nothing may be arriving at all, which is the
same dishonesty as an animated fake success.

## Progress Nodes Are Backed by Persisted Facts

**Every rendered progress node is backed by a persisted state transition
or a receipt** — never by a client-side inference about what probably
happened next. The Projection therefore carries a step-and-receipt
structure: an ordered set of steps, each with its own completion status
and a reference to the persisted transition or receipt that backs it,
alongside the workflow instance's current state, its `projection_revision`,
and what it is currently waiting on.

The shape is what matters, not any particular field naming. What the
structure guarantees is that a node cannot be rendered as complete unless
something durable says it is — so a progress display is factual rather
than decorative, and an unfinished step reads as unfinished rather than as
motion.

## Subscription Lifecycle

Subscribe when the view selects an instance; tear the subscription down
when the view unmounts. A subscription that outlives the view it feeds is
both a leak and a source of stale rendering: it keeps delivering updates
into state nobody is showing, and it can repaint a stale instance over a
newly selected one. One live view, one subscription, ended deliberately.

## Where the Rest Lives

- The **frontend-authority reference** owns the governing rule, the
  document classes, and the prohibition on reconstructing Workflow State.
- The **generated-clients reference** owns the generated query hooks and
  mutators this model reads and writes through.
- The **problem-details reference** owns the typed conflict, Command
  idempotency from the client side, and honest failure states.
- The **cloud-platform doctrine** owns the subscription transport,
  its authorization, and how Projection documents are written and rebuilt
  on the server side. This profile consumes the channel and does not
  specify it.

## Related

- [Frontend Authority](frontend_authority.md)
- [Generated Clients](../../contracts/reference/generated_clients.md)
- [Problem Details](problem_details.md)
- [OpenAPI Consumption](../../contracts/reference/openapi_consumption.md)
