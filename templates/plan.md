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

{Brief statement of what will be achieved when this plan is complete.}

## Technology Decisions

| Decision | Choice | Rationale |
|----------|--------|-----------|
| {decision} | {choice} | {why} |

## Progress

- [ ] Phase 1 — {name}
- [ ] Phase 2 — {name}
- [backlog] Phase N — {name}

## Phases

### Phase 1 — {Name} (`{path/to/file}`) [TODO]

**Depends on:** none
**Implements:** [SP_XXX_01](./spec.sp.md#SP_XXX_01)

What to create:
| Entity | Module | Purpose |
|--------|--------|---------|
| {entity} | {module} | {purpose} |

Notes:
- {implementation notes}

### Phase 2 — {Name} [TODO]

**Depends on:** Phase 1
**Implements:** [SP_XXX_02](./spec.sp.md#SP_XXX_02)

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

| Date | Change |
|------|--------|
| YYYY-MM-DD | Initial version |
