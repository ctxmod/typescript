# Environment Setup

Development environment configuration for React Router v7 SSR with Cloudflare Workers, Tailwind v4, and TypeScript.

Procedure for configuring a React Router v7 SSR development environment targeting Cloudflare Workers. This covers dependency installation, TypeScript configuration, Tailwind v4 integration, project structure, and Vite build setup.

## Setup Workflow

### 1. Install Core Dependencies

Install the React Router v7 framework, Cloudflare adapter, and build tooling:

```bash
npm install react-router @react-router/cloudflare
npm install -D @react-router/dev @cloudflare/vite-plugin vite typescript
```

Install Tailwind CSS v4 with the Vite plugin (replaces PostCSS integration):

```bash
npm install -D tailwindcss @tailwindcss/vite
```

Install additional tooling as needed:

```bash
npm install i18next react-i18next remix-i18next    # Internationalization
npm install zod                                     # Input validation
npm install class-variance-authority clsx            # Styling utilities
```

### 2. Configure React Router

Create the framework configuration file:

```typescript
// react-router.config.ts
import type { Config } from "@react-router/dev/config";

export default {
  ssr: true,
  future: {
    v8_middleware: true,
    v8_splitRouteModules: true,
  },
} satisfies Config;
```

Enable middleware support (`v8_middleware`) for centralized concerns like i18n detection. Enable route module splitting (`v8_splitRouteModules`) for granular code splitting.

### 3. Configure Vite

Set up Vite with React Router, Cloudflare, and Tailwind plugins:

```typescript
// vite.config.ts
import { reactRouter } from "@react-router/dev/vite";
import { cloudflare } from "@cloudflare/vite-plugin";
import tailwindcss from "@tailwindcss/vite";
import { defineConfig } from "vite";

export default defineConfig({
  plugins: [tailwindcss(), reactRouter(), cloudflare()],
  build: {
    sourcemap: "hidden",
  },
});
```

Plugin order matters: Tailwind processes styles before React Router bundles routes. Use `sourcemap: "hidden"` for proper error locations without sourcemap references in output files.

For monorepo projects, configure filesystem access:

```typescript
server: {
  fs: {
    allow: ["../.."], // Allow access to parent directories
  },
},
```

### 4. Configure TypeScript

Set up TypeScript with React Router's generated types and path aliases:

```json
{
  "compilerOptions": {
    "target": "ES2022",
    "module": "ES2022",
    "moduleResolution": "Bundler",
    "jsx": "react-jsx",
    "strict": true,
    "skipLibCheck": true,
    "rootDirs": [".", "./.react-router/types"],
    "paths": {
      "@app/*": ["./src/*"]
    }
  },
  "include": [
    "src/**/*",
    ".react-router/types/**/*"
  ]
}
```

The `rootDirs` configuration enables generated route types (`Route.ComponentProps`, `Route.LoaderArgs`) to resolve correctly. Path aliases (`@app/*`) simplify imports across the project.

### 5. Set Up Tailwind CSS v4

Create the main stylesheet with CSS-first configuration:

```css
/* src/styles/app.css */
@import "tailwindcss";

@theme {
  --color-primary: oklch(0.53 0.12 118.34);
  --color-background: oklch(0.98 0 0);
  --color-foreground: oklch(0.15 0 0);
  --font-sans: "Inter", system-ui, sans-serif;
  --radius-default: 0.5rem;
}
```

No `tailwind.config.js` file needed. Tailwind v4 uses the `@theme` directive for all token definitions. Tokens in `@theme` automatically generate utilities (`bg-primary`, `text-foreground`).

For projects with specific source directories, control content detection:

```css
@source "../components/**/*.tsx";
@source "../routes/**/*.tsx";
@source not "../legacy-vendor";
```

### 6. Create Project Structure

Establish the directory layout:

```
project/
  src/
    root.tsx                 # Root layout, middleware, error boundary
    routes.ts                # Route configuration
    entry.server.tsx         # SSR entry (renderToReadableStream)
    entry.client.tsx         # Hydration entry
    middleware/
      i18next.ts             # i18n middleware
    routes/
      _index.tsx             # Home route
    components/
      atoms/                 # Buttons, inputs, badges
      molecules/             # Search bars, form fields
      organisms/             # Headers, sidebars
    styles/
      app.css                # Main stylesheet with @theme
      tokens.css             # Design tokens (optional separate file)
      print.css              # Print styles
  workers/
    app.ts                   # Cloudflare Worker fetch handler
  public/
    favicon.ico
  react-router.config.ts
  vite.config.ts
  tsconfig.json
  wrangler.jsonc
```

### 7. Configure Environment Variables

**Local development**: Create `.dev.vars` for Cloudflare Workers local variables:

```
SESSION_SECRET=local-dev-secret
DATABASE_URL=http://localhost:8787
```

Use `.dev.vars` (not `.env.local`) for Cloudflare Workers compatibility. The Cloudflare Vite plugin reads this file during local development.

**Public variables**: Prefix with `VITE_` for client-side access. These build into client bundles:

```jsonc
// wrangler.jsonc
{
  "vars": {
    "VITE_PUBLIC_API_URL": "https://api.example.com"
  }
}
```

**Secret variables**: Add through Cloudflare Dashboard (Settings > Variables and Secrets). Never commit secrets to configuration files.

### 8. Configure Wrangler

Create the Cloudflare Workers configuration:

```jsonc
// wrangler.jsonc
{
  "name": "my-app",
  "main": "workers/app.ts",
  "compatibility_date": "2025-01-01",
  "vars": {
    "VITE_APP_NAME": "My Application"
  },
  "kv_namespaces": [],
  "d1_databases": [],
  "r2_buckets": []
}
```

Add bindings (KV, D1, R2) as the application requires them. Each binding becomes available through `context.cloudflare.env` in loaders and actions.

## Verification

After setup, verify the environment:

```bash
npm run dev          # Starts Vite dev server with Cloudflare Workers
npm run build        # Builds for production
npm run typecheck    # Verifies TypeScript compilation
```

The dev server should:
- Serve pages with SSR (view source shows rendered HTML)
- Apply Tailwind utilities (styles visible without FOUC)
- Generate route types in `.react-router/types/`

## Setup Checklist

- [ ] React Router v7 with SSR enabled in config
- [ ] Vite configured with Tailwind, React Router, and Cloudflare plugins
- [ ] TypeScript includes `.react-router/types` with `rootDirs`
- [ ] Tailwind v4 uses `@theme` directive (no JavaScript config)
- [ ] Project structure has `src/`, `workers/`, `public/`
- [ ] `.dev.vars` created for local environment variables
- [ ] `wrangler.jsonc` configured with compatibility date
- [ ] Dev server renders SSR HTML successfully

## Related

- [Deployment](deployment.md)
- [Cloudflare Workers Deployment](cloudflare_workers.md)
