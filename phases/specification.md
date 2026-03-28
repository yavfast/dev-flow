# Phase 2: Specification Authoring

## Purpose

Define data structures, contracts, validation rules, and processing logic.
A specification answers "what exactly", not "how to implement in language X".

## Language Independence

Specifications MUST NOT reference specific programming languages, frameworks, or libraries.
Use abstract type notation and pseudocode. Implementation language is chosen in the plan.

## Key Rules

- **No final code:** Specs do NOT contain implementation code.
  They may contain abstract pseudocode, data structures, contract signatures, and validation rules.
- **Data structures are detailed:** Describe every field with type, constraints, default, and purpose.
- **Structure over prose:** Avoid vague descriptions. Define concrete fields, methods, and constraints.
- **Isolation and modularity:** Each spec is strictly isolated. Changes in one must not affect others
  unless explicitly defined in contracts.
- **Single Source of Truth:** Once created, source code becomes a derived artifact.
  All changes start in the specification first.

## Self-Validation Checklist

Every specification must pass these checks before advancing to the plan:

- [ ] Do all fields have data types defined?
- [ ] Are error handling responses specified?
- [ ] Is the spec precise enough to prevent hallucinations during code generation?
- [ ] Are all constraints and invariants explicitly stated?
- [ ] Are state transitions documented with conditions and side effects?

## Structure

```markdown
# Feature Name — Specification  {#SP_XXX}

> **Code:** SP_XXX
> **Status:** draft | active | in-progress | completed | deprecated
> **Created:** YYYY-MM-DD
> **Updated:** YYYY-MM-DD
>
> **Concept:** [C_XXX](./concept_file.md)
> **Depends on specs:** [SP_YYY](./other.sp.md)
> **Used by specs:** [SP_ZZZ](./another.sp.md)
> **Plan:** [name.plan.md](./name.plan.md)
>
> Brief description of what this specification defines.

## 01. Data Structures  {#SP_XXX_01}

> Implements: [C_XXX_02](./concept_file.md#C_XXX_02)

### 01_01. EntityName  {#SP_XXX_01_01}

Description of the entity and its purpose.

Fields:
| Field | Type | Required | Default | Constraints | Description |
|-------|------|----------|---------|-------------|-------------|
| id | string | yes | auto-generated | UUID v4 | Unique identifier |
| name | string | yes | — | 1..100 chars | Human-readable name |
| status | enum | yes | "draft" | draft, active, archived | Current state |

Invariants:
- `id` is immutable after creation
- `status` transitions: draft -> active -> archived (no reverse)

## 02. Contracts  {#SP_XXX_02}

### 02_01. OperationName  {#SP_XXX_02_01}

Purpose: What this operation does.

Input:
| Parameter | Type | Required | Constraints |
|-----------|------|----------|-------------|
| entity_id | string | yes | must exist |

Output:
| Field | Type | Description |
|-------|------|-------------|
| result | EntityName | Updated entity |

Errors:
| Code | Condition | Guidance |
|------|-----------|----------|
| NOT_FOUND | entity_id doesn't exist | Check ID |
| FORBIDDEN | caller lacks permission | Request access |

Processing logic (pseudocode):
    FUNCTION OperationName(entity_id, options):
        entity = LOOKUP(entity_id)
        IF entity IS NULL: RAISE NOT_FOUND
        APPLY options TO entity
        PERSIST(entity)
        RETURN entity

## 03. Validation Rules  {#SP_XXX_03}

### 03_01. Input Validation  {#SP_XXX_03_01}

- All string inputs are trimmed and checked for max length
- Enum fields reject unknown values with descriptive error
- Nested structures are validated recursively

## 04. State Transitions  {#SP_XXX_04}

### 04_01. Lifecycle  {#SP_XXX_04_01}

State diagram:
    [draft] --activate()--> [active] --archive()--> [archived]

Transition rules:
| From | To | Condition | Side effects |
|------|----|-----------|-------------|
| draft | active | all required fields set | notify subscribers |
| active | archived | no dependent entities | cascade cleanup |
```

## Changelog Requirement

Every specification must include a Changelog table at the bottom. Record significant changes:
added/removed fields, changed contracts, modified validation rules, changed state transitions.
This is critical for tracing when and why a contract was modified.

## Explicit Cross-References

If a specification uses entities or contracts from another specification:

1. **In metadata:** list in `Depends on specs` field.
2. **In body:** reference specific sections:
   ```
   > Uses [SP_ENT_01_03](./entity.sp.md#SP_ENT_01_03) AgentState structure.
   ```
3. **Reverse references:** the referenced document lists this in `Used by specs`.
