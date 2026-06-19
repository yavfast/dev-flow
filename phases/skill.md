# Phase: Skill — Manage Project Knowledge Base

## Purpose

Add, find, update, or remove skills in `.dev_flow/skills/`. Skills are consolidated project-specific knowledge about technologies, APIs, patterns, and tools — acquired through external research, user explanations, or implementation discoveries — that must be preserved across sessions.

Skills differ from rules and from the cache:
- **Rules** (`.dev_flow/rules/`) — how to write code in this project (naming, style, architecture)
- **Skills** (`.dev_flow/skills/`) — what you need to know about a technology or pattern to do a task well
- **Cache** (`.dev_flow/cache/`) — fetched artifacts (design exports, downloaded documents, baselines): skills capture what you *learned*, the cache keeps what you *fetched*. A skill may link cache entries. See [Resource Cache](../references/cache.md)

## Command

```
/dev-flow skill <request>
```

The request is a freeform description in any language. Interpret the intent and apply the appropriate action to `.dev_flow/skills/`.

### Examples

```
/dev-flow skill REST authentication flow with token refresh
/dev-flow skill save caching patterns discovered during image processing work
/dev-flow skill update message-bus-patterns with new routing example
/dev-flow skill remove obsolete-legacy-sdk
```

## When Skills Are Created Automatically

Skills are created or updated automatically — without an explicit command — when:

| Trigger | Action |
|---------|--------|
| User explains a project-specific pattern | Create or update skill |
| A [research](research.md) run (spike) produced durable findings | The research phase's Step 4 persists them here — this is the primary owner of "external research → consolidated skill" |
| Web/context7 research was needed mid-task (outside a formal spike) | Save consolidated results after task completes |
| Non-obvious pattern discovered during `implement` | Add to relevant skill |
| Bug traced to API/SDK misuse | Add "Pitfalls" entry to relevant skill |
| `onboard` reverse-engineers a non-trivial pattern | Create skill for it |

**Always check for an existing skill BEFORE doing external research.** If a skill exists and is current — use it. If outdated — research and update it.

## Non-Triviality Filter

Only save knowledge that:
- Is **project-specific** (how *this* project uses the technology, not just how the technology works)
- Required **external research** OR was discovered through **debugging/implementation**
- Would **not be obvious** to a senior developer arriving fresh to this project
- Has a **non-zero chance of being needed again** in future tasks

Do NOT save:
- General SDK/framework knowledge well-known to senior devs
- Standard language syntax or build tool basics
- Knowledge fully derivable by reading the codebase
- Information already in `onboard/analysis/`

## Procedure

### Finding a Skill

1. **Read** `.dev_flow/skills/_index.yaml` — identify candidate domains by topic match.
2. **Read** `_index.yaml` for each candidate domain — find specific skill files by `topics` field.
3. **Load** only the matched skill files — never load the full tree.
4. **Return** the relevant knowledge or note that no skill exists yet.

### Creating or Updating a Skill

1. **Read root `_index.yaml`** — identify the appropriate domain path.
2. **Read domain `_index.yaml`** — check if a matching skill already exists.
3. **Determine domain path** — if ambiguous, ask (max 2 questions).
4. **Write or update** the skill file using the standard template.
5. **Update** the domain `_index.yaml` and root `_index.yaml`.
6. **Confirm** — report skill name, domain, action taken.

### Removing a Skill

Request must explicitly say "remove", "delete", "видалити", "видали".

1. Read root `_index.yaml` → domain `_index.yaml` → locate the file.
2. Delete the skill file.
3. Remove the entry from domain `_index.yaml` and root `_index.yaml`.
4. Confirm deletion.

## Skill File Template

```markdown
---
skill: {SkillName}
domain: {top/sub/domain}
topics: [keyword1, keyword2, keyword3]
source: {user | research | implementation | onboard}
updated: {YYYY-MM-DD}
---

# {Skill Title}

## Context
{Why this knowledge matters for the project — project-specific constraints.}

## Key Concepts
{Consolidated knowledge: patterns, APIs, configuration, gotchas.}
{Include code snippets only when they are non-obvious project patterns.}

## Usage in This Project
{Concrete examples from the codebase with file paths.}

## Pitfalls
{Non-obvious issues discovered during research or implementation.}

## References
{Only if external sources were used — library docs, API references.}
```

## `_index.yaml` Format (YAML)

Root `_index.yaml` (`.dev_flow/skills/_index.yaml`):

```yaml
updated: {YYYY-MM-DD}

domains:
  - path: api/
    description: REST/GraphQL API patterns, authentication, rate limiting
    topics: [REST, GraphQL, auth, rate-limit, middleware]
  - path: data/
    description: Data access, ORM, caching, migrations
    topics: [ORM, migrations, caching, queries, transactions]
```

Domain-level `_index.yaml` (e.g., `.dev_flow/skills/api/_index.yaml`):

```yaml
domain: api
description: API integration patterns specific to this project
updated: {YYYY-MM-DD}

subdomains:
  - path: auth/
    covers: Token refresh, OAuth flow, session management

skills:
  - file: rate-limiting.md
    skill: RateLimitingPatterns
    topics: [rate-limit, throttle, backoff, retry]
    source: onboard
    updated: {YYYY-MM-DD}
```

## Loading Algorithm

The index chain is the navigation layer. **Never load the full `.dev_flow/skills/` tree.**

```
Step 1: Read root _index.yaml (~20 lines)
Step 2: Match task topic → candidate domains (by domains[].topics)
Step 3: Read _index.yaml for each candidate domain (~30 lines, typically 1-2)
Step 4: Match topic → skill files (by skills[].topics)
Step 5: Load only matched skill files (typically 1-3 files, ~100-200 lines each)
```

If no domain matches at Step 2 — stop. Cost: one small file read.

## Initialization

If `.dev_flow/skills/` does not exist, create the directory structure with empty `_index.yaml` files before proceeding. See [onboard phase](onboard.md) for the full initialization procedure that runs during project onboard.

## Skills Are Living Documents

- Skills apply to the task at hand; there is no retroactive obligation.
- Update when: research reveals new information, implementation finds new pitfalls, library version changes, user corrects existing knowledge.
- If new research contradicts an existing skill — update the skill and add a note in "Pitfalls".
- Skills may reference related rules (e.g., a skill about message bus → rule `MessageBusForCrossLayerCommunication`).
