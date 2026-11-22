# Package Development

Building and publishing ECMAScript packages with optional dual CJS support.

Build, configure, and publish ECMAScript/TypeScript packages. This procedure covers ESM-only packages, dual ESM/CJS packages, and internal monorepo packages that skip compilation during development.

- [Module Organization](module_organization.md)

## ESM-Only Package

The simplest configuration. The package produces only ESM output.

### package.json

```json
{
  "name": "@scope/utils",
  "version": "1.0.0",
  "type": "module",
  "exports": {
    ".": {
      "types": "./dist/index.d.ts",
      "import": "./dist/index.js"
    }
  },
  "files": ["dist"]
}
```

### Build Configuration

```typescript
// tsup.config.ts
import { defineConfig } from "tsup";

export default defineConfig({
  entry: ["src/index.ts"],
  format: ["esm"],
  dts: true,
  clean: true,
  splitting: false,
});
```

### Build Output

```
dist/
├── index.js          # ESM JavaScript
├── index.js.map      # Source map
├── index.d.ts        # Type declarations
└── index.d.ts.map    # Declaration map
```

ESM-only is the preferred approach for new packages. Node.js 22+ supports `require(esm)` natively, reducing the need for CJS output. Choose ESM-only unless consumers explicitly require CommonJS.

## Dual ESM/CJS Package

When consumers include both ESM and CJS environments, produce both formats with conditional exports.

### package.json

```json
{
  "name": "@scope/utils",
  "version": "1.0.0",
  "type": "module",
  "exports": {
    ".": {
      "types": "./dist/index.d.ts",
      "import": "./dist/index.mjs",
      "require": "./dist/index.cjs",
      "default": "./dist/index.mjs"
    },
    "./package.json": "./package.json"
  },
  "main": "./dist/index.cjs",
  "module": "./dist/index.mjs",
  "types": "./dist/index.d.ts",
  "files": ["dist"]
}
```

Condition order matters: `types` first for TypeScript resolution, then `import` and `require` for format selection, then `default` as fallback. The `main` and `module` fields provide backward compatibility for older bundlers that do not read `exports`.

### Build Configuration

```typescript
// tsup.config.ts
import { defineConfig } from "tsup";

export default defineConfig({
  entry: ["src/index.ts"],
  format: ["esm", "cjs"],
  dts: true,
  clean: true,
  splitting: false,
  outExtension({ format }) {
    return { js: format === "esm" ? ".mjs" : ".cjs" };
  },
});
```

### Dual Package Hazard

When both ESM and CJS copies load in the same process, Node.js creates separate module instances. This breaks `instanceof` checks and causes state divergence between the two copies.

Mitigation strategies:

| Strategy | Approach |
|----------|----------|
| ESM wrapper | Write one CJS implementation; create a thin ESM wrapper that re-exports from CJS |
| Stateless design | Ensure the package carries no module-level state; separate instances behave identically |
| Single format | Publish ESM-only and rely on `require(esm)` in Node.js 22+ |

For packages with internal state (caches, registries, singletons), the ESM wrapper approach eliminates the hazard entirely by maintaining a single implementation copy.

## Internal Monorepo Package

Monorepo packages consumed only by sibling packages can skip compilation during development. Point `exports` at TypeScript source directly.

### package.json

```json
{
  "name": "@scope/shared",
  "version": "0.0.0",
  "type": "module",
  "exports": {
    ".": {
      "types": "./src/index.ts",
      "import": "./src/index.ts"
    }
  },
  "publishConfig": {
    "exports": {
      ".": {
        "types": "./dist/index.d.ts",
        "import": "./dist/index.js"
      }
    }
  }
}
```

Benefits: no build step for inter-package development, instant type checking across packages, faster test execution. The `publishConfig` field overrides exports at publish time, so published packages reference compiled output.

This pattern requires consuming applications to use a bundler or TypeScript project references that can resolve `.ts` imports. It does not work for direct Node.js execution without compilation.

## Multiple Entry Points

Packages exposing multiple subpath exports need matching build entry points:

```typescript
// tsup.config.ts
import { defineConfig } from "tsup";

export default defineConfig({
  entry: {
    index: "src/index.ts",
    utils: "src/utils.ts",
    types: "src/types.ts",
  },
  format: ["esm"],
  dts: true,
  clean: true,
});
```

```json
{
  "exports": {
    ".": { "types": "./dist/index.d.ts", "import": "./dist/index.js" },
    "./utils": { "types": "./dist/utils.d.ts", "import": "./dist/utils.js" },
    "./types": { "types": "./dist/types.d.ts", "import": "./dist/types.js" }
  }
}
```

Each entry point produces its own JavaScript and declaration file. Consumers import only the subpaths they need, enabling effective tree-shaking at the entry point level.

## Publishing Checklist

Before publishing, verify:

- [ ] `"type": "module"` set in package.json
- [ ] `exports` field defines all public entry points
- [ ] `files` field includes only `dist/` (or relevant output)
- [ ] `types` condition appears first in each export
- [ ] Build output matches all declared export paths
- [ ] `pnpm pack --dry-run` shows expected files

- [TypeScript Validation](../../toolchain/reference/typescript_validation.md)

## Related

- [Project Setup](../../toolchain/reference/project_setup.md)
