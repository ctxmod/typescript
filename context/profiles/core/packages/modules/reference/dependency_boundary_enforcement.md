# Dependency Boundary Enforcement

Wiring dependency-cruiser so each TypeScript package is a deep module — entry points public, subfolders private, tests through entry points, no cycles — and proving the rules bite with a pass-fail-pass check.

Make every package in a TypeScript repository a **deep module**: much behaviour behind a small interface. A package's public surface is its **entry points** — the files at the package root — and everything in its subfolders is private. This procedure installs [dependency-cruiser](https://github.com/sverweij/dependency-cruiser) with the rules that make entry points the only way in, then proves the rules bite. The design vocabulary — why packages are deep at all — is [Deep Module Design](deep_module_design.md); publishing-level encapsulation through the `exports` field is [Module Organization](module_organization.md).

## The shape this enforces

```
src/packages/
  <name>/
    index.ts        ← an entry point (public). Import this from outside.
    client.ts       ← another entry point. Packages may expose several.
    lib/            ← implementation: private, free to import its siblings.
    tests/          ← co-located tests + fixtures (a subfolder, so private).
```

The public surface is the package's root files, not one designated `index.ts`: a package exposes as many small entry points as its consumers need. Implementation lives in `lib/` and tests in `tests/` by convention, and the rule itself is more general — anything in any subfolder is private, so a new folder never needs a config change.

Four rules, all `error` severity:

1. **Entry-point boundary**: code outside a package — application code or another package — imports only that package's entry points, never anything in its subfolders.
2. **Intra-package freedom**: a package's own files import each other freely.
3. **Tests through the entry points**: files under a package's `tests/` import any package's entry points and their own fixtures, and never any package's subfolder internals — not even their own package's.
4. **No cycles**: the dependency graph is acyclic.

**Entry points, not a barrel.** Several small entry points replace one re-exporting `index.ts`; barrel files that re-export a whole subtree are discouraged — keep entry points small and hide implementation in subfolders. Layering — which packages may depend on which — is a different concern and stays a commented stub in the config until the repository needs it.

## Steps

### 1. Detect the environment

Detect the package manager from the lockfile (`pnpm-lock.yaml` → pnpm, `yarn.lock` → yarn, `bun.lockb` → bun, else npm) and use it for every command below. Detect the packages root: `src/packages` when `src/` exists, else `packages`; confirm the choice with the requester when the repository already follows a different obvious convention. Check for an existing `.dependency-cruiser.*` config: when one exists, merge the four rules and the options into it rather than overwriting, and report what was added.

Done when package manager, packages root, and existing-config status are all decided and recorded.

### 2. Install dependency-cruiser

Install `dependency-cruiser` as a devDependency with the detected package manager.

Done when `dependency-cruiser` appears in `devDependencies`.

### 3. Write the config

Write the config from the block below to the repository root as `.dependency-cruiser.cjs` — `.cjs`, not `.js`, so `module.exports` works even in `"type": "module"` repositories — and set `PACKAGES_ROOT` to the root detected in step 1. It is the only edit point: the rules are path-depth based and extension-agnostic, so nothing else needs adapting. When merging into an existing config, carry the `forbidden` rules and the `options` block in.

Done when `.dependency-cruiser.cjs` exists with the correct `PACKAGES_ROOT` and all four forbidden rules present.

### 4. Wire it into the checks

Add a `lint:boundaries` script running `depcruise <packages-root>` (or `depcruise src`), and fold it into the repository's umbrella check command — the one that already runs typecheck. Leave `tsconfig` and path aliases untouched. When no umbrella script exists, keep `lint:boundaries` standalone and tell the requester to include it in CI.

Done when `lint:boundaries` exists and runs as part of the same command as typecheck.

### 5. Scaffold the example package

Create a committed `<packages-root>/example/` as a copy-me template: `index.ts` is an entry point exporting one function that delegates to an internal file, so the package is visibly deep rather than a pass-through; `lib/impl.ts` is the internal file, imported by `index.ts` and unreachable from outside; `tests/example.test.ts` imports only `../index` and asserts against the public function. Tell the requester this is a starter template to copy or delete.

Done when the example package exposes its behaviour through a root entry point and hides the implementation in a subfolder.

### 6. Prove the rules bite

