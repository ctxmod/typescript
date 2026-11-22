# Dependency Analysis

Validate ECMAScript dependency patterns, circular imports, and export encapsulation.

Validate dependency health in ECMAScript/TypeScript projects: detect circular imports, verify export encapsulation, identify unused exports, enforce workspace protocol usage, and check for dual package hazards.

- [Module Organization](module_organization.md)

## Checks

Six checks compose the full dependency analysis. Four are automated; two are semi-automated.

| Check | Method | Tool | Pass | Warn | Fail |
|-------|--------|------|------|------|------|
| Circular imports | Automated | `madge --circular` | 0 cycles | - | Any cycle |
| Unused exports | Automated | `knip` | 0 unused | 1-5 unused | >5 unused |
| Workspace protocol | Automated | Config inspection | All internal deps use `workspace:` | - | File paths or version ranges for internal deps |
| Export encapsulation | Automated | `attw` + manual | All imports through `exports` | - | Deep imports bypassing `exports` |
| Dual package hazard | Semi-automated | Manual inspection | Single module instance | - | Separate ESM/CJS instances with state |
| Dependency weight | Semi-automated | `bundlephobia` / `pkg-size` | Acceptable bundle size | Large transitive deps | Bloated dependency tree |

## Automated Checks

### Circular Import Detection

`madge` analyzes the import graph and reports cycles:

```bash
npx madge --circular --extensions ts src/
```

For monorepos, run per package:

```bash
pnpm -r exec npx madge --circular --extensions ts src/
```

Any cycle is a hard failure. Circular imports cause initialization order problems in ESM: a module importing from a cycle may receive `undefined` for bindings that have not initialized yet. Unlike CommonJS, where circular requires produce partially populated objects, ESM circular imports fail silently with temporal dead zone violations.

**Remediation**: Extract shared types or functions into a third module that both sides import. Common extraction targets: type definitions, constants, utility functions, and interface declarations. If two modules share complex bidirectional logic, they likely belong in a single module.

### Unused Export Detection

`knip` identifies exported symbols with no consumers:

```bash
npx knip
```

Knip analyzes source files, configuration, and entry points. Configure entry points and ignore patterns in `knip.json`.

One entry class is invisible to the default detection: entry points referenced only from GitHub Actions workflow `run:` steps. The supported shape is a repo-root `knip.json` declaring them under `workspaces."<workspace>".entry` — root placement is what knip's configuration discovery reads, and keeping the file committed means fresh clones analyze the same surface.

**Pass**: Zero unused exports.

**Warn**: 1-5 unused exports may be intentionally preserved for future consumers or public API completeness.

**Fail**: More than 5 unused exports signals API surface bloat. Audit each export: remove those without consumers, document those preserved intentionally.

### Workspace Protocol Enforcement

In monorepos, all internal package references should use `workspace:*`:

```json
{
  "dependencies": {
    "@scope/utils": "workspace:*",
    "@scope/ui": "workspace:*"
  }
}
```

Scan `package.json` files for internal packages referenced with version ranges instead of `workspace:` protocol. Version ranges resolve from the registry rather than the local workspace, causing version mismatches.

**Pass**: All internal dependencies use `workspace:*` or `workspace:^`.

### Export Encapsulation

Verify that consumers access packages only through declared `exports` paths. Deep imports bypassing the `exports` field break when packages reorganize internal files.

Run `attw` to verify export resolution, then search for deep imports in consuming packages:

```bash
# Find imports reaching into package internals
grep -rn "from '@scope/utils/src/" apps/ packages/
grep -rn "from '@scope/utils/dist/" apps/ packages/
```

Any import referencing `src/` or `dist/` paths directly (rather than declared subpath exports) is a failure. These imports bypass the `exports` contract and break on internal reorganization.

## Semi-Automated Checks

### Dual Package Hazard

When a package publishes both ESM and CJS formats, both copies may load in the same Node.js process. This creates separate module instances with independent state.

Identify packages at risk:

1. Package has both `import` and `require` conditions in `exports`
2. Package maintains module-level state (caches, registries, singletons, event emitters)
3. Consumers include both ESM and CJS modules in the same application

**Pass**: Package is ESM-only, or package is stateless, or package uses an ESM wrapper over single CJS implementation.

**Fail**: Stateful package publishes independent ESM and CJS implementations.

**Remediation**: For stateful packages, use the ESM wrapper pattern: implement in CJS, create a thin ESM re-export layer. Alternatively, publish ESM-only and rely on `require(esm)` support in Node.js 22+.

### Dependency Weight

Evaluate the size impact of dependencies using `pkg-size` or `bundlephobia`:

```bash
npx pkg-size .
pnpm ls --depth 3
pnpm why lodash
```

Large transitive dependencies imported for a single utility function warrant replacement with targeted implementations or lighter alternatives.

## Integration

Run dependency analysis in CI alongside build and type validation:

```yaml
steps:
  - run: npx tsc --noEmit
  - run: npx madge --circular --extensions ts src/
  - run: npx knip
  - run: pnpm build
  - run: npx @arethetypeswrong/cli --pack .
```

Place circular import detection early in the pipeline. Cycles indicate structural problems that affect all downstream checks.

- [TypeScript Validation](../../toolchain/reference/typescript_validation.md)

## Complementary boundary tooling

Dependency-cruiser enforces the complementary source-tree boundary concern — a package's entry points public, its subfolders private — and its `no-circular` rule substitutes for `madge --circular` where boundary rules are already wired, while `knip` keeps the unused-export role either way.

- [Dependency Boundary Enforcement](dependency_boundary_enforcement.md)

## Related

- [Project Setup](../../toolchain/reference/project_setup.md)
