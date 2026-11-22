# Locale Strategies

Centralized locale detection and negotiation as the technology-neutral normative principle both implementation families serve — the detection chain and where it runs, the fallback ladder, single-locale versus multi-locale application profiles, CLDR plural categories, RTL/BiDi support, and server-render/hydrate/client considerations including locale flash.

**Locale detection and negotiation are centralized.** An application
resolves the active locale exactly once, in one place, before any
rendering or data loading consults it, and every consumer — loaders,
views, formatting, document metadata — receives the resolved result from
that one place. This is the technology-neutral normative principle of
localization in this profile, and it binds every application regardless
of which implementation family turns catalogs into rendered messages.

The centralized principle is what both admissible implementation
families implement. The compile-step family resolves the locale once and
selects compiled per-locale output under it; the runtime-lookup family
resolves the locale once and initializes its lookup surface under it.
Neither family is exempt, and neither satisfies the principle more
correctly than the other — the resolution site is a placement decision
that precedes the family choice.

The failure the principle prevents is per-route, per-component, or
per-loader detection. When each consumer re-derives the locale, two
consumers eventually derive different answers on the same request: the
document says one language while a lazily loaded area says another, the
fallback fires in one place and not the next, and no single site is
answerable for which locale the user is seeing. Centralizing resolution
makes the locale a resolved input to rendering rather than a question
each renderer re-asks.

## The Detection Chain

Detection consults sources in a **fixed, declared priority order**. A
typical chain, from highest to lowest precedence:

1. **An explicit user preference** persisted on the client — a cookie
   written when the user chose a language. The cookie carries only the
   locale identifier, never message data.
2. **A URL-derived locale** where the application encodes the locale in
   its URLs. A URL is the most shareable expression of a locale — it
   survives bookmarks and links — so it is what a visitor arriving
   without a persisted preference resolves from. A visitor who has
   expressed a choice keeps it: the persisted preference outranks the
   URL, so following a link into another locale does not silently switch
   a user who has already chosen.
3. **A client-supplied language preference** — the header a browser
   sends listing the user's preferred languages, ranked.
4. **The authoring locale as fallback**, the final answer when no other
   source speaks.

The exact order is a declared property of the application, stated in one
place with the resolution code. What is not negotiable is that the chain
is total — every request ends with a locale, never an unresolved
question — and that **negotiation reduces an arbitrary language
preference onto the supported set**. A preference naming a language the
application does not support resolves onto the nearest supported locale,
using regional-form matching (a preference for a regional variant
resolves onto the base language, and vice versa) rather than dropping
straight to the authoring locale.

## Where Detection Runs

Detection runs **on the server, before rendering**, wherever the
application renders server-side. The centralized resolution site sits in
front of the rendering pipeline — in entry middleware or the equivalent
single seam — so that:

- loaders and data fetching consult an already-resolved locale and fetch
  locale-appropriate data rather than re-deriving it;
- server-rendered HTML carries the resolved locale in its document
  attributes, which is the seed the client reads;
- no rendering path bypasses the seam, because there is no second place
  resolution happens.

A purely client-rendered application runs the same resolution at its
client entry point, once, before the first render — the placement rule
is about there being one site, not about which side of the network it
sits on.

## The Fallback Ladder

Fallback operates at three distinct levels, and conflating them is how
an interface ends up rendering a raw identifier to a user:

| Level | What is missing | The fallback | The remedy |
|---|---|---|---|
| **Message** | One identifier in the active locale's catalog | The same identifier in the fallback locale | Fix the catalog — the completeness check should have caught it |
| **Locale** | The negotiated locale is not in the supported set | Negotiate onto the nearest supported locale | Declared negotiation, not a silent default |
| **Degradation** | The application runs a single-locale configuration | Render entirely in that locale; locale machinery is a no-op | First-class mode, below |

**A missing message is a defect, not a strategy.** The message-level
fallback exists so an incomplete catalog degrades to a readable other
language instead of an identifier — but every fallback rendering is a
finding. The catalog completeness check (message-catalogs reference)
exists precisely so this level fires only between the introduction of a
gap and its fix.

## Single-Locale versus Multi-Locale Profiles

Whether an application supports one locale or many is an
**application-profile decision**, made from what the application is for
and recorded with it. Neither profile is the more correct one.

**A single-locale application** fixes its locale at build time. The
detection and negotiation machinery is a no-op: no locale switching, no
per-locale URLs, no preference persistence, no alternate-catalog
emission. Declaring the mode — rather than simply not bothering — is
what makes the machinery degrade **deliberately and completely**:
everything that would consult a second locale consults a single verdict
source and does nothing, rather than half-running and emitting empty
alternates or dead links for locales that do not exist. Degradation is a
first-class, declared mode with one source of truth for the degraded
verdict, not an accident of configuration.

