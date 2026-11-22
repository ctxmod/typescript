# Locale-Text Canonicalization

Turning free natural-language text into canonical values — refuse rather than guess when the vocabulary cannot be enumerated, let a colon or label scope everything it announces until the sentence ends, read qualifiers against the values they scope, and match Unicode word boundaries with letter-class lookarounds.

Parsing human-written text into canonical values is a guard, not a
convenience. A wrong value that looks valid is worse than no value: it
flows downstream and nothing flags it.

## Refuse, Don't Guess

- When the natural-language vocabulary cannot be fully enumerated, an
  unresolvable term never falls through to a default or a nearest match.
- The parse result becomes `unparseable` with a named reason — the
  unresolved term and the rule that failed — so the caller can report,
  queue for review, or ask for clarification.
- A partial parse is not a success. If one term in a list is unknown, the
  whole list is `unparseable`; returning the recognised subset silently
  drops the rest.
- Keep the reason machine-readable (a stable code) plus the offending
  input; tests assert the reason, not only the failure.

## A Label Scopes Until the Sentence Ends

- A colon or label announces a set: in `Sizes: small, medium` the label
  `Sizes` scopes every value after the colon until the sentence ends.
- Do not bind the label to the first value only. `small` and `medium` are
  both sizes; a parser that stops at the first comma misreads the second
  value as unlabelled free text.
- The scope ends at the sentence boundary or at the next label, whichever
  comes first. A new label opens a new scope; it never extends the old one.

## Read Qualifiers Before Accepting a Set

- A qualifier changes the meaning of the values it scopes: `all sizes
  except large`, `only small`, `medium or larger`.
- Read the qualifier against the values it scopes before a complete set is
  accepted. Enumerating the values and then ignoring `except` inverts the
  result.
- A qualifier the parser does not recognise makes the set `unparseable`
  with a named reason; it is never dropped as noise.

## Unicode Word Boundaries

- `\b` is defined over ASCII word characters. Next to a non-ASCII letter
  it reports a boundary inside the word, so `\bcaf\b` matches inside
  `café`, and a pattern ending in `é` misses a real boundary.
- The `u` flag does not fix `\b`; it enables Unicode escapes and code
  point matching, but `\b` stays ASCII-based.
- Use letter-class lookarounds instead:
  `(?<!\p{L})café(?!\p{L})` matches `café` as a whole word and rejects it
  inside a longer word. Add `\p{N}` or `\p{M}` to the class when digits or
  combining marks belong to words in your vocabulary. In JavaScript the
  property escape itself needs the `u` (or `v`) flag.
- Normalise input (for example to NFC) before matching, so `naïve` written
  with a combining diaeresis matches the precomposed form.
- Match against the normalised, case-folded text, but report the original
  span in any `unparseable` reason.

## Related

- [Locale Strategies](locale_strategies.md)
- [Message Catalogs](message_catalogs.md)
- [Guardrail Placement](../../frontend/reference/guardrail_placement.md)
