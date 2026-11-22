# Framework Map

> **Updated:** YYYY-MM-DD
> **Coverage:** `{full / partial}` — {full: every area mapped and Not mapped yet empty — after onboard or `audit code` over the whole codebase with no narrowed scope or mode, or once a narrowed run or a docs sync leaves no area under Not mapped yet; partial: the areas mapped so far, the rest under Not mapped yet}
>
> {Lead summary, within `lead_lines` (see `references/docs-scaling.md`, dev-flow skill): what the project is built from — read before writing code to find the layer a change belongs to, the extension point to use, the shared utility that already exists.}

<!-- The project's architectural map: an overview with links down to concepts,
     `.dev_flow/rules/` and `.dev_flow/skills/`; no enforceable detail inlined —
     a rule's text lives in its rule file, the map links to it.
     Created and updated by the docs sync at task end (phases/propagate.md
     § Docs Sync at Task End → Framework map), built in full by onboard (Step 7)
     and the `audit code` scope (Step 9). See references/conventions.md
     § File Organization (dev-flow skill). An empty section keeps its heading
     with "— none yet". -->

## Layers

| Layer | Areas (paths) | May depend on | Concepts |
|-------|---------------|---------------|----------|
| 0 — {name, e.g. shared utilities} | `{path/}` | — | [C_XXX](xxx.concept.md) |
| 1 — {name} | `{path/}` | 0 | [C_YYY](yyy.concept.md) |

## Core Abstractions

| Abstraction | Where | What it is for | Detail |
|-------------|-------|----------------|--------|
| {Name} | `{path}` | {one line} | [C_XXX](xxx.concept.md) · `.dev_flow/rules/architecture.md` § {RuleName} |

## Extension Points

| Extension point | Where | How to extend | Detail |
|-----------------|-------|---------------|--------|
| {Name} | `{path}` | {one line: implement X, register it in Y} | {link to a concept, rule or skill} |

## Shared Utilities

| Utility | Where | Use it for |
|---------|-------|------------|
| {Name} | `{path}` | {one line} |

## Conventions

- {Convention in one line} — `.dev_flow/rules/{category}.md` § {RuleName}

## Not mapped yet

- `{path/}` — {a top-level area of the source tree from a listing; mapped by the next docs sync that touches it, or by onboard / `audit code`}
