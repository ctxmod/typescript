# Application Development

Application structure, bundling, and deployment for ECMAScript/TypeScript.

Structure, build, and deploy ECMAScript/TypeScript applications. Applications differ from packages: they are deployable artifacts rather than importable libraries, they use bundlers for optimization, and their `exports` field serves development tooling rather than external consumers.

- [Module Organization](module_organization.md)

## Application Structure

Applications live in `apps/` within a monorepo or as standalone repositories:

```
apps/web/
├── src/
│   ├── index.ts           # Application entry point
│   ├── routes/            # Route handlers or pages
│   ├── services/          # Business logic
│   ├── middleware/         # Request/response middleware
│   └── config/            # Environment configuration
├── public/                # Static assets (web apps)
├── package.json
├── tsconfig.json
└── vite.config.ts         # Or equivalent bundler config
```

The `src/` directory contains all application source. Unlike packages, applications typically do not produce `dist/` output separately -- the bundler handles output generation as part of the build step.

## TypeScript Configuration

Applications targeting bundler environments use `moduleResolution: "bundler"`:

```jsonc
{
  "extends": "../../tsconfig.base.json",
  "compilerOptions": {
    "module": "ESNext",
    "moduleResolution": "bundler",
    "rootDir": "src",
    "outDir": "dist",
    "noEmit": true,
    "esModuleInterop": true
  },
  "include": ["src"]
}
```

Set `noEmit: true` when the bundler handles compilation. TypeScript serves as a type checker only; the bundler (Vite, esbuild, webpack) performs the actual transformation and output generation.

For Node.js server applications that run without a bundler:

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

Server applications compile with `tsc` and run the emitted JavaScript directly in Node.js.

## Bundler Configuration

### Vite (Web Applications)

```typescript
// vite.config.ts
import { defineConfig } from "vite";

export default defineConfig({
  build: {
    target: "es2022",
    outDir: "dist",
    sourcemap: true,
  },
  resolve: {
    conditions: ["import", "module", "default"],
  },
});
```

Vite uses esbuild for development transforms and Rollup for production builds. The `resolve.conditions` array controls which `exports` conditions Vite evaluates when resolving workspace packages.

### esbuild (Node.js Applications)

```typescript
// build.ts
import { build } from "esbuild";

await build({
  entryPoints: ["src/index.ts"],
  bundle: true,
  platform: "node",
  target: "node20",
  format: "esm",
  outfile: "dist/index.js",
  sourcemap: true,
  packages: "external",
});
```

Setting `packages: "external"` excludes `node_modules` from the bundle, keeping the output small and letting Node.js resolve dependencies at runtime. This is appropriate for server applications where dependencies install alongside the built code.

## Consuming Workspace Packages

Applications import workspace packages by name:

```typescript
import { formatDate } from "@scope/utils";
import { Button } from "@scope/ui";
```

When workspace packages use the internal TypeScript package pattern (exports pointing at `src/`), the bundler resolves TypeScript source directly. This eliminates build ordering: application development does not require pre-building every dependency.

For non-bundler applications, workspace packages must compile first. Use PNPM's recursive build or a task runner:

```bash
pnpm -r --filter @apps/web... build
```

The `...` suffix includes all transitive dependencies of `@apps/web` in the build graph.

## Environment Configuration

Separate configuration from source code. Applications load environment-specific settings at startup rather than embedding them in source.

```typescript
// src/config/index.ts
export const config = {
  port: parseInt(process.env.PORT ?? "3000", 10),
  databaseUrl: process.env.DATABASE_URL,
  logLevel: process.env.LOG_LEVEL ?? "info",
} as const;
```

For web applications, bundlers inject environment variables at build time through define/replace plugins. Prefix client-exposed variables consistently (e.g., `VITE_` for Vite, `NEXT_PUBLIC_` for Next.js) to prevent server secrets from leaking into client bundles.

## Deployment Patterns

| Application Type | Build Command | Output | Runtime |
|-----------------|---------------|--------|---------|
| Web SPA | `vite build` | Static files in `dist/` | CDN/static host |
| SSR web app | Framework build | Server + client bundles | Node.js server |
| Node.js API | `tsc` or `esbuild` | JavaScript in `dist/` | Node.js directly |
| CLI tool | `tsup --format esm` | Single bundled file | Node.js with shebang |

### Production Dependencies

Applications install only production dependencies at deploy time:

```bash
pnpm install --prod --frozen-lockfile
```

Use `--frozen-lockfile` in CI to ensure reproducible installs. PNPM's content-addressable storage reduces disk usage by approximately 40% compared to npm, making deployment artifacts smaller.

## Monorepo Task Orchestration

For larger monorepos with many applications and packages, integrate a task runner for build caching and dependency-aware execution:

```bash
pnpm add -Dw nx
```

Configure task dependencies so builds execute in correct order:

```json
{
  "targetDefaults": {
    "build": { "dependsOn": ["^build"] },
    "test": { "dependsOn": ["build"] }
  }
}
```

The `^build` notation means "build all dependencies before building this package." Nx caches build outputs and skips unchanged packages, reducing CI time significantly for incremental changes.

## Related

- [Project Setup](../../toolchain/reference/project_setup.md)
