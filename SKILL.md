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
| — | `/dev-flow audit [scope] [--dry-run]` | Revise `.dev_flow/` — reconcile task state with reality, trim context, compact closed tasks, groom rules/skills | Audit report + cleaned context |
| — | `/dev-flow ask <question>` | Read-only Q&A about code or feasibility — no changes | Answer + optional next-step suggestion |
| — | `/dev-flow subtask <task>` | Delegate secondary task to subagent (runs any dev-flow phase: fix, test, ask, etc.) | Subtask report |
| — | `/dev-flow do <request>` | Freeform routing — interpret intent and run the right phases | Phase output + updated context |

> **Default command:** Any invocation of `/dev-flow <text>` that does not match a
> recognized phase keyword routes automatically to `do`. For example,
> `/dev-flow add Skip button to login` is equivalent to `/dev-flow do add Skip button to login`.

> **Ask command:** Use `/dev-flow ask <question>` for read-only questions about
> the codebase or feasibility of changes. No files are modified, no context is updated.

> **Audit command:** Use `/dev-flow audit` for the periodic whole-directory revision
> of `.dev_flow/` — it reconciles every task's recorded state against reality (linked
> docs + git), trims the dashboard, compacts/reflects on closed tasks (archiving noise,
> harvesting lessons into rules/skills), and grooms the `rules/` and `skills/` catalogues.
> Where [status](phases/status.md) *reports* drift, `audit` *resolves* it. Supports
> `--dry-run` (report only) and a `scope` (`context` / `tasks` / `rules` / `skills` / `all`).

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
- Design decisions settled: every material fork is resolved (consensus + rationale) or recorded as an open decision with a resolution trigger (see [Interview Mode](references/interview-mode.md))

**Specification -> Plan:**
- All data structures fully defined with types, constraints, invariants
- All contracts specified with inputs, outputs, error cases
- Verification criteria defined for all contracts (expected outcomes, edge cases)
- Integration scenarios described for cross-module interactions
- Rollback strategy documented (Section 06)
- No banned phrases (see specification phase — Banned Phrases)
- Minimality: no contracts or entities without a stated consumer
- Self-validation checklist passes (see [specification phase](phases/specification.md))
- Design decisions settled: every material modelling/contract fork is resolved (consensus + rationale) or recorded as an open decision with a resolution trigger (see [Interview Mode](references/interview-mode.md))

**Plan -> Code:**
- Plan covers ALL specification sections
- Technology decisions documented with rationale
- Contested technology forks resolved (consensus + rationale) or recorded as open decisions with a resolution trigger (see [Interview Mode](references/interview-mode.md))
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
| Design decision | `<DocID>_DEC_NN` | `C_ACS_DEC_01`, `SP_ACS_DEC_02`, `PL_ACS_DEC_01` |

The `_DEC_NN` form identifies a record in a document's **Design Decisions** section
(see [Interview Mode](references/interview-mode.md)); the `_DEC` segment is reserved
and never used as a numeric section number.

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
| Glossary | `_glossary.md` | `docs/_glossary.md` |

> **Spike** is an optional pre-concept investigation artifact. Use it when
> the problem domain is unclear and you need to explore approaches before
> committing to a concept. Spikes do not pass through the pipeline gates.

When `docs/` has more than 5 documents, maintain an `_index.md` catalog.

`docs/_glossary.md` is the project's canonical domain vocabulary (term → definition
+ aliases to avoid). It is created lazily (during onboard, or when the first
cross-concept term is resolved) and, whenever present, is **loaded into context
alongside `_index.md`** (independent of the >5-doc threshold that gates `_index.md`)
— so authoring uses one canonical term per concept. See [Glossary](references/glossary.md).

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

dev-flow uses a **collaborative per-task context model** so multiple AI agents can
work on the same project in parallel. A task file is a **shared document with
multiple contributors**; each contributor owns the parts they add and never
rewrites parts authored by others.

```
.dev_flow/
├── active_context.md          # Dashboard — table of active tasks + recently completed
├── tasks/
│   ├── _index.md              # Catalog of task files (conventions + active/recent lists)
│   ├── task_<ID>.md           # Per-task shared context — multiple contributors
│   └── ...
└── session_history/           # Archived completed tasks
```

**Source of truth = the task files.** The two index files (`active_context.md`,
`tasks/_index.md`) are derived views — any contributor can rebuild them from
`tasks/*.md` when they drift.

**Task file naming:**
- Tied to a traceable doc → `task_C_AUTH.md`, `task_PL_RATE_LIMITER.md`.
- Otherwise → `task_YYYYMMDD_HHMMSS_<slug>.md` (slug = 1–3-word kebab-case).

**Each task file contains:**
- Header: Task ID, Created, Last updated, Status, **Contributors** (list of agent IDs).
- **Current Work Item** — shared metadata (Document, Phase, Traceable ID).
- **Description** — shared, additive paragraphs signed by contributor.
- **Subtasks** — one block per contributor; each block has Author/Status/Goal/
  Progress checklist/Activity. The block's author is its sole editor.
- **Coordination Notes** — append-only conversation between contributors.
- **Blocking Issues** — each tagged with the reporter who raised it.
- **Relevant Context** — table; each row tagged with the contributor who added it.
- **Shared Activity Log** — task-level events (subtask added, contributor joined,
  status changed). Append-only, newest first.

**`active_context.md`** is a lightweight dashboard listing active tasks with phase,
status, contributors, and last-updated. **No per-task details live here.**

