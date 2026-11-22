# Test Effectiveness

A guardrail test counts as coverage only once it has been shown able to fail.

- A test counts as coverage only once it has been shown able to fail, by mutation or
  red-first (watch it fail before the fix, then pass after it).
- Never assert schema validation under a guard's name; a guard test exercises the guarded behaviour.
- Keep one invariant per test, so a failure names exactly what broke.
- A verification number that nothing checks is a defect: assert it or drop it.
- Mutation method: take a `git archive` scratch copy, mutate the guard there, run the test,
  and expect red. A test that stays green guards nothing.
- Mutation verification is a heavy, opt-in stage: run it when selected on its own, never as
  part of the light per-change check.
- The doctrine owner is the negative-controls reference in the agentic module's verification
  doctrine; this page restates it for ECMAScript/TypeScript tests.

## Related

- [Generated Mock Validation](../../contracts/reference/generated_mocks.md)
- [Contract Drift Gates](../../contracts/reference/contract_drift_gates.md)
