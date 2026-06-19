# Roles — Reusing and Creating Subagent Roles

Cross-cutting reference for how dev-flow's subagent roles are organized, found, reused, and created. Every phase that hands work to a subagent (see [Delegation for Focus](delegation.md)) runs it as a *role* — this document is where roles come from. It is **not** a standalone pipeline stage.

## Why this exists

A role is the reusable description of *who* does a piece of work — its responsibilities, its capabilities, and the contract for what it returns. Writing that once and reusing it keeps behavior consistent (every tester returns its results the same way) and keeps shared protocols — like "return the conclusion, not the dump" — in one place instead of re-stated in every prompt. It mirrors Skill-First Execution: inspect what already exists, reuse it, and persist what you learn so the next run starts ahead.

## Two layers

**Base roles — the skill.** The roles under the skill's `roles/` directory (implementer, tester, reviewer, …) are the portable base: generic, stable, the same across every project. They carry the responsibilities and the output contract for each phase.

**Project overlays — `.dev_flow/roles/`.** A project keeps its own roles under `.dev_flow/roles/`. Two kinds:
- An **overlay** of a base role (same slug, e.g. `tester.ai.md`) that adds *this project's* specifics — its test command, where live scenarios live, what counts as pass/fail.
- A **new specialization** (a slug the base doesn't have, e.g. `migration-verifier`) for a role this project needs and the base doesn't provide.

The base carries the contract; the overlay carries the project specifics. A role declares its base(s) by naming them in an `inherits:` field — a convention the executing agent honours when it reads the role, not engine machinery, so it works the same on any project (with or without an AI-DSL runtime).

## Two framings: obligations vs bounded autonomy

Roles are written in one of two framings — a **convention**, not a fixed schema (these files are prose projections; see the closing note). Pick the framing that fits what the role is:

- **Obligation framing** — `responsibilities` + binding `rules` (+ a `skills` gate + `workflow`). Use it for **gated pipeline executors** whose output must pass a validation gate: concept/spec/plan authors, implementer, tester, reviewer, propagator, auditor, context-tracker, the onboard-* roles, the orchestrator. These roles *must* do certain things, so they state duties and binding rules.
- **Bounded-autonomy framing** — `capabilities` (what it MAY do) + `constraints` (hard `NEVER`s). Use it for **read-only or self-directing roles** that sit outside the strict gate chain: `advisor` (read-only Q&A), `researcher` (time-boxed spike, no gates), `subtask-executor` (delegated full participant — it also carries `responsibilities`, as a deliberate hybrid). These roles act with judgement inside guardrails, so framing them as capability + limit is clearer than a duty list.

Both framings typically carry the shared spine (`title`/`description`/`inputs`/`outputs` — a lean role like `advisor` keeps only what it needs) and a `workflow` step list (structured `step_N:` entries, or a short prose block for a simple role). Everything else is role-specific and added freely. Don't normalise a role into the other framing for uniformity's sake; choose by whether the role is *obligated* or *autonomous*.

## Base roles by phase

These are the base roles the skill ships, by phase (paths relative to the skill root). Some phases use several roles; a few are handled inline without a subagent. A project overlay or specialization with the same concern takes precedence over its base where it applies.

| Phase | Base role | Purpose |
|-------|-----------|---------|
| Onboard | [onboard-coordinator.ai.md](../roles/onboard-coordinator.ai.md) | Orchestrates the full onboard procedure |
| Onboard | [onboard-analyzer.ai.md](../roles/onboard-analyzer.ai.md) | Analyzes a single module (parallelizable) |
| Onboard | [onboard-rules-extractor.ai.md](../roles/onboard-rules-extractor.ai.md) | Extracts coding rules from codebase into `.dev_flow/rules/` |
| Onboard | [onboard-docgen.ai.md](../roles/onboard-docgen.ai.md) | Generates docs from analysis (parallelizable) |
| Research | [researcher.ai.md](../roles/researcher.ai.md) | Time-boxed investigation (spike) — closes knowledge gaps before concept/spec/plan; produces options, never designs |
| Concept | [concept-author.ai.md](../roles/concept-author.ai.md) | Creates and updates concept documents |
| Specification | [spec-author.ai.md](../roles/spec-author.ai.md) | Creates and updates specifications |
| Plan | [plan-author.ai.md](../roles/plan-author.ai.md) | Creates implementation plans |
| Implement | [implementer.ai.md](../roles/implementer.ai.md) | Writes code following plans |
| Test | [tester.ai.md](../roles/tester.ai.md) | Verifies code against spec contracts |
| Review | [reviewer.ai.md](../roles/reviewer.ai.md) | Validates gates and resolves conflicts |
| Verify | Uses [tester.ai.md](../roles/tester.ai.md) | Regression, integration, and live testing |
| Propagate | [propagator.ai.md](../roles/propagator.ai.md) | Propagates changes across pipeline |
| Fix | Orchestrates multiple roles (analysis inline, implementation via [implementer.ai.md](../roles/implementer.ai.md)) | Analyzes bug, plans fix, implements, verifies |
| Rule | — (inline, no subagent) | Manages `.dev_flow/rules/` files directly |
| Skill | — (inline, no subagent) | Manages `.dev_flow/skills/` files directly |
| Resource cache (cross-cutting, not a phase) | — (inline, no subagent) | Manages `.dev_flow/cache/` files and index directly — see [Resource Cache](cache.md) |
| Status / all phases | [context-tracker.ai.md](../roles/context-tracker.ai.md) | Reads, writes, and regenerates the per-task context model under `.dev_flow/` (task files + dashboard + catalog) |
| Audit | [auditor.ai.md](../roles/auditor.ai.md) | Revises the whole `.dev_flow/` tree: reconciles task state, compacts + reflects on closed tasks, grooms rules/skills/cache |
| Ask | [advisor.ai.md](../roles/advisor.ai.md) | Read-only Q&A about code and feasibility |
| Subtask | [subtask-executor.ai.md](../roles/subtask-executor.ai.md) | Full dev-flow participant with delegated rights — assembles its own context, executes a secondary task end to end, escalates real decisions to its initiator, reports fully |
| Do (default) | [dev-flow-orchestrator.ai.md](../roles/dev-flow-orchestrator.ai.md) | Interprets freeform requests and routes to the right phases |

**Specialist focus helpers** are a *kind of project specialization*, **not shipped base roles**. When a noisy task-type recurs in a project — wide code search, log/trace triage, screenshot analysis — create a project-tailored specialist under `.dev_flow/roles/` (the "new specialization" path in [Creating or extending a role](#creating-or-extending-a-role)) and route to it by description (the [delegation routing reflex](delegation.md#named-specialists-and-the-routing-reflex)). The skill ships none: a generic specialist can't know this project's log format or UI. Their experience store is **hybrid** — narrow operational heuristics accumulate in a role-local memory file (`.dev_flow/roles/<name>.memory.md`, the same way overlays accumulate project specifics), while broadly-useful lessons are *promoted* to `.dev_flow/skills/` as proposals via [Experience Capture](experience-capture.md). They warm-start from the memory file, store only distilled heuristics (never raw payload), and stay read-only.

## The project role index

A project's `.dev_flow/roles/` carries an `_index.yaml` — a one-line-per-role catalogue (role name → what it specializes, what project specifics it adds), serving the same purpose for roles as `.dev_flow/rules/_index.yaml` does for rules. It is the first place to look before writing a new role, so you reuse an existing one instead of duplicating it. Like the other indexes it's a derived view: if it drifts from the actual role files, any contributor can rebuild it from them. Add a line to it whenever you create a role, and keep each line short — the index is for *finding* a role, the role file holds the detail.

## Using an existing role

Before delegating a step or running a phase, check what already fits — don't re-derive behavior a role already captures:
1. The phase's **base role** in the skill's `roles/` — the portable default.
2. Any **overlay or specialization** under `.dev_flow/roles/` (start from its `_index.yaml`) — these refine or replace the base for this project, so they win where they apply.

Point the subagent at the role(s) that fit and let it read them. If a project overlay exists for the phase, it's almost always the one to use.

## Creating or extending a role

When a recurring, project-specific need isn't covered:
- **Overlay** an existing role (same slug under `.dev_flow/roles/`) when you're adding this project's specifics to a base role.
- **Create a new role** (a fresh slug) when the specialization is genuinely new.

Use `inherits: [base-role, …]` to build on what exists — read it as "take these views into account," not as an algorithm that merges fields. The executing agent reads the base plus any overlays and applies judgment; when a genuine conflict appears, it asks the initiator rather than following a resolution rule. Keep roles short and in prose.

**The bar for a role is reusability.** Create a role only for something that will be used again across tasks. One-off work tied to a single ticket, date, or artifact is a *subtask*, not a role — don't mint roles that can never be reused.

## Reuse and accumulation

- Project specifics **accumulate in the overlay** over time, the same way `rules/` and `skills/` do: when a subagent hits an operational detail of its role (a flag, a path, a gotcha), record it in the overlay so the next run inherits it for free.
- What proves **broadly useful and isn't project-specific** is a candidate to lift up into the skill's base roles, so other projects get it too.

These are **projections, not formal specs** — recommended views the agent applies with judgment, not programs with inheritance semantics. A strong model authoring a role for a weaker one to execute should write it plain and unambiguous up front; that clarity is the author's job, not a runtime resolution mechanism. Don't encode merge rules or conflict-resolution machinery here.
