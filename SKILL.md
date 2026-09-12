---
name: dev-flow
description: >
  Feature development workflow following concept-spec-plan-implementation pipeline
  with traceable IDs. Use when: creating concept/spec/plan files, planning new features,
  modifying existing functionality, fixing bugs (analyze, fix, verify),
  propagating changes across concept-spec-plan-code, reviewing the documentation
  pipeline or code before commit, assessing impact of architectural changes,
  asking questions about the codebase or feasibility of changes (read-only),
  researching unknowns before committing to a design (spike),
  capturing future work as todos, delegating secondary tasks to subagents,
  onboarding an existing codebase (reverse-engineer docs from code),
  resuming a previous session or checking status, auditing and grooming
  project context (.dev_flow/, docs/) or the whole codebase,
  managing project coding rules and knowledge skills,
  analyzing an external repository to decide what to borrow for THIS project and
  writing the result into docs/ext_adoption/ ("adopt", "analyze repo", "adoption",
  "what can we take from X", "which ideas from X are useful here", "розбери репозиторій",
  "проаналізуй репо", "що корисного взяти", "що запозичити") — prefer this over a
  standalone repo-analysis skill whenever the project uses dev-flow (docs/ or .dev_flow/ present),
  or working with concept/spec documents.
user-invocable: true
argument-hint: "[phase] [target]"
---

# Concept-Driven Development

All changes to the system start from concepts and specifications, not from code. The pipeline is strictly top-down:

```
Concept -> [Gate] -> Specification -> [Gate] -> Plan -> [Gate] -> Code -> [Gate] -> Test* -> [Gate] -> Review** -> [Gate] -> Verify* -> Commit -> Propagate
```

Each transition includes a validation gate to prevent drift.

`*` — Test and Verify phases are conditional: they activate only when the project has tests and/or defined rules for running/validating them.

`**` — Pre-commit review is performed by a subagent with a clean context to ensure an unbiased perspective on the changes.

## Contents

