# Scratch Probes

Write throwaway TypeScript probes as .mts files in /tmp and run them with node directly, falling back to the repository's local tsx CLI when the code needs a full transform.

A scratch probe is a throwaway TypeScript file an agent runs once to check
how a library, a module, or a runtime behaves. The probe answers a question
and is then discarded; it is never part of the change. Linux and macOS only.

## Symptoms this prevents

- `npx tsx probe.ts` fails with `npm error Missing script: "tsx"`.
- A probe using top-level await fails with
  `Top-level await is currently not supported with the "cjs" output format`.
- Probe files land in the repository as stray, untracked artifacts and get
  swept into a commit or a lint run.

## Rules

- First choice: run the probe with node itself — `node /tmp/probe.mts`.
  Every current Node.js LTS line (22.18+, 24, 26) strips erasable
  TypeScript natively, so a probe needs no loader when it and every
  repository file it imports use only erasable syntax and explicit file
  extensions. Node never strips types under `node_modules`, and it rejects
  `enum`, value namespaces, parameter properties, and decorators with
  `ERR_UNSUPPORTED_TYPESCRIPT_SYNTAX`.
- When the probe or the source it imports needs a full transform
  (non-erasable syntax, extensionless relative imports, path aliases), run
  it with node against the repository's local tsx CLI module:
  `node /abs/repo/node_modules/tsx/dist/cli.mjs /tmp/probe.mts`.
- Invoke the CLI module by path. An absolute path to
  `node_modules/tsx/dist/cli.mjs` works from any working directory.
- Do not use `npx tsx`. In an agent session the command-rewrite proxy
  routes `npx` to `npm run`, so `npx tsx` resolves as a missing package
  script and the probe never runs. The `node_modules/.bin/tsx` wrapper is
  equally fragile under the rewrite proxy; invoke node with the CLI module
  path instead.
- Name probes `.mts`, never `.ts`. In a package whose `type` is
  `commonjs` (the value `npm init` now writes), tsx transforms a `.ts`
  file to CommonJS (CJS), and top-level await fails under that CJS
  transform. A `.mts` file is always ESM, whatever the package type.
- Keep probes in `/tmp`, never inside the repository, so no probe file
  lands in the working tree, the index, or a commit.

## Imports

- Import repository source by absolute import path:
  `import { add } from "/abs/repo/src/lib.ts";`.
- Caveat: a bare package specifier such as `import "esbuild"` fails from a
  `/tmp` probe with `ERR_MODULE_NOT_FOUND`. Node resolves packages relative
  to the probe file, not the working directory.
- To load a dependency, import its entry point by absolute path into the
  repository's `node_modules`, for example
  `/abs/repo/node_modules/<pkg>/<entry>`; read the entry from the
  package's `package.json` `exports` or `main` field.

## Cleanup

- Delete the probe from `/tmp` when the question is answered; record the
  finding in the report, not in a file beside the source.

## Related

- [Project Setup](project_setup.md)
- [Import and Export Conventions](../../modules/reference/import_export_conventions.md)
- [Local Server Lifecycle](../../testing/reference/local_server_lifecycle.md)

## Sources

- https://nodejs.org/api/typescript.html — type stripping stable and enabled by default (v22.18.0/v23.6.0; stable v24.12.0/v25.2.0), unsupported syntax, no stripping under `node_modules`, mandatory extensions (Node.js v26.13.1 docs; accessed 2026-11-22)
