# {Feature Name} — Specification  {#SP_XXX}

> **Code:** SP_XXX
> **Status:** draft
> **Created:** YYYY-MM-DD
> **Updated:** YYYY-MM-DD
>
> **Concept:** [C_XXX](./concept_file.md)
> **Depends on:** {list of [SP_YYY](./path) references, or "none"}
> **Used by:** {list of [SP_ZZZ](./path) references, or "—"}
> **Plan:** [name.plan.md](./name.plan.md)
>
> {Brief description of what this specification defines.}

## 01. Data Structures  {#SP_XXX_01}

> Implements: [C_XXX_02](./concept_file.md#C_XXX_02)

### 01_01. {EntityName}  {#SP_XXX_01_01}

{Description of the entity and its purpose.}

Fields:
| Field | Type | Required | Default | Constraints | Description |
|-------|------|----------|---------|-------------|-------------|
| id | string | yes | auto-generated | UUID v4 | Unique identifier |
| {field} | {type} | {yes/no} | {default} | {constraints} | {description} |

Invariants:
- {invariant 1}
- {invariant 2}

## 02. Contracts  {#SP_XXX_02}

### 02_01. {OperationName}  {#SP_XXX_02_01}

Purpose: {What this operation does.}

Input:
| Parameter | Type | Required | Constraints |
|-----------|------|----------|-------------|
| {param} | {type} | {yes/no} | {constraints} |

Output:
| Field | Type | Description |
|-------|------|-------------|
| {field} | {type} | {description} |

Errors:
| Code | Condition | Guidance |
|------|-----------|----------|
| {ERROR_CODE} | {when it occurs} | {what to do} |

Processing logic (pseudocode):
    FUNCTION {OperationName}({params}):
        {pseudocode steps}

## 03. Validation Rules  {#SP_XXX_03}

### 03_01. Input Validation  {#SP_XXX_03_01}

- {validation rule 1}
- {validation rule 2}

## 04. State Transitions  {#SP_XXX_04}

### 04_01. Lifecycle  {#SP_XXX_04_01}

State diagram:
    [{state1}] --{action}()--> [{state2}]

Transition rules:
| From | To | Condition | Side effects |
|------|----|-----------|-------------|
| {from} | {to} | {condition} | {side effects} |

## 05. Verification Criteria  {#SP_XXX_05}

### 05_01. Functional Expectations  {#SP_XXX_05_01}

| Contract | Scenario | Input | Expected outcome |
|----------|----------|-------|------------------|
| {OperationName} | Happy path | {valid input} | {expected result} |
| {OperationName} | Error case | {invalid input} | {expected error} |

### 05_02. Invariant Checks  {#SP_XXX_05_02}

| Invariant | Verification method |
|-----------|-------------------|
| {invariant} | {how to verify it holds} |

### 05_03. Integration Scenarios  {#SP_XXX_05_03}

| Scenario | Preconditions | Steps | Expected result |
|----------|--------------|-------|-----------------|
| {scenario} | {setup} | {steps} | {expected} |

### 05_04. Edge Cases and Boundaries  {#SP_XXX_05_04}

| Case | Input | Expected behavior |
|------|-------|-------------------|
| {edge case} | {input} | {behavior} |

## 06. Reversibility  {#SP_XXX_06}

### 06_01. Rollback Strategy  {#SP_XXX_06_01}

{How to undo this feature if it fails or is deprecated.}

| Aspect | Rollback approach |
|--------|-------------------|
| Data/state changes | {Can new data structures be dropped? Migration reversibility} |
| Artifacts | {What files, configs, or resources need cleanup?} |
| Dependent modules | {Which specs reference this one? What breaks if removed?} |
| External contracts | {Published APIs or events that consumers depend on?} |

## 07. Design Decisions  {#SP_XXX_DEC}

<!-- One record per material fork surfaced via Interview Mode. Delete this section
     if spec authoring surfaced no decision points. See references/interview-mode.md. -->

### DEC_01 — {short question}  {#SP_XXX_DEC_01}

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
