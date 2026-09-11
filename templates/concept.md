# {Concept Name}  {#C_XXX}

> **Code:** C_XXX
> **Status:** draft
> **Created:** YYYY-MM-DD
> **Updated:** YYYY-MM-DD
> **Author:** {author}
> **Owner:** {role / team / module responsible for long-term maintenance}
> **Complexity:** {low | medium | high — estimated maintenance burden}
> **Criticality:** {peripheral | supporting | core | critical — weight of this module in the whole application; optional, omit when unknown}
>
> **Depends on:** {list of [C_YYY](./path) references, or "none"}
> **Used by:** {list of [C_ZZZ](./path) references, or "—"}
> **Spike:** {[name.spike.md](./name.spike.md) or "—"}
> **Specification:** [SP_XXX](./name.sp.md)
> **Plan:** [name.plan.md](./name.plan.md)
>
> {Lead summary, within `lead_lines` (see references/docs-scaling.md): what this concept covers, who reads it in which situation, what parts it consists of. The header alone must suffice to decide "read on or not".}

## Contents

<!-- Substantive h2 sections only (no h3, no service sections); one item per section, in order;
     annotation states what the section defines, not a restatement of the title.
     Update the matching item in the same edit that adds/renames/removes an h2.
     See references/docs-scaling.md. -->

- [1. Philosophy](#C_XXX_01) — {what problem this solves and which constraints bound it}
- [2. Domain Model](#C_XXX_02) — {which entities and data flows it defines}
- [3. Mechanisms](#C_XXX_03) — {how the solution works}
- [4. Integration Points](#C_XXX_04) — {what it depends on and exposes}
- [5. Design Decisions](#C_XXX_DEC) — {which forks were resolved or stay open}

## 1. Philosophy  {#C_XXX_01}

### 1.1. Core Principle  {#C_XXX_01_01}

{Why this feature exists, what problem it solves.}

### 1.2. Design Constraints  {#C_XXX_01_02}

{Architectural boundaries and invariants.}

## 2. Domain Model  {#C_XXX_02}

### 2.1. Key Entities  {#C_XXX_02_01}

{Entities, responsibilities, and relationships. Use diagrams.}

### 2.2. Data Flows  {#C_XXX_02_02}

{How data moves through the system. Sequence diagrams or flow descriptions.}

## 3. Mechanisms  {#C_XXX_03}

### 3.1. Core Algorithm  {#C_XXX_03_01}

> **Criticality:** {optional — weight of this mechanism inside the module; omit to inherit the header}

{High-level logic in prose or abstract pseudocode. NOT code.}

### 3.2. Edge Cases  {#C_XXX_03_02}

{Error scenarios, boundary conditions, degradation strategies.}

## 4. Integration Points  {#C_XXX_04}

### 4.1. Dependencies  {#C_XXX_04_01}

{What other concepts this depends on (with [C_XXX] references).}

### 4.2. API Surface  {#C_XXX_04_02}

{What this concept exposes to the rest of the system (abstract contracts, not signatures).}

## 5. Design Decisions  {#C_XXX_DEC}

<!-- One record per material fork surfaced via Interview Mode. Delete this section
     if authoring surfaced no decision points. See references/interview-mode.md. -->

### DEC_01 — {short question}  {#C_XXX_DEC_01}

> **Status:** {proposed | resolved | resolved (delegated) | open}
> **Date:** YYYY-MM-DD

**Question:** {the fork, in one sentence}

**Options considered:**
| Option | Consequence |
|--------|-------------|
| A — {approach} | {what it commits us to} |
| B — {approach} | {what it commits us to} |

**Decision:** {chosen option, or "OPEN — see resolution trigger"}
**Rationale:** {why this option / what trade-off it optimises for}
**Rejected because:** {one line per rejected option}
**Resolution trigger:** {open decisions only: the event/date by which this must close}

## Changelog

<!-- Optional when vcs is the primary history medium — see references/docs-scaling.md (History policy).
     Entry classes only: incident / ambiguous-decision / structural-event; an event outside these
     classes is not written (TRIVIAL_ENTRY refusal — progress lives in plan/task checklists).
     One entry = one logical line. -->

| Date | Change |
|------|--------|
| YYYY-MM-DD | Initial version |
