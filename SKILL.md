---
name: dev-flow
description: >
  Feature development workflow following concept-spec-plan-implementation pipeline
  with traceable IDs. Use when: creating concept/spec/plan files, planning new features,
  modifying existing functionality, propagating changes across concept-spec-plan-code,
  reviewing documentation pipeline, assessing impact of architectural changes,
  asking questions about the codebase or feasibility of changes (read-only),
  or working with concept/spec documents.
user-invocable: true
argument-hint: "[phase] [target]"
---

# Concept-Driven Development

All changes to the system start from concepts and specifications, not from code.
The pipeline is strictly top-down:

```
Concept -> [Gate] -> Specification -> [Gate] -> Plan -> [Gate] -> Code -> [Gate] -> Test* -> [Gate] -> Review** -> [Gate] -> Verify* -> Commit -> Propagate
```

Each transition includes a validation gate to prevent drift.

`*` — Test and Verify phases are conditional: they activate only when the project
has tests and/or defined rules for running/validating them.

`**` — Pre-commit review is performed by a subagent with a clean context to ensure
an unbiased perspective on the changes.

**Two-stage testing:**
- **Test** — functional tests only (unit + mock) covering the changed code.
- **Verify** — regression, integration, and live testing (end-to-end flows, app/service launch).
  If Verify finds issues → fix code → re-run Test (if exists) → re-run Review → re-run Verify.

## Pipeline Phases

| Phase | Command | Purpose | Output |
|-------|---------|---------|--------|
| 0. Onboard | `/dev-flow onboard` | Reverse-engineer docs from existing code | `*.concept.md`, `*.sp.md`, `*.plan.md` |
| 1. Concept | `/dev-flow concept` | Define the idea, architecture, mechanisms | `*.concept.md` |
| 2. Specification | `/dev-flow spec` | Define data structures, contracts, rules | `*.sp.md` |
| 3. Plan | `/dev-flow plan` | Break spec into actionable phases | `*.plan.md` |
| 4. Implement | `/dev-flow implement` | Write code following the plan | Source code |
| 5. Test | `/dev-flow test [target]` | Run functional tests (unit + mock) for changed code | Test results |
| 6. Review | `/dev-flow review` | Pre-commit review + validate gates, resolve conflicts | Review report |
| 7. Verify | `/dev-flow verify [target]` | Regression, integration, and live testing | Verification results |
| 8. Propagate | `/dev-flow propagate` | Update docs when code changes | Updated docs |
| — | `/dev-flow fix <problem>` | Analyze bug, plan fix, implement, verify | Fixed code + build/test result |
| — | `/dev-flow rule <request>` | Add, edit, or remove coding rules (freeform) | Updated `.dev_flow/rules/` |
| — | `/dev-flow skill <request>` | Find, add, update, or remove project knowledge skills | Updated `.dev_flow/skills/` |
| — | `/dev-flow status` | Show current state, resume previous session | Status summary |
| — | `/dev-flow ask <question>` | Read-only Q&A about code or feasibility — no changes | Answer + optional next-step suggestion |
| — | `/dev-flow subtask <task>` | Delegate secondary task to subagent (runs any dev-flow phase: fix, test, ask, etc.) | Subtask report |
| — | `/dev-flow do <request>` | Freeform routing — interpret intent and run the right phases | Phase output + updated context |

> **Default command:** Any invocation of `/dev-flow <text>` that does not match a
> recognized phase keyword routes automatically to `do`. For example,
> `/dev-flow add Skip button to login` is equivalent to `/dev-flow do add Skip button to login`.

> **Ask command:** Use `/dev-flow ask <question>` for read-only questions about
> the codebase or feasibility of changes. No files are modified, no context is updated.

> **Phase 0 (Onboard):** Optional. Run once when adopting dev-flow for an existing project.
> Analyzes code bottom-up (utilities first), generates concepts, specs, and plans.
> Supports `--resume` for continuation across sessions. See [onboard phase](phases/onboard.md).

> **Phase 5 (Test) activation condition:** The project must have an existing test suite
> AND defined rules for running tests (e.g., `pytest`, `jest`, CI pipeline).
> If neither exists — skip directly to Review. Runs only **functional tests** (unit + mock)
> covering the changed code. Does NOT run integration or live tests at this stage.

> **Phase 6 (Review):** Pre-commit code review is performed by a subagent with a
> clean context to ensure an unbiased perspective. This is mandatory before any commit.