A config that has never failed on a violation proves nothing. Run `lint:boundaries` on the clean example — it must pass. Temporarily add a deep import to `tests/example.test.ts` (`import { thing } from "../lib/impl"`) and run again — it must fail with `tests-through-entrypoints`. Revert the import and run once more — it must pass.

This is deliberate verification in miniature: observe the check fail on the seeded violation, never trust a green exit status alone — the same spirit as deliberate verification at repository level: evidence comes from an observed outcome, not an exit code.

Done when a pass, then a failure on the deep import, then a pass have all been observed. A silent second step means the rules are miswired — fix before finishing.

### 7. Document the convention

Write a `README.md` in the packages folder — `<packages-root>/README.md`, next to the packages it governs — covering the layout (entry points at the root, `lib/` for implementation, `tests/` for tests), the import rule (only through a package's entry points), and how to run `lint:boundaries`, and discouraging barrels explicitly. Then add a context pointer to it from the repository's agent-instructions convention: the repo's steering file — the project README, agent-instructions file, or equivalent agents read first; in ctxmod-managed repositories the module's `context/` tree is that surface. One line is enough: "Packages are deep modules: see `<packages-root>/README.md` before adding or importing one." This is what makes an agent discover the boundary rule instead of tripping over it.

Done when the README exists and discourages barrels, and the agent-instructions surface links to it.

## The config

Minimal but complete: all four rules, the `$1` back-reference grouping that gives each package its intra-package freedom, and the optional layering stub. `PACKAGES_ROOT` is the single edit point.

```javascript
// Deep-module enforcement for dependency-cruiser. Each package under the
// packages root is a deep module: its public surface is its entry points
// (the root files); subfolders are private. PACKAGES_ROOT is the only
// edit point.
const PACKAGES_ROOT = "src/packages";

const R = PACKAGES_ROOT;
// Anything nested inside a package subfolder; root files stay public.
const PACKAGE_INTERNALS = `^${R}/[^/]+/[^/]+/`;

/** @type {import('dependency-cruiser').IConfiguration} */
module.exports = {
  forbidden: [
    {
      // outside code reaches packages through entry points only
      name: "entrypoint-boundary-from-app",
      severity: "error",
      from: { pathNot: `^${R}/` },
      to: { path: PACKAGE_INTERNALS },
    },
    {
      // a package's own files import each other freely
      name: "entrypoint-boundary-across-packages",
      severity: "error",
      from: { path: `^${R}/([^/]+)/`, pathNot: `^${R}/[^/]+/tests/` },
      to: { path: PACKAGE_INTERNALS, pathNot: `^${R}/$1/` },
    },
    {
      // tests exercise packages through entry points like everyone else
      name: "tests-through-entrypoints",
      severity: "error",
      from: { path: `^${R}/([^/]+)/tests/` },
      to: { path: PACKAGE_INTERNALS, pathNot: `^${R}/$1/tests/` },
    },
    {
      // a package's tests/ folder is reachable only from tests
      name: "tests-folder-is-private",
      severity: "error",
      from: { pathNot: `^${R}/[^/]+/tests/` },
      to: { path: `^${R}/[^/]+/tests/` },
    },
    {
      name: "no-circular",
      severity: "error",
      from: {},
      to: { circular: true },
    },
    // Layering (which packages may depend on which) is a different
    // concern; add rules when the repository needs them, e.g.
    // { name: "ui-not-from-billing", severity: "error",
    //   from: { path: `^${R}/ui/` }, to: { path: `^${R}/billing/` } },
  ],
  options: {
    doNotFollow: { path: "node_modules" },
    tsConfig: { fileName: "tsconfig.json" },
    enhancedResolveOptions: {
      extensions: [".ts", ".tsx", ".js", ".jsx", ".json"],
    },
  },
};
```

## Notes

- The `$1` back-references are dependency-cruiser's group matching: they let a package reach its own internals while outsiders cannot. Keep them; flattening into per-package rules loses the grouping.
- Public versus private is decided by depth, not folder name: root files are entry points, any subfolder is private, and adding an entry point is just adding a root file.
- Packages are flat: one tier of immediate children under the packages root. A package's internals nest as deep as needed; a package never contains another package.

Dependency-level validation — cycle detection, unused exports, export encapsulation — is [Dependency Analysis](dependency_analysis.md).
