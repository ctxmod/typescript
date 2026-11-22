# Module Organization

Package structure and exports field design for ECMAScript/TypeScript projects.

Organize ECMAScript/TypeScript projects around explicit entry points, conditional exports, and clear separation between publishable packages and deployable applications.

- KISS (engineering module, `principles` profile: `design/reference/kiss.md`)
- SSOT (engineering module, `principles` profile: `design/reference/ssot.md`)

## Package Structure

Every package follows a consistent layout: source in `src/`, compiled output in `dist/`, entry points declared in `package.json` exports.

```
packages/my-lib/
├── src/
│   ├── index.ts          # Public entry point
│   ├── utils.ts          # Internal module
│   └── types.ts          # Shared type definitions
├── dist/                  # Compiled output (gitignored)
├── package.json
└── tsconfig.json
```

The `src/` directory contains all TypeScript source files. The `dist/` directory holds compiled JavaScript, declaration files, source maps, and declaration maps. Build tools generate `dist/` contents; never edit them directly.

## Exports Field Design

The `exports` field in `package.json` replaces `main` and `types` as the authoritative entry point declaration. It controls what consumers can import and hides everything else.

```json
{
  "type": "module",
  "exports": {
    ".": {
      "types": "./dist/index.d.ts",
      "import": "./dist/index.js"
    },
    "./utils": {
      "types": "./dist/utils.d.ts",
      "import": "./dist/utils.js"
    }
  }
}
```

Condition key order matters. Place `types` first so TypeScript resolves declarations before runtime conditions. Follow with `import` for ESM consumers, `require` for CJS consumers (when dual-publishing), and `default` last as fallback.

### Subpath Exports

Expose internal modules through explicit subpath exports rather than deep file imports. Subpath exports create a stable public API: internal file reorganization does not break consumers as long as the subpath mapping remains consistent.

Paths not listed in `exports` become inaccessible to external consumers, producing `ERR_PACKAGE_PATH_NOT_EXPORTED`. This encapsulation is intentional. Adding `exports` to an existing package can break consumers who relied on deep imports, so plan exports before the first publish.

### Self-Referencing

Packages can import their own exports by name. This validates that the `exports` field resolves correctly from within the package:

```typescript
import { helper } from "mypackage/utils";
```

Use self-referencing in tests to verify export configuration matches consumer experience.

## Monorepo Organization

PNPM workspace monorepos separate deployable applications from reusable libraries:

```
{project}/
├── apps/                  # Deployable applications
│   ├── web/
│   └── api/
├── packages/             # Reusable libraries
│   ├── ui/
│   ├── utils/
│   └── api-client/
├── pnpm-workspace.yaml
├── tsconfig.base.json     # Shared compiler options
└── package.json           # Root: private: true
```

### Workspace Configuration

```yaml
# pnpm-workspace.yaml
packages:
  - "apps/*"
  - "packages/*"
  - "packages/*/*" # if grouping of packages is appropriate
```

The library root's name is a repository convention: `packages/` is the common default, and a polyglot monorepo may namespace libraries by language instead (for example `pkg/es/`, with the workspace globs following it).

### Internal Dependencies

Reference workspace packages with the `workspace:` protocol:

```json
{
  "dependencies": {
    "@company/utils": "workspace:*"
  }
}
```

This protocol resolves to the local workspace package during development and converts to the actual version on publish. Never use file path references (`file:../utils`) for workspace dependencies.

### Naming Conventions

Apply a consistent scope prefix across all packages: `@company/ui-*` for component libraries, `@company/util-*` for utilities, `@company/api-*` for API packages. The scope groups packages visually in `node_modules` and prevents name collisions with public packages.

## Shared Configuration

Root `tsconfig.base.json` defines compiler options inherited by all packages:

```jsonc
{
  "compilerOptions": {
    "target": "ES2022",
    "strict": true,
    "declaration": true,
    "declarationMap": true,
    "sourceMap": true
  }
}
```

Individual packages extend the base and specify their own `rootDir`, `outDir`, and module settings. This keeps shared settings in one place while allowing per-package overrides for different target environments (Node.js libraries vs bundler-based applications).

## File-level organization

Package layout does not keep a single file small. Apply these rules to every source file:

- **Split at a few hundred lines.** Once a file reaches a few hundred lines, split it along its concerns before adding more. A large file invites unrelated edits into one diff, and a later split churns every line it moves.
- **One concern per file, decided at creation.** Choose the file's single concern when you create it, and put new behavior in a new file when it does not fit. Splitting later is a large mechanical move that hides real behavior changes inside the churn.
- **Keep the dependency direction.** Write-path and decision code may feed a renderer; a renderer never feeds a decision. A pure renderer is not a verdict source: code that decides whether to write, block, or accept never imports its answer from a display or formatting module.
- **Prove guard independence, or call it defence in depth.** A claim that two guards are independent holds only when you enumerate each guard's inputs and show the two sets do not overlap. When you cannot enumerate them, restate the claim as defence in depth: two layers that may share a failure mode.

## Decision Support

| Scenario | Structure |
|----------|-----------|
| Single package | `src/` + `dist/` + `package.json` with exports |
| Monorepo with shared libs | `apps/` + `packages/` + PNPM workspace |
| Internal packages only | Point exports at `src/` directly; use `publishConfig` for publish-time override |
| Dual ESM/CJS publishing | Separate `dist/esm/` and `dist/cjs/` with conditional exports |

## Related

- [Import and Export Conventions](import_export_conventions.md)
