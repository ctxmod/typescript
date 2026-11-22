# Contract Drift Gates

The contract drift gate as a procedure — lint, bundle, regenerate, regenerate-again, byte-compare, fail on any difference; byte-stable regeneration under a pinned toolchain; the retained bundle artifact; the es-side legs and their execution; scenario replayability; and the es-side definition-of-done assertions.

Procedure for proving committed generated output is exactly what the contract source and the pinned toolchain produce. Drift is rejected, never warned about. Run this gate on every contract change, on every change under the generated tree, on every toolchain bump, and as the generated-diff leg of CI. It is offline and deterministic.

The principle the gate enforces — the four properties of generated output and the two editing prohibitions — is owned by the generated-code discipline reference. This procedure owns the mechanics.

[Generated-Code Discipline](generated_code_discipline.md)

## The Gate

Six steps, in order, each a hard failure point:

```text
gate(contract_source, committed_generated, targets):
    lint(contract_source)            # 1. contract quality rules; any finding fails
    bundle = bundle(contract_source) # 2. one portable artifact from the source graph
    regenerate(bundle, targets)      # 3. every language output, from the same bundle
    tmp = regenerate_to_temp(bundle, targets)   # 4. write nothing committed
    drifted = [f for f in outputs
               if bytes(tmp[f]) != bytes(committed_generated[f])]
    if drifted: fail(drifted)        # 5. per-file drifted list, non-zero exit
    retain(bundle)                   # 6. kept as a reviewed build artifact
```

Steps 3 and 4 are deliberately separate. Step 3 is the regeneration a developer runs and commits; step 4 proves, in a clean temporary location, what the pinned toolchain produces from the same bundle. The gate writes nothing back, so it can never repair the drift it found, only name it.

## Byte-Stable Regeneration

Determinism is a property of the gate, not a nicety beside it. The same contract source and the same pinned tool versions must produce identical output, byte for byte, on every machine and every CI run:

- **The comparison is a byte comparison**, not a semantic one. Any difference — a reordered field, a changed timestamp placeholder, trailing whitespace — is a failure. There is no tolerated diff.
- **The failure report is a per-file drifted list**, so the failing change is legible without re-running anything: which files drifted, and therefore which generation target moved.

Every language output — the browser client, query hooks, mutators, and mocks, and every other language's output — is generated from that same bundled artifact, so no two consumers can resolve the modular source graph differently and produce outputs that disagree.

## The Pinned Toolchain

The generator, the linter, and the bundler are pinned in lockfiles, and generation runs through the repository's package manager against those pins. The gate therefore proves something stronger than "the output is current": **committed generated output matches the pinned toolchain.**

A toolchain bump is itself a reviewed change that flows through the gate. Bumping the pin regenerates every target; the resulting diff is reviewed as the bump's effect; the gate then enforces the new baseline. An artifact that regenerates differently on two machines fails the reproducible property, and the fix is always in the pinning — never in editing the output until the machines agree.

## The Retained Bundle Artifact

The bundled contract is retained as a reviewed build artifact, displayed in the repository for inspection. Reviewers and CI consumers read one portable artifact instead of reassembling the modular source graph in their heads. The bundle is also the single input to every generator, which is what makes the comparison meaningful across targets.

## Scope Boundary: Conformance Is Not Generation

Generation proves the types match the contract. It does not prove the service does. Contract-conformance testing checks request and response behavior against the retained bundled artifact, so an implementation cannot drift from the contract even where generated types happen to line up.

**The conformance test procedure is owned by the python module's contract-testing validation procedure** (`github.com/ctxmod/python/context`, core profile, `contracts` package); this profile consumes that boundary and does not restate it. Likewise **the full CI gate list and the test-tier model are owned by the python module's testing philosophy reference** (core profile, `testing` package); this procedure covers only the es-side legs.

## The es-Side Legs

The generated-diff leg is a deterministic generation check and may run as part of the pipeline; the other two legs are test execution. Execution follows the repository's test policy: these checks run where it places them (a CI gate, a hook, or explicit invocation), and a claim about the real built application is established by a check that drives the built application from outside.

| Leg | Must prove | Execution |
|---|---|---|
| Generated-diff | Every committed generated artifact equals fresh regeneration from the bundle under the pinned toolchain | Deterministic generation check |
| Mock-driven UI checks | Generated mocks drive all major interface states; the keyboard and accessibility smoke pass holds | Test execution -- component-level tests or checks against the built application |
| Mutation checks | Every mutating operation carries an idempotency check and a stale-`workflow_revision` precondition check | Test execution -- component-level tests or checks against the built application |

The mock-driven leg is the generated-mock validation procedure; the pairing and state classes it asserts are specified there ([Generated Mock Validation](generated_mocks.md)).

## Scenario Replayability

Deterministic, visibly-labeled synthetic scenarios run through the real pipeline rather than a hard-coded animation standing in for its output:

1. **A synthetic scenario is a seeded, deterministic input sequence.** Replaying it produces the same observable outcome every time, which is what makes it a repeatable test rather than a demonstration.
2. **It drives the same public Command path production traffic drives.** A scenario that reaches the workflow through a test-only pathway tests that pathway, not the system. A scenario can be reset and replayed through the same public path.
3. **It is visibly labeled as synthetic in the interface.** An unlabeled synthetic state is indistinguishable from real state and will eventually be mistaken for it.
4. **Nothing is animated as a substitute for pipeline output.** Rendering follows Projection updates and receipts; a timed animation claiming progress the pipeline has not produced is the same dishonesty as an animated fake success.

## Definition of Done, es Side

The change is not complete until each assertion holds:

- No handwritten request or response type exists outside generated output.
- Every mutating operation has an idempotency test and a stale-`workflow_revision` precondition test.
- Committed generated output matches the pinned toolchain under the gate.
- Deterministic synthetic scenarios are replayable through the same public Command path.
- The interface handles connection, loading, and error states without implying an action occurred when it did not.

## Troubleshooting

**Drift with no contract change**: someone hand-edited generated output. Revert the edit and regenerate; never commit a hand-mended generated file.

**Drift on an untouched contract**: the toolchain moved. Re-pin the lockfile — the lockfile, not the local environment, selects tool versions — and regenerate as a reviewed change.

**Drift confined to the bundle**: the source graph resolved differently. Check for a remote or registry reference that crept in; the contract must bundle offline from the repository alone.

[OpenAPI Consumption](openapi_consumption.md)
[TypeScript Validation](../../toolchain/reference/typescript_validation.md)

## Related

- [Generated Clients](generated_clients.md)
