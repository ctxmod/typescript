# Generated-Code Discipline

Generated-code discipline — the four properties of generated artifacts, the two editing prohibitions, the committed-versus-regenerated posture, and deterministic generation from a pinned toolchain.

Generated code is always derived. It flows in one direction — contract
source, then generation, then generated code — and never back. This
reference owns what a generated artifact is, what may and may not be done
to it, and how a repository decides whether to commit it.

## The Four Properties

Every generated artifact carries exactly four properties. They are stated
together because each one is misread when taken alone.

| Property | What it means |
|---|---|
| **Reproducible** | The artifact is regenerable from its contract source by a deterministic generation step; the same inputs produce the same output. |
| **Non-authoritative** | The contract source is the single source of truth. A generated artifact never becomes one, however carefully it is reviewed. |
| **Committable** | Generated output may be committed for ergonomic and build reasons. Committing it does not make it source. |
| **Drift-checked** | CI regenerates the output and rejects drift: committed generated output must equal fresh regeneration. |

The pairing that matters most is committable with drift-checked. Output
that is committed but not drift-checked silently becomes a second source
of truth the moment someone edits it; output that is drift-checked but
treated as authoritative inverts the direction of the pipeline.

## Two Prohibitions

1. **Generated code is never manually edited.** Not to fix a type, not to
   add a convenience helper, not "just this once until the contract
   catches up". An edit to generated output is erased by the next
   regeneration at best, and survives as undetectable divergence at
   worst.
2. **The path from generated code through manual modification to a new
   contract is prohibited.** A needed change goes into the contract
   source and is regenerated. Deriving a contract from an edited artifact
   reverses the authority relationship and produces a contract that no
   longer describes what any other consumer generates.

Everything a developer is tempted to add by editing generated output has
a legitimate home: a wrapper is a view model or composition code, a fix
is a contract change, a missing operation is a contract addition.

## "Disposable" Is Retired

Generated output is deliberately not described as disposable. The word
reads as a contradiction against committable output, and it invites the
opposite of the intended handling — a directory that can be deleted
casually is also one that can be edited casually. **Committable** and
**regenerable** are the operative properties. The half of the older
phrasing that survives verbatim is "never hand-edit".

## Committed Output versus Regenerate-on-Clone

Two postures are legitimate; a repository picks one per generated tree
and states it.

- **Committed.** The generated tree is tracked, so a fresh clone builds
  without a generation step and reviewers can read the diff a contract
  change produced. The cost is that every contract change carries a
  generated diff through review.
- **Regenerate-on-clone.** The generated tree is untracked and produced
  by a setup or build step. The cost is that nothing in the repository
  shows a reviewer what the contract change did to consumers.

Both postures are subject to the drift check; neither weakens the
non-authoritative property.

One consequence has bitten repositories that mix the two: **any CI
mirror of an untracked generated file carries a "must change with it"
contract.** Where a build or workflow definition restates the contents of
a generated file because the file is not in the tree, the duplicate is a
documented, marked duplication with an explicit obligation to change in
lockstep with its source. An undocumented mirror is a second source of
truth wearing a build script's clothes.

## Deterministic Generation and a Pinned Toolchain

Reproducibility is a property of the toolchain, not a hope about it.

- The generator, the linter, and the bundler are pinned in lockfiles, so
  every developer and every CI run resolves the same tool versions.
- Generation is invoked through the repository's package manager against
  those pinned versions, never against a globally installed tool whose
  version varies by machine.
- Generator configuration lives in the repository next to the contract
  source and is reviewed with it. A configuration change is a contract
  change in effect, because it changes what consumers receive.
- The bundled contract artifact is retained as a reviewed build artifact,
  so the exact input to generation is inspectable after the fact.

An artifact that regenerates differently on two machines fails the
reproducible property, and the fix is always in the toolchain pinning —
never in editing the output until the machines agree.

## Where the Rest Lives

- The **drift gate mechanics** — when the regeneration check runs, what
  it compares, and how a failure is reported — are a validation
  procedure, owned by
  [Contract Drift Gates](contract_drift_gates.md). This
  reference states the principle that drift is rejected; it does not
  describe the gate.
- The **contract-first architecture reference** owns the handwritten-
  versus-generated split and contract-first repository placement.
- The **generated-clients reference** owns what specifically is generated
  for the browser and how it is consumed.
- The **OpenAPI consumption reference** owns the contract source shape
  and the lint, bundle, and generate pipeline in semantics.

## Related

- SSOT (engineering module, `principles` profile: `design/reference/ssot.md`)
- [Contract-First Architecture](contract_first_architecture.md)
- [Generated Clients](generated_clients.md)
- [OpenAPI Consumption](openapi_consumption.md)
