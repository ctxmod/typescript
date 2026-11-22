# Frontend Authority

The browser as consumer and Command issuer — the prohibition on reconstructing Workflow State, frontend state as a Projection rather than a second workflow engine, the three document classes and the designed-authorized-Projection consequence, the view-model allowance, and approval-moment, motion, and accessibility discipline.

**The browser is a consumer and a Command issuer.** It reads what the
backend publishes for it to read, and it asks that something be performed.
It is not a place where business truth is computed. This reference owns
that governing rule, the document classes it rests on, and the interface
discipline that follows from it.

## No Reconstruction of Workflow State

The browser must not reconstruct Workflow State from any of the following:

- Domain Event or Audit Record ordering;
- agent messages;
- local state;
- inferred transitions;
- animation state.

Each is a plausible-looking substitute for authority and each is wrong for
the same reason: it produces a value that no authoritative system agreed
to. All interface state comes from API queries or from the read model —
there is **no client-side reconstruction**.

## A Projection, Not a Second Workflow Engine

Frontend state is a Projection of backend truth, not a second workflow
engine. Concretely, the browser:

- does not re-implement transition rules;
- does not decide whether a transition is legal;
- does not compute what the next state should be.

A frontend that answers any of those questions has become a second
authority, and two authorities disagreeing is the failure the whole
architecture exists to prevent. The browser renders the transition the
backend committed, and offers the Commands the backend says are available.

## Document Classes

Three classes are distinguished, and the distinction is hard:

| Class | What it is | Exposure |
|---|---|---|
| Server-authoritative | The authoritative records the workflow transitions against | **Never directly exposed to the browser** |
| Browser-readable Projection | A read model deliberately designed as an authorized Projection for a client | Read by the browser, subscription-fed |
| Private execution data | Agent execution context and Agent Session Events | Never exposed to the browser |

The load-bearing point is the middle row. A browser-readable Projection is
**designed** as an authorized Projection — not the same authoritative
records with access rules applied over them. The consequence is worth
stating plainly: **the browser is not reading authority directly; it is
reading a designed, authorized Projection.**

The weaker reading — "the authoritative records, with rules" — fails
because it makes exposure a per-field access decision rather than a design
decision. Under it, a field gets exposed eventually simply because the
interface needs it, and authoritative state or sensitive agent context
leaks one convenience at a time. Designing the Projection makes the
question "does this belong in the client's read model?" rather than "can
we permit this read?".

## The View-Model Allowance

The no-handwritten rule is precise, not blanket: **never hand-author
models that duplicate an external transport contract.** It is not "no
handwritten models". View composition models, local state, form state,
derived UI state, and domain-specific presentation models are legitimate
handwritten code and are not contract duplication — they present or
assemble information rather than restating a wire shape. The allowance and
its semantic test are owned by the generated-clients reference.

## Temporary UI State Is Not Backend State

The browser may hold temporary state — a draft, a selection, an expanded
row, an in-flight indicator. That state must never be mistaken for backend
state: it is never treated as a source of truth, never used to decide what
a Command may do, and never allowed to outlive the authoritative answer it
was standing in for.

## The Approval Moment

The approval interaction is the point where a human takes responsibility,
so it is built deliberately rather than as a generic confirmation:

- **a clear action verb** naming what will happen, not a generic
  affirmative;
- **the exact audience or target** of the action;
- **a concise generated draft** with editable human notes;
- **evidence and policy references** alongside the decision;
- **an explicit impact statement**;
- **approve, reject, and revise** options;
- the approval's `action_revision` and its `workflow_revision`
  precondition **inspectable, though hidden by default**;
- **after approval, the persisted receipt** — never an animated fake
  success.

The typed conflict that a stale precondition produces, and the refetch
remedy, are owned by the problem-details reference.

## Motion, Visual, and Accessibility

- **Subtle transitions are reserved for real state transitions.** Motion
  is a signal that something changed in the backend, so decorative motion
  spends the signal on nothing.
- **Color is a redundant signal, never the sole indicator** of severity or
  status.
- **A reduced-motion mode is provided.**
- **Keyboard-navigation and contrast requirements are met.**
- **Skeleton placeholders appear only while the corresponding query or
  read model is loading** — never on a timer.
- **No pseudo-cinematic clutter and no continuously moving backgrounds.**
  A continuously moving surface trains the user to ignore motion, which
  disables the first rule above.

## Where the Rest Lives

- The **live-read-model reference** owns the read path itself: the initial
  query, the subscription, staleness discarding, and subscription
  lifecycle.
- The **generated-clients reference** owns the view-model allowance and
  the generated surface every read and mutation goes through.
- The **problem-details reference** owns the typed conflict, Command
  idempotency from the client side, and honest failure states.
- Server-side authorization of browser-readable Projections — how the
  authorized read model is designed, secured, and enforced — is owned by
  the cloud-platform doctrine. This profile stops at the browser
  boundary and does not specify it.

## Related

- [Generated Clients](../../contracts/reference/generated_clients.md)
- [Live Read Model](live_read_model.md)
- [Problem Details](problem_details.md)
- [Contract-First Architecture](../../contracts/reference/contract_first_architecture.md)
