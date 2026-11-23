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
// tsdown.config.ts
import { defineConfig } from "tsdown";

export default defineConfig({
  entry: ["src/index.ts"],
  format: ["esm"],
  dts: true,
  clean: true,
});
```

`tsdown` (built on Rolldown) is the library bundler of record; `tsup`, the
previous default, is no longer actively maintained and points its users to
`tsdown`, whose `tsdown migrate` command converts an existing config. A
package with nothing to bundle can emit with `tsc` alone.

### Build Output

```
dist/
├── index.js          # ESM JavaScript
├── index.js.map      # Source map
├── index.d.ts        # Type declarations
└── index.d.ts.map    # Declaration map
```

ESM-only is the preferred approach for new packages. Every supported Node.js line (22.12+, 24, 26) loads ESM through `require(esm)` without a flag, so CommonJS consumers can `require()` an ESM-only package (one without top-level `await`). Choose ESM-only unless consumers explicitly require a CommonJS build — for example, runtimes older than the current LTS lines.

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
      "import": {
        "types": "./dist/index.d.mts",
        "default": "./dist/index.mjs"
      },
      "require": {
        "types": "./dist/index.d.cts",
        "default": "./dist/index.cjs"
      }
    },
    "./package.json": "./package.json"
  },
  "main": "./dist/index.cjs",
  "module": "./dist/index.mjs",
  "types": "./dist/index.d.cts",
  "files": ["dist"]
}
```

Each format gets its own declaration file, nested under its condition: a
single `.d.ts` shared by `import` and `require` describes one module format to
both consumers ("masquerading" — CJS consumers type-check against ESM
declarations and the reverse), which `attw` reports as an error. Within each
condition `types` comes first. The `main`, `module`, and top-level `types`
fields provide backward compatibility for tools that do not read `exports`.

### Build Configuration

```typescript
// tsdown.config.ts
import { defineConfig } from "tsdown";

export default defineConfig({
  entry: ["src/index.ts"],
  format: ["esm", "cjs"],
  dts: true,
  clean: true,
  fixedExtension: true, // always .mjs/.cjs, whatever the package type
});
```

`fixedExtension` pins `.mjs`/`.cjs` output (and the matching `.d.mts`/`.d.cts`
declarations) regardless of the package's `type`; set it explicitly rather
than relying on its platform-dependent default.

### Dual Package Hazard

When both ESM and CJS copies load in the same process, Node.js creates separate module instances. This breaks `instanceof` checks and causes state divergence between the two copies.

Mitigation strategies:

| Strategy | Approach |
|----------|----------|
| ESM wrapper | Write one CJS implementation; create a thin ESM wrapper that re-exports from CJS |
| Stateless design | Ensure the package carries no module-level state; separate instances behave identically |
| Single format | Publish ESM-only and rely on `require(esm)` (unflagged in Node.js 20.19+/22.12+, so on every supported line) |

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

This pattern requires consuming applications to use a bundler or TypeScript project references that can resolve `.ts` imports. Direct Node.js execution works only when the package source is erasable TypeScript with explicit `.ts` import extensions and resolves outside `node_modules` (Node.js refuses to strip types under `node_modules`); otherwise compile first.

## Multiple Entry Points

Packages exposing multiple subpath exports need matching build entry points:

```typescript
// tsdown.config.ts
import { defineConfig } from "tsdown";

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
- [ ] `attw --pack .` and `publint` pass (each format resolves its own declarations)

- [TypeScript Validation](../../toolchain/reference/typescript_validation.md)

## Related

- [Project Setup](../../toolchain/reference/project_setup.md)

## Sources

- https://cdn.jsdelivr.net/gh/egoist/tsup@main/README.md — tsup "not actively maintained anymore", recommends tsdown (accessed 2026-11-22)
- https://tsdown.dev/guide/ — tsdown, the Rolldown-based library bundler (v0.23.7; accessed 2026-11-22)
- https://www.npmjs.com/package/tsdown — `fixedExtension` option: always `.mjs`/`.cjs`, default `platform === 'node'` (type declarations of tsdown 0.23.7; accessed 2026-11-22)
- https://nodejs.org/api/modules.html#loading-ecmascript-modules-using-require — `require(esm)` (accessed 2026-11-22)
- https://nodejs.org/api/typescript.html — no type stripping under `node_modules` (Node.js v26.13.1 docs; accessed 2026-11-22)
- https://github.com/arethetypeswrong/arethetypeswrong.github.io — "masquerading as ESM/CJS" problem class (accessed 2026-11-22)
