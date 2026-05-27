# {Concept Name}  {#C_XXX}

> **Code:** C_XXX
> **Status:** draft
> **Created:** YYYY-MM-DD
> **Updated:** YYYY-MM-DD
> **Author:** {author}
> **Owner:** {role / team / module responsible for long-term maintenance}
> **Complexity:** {low | medium | high — estimated maintenance burden}
>
> **Depends on:** {list of [C_YYY](./path) references, or "none"}
> **Used by:** {list of [C_ZZZ](./path) references, or "—"}
> **Spike:** {[name.spike.md](./name.spike.md) or "—"}
> **Specification:** [SP_XXX](./name.sp.md)
> **Plan:** [name.plan.md](./name.plan.md)
>
> {Brief one-paragraph description of what this concept covers.}

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

> **Status:** {resolved | open}
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

| Date | Change |
|------|--------|
| YYYY-MM-DD | Initial version |
