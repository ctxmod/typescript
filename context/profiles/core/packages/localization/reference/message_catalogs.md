# Message Catalogs

Message catalogs as the localization source of truth — the namespace x locale two-axis layout, one catalog file per locale per namespace, dotted message identifiers with per-entry descriptions, ICU MessageFormat-class interpolations, repo-level placement outside every package, and the single settings declaration.

The message catalog tree is the single source of truth for every
human-readable string an application renders. It is authored, reviewed,
and versioned as source — never compiled by hand, never restated inside
application code, and never duplicated per consumer. This reference owns
the catalog layout, the entry shape, and where the tree lives relative to
the workspace.

## Two Axes: Namespace and Locale

A catalog tree is addressed along two independent axes, and the two axes
must not be collapsed into one dimension:

| Axis | Selects | Changes when |
|---|---|---|
| **Namespace** | Which slice of the interface the messages serve | The interface grows a new feature area |
| **Locale** | Which human language the messages are written in | A new language joins the supported set |

The axes answer different questions and change for different reasons. A
new feature area adds a row to every locale at once; a new language adds
a column to every namespace at once. Either edit is a mechanical,
reviewable addition when the axes are kept apart — and a scattered
multi-file hunt when they are not.

**The layout convention: one directory per namespace, one catalog file
per locale inside it.** A namespace directory holds exactly one file for
each supported locale, and nothing else. The namespace axis is expressed
by the directory, the locale axis by the file name. There is no third
shape: no per-locale directories nesting namespaces, no single flat file
mixing namespaces together.

Namespaces are named after the slice of the interface they serve, not
after the component that renders them, so a message survives the
component being renamed or relocated. Two granularities earn their keep
in most repositories:

- **a cross-cutting namespace** for messages reused across more than one
  area — shared actions, common field labels, generic error wording;
- **feature-scoped namespaces**, one per coherent area of the interface,
  each owning the messages only that area renders.

How many feature namespaces exist is an application decision. The rule
that transfers is the split itself: a message reused by more than one
area belongs to the shared namespace rather than copied into each
consumer, and a message rendered by exactly one area belongs to that
area's namespace and nowhere else.

## Catalog Entry Shape

Every entry carries the message itself **and a description of its use**,
duplicated identically in every locale's file for the same identifier:

- **a dotted, hierarchical identifier** — area-prefixed segments joined
  by a separator, so the identifier names where a message belongs and
  stays stable when the string it holds is reworded;
- **the message body** for that locale;
- **a description** documenting where the message renders and what
  arguments, if any, it interpolates — written for the translator, not
  the developer.

The description is the part most often dropped and most often regretted.
A translator given a bare string cannot know whether it stands alone,
sits mid-sentence, or interpolates a count that pluralizes — and a wrong
guess ships as a wrong translation. Because descriptions repeat across
every locale's file for the same identifier, keeping them identical is a
mechanical check, not a judgment call.

Message identifiers are consumer-facing the way an operation identifier
is on an API boundary: they are referenced from view code, they outlive
rewordings, and renaming one breaks every consumer at once. Rename
deliberately, never as an incidental tidy-up.

**Structural completeness is a hard requirement.** Every locale in the
supported set carries an entry for every identifier every other locale
carries, in every namespace. A missing entry is a build failure, not a
runtime fallback — the fallback strategy is a locale-strategies concern
and never a substitute for a complete catalog. The check runs at compile
or load time, so an incomplete catalog cannot reach a user.

## Interpolations

Placeholders inside a message body follow the **ICU MessageFormat**
family of conventions — `{placeholder}` for simple substitution, with the
standard selectors for plural forms and select variants. This is a format
family, not a library choice: any localization technology this profile
admits must express these constructs, and a catalog written against them
is portable across the implementation families described in the
compile-pipeline reference.

Two rules bind interpolation authorship:

- **Argument names are part of the message contract.** A placeholder name
  that changes is a change to every locale's entry for that identifier,
  made together and reviewed together.
- **Pluralization is expressed in the message, not in the caller.** The
  count travels into the message and the message selects its own form;
  the caller never pre-builds one string per count, and never concatenates
  a plural suffix onto a translated fragment.

Number, date, and currency formatting follow the active locale's standard
international formatting support — the formatting layer is selected by
locale, not re-implemented per message.

## Placement Relative to the Workspace

Message sources live **outside every package**, in a repository-level
tree of their own. Not inside an application's source tree, and not
inside the runtime package that consumes the compiled output — the
catalog tree is an input, and inputs are not owned by any one consumer.

```
<repo>/
  intl/messages/              # message sources; one directory per
                              # namespace, one catalog file per locale
                              # inside each namespace directory
  packages/                   # runtime consumers live among the others
  apps/
    <app>/
```

The tree follows the same one-way dependency rule as the contract-first
layout: the catalog source is an input tree no consumer owns, and the
generated localization output is a separate tree owned by the compile
step (the compile-pipeline reference owns that side). Package boundaries
and workspace wiring stay authoritative in the module-organization
reference ([Module Organization](../../modules/reference/module_organization.md));
this reference places only the catalog tree.

A repository hosting more than one application shares one catalog tree
rather than one per application. The namespace axis is the intended
partitioning mechanism — cross-cutting messages in the shared namespace,
each application's messages in its own. One tree declares the
supported-locale set exactly once; a tree per application reintroduces
the two-axis problem as a whole-tree problem.

## The Single Settings Declaration

The catalog tree is bound together by **one settings declaration** that
states, in a single machine-readable file:

- the **authoring locale** — the locale the source of truth is written
  in, and the one translators translate from;
- the **supported locale set** — the full list, including the authoring
  locale;
- the **catalog path pattern** — how the tree maps namespace and locale
  onto files, so tooling enumerates the whole tree from the pattern alone.

The declaration is the catalog tree's configuration as source. It is
reviewed with the tree, and a change to it is a contract change in
effect — adding a locale adds a file to every namespace. It is never
split across packages: a per-package declaration of the supported set is
how two halves of one interface come to disagree about which languages
exist. The authoring locale is named in the settings, never inferred
from directory order or the first file, because the completeness check
and the fallback chain both key off it.

## Catalogs Are Source, Not Output

- Catalog files are hand-authored and code-reviewed. They are never
  generated, never machine-reformatted wholesale, and never edited
  through a translation tool that rewrites the tree without a reviewable
  diff.
- Compiled localization output is a different artifact with a different
  owner — the compile-pipeline reference. The two trees never merge: a
  catalog edited inside a generated directory is lost on the next
  compilation, exactly as any other edit to generated output is.
- The editing prohibitions, the four properties, and the committed-
  versus-regenerated posture for generated localization output are owned
  by the generated-code discipline reference
  ([Generated-Code Discipline](../../contracts/reference/generated_code_discipline.md));
  this reference states the boundary and does not restate them.

## Where the Rest Lives

- The **compile-pipeline reference** owns how catalogs become compiled
  per-locale output, the compiler settings file, the drift gate, and the
  honest contrast with the runtime-lookup family.
- The **locale-strategies reference** owns detection, negotiation,
  fallback ordering, the single-locale versus multi-locale application
  profile decision, and degradation when a locale is not available.
- The **module-organization reference** owns the package boundaries and
  workspace wiring the catalog tree's placement must compose with.

## Related

- [Localization Compile Pipeline](localization_compile_pipeline.md)
- [Locale Strategies](locale_strategies.md)