- [Pipeline Phases](#pipeline-phases) — command per phase and service command; default routing to `do`; conditional Test/Verify; Quick Start preflight
- [Project Knowledge Is Binding](#project-knowledge-is-binding) — mandatory rules/skills gate, severities, precedence, re-trigger at the moment of action
- [Validation Gates](#validation-gates) — the pass/fail checklist at every transition from Concept to Commit
- [Developer Checkpoints](#developer-checkpoints) — design sign-off and commit sign-off; the `Autonomy` field
- [Git Workflow Integration](#git-workflow-integration) — branch/PR mapping; the commit rules
- [Active Context & Session Continuity](#active-context--session-continuity) — `.dev_flow/` task-context model, memory tiers, rules for all phases (start / step / transition / completion), multi-contributor tolerance, hygiene
- [Document Status Vocabulary](#document-status-vocabulary) — lifecycle statuses of concepts, specs, plans
- [Versioning](#versioning) — breaking change → new version; non-breaking → edit in place
- [When Modifying Existing Functionality](#when-modifying-existing-functionality) — change classes; concept → spec → plan → sign-off → code → test → review → verify → sign-off
- [Traceable Identifiers](#traceable-identifiers) — ID formats for concept, spec, plan, epic, design decision
- [File Organization](#file-organization) — `docs/` file set per concept; index, glossary, framework map, adoption files; umbrella sets
- [Project Rules](#project-rules) — `.dev_flow/rules/` layout, severities, unit form, auto-discovery of rules
- [Greenfield vs Takeover](#greenfield-vs-takeover) — entry routes into the pipeline
- [Phase Details & Templates](#phase-details--templates) — router to every phase, reference, template, and example file
- [Delegation for Focus (Context Isolation)](#delegation-for-focus-context-isolation) — hand noisy work to a subagent, keep only the conclusion
- [Subagent Roles (AI-DSL)](#subagent-roles-ai-dsl) — base roles, project overlays, specialist focus helpers

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
| — | `/dev-flow research <topic>` | Time-boxed investigation (spike) when knowledge is insufficient for a concept/spec/plan or to close an open decision | `*.spike.md` + updated skills |
| — | `/dev-flow fix <problem>` | Analyze bug, plan fix, implement, verify | Fixed code + build/test result |
| — | `/dev-flow rule <request>` | Add, edit, remove, or list coding rules (freeform) | Updated `.dev_flow/rules/` |
| — | `/dev-flow skill <request>` | Find, add, update, or remove project knowledge skills | Updated `.dev_flow/skills/` |
| — | `/dev-flow status` | Show current state, resume previous session | Status summary |
| — | `/dev-flow audit [scope] [--dry-run]` | Revise `.dev_flow/` and `docs/` — reconcile task state with reality, trim context, compact closed tasks, groom rules/skills/cache, check docs integrity (index/statuses/refs/orphans/freshness/duplicated sets/scaling); opt-in `code` scope audits the whole codebase → refactoring plan | Audit report + cleaned context (or, for `code`, a refactoring plan) |
| — | `/dev-flow adopt <repo>` | Analyze an external repository at concept altitude and produce the adoption document for this project (what to borrow, what to skip, in what order) | `docs/ext_adoption/*.concept.md` + `docs/ext_adoption/*.md` |
| — | `/dev-flow ask <question>` | Read-only Q&A about code or feasibility — no changes | Answer + optional next-step suggestion |
| — | `/dev-flow todo <description>` | Capture future work — find relevant docs, assess feasibility, file a planning record with a return trigger (does not build) | Plan backlog item or `.dev_flow/todos/` entry |
| — | `/dev-flow subtask <task>` | Delegate a secondary task to a subagent — a full dev-flow participant that assembles its own context, runs any phase (fix, test, research, etc.), and can converse with its initiator | Full subtask report |
| — | `/dev-flow do <request>` | Freeform routing — interpret intent and run the right phases | Phase output + updated context |

**Default command:** `/dev-flow <text>` with no recognized phase keyword routes to `do`.

**Service commands — facts the table omits.** `research` (alias `spike`) is time-boxed, cost-gated, passes no gate, and persists durable findings to `.dev_flow/skills/`. `todo` infers flavor (`deferred` / `queued` / `contested`) and trigger from plan/task state, files into the owning plan's backlog else `.dev_flow/todos/`, and builds nothing. `adopt` (aliases `analyze-repo`, `adoption`) is advisory — no ID, no gate — and writes only its two documents, the clone in `ext_repos/`, one `.gitignore` line, and task context. `audit`'s `code <intent>` scope is opt-in, excluded from `all`, writes its plan under `.dev_flow/audit/`, and stops at the Plan→Code gate. `onboard` runs once. The **resource cache** (`.dev_flow/cache/` + `_index.yaml`, not a phase) is checked before any expensive re-fetch; transients go to `/tmp/{project-slug}/` with timestamped names ([Resource Cache](references/cache.md)).

**Conditional and mandatory phases.** Test (5) runs only when the project has a test suite AND rules for running it — otherwise skip to Review and record the skip as `unobserved` with its reason ([Evidence Discipline](references/evidence-discipline.md)); it runs functional tests (unit + mock) on the changed code only. Review (6) is mandatory before any commit and is done by a clean-context subagent; the fix → re-review loop is bounded by [Review Convergence](references/review-convergence.md), and a contained non-`must`, non-security fix is confirmed mechanically instead of re-reviewed ([Verification Economy](references/verification-economy.md)). Verify (7) — regression, integration, live — runs after Review passes; issues → fix → Test → Review → Verify at the scope the fix earns, never narrowed for a `must` or security finding; ask before creating new integration/live scenarios; with no automation, give manual steps and record `unobserved`.

### Quick Start

Before any code edit: the concept must describe the change and the spec must define its structures and contracts — otherwise update them first, in that order, then the plan.

## Project Knowledge Is Binding

Loading `.dev_flow/rules/` and `.dev_flow/skills/` is a mandatory gate at the start of every code/test/doc phase; applying them is not optional.

- **Rules (`.dev_flow/rules/`)** — load the rules whose selectors match the paths you touch and the current phase, plus every always-on category (no selector) — the *relevant set*: computed, directives first, a body only on a trigger; over `activation_budget` drops nothing and lowers no severity (see [Knowledge Scaling](references/knowledge-scaling.md)); new code MUST comply:
  - `must` — violation **blocks**: fix it or surface the conflict and stop.
  - `should` — violation needs a recorded justification.
  - `prefer` — follow unless there is a stated local reason not to.
- **Skills (`.dev_flow/skills/`)** — check `_index.yaml` and load matching skills BEFORE external research; a skill's "Pitfalls"/"Usage in This Project" override generic knowledge. Skills are distilled **procedural** memory: a *current* matching skill outranks the model's general prior, while a *stale* one (tool/framework version or context drifted) is demoted and must be re-grounded before it can override fresh research. See [Procedural Skills](references/procedural-skills.md).
- **Precedence** — project rules override generic guidance (incl. the [SOLID reference](references/solid-architecture.md)); project skills override generic technology knowledge. On conflict: project wins, or surface it.

If the directory is absent, the gate is a no-op. Each relevant phase restates this as its "Skill check" / "Rule check" — gates, not reminders. The gate is also **re-triggered at the moment of action** — the relevant set is re-surfaced per action burst beside the work (a pointer-only Pre-Action Marker), not only once at phase start. See [Application Enforcement](references/application-enforcement.md).

A rule or skill carries an optional **evidence state** (`present` → `wired` → `exercised` → `outcome-supported`, or `unobserved`) with mandatory provenance; configured ≠ used, count ≠ conclusion. See [Evidence Discipline](references/evidence-discipline.md).

## Validation Gates

**Concept -> Specification:**
- No contradictions with existing active concepts
- All integration points listed in Dependencies
- Scope clearly bounded (what this IS and IS NOT)
- Pre-Concept Checklist answered (see concept phase) — answers rest on verified knowledge: an unverified critical assumption goes through [research](phases/research.md) (spike) first, or is explicitly accepted by the user as an open decision with a trigger
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
- Every phase declares what to verify on completion — a `Verify:` field naming the spec Verification Criteria (`SP_XXX_05_*`) for its contracts plus any phase-local acceptance check (the reusable checklist for [Test](phases/testing.md)/[Verify](phases/verify.md))

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
- Regression tests pass (if applicable)
- Integration tests pass (if applicable)
- Live tests pass or manual verification completed (app/service launched, scenario checked)
- If Verify finds issues → fix code → re-run Test (if exists) → re-run Review → re-run Verify — no fix cycle left incomplete
- If a failure traces to the spec/plan itself (not the code) — do not bend the code: escalate upstream first (see [Upstream Escalation](references/escalation.md))
- Reflection checkpoint run — durable lessons harvested (see [Experience Capture](references/experience-capture.md))
- Ask the user for explicit commit approval before committing — the commit sign-off of [Developer Checkpoints](#developer-checkpoints)

## Developer Checkpoints

Stops in every route where the main agent presents its work to the developer and waits. Stops, not gate criteria.

- **Design sign-off** — after the last design document, before the first edit to code or shipped files. Present: the Design Decisions of the change (question → options → recommendation), the documents created or changed, the files implementation will touch. A record the developer has not answered is `proposed` ([Interview Mode](references/interview-mode.md)); implementation never starts on one. A Trivial change has no design documents — no stop.
- **Commit sign-off** — after review passes, before `git commit`. Present: changed files, review verdict with open advisories, intent verdict. Ask "Ready to commit?" and wait — never on an inferred yes.

**Autonomy.** Skipped only when the request itself orders it ("автономно", "без зупинок", "commit without asking"); a harness or system autonomy setting does not count. Record `Autonomy: full — "<quote>"` in the task header at intake; absent → `checkpoints`. Task-scoped; a decision settled under `full` is `resolved (delegated)`. Subagents never hold a checkpoint — only the main agent talks to the developer.

## Git Workflow Integration

| Scope | Branch / PR |
|-------|-------------|
| Concept + Specification | One PR — reviewed together as a design unit |
| Implementation Plan | Separate PR — technology decisions reviewed independently |
| Implementation (per plan phase) | One PR per plan phase — incremental, reviewable |
| Review + Propagation | Same PR as the triggering change |

**Commit rules:**
1. **Never commit without explicit user approval.** After completing a phase, stop at the commit sign-off ([Developer Checkpoints](#developer-checkpoints)): present the changes and ask "Ready to commit?". Only `Autonomy: full` in the task header skips it.
2. **Run pre-commit review** by a clean-context subagent before asking for commit approval.
3. Only commit after the review passes and the user confirms.
4. Commit message must reference the traceable ID: `[C_XXX] / [SP_XXX] / [PL_XXX]`. When the task is tied to a tracker ticket, the ticket key joins the ID in the message (e.g. `[PROJ-123][SP_XXX] …`) and the tracker's commit-time conventions apply, **each outward write confirmed first**. See [Ticket Tracker Integration](references/ticket-tracker.md).
5. **After the commit (and push, when pushed)** on a ticket-tied task, proactively propose adding the corresponding ticket comment/worklog + status transition per the tracker's conventions — confirmed, never silent. See [Ticket Tracker Integration](references/ticket-tracker.md).

## Active Context & Session Continuity

dev-flow keeps a **collaborative per-task context** in `.dev_flow/` so several agents can work one project in parallel. A task file is a shared document: each contributor owns the parts it adds and never rewrites another's. The full read/write protocol, regeneration procedure, and archive flow live in the [status phase](phases/status.md).

**Memory tiers.** L0 = the live transcript (lost on compact); L1 = session scratch — the [session working memory](references/cache.md#session-working-memory-l1) (notes / params / reminders / reads) plus the `/tmp/{project-slug}/` data cache (survives compact, not restart); L2 = `.dev_flow/` (durable). [Experience Capture](references/experience-capture.md) promotes L1 → L2; [salience markers](phases/status.md#salience-markers) decide what survives a compaction. Write to working memory as you work (parameter set, non-obvious fact, deferred action, every file read) and re-read the whole area whenever the thread is lost; a file already read this session, unchanged and uninvalidated, is not read again ([Verification Economy](references/verification-economy.md)).

Layout of `.dev_flow/`: [status phase → Context Files](phases/status.md#context-files).

**Source of truth = the task files**; `active_context.md` and `tasks/_index.md` are derived views any contributor may rebuild. Naming: `task_C_AUTH.md` when tied to a traceable doc, else `task_YYYYMMDD_HHMMSS_<slug>.md`. Task-file sections (header with `Autonomy`, Current Work Item, **Intent**, Subtask blocks, Coordination Notes, Blocking Issues, Relevant Context, Shared Activity Log) and their ownership: [templates/task_context.md](templates/task_context.md); dashboard and catalog: [templates/active_context.md](templates/active_context.md), [templates/tasks_index.md](templates/tasks_index.md).

### Rules for all phases

- **Project-knowledge gate (first).** Read `.dev_flow/rules/_index.yaml` and `.dev_flow/skills/_index.yaml`, load what matches the area you touch, obey it ([Project Knowledge Is Binding](#project-knowledge-is-binding)).
- **Style gate.** Read `.dev_flow/output_styles.md`; documentation register for files, chat register for the developer (every link/path/ID carries a short description). Absent → shipped defaults ([Output Styles](references/output-styles.md)).
- **Resource gate.** Before an expensive fetch check `.dev_flow/cache/_index.yaml`; after one, save the artifact back (`trust: public` + safety check for open-internet sources); transients to `/tmp/{project-slug}/` ([Resource Cache](references/cache.md)).
- **Phase start.** Continuation: locate the task via `active_context.md`, resume your Subtask block or add a new one (you become a Contributor). New task: create `tasks/task_<ID>.md` from the template with your Subtask block, the **Intent**, and the **Autonomy** field (`full` only when the request says so, quoted); add a row to `active_context.md` and `tasks/_index.md` by targeted Edit.
- **After a step.** Check off Progress, set the next item, append Activity (incidents, ambiguous decisions, structural events only); refresh `Last updated`.
- **Phase boundary.** Targeted Edit on the dashboard row. **Transition** (phase/subtask boundary, task switch): run the Transition Checkpoint — `{s:pin}` summary, demote raw entries, harvest durable lessons through the structural rule/skill gate (never an auto-`must`), promote working memory to the task file ([Experience Capture](references/experience-capture.md)).
- **Spotted-defect reflex.** An out-of-scope, deferrable defect is filed as an agent-initiated [`todo`](phases/todo.md) (cheap subagent, or inline if trivial) — never chased, never lost. In-scope or urgent → fix or escalate.
- **Subtask finished.** Set its `Status: done`; the task stays `in-progress` until every subtask is done.
- **Task completion.** Any contributor sets `Status: done`, moves the dashboard row to Recently Completed, updates the catalog, then **surfaces queued follow-ups** (`.dev_flow/todos/` and plan backlogs triggered `after task_<this ID>`) as offers, not auto-runs.

### Multi-contributor tolerance

Own your Subtask block and your tagged entries; append to shared sections, never rewrite others'; targeted edits over rewrites; re-read a shared file immediately before writing it; logs append-only, newest first; indexes are regenerable; no locks and no time-based takeover — continue a stale line of work by adding a new block that references the original. Full rules: [status phase → Collaboration Model](phases/status.md#collaboration-model-read-first).

### Hygiene

Salience-ordered compaction (`noise`/`superseded` first, `pin` of active tasks retained); Shared Activity Log ≤ 10 entries and per-subtask Activity ≈ 10 (overflow → `session_history/`); a task file past ~300 lines triggers an archive cycle; the dashboard stays under ~80 lines with the latest 5 completed; no logs, diffs, or narratives in context files — reference `.dev_flow/cache/` or the `/tmp` workspace instead, and never link a `/tmp` path from a doc or task file. Detail: [status phase → Context Hygiene](phases/status.md#context-hygiene).

## Document Status Vocabulary

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

When a concept or specification undergoes a **breaking change** (incompatible contract changes, removed entities, fundamentally different approach), create a new version instead of editing in place:

1. Set old document to `Status: deprecated` with `Deprecated-reason: Replaced by [C_XXX_v2](./path)`
2. Create new document with version suffix: `C_XXX_v2`, `SP_XXX_v2`
3. Update all `Depends on` / `Used by` references in dependent documents
4. Both versions may coexist while dependents migrate

For **non-breaking changes** (adding fields, extending contracts, fixing descriptions), edit the existing document in place and update the Changelog (if maintained — see [Docs Scaling](references/docs-scaling.md)).

## When Modifying Existing Functionality

Scale the ceremony to the change class first (see [do phase → Change Classes](phases/do.md#change-classes)): a **trivial** change takes the short route, a **standard** change starts at the spec, an **architectural** change runs the full pipeline below, an **internal refactor** (contracts unchanged) takes the plan-only [Refactoring Protocol](phases/plan.md#refactoring-protocol).

1. Update the **concept** — what changed in the idea or architecture?
2. Update the **specification** — what data structures or contracts changed?
3. Update the **implementation plan** — mark completed, add new tasks. Then stop at the **design sign-off** ([Developer Checkpoints](#developer-checkpoints)) before touching code.
4. Update the **code** — implement according to the updated spec.
5. Run **functional tests** — unit + mock tests covering the changed code (if test suite exists).
6. Run **pre-commit review** — subagent with clean context reviews the changes.
7. Run **verification** — regression, integration, and/or live tests; or provide manual verification steps. If issues found → fix → re-run steps 5 (if tests exist), 6, 7 at the scope the fix earns, never narrowing for a `must` or security finding (see [Verification Economy](references/verification-economy.md)).
8. **Commit sign-off** — present the changed files, the review verdict, and the intent verdict; commit only after the developer's explicit yes ([Developer Checkpoints](#developer-checkpoints)).

## Traceable Identifiers

Every section has a unique, immutable identifier:

| Document | Format | Example |
|----------|--------|---------|
| Concept | `C_XXX_NN_NN` | `C_ACS_01_01` |
| Specification | `SP_XXX_NN_NN` | `SP_ACS_01_01` |
| Plan | `PL_XXX` | `PL_ACS` |
| Epic | `E_XXX` | `E_ACM` |
| Design decision | `<DocID>_DEC_NN` | `C_ACS_DEC_01`, `SP_ACS_DEC_02`, `PL_ACS_DEC_01` |

The `_DEC_NN` form identifies a record in a document's **Design Decisions** section (see [Interview Mode](references/interview-mode.md)); the `_DEC` segment is reserved and never used as a numeric section number.

In code, reference these as comments: `# [C_ACS_03_01] PermissionInterceptor`

## File Organization

All documents live in `docs/`, follow [Documentation Formatting](references/formatting.md) (one paragraph = one logical line, no hard wraps inside sentences) and the `documentation` register of [Output Styles](references/output-styles.md). One concept = one file set:

| File | Extension | Example |
|------|-----------|---------|
| Spike | `*.spike.md` | `access_control.spike.md` |
| Concept | `*.concept.md` | `access_control.concept.md` |
| Specification | `*.sp.md` | `access_control.sp.md` |
| Plan | `*.plan.md` | `access_control.plan.md` |
| Epic | `*.epic.md` | `access_management.epic.md` |
| Split-set child | `*.p<NN>.plan.md` / `*.<module>.sp.md` | `access_control.p02.plan.md`, `access_control.audio.sp.md` |
| Index | `_index.md` | `docs/_index.md` |
| Glossary | `_glossary.md` | `docs/_glossary.md` |
| Framework map | `_framework.md` | `docs/_framework.md` |
| Adoption notes | `ext_adoption/*.md` | `docs/ext_adoption/other_repo.md` |

- **Umbrella sets.** A document split under [Docs Scaling](references/docs-scaling.md) becomes an umbrella (the original file name, the canonical entry point) plus child files; `_index.md` lists only umbrellas; no split renames or renumbers a traceable ID.
- **Spike** — optional pre-concept investigation from [research](phases/research.md); passes no gate. **Epic** — optional grouping for a feature spanning 3+ closely related concepts; template [templates/epic.md](templates/epic.md).
- **`_index.md`** — maintained once `docs/` exceeds 5 documents; a router (one-line annotation per file + ID-prefix → file map), never a section catalogue ([Docs Scaling](references/docs-scaling.md)).
- **`_glossary.md`** — canonical domain vocabulary, created lazily, and **loaded alongside `_index.md` whenever present**, independent of the 5-doc threshold ([Glossary](references/glossary.md)).
- **`_framework.md`** — the living architectural map, maintained only by onboard and `audit code`, **loaded alongside `_index.md` on code-touch phases**; it links down to rules/skills and inlines none of their detail ([Code Audit](references/code-audit.md)).
- **`ext_adoption/`** — advisory analyses from [`adopt`](references/repo-adoption.md): no traceable ID, no gate, not a backlog; consulted by the concept phase's Reuse Check; excluded from the `audit docs` integrity checks.
- **Index format.** Machine-read catalogues (`.dev_flow/rules/`, `skills/`, `roles/`, `cache/`) use `_index.yaml`; human-browsed ones (`docs/`, `.dev_flow/tasks/`, `todos/`) use `_index.md`. Apply the split to any new collection.

## Project Rules

When `.dev_flow/rules/` exists, all new code MUST comply ([Project Knowledge Is Binding](#project-knowledge-is-binding)). Rules are extracted during onboard and updated during implement/fix/review. Severity: **must** (blocks review) · **should** (warning) · **prefer** (advisory); rules apply to new code only.

```
.dev_flow/rules/
├── _index.yaml         # Derived router — regenerated from the files
├── naming.md · structure.md · architecture.md · error-handling.md · style.md · testing.md
```

Each rule is one h2 unit — directive and severity in the heading over a bounded body, immutable id; a category above `digest_min` opens with a `## Contents` digest; an optional `applies_to` selector is what the knowledge gate matches. A skill forms from an accumulated rule cluster at the agent's discretion. See [Knowledge Scaling](references/knowledge-scaling.md) and the [rule phase](phases/rule.md).

**Auto-discovery (self-learning).** At the `implement` and `fix` reflection checkpoints ([Experience Capture](references/experience-capture.md)), a coding pattern, constraint, or violated invariant with no matching rule is **written automatically, without asking**: default severity `should` (`prefer` for advisory); **never an auto-`must`** — a `must`-strength lesson or a contradiction with an existing rule goes to an independent clean-context review first ([Delegation for Focus](references/delegation.md)). The rule phase writes the file and the index entry; the commit message names the rule; the developer reviews every auto-written rule in the commit diff — the commit sign-off is untouched.

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

| File | Holds |
|------|-------|
| [Onboard](phases/onboard.md) | Takeover: reverse-engineer docs + rules bottom-up |
| [Research](phases/research.md) · [spike template](templates/spike.md) | Time-boxed investigation (alias `spike`) |
| [Concept](phases/concept.md) · [template](templates/concept.md) | Idea, architecture, mechanisms; checklist, Reuse Check, banned phrases |
| [Specification](phases/specification.md) · [template](templates/specification.md) | Structures, contracts, rules, verification criteria, rollback |
| [Plan](phases/plan.md) · [template](templates/plan.md) | Phases with `Verify:`; technology decisions; Refactoring Protocol |
| [Implement](phases/implement.md) | Code per plan under the knowledge gate |
| [Test](phases/testing.md) | Functional tests on changed code; conditional |
| [Review](phases/review.md) | Clean-context pre-commit review; check matrix |
| [Verify](phases/verify.md) | Regression, integration, live; conditional |
| [Propagate](phases/propagate.md) | Docs ↔ code drift; mechanical auto-fixes |
| [Fix](phases/fix.md) | Analyze → plan → fix → verify; diagnosis loop; rule detection |
| [Rule](phases/rule.md) · [Skill](phases/skill.md) | Manage `.dev_flow/rules/` and `.dev_flow/skills/`; index formats |
| [Status](phases/status.md) · templates [task_context](templates/task_context.md), [active_context](templates/active_context.md), [tasks_index](templates/tasks_index.md) | Context protocol, regeneration, salience markers, hygiene, archive |
| [Audit](phases/audit.md) | `.dev_flow/` + `docs/` revision; opt-in `code` scope |
| [Ask](phases/ask.md) · [Todo](phases/todo.md) · [todo_index template](templates/todo_index.md) | Read-only Q&A · future work with a return trigger |
| [Subtask](phases/subtask.md) | Delegate a secondary task to a full dev-flow participant |
| [Do](phases/do.md) | Freeform routing; change classes |
| [Interview Mode](references/interview-mode.md) | Design forks as marked options; open decisions with triggers |
| [Upstream Escalation](references/escalation.md) | Wrong doc → fix the doc, not the code |
| [Delegation for Focus](references/delegation.md) | Noisy work to a subagent; specialist routing |
| [Experience Capture](references/experience-capture.md) | Transition Checkpoint; harvest; trailer; context pressure |
| [Impact Walk](references/impact.md) | Blast radius: docs, code bindings, active tasks |
| [Task Intent](references/task-intent.md) | Goal / target / expected result at intake |
| [Ticket Tracker Integration](references/ticket-tracker.md) | Ticket-tied tasks; tracker skill owns conventions |
| [Consequence Forecasting](references/consequence-forecasting.md) | Forecast at altitude; YAGNI gate: build / seam / drop+record |
| [Code Reuse](references/code-reuse.md) | Search before creating; YAGNI-gated reuse |
| [Procedural Skills](references/procedural-skills.md) | Procedural memory; boundary + freshness; promotion |
| [Application Enforcement](references/application-enforcement.md) | Per-burst Pre-Action Marker; enforcement tiers |
| [Evidence Discipline](references/evidence-discipline.md) | Evidence states; ledger; report ceiling; `unobserved` |
| [Resource Cache](references/cache.md) | `.dev_flow/cache/`; `/tmp` workspace; working memory (L1) |
| [Roles](references/roles.md) | Base vs project-overlay subagent roles; `inherits:` |
| [Glossary](references/glossary.md) | `docs/_glossary.md` — canonical vocabulary |
| [Documentation Formatting](references/formatting.md) | Formatting conventions for generated doc files |
| [Knowledge Scaling](references/knowledge-scaling.md) | Rules/skills at scale: unit form, selectors, digests, relevant set |
| [Docs Scaling](references/docs-scaling.md) | Grep-first reading, Contents, history, splits; **single source of thresholds** |
| [Output Styles](references/output-styles.md) | Documentation vs chat registers; `.dev_flow/output_styles.md` profiles |
| [Review Convergence](references/review-convergence.md) | Criticality → materiality; `contested` todo; delta rounds |
| [Verification Economy](references/verification-economy.md) | Entry conditions: Signal, Invalidator, ConfirmDiscriminator |
| [Design Compliance](references/design-compliance.md) | UI vs design source of truth, in verify |
| [Code Audit](references/code-audit.md) | `audit code`: lenses, walk, antipatterns, playbook |
| [External Repo Adoption](references/repo-adoption.md) | `adopt`: clone, analysis, advisory adoption document |
| [SOLID reference](references/solid-architecture.md) | Generic architecture guidance — project rules override it |
| [Epic template](templates/epic.md) · [End-to-end example](examples/rate-limiter.md) | Grouping for 3+ concepts · full pipeline walk-through |

## Delegation for Focus (Context Isolation)

During implement / fix / verify hand noisy secondary work — test output, build logs, screenshots, wide searches, reproduction traces — to a subagent and keep the *conclusion, not the dump*; pick the model by the task's nature, never a hardcoded name. The core-vs-delegate line, scenarios, and model guidance: [Delegation for Focus](references/delegation.md). For a whole secondary *task*, use the [subtask phase](phases/subtask.md): the subagent is a full dev-flow participant with delegated rights — it assembles its own context, joins the task file as a contributor, persists skills/cache per their protocols, escalates real decisions to its initiator, and returns a full report; commits stay with the main context.

## Subagent Roles (AI-DSL)

Each phase has a **base role** for subagent execution (implementer, tester, reviewer, …); a project adds **overlays** under `.dev_flow/roles/` via `inherits:`. Catalogue and how to find, reuse, or create roles: [Roles](references/roles.md). A project may also grow **specialist focus helpers** — read-only roles owning a recurring noisy step (wide code search, log triage, screenshot analysis) — created when the pattern recurs; routing, role-local memory and promotion: [Delegation routing reflex](references/delegation.md#named-specialists-and-the-routing-reflex).
