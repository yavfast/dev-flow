# Phase 1: Concept Authoring

## Purpose

A concept describes the idea, architecture, mechanisms, and philosophy. It answers "what" and "why", not "how to implement".

## Language Independence

Concepts MUST NOT reference specific programming languages, frameworks, or libraries. They describe domain logic, architecture patterns, and data flows at an abstract level.

## Pre-Concept Checklist

Before creating a new concept, answer these questions. If any answer is unclear — resolve it first.

- [ ] **What exactly changes?** Define the scope: what the system will do differently.
- [ ] **Why is this needed?** State the motivation — not "because we can", but the problem being solved.
- [ ] **What already exists?** Search existing concepts and codebase for overlapping solutions (see Reuse Check below).
- [ ] **What is the rollback strategy?** How to undo this if it fails or is deprecated. Record in "1.2 Design Constraints".
- [ ] **Who will maintain this?** Which role, team, or module is responsible long-term.

**Answers must rest on verified knowledge, not guesses.** If a critical answer depends on something nobody has confirmed — an unfamiliar domain, an unverified capability, an unexplored solution space — run **[research](research.md)** (`/dev-flow research`) before authoring. Proceeding on an unverified assumption is a user decision: record it as an **open** Design Decision with a resolution trigger, never as silent confidence. The Concept → Spec gate checks this.

## Reuse Check

**Mandatory before creating a new concept.** Prevents duplication and "reinventing the wheel".

1. Search existing concepts (`docs/*.concept.md`) for overlapping domain entities or mechanisms.
2. Search the codebase for existing implementations that partially solve the problem.
3. If overlap found — justify why a new concept is needed instead of extending the existing one.
4. Document findings in section "1.2 Design Constraints" or as a spike.

If reuse check reveals that the problem is already covered — extend the existing concept instead.

## Context Loading