**A multi-locale application** resolves per request or per session and
supports switching. The costs it accepts are real: locale-appropriate
URLs (and their reciprocal alternates), the full detection chain,
per-locale document metadata, and hydration that must not contradict
what the server rendered. The payoff is an interface each user reads in
their own language.

A single-locale application that might one day be multi-locale is still
a single-locale application today. Building the multi-locale machinery
on speculation pays its costs now for a benefit that may never arrive —
but writing catalogs against the shared rules (message-catalogs
reference) keeps the door open at near-zero cost, which is the
proportionate hedge.

## Pluralization: CLDR Plural Categories

Plural forms follow the **CLDR plural category rules** — the standard
partition of cardinal numbers into categories such as one, other, and
the additional categories some languages require (few, many, and the
dual forms). The count of plural categories varies by language: some
languages need two, others six, and a language's set is a property of
the language, not a choice.

The binding rules:

- **Every plural form a language requires is defined for every
  pluralized message in that language's catalog.** The completeness
  requirement extends to plural categories: an entry that selects on
  count carries all the categories its language uses, or the
  completeness check fails.
- **The message selects its own form.** The count is interpolated into
  the message and the message chooses among its categories; the caller
  never branches on the count to pick between two separately identified
  messages, and never appends a plural suffix to a translated fragment.
  A caller-side branch encodes one language's plural rules into
  application code, where every other locale silently mis-renders.
- **A copied rule is a wrong rule.** No language's category boundaries
  are inferred from another's. The rules come from the standard plural
  data; they are not hand-derived per project.

Number, date, and currency formatting follow the active locale's
standard international formatting support, selected by locale rather
than re-implemented per message. A message interpolates a value and
names its formatting intent; the formatting layer resolves the intent
against the locale.

## RTL and BiDi Support

Where any supported locale reads right-to-left, directionality is a
document-level property driven by the active locale, applied as layout
rules rather than per-element overrides:

- **The document element carries the locale and its direction.** The
  rendered document states which language it is in and which direction
  it reads, derived from the resolved locale — never hardcoded to one
  direction, never set per component.
- **Layout is written in logical properties.** Inline-start and
  inline-end, block-start and block-end, and logical border and corner
  properties, so a direction change re-flows the layout by rule rather
  than by a stylesheet of per-direction overrides. Physical-direction
  properties are reserved for the rare genuinely direction-independent
  element.
- **Directional icons flip under the active direction.** Arrows and
  chevrons that mean "forward" and "back" mirror when the reading
  direction mirrors; icons whose meaning is not directional do not.
- **Bidirectional text is handled where it occurs.** Embedded runs of
  the opposite script inside a message are isolated so their direction
  does not leak into the surrounding text — isolation is part of message
  authorship, not a rendering afterthought.

An application whose supported set contains no right-to-left locale owes
none of this — but the logical-properties rule costs nothing to follow,
which is why it is stated as layout discipline rather than as
localization machinery.

## Server Render, Hydration, and the Client

The locale must survive the transition from server rendering to client
hydration without a flash. Three rules close the gap:

- **The server resolves, the client inherits, then the client takes
  over.** The server resolves the locale and renders under it; the
  client reads the resolved locale from the document rather than
  re-detecting it, so the first client render agrees with what the
  server produced; only after hydration does a client-side change of
  preference take effect, through the same centralized seam that
  resolved the initial locale.
- **The initial client render never re-negotiates.** A client that
  re-runs detection on mount can disagree with the server — a different
  cookie view, a different header reading — and the disagreement renders
  as a visible language change after first paint. The document the
  server rendered is the authority for the first client render.
- **Catalogs needed for first paint are available synchronously.** The
  namespaces the initial view renders are loaded before hydration
  completes — bundled or fetched ahead of the first render — so no
  message renders as an identifier for a moment and then pops into
  language. An asynchronous catalog load introduces a timing gap between
  render and translation, and the gap is visible.

Keeping the active locale synchronized across subsequent client-side
navigations is the centralized seam's job, not each view's: navigation
carries the locale through the seam, and a view that re-derives it has
reintroduced the per-consumer detection the principle prohibits.

## Where the Rest Lives

- The **message-catalogs reference** owns the catalog tree, the entry
  shape, the completeness check that keeps the message-level fallback
  rare, and the supported-locale declaration.
- The **compile-pipeline reference** owns the two implementation
  families that render under the resolved locale, and the boundary
  between generated output and handwritten localization code.
- The **frontend-authority reference** owns the wider interface honesty
  rules this profile's rendering obligations sit inside; locale
  resolution is an input to rendering, not a form of frontend authority.

## Related

- [Message Catalogs](message_catalogs.md)
- [Localization Compile Pipeline](localization_compile_pipeline.md)
- [Frontend Authority](../../frontend/reference/frontend_authority.md)
