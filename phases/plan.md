# Phase 3: Implementation Plan Authoring

## Purpose

Break the specification into actionable phases with concrete technology choices, file paths, and task tracking. This is where language, framework, and library decisions are made.

## Key Rules

- **Concrete technology choices:** This is the only document where you specify languages, frameworks, libraries, file paths, and module names.
- **Always track progress:** Every phase must have a status: `[DONE]`, `[IN PROGRESS]`, `[TODO]`, or `[BACKLOG]`.
- **Top-level progress summary:** The plan must have a checkbox summary showing overall progress at a glance.
- **Backlog section:** Out-of-scope items go to the backlog at the bottom. Each item names the **trigger** that returns it to scope (event or date) or an owner who decides — an untriggered backlog item is a hidden "later" ([audit](audit.md) flags them).
- **Declare Required Knowledge:** list the rules/skills this plan's work must apply, plus any skill *use / create / update* as tracked work items. This is the source of truth for the per-burst [Knowledge Activation](../references/application-enforcement.md) — re-surfaced beside each implementation burst, not just loaded once.
- **No orphaned phases:** Every phase must reference which spec sections it implements.
- **Pseudocode, not production code:** Include short sketches to clarify intent, but do not write full implementation.

## Context Loading

Loading project knowledge is a **gate** (see [Project Knowledge Is Binding](../SKILL.md#project-knowledge-is-binding)):

**Skill check (gate).** MUST read `.dev_flow/skills/_index.yaml` and load skills matching the plan's scope — they inform technology decisions and record known pitfalls. See [skill phase](skill.md).

**Rule check (gate).** When `.dev_flow/rules/` exists, MUST read `.dev_flow/rules/_index.yaml` and load rules for the planned area; the plan MUST NOT propose anything violating a `must` rule. Reference applicable rules in the Technology Decisions table. See [rule phase](rule.md).

## Interview Mode for Technology Decisions

The plan is where technology is chosen — language, framework, library, persistence, concurrency model — and these are precisely forks with two or more viable, hard-to- reverse options. The Technology Decisions table records *what* was chosen; when a choice is genuinely contested (e.g. two libraries that both fit, a build-vs-buy call, a sync strategy), do not pick silently. Run an interview: present 2–4 options with your **recommended answer**, reach consensus with the developer, and capture the *why* in **Design Decisions** so the Technology Decisions row has a traceable rationale behind it.

A rule, an existing concept, or a documented convention that already settles a choice is **not** a decision point — follow it and cite it. Interview only the genuinely open technology forks. See **[Interview Mode](../references/interview-mode.md)**.

**Forecast check (advisory).** Before fixing phase order or a structural choice, forecast at *plan altitude* — which phase unblocks which, and the cost a later structural change would force — and route each anticipation through the YAGNI-gate (`build now` / `seam+flag` / `drop+record`); a deferred phase goes to the backlog with a return trigger, never dropped silently. See [Consequence Forecasting](../references/consequence-forecasting.md).

## Structure

```markdown
# Implementation Plan: Feature Name  {#PL_XXX}

> **Code:** PL_XXX
> **Status:** draft | in-progress | completed | deprecated
> **Created:** YYYY-MM-DD
> **Updated:** YYYY-MM-DD
>
> **Concept:** [C_XXX](./concept_file.md)
> **Specification:** [SP_XXX](./spec_file.sp.md)
> **Depends on:** [PL_YYY](./other.plan.md)
> **Used by:** [PL_ZZZ](./another.plan.md)
>
> Brief description of the implementation goal.

## Goal

Brief statement of what will be achieved when this plan is complete.

## Technology Decisions

| Decision | Choice | Rationale |
|----------|--------|-----------|
| Language | Python 3.12 | Project standard |
| Serialization | dataclass + JSON | Lightweight, native |

## Required Knowledge

Rules/skills this plan's work must apply — the source of truth for the per-burst [Knowledge Activation](../references/application-enforcement.md). Skill *use / create / update* are tracked items, not left implicit. Omit if nothing applies.

| Kind | Ref | Applies to | Note |
|------|-----|-----------|------|
| rule | NamingConvention | Phase 2 | parser identifiers |
| skill (apply) | RateLimitingPatterns | Phase 3 | current — outranks the prior |
| skill (create) | — | Phase 3 | capture the retry-backoff procedure if it passes the non-triviality filter |

## Progress

- [x] Phase 1 — Data models
- [ ] Phase 2 — Parser (current)
- [ ] Phase 3 — Integration
- [backlog] Phase 4 — Admin UI

## Phases

### Phase 1 — Data Models (`engine/model.py`) [DONE]

**Depends on:** none
**Implements:** [SP_XXX_01](./spec.sp.md#SP_XXX_01)

What to create:
| Entity | Module | Purpose |
|--------|--------|---------|
| Model | engine/model.py | Core data model |

Notes:
- All fields from SP_XXX_01_01 table must be represented

### Phase 2 — Parser (`engine/parser.py`) [IN PROGRESS]

**Depends on:** Phase 1
**Implements:** [SP_XXX_02](./spec.sp.md#SP_XXX_02)

What to implement:
- Parse input format
- Validate against spec constraints

Pseudocode sketch:
    ON parse(input):
        validate(input)
        model = build_model(input)
        RETURN model

### Phase 3 — Integration [TODO]

**Depends on:** Phases 1, 2
**Implements:** [SP_XXX_03](./spec.sp.md#SP_XXX_03)

...

## Backlog

Items deferred from the current implementation cycle (each with a return trigger):
- Admin UI for management — return when: first admin-facing feature is scheduled
- Dynamic configuration — return when: second consumer of the config appears

## Design Decisions  {#PL_XXX_DEC}

ADR-style record of every contested technology fork surfaced via Interview Mode.
Omit if no technology choice was contested. One record per decision:

### DEC_01 — {short question}  {#PL_XXX_DEC_01}

> **Status:** resolved | open
> **Date:** YYYY-MM-DD

**Question:** {the fork, in one sentence}

**Options considered:**
| Option | Consequence |
|--------|-------------|
| A — {choice} | {what it commits us to} |
| B — {choice} | {what it commits us to} |

**Decision:** {chosen option, or "OPEN — see resolution trigger"}
**Rationale:** {why this option / what trade-off it optimises for}
**Rejected because:** {one line per rejected option}
**Resolution trigger:** {open decisions only: the event/date by which this must close}
```

## Changelog Requirement

Every plan must include a Changelog table at the bottom. Record significant changes: technology decision changes, added/removed phases, scope changes. Minor status updates (TODO → DONE) do not require a changelog entry.

## Refactoring Protocol

When a code change restructures modules, files, or internal organization **without changing the specification contracts**, use a lightweight plan-only workflow:

### When This Applies

- Renaming or moving modules/files
- Extracting helper functions or classes (no new public API)
- Splitting a large module into smaller ones (same contracts)
- Merging closely related modules
- Internal dependency reorganization

### Refactoring Workflow

```
1. Verify: spec contracts remain unchanged (inputs, outputs, errors — all identical)
2. Update the plan:
   - Add a Refactor phase with [TODO] status
   - List file moves, renames, and structural changes
   - Reference which existing plan phases are affected
3. Implement the refactoring
4. Run tests — all existing tests must pass without modification
5. Update the plan: mark Refactor phase [DONE], update file paths in other phases
6. Skip Concept and Specification updates — contracts did not change
7. Ask user for commit approval
```

### Refactor Phase Format in Plan

```markdown
### Phase N — Refactor: {description} [TODO]

**Type:** refactor (no spec changes)
**Depends on:** {phases whose file paths will change}
**Affects phases:** {list of phases that reference moved/renamed files}

Changes:
| From | To | Reason |
|------|----|--------|
| engine/utils.py | engine/utils/strings.py | SRP — separate string helpers |
| engine/utils.py | engine/utils/paths.py | SRP — separate path helpers |

Post-refactor updates:
- Update Phase 1 file path: engine/utils.py → engine/utils/strings.py
- Update Phase 3 imports
```

### When NOT to Use Refactor Protocol

- If you add a new public function or endpoint → this is a spec change, use full pipeline.
- If error handling changes → spec contract change, update spec first.
- If you rename a public API → breaking change, requires spec versioning.

## Staleness Detection

- If the `Updated` date is more than 3 months old and related code changed — the plan is stale.
- If a phase has `[IN PROGRESS]` status older than 2 months — reassess whether it's abandoned.
