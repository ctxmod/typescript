# TypeScript Validation

Validate ECMAScript/TypeScript configuration and module resolution correctness.

Verify that ECMAScript/TypeScript configuration is correct: module resolution settings align with the target environment, `verbatimModuleSyntax` is enabled, file extensions are consistent, and package.json exports match build output.

## Checks

Seven checks compose the full TypeScript configuration validation. Five are automated; two are semi-automated.

| Check | Method | Tool | Pass | Warn | Fail |
|-------|--------|------|------|------|------|
| Type checking | Automated | `tsc --noEmit` | 0 errors | 0 errors, warnings present | Any error |
| Module/resolution pair | Automated | Config inspection | Valid pair | - | Invalid combination |
| `verbatimModuleSyntax` | Automated | Config inspection | Enabled | - | Disabled |
| File extension consistency | Semi-automated | Custom script or ESLint | All imports have extensions | >90% with extensions | <90% |
| Export path resolution | Automated | `attw` (arethetypeswrong) | All conditions resolve | Minor issues | Missing types condition |
| Declaration file generation | Automated | Build + verify | `.d.ts` for every `.js` | Missing declaration maps | Missing declarations |
| Self-referencing imports | Semi-automated | Test execution | Imports resolve | - | Resolution failure |

## Automated Checks

### Type Checking

Run type checking with `tsc --noEmit` (or the project's `typecheck` script, which wraps it):

```bash
npx tsc --noEmit
```

This validates all type relationships without producing output files. In monorepos, run it per package (for example through the workspace task runner). Zero errors is the pass threshold. Enable `strict: true` from project initialization rather than adding it retroactively (it is the default since TypeScript 6.0; keep it explicit).

On TypeScript 7 the same `tsc --noEmit` invocation runs the native compiler; with the default side-by-side install the `tsc` binary is the native one and the 6.0 API serves only library consumers ([Native TypeScript Compiler](typescript_native_compiler.md)). A config that still carries a 6.0-deprecated option (`baseUrl`, `moduleResolution: "node"`) fails here on 7.0 — treat that as a configuration failure, not a type error.

### Module and Resolution Pair Validation

Valid configuration pairs:

| module | moduleResolution | Valid |
|--------|-----------------|-------|
| `NodeNext` | `NodeNext` | Yes |
| `Node16` | `Node16` | Yes |
| `ESNext` | `bundler` | Yes |
| `ESNext` | `NodeNext` | No -- mismatched semantics |
| `CommonJS` | `bundler` | No -- CJS module with bundler resolution |

TypeScript enforces some invalid combinations at compile time. Others produce subtle resolution mismatches that surface only at runtime. Validate the pair explicitly during setup.

### verbatimModuleSyntax

Inspect `tsconfig.json` for `"verbatimModuleSyntax": true`. This flag supersedes the deprecated `importsNotUsedAsValues` and `preserveValueImports` flags, and enabling it turns `isolatedModules` on by default. Projects using the older flags should migrate. Where code runs under Node.js type stripping or another strip-only transformer, also require `"erasableSyntaxOnly": true`.

When disabled, the compiler may silently elide imports that appear to be type-only, causing runtime failures if those imports had side effects.

### Export Path Resolution

Use `attw` (Are The Types Wrong?) to validate that every condition in the `exports` field resolves correctly:

```bash
npx @arethetypeswrong/cli --pack .
```

The tool verifies that `types`, `import`, and `require` conditions each resolve to valid files without conflicts — including the "masquerading" failures where one declaration file is served to both ESM and CJS consumers. Pair it with `publint`, which lints the manifest itself (missing files, wrong condition order, `type`/extension mismatches).

**Pass**: All conditions resolve without issues. **Fail**: Missing `types` condition, absent declaration files, or format mismatches.

### Declaration File Generation

After building, verify that every emitted `.js` file has a corresponding `.d.ts` file:

```bash
# Verify declaration file parity
for js in dist/**/*.js; do
  dts="${js%.js}.d.ts"
  [ -f "$dts" ] || echo "Missing: $dts"
done
```

Declaration maps (`.d.ts.map`) enable "go to source" functionality in editors. Their absence is a warning, not a failure.

## Semi-Automated Checks

### File Extension Consistency

With `moduleResolution: "nodenext"`, all relative imports must include `.js` extensions. Verify with an ESLint rule or a custom script:

```bash
# Find imports missing extensions
grep -rn "from '\./[^']*'" src/ | grep -v "\.js'" | grep -v "\.json'"
```

Enforce it with the lint toolchain's import-extension rule (Biome's `useImportExtensions`, or `import/extensions` where the legacy linter is scoped in) run through the project's lint script. With `moduleResolution: "bundler"`, extensionless imports are valid and this check does not apply. Under `nodenext` the compiler itself already errors on a missing extension, so the lint rule matters mainly for JavaScript sources and mixed trees.

### Self-Referencing Imports

Test that the package can import its own exports by name. This validates the `exports` field from the consumer perspective:

```typescript
// test/exports.test.ts
import { main } from "@scope/my-package";
import { helper } from "@scope/my-package/utils";
```

If self-referencing imports fail, the `exports` field has a resolution error that consumers will also encounter.

## Remediation

**Type errors**: Fix type mismatches at their source. Avoid `@ts-ignore` or `any` casts as workarounds -- these mask problems that surface later as runtime failures.

**Invalid module/resolution pair**: Align both settings to the target environment. For Node.js targets, use `NodeNext`/`NodeNext`. For bundler targets, use `ESNext`/`bundler`.

**Missing extensions**: Add `.js` extensions to all relative imports. Tools like `ts2esm` automate this migration for existing codebases.

**Export resolution failures**: Verify that every path in `exports` points to a file that exists after building. Mismatches between `outDir` and `exports` paths are the most common cause.

**Missing declarations**: Enable `declaration: true` and `declarationMap: true` in `tsconfig.json`. With `tsdown`, set `dts: true`; enabling `isolatedDeclarations` lets declaration emit run per file without a full type check.

## Integration

Run validation checks in CI as part of the build pipeline:

```yaml
steps:
  - run: pnpm tsc --noEmit
  - run: pnpm build
  - run: npx @arethetypeswrong/cli --pack .
  - run: npx publint
```

Place type checking before building to catch errors early. Place export validation after building to verify the complete output.

- [Dependency Analysis](../../modules/reference/dependency_analysis.md)

## Related

- [Project Setup](project_setup.md)
- [Import and Export Conventions](../../modules/reference/import_export_conventions.md)

## Sources

- https://www.typescriptlang.org/tsconfig/ — `verbatimModuleSyntax` supersedes `importsNotUsedAsValues`/`preserveValueImports` and defaults `isolatedModules` on; `erasableSyntaxOnly`; `isolatedDeclarations` (accessed 2026-11-22)
- https://devblogs.microsoft.com/typescript/announcing-typescript-7-0/ — 6.0-deprecated options are errors in 7.0; native `tsc` (2026-07-08; accessed 2026-11-22)
- https://devblogs.microsoft.com/typescript/announcing-typescript-6-0/ — `strict` default `true` (2026-03-23; accessed 2026-11-22)
