# Project Setup

ECMAScript project initialization and configuration for TypeScript.

Initialize an ECMAScript/TypeScript project with correct module configuration, package.json exports, and optional PNPM workspace structure. These steps apply to both standalone packages and monorepo setups.

- [Module Organization](../../modules/reference/module_organization.md)

## Prerequisites

- Node.js 20+ (LTS with stable ESM support)
- PNPM 9+ (for workspace features and `workspace:` protocol)
- TypeScript 5.5+ (`verbatimModuleSyntax`, `nodenext` module resolution)

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
| Node.js application | `"nodenext"` | `"nodenext"` | Required (`.js`) |
| Published library | `"nodenext"` | `"nodenext"` | Required (`.js`) |
| Bundler application | `"esnext"` | `"bundler"` | Optional |

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
    "skipLibCheck": true
  },
  "include": ["src"]
}
```

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
    "esModuleInterop": true,
    "skipLibCheck": true
  },
  "include": ["src"]
}
```

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

For packages requiring compilation, configure `tsup`:

```typescript
// tsup.config.ts
import { defineConfig } from "tsup";

export default defineConfig({
  entry: ["src/index.ts"],
  format: ["esm"],
  dts: true,
  clean: true,
});
```

Install as a dev dependency:

```bash
pnpm add -D tsup typescript
```

For internal monorepo packages that do not publish externally, skip the build tool and point exports directly at TypeScript source during development. Use `publishConfig` to override exports at publish time.

### 5. Configure PNPM Workspace (Monorepo)

For monorepo setups, create workspace configuration at the repository root.

`pnpm-workspace.yaml`:

```yaml
packages:
  - "apps/*"
  - "packages/*"
```

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
pnpm add -Dw typescript tsup
```

Install package-specific dependencies using `--filter`:

```bash
pnpm add react --filter @company/ui
```

### 7. Verify Setup

Run verification checks:

```bash
tsc --noEmit                    # Type checking passes
pnpm tsup                       # Build produces expected output
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
