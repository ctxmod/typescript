# shadcn/ui Integration

Component ownership model, Base UI or Radix UI primitives, CVA patterns, the CLI and registries, and customization workflow.

shadcn/ui provides copy-paste component source built on a headless primitives library — Base UI (the default for new projects since July 2026) or Radix UI — and styled with Tailwind CSS. The project owns all component code after installation. No runtime dependency on shadcn exists -- the CLI copies source files into the project, and maintenance responsibility transfers entirely to the consumer.

## Component Ownership Model

The `npx shadcn@latest add [component]` command copies component source into the project's codebase. After installation, the project controls every line. There is no npm package to pin, no upstream version to track, and no breaking changes to absorb through dependency updates.

This model provides full customization control and an explicit dependency tree. The tradeoff: security patches and bug fixes in the primitives library or utility libraries require manual attention — the primitives package itself is an ordinary npm dependency that is pinned and updated like any other. When upstream improvements matter, inspect them first (`add <component> --diff`), then re-add with `--overwrite` and reapply local modifications.

## Architecture Stack

Four libraries form the component architecture, each handling a distinct concern.

| Layer | Library | Role |
|-------|---------|------|
| Behavior | Base UI or Radix UI (one per project) | Accessible primitives -- keyboard navigation, ARIA attributes, focus management |
| Styling | Tailwind CSS v4 | Utility-first classes, `@theme` design tokens |
| Variants | CVA | Multi-dimensional variant schemas with type safety |
| Composition | `cn()` | Class merging via clsx + tailwind-merge |

The primitives library handles all interaction logic: focus trapping in dialogs, arrow-key navigation in menus, typeahead in selects, and roving tabindex in tab groups. Components inherit WAI-ARIA compliance without manual ARIA attribute management.

**One primitives library per project.** `npx shadcn@latest init` selects Base UI by default; `init --base radix` (`-b radix`) keeps the Radix path, and Radix remains fully supported — every new component ships for both libraries except those that exist only in Base UI. Existing Radix projects need no migration. The two differ at the composition seam: Radix components compose through `asChild`, Base UI components through a `render` prop. Mixing both in one project duplicates the behavior layer and splits that seam, so a switch is a recorded migration (the shadcn skill migrates one component at a time), not a gradual drift. Scripts and CI that expect Radix pass `-b radix` explicitly.

## CVA Variant Management

Class Variance Authority manages multi-dimensional component variants through declarative schemas. Use CVA when a component has two or more variant dimensions (e.g., size + variant + state). For binary states, conditional classes suffice.

```typescript
const buttonVariants = cva("inline-flex items-center justify-center rounded-md font-medium", {
  variants: {
    variant: {
      default: "bg-primary text-primary-foreground",
      outline: "border border-input bg-background",
      ghost: "hover:bg-accent hover:text-accent-foreground",
    },
    size: {
      sm: "h-9 px-3 text-sm",
      md: "h-10 px-4 py-2",
      lg: "h-11 px-8 text-lg",
    },
  },
  compoundVariants: [
    { variant: "outline", size: "lg", className: "border-2" },
  ],
  defaultVariants: { variant: "default", size: "md" },
});
```

Extract type-safe props with `VariantProps<typeof buttonVariants>` to enforce compile-time variant validation. Colocate variant schemas in component files for discoverability.

## The cn() Utility

`cn()` combines `clsx` for conditional class composition and `tailwind-merge` for Tailwind conflict resolution. Every component that accepts a `className` prop should use it.

```typescript
import { clsx, type ClassValue } from "clsx";
import { twMerge } from "tailwind-merge";

export function cn(...inputs: ClassValue[]) {
  return twMerge(clsx(inputs));
}
```

The standard application pattern: `cn(buttonVariants({ variant, size }), className)`. Base variants apply first, then consumer overrides. `tailwind-merge` resolves conflicts so the last specified utility wins -- `cn("bg-red-500", "bg-blue-500")` produces `bg-blue-500`.

