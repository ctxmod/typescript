# Deep Module Design

Deep-module design vocabulary for TypeScript — module, interface, seam, adapter, depth, leverage, locality — and the principles that put a lot of behaviour behind a small interface.

Design **deep modules**: a lot of behaviour behind a small interface, placed at a clean seam, testable through that interface. The aim is leverage for callers, locality for maintainers, and testability for everyone. Use this vocabulary wherever code is designed or restructured.

## Terminology bridge

Deep-module **interface** means everything a caller must know to use the module correctly: the type signature plus invariants, ordering constraints, error modes, required configuration, and performance characteristics. That is broader than the wire contract owned by [contract-first architecture](../../contracts/reference/contract_first_architecture.md), which governs the transport-level API surface a client and server share. The two are complementary, not competing: a module's interface may wrap exactly one contract operation or none, and a contract-first repository still needs depth decisions about what sits behind each contract. Nothing here rewrites contract discipline — depth decisions start where the contract ends.

## Glossary

Use these terms exactly. Consistent language is the point; "component", "service", "API", and "boundary" each blur a distinction these terms carry.

- **Module** — anything with an interface and an implementation, deliberately scale-agnostic: a function, class, package, or tier-spanning slice.
- **Interface** — everything a caller must know to use the module correctly: the type signature plus invariants, ordering constraints, error modes, required configuration, and performance characteristics.
- **Implementation** — what is inside a module, its body of code.
- **Adapter** — a concrete thing that satisfies an interface at a seam. It names the role (what slot it fills), not the substance: a small adapter can carry a large implementation (a database repository), a large adapter a small one (an in-memory fake). Reach for "adapter" when the seam is the topic and "implementation" otherwise.
- **Depth** — leverage at the interface: the amount of behaviour a caller or test exercises per unit of interface it must learn. A module is **deep** when much behaviour sits behind a small interface, **shallow** when the interface is nearly as complex as the implementation.
- **Seam** — a place where behaviour can be altered without editing that place; the location where a module's interface lives. Where the seam goes is its own design decision, distinct from what sits behind it.
- **Leverage** — what callers get from depth: more capability per unit of interface learned; one implementation pays back across every call site and test.
- **Locality** — what maintainers get from depth: change, bugs, knowledge, and verification concentrate in one place instead of spreading across callers; fix once, fixed everywhere.

The terms relate as follows: a module presents exactly one interface; depth is measured against that interface; the seam is where the interface lives; adapters sit at the seam and satisfy the interface; depth produces leverage for callers and locality for maintainers.

## Deep versus shallow

**Deep module** — small interface, lots of implementation:

```
┌─────────────────────┐
│   Small Interface   │  ← few methods, simple parameters
├─────────────────────┤
│                     │
│ Deep Implementation │  ← complex logic hidden
│                     │
└─────────────────────┘
```

**Shallow module** — large interface, thin implementation:

```
┌─────────────────────────────────┐
│        Large Interface          │  ← many methods, complex parameters
├─────────────────────────────────┤
│  Thin Implementation            │  ← passes through
└─────────────────────────────────┘
```

When designing an interface, reduce the number of methods, simplify the parameters, and hide more complexity inside.

## Principles

- **Depth is a property of the interface, not the implementation.** A deep module can be internally composed of small, swappable parts; they stay out of the interface. A module can carry **internal seams** — private to its implementation, used by its own tests — alongside the **external seam** at its interface.
- **The deletion test.** Imagine deleting the module: if the complexity vanishes, it was a pass-through; if the complexity reappears in every caller, it was earning its keep.
- **The interface is the test surface.** Callers and tests cross the same seam. Wanting to test past the interface signals the module is the wrong shape.
- **Two adapters make a seam real.** One adapter means a hypothetical seam; introduce a seam only when something actually varies across it.

## Designing for testability

1. **Accept dependencies; do not create them.**

   ```typescript
   // Testable: the dependency arrives through the interface
   function processOrder(order: Order, paymentGateway: PaymentGateway) {}

   // Hard to test: the dependency is constructed inside
   function processOrder(order: Order) {
     const gateway = new StripeGateway();
   }
   ```

2. **Return results; do not produce side effects.**

   ```typescript
   // Testable: the result comes back through the interface
   function calculateDiscount(cart: Cart): Discount {}

   // Hard to test: the effect is hidden in the parameter
   function applyDiscount(cart: Cart): void {
     cart.total -= discount;
   }
   ```

3. **Keep the surface small.** Fewer methods mean fewer tests to write; fewer parameters mean simpler test setup.

## Rejected framings

- Depth as a ratio of implementation lines to interface lines rewards padding the implementation; depth means leverage at the interface.
- "Interface" as the TypeScript `interface` keyword or a class's public methods is too narrow: an interface includes every fact a caller must know.
- "Boundary" names enforced import rules, not design locations — say **seam** for where an interface lives, and reserve boundary for the enforcement sense in [dependency boundary enforcement](dependency_boundary_enforcement.md).

## Going deeper

- Deepening a cluster of shallow modules given its dependencies: [Deepening Shallow Modules](deepening_shallow_modules.md)
- Exploring alternative interfaces in parallel before committing: [Interface Alternatives](interface_alternatives.md)
- Testing a module through its public interface: [Test-Driven Development](../../testing/reference/test_driven_development.md)

## Related

- [Module Organization](module_organization.md)
