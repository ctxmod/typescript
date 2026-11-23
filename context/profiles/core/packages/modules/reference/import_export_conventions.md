# Import and Export Conventions

Type-only imports, file extensions, and export patterns for ECMAScript/TypeScript.

Enforce strict import/export discipline through `verbatimModuleSyntax`, consistent file extensions, and explicit export patterns. These conventions eliminate ambiguity between type-level and value-level imports and ensure emitted JavaScript matches TypeScript source structure.

## Type-Only Imports

Enable `verbatimModuleSyntax` in all TypeScript configurations. This flag enforces one-to-one correspondence between TypeScript imports and emitted JavaScript: type-only imports must use the `import type` syntax, and the compiler preserves all non-type imports verbatim in output.

```typescript
import type { User, Product } from "./types.js";
import { validateEmail, API_VERSION } from "./utils.js";
import { createClient, type ClientConfig } from "./client.js"; // inline modifier
```

Importing a type without a `type` modifier is a compiler error when `verbatimModuleSyntax` is enabled. A statement may mix values and types as long as each type carries an inline `type` modifier; the type specifiers are dropped and the value import is kept. Prefer a separate `import type` statement when the whole import is type-only, since `import { type A }` still emits `import {} from "..."` and so keeps the module's side effects. This explicit marking provides three benefits:

1. **Side effect safety**: The compiler never removes an import that could trigger side effects, because type imports are explicitly marked for removal.
2. **Refactoring confidence**: Renaming or deleting a type never accidentally removes a value import from the same module.
3. **Module system consistency**: The emitted JavaScript contains exactly the imports the developer wrote, with no surprises from type elision.

`verbatimModuleSyntax` replaces the deprecated `importsNotUsedAsValues` and `preserveValueImports` flags and turns `isolatedModules` on by default. Projects still using the deprecated flags should migrate. Pair it with `erasableSyntaxOnly` wherever source is executed by type stripping (Node.js runs erasable TypeScript natively on every current LTS line), so no construct with runtime semantics — `enum`, value namespaces, parameter properties — reaches a strip-only runtime.

## File Extensions

Extension requirements depend on the `moduleResolution` setting.

| Resolution Mode | Extension Required | Typical Use |
|----------------|-------------------|-------------|
| `nodenext` | `.js` in all relative imports | Node.js applications, published libraries |
| `nodenext` + `rewriteRelativeImportExtensions` | `.ts` in all relative imports | Node.js code run from source by type stripping that is also compiled |
| `bundler` | Optional (extensionless allowed) | Bundler-based applications (Vite, webpack) |

Pick one extension convention per package and hold it: `.js` specifiers for
code that is only ever compiled, `.ts` specifiers (with
`rewriteRelativeImportExtensions` rewriting them on emit) for code that also
runs directly under Node.js type stripping, which requires the real `.ts`
extension and never resolves a `.js` specifier to a `.ts` file.

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

## Sources

- https://www.typescriptlang.org/tsconfig/ — `verbatimModuleSyntax` elision rules (inline `type` modifiers, `import {} from` output), `erasableSyntaxOnly`, `rewriteRelativeImportExtensions` (accessed 2026-11-22)
- https://nodejs.org/api/typescript.html — type stripping requires `.ts` extensions in specifiers; unsupported syntax (Node.js v26.13.1 docs; accessed 2026-11-22)
