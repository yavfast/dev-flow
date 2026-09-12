# Phase 2: Specification Authoring

## Purpose

Define data structures, contracts, validation rules, and processing logic. A specification answers "what exactly", not "how to implement in language X".

## Role Responsible

This phase is handled by **SpecAuthor**: [roles/spec-author.ai.md](../roles/spec-author.ai.md).

## Language Independence

Specifications MUST NOT reference specific programming languages, frameworks, or libraries. Use abstract type notation and pseudocode. Implementation language is chosen in the plan.

## Key Rules

- **No final code:** Specs do NOT contain implementation code. They may contain abstract pseudocode, data structures, contract signatures, and validation rules.
- **Data structures are detailed:** Describe every field with type, constraints, default, and purpose.
- **Structure over prose:** Avoid vague descriptions. Define concrete fields, methods, and constraints.
- **Isolation and modularity:** Each spec is strictly isolated. Changes in one must not affect others unless explicitly defined in contracts.
- **Single Source of Truth:** Once created, source code becomes a derived artifact. All changes start in the specification first.
- **Follow the glossary:** `docs/_glossary.md` is loaded with `docs/_index.md`; name every entity, field, and contract with its canonical term (not an `_Avoid_` alias). If the spec introduces a genuinely new domain term, add it to the glossary. Term *formation* is mainly a concept-phase job — here you consume the canonical vocabulary. See [Glossary](../references/glossary.md).
- **Formatting:** Follow [Documentation Formatting](../references/formatting.md) — no hard line wraps inside sentences or paragraphs; one paragraph = one logical line.
- **Contents + lead summary:** the template's Contents section and the lead summary in the header follow [Docs Scaling](../references/docs-scaling.md) — update the Contents item in the same edit that changes an h2.
- **Criticality is inherited, then refined:** A spec has no application-level declaration of its own — it inherits the one in its concept's header through `Concept:`. Add a `Criticality:` line to an entity (`§01_xx`) or a contract (`§02_xx`) only where that part's weight inside the module differs from the rest. Optional; an omitted line inherits and lowers nothing. Read by [review](review.md) — see [Review Convergence](../references/review-convergence.md).

## Context Loading

Loading project knowledge is a **gate** (see [Project Knowledge Is Binding](../SKILL.md#project-knowledge-is-binding)):

**Skill check (gate).** MUST read `.dev_flow/skills/_index.yaml` and load skills matching the spec's domain — known pitfalls and platform constraints shape contracts and error models. See [skill phase](skill.md).

**Rule check (gate).** When `.dev_flow/rules/` exists, MUST read `.dev_flow/rules/_index.yaml` and load rules for the spec'd area; the spec MUST NOT define contracts that violate a `must` rule (e.g. an error-handling pattern the project forbids). See [rule phase](rule.md).

## Interview Mode for Design Decisions

A fork here — a field's type, an error model, a state transition, a sync strategy — binds consumers once shipped. When authoring surfaces **two or more materially different ways to model or contract something**, do not pick one silently. Stop and run an interview: present the fork with 2–4 options and your **recommended answer**, reach a consensus, and record the outcome.

See **[Interview Mode](../references/interview-mode.md)** for the full procedure. Record every proposed, resolved, or open decision in the spec's **Design Decisions** section.

**Interview vs Banned Phrases.** A documented **open** decision (options + trade-offs + a resolution trigger) is *not* a banned "TBD". The banned phrases below are *undocumented* deferrals with no owner and no trigger. An open decision records the alternatives and the concrete event/date that closes it — that is the sanctioned way to leave something open (e.g. for research work).

**Forecast check (advisory).** Before pinning a data shape or contract, forecast at *spec altitude* — who will consume this contract and what must not break as it evolves — and route each anticipation through the YAGNI-gate (`build now` / `seam+flag` / `drop+record`); prefer a versioning seam over a speculative field with no stated consumer (Minimality). See [Consequence Forecasting](../references/consequence-forecasting.md).

## Banned Phrases

Reject or flag the following phrases in any specification.

| Phrase | Problem | What to write instead |
|--------|---------|----------------------|
| "temporarily" / "тимчасово" | Temporary contracts become permanent API surface | Define the final contract; if phased — specify both phases |
| "at first" / "на першому етапі" | Implies unspecified future changes | Specify the complete behavior; use versioning for evolution |
| "will refactor later" / "потім переробимо" | Deferred design decisions become frozen mistakes | Design it correctly now or create a separate spec |
| "for now" / "поки що" | Same as "temporarily" | State the permanent design decision |
| "implementation detail" | Specs must be precise, not hand-wavy | Describe the contract: input, output, errors, invariants |
| "TBD" / "to be defined" | Incomplete spec passes an incomplete gate | Define it now or mark the section as blocked with a reason |

## Self-Validation Checklist

Every specification must pass these checks before advancing to the plan:

- [ ] Do all fields have data types defined?
- [ ] Are error handling responses specified?
- [ ] Is the spec precise enough to prevent hallucinations during code generation?
- [ ] Are all constraints and invariants explicitly stated?
- [ ] Are state transitions documented with conditions and side effects?
- [ ] Are verification criteria defined for all contracts (expected outcomes, edge cases)?
- [ ] Are integration scenarios described for cross-module interactions?
- [ ] Is the rollback strategy documented (Section 06)?
- [ ] Does the spec contain no banned phrases (see Banned Phrases above)?
- [ ] Is the spec minimal — no "just in case" contracts with no stated consumer?
- [ ] Is every material design fork either resolved (consensus + rationale) or recorded as an open decision with a resolution trigger in Design Decisions (see Interview Mode)?

## Structure

File structure: [templates/specification.md](../templates/specification.md).

## Changelog Requirement

A specification's Changelog follows the history policy of [Docs Scaling](../references/docs-scaling.md): entries pass the content filter (`incident` / `ambiguous-decision` / `structural-event` — an added or removed field, a changed contract, a changed validation rule, a changed state transition), and the table is optional when vcs is the primary history medium.

## Explicit Cross-References

If a specification uses entities or contracts from another specification:

1. **In metadata:** list in the `Depends on` field (same-type references — a spec depends on specs; the `Concept:` field carries the cross-type link).
2. **In body:** reference specific sections:
   ```
   > Uses [SP_ENT_01_03](./entity.sp.md#SP_ENT_01_03) AgentState structure.
   ```
3. **Reverse references:** the referenced document lists this in `Used by`.
