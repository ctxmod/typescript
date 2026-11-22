---
paths:
  - "tsconfig.json"
  - "*/tsconfig.json"
  - "*/*/tsconfig.json"
  - "*/*/*/tsconfig.json"
  - "*/*/*/*/tsconfig.json"
  - "*/*/*/*/*/tsconfig.json"
  - "tsconfig.*.json"
  - "*/tsconfig.*.json"
  - "*/*/tsconfig.*.json"
  - "*/*/*/tsconfig.*.json"
  - "*/*/*/*/tsconfig.*.json"
  - "*/*/*/*/*/tsconfig.*.json"
---

# TypeScript Compiler Configuration

Compiler configuration valid on both TypeScript 6.0 and the native TypeScript 7 compiler, with module settings matched to the runtime.

Pair `module` and `moduleResolution` for the target (`nodenext` for Node.js
and published libraries, `esnext` + `bundler` for bundled applications),
keep `strict` and `verbatimModuleSyntax` on, add `erasableSyntaxOnly` where
code runs under type stripping, and state the defaults TypeScript 6.0 moved
(`types`, `rootDir`). No 6.0-deprecated option (`baseUrl`,
`moduleResolution: "node"`, `target: "es5"`) survives anywhere in the
`extends` chain — TypeScript 7 rejects them.

- [Project Setup](../reference/project_setup.md)
- [TypeScript Validation](../reference/typescript_validation.md)
- [Native TypeScript Compiler](../reference/typescript_native_compiler.md)
