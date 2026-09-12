# Phase 1: Concept Authoring

## Contents

- [Purpose](#purpose) — a concept answers "what" and "why": idea, architecture, mechanisms, philosophy
- [Role Responsible](#role-responsible) — The ConceptAuthor role that executes this phase
- [Language Independence](#language-independence) — no languages, frameworks, or libraries in a concept
- [Pre-Concept Checklist](#pre-concept-checklist) — the questions to answer first; unverified answers go to research or an open decision
- [Reuse Check](#reuse-check) — mandatory search of existing concepts, code, and `docs/ext_adoption/` before creating a new concept
- [Context Loading](#context-loading) — skill/rule gates: load domain skills and architecture rules; no mechanism may violate a `must`
- [Project Glossary](#project-glossary) — reuse canonical terms from `docs/_glossary.md`, add new terms inline, escalate only material conflicts
- [Authoring Principles](#authoring-principles) — ideas over implementation, domain language, diagrams, boundaries, criticality header, formatting, Contents, changelog
- [Interview Mode for Design Decisions](#interview-mode-for-design-decisions) — surface material forks as interviews; open decisions vs banned deferrals; forecast check
- [Banned Phrases](#banned-phrases) — table of deferral phrases ("temporarily", "for now", "TBD", …) with what to write instead
- [Structure](#structure) — points at `templates/concept.md` for the concept file structure
- [Spike / Exploration (Optional)](#spike--exploration-optional) — when to run a research spike before or during authoring and how to reference it
- [Concept Granularity](#concept-granularity) — when to split, when to keep together, rule of thumb; size thresholds from Docs Scaling
- [Epic / Feature Group](#epic--feature-group) — create an `*.epic.md` when 3+ related concepts have ordering dependencies

## Purpose

A concept describes the idea, architecture, mechanisms, and philosophy. It answers "what" and "why", not "how to implement".

## Role Responsible

This phase is handled by **ConceptAuthor**: [roles/concept-author.ai.md](../roles/concept-author.ai.md).

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

**Mandatory before creating a new concept.**

1. Search existing concepts (`docs/*.concept.md`) for overlapping domain entities or mechanisms.
2. Search the codebase for existing implementations that partially solve the problem.
3. If overlap found — justify why a new concept is needed instead of extending the existing one.
4. Document findings in section "1.2 Design Constraints" or as a spike.

**External prior art.** Steps 1–2 search *inside* the project. When `docs/ext_adoption/` exists, read the adoption documents relevant to this concept's domain — an external repository may already have solved the problem, and the adoption document carries the relevance verdict and the integration risk. Cite it as the concept's origin (`Spike:` or a provenance note), the way a spike is cited. If no adoption document covers a domain where external prior art plainly exists, [`/dev-flow adopt <repo>`](../references/repo-adoption.md) produces one. Advisory only — it informs the concept, never authorizes it.

Code-altitude counterpart, applied per action burst in implement/fix: [Code Reuse](../references/code-reuse.md).

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
- **Declare criticality:** Set the header `Criticality:` — the weight of this module inside the whole application (`peripheral` / `supporting` / `core` / `critical`) — and, where a single mechanism (`§3.x`) carries a different weight than the rest, a `Criticality:` line on that section. Optional and never guessed: omit it when the weight is genuinely unknown, since an absent declaration lowers nothing. The [review](review.md) phase reads it to decide whether a finding here may block a commit. See [Review Convergence](../references/review-convergence.md).
- **Formatting:** Follow [Documentation Formatting](../references/formatting.md) — no hard line wraps inside sentences or paragraphs; one paragraph = one logical line.
- **Contents + lead summary:** the template's Contents section (substantive h2s + one-line annotations) and the lead summary in the header follow [Docs Scaling](../references/docs-scaling.md) — update the Contents item in the same edit that changes an h2.
- **Changelog:** the `## Changelog` table records history through the content filter and medium policy of [Docs Scaling](../references/docs-scaling.md) — optional when vcs is the primary history medium.

## Interview Mode for Design Decisions

When authoring surfaces **two or more materially different ways forward** (a different domain model, a different boundary, a different mechanism), do **not** pick one silently and bury it in the prose. Stop and run an interview: present the fork to the developer with 2–4 options and your **recommended answer**, reach a consensus, and record the outcome.

See **[Interview Mode](../references/interview-mode.md)** for the full procedure — when a fork counts as a decision point (and when it does not), how to frame options with a recommendation, sequential vs batched questions, and the valid outcomes (consensus vs documented open alternatives with a resolution trigger). Record every proposed, resolved, or open decision in the concept's **Design Decisions** section.

**Interview vs Banned Phrases.** A documented **open** decision (options + trade-offs + a resolution trigger) is *not* a banned deferral. The phrases below ("temporarily", "for now", …) are *undocumented* deferrals with no owner and no trigger; an open decision records the alternatives and the concrete event/date that closes it — the sanctioned way to leave something open (e.g. for research work).

**Forecast check (advisory).** Before settling a material design fork, forecast its consequences at *concept altitude* — the future intents, consumers, and use-areas it shapes — and route each anticipation through the YAGNI-gate (`build now` / `seam+flag` / `drop+record`). At this altitude forecasting is cheap and encouraged, but it yields *direction and seams*, not speculative machinery with no stated consumer; a genuinely uncertain build-vs-defer fork becomes an interview above. See [Consequence Forecasting](../references/consequence-forecasting.md).

## Banned Phrases

Reject or flag the following phrases in any concept.

| Phrase | Problem | What to write instead |
|--------|---------|----------------------|
| "temporarily" / "тимчасово" | Temporary solutions become permanent | Define when and how it ends — or design the full solution |
| "at first" / "на першому етапі" | Implies an unwritten "second stage" that never comes | Describe the complete solution; if phased — define all phases in the plan |
| "will refactor later" / "потім переробимо" | Deferred cleanup rarely happens | Either do it now or create a separate concept for the refactoring |
| "for now" / "поки що" | Same as "temporarily" | State the permanent design decision |
| "simple wrapper around X" | Wrappers grow into unmaintainable layers | Describe the actual abstraction and its boundaries |
| "TBD" / "to be defined" | A hidden deferral with no owner and no trigger | Resolve it now, run a [research spike](research.md), or record an open decision with a resolution trigger |

## Structure

File structure: [templates/concept.md](../templates/concept.md).

## Spike / Exploration (Optional)

A **spike** is a time-boxed exploration that feeds the concept, executed by the **[research phase](research.md)** (`/dev-flow research`, alias `spike`).

### When to Spike

- The domain is unfamiliar — need to understand constraints before committing to an architecture.
- Multiple competing approaches exist — need to compare before choosing (the spike *discovers* options; the [interview](../references/interview-mode.md) then *chooses*).
- A critical technical question blocks concept authoring (e.g., "can library X handle Y?").
- A Pre-Concept Checklist answer rests on an unverified assumption.
- The user explicitly asks for exploration before design.

When any of these hold mid-authoring, pause the concept, run the [research phase](research.md), then resume with the findings. Reference the spike from the concept: `Spike: [name.spike.md](./name.spike.md)`.

Spike file structure: [spike template](../templates/spike.md).

## Concept Granularity

### When to Split

A concept should be split when:
- It describes more than one independent responsibility (SRP for documents).
- Different parts change for different reasons or at different rates.
- The audit's size-trigger verdict says so — split thresholds are measured in words, never lines, and live in [Docs Scaling](../references/docs-scaling.md).

**Example:** "Access Control" might split into:
- `C_AUTHN` — Authentication (identity verification)
- `C_AUTHZ` — Authorization (permission checks)
- `C_AUDIT` — Audit trail (logging access decisions)

### When to Keep Together

Keep unified when:
- Parts are tightly coupled — changing one always requires changing the other.
- Splitting would create excessive cross-references.
- The concept is small — well under the split thresholds of [Docs Scaling](../references/docs-scaling.md) — and focused.

### Rule of Thumb

Ask: "Can someone work on part A without understanding part B?" If yes — split.

## Epic / Feature Group

When a large feature requires 3+ related concepts with ordering dependencies, create an epic document (`*.epic.md`) to coordinate them. See [epic template](../templates/epic.md) for the structure.
