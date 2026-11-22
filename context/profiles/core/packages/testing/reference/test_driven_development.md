# Test-Driven Development

Test-driven development for TypeScript — good tests through public interfaces, tests at pre-agreed seams, the anti-patterns, loop rules, and mocking at system boundaries.

TDD is the red → green loop; this reference makes the loop produce tests worth keeping: what a good test is, where tests go, the anti-patterns, and the rules of the loop. Every section applies on every cycle — consult it before and during the loop, not after. Before the loop, read the project's own documentation and context for the area so test names and interface vocabulary match the project's domain language, and respect the area's recorded decisions (ADR discipline: engineering module, `repositories` profile: `structure/reference/adr_discipline.md`).

## Precedence: the repository's test policy

Where verification policy intersects — test persistence, tiering, and execution — the repository's own test policy wins, and this reference contributes TS/JS test-design practice only. Nothing here mandates hook, pre-commit, pre-push, or CI test runs, or where tests are stored. Contract-generated wire-shape mock handlers keep their own doctrine in [generated mocks](../../contracts/reference/generated_mocks.md); this reference owns when and where to mock at system boundaries.

## What a good test is

Tests verify behaviour through public interfaces; implementation details can change entirely without the tests changing. A good test states a capability: "user can checkout with valid cart" tells you exactly what exists, and it survives refactors because it carries nothing about internal structure.

```typescript
// Good: asserts observable behaviour through the public function
test("user can checkout with valid cart", async () => {
  const cart = createCart();
  cart.add(product);
  const result = await checkout(cart, paymentMethod);
  expect(result.status).toBe("confirmed");
});
```

Characteristics: tests behaviour callers care about; touches public interfaces only; survives internal refactors; describes what, not how; one logical assertion per test.

## Seams: where tests go

A **seam** is the public boundary you test at — the interface where behaviour is observed without reaching inside; [deep module design](../../modules/reference/deep_module_design.md) owns the vocabulary. Tests live at seams, never against internals.

Test only at pre-agreed seams. Before any test is written, the seams under test are named and agreed during intent capture and recorded in the work item; no test lands at an unagreed seam. Testing effort cannot cover everything, so agreeing the seams up front lands it on the critical paths and complex logic instead of every edge case.

When the shape of the interface is itself in question — how deep the module is, where the seam belongs, what the interface should expose — settle it with deep module design first, then write tests at the settled seam.

## Anti-patterns

- **Implementation-coupled**: mocks an internal collaborator, tests a private method, or verifies through a side channel. The tell: the test breaks under refactor while behaviour has not changed.

  ```typescript
  // Implementation-coupled: stubs an internal collaborator, asserts the call
  test("checkout calls payment processing", async () => {
    const paymentService = { process: recordingStub() };
    await checkout(cart, paymentService);
    expect(paymentService.process.calls).toEqual([[cart.total]]);
  });
  ```

- **Tautological**: the expected value is recomputed the way the code computes it, so the test passes by construction and can never disagree with the code. Expected values come from an independent source: a known-good literal, a worked example.

  ```typescript
  // Tautological: the assertion mirrors the implementation
  test("calculateTotal sums line items", () => {
    const items = [{ price: 10 }, { price: 5 }];
    expect(calculateTotal(items)).toBe(items.reduce((sum, i) => sum + i.price, 0));
  });

  // Independent: the expected value is a known literal
  test("calculateTotal sums line items", () => {
    expect(calculateTotal([{ price: 10 }, { price: 5 }])).toBe(15);
  });
  ```

- **Horizontal slicing**: all tests first, then all implementation. Bulk tests verify imagined behaviour — the shape of things rather than user-facing behaviour — and commit to test structure before the implementation is understood. Work in **vertical slices** instead: one test, one implementation, repeat; each test is a **tracer bullet** that responds to what the last cycle taught.

## Rules of the loop

- **Red before green.** Write the failing test first, then only enough code to pass it — nothing speculative, no features a future test might want.
- **One slice at a time.** One seam, one test, one minimal implementation per cycle.
- **Refactoring sits outside the loop.** Restructuring belongs to the review workflow, not the red → green cycle; enter review with green tests and leave it with them still green.

## Mock at system boundaries

Mock at system boundaries only: external services (payment, email), time and randomness, and — where no real substitute exists — databases and the file system. Reach your own modules and internal collaborators through real code at the seam; they need no substitutes.

Design boundary interfaces for mockability:

1. **Accept dependencies; do not create them.**

   ```typescript
   // Easy to substitute: the client arrives through the interface
   function processPayment(order: Order, paymentClient: PaymentClient) {
     return paymentClient.charge(order.total);
   }

   // Hard to substitute: the client is constructed inside
   function processPayment(order: Order) {
     const client = new PaymentClient(process.env.PAYMENT_KEY);
     return client.charge(order.total);
   }
   ```

2. **Prefer SDK-style interfaces over generic fetchers.** One specific function per external operation keeps every substitute single-shaped and typed:

   ```typescript
   // Each function is independently substitutable
   const api = {
     getUser: (id: string) => fetch(`/users/${id}`),
     getOrders: (userId: string) => fetch(`/users/${userId}/orders`),
     createOrder: (data: OrderDraft) => fetch("/orders", { method: "POST", body: data }),
   };
   ```

   A generic `fetch(endpoint, options)` surface pushes conditional logic into every substitute and hides which endpoints a test exercises.

For interface-shape questions at a boundary — depth, seam placement, what to expose — deep module design carries the vocabulary; for contract-generated wire-shape handlers, generated mocks carries the doctrine.
