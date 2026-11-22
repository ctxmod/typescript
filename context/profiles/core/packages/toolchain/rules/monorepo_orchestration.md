---
paths:
  - "pnpm-workspace.yaml"
  - "*/pnpm-workspace.yaml"
  - "*/*/pnpm-workspace.yaml"
  - "*/*/*/pnpm-workspace.yaml"
  - "*/*/*/*/pnpm-workspace.yaml"
  - "*/*/*/*/*/pnpm-workspace.yaml"
  - "turbo.json"
  - "*/turbo.json"
  - "*/*/turbo.json"
  - "*/*/*/turbo.json"
  - "*/*/*/*/turbo.json"
  - "*/*/*/*/*/turbo.json"
  - "nx.json"
  - "*/nx.json"
  - "*/*/nx.json"
  - "*/*/*/nx.json"
  - "*/*/*/*/nx.json"
  - "*/*/*/*/*/nx.json"
---

# ECMAScript Monorepo Orchestration

Workspace catalog version SSOT, orchestrator choice, and task-graph ownership for workspace configuration and pipeline files.

Shared dependency versions pin once in the workspace `catalog:` table and are
referenced by protocol from package manifests — one library, one version row.
The orchestrator posture (graph-based suite or minimal-cache layer) is a
deliberate, recorded choice; task caching is only as sound as the declared
task inputs.

[Monorepo Orchestration](../reference/monorepo_orchestration.md)
[Module Organization](../../modules/reference/module_organization.md)
