# Project Setup

ECMAScript project initialization and configuration for TypeScript.

Initialize an ECMAScript/TypeScript project with correct module configuration, package.json exports, and optional PNPM workspace structure. These steps apply to both standalone packages and monorepo setups.

- [Module Organization](../../modules/reference/module_organization.md)

## Prerequisites

- Node.js on a supported LTS line — 26 (latest LTS), 24, or 22; Node.js 20
  reached end of life in 2026. Every current LTS line runs erasable
  TypeScript directly (stable type stripping).
- pnpm 10+, with pnpm 11 the current major (catalogs, `workspace:` protocol,
  and the supply-chain defaults described in
  [Monorepo Orchestration](monorepo_orchestration.md)).
- TypeScript 7.x as the compiler — the native (Go) compiler shipped as the
  ordinary `typescript` package's `tsc` — with the TypeScript 6.0 API
  installed alongside (npm alias) for tooling that imports the compiler API,
  until TypeScript 7.1 ships its API. Configuration stays valid on both — see
  [Native TypeScript Compiler](typescript_native_compiler.md).

## Steps

### 1. Initialize Package

Create `package.json` with ESM as the default module system:

```json
{
  "name": "@company/my-package",
  "version": "0.0.0",
  "type": "module",
  "private": true
}
```

Setting `"type": "module"` causes Node.js to treat all `.js` files in the package as ES modules. Without this field, `.js` files default to CommonJS.

### 2. Configure TypeScript

Select the `module` and `moduleResolution` pair matching the target environment:

| Target | module | moduleResolution | Extensions |
|--------|--------|-----------------|------------|
| Node.js application (compiled) | `"nodenext"` | `"nodenext"` | Required (`.js`) |
| Node.js application (run from source by type stripping) | `"nodenext"` | `"nodenext"` | Required (`.ts`, with `rewriteRelativeImportExtensions` if anything emits) |
| Published library | `"nodenext"` | `"nodenext"` | Required (`.js`) |
| Bundler application | `"esnext"` | `"bundler"` | Optional |

`"nodenext"` floats with the newest Node.js behavior and implies
`target: "esnext"`; pin `"node20"` instead when the emitted module semantics
must stay fixed across compiler upgrades.

Three TypeScript 6.0 default changes (carried into 7.0) make some options
mandatory to state rather than inherit: `types` now defaults to `[]` (list
`"node"` and any other global type packages explicitly), `rootDir` defaults to
the directory holding `tsconfig.json` (set it to `src`), and `strict` defaults
to `true` (keep it explicit for readers). `esModuleInterop` and
`allowSyntheticDefaultImports` can no longer be `false`, so they need not be
set at all.

Create `tsconfig.json` for a Node.js ESM project:

```jsonc
{
  "compilerOptions": {
    "target": "ES2022",
    "module": "NodeNext",
    "moduleResolution": "NodeNext",
    "rootDir": "src",
    "outDir": "dist",
    "declaration": true,
    "declarationMap": true,
    "sourceMap": true,
    "strict": true,
    "verbatimModuleSyntax": true,
    "erasableSyntaxOnly": true,
    "types": ["node"],
    "skipLibCheck": true
  },
  "include": ["src"]
}
```

`erasableSyntaxOnly` rejects TypeScript constructs with runtime semantics
(`enum`, namespaces with values, parameter properties, `import =`), so the
same source runs under Node.js type stripping and any strip-only transformer
unchanged. For an application executed straight from `.ts` source, add
`"noEmit": true` (or `"rewriteRelativeImportExtensions": true` when a build
also emits) and write `.ts` extensions in relative imports — the
configuration the Node.js TypeScript documentation recommends.

For bundler-based applications:

```jsonc
{
  "compilerOptions": {
    "target": "ES2022",
    "module": "ESNext",
    "moduleResolution": "bundler",
    "rootDir": "src",
    "outDir": "dist",
    "strict": true,
    "verbatimModuleSyntax": true,
    "noEmit": true,
    "types": ["vite/client"], // the bundler's ambient types, listed explicitly
    "skipLibCheck": true
  },
  "include": ["src"]
}
```

Path aliases use `paths` alone: `baseUrl` is deprecated in TypeScript 6.0 and
removed in 7.0, and `paths` entries resolve relative to the `tsconfig.json`
that declares them.

### 3. Configure Package Exports

Define the public API through the `exports` field:

```json
{
  "exports": {
    ".": {
      "types": "./dist/index.d.ts",
      "import": "./dist/index.js"
    }
  }
}
```

Place `types` first in condition order. Add subpath exports for additional entry points. Omit paths that should remain private to consumers.

### 4. Set Up Build Tooling

For packages requiring compilation, configure `tsdown` (the Rolldown-based
library bundler; `tsup`, the previous default, is no longer actively
maintained and its README points to `tsdown`):

```typescript
// tsdown.config.ts
import { defineConfig } from "tsdown";

export default defineConfig({
  entry: ["src/index.ts"],
  format: ["esm"],
  dts: true,
  clean: true,
});
```

