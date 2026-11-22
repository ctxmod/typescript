# Internationalization

The i18next middleware as the router-lens binding of the es centralized locale-detection principle — middleware wiring, server-side detection, route-loader locale, SSR hydration; catalog organization, pluralization, and RTL/BiDi live in the es localization home.

This file is the router lens of localization. The i18next middleware
wiring below is this router's binding of the es-level principle that
locale detection and negotiation are centralized — resolved once, ahead
of the loaders, at a single seam — and, as the runtime-lookup family's
concrete instance here, one admissible implementation of that principle
rather than a competing technology stance. The family-neutral law —
catalog organization, pluralization, RTL/BiDi, and the
server-render/hydrate/client rules — lives in the es localization home
and is linked below, not restated here.

- [Locale Strategies](../../../../core/packages/localization/reference/locale_strategies.md)
- [Message Catalogs](../../../../core/packages/localization/reference/message_catalogs.md)
- [Localization Compile Pipeline](../../../../core/packages/localization/reference/localization_compile_pipeline.md)
- SSOT (engineering module, `principles` profile: `design/reference/ssot.md`)

## Middleware Architecture

The middleware pattern uses `createI18nextMiddleware` to produce three
exports: the middleware function, a locale getter, and an i18next
instance getter. The middleware registers in `root.tsx`, the locale
getter serves loaders, and the instance getter configures the server
entry point.

```typescript
export const [i18nextMiddleware, getLocale, getInstance] =
  createI18nextMiddleware({
    detection: {
      supportedLanguages: ["en", "sv"],
      fallbackLanguage: "sv",
      cookie: localeCookie,
    },
    i18next: {
      resources: { en: { translation: en }, sv: { translation: sv } },
    },
  });
```

This single configuration point is the centralized principle's binding
in this router: one resolution site, ahead of every loader, so no route
re-derives the locale. The middleware initializes a fully configured
i18next instance on the server before rendering, preventing translation
flickering during SSR.

## Detection Wiring

Which sources detection consults, in what order, and how a preference
negotiates onto the supported set are declared at the es level — this
lens owns only how the middleware declares its inputs:

- The detection options name the supported languages, the fallback
  language, and the cookie carrying the persisted user preference.
- The locale cookie is configured `httpOnly`, `secure`, and
  `sameSite: "lax"`. A language-switcher action writes it and triggers a
  document reload so the middleware re-resolves on the next request.
- Where the application encodes the locale in its URLs, a `:locale`
  route parameter feeds the URL segment into the same resolution; the
  root index redirect resolves first visits through the declared chain
  rather than hard-coding a default.

## SSR Hydration

The client entry inherits the server's resolved locale rather than
re-negotiating it:

- Detect the initial language from the URL path or the `lang` attribute
  on the `<html>` element the server rendered, initialize i18next
  synchronously, and only then call `hydrateRoot`.
- Bundle the namespaces the first view renders into the client entry
  rather than fetching them asynchronously.
- The `useChangeLanguage` hook in the root component keeps the client
  instance aligned with the server-resolved locale across subsequent
  navigations — synchronization flows through the middleware's seam, not
  per-view re-derivation.

## Runtime Thresholds

These bands are this lens's runtime escalation signals for review, not
a substitute for the es-level build-time completeness gate, which
remains the authority on whether a catalog may ship:

| Metric | Pass | Warn | Fail |
|--------|------|------|------|
| Namespaces loaded per route | 1-2 | 3-4 | >4 |
| Translation key depth | 1-2 levels | 3 levels | >3 levels |

## Where the Family Law Lives

Everything this lens defers to is owned by the es localization home:

- [Locale Strategies](../../../../core/packages/localization/reference/locale_strategies.md) — the
  centralized detection-and-negotiation principle this middleware
  implements, the detection chain and its declared order, the fallback
  ladder, single- versus multi-locale application profiles,
  pluralization, and RTL/BiDi discipline.
- [Message Catalogs](../../../../core/packages/localization/reference/message_catalogs.md) — the
  catalog tree: the namespace and locale axes, entry shape, structural
  completeness, placement, and the single settings declaration.
- [Localization Compile Pipeline](../../../../core/packages/localization/reference/localization_compile_pipeline.md)
  — the two admissible implementation families and the honest contrast
  that frames this middleware as one binding of the principle, not a
  stance.

## Related

- [Route Module Design](../../routes/reference/route_module_design.md)
- [Deployment](../../deployment/reference/deployment.md)
