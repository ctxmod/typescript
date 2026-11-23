---
paths:
  - "/intl/**"
---

# ECMAScript Localization

Localization discipline for the message catalog tree — namespace-per-directory, locale-per-file organization, structural completeness, and centralized locale detection.

Catalog files follow the namespace and locale axes and are structurally complete across the supported set; locale detection and negotiation are centralized in one place ahead of rendering, never re-derived per route or per component.

- [Message Catalogs](../reference/message_catalogs.md)
- [Locale Strategies](../reference/locale_strategies.md)
