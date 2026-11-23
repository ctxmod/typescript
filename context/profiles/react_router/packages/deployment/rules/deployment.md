# Deployment Context

Deployment principles and Cloudflare Workers procedures for deployment configuration and entry point files.

Cloudflare Workers deployment principles, the deployment procedure, and environment setup govern worker handlers, build configuration, and SSR entry points. On React Router v8 the worker seeds a `RouterContextProvider` with a typed Cloudflare context (`context.get(...)` in loaders), and the config carries no `future.v8_*` flags.

- [Deployment](../reference/deployment.md)
- [Cloudflare Workers Deployment](../reference/cloudflare_workers.md)
- [Environment Setup](../reference/environment_setup.md)
