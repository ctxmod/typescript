# Guardrail Placement

Where safety guards live in a TypeScript application — refusal at the write site, pure renderers, validating entry points, reported opt-outs, staleness checks that stay armed, end-to-end invariants, and coverage measured by intersection.

A guard protects data only where the data changes. Place every refusal on
the path that writes, keep the paths that show data free of verdicts, and
prove the guard works by asserting the invariant it exists to protect.

## Refuse at the Write Site

- A safety refusal lives at the write site, not in the display layer.
  Hiding a button or greying out a row is presentation; the function that
  persists, sends, or mutates is where an unsafe operation is refused.
- A write path that relies on the display layer to have filtered its input
  is unguarded: any other caller — a script, a job, a second screen —
  reaches the write with nothing in the way.

## Keep Renderers Pure

- Renderers stay pure: they map data to output and decide nothing that a
  write depends on.
- Write-path code never imports verdicts from a renderer. When a write
  needs the same judgement a view shows, move that judgement into a shared
  module both import; the dependency runs from renderer and writer toward
  the rule, never from writer to renderer.

## Validate at Entry Points

- Entry points validate: every place outside input enters the application
  — a request handler, a CLI command, a message consumer, a file loader —
  checks its input before anything downstream trusts it.
- Opt-outs are explicit and reported. A flag that skips a guard is named,
  passed deliberately, and logged or surfaced in output every time it
  takes effect; a silent bypass is indistinguishable from a broken guard.

## Staleness Checks Stay Armed

- A stale-data check compares the current value against a prior one. When
  no prior value exists, the check must fail closed or report the gap —
  it never self-disables by treating "nothing to compare" as "fresh".
- The first run, a cleared cache, and a renamed key all produce an absent
  prior value; each is exactly when the check matters most.

## Assert the Invariant End to End

- A guard can be fed by an upstream bug: a parser that drops records, a
  mapper that defaults a field, a query that returns a subset. The guard
  then passes on wrong input.
- Assert the invariant end to end — from the source data to the final
  written state — not the parse. A test that only checks the parsed shape
  proves the parser, not the guarantee.

## Measure Coverage by Intersection

- When a replacement set supersedes held data, measure retained coverage
  by intersecting the replacement with the held data, key by key.
- Measure it never by net or aggregate quantities. Equal totals can hide
  entries that were dropped and replaced by unrelated new ones —
  displacement is not refinement.

## Fix the Rule, Never the Fixture

- When a guard rejects a test fixture that represents real input, the rule
  is wrong; fix the rule. Editing the fixture until the guard passes hides
  the defect from every later reader.

## Related

- [Contract-First Architecture](../../contracts/reference/contract_first_architecture.md)
- [Module Organization](../../modules/reference/module_organization.md)
- [Import and Export Conventions](../../modules/reference/import_export_conventions.md)
- [Frontend Authority](frontend_authority.md)