**Templates:**
- [templates/task_context.md](templates/task_context.md) — task file
- [templates/active_context.md](templates/active_context.md) — dashboard
- [templates/tasks_index.md](templates/tasks_index.md) — `tasks/_index.md`

### Rules for all phases

- At the **start** of any phase:
  - **Continuation** — locate the task via `active_context.md`. If your own
    Subtask block exists, resume it. If you have no block in this task yet,
    add a new `### Subtask:` block (you become a Contributor).
  - **New task** — create `tasks/task_<ID>.md` from the template with one
    Subtask block (yours). Add a row to `active_context.md` and `tasks/_index.md`
    via a **targeted Edit**.
- After **completing a step**: in your own Subtask block, check off the
  Progress item, set the next item, append to your Activity bullet list.
  Refresh the task header's `Last updated`.
- At a **phase boundary**: targeted Edit on the dashboard row (Phase / Status /
  Contributors / Updated).
- When you **finish your subtask**: set your Subtask's `Status: done`. The task
  itself stays `in-progress` until all subtasks are done.
- On **task completion** (all subtasks done): any contributor may set the
  task's overall `Status: done`, move its row from "Active" to "Recently
  Completed" in the dashboard, and update the catalog.

### Multi-contributor tolerance

Each contributor edits only their own subtask block and their own tagged entries
in shared sections. The dashboard and catalog are updated with **targeted edits**
(Edit, single row), never full rewrites.

1. **Own your block, leave others':** each Subtask block has an `Author` tag.
   Only the author edits it. Reading other contributors' blocks is encouraged —
   they are shared context.
2. **Additive shared sections:** Description paragraphs, Coordination Notes,
   Blocking Issues, Relevant Context rows, and Shared Activity Log entries are
   all tagged with their author. Append your own; do not rewrite others'.
3. **Targeted edits over rewrites:** when updating the dashboard, the catalog,
   or the task header (Last updated / Contributors / Status), use `Edit` on
   the specific field/row rather than rewriting the whole file.
4. **Read-before-write:** immediately before writing to a shared file, re-read it.
   If another contributor's edit has landed, re-apply your edit on the latest content.
5. **Append-only logs:** Coordination Notes and Shared Activity Log entries are
   added newest-first; never rewrite an existing entry.
6. **Indexes are regenerable:** if `active_context.md` or `_index.md` drift, any
   contributor may rebuild them from `tasks/*.md` headers.
7. **No exclusive locks, no time-based takeover.** If a contributor's subtask
   is stale and another wants to continue that line of work, they **add a new
   subtask block** referencing the original — they do not edit the original
   block. Coordinate explicitly via Coordination Notes.

### Hygiene

- **Shared Activity Log** per task: keep at most **10 entries** (newest first).
  Archive overflow to `.dev_flow/session_history/session_YYYY-MM-DD.md`.
- **Per-subtask Activity:** also capped at ~10 entries per block.
- **Task file size:** if a task file exceeds ~300 lines, trigger an archive cycle —
  archive completed (done) subtask blocks first.
- **Dashboard size:** `active_context.md` stays under ~80 lines. "Recently
  completed" keeps the latest 5 — older entries go to `session_history/`.
- **No large blobs** in any context file: never store logs, diffs, or verbose
  narratives — reference a file instead.

See [status phase](phases/status.md) for the full read/write protocol, regeneration
procedure, and archive flow.

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
- [Interview Mode](references/interview-mode.md) *(cross-cutting sub-procedure of concept/spec/plan/fix — surface design forks to the developer instead of choosing silently)*
- [Glossary](references/glossary.md) *(`docs/_glossary.md` — canonical domain vocabulary; created at onboard/concept, loaded with `_index.md`)*
- [Implement phase](phases/implement.md)
- [Test phase](phases/testing.md) *(conditional — functional tests: unit + mock)*
- [Review phase](phases/review.md) *(pre-commit review by clean-context subagent)*
- [Verify phase](phases/verify.md) *(conditional — regression, integration, live testing)*
- [Propagate phase](phases/propagate.md)
- [Fix phase](phases/fix.md) *(analyze, plan, fix, verify)*
- [Rule phase](phases/rule.md) *(add/edit/remove coding rules)*
- [Skill phase](phases/skill.md) *(manage project knowledge skills)*
- [Status phase](phases/status.md) | Templates: [task_context](templates/task_context.md), [active_context (dashboard)](templates/active_context.md), [tasks_index](templates/tasks_index.md)
- [Audit phase](phases/audit.md) *(full `.dev_flow/` revision — reconcile, trim, compact + reflect, groom rules/skills)*
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
| Status / all phases | [context-tracker.ai.md](roles/context-tracker.ai.md) | Reads, writes, and regenerates the per-task context model under `.dev_flow/` (task files + dashboard + catalog) |
| Audit | [auditor.ai.md](roles/auditor.ai.md) | Revises the whole `.dev_flow/` tree: reconciles task state, compacts + reflects on closed tasks, grooms rules/skills |
| Ask | [advisor.ai.md](roles/advisor.ai.md) | Read-only Q&A about code and feasibility |
| Subtask | [subtask-executor.ai.md](roles/subtask-executor.ai.md) | Executes delegated secondary tasks independently |
| Do (default) | [dev-flow-orchestrator.ai.md](roles/dev-flow-orchestrator.ai.md) | Interprets freeform requests and routes to the right phases |
