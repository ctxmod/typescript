# Native TypeScript Compiler

Adopting TypeScript 7, the native (Go) compiler, as the default: configuration valid on both 6.0 and 7.0, and the side-by-side 6.0 API install for tooling until 7.1 ships its API.

TypeScript 7.0 (July 2026) is the native port of the compiler: the same
language, type checker, and `tsconfig.json` surface, compiled to native code
with shared-memory parallelism and typically about ten times faster than 6.0.
It ships as the ordinary `typescript` package — `npm install -D typescript`
puts the native `tsc` in the workspace. The `tsgo` binary of the
`@typescript/native-preview` package was the preview vehicle; repository
scripts call `tsc`, never `tsgo`.

TypeScript 6.0 (March 2026) is the last JavaScript-based release and the
bridge: everything it deprecated is a hard error in 7.0.

## Configuration That Is Valid on Both

Write one `tsconfig.json` that 6.0 and 7.0 both accept, so the compiler can
switch without a configuration change:

- **No 6.0-deprecated option, anywhere in the `extends` chain** —
  `baseUrl`, `moduleResolution: "node"`/`"node10"`/`"classic"`, `module:
  "amd"`/`"umd"`/`"systemjs"`/`"none"`, `target: "es5"`,
  `downlevelIteration`, `outFile`, `esModuleInterop: false`,
  `alwaysStrict: false`. `"ignoreDeprecations": "6.0"` silences them on 6.0
  only; it is a migration aid, never a resting state.
- **State the defaults that moved.** `types` defaults to `[]` (list `"node"`,
  test-runner globals, and other ambient packages explicitly), `rootDir`
  defaults to the `tsconfig.json` directory (set `"rootDir": "src"`), `strict`
  defaults to `true`, `module` to `esnext`, `target` to the current-year ES
  version.
- **Type-ordering differences are not bugs in your code.** 7.0 always uses
  the stable type ordering; on 6.0, `--stableTypeOrdering` reproduces it to
  diagnose declaration-output diffs during migration, and is removed once the
  migration lands.

JavaScript files checked with `checkJs` are analyzed like `.ts` files in 7.0:
values are no longer usable as types (`typeof value`), `@enum` and `@class`
lose their special meaning, and Closure-style function types are rejected.
JSDoc-typed codebases budget a migration pass.

## The Default Install: TypeScript 7 Plus the 6.0 API Alongside

TypeScript 7.0 ships **no programmatic compiler API**; a new, differently
shaped API arrives with 7.1 (in beta as of late 2026). Tools that import the
compiler as a library — typed linting through typescript-eslint, template
checkers built on Volar (Vue, Svelte, Astro, MDX), Angular's language
service, compiler-API code generators and codemods — need the 6.0 API until
they support 7.1's.

**The default is both, side by side:** TypeScript 7 is the compiler every
script and CI leg runs, and the 6.0 API is installed next to it for the tools
that import `typescript`. Install them with npm aliases, the layout the
TypeScript team recommends, so `import "typescript"` resolves to the 6.0 API
while the `tsc` binary is the native compiler:

```json
{
  "devDependencies": {
    "@typescript/native": "npm:typescript@^7.0.2",
    "typescript": "npm:@typescript/typescript6@^6.0.2"
  },
  "scripts": {
    "typecheck": "tsc --noEmit"
  }
}
```

- `@typescript/native` (the aliased `typescript@7`) provides the native `tsc`
  binary; `tsc --noEmit` and any `tsc` build run TypeScript 7.
- `typescript` (the aliased `@typescript/typescript6`) re-exports the 6.0 API
  for library consumers and installs its CLI as `tsc6`, so it never shadows
  the native `tsc`.
- In a workspace, pin both aliases once in the `catalog:` table and reference
  them from each package that needs them.
- A workspace with no compiler-API consumer at all (formatter and linter that
  do not embed TypeScript) installs `typescript@^7` under its own name and
  skips the alias.

**When to drop the 6.0 alias:** once TypeScript 7.1 ships its stable API and
each API consumer in the workspace has released support for it. Then point
`typescript` back at `typescript@^7` and remove `@typescript/native` in one
change. Until then, keep a one-line note beside the manifest naming the
consumers that still need the 6.0 API.

## Editors and CI

- Editors use the TypeScript 7 language server (the VS Code TypeScript 7
  extension, or any LSP client); editor-embedded template checkers that need
  the 6.0 API keep running on the 6.0 install.
- CI's typecheck leg runs the native `tsc --noEmit`; the speedup is the point
  of adoption, so the leg stops being the slow tier and can run on every
  change.
- Linters with their own type-aware engines track the native compiler
  directly (see [Biome-First Toolchain](biome_toolchain.md)).

## Related

- [Project Setup](project_setup.md)
- [TypeScript Validation](typescript_validation.md)

## Sources

- https://devblogs.microsoft.com/typescript/announcing-typescript-7-0/ — 7.0 release, recommended npm-alias side-by-side install, native `tsc` in the `typescript` package, no API, `@typescript/typescript6` side-by-side aliases, moved defaults, JS/JSDoc differences, editor support (2026-07-08; accessed 2026-11-22)
- https://devblogs.microsoft.com/typescript/announcing-typescript-6-0/ — 6.0 as the bridge release: changed defaults, deprecations removed in 7.0, `ignoreDeprecations`, `--stableTypeOrdering` (2026-03-23; accessed 2026-11-22)
- https://devblogs.microsoft.com/typescript/announcing-typescript-7-1-beta/ — 7.1 beta with the new programmatic API entry points (2026-10-20; accessed 2026-11-22)
