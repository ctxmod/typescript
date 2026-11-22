# Import and Export Conventions

Type-only imports, file extensions, and export patterns for ECMAScript/TypeScript.

Enforce strict import/export discipline through `verbatimModuleSyntax`, consistent file extensions, and explicit export patterns. These conventions eliminate ambiguity between type-level and value-level imports and ensure emitted JavaScript matches TypeScript source structure.

## Type-Only Imports

Enable `verbatimModuleSyntax` in all TypeScript configurations. This flag enforces one-to-one correspondence between TypeScript imports and emitted JavaScript: type-only imports must use the `import type` syntax, and the compiler preserves all non-type imports verbatim in output.

```typescript
import type { User, Product } from "./types.js";
import { validateEmail, API_VERSION } from "./utils.js";
```

Mixing types and values in a single import statement causes a compiler error when `verbatimModuleSyntax` is enabled. This separation provides three benefits:

1. **Side effect safety**: The compiler never removes an import that could trigger side effects, because type imports are explicitly marked for removal.
2. **Refactoring confidence**: Renaming or deleting a type never accidentally removes a value import from the same module.
3. **Module system consistency**: The emitted JavaScript contains exactly the imports the developer wrote, with no surprises from type elision.

`verbatimModuleSyntax` replaces the older `isolatedModules` and `importsNotUsedAsValues` flags. Projects still using those flags should migrate.

## File Extensions

Extension requirements depend on the `moduleResolution` setting.

| Resolution Mode | Extension Required | Typical Use |
|----------------|-------------------|-------------|
| `nodenext` | `.js` in all relative imports | Node.js applications, published libraries |
| `bundler` | Optional (extensionless allowed) | Bundler-based applications (Vite, webpack) |

With `moduleResolution: "nodenext"`, TypeScript requires the runtime extension (`.js`) even though the source file is `.ts`:

```typescript
import { parse } from "./parser.js";       // Source is parser.ts
import { config } from "./config/index.js"; // Directory index explicit
```

This mirrors Node.js ESM resolution, which does not perform automatic extension resolution or directory index lookup. The `.js` extension in source tells TypeScript to find `parser.ts` at compile time and emit an import referencing `parser.js` at runtime.

With `moduleResolution: "bundler"`, extensionless imports work because bundlers handle resolution internally. However, mixing extensionless source imports with a Node.js runtime target creates mismatches. For libraries consumed outside bundler contexts, prefer explicit extensions regardless of resolution mode.

## Export Patterns

### Named Exports

Prefer named exports over default exports. Named exports enable precise tree-shaking, produce consistent import syntax across consumers, and eliminate the ambiguity of anonymous default exports that can receive arbitrary local names.

```typescript
// Preferred: named exports
export function createClient(config: ClientConfig): Client { ... }
export interface ClientConfig { ... }

// Avoid: default export
export default class Client { ... }
```

Default exports are acceptable for framework-conventional files (React components matching filenames, Next.js page exports) where the ecosystem mandates them.

### Re-export Patterns

Barrel files (`index.ts`) aggregate exports from child modules. Keep barrel files simple: use `export * from` syntax without renaming, and re-export only from immediate children.

```typescript
// packages/utils/src/index.ts
export * from "./string.js";
export * from "./number.js";
export * from "./date.js";
```

For deeper module trees, create additional barrel files at each level rather than re-exporting deeply nested paths from the root. A root barrel importing from `./nested/deep/module.js` couples consumers to internal directory structure.

Avoid selective re-exports (`export { foo, bar } from "./module.js"`) in barrel files. These fragment the export surface and require maintenance when the child module adds exports. Use `export *` and control visibility through what each child module exports.

### Internal Module Imports

The `imports` field in `package.json` provides project-internal path aliases using the `#` prefix:

```json
{
  "imports": {
    "#utils/*": "./src/utils/*.js",
    "#db/*": "./src/db/*.js"
  }
}
```

```typescript
import { hash } from "#utils/crypto";
```

Unlike TypeScript `paths`, the `imports` field works at runtime without bundler transformation. Node.js resolves `#` imports natively. Use `imports` for stable internal aliases; avoid TypeScript `paths` in packages intended for Node.js execution.

## Trade-offs

Strict extension requirements add friction during development. Developers writing `./module.js` for a `.ts` file find it counterintuitive initially. The payoff arrives at scale: emitted code runs directly in Node.js without post-processing, and import resolution behaves identically across development and production.

`export *` in barrel files risks exposing unintended internal symbols. Mitigate this by keeping each child module's exports intentional. If a module contains internal helpers alongside public functions, move helpers to a separate unexported file rather than mixing them with exports.

## Related

- the clarity guidance in the writing_context module (`github.com/ctxmod/writing_context/context`, core profile)
- [Module Organization](module_organization.md)