Install as a dev dependency:

```bash
pnpm add -D tsdown typescript
```

A package with no bundling needs can skip the bundler entirely and emit with
`tsc` (`declaration: true`); the bundler earns its place only when output must
be bundled, minified, or produced in several formats.

For internal monorepo packages that do not publish externally, skip the build tool and point exports directly at TypeScript source during development. Use `publishConfig` to override exports at publish time.

### 5. Configure PNPM Workspace (Monorepo)

For monorepo setups, create workspace configuration at the repository root.

`pnpm-workspace.yaml`:

```yaml
packages:
  - "apps/*"
  - "packages/*"
catalog:
  "@typescript/native": npm:typescript@^7.0.2       # native tsc
  typescript: npm:@typescript/typescript6@^6.0.2    # 6.0 API for tooling
```

Shared versions live in the `catalog:` table and manifests reference them as
`"typescript": "catalog:"`; since pnpm 11 the workspace file is also where
pnpm-specific settings go (`.npmrc` keeps only registry and auth settings).

Root `package.json`:

```json
{
  "private": true,
  "scripts": {
    "build": "pnpm -r build",
    "test": "pnpm -r test",
    "lint": "pnpm -r lint"
  }
}
```

Root `tsconfig.base.json`:

```jsonc
{
  "compilerOptions": {
    "target": "ES2022",
    "strict": true,
    "declaration": true,
    "declarationMap": true,
    "sourceMap": true,
    "verbatimModuleSyntax": true,
    "skipLibCheck": true
  }
}
```

Individual packages extend the base:

```jsonc
{
  "extends": "../../tsconfig.base.json",
  "compilerOptions": {
    "module": "NodeNext",
    "moduleResolution": "NodeNext",
    "rootDir": "src",
    "outDir": "dist"
  },
  "include": ["src"]
}
```

### 6. Add Internal Dependencies

Reference workspace packages with the `workspace:` protocol:

```json
{
  "dependencies": {
    "@company/utils": "workspace:*"
  }
}
```

Install shared dev dependencies at the workspace root:

```bash
pnpm add -Dw tsdown
```

and reference the catalog pins from the root manifest:

```json
{
  "devDependencies": {
    "@typescript/native": "catalog:",
    "typescript": "catalog:"
  }
}
```

The two TypeScript entries are the default side-by-side install (native
`tsc` plus the 6.0 API for tooling); see
[Native TypeScript Compiler](typescript_native_compiler.md).

Install package-specific dependencies using `--filter`:

```bash
pnpm add react --filter @company/ui
```

### 7. Verify Setup

Run verification checks:

```bash
tsc --noEmit                    # Type checking passes
pnpm tsdown                     # Build produces expected output
node dist/index.js              # Runtime execution works (Node.js target)
```

- [TypeScript Validation](typescript_validation.md)

## Decision Support

| Situation | Action |
|-----------|--------|
| Library consumed by bundlers and Node.js | Use `moduleResolution: "nodenext"` with `.js` extensions |
| Application using Vite/webpack only | Use `moduleResolution: "bundler"`, extensions optional |
| Internal monorepo package, no external publish | Skip build tool; point exports at `src/` with `publishConfig` override |
| Existing CJS project migrating | Add `"type": "module"`, update imports with extensions, enable `verbatimModuleSyntax` |

## Related

- [Import and Export Conventions](../../modules/reference/import_export_conventions.md)
- [Native TypeScript Compiler](typescript_native_compiler.md)

## Sources

- https://devblogs.microsoft.com/typescript/announcing-typescript-6-0/ — TypeScript 6.0 default changes (`strict`, `module`, `target`, `types`, `rootDir`) and deprecations (`baseUrl`, `moduleResolution node`, `esModuleInterop: false`) (2026-03-23; accessed 2026-11-22)
- https://devblogs.microsoft.com/typescript/announcing-typescript-7-0/ — TypeScript 7.0 native `tsc`, removed 6.0-deprecated options (2026-07-08; accessed 2026-11-22)
- https://www.typescriptlang.org/tsconfig/ — `module` `node20`/`nodenext` semantics, `erasableSyntaxOnly`, `rewriteRelativeImportExtensions`, `verbatimModuleSyntax` (accessed 2026-11-22)
- https://nodejs.org/api/typescript.html — stable type stripping, unsupported syntax, recommended tsconfig, mandatory `.ts` extensions (Node.js v26.13.1 docs; accessed 2026-11-22)
- https://nodejs.org/en/about/previous-releases — release-line status: v26/v24/v22 LTS, v20 EOL (accessed 2026-11-22)
- https://cdn.jsdelivr.net/gh/egoist/tsup@main/README.md — tsup "not actively maintained", points to tsdown (accessed 2026-11-22)
- https://pnpm.io/blog/releases/11.0 — pnpm 11 settings move to `pnpm-workspace.yaml`, Node.js 22+ requirement (accessed 2026-11-22)
