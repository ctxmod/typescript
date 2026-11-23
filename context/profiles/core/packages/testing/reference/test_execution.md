# Test Execution

Iterate with the repository's npm scripts, know what a bare TypeScript module gets checked for, and scope claims to your own files.

## Iteration

- While a change is in progress, use the repository's own scripts:
  `npm test -- <file>` for one test file and `npm run typecheck` for types.
- These scripts give fast feedback on the code you are editing. When the
  repository provides a scoped check wrapper for reported results, the results
  you report come from that wrapper, scoped to the module or directory you
  changed; an unscoped run checks the whole repository.

## What a bare repo gets

A TypeScript module with no lint or format config of its own can still be
checked in four stages:

- file: a content scan.
- compile: `tsc` against the module's `tsconfig.json`.
- lint: only checks that need no config file, such as knip, syncpack and ts-unused-exports.
- format: the configured formatter (biome, oxfmt, or prettier) checks.

No test stage runs unless tests are explicitly selected.

Install dependencies before trusting such a run. With `node_modules` absent, `tsc`
fails, a lint stage that depends on compilation is skipped, and a summary can
still list compile as passed, with 0 issues and exit 0. Read the skipped stages
in the summary line before calling a run green.

A run that declares a test runner but resolves zero test targets is a skip, not
a pass: no test ran.

## Peer in-flight state

- In a shared working tree, sibling agents have edits in flight. A repo-global
  test or typecheck run reflects their in-flight state as well as yours.
- Scope every claim to your own files: run on the directories you changed, and
  treat a failure outside them as unattributed until you know whose edit caused it.

## Related

- [TypeScript Validation](../../toolchain/reference/typescript_validation.md)
- [Test Effectiveness](test_effectiveness.md)
