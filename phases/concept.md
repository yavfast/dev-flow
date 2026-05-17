# Phase 1: Concept Authoring

## Purpose

A concept describes the idea, architecture, mechanisms, and philosophy.
It answers "what" and "why", not "how to implement".

## Language Independence

Concepts MUST NOT reference specific programming languages, frameworks, or libraries.
They describe domain logic, architecture patterns, and data flows at an abstract level.

## Pre-Concept Checklist

Before creating a new concept, answer these questions. If any answer is unclear — resolve it first.

- [ ] **What exactly changes?** Define the scope: what the system will do differently.
- [ ] **Why is this needed?** State the motivation — not "because we can", but the problem being solved.
- [ ] **What already exists?** Search existing concepts and codebase for overlapping solutions (see Reuse Check below).
- [ ] **What is the rollback strategy?** How to undo this if it fails or is deprecated. Record in "1.2 Design Constraints".
- [ ] **Who will maintain this?** Which role, team, or module is responsible long-term.

## Reuse Check

**Mandatory before creating a new concept.** Prevents duplication and "reinventing the wheel".

1. Search existing concepts (`docs/*.concept.md`) for overlapping domain entities or mechanisms.
2. Search the codebase for existing implementations that partially solve the problem.
3. If overlap found — justify why a new concept is needed instead of extending the existing one.
4. Document findings in section "1.2 Design Constraints" or as a spike.

If reuse check reveals that the problem is already covered — extend the existing concept instead.

## Authoring Principles

- **Ideas over implementation:** Describe "what" and "why", never "how in language X".
  If you find yourself writing class names or imports — you've gone too far.
- **Domain language:** Use problem domain language, not solution domain.
  Say "permission check" not "decorator that wraps the function".
- **Diagrams over prose:** Use ASCII diagrams, mermaid, or structured tables
  to show relationships, hierarchies, and flows.
- **Explicit boundaries:** State what this concept IS and IS NOT.
- **Cross-references:** Link to related concepts with `[C_XXX]` identifiers.
- **Changelog:** Every concept ends with a `## Changelog` table recording significant changes.

## Banned Phrases

Reject or flag the following phrases in any concept. They signal deferred decisions
that become permanent technical debt:

| Phrase | Problem | What to write instead |
|--------|---------|----------------------|
| "temporarily" / "тимчасово" | Temporary solutions become permanent | Define when and how it ends — or design the full solution |
| "at first" / "на першому етапі" | Implies an unwritten "second stage" that never comes | Describe the complete solution; if phased — define all phases in the plan |
| "will refactor later" / "потім переробимо" | Deferred cleanup rarely happens | Either do it now or create a separate concept for the refactoring |
| "for now" / "поки що" | Same as "temporarily" | State the permanent design decision |
| "simple wrapper around X" | Wrappers grow into unmaintainable layers | Describe the actual abstraction and its boundaries |

## Structure

```markdown
# Concept Name  {#C_XXX}

> **Code:** C_XXX
> **Status:** active | draft | deprecated
> **Created:** YYYY-MM-DD
> **Updated:** YYYY-MM-DD
> **Author:** agent / human name
> **Owner:** role / team / module responsible for long-term maintenance
> **Complexity:** low | medium | high (estimated maintenance burden)
>
> **Depends on:** [C_YYY](./other.concept.md)
> **Used by:** [C_ZZZ](./another.concept.md)
> **Spike:** {[name.spike.md](./name.spike.md) or "—"}
> **Specification:** [SP_XXX](./name.sp.md)
> **Plan:** [name.plan.md](./name.plan.md)
>
> Brief one-paragraph description.

## 1. Philosophy  {#C_XXX_01}

### 1.1. Core Principle  {#C_XXX_01_01}
Why this feature exists, what problem it solves.

### 1.2. Design Constraints  {#C_XXX_01_02}
Architectural boundaries and invariants.

## 2. Domain Model  {#C_XXX_02}

### 2.1. Key Entities  {#C_XXX_02_01}
Entities, responsibilities, and relationships.
Use diagrams to show relationships.

### 2.2. Data Flows  {#C_XXX_02_02}
How data moves through the system.

## 3. Mechanisms  {#C_XXX_03}

### 3.1. Core Algorithm  {#C_XXX_03_01}
High-level logic in prose or abstract pseudocode. NOT code.

### 3.2. Edge Cases  {#C_XXX_03_02}
Error scenarios, boundary conditions, degradation strategies.

## 4. Integration Points  {#C_XXX_04}

### 4.1. Dependencies  {#C_XXX_04_01}
What other concepts this depends on (with [C_XXX] references).

### 4.2. API Surface  {#C_XXX_04_02}
What this concept exposes (abstract contracts, not signatures).

## Changelog

| Date | Change |
|------|--------|
| YYYY-MM-DD | Initial version |
```

## Spike / Exploration (Optional)

Before writing a concept, you may need to explore an idea: research libraries,
prototype approaches, or discuss trade-offs. This is formalized as a **spike** —
a lightweight, time-boxed exploration that feeds into the concept.

### When to Spike

- The domain is unfamiliar — need to understand constraints before committing to an architecture.
- Multiple competing approaches exist — need to compare before choosing.
- A critical technical question blocks concept authoring (e.g., "can library X handle Y?").
- The user explicitly asks for exploration before design.

### Spike Workflow

```
1. Create a spike file: docs/{name}.spike.md
2. Define the question(s) to answer and a time/scope boundary
3. Research, prototype, discuss — log findings as entries
4. Conclude with a verdict and status
5. Reference the spike from the concept: Spike: [name.spike.md](./name.spike.md)
```

### Spike Rules

- A spike is **throwaway research**, not a deliverable. It does not go through validation gates.
- Spike conclusions feed into the concept — they do not replace it.
- A spike that produces no clear verdict should be marked `inconclusive` with reasons.
- Prototypes created during a spike should be deleted or moved to a scratch directory — they must NOT become production code.

See [spike template](../templates/spike.md) for the file structure.

## Concept Granularity

### When to Split

A concept should be split when:
- It describes more than one independent responsibility (SRP for documents).
- Different parts change for different reasons or at different rates.
- The file exceeds ~300 lines.

**Example:** "Access Control" might split into:
- `C_AUTHN` — Authentication (identity verification)
- `C_AUTHZ` — Authorization (permission checks)
- `C_AUDIT` — Audit trail (logging access decisions)

### When to Keep Together

Keep unified when:
- Parts are tightly coupled — changing one always requires changing the other.
- Splitting would create excessive cross-references.
- The concept is small (<100 lines) and focused.

### Rule of Thumb

Ask: "Can someone work on part A without understanding part B?" If yes — split.

## Epic / Feature Group

When a large feature requires 3+ related concepts with ordering dependencies,
create an epic document (`*.epic.md`) to coordinate them.
See [epic template](../templates/epic.md) for the structure.
