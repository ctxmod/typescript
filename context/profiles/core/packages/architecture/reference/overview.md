# TypeScript Module Overview

Base profile for ECMAScript/TypeScript frontend development with modern JavaScript patterns. The module answers to the names es, ts, js, typescript, and javascript.

The core profile is the contract-first ECMAScript application tier of this module, and every framework profile (nextjs, react_router, tanstack_start, shadcn, tailwind) specializes this tier rather than restating it. The architecture vocabulary used throughout — Command, Domain Event, Workflow State, Projection, Audit Record, the three idempotency levels — is defined in the [glossary](glossary.md).

## Browser tooling

A frontend-web session often also wants browsing and scraping reference context (Lightpanda light browsing, PandaScript extraction) — useful for gathering visual/design references and extracting content from existing sites. That context lives in the browser module (`github.com/ctxmod/browser/context`). This module does not extend it; select the browser module's profile alongside this one when a session needs it.

## Normative references

The reference layer is the source of architectural truth; the rules layer enforces it; the skills layer operationalizes it.

- **Architecture** — [contract-first architecture](../../contracts/reference/contract_first_architecture.md) · [generated-code discipline](../../contracts/reference/generated_code_discipline.md) · [guardrail placement](../../frontend/reference/guardrail_placement.md)
- **Contracts** — [generated clients](../../contracts/reference/generated_clients.md) · [OpenAPI consumption](../../contracts/reference/openapi_consumption.md)
- **Errors** — [problem details](../../frontend/reference/problem_details.md)
- **Frontend** — [frontend authority](../../frontend/reference/frontend_authority.md) · [live read model](../../frontend/reference/live_read_model.md)
- **Testing** — [generated mocks](../../contracts/reference/generated_mocks.md) · [contract drift gates](../../contracts/reference/contract_drift_gates.md) · [test effectiveness](../../testing/reference/test_effectiveness.md) · [test execution](../../testing/reference/test_execution.md) · [test-driven development](../../testing/reference/test_driven_development.md)
- **Localization** — [message catalogs](../../localization/reference/message_catalogs.md) · [localization compile pipeline](../../localization/reference/localization_compile_pipeline.md) · [locale strategies](../../localization/reference/locale_strategies.md) · [locale-text canonicalization](../../localization/reference/locale_text_canonicalization.md)
- **Language and package layer** — [module organization](../../modules/reference/module_organization.md) · [import and export conventions](../../modules/reference/import_export_conventions.md) · [application development](../../modules/reference/application_development.md) · [package development](../../modules/reference/package_development.md) · [project setup](../../toolchain/reference/project_setup.md) · [dependency analysis](../../modules/reference/dependency_analysis.md) · [TypeScript validation](../../toolchain/reference/typescript_validation.md) · [local server lifecycle](../../testing/reference/local_server_lifecycle.md) · [scratch probes](../../toolchain/reference/scratch_probes.md) · [deep-module design](../../modules/reference/deep_module_design.md) · [deepening shallow modules](../../modules/reference/deepening_shallow_modules.md) · [interface alternatives](../../modules/reference/interface_alternatives.md) · [dependency-boundary enforcement](../../modules/reference/dependency_boundary_enforcement.md)
- **Toolchain and orchestration** — [fast toolchain (Biome or Oxlint)](../../toolchain/reference/fast_toolchain.md) · [monorepo orchestration](../../toolchain/reference/monorepo_orchestration.md)
- **Skills** — [develop a Vite plugin](../../toolchain/skills/develop_vite_plugin/SKILL.md)
