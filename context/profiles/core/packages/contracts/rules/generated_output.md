---
paths:
  - "gen/*"
  - "gen/*/*"
  - "gen/*/*/*"
  - "gen/*/*/*/*"
  - "gen/*/*/*/*/*"
  - "*/gen/*"
  - "*/gen/*/*"
  - "*/gen/*/*/*"
  - "*/gen/*/*/*/*"
  - "*/*/gen/*"
  - "*/*/gen/*/*"
  - "*/*/gen/*/*/*"
  - "*/*/*/gen/*"
  - "*/*/*/gen/*/*"
  - "*/*/*/*/gen/*"
---

# ECMAScript Generated Output

Generated-code discipline for the repository's generated output tree — never hand-edited, regenerated from contract source.

The generated tree is written only by the generator. A change starts in the contract source and is regenerated; editing generated output is erased by the next regeneration at best and survives as undetected drift at worst.

- [Generated-Code Discipline](../reference/generated_code_discipline.md)
- [Generated Clients](../reference/generated_clients.md)