## CLI Workflow

Initialize a project with `npx shadcn@latest init`, which creates `components.json` and installs base dependencies (`--base` picks the primitives library, `--preset <code>` applies a packed design-system configuration, `--template` scaffolds a Next.js, Vite, React Router, TanStack Start, Astro, or Laravel app, with `--monorepo` for a workspace layout). Add components individually with `npx shadcn@latest add [component]` or in bulk with `--all`. Before anything is written, `--dry-run`, `--view`, and `--diff` show what a registry item would add or change; `info` reports the detected framework and installed components, and `docs <component>` prints usage from the CLI.

Configure `components.json` to match the project's directory structure through the `aliases` field. The CLI places UI primitives at the `aliases.ui` path (default `@/components/ui`) and composed components at `aliases.components`.

Third-party and private registries are declared as namespaces in `components.json` (`"registries": { "@acme": "https://registry.acme.com/{name}.json" }`, with headers and environment-variable expansion for auth) and installed as `add @acme/<item>`; a `registry:base` item distributes an entire design system and can pin the primitives library. Treat a registry like any other dependency source: review its items with `--view` before adding them.

To incorporate upstream improvements, re-add with `npx shadcn@latest add [component] --overwrite`. This replaces the component source entirely, so reapply local customizations afterward. Track customized components in documentation to simplify this process.

## Customization Patterns

Two strategies cover most customization needs. Wrapping handles the majority of cases while preserving the ability to re-add from upstream.

**Wrapping** (preferred, ~80% of cases): Create a project-specific component that composes the shadcn component with additional props, defaults, or behavior. The original component remains unmodified.

**Forking** (structural changes): Modify the copied source directly when deep behavioral or layout changes are needed. Document deviations from upstream. Re-adding requires manual merge.

| Change Type | Strategy | Reasoning |
|-------------|----------|-----------|
| New variant added | Extend CVA schema | Additive change, no upstream conflict |
| Default props changed | Wrap with new defaults | Original preserved for re-add |
| Layout restructured | Fork and document | Structural change requires source modification |
| Accessibility enhanced | Fork, contribute upstream | Benefits the ecosystem |

## Anti-Patterns

- **Importing from node_modules**: shadcn components live in the project, not in a package. Importing from `node_modules` indicates a misunderstanding of the ownership model.
- **Dynamic class construction**: Expressions like `` bg-${color}-500 `` break Tailwind's static analysis. Use complete class strings with conditional logic instead.
- **CVA for single dimensions**: A ternary or simple conditional handles one variant axis more clearly than a CVA schema. Reserve CVA for two or more dimensions.
- **Skipping cn() with className props**: Components that accept `className` but apply it without `cn()` prevent consumers from overriding base styles. Always merge through `cn()`.

- SSOT (engineering module, `principles` profile: `design/reference/ssot.md`)
the react_router profile's *Component Composition* reference (`profiles/react_router/packages/components/reference/component_composition.md`)
- [Design Token Architecture](design_token_architecture.md)

## Sources

- https://ui.shadcn.com/docs/changelog/2026-03-cli-v4 — shadcn CLI v4: `init --base`, `--preset`, `--template`/`--monorepo`, `--dry-run`/`--diff`/`--view`, `info`, `docs`, `registry:base`/`registry:font`, shadcn/skills (2026-03; accessed 2026-11-22)
- https://ui.shadcn.com/docs/changelog/2026-07-base-ui-default — Base UI default for new projects, Radix fully supported (`init -b radix`), no migration required, `asChild` → `render`, per-component migration skill (2026-07; accessed 2026-11-22)
- https://ui.shadcn.com/docs/components-json — `style` (`new-york`; `default` deprecated), `tailwind.config` blank for v4, `tailwind.css`, `cssVariables`, `baseColor`, `aliases`, namespaced `registries` (accessed 2026-11-22)