> **Phase 7 (Verify):** Regression, integration, and live testing — runs after Review
> passes. Includes launching the app/service and verifying end-to-end scenarios.
> If issues are found during Verify, the cycle repeats: fix → Test → Review → Verify.
> Ask user permission before creating new integration/live scenarios.
> If no automated verification exists — provide manual verification steps for the user.

### Quick Start

Before writing or changing any code, ask yourself:
1. Does the concept describe what I'm building? If not — update the concept first.
2. Does the specification define the data structures and contracts? If not — update the spec first.
3. Only then — create/update the implementation plan and write code.

## Validation Gates

**Concept -> Specification:**
- No contradictions with existing active concepts
- All integration points listed in Dependencies
- Scope clearly bounded (what this IS and IS NOT)
- Pre-Concept Checklist answered (see concept phase)
- Reuse Check completed — no unjustified overlap with existing concepts
- No banned phrases (see concept phase — Banned Phrases)
- Minimality: concept describes the minimum viable solution — no "just in case" sections with no stated consumer

**Specification -> Plan:**
- All data structures fully defined with types, constraints, invariants
- All contracts specified with inputs, outputs, error cases
- Verification criteria defined for all contracts (expected outcomes, edge cases)
- Integration scenarios described for cross-module interactions
- Rollback strategy documented (Section 06)
- No banned phrases (see specification phase — Banned Phrases)
- Minimality: no contracts or entities without a stated consumer
- Self-validation checklist passes (see [specification phase](phases/specification.md))

**Plan -> Code:**
- Plan covers ALL specification sections
- Technology decisions documented with rationale
- Phase dependencies explicitly stated

**Code -> Test** (conditional — functional tests):
- All spec contracts have corresponding test cases
- All error cases from spec's Errors tables are tested
- All invariants from spec are verified in tests
- Only tests covering the changed code are run (unit + mock)

**Test -> Review:**
- All relevant functional tests pass
- No regressions in changed code area

**Review -> Verify** (conditional — regression/live):
- Pre-commit review by a clean-context subagent passes (no blocking issues)
- Warnings presented to user and acknowledged
- Ask user permission before creating new integration/live test scenarios
- If no automated verification is possible — provide manual verification steps

**Verify -> Commit:**
- Integration tests pass (if applicable)
- Live tests pass or manual verification completed (app/service launched, scenario checked)
- If Verify finds issues → fix code → re-run Test (if exists) → re-run Review → re-run Verify
- Ask the user for explicit commit approval before committing

## Document Status Vocabulary

All document types use a unified set of statuses with clear lifecycle:

```
draft -> active -> deprecated        (concepts, specifications — living documents)
draft -> in-progress -> completed    (plans — finite work items)
```

| Status | Meaning | Applies to |
|--------|---------|------------|
| `draft` | Initial version, not yet validated through gate | All |
| `active` | Validated and current — the authoritative version | Concepts, Specifications |
| `in-progress` | Being actively worked on | Plans |
| `completed` | All work done, no further changes expected | Plans |
| `deprecated` | Superseded or no longer relevant — excluded from conflict checks | All |

## Versioning

When a concept or specification undergoes a **breaking change** (incompatible contract changes,
removed entities, fundamentally different approach), create a new version instead of editing in place:

1. Set old document to `Status: deprecated` with `Deprecated-reason: Replaced by [C_XXX_v2](./path)`
2. Create new document with version suffix: `C_XXX_v2`, `SP_XXX_v2`
3. Update all `Depends on` / `Used by` references in dependent documents
4. Both versions may coexist while dependents migrate

For **non-breaking changes** (adding fields, extending contracts, fixing descriptions),
edit the existing document in place and update the Changelog.

## When Modifying Existing Functionality

1. Update the **concept** — what changed in the idea or architecture?
2. Update the **specification** — what data structures or contracts changed?
3. Update the **implementation plan** — mark completed, add new tasks.
4. Update the **code** — implement according to the updated spec.
5. Run **functional tests** — unit + mock tests covering the changed code (if test suite exists).
6. Run **pre-commit review** — subagent with clean context reviews the changes.
7. Run **verification** — regression, integration, and/or live tests; or provide manual verification steps.
   If issues found → fix → re-run steps 5 (if tests exist), 6, 7.
8. **Ask for commit approval** — present changes for review before committing.

## Traceable Identifiers

Every section has a unique, immutable identifier:

| Document | Format | Example |
|----------|--------|---------|
| Concept | `C_XXX_NN_NN` | `C_ACS_01_01` |
| Specification | `SP_XXX_NN_NN` | `SP_ACS_01_01` |
| Plan | `PL_XXX` | `PL_ACS` |

In code, reference these as comments: `# [C_ACS_03_01] PermissionInterceptor`

## File Organization

All documents live in `docs/` directories. One concept = one file set:

| File | Extension | Example |
|------|-----------|---------|
| Spike | `*.spike.md` | `access_control.spike.md` |
| Concept | `*.concept.md` | `access_control.concept.md` |
| Specification | `*.sp.md` | `access_control.sp.md` |
| Plan | `*.plan.md` | `access_control.plan.md` |
| Index | `_index.md` | `docs/_index.md` |

> **Spike** is an optional pre-concept investigation artifact. Use it when
> the problem domain is unclear and you need to explore approaches before
> committing to a concept. Spikes do not pass through the pipeline gates.

When `docs/` has more than 5 documents, maintain an `_index.md` catalog.

## Git Workflow Integration

Pipeline artifacts map to git workflow as follows:

| Scope | Branch / PR |
|-------|-------------|
| Concept + Specification | One PR — reviewed together as a design unit |
| Implementation Plan | Separate PR — technology decisions reviewed independently |
| Implementation (per plan phase) | One PR per plan phase — incremental, reviewable |
| Review + Propagation | Same PR as the triggering change |

**Commit rules:**
1. **Never commit without explicit user approval.** After completing a phase, present the changes and ask: "Ready to commit?"
2. **Run pre-commit review** by a clean-context subagent before asking for commit approval.
3. Only commit after the review passes and the user confirms.
4. Commit message must reference the traceable ID: `[C_XXX] / [SP_XXX] / [PL_XXX]`.

## Active Context & Session Continuity

All dev-flow commands maintain a persistent state file:

```
.dev_flow/active_context.md
```

This file tracks the current work item, pipeline phase, progress state, blocking
issues, and relevant documents. It is the single source of truth for resuming work
across sessions.

**Rules for all phases:**
- At the **start** of any phase: update `active_context.md` — set document, phase, status = in-progress.
- After **completing a step**: check it off in Progress State and update Next step.
- After **completing a phase**: set status = done, add entry to Recent Changes.
- **Template** for new files: [templates/active_context.md](templates/active_context.md).

**Hygiene:** `active_context.md` is "state as of now", not a cumulative journal.
- **Recent Changes**: keep at most **10 entries**. Archive older entries to `.dev_flow/session_history/session_YYYY-MM-DD.md`.
- **Current Task**: describe only the active work item. Do not accumulate narratives from completed tasks.
- **File size**: if the file exceeds ~150 lines, trigger an archive cycle.
- **No large blobs**: never store logs, diffs, or verbose narratives directly — reference a file instead.

See [status phase](phases/status.md) for full update protocol and archive procedure.

## Project Rules

When `.dev_flow/rules/` exists, all new code must comply with the documented rules.
Rules are extracted during onboard and updated during implement/review phases.

```
.dev_flow/rules/
├── _index.yaml         # Index of all rules (YAML)
├── naming.md           # Naming conventions
├── structure.md        # Code structure patterns
├── architecture.md     # Architectural constraints
├── error-handling.md   # Error handling patterns
├── style.md            # Code style and formatting
└── testing.md          # Testing patterns (if tests exist)
```

Severity levels: **must** (blocks review) | **should** (warning) | **prefer** (advisory).
Rules apply to new code only — no retroactive refactoring required.

**Auto-discovery of new rules:**
During `implement` and `fix` phases, if the implementation plan or fix analysis
reveals a coding pattern, constraint, or convention that is not yet captured in
`.dev_flow/rules/` — add it automatically. Specifically:
- If a plan phase describes a new architectural constraint, naming convention,
  or error-handling pattern — create or update the corresponding rule file.
- If a `fix` phase root-cause analysis identifies a violated invariant that has
  no matching rule — add the rule so the same class of bug is prevented in the future.
- New rules default to severity **should** unless the plan/fix explicitly marks them
  as **must** or **prefer**.
- After adding a rule, update `.dev_flow/rules/_index.yaml` (add entry with `file` and `summary`)
  and mention the new rule in the commit message.

## Greenfield vs Takeover

