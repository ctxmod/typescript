# Deepening Shallow Modules

Procedure for deepening a cluster of shallow TypeScript modules — classify dependencies into four categories, keep seam discipline, and replace rather than layer the tests.

Merge a cluster of shallow modules into one deep module without losing its tests. The vocabulary — module, interface, seam, adapter, depth — is [Deep Module Design](deep_module_design.md). When several interface shapes are worth exploring before committing to one, run [Interface Alternatives](interface_alternatives.md) first.

## 1. Classify the dependencies

Every dependency of the candidate cluster falls into one of four categories; the category decides how the deepened module is tested across its seam.

| Category | Shape | Testing across the seam |
|---|---|---|
| In-process | Pure computation, in-memory state, no I/O | Merge the modules and test through the new interface directly; no adapter needed |
| Local-substitutable | A local test stand-in exists (an embedded database, an in-memory filesystem) | Run the deepened module's tests against the stand-in; the seam stays internal — no port at the module's external interface |
| Remote but owned | Your own services across a network boundary (internal APIs, queues) | Define a port at the seam; the deep module owns the logic, an in-memory adapter serves tests, an HTTP/gRPC/queue adapter serves production |
| True external | Third-party services outside your control | Inject the external dependency as a port; tests provide a mock adapter |

A port is an interface defined at the seam; an adapter is a concrete thing satisfying it. "Remote but owned" is worth the port even though you control both sides: the logic concentrates in one deep module while the transport stays swappable.

## 2. Keep seam discipline

- Two adapters make a seam real: introduce a port only when at least two adapters are justified — production and test is the canonical pair. A single-adapter seam is indirection.
- Internal seams stay internal: a deep module may carry seams private to its implementation for its own tests, and the interface exposes only the external seam.

## 3. Replace, do not layer, the tests

Tests at the deepened module's interface replace the tests the shallow modules carried:

- Delete the old unit tests once interface tests exist; they assert structure that no longer exists and become permanent suite weight.
- Write the new tests at the deepened module's interface — the interface is the test surface.
- Assert observable outcomes through the interface, not internal state.
- A test that must change when the implementation changes is testing past the interface: move it to the interface or delete it.

Done when the merged module exposes one interface, every kept test crosses that interface, and the replaced tests are deleted rather than parked.

## Related

- [Dependency Boundary Enforcement](dependency_boundary_enforcement.md)
