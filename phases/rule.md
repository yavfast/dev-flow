# Phase: Rule — Manage Project Coding Rules

## Purpose

Add, edit, or remove coding rules in `.dev_flow/rules/`. Rules define naming conventions, code structure patterns, architectural constraints, and style requirements that all new code must follow. The review phase validates code against these rules.

## Command

```
/dev-flow rule <request>
```

The request is a freeform description in any language. Interpret the intent and apply the appropriate changes to `.dev_flow/rules/`.

### Examples

```
/dev-flow rule Не використовувати префікс m для полів
/dev-flow rule Use ReadonlyConfig<T> instead of MutableConfig<T> when only get() needed
/dev-flow rule Для нового коду використовувати Enum замість списку констант
/dev-flow rule Видалити правило SomeObsoleteRule
/dev-flow rule Змінити severity для NoMPrefixForFields на must
```

## Procedure

1. **Read** `.dev_flow/rules/_index.yaml` to understand existing rules — by category block when the index is above `full_read_limit`; a rule body only on a trigger — the unit you edit, a suspected violation, an example needed, a directive that does not decide the case (see [Knowledge Scaling](../references/knowledge-scaling.md)).

2. **Interpret the request:**
   - Adding a new rule — no matching rule exists.
   - Editing an existing rule — a matching rule is found by name or by semantics.
   - Removing a rule — request explicitly says "remove", "delete", "видалити".
   - Listing/showing rules — request says "list", "show", "покажи": read-only — print the matching rules from `_index.yaml` and the category files, write nothing (skip steps 4-7).