Loading project knowledge is a **gate** (see [Project Knowledge Is Binding](../SKILL.md#project-knowledge-is-binding)):

**Skill check (gate).** MUST read `.dev_flow/skills/_index.yaml` and load skills for the concept's domain — known constraints and pitfalls bound what the architecture can promise. See [skill phase](skill.md).

**Rule check (gate).** When `.dev_flow/rules/` exists, MUST read `.dev_flow/rules/_index.yaml` and load architecture rules for the area; the concept MUST NOT propose a mechanism that violates a `must` rule. See [rule phase](rule.md).

## Project Glossary

Load `docs/_glossary.md` together with `docs/_index.md` before authoring (it is the project's canonical domain vocabulary). While writing:

- **Reuse canonical terms.** For every domain noun, use the glossary's term — not an alias listed under `_Avoid_`. This is what keeps the same referent named the same way across all concepts.
- **Add new terms inline.** When the concept introduces a genuinely new domain term, add it to `docs/_glossary.md` as you go (term + tight definition + aliases to avoid). Create the file lazily if it does not exist yet.
- **Challenge against the glossary.** If your wording conflicts with the glossary, use the canonical term — a routine naming choice the glossary settles is not an interview. Escalate to [Interview Mode](../references/interview-mode.md) only when the conflict is *material* (two genuinely different concepts being conflated, or a canonical choice that shapes contracts), then update the glossary once resolved.

Boundary: the glossary says *what a word means*; this concept's Domain Model (§2) says *how the entities relate and behave*. See [Glossary](../references/glossary.md).

## Authoring Principles

- **Ideas over implementation:** Describe "what" and "why", never "how in language X". If you find yourself writing class names or imports — you've gone too far.
- **Domain language:** Use problem domain language, not solution domain. Say "permission check" not "decorator that wraps the function".
- **Diagrams over prose:** Use ASCII diagrams, mermaid, or structured tables to show relationships, hierarchies, and flows.
- **Explicit boundaries:** State what this concept IS and IS NOT.
- **Cross-references:** Link to related concepts with `[C_XXX]` identifiers.
- **Changelog:** Every concept ends with a `## Changelog` table recording significant changes.

## Interview Mode for Design Decisions

Concepts are where the biggest, least-reversible choices are made — and where they are easiest to hide. When authoring surfaces **two or more materially different ways forward** (a different domain model, a different boundary, a different mechanism), do **not** pick one silently and bury it in the prose. Stop and run an interview: present the fork to the developer with 2–4 options and your **recommended answer**, reach a consensus, and record the outcome.

The developer holds context you don't (roadmap, business constraints, team) and owns the consequences; an architectural mistake set here is expensive to undo once it reaches code. A cheap question now beats an expensive reversal later.

See **[Interview Mode](../references/interview-mode.md)** for the full procedure — when a fork counts as a decision point (and when it does not), how to frame options with a recommendation, sequential vs batched questions, and the two valid outcomes (consensus vs documented open alternatives with a resolution trigger). Record every resolved or open decision in the concept's **Design Decisions** section.

**Interview vs Banned Phrases.** A documented **open** decision (options + trade-offs + a resolution trigger) is *not* a banned deferral. The phrases below ("temporarily", "for now", …) are *undocumented* deferrals with no owner and no trigger; an open decision records the alternatives and the concrete event/date that closes it — the sanctioned way to leave something open (e.g. for research work).

**Forecast check (advisory).** Before settling a material design fork, forecast its consequences at *concept altitude* — the future intents, consumers, and use-areas it shapes — and route each anticipation through the YAGNI-gate (`build now` / `seam+flag` / `drop+record`). At this altitude forecasting is cheap and encouraged, but it yields *direction and seams*, not speculative machinery with no stated consumer; a genuinely uncertain build-vs-defer fork becomes an interview above. See [Consequence Forecasting](../references/consequence-forecasting.md).

## Banned Phrases

Reject or flag the following phrases in any concept. They signal deferred decisions that become permanent technical debt:

| Phrase | Problem | What to write instead |
|--------|---------|----------------------|
| "temporarily" / "тимчасово" | Temporary solutions become permanent | Define when and how it ends — or design the full solution |
| "at first" / "на першому етапі" | Implies an unwritten "second stage" that never comes | Describe the complete solution; if phased — define all phases in the plan |
| "will refactor later" / "потім переробимо" | Deferred cleanup rarely happens | Either do it now or create a separate concept for the refactoring |
| "for now" / "поки що" | Same as "temporarily" | State the permanent design decision |
| "simple wrapper around X" | Wrappers grow into unmaintainable layers | Describe the actual abstraction and its boundaries |
| "TBD" / "to be defined" | A hidden deferral with no owner and no trigger | Resolve it now, run a [research spike](research.md), or record an open decision with a resolution trigger |

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

## 5. Design Decisions  {#C_XXX_DEC}

ADR-style record of every material fork surfaced via Interview Mode. Omit the
section only if authoring surfaced no decision points. One record per decision:

### DEC_01 — {short question}  {#C_XXX_DEC_01}

> **Status:** resolved | open
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
```

## Spike / Exploration (Optional)

Before writing a concept, you may need to explore an idea: research libraries, prototype approaches, or discuss trade-offs. This is formalized as a **spike** — a lightweight, time-boxed exploration that feeds into the concept. Spikes are executed by the **[research phase](research.md)** (`/dev-flow research`, alias `spike`) — it owns the cost gate, the delegation to the researcher subagent, the spike rules, and persisting durable findings to `.dev_flow/skills/`.

### When to Spike

- The domain is unfamiliar — need to understand constraints before committing to an architecture.
- Multiple competing approaches exist — need to compare before choosing (the spike *discovers* options; the [interview](../references/interview-mode.md) then *chooses*).
- A critical technical question blocks concept authoring (e.g., "can library X handle Y?").
- A Pre-Concept Checklist answer rests on an unverified assumption.
- The user explicitly asks for exploration before design.

When any of these hold mid-authoring, pause the concept, run the [research phase](research.md), then resume with the findings. Reference the spike from the concept: `Spike: [name.spike.md](./name.spike.md)`.

See [research phase](research.md) for the full procedure and spike rules, and the [spike template](../templates/spike.md) for the file structure.

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

When a large feature requires 3+ related concepts with ordering dependencies, create an epic document (`*.epic.md`) to coordinate them. See [epic template](../templates/epic.md) for the structure.
