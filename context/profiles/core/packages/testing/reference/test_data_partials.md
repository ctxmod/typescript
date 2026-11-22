# Test-Data Partials

Test-data partial construction for unit tests — fromPartial/fromAny/fromExact replacing `as` assertions in test code only, migrated and then type-checked.

Procedure for building unit-test function arguments that satisfy the compiler while carrying only the data the test asserts on: typed partial builders replace `as` type assertions, so every provided property stays checked against the target type and the test states only what it needs. Run this migration when an `as` assertion is causing friction or a claim about the migrated tests needs proving; it is a refactoring step, not an automatic gate.

## The boundary with generated mocks

This procedure covers **unit-test function arguments** where no wire shape exists: an argument object the test constructs under its own control and passes straight to a function. Where a test needs request or response data that crosses a transport boundary, the generated mock handlers remain the only author of the wire shape and hand-shaped payloads stay prohibited. The two rules are complements, not exceptions.

[Generated Mock Validation](../../contracts/reference/generated_mocks.md)

## Why `as` fails in tests

Three problems with `as` assertions in test code:

- The compiler is trained out of the loop: an `as` is an unchecked claim accepted without evidence, so a shape drift the compiler would have caught passes silently.
- The target type is restated by hand at every cast site, so a rename or shape change breaks the cast even when the data itself is still valid.
- Intentionally wrong data is indistinguishable from a typo: `as unknown as Type` reads exactly like an accidental double cast at the call site.

## The three builders

| Builder | Use it when |
|---|---|
| `fromPartial()` | The test needs a subset of a large argument object and every provided property must type-check against the target |
| `fromAny()` | The test needs intentionally wrong data (an error-path case) while keeping autocomplete on the literal |
| `fromExact()` | The full object should be required — a placeholder that keeps the call compiling until the test supplies complete data |

The common case is the argument object with twenty properties where the test touches two: `fromPartial` states the two, and the compiler holds the rest against the target type without the test faking them.

```typescript
import { fromPartial } from "@total-typescript/shoehorn";

it("resolves the user from the request body", () => {
  getUser(fromPartial({ body: { id: "123" } }));
});
```

Deliberately wrong data keeps its own builder, so the intent is visible at the call site:

```typescript
import { fromAny } from "@total-typescript/shoehorn";

it("rejects a non-string id", () => {
  getUser(fromAny({ body: { id: 123 } }));
});
```

## Test code only

Partial builders exist for test arguments and are imported only from test files. Production code takes complete values: a production caller that appears to need a partial is missing a constructor or a default, never a candidate for a partial builder.

## Install

```bash
npm i @total-typescript/shoehorn
```

## Migration workflow

1. **Find the cast sites**: search test files for capitalized `as` assertions.

   ```bash
   grep -rn " as [A-Z]" --include="*.test.ts" --include="*.spec.ts" src/
   ```

2. **Replace** `as Type` with `fromPartial()` around the object literal, and `as unknown as Type` with `fromAny()` where the wrong data is deliberate.
3. **Add the imports** from `@total-typescript/shoehorn` to each migrated file.
4. **Verify**: run the type check — `tsc --noEmit` (or the project's `typecheck` script), per the typescript-validation reference. Zero errors completes the migration. The check is a step the migration claim needs, run on demand; it is never an automatic gate watching for casts.

[TypeScript Validation](../../toolchain/reference/typescript_validation.md)