2a. **Owner.** A lesson from an incident is a rule. Route to the [skill phase](skill.md) only knowledge that is not an artifact constraint, or a consolidation of an accumulated rule cluster ([Knowledge Scaling → Consolidation](../references/knowledge-scaling.md#consolidation-rules--skill)). In doubt — a rule.

3. **Determine rule properties:**
   - **Category:** naming | structure | architecture | error-handling | style — or an additional category (concurrency / performance / security / testing / ui / data; see [Rule Categories](#rule-categories) below)
   - **Severity:** must | should | prefer — closed set; normalize any other word (see [Severity Normalization](#severity-normalization)); never auto-write a `must` (an independent clean-context review confirms it first)
   - **Rule id:** PascalCase, concise (e.g., `NoMPrefixForFields`, `EnumOverConstants`), or the project's `PREFIX-NNN` scheme when one exists; unique in the catalogue, immutable once written
   - **Directive:** one imperative logical line, within `directive_max`; over the limit → split into two rules (never truncate)
   - **Selector:** inherit the category's `applies_to` (`paths` globs / `phases`) or narrow it; a unit selector never widens its category
   - If ambiguous, ask the user (max 2 questions).

4. **Find code examples** — search the codebase for 1-2 real examples matching the rule.

4a. **Index format:** `.dev_flow/rules/_index.yaml` is YAML (not Markdown). If the file is still in Markdown from an older onboard, convert it on first edit. Move the "Key Patterns Quick Reference" section to `.dev_flow/rules/quick-reference.md`.

5. **Write the rule** to `.dev_flow/rules/{category}.md` using the standard template — the heading is the single source of the directive and severity:

```markdown
## {RuleId} — {directive, one imperative line} ({must | should | prefer})  {#{RuleId}}

**Applies to:** {scope, one line}

### Why
{Within `rationale_max` logical lines.}

### Examples

**Correct:**
\```
// {file path}
{code}
\```

**Incorrect:**
\```
{code}
\```

**Provenance:** {task file / ticket / commit / `{category}.log.md` entry}
```

   Whole body within `unit_body_max`; the investigation narrative stays in the task file or goes to `{category}.log.md`, never into the body. A legacy `## Rule: {Name}` block is rewritten into this form at its first substantive edit (`id` = `{Name}`).

6. **Update the derived layers in the same edit** — the `## Contents` digest line of the category file (`- [{RuleId}](#{RuleId}) ({severity}) — {directive}`; create the digest when the file holds more than `digest_min` rules) and the entry in `categories[].rules` of `_index.yaml` (see [Index Format](#index-format)).

7. **Confirm** — report what was done: rule id, category, severity, action taken.

## Index Format

`.dev_flow/rules/_index.yaml` is grouped by category and is a **derived router** — regenerated from the category files, never the source of truth. Each category entry carries the backing `file`, a one-line `summary` of the category's scope (within `directive_max`, never an accumulating list of directives), an optional `applies_to` selector (`paths` globs / `phases` from the closed set) that is the default for its rules, and an optional `log`; each rule under it carries `name` (= the rule id), `severity`, `summary` (= the heading directive), an optional narrowing `applies_to`, an **optional** `evidence` record (a missing key reads as `unobserved`; a state other than `unobserved` requires `source` + `ref` — see [Evidence Discipline](../references/evidence-discipline.md)), or — for a moved or folded-away rule — `moved_to` in place of `summary` (an alias: keeps the id resolvable, never activated). See [Knowledge Scaling → Index fields](../references/knowledge-scaling.md#index-fields).

```yaml
categories:
  - category: naming
    file: naming.md
    summary: Names of classes, methods, variables, constants, packages
    applies_to: { paths: ["src/**"], phases: [implement, review] }
    rules:
      - name: NoMPrefixForFields
        severity: must
        summary: Do not prefix instance fields with `m`
        evidence:
          state: exercised          # present | wired | exercised | outcome-supported | unobserved
          source: tripwire
          observed_at: 2026-07-30
          ref: .dev_flow/tasks/task_C_AUTH.md
      - name: EnumOverConstants
        severity: prefer
        summary: Use an enum instead of a list of integer constants
  - category: error-handling
    file: error-handling.md
    summary: Exceptions, logging, null safety, lifecycle
    rules:
      - name: WrapCheckedExceptions
        severity: should
        summary: Wrap checked exceptions in a domain exception at the boundary
      - name: RetryWithBackoff
        moved_to: skills/api/retry-procedure.md   # folded into a consolidated skill; id kept as an alias
```

## Severity Normalization

Closed set: `must` | `should` | `prefer`. Any other word is mapped at write time and by audit, each rename reported: `recommended` / "рекомендується" → `should`; `advisory` / `optional` / "бажано" / "краще" → `prefer`; `mandatory` / `required` / `never` / "заборона" → `must`; no mapping → `should` with an `unobserved` note. Normalization never raises an ambiguous value to `must`; an existing `must` stays `must`.

## Executing Confirmed Audit Proposals

The [audit](audit.md) `rules` scope proposes; this phase executes — **only with the developer's confirmation** (`NOT_CONFIRMED` otherwise), and never changing a rule id (`ID_MUTATION` refusal):

- **Consolidation is not a proposal** — a `skill-candidate` cluster is the agent's call, executed through the [skill phase](skill.md); folding a rule into the skill body (removing it, `moved_to` alias) is a deletion and passes an independent clean-context review first.
- **Compact** (`unit-verbosity`): keep heading · Applies to · Why (within `rationale_max`) · Examples · Provenance; move the rest verbatim to `{category}.log.md` under `## {id} — {date}`; link it from Provenance. Directive, examples, and provenance never change (`CONTENT_LOSS` refusal).
- **Split** (`split-planned` / `split-required`): move rules verbatim into a new category file with its own selector; regenerate digests and index blocks; fix the path part of every `<file>#<id>` reference; the set of anchors before = after. No umbrella — the index is the router.
- **Overload remedies** (`knowledge-overload`): narrow selectors · split the category by selector · consolidate the cluster into a skill · merge duplicates · lower to `prefer` — never a `must`.

## Rule Categories

Standard categories (created during onboard):

| Category | File | Scope |
|----------|------|-------|
| naming | `naming.md` | Names of classes, methods, variables, constants, packages |
| structure | `structure.md` | Code organization, class layout, module structure |
| architecture | `architecture.md` | Layer boundaries, dependency direction, communication |
| error-handling | `error-handling.md` | Exceptions, logging, null safety, lifecycle |
| style | `style.md` | Formatting, utility usage, code idioms, documentation |

Additional categories (created on demand when rules don't fit standard ones):

| Category | File | Scope |
|----------|------|-------|
| concurrency | `concurrency.md` | Threading, executors, synchronization, race conditions |
| performance | `performance.md` | Memory, allocations, lazy init, caching, batch operations |
| security | `security.md` | Permissions, data exposure, encryption, token handling |
| testing | `testing.md` | Test patterns, mocking, assertions, test naming |
| ui | `ui.md` | View patterns, layouts, animations, accessibility |
| data | `data.md` | DB operations, ORM, queries, migrations |

New category files are created automatically when the first rule for that category is added.

## Severity Levels

| Severity | Meaning | Review action |
|----------|---------|---------------|
| `must` | Mandatory | Blocks review |
| `should` | Recommended | Warning, acceptable if justified |
| `prefer` | Advisory | Informational |

## Initialization

If `.dev_flow/rules/` does not exist, create the directory with empty category files and `_index.yaml` before proceeding.

## Rules Are Living Documents

Rules apply to **new code only** — no retroactive refactoring required. Updated during: onboard, implement, review, propagate, or by user request. Reading a rules catalogue at a gate follows the [Knowledge Scaling](../references/knowledge-scaling.md) activation protocol: category blocks by selector → digests → the relevant set of directives; a body only on a trigger.
