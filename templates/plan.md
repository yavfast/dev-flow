# Implementation Plan: {Feature Name}  {#PL_XXX}

> **Code:** PL_XXX
> **Status:** draft
> **Created:** YYYY-MM-DD
> **Updated:** YYYY-MM-DD
>
> **Concept:** [C_XXX](./concept_file.md)
> **Specification:** [SP_XXX](./spec_file.sp.md)
> **Depends on:** {list of [PL_YYY](./path) references, or "none"}
> **Used by:** {list of [PL_ZZZ](./path) references, or "—"}
>
> {Brief description of the implementation goal.}

## Goal

{What will be achieved when this plan is complete — restate the task's recorded intent (goal / target state / expected result) for this plan's scope.}

## Technology Decisions

| Decision | Choice | Rationale |
|----------|--------|-----------|
| {decision} | {choice} | {why} |

## Required Knowledge

<!-- Rules/skills this plan's work must apply — the source of truth for the
     per-burst Knowledge Activation (references/application-enforcement.md).
     Skill use / create / update are tracked items. Delete the section only
     if nothing applies. -->

| Kind | Ref | Applies to | Note |
|------|-----|-----------|------|
| rule | {RuleId} | Phase {N} | {what it constrains} |
| skill (apply) | {SkillName} | Phase {N} | {current — outranks the prior / stale — re-ground first} |

## Progress

<!-- A plan has no Contents section — this linked list serves that role: every item for an
     authored phase links to its phase anchor (a [backlog] item has no phase section yet, so
     no link). Update the matching item in the same edit that adds/renames/removes a phase.
     See references/docs-scaling.md. -->

- [ ] [Phase 1 — {name}](#PL_XXX_P1)
- [ ] [Phase 2 — {name}](#PL_XXX_P2)
- [backlog] Phase N — {name}

## Phases

### Phase 1 — {Name} (`{path/to/file}`) [TODO]  {#PL_XXX_P1}

**Depends on:** none
**Implements:** [SP_XXX_01](./spec.sp.md#SP_XXX_01)
**Verify:** {spec Verification Criteria SP_XXX_05_* for this phase's contracts, + any phase-local acceptance check}

What to create:
| Entity | Module | Purpose |
|--------|--------|---------|
| {entity} | {module} | {purpose} |

Notes:
- {implementation notes}

### Phase 2 — {Name} [TODO]  {#PL_XXX_P2}

**Depends on:** Phase 1
**Implements:** [SP_XXX_02](./spec.sp.md#SP_XXX_02)
**Verify:** {spec Verification Criteria SP_XXX_05_* for this phase's contracts, + any phase-local acceptance check}

What to implement:
- {task 1}
- {task 2}

Pseudocode sketch:
    {pseudocode}

## Backlog

Items deferred from the current implementation cycle (each names the trigger that returns it to scope, or the owner who decides):
- {deferred item 1} — return when: {event or date}
- {deferred item 2} — return when: {event or date}

## Design Decisions  {#PL_XXX_DEC}

<!-- One record per contested technology fork surfaced via Interview Mode. Delete
     this section if no technology choice was contested. See references/interview-mode.md. -->

### DEC_01 — {short question}  {#PL_XXX_DEC_01}

> **Status:** {resolved | open}
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

## Changelog

<!-- Optional when vcs is the primary history medium — see references/docs-scaling.md (History policy).
     Entry classes only: incident / ambiguous-decision / structural-event; an event outside these
     classes is not written (TRIVIAL_ENTRY refusal — progress lives in the Progress checklist).
     One entry = one logical line. -->

| Date | Change |
|------|--------|
| YYYY-MM-DD | Initial version |
