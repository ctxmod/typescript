# TanStack Start Routing Context

File-based routing, route-module boundaries, rendering modes, and Vite composition for TanStack Start route files and configuration.

Route modules are full-stack boundaries: loaders and server functions own the server-side data and mutation seams, error components render the es error contract, and per-route rendering modes narrow monotonically down the route tree. Server-side data access goes through the generated client; a hand-written fetch wrapper on either side of the network is prohibited.

[Routing](../reference/routing.md)
[Rendering Modes](../reference/rendering_modes.md)
[Vite Configuration](../reference/vite_config.md)
[Generated Clients](../../../../core/packages/contracts/reference/generated_clients.md)
[Problem Details](../../../../core/packages/frontend/reference/problem_details.md)