- **Greenfield:** concept -> domain model -> specification -> plan -> code.
- **Takeover:** `/dev-flow onboard` — automated reverse-engineering pipeline:
  1. Map project structure and module dependencies.
  2. Group modules into dependency layers (leaf utilities = Layer 0).
  3. Analyze each module bottom-up, extracting entities, contracts, invariants.
  4. Extract project coding rules into `.dev_flow/rules/`.
  5. Generate concept → spec → plan for each module, maintaining cross-references.
  6. Intermediate state saved in `.dev_flow/onboard/` — supports `--resume`.
  See [onboard phase](phases/onboard.md) for full procedure.

## Phase Details & Templates

- [Onboard phase](phases/onboard.md) *(optional, for takeover)*
- [Concept phase](phases/concept.md) | [Template](templates/concept.md) | [Spike template](templates/spike.md)
- [Specification phase](phases/specification.md) | [Template](templates/specification.md)
- [Plan phase](phases/plan.md) | [Template](templates/plan.md)
- [Implement phase](phases/implement.md)
- [Test phase](phases/testing.md) *(conditional — functional tests: unit + mock)*
- [Review phase](phases/review.md) *(pre-commit review by clean-context subagent)*
- [Verify phase](phases/verify.md) *(conditional — regression, integration, live testing)*
- [Propagate phase](phases/propagate.md)
- [Fix phase](phases/fix.md) *(analyze, plan, fix, verify)*
- [Rule phase](phases/rule.md) *(add/edit/remove coding rules)*
- [Skill phase](phases/skill.md) *(manage project knowledge skills)*
- [Status phase](phases/status.md) | [Active context template](templates/active_context.md)
- [Ask phase](phases/ask.md) *(read-only Q&A, no file changes)*
- [Subtask phase](phases/subtask.md) *(delegate secondary tasks to subagent)*
- [Do phase](phases/do.md) *(default fallback for freeform requests)*
- [End-to-end example](examples/rate-limiter.md)

## Subagent Roles (AI-DSL)

Each phase has a specialized role for subagent execution:

| Phase | Role file | Purpose |
|-------|-----------|---------|
| Onboard | [onboard-coordinator.ai.md](roles/onboard-coordinator.ai.md) | Orchestrates the full onboard procedure |
| Onboard | [onboard-analyzer.ai.md](roles/onboard-analyzer.ai.md) | Analyzes a single module (parallelizable) |
| Onboard | [onboard-rules-extractor.ai.md](roles/onboard-rules-extractor.ai.md) | Extracts coding rules from codebase into `.dev_flow/rules/` |
| Onboard | [onboard-docgen.ai.md](roles/onboard-docgen.ai.md) | Generates docs from analysis (parallelizable) |
| Concept | [concept-author.ai.md](roles/concept-author.ai.md) | Creates and updates concept documents |
| Specification | [spec-author.ai.md](roles/spec-author.ai.md) | Creates and updates specifications |
| Plan | [plan-author.ai.md](roles/plan-author.ai.md) | Creates implementation plans |
| Implement | [implementer.ai.md](roles/implementer.ai.md) | Writes code following plans |
| Test | [tester.ai.md](roles/tester.ai.md) | Verifies code against spec contracts |
| Review | [reviewer.ai.md](roles/reviewer.ai.md) | Validates gates and resolves conflicts |
| Verify | Uses [tester.ai.md](roles/tester.ai.md) | Regression, integration, and live testing |
| Propagate | [propagator.ai.md](roles/propagator.ai.md) | Propagates changes across pipeline |
| Fix | Orchestrates multiple roles (analysis inline, implementation via [implementer.ai.md](roles/implementer.ai.md)) | Analyzes bug, plans fix, implements, verifies |
| Rule | — (inline, no subagent) | Manages `.dev_flow/rules/` files directly |
| Skill | — (inline, no subagent) | Manages `.dev_flow/skills/` files directly |
| Status / all phases | [context-tracker.ai.md](roles/context-tracker.ai.md) | Reads and writes `.dev_flow/active_context.md` |
| Ask | [advisor.ai.md](roles/advisor.ai.md) | Read-only Q&A about code and feasibility |
| Subtask | [subtask-executor.ai.md](roles/subtask-executor.ai.md) | Executes delegated secondary tasks independently |
| Do (default) | [dev-flow-orchestrator.ai.md](roles/dev-flow-orchestrator.ai.md) | Interprets freeform requests and routes to the right phases |
