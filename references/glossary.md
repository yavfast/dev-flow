# Project Glossary — `docs/_glossary.md`

The project's **canonical domain vocabulary**, shared across every concept and spec.
It is a glossary of *terms* — what a word means — not a model of how entities relate
(that lives in each concept's Domain Model). One file: `docs/_glossary.md`.

## Why it exists

dev-flow already demands "domain language" and per-concept domain models, but nothing
reconciles vocabulary **across** concepts — so the same referent drifts into different
words in different `*.concept.md` files ("account" here, "customer" there). The glossary
is the single source of truth a naming rule can actually be checked against, and it cuts
tokens by killing "same thing, different word" churn.

It sits beside `docs/_index.md`, on a different axis: `_index.md` catalogs **documents**
(which files exist); `_glossary.md` catalogs **terms** (what words mean). Many-to-many —
one document introduces many terms; one term appears in many documents.

## Loading

`docs/_glossary.md` is **loaded into context together with `docs/_index.md`** — wherever
a phase reads the catalog (concept/spec authoring, onboard, audit), it reads the glossary
too. It loads whenever it exists, even before a project has enough documents to warrant
`_index.md`. That standing presence is what lets authoring *challenge against the glossary*.

## Format

```md
# {Project} — Glossary

> Canonical domain vocabulary, shared across all concepts. Loaded with _index.md.

## Terms

**{Term}**
{One or two sentences — what it IS, not what it does.}
_Avoid_: {aliases that must not be used for this concept}

## Flagged ambiguities

- **{Term A vs Term B}** — {the conflict}; {the resolution, or "← decision pending"}.
```

Group terms under sub-headings only when natural clusters emerge; a flat list is fine.

## Rules

- **Opinionated.** When several words exist for one concept, pick the best and list the
  rest under `_Avoid_`. The glossary's job is to *end* the synonym debate, not catalog it.
- **Tight.** One or two sentences. Define what the term **is**, not what it does.
- **Project-specific only.** Include a term only if it is a concept unique to this
  project's domain. General programming/infrastructure/architecture mechanisms (timeouts,
  interceptors, DI, a vendor-isolation pipeline) do **not** belong — document those in the
  relevant concept. Ask: "domain noun, or general tech?" Only the former.
- **No implementation, not a spec.** The glossary is a dictionary and nothing else — no
  fields, no contracts, no decisions. Relationships belong to the concept; at most note
  obvious cardinality.
- **Flag conflicts.** A term used two ways goes under *Flagged ambiguities* with a
  resolution — or, if the canonical choice is unsettled, as a decision (see below).
- **Lazy.** Don't mandate a glossary up front. Create `docs/_glossary.md` when the first
  cross-concept term is resolved (or populate it during onboard). A one-concept project
  rarely needs one — like `_index.md`, it earns its place.

## Lifecycle

- **Onboard** — extract the project's domain terms from the codebase into
  `docs/_glossary.md`, alongside rule extraction (see [onboard phase](../phases/onboard.md)).
- **Concept authoring** — with the glossary loaded, challenge every domain noun against
  it: reuse the canonical term, add genuinely new terms inline, and flag conflicts (see
  [concept phase](../phases/concept.md)).
- **Propagate** — if a change renames or retires a domain term, update the glossary so it
  never lags the documents (see [propagate phase](../phases/propagate.md)).
- **Audit** — groom the glossary: merge duplicate terms, resolve stale *Flagged
  ambiguities*, and strip any implementation detail that crept in (see [audit phase](../phases/audit.md)).

## Challenge against the glossary

When authoring uses a term that conflicts with the glossary, call it out — but most term
work is **not** an interview. Picking a canonical term, adding one, or applying an
existing `_Avoid_` alias is a cheap, reversible naming choice the glossary itself settles;
just use the canonical term and move on (this is exactly what [Interview Mode](./interview-mode.md)
means by "a convention already settles it / skip a rename" under *Do NOT interview on*).

Escalate to Interview Mode **only when the conflict is material** — when it reveals two
genuinely different concepts being conflated ("is *cancellation* the same as *refund*?"),
or when the canonical choice shapes a contract/model and is hard to reverse. Then surface
it with a recommended term and update the glossary once the developer decides. The
developer owns the project's language; routine naming just follows the glossary.

The matching code-side rule — *naming follows the glossary* — belongs in
`.dev_flow/rules/naming.md`.
