# Generated Mock Validation

Generated-mock validation — contract-generated mock handlers driving every honest interface state, idempotency and stale-workflow_revision mutation checks, honest-success assertions, deterministic Projection-update ordering, and keyboard and accessibility smoke coverage.

Procedure for proving the browser interface behaves honestly in every state it can render, driven by mock request handlers generated from the same bundled contract as the client. Run these checks when a contract change, a change to view composition, or an interface claim needs them, and wherever the repository's test policy places them (the mock-driven UI leg of a CI gate, a hook, or explicit invocation); a claim about the real built application is established by a check that drives the built application from outside. The checks run offline: handlers intercept at the transport boundary, and nothing reaches a network or a credential.

## Prerequisites

- Mock handlers are a generation target of the client generator (Orval producing MSW handlers), regenerated in the same step as the client, query hooks, and mutators — never maintained apart from them.
- The mock tree sits inside the drift gate's target list.
- Whatever form checks the interface mounts real view composition — not a flattened stand-in for it — against the generated hooks and mutators.

- [Generated Clients](generated_clients.md)

## Validation Workflow

### 1. Confirm the mocks are generated, never shaped

Every request and response shape in the suite comes from the generated handlers. A hand-shaped mock payload is a second author of the wire shape — contract duplication with the same failure mode as a hand-written fetch wrapper — and is prohibited on the same terms. A mock fixture that lags the contract is drift: it fails the same gate, and the remedy is regeneration.

- [Contract Drift Gates](contract_drift_gates.md)

The boundary with test-data partials is the transport: hand-shaped **wire** payloads stay prohibited (the generated handler is the only author of a wire shape), while `fromPartial`-style partials serve **unit-test function arguments** where no wire shape exists. The two rules are complements, not exceptions: the generated handler owns every shape crossing the transport; a partial builder owns an argument object the test constructs under its own control.

- [Test-Data Partials](../../testing/reference/test_data_partials.md)

### 2. Drive every honest state class

Enumerate states as rendering responsibilities, not as one product's screens. Every state the interface can honestly render must be reachable from a mock-driven test and asserted as a distinct rendering:

| State class | The test proves |
|---|---|
| Initial loading | Skeleton placeholders are tied to query state and clear exactly when the query resolves — never on a timer |
| Loaded | The authoritative read model renders through the generated query hook, with no local stand-in substituting for it |
| Empty | A genuine zero-result renders as empty, not as loading and not as an error |
| Partial or degraded | A response carrying less than the complete shape renders what is true and marks what is missing, rather than guessing |
| Contract-level failure | A typed problem-details response — malformed or unauthorized — renders as its own state |
| Business-rule rejection | A request understood and refused renders as a distinct state; collapsing it into the row above destroys the only information the person needed |
| Typed stale-`workflow_revision` conflict | The typed 409 renders as a typed conflict with refetch and re-presentation, never as a generic failure or a silent retry |
| Transport failure | No answer received renders as an unknown outcome, never as a failure of the operation itself |

The three failure kinds stay distinguishable because they call for different actions, and a mock-driven suite is the only place all three can be produced on demand.

- [Problem Details](../../frontend/reference/problem_details.md)

### 3. Run the mutation tests

Every mutating operation carries a mandatory pair:

- **The idempotency test** submits the same Command twice under the same idempotency key and asserts the second submission returns the original semantic outcome with no second effect — and that the interface does not render the replayed outcome as a fresh one.
- **The stale-`workflow_revision` mutation test** submits against a superseded `workflow_revision` precondition and asserts a typed 409 conflict response rather than an incorrect effect. An approval operation additionally asserts its `action_revision` precondition, so an approval granted against a proposal since replaced cannot authorize the replacement.

Name these stale-`workflow_revision` mutation tests. The identifier is the assertion: a shorter name hides which precondition moved. The wider identity model — Domain Event delivery deduplication and action execution — belongs to the cloud-platform doctrine; the browser's obligation is the pair above.

### 4. Assert honest success

- The interface never renders success without a persisted receipt; where none exists, the rendered state is "not yet confirmed".
- The interface never implies an action occurred when it did not — asserted across the degraded and failure states, not only the success path. No success affordance survives into the transport-failure or conflict state.

### 5. Drive real-time state deterministically

Read-model-driven views are tested with scripted Projection-update sequences played through the subscription path: an ordered run of updates with increasing `projection_revision`, then a deliberately out-of-order update whose older `projection_revision` must be discarded rather than rendered. Assert the discarded update changed nothing visible — the interface did not walk backwards through states the workflow has already left — and that the subscription is torn down when the view unmounts.

- [Live Read Model](../../frontend/reference/live_read_model.md)

### 6. Run the keyboard and accessibility smoke pass

Keyboard traversal, focus, and contrast smoke coverage runs over the major states — loading, loaded, empty, each failure state, and the approval moment — against mock-driven state, so the pass covers the states a person actually meets rather than only the happy path.

## Thresholds

| Check | Pass | Fail |
|---|---|---|
| Mock provenance | Every shape drawn from generated handlers | Any hand-shaped request or response object |
| State-class coverage | Every class reachable and distinctly asserted | A class unreachable, or two collapsed into one rendering |
| Mutation pairing | Every mutating operation carries both tests | Either half missing |
| Honest success | Success rendered only with a receipt | Any success rendering without one |
| Projection ordering | An older `projection_revision` is discarded | Any backwards render |
| Mock currency | Committed mocks equal fresh regeneration | Any stale mock fixture |

## Troubleshooting

**A state class cannot be produced from the mocks**: the handlers cannot express that response. That is a contract finding — if the contract cannot express the state, the interface cannot render it honestly either. Extend the contract and regenerate.

**The suite passes against a stale mock**: currency is the drift gate's job. Add the mock tree to the gate's target list.

**The out-of-order update renders**: the discarding comparison must run before the update reaches anything visible.

- [Generated-Code Discipline](generated_code_discipline.md)
