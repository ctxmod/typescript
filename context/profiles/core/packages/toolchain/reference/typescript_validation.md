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

This validates all type relationships without producing output files. In monorepos, run it per package (for example through the workspace task runner). Zero errors is the pass threshold. Enable `strict: true` from project initialization rather than adding it retroactively.

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

Inspect `tsconfig.json` for `"verbatimModuleSyntax": true`. This flag supersedes `isolatedModules` and `importsNotUsedAsValues`. Projects using the older flags should migrate.

When disabled, the compiler may silently elide imports that appear to be type-only, causing runtime failures if those imports had side effects.

### Export Path Resolution

Use `attw` (Are The Types Wrong?) to validate that every condition in the `exports` field resolves correctly:

```bash
npx @arethetypeswrong/cli --pack .
```

The tool verifies that `types`, `import`, and `require` conditions each resolve to valid files without conflicts.

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

Configure ESLint with `eslint-plugin-import` and the `import/extensions` rule set to `"error"` for automated enforcement (via the project's lint script). With `moduleResolution: "bundler"`, extensionless imports are valid and this check does not apply.

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

**Missing declarations**: Enable `declaration: true` and `declarationMap: true` in `tsconfig.json`. With `tsup`, set `dts: true`.

## Integration

Run validation checks in CI as part of the build pipeline:

```yaml
steps:
  - run: pnpm tsc --noEmit
  - run: pnpm build
  - run: npx @arethetypeswrong/cli --pack .
```

Place type checking before building to catch errors early. Place export validation after building to verify the complete output.

[Dependency Analysis](../../modules/reference/dependency_analysis.md)

## Related

- [Project Setup](project_setup.md)
- [Import and Export Conventions](../../modules/reference/import_export_conventions.md)
