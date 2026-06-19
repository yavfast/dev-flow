# Phase: Rule — Manage Project Coding Rules

## Purpose

Add, edit, or remove coding rules in `.dev_flow/rules/`. Rules define naming conventions,
code structure patterns, architectural constraints, and style requirements that all new
code must follow. The review phase validates code against these rules.

## Command

```
/dev-flow rule <request>
```

The request is a freeform description in any language. Interpret the intent and apply
the appropriate changes to `.dev_flow/rules/`.

### Examples

```
/dev-flow rule Не використовувати префікс m для полів
/dev-flow rule Use ReadonlyConfig<T> instead of MutableConfig<T> when only get() needed
/dev-flow rule Для нового коду використовувати Enum замість списку констант
/dev-flow rule Видалити правило SomeObsoleteRule
/dev-flow rule Змінити severity для NoMPrefixForFields на must
```

## Procedure

1. **Read** `.dev_flow/rules/_index.yaml` to understand existing rules.

2. **Interpret the request:**
   - Adding a new rule — no matching rule exists.
   - Editing an existing rule — a matching rule is found by name or by semantics.
   - Removing a rule — request explicitly says "remove", "delete", "видалити".

3. **Determine rule properties:**
   - **Category:** naming | structure | architecture | error-handling | style
   - **Severity:** must | should | prefer (infer from wording: "заборона"/"never" → must, "рекомендується"/"should" → should, "бажано"/"prefer"/"краще"/"advisory" → prefer)
   - **Rule name:** PascalCase, concise (e.g., `NoMPrefixForFields`, `EnumOverConstants`)
   - If ambiguous, ask the user (max 2 questions).

4. **Find code examples** — search the codebase for 1-2 real examples matching the rule.

4a. **Index format:** `.dev_flow/rules/_index.yaml` is YAML (not Markdown).
    If the file is still in Markdown from an older onboard, convert it on first edit.
    Move the "Key Patterns Quick Reference" section to `.dev_flow/rules/quick-reference.md`.

5. **Write the rule** to `.dev_flow/rules/{category}.md` using the standard template:

```markdown
---

## Rule: {RuleName}

**Category:** {category}
**Severity:** {must | should | prefer}
**Applies to:** {scope}

### Description
{Clear, concise description.}

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

### Rationale
{Why this rule exists.}
```

6. **Update `_index.yaml`** — add/update the rule entry in `categories[].rules`
   (see [Index Format](#index-format) below for the canonical schema).

7. **Confirm** — report what was done: rule name, category, severity, action taken.

## Index Format

`.dev_flow/rules/_index.yaml` is grouped by category. Each category entry carries the
backing `file` and a `summary`; each rule under it carries `name`, `severity`, and a
one-line `summary`:

```yaml
categories:
  - category: naming
    file: naming.md
    summary: Names of classes, methods, variables, constants, packages
    rules:
      - name: NoMPrefixForFields
        severity: must
        summary: Do not prefix instance fields with `m`
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
```

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

New category files are created automatically when the first rule for that category
is added. Each file follows the same header format as standard categories.

## Severity Levels

| Severity | Meaning | Review action |
|----------|---------|---------------|
| `must` | Mandatory | Blocks review |
| `should` | Recommended | Warning, acceptable if justified |
| `prefer` | Advisory | Informational |

## Initialization

If `.dev_flow/rules/` does not exist, create the directory with empty category files
and `_index.yaml` before proceeding.

## Rules Are Living Documents

Rules apply to **new code only** — no retroactive refactoring required.
Updated during: onboard, implement, review, or by user request.
