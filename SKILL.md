---
name: dev-flow
description: >
  Feature development workflow following concept-spec-plan-implementation pipeline
  with traceable IDs. Use when: creating concept/spec/plan files, planning new features,
  modifying existing functionality, propagating changes across concept-spec-plan-code,
  reviewing documentation pipeline, assessing impact of architectural changes,
  asking questions about the codebase or feasibility of changes (read-only),
  researching unknowns before committing to a design (spike),
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

**Two-stage testing:**
- **Test** — functional tests only (unit + mock) covering the changed code.
- **Verify** — regression, integration, and live testing (end-to-end flows, app/service launch). If Verify finds issues → fix code → re-run Test (if exists) → re-run Review → re-run Verify.

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
| — | `/dev-flow rule <request>` | Add, edit, or remove coding rules (freeform) | Updated `.dev_flow/rules/` |
| — | `/dev-flow skill <request>` | Find, add, update, or remove project knowledge skills | Updated `.dev_flow/skills/` |
| — | `/dev-flow status` | Show current state, resume previous session | Status summary |
| — | `/dev-flow audit [scope] [--dry-run]` | Revise `.dev_flow/` and `docs/` — reconcile task state with reality, trim context, compact closed tasks, groom rules/skills/cache, check docs integrity (index/statuses/refs); opt-in `code` scope audits the whole codebase → refactoring plan | Audit report + cleaned context (or, for `code`, a refactoring plan) |
| — | `/dev-flow ask <question>` | Read-only Q&A about code or feasibility — no changes | Answer + optional next-step suggestion |
| — | `/dev-flow todo <description>` | Capture future work — find relevant docs, assess feasibility, file a planning record with a return trigger (does not build) | Plan backlog item or `.dev_flow/todos/` entry |
| — | `/dev-flow subtask <task>` | Delegate a secondary task to a subagent — a full dev-flow participant that assembles its own context, runs any phase (fix, test, research, etc.), and can converse with its initiator | Full subtask report |
| — | `/dev-flow do <request>` | Freeform routing — interpret intent and run the right phases | Phase output + updated context |

**Default command:** Any invocation of `/dev-flow <text>` that does not match a recognized phase keyword routes automatically to `do`. For example, `/dev-flow add Skip button to login` is equivalent to `/dev-flow do add Skip button to login`.

**Ask command:** Use `/dev-flow ask <question>` for read-only questions about the codebase or feasibility of changes. No files are modified, no context is updated.

**Research command:** Use `/dev-flow research <topic>` (alias: `spike`) when a concept/spec/plan cannot be confidently authored — unfamiliar domain, unverified library capability, unknown solution space — or to close an open Design Decision waiting on facts. Time-boxed and cost-gated; spikes pass through no validation gates. Produces `docs/*.spike.md` and persists durable findings to `.dev_flow/skills/`. See [research phase](phases/research.md).

**Todo command:** Use `/dev-flow todo <description>` to capture work to do later. It finds the documentation the work would touch, assesses its execution prospect (feasibility + scope), and files a single planning record with a return trigger — into an owning plan's backlog if one exists, otherwise into `.dev_flow/todos/`. Two flavors: a **deferred** speculative idea (trigger = date/event, may be dropped), or a **queued follow-up** — a fix noticed *during* the current task that must wait until it finishes because the contexts overlap (trigger = `after task_<ID>`, never dropped; a parallel `subtask` can't cover it since contexts aren't disjoint). A queued follow-up is surfaced automatically when its task completes (offered, not auto-run). `todo` **infers** the flavor, urgency, and trigger from the state of the relevant plans/tasks — not from your wording (the description may carry no timing words) — and may even recommend doing the work now instead of deferring. It builds nothing and passes through no gates; a later `do`/`plan` run executes the record. An **agent can also file a `todo` itself** — when it spots an out-of-scope, deferrable defect mid-work it spawns a cheap subagent running the todo flow (or files a trivial one inline), so the finding is captured without derailing its current task. Completes the routing triad: `ask` analyzes and writes nothing, `todo` analyzes and files for later, `do` analyzes and acts now. See [todo phase](phases/todo.md).

**Resource cache (not a phase):** `.dev_flow/cache/` is the durable, indexed store for expensive-to-reacquire resources (Figma exports, downloaded documents, baseline screenshots). Every phase checks its `_index.yaml` before an expensive re-fetch and saves new fetches back; anything linked from docs or task files lives here, never in `/tmp`. Transient artifacts go to the project workspace `/tmp/{project-slug}/` with timestamped names. Freeform cache requests ("збережи цей макет", "find the cached RFC") route through `do` and are applied inline. See [Resource Cache](references/cache.md).

**Audit command:** Use `/dev-flow audit` for the periodic revision of `.dev_flow/` and the `docs/` documentation set — it reconciles every task's recorded state against reality (linked docs + git), trims the dashboard, compacts/reflects on closed tasks (archiving noise, harvesting lessons into rules/skills), grooms the `rules/`, `skills/`, and `cache/` catalogues, and reconciles `docs/` integrity (index, statuses, cross-references, glossary, drift). Where [status](phases/status.md) *reports* drift, `audit` *resolves* it. Supports `--dry-run` (report only) and a `scope` (`context` / `tasks` / `rules` / `skills` / `cache` / `docs` / `all`). A separate, opt-in **`code`** scope (`/dev-flow audit code <free-form intent>`, excluded from `all`) extends the same idea to source code: it audits the whole codebase through parallel lenses (architecture / SOLID / DRY / security / …) and emits a prioritized refactoring **plan** + run report — written to `.dev_flow/audit/` with a timestamp, not into `docs/` — plus a `docs/_framework.md` map update — read-only, stopping at the Plan→Code gate and handing off to the standard pipeline (it never commits). See [Code Audit](references/code-audit.md).

**Phase 0 (Onboard):** Optional. Run once when adopting dev-flow for an existing project. Analyzes code bottom-up (utilities first), generates concepts, specs, and plans. Supports `--resume` for continuation across sessions. See [onboard phase](phases/onboard.md).

**Phase 5 (Test) activation condition:** The project must have an existing test suite AND defined rules for running tests (e.g., `pytest`, `jest`, CI pipeline). If neither exists — skip directly to Review. Runs only **functional tests** (unit + mock) covering the changed code. Does NOT run integration or live tests at this stage.

**Phase 6 (Review):** Pre-commit code review is performed by a subagent with a clean context to ensure an unbiased perspective. This is mandatory before any commit.

**Phase 7 (Verify):** Regression, integration, and live testing — runs after Review passes. Includes launching the app/service and verifying end-to-end scenarios. If issues are found during Verify, the cycle repeats: fix → Test → Review → Verify. Ask user permission before creating new integration/live scenarios. If no automated verification exists — provide manual verification steps for the user.

### Quick Start

Before writing or changing any code, ask yourself:
1. Does the concept describe what I'm building? If not — update the concept first.
2. Does the specification define the data structures and contracts? If not — update the spec first.
3. Only then — create/update the implementation plan and write code.

## Project Knowledge Is Binding

Loading `.dev_flow/rules/` and `.dev_flow/skills/` is a mandatory gate at the start of every code/test/doc phase; applying them is not optional.

- **Rules (`.dev_flow/rules/`)** — load rules for the area you touch; new code MUST comply:
  - `must` — violation **blocks**: fix it or surface the conflict and stop.
  - `should` — violation needs a recorded justification.
  - `prefer` — follow unless there is a stated local reason not to.
- **Skills (`.dev_flow/skills/`)** — check `_index.yaml` and load matching skills BEFORE external research; a skill's "Pitfalls"/"Usage in This Project" override generic knowledge. Skills are distilled **procedural** memory: a *current* matching skill outranks the model's general prior, while a *stale* one (tool/framework version or context drifted) is demoted and must be re-grounded before it can override fresh research. See [Procedural Skills](references/procedural-skills.md).
- **Precedence** — project rules override generic guidance (incl. the [SOLID reference](references/solid-architecture.md)); project skills override generic technology knowledge. On conflict: project wins, or surface it.

If the directory is absent, the gate is a no-op. Each relevant phase restates this as its "Skill check" / "Rule check" — gates, not reminders. The gate is also **re-triggered at the moment of action** — the relevant set is re-surfaced per action burst beside the work (a pointer-only Pre-Action Marker), not only once at phase start, because loaded ≠ applied in a long session. See [Application Enforcement](references/application-enforcement.md).

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
- Integration tests pass (if applicable)
- Live tests pass or manual verification completed (app/service launched, scenario checked)
- If Verify finds issues → fix code → re-run Test (if exists) → re-run Review → re-run Verify
- If a failure traces to the spec/plan itself (not the code) — do not bend the code: escalate upstream first (see [Upstream Escalation](references/escalation.md))
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

When a concept or specification undergoes a **breaking change** (incompatible contract changes, removed entities, fundamentally different approach), create a new version instead of editing in place:

1. Set old document to `Status: deprecated` with `Deprecated-reason: Replaced by [C_XXX_v2](./path)`
2. Create new document with version suffix: `C_XXX_v2`, `SP_XXX_v2`
3. Update all `Depends on` / `Used by` references in dependent documents
4. Both versions may coexist while dependents migrate

For **non-breaking changes** (adding fields, extending contracts, fixing descriptions), edit the existing document in place and update the Changelog.

## When Modifying Existing Functionality

Scale the ceremony to the change class first (see [do phase → Change Classes](phases/do.md#change-classes)): a **trivial** change takes the short route, a **standard** change starts at the spec, an **architectural** change runs the full pipeline below.

1. Update the **concept** — what changed in the idea or architecture?
2. Update the **specification** — what data structures or contracts changed?
3. Update the **implementation plan** — mark completed, add new tasks.
4. Update the **code** — implement according to the updated spec.
5. Run **functional tests** — unit + mock tests covering the changed code (if test suite exists).
6. Run **pre-commit review** — subagent with clean context reviews the changes.
7. Run **verification** — regression, integration, and/or live tests; or provide manual verification steps. If issues found → fix → re-run steps 5 (if tests exist), 6, 7.
8. **Ask for commit approval** — present changes for review before committing.

## Traceable Identifiers

Every section has a unique, immutable identifier:

| Document | Format | Example |
|----------|--------|---------|
| Concept | `C_XXX_NN_NN` | `C_ACS_01_01` |
| Specification | `SP_XXX_NN_NN` | `SP_ACS_01_01` |
| Plan | `PL_XXX` | `PL_ACS` |
| Design decision | `<DocID>_DEC_NN` | `C_ACS_DEC_01`, `SP_ACS_DEC_02`, `PL_ACS_DEC_01` |

The `_DEC_NN` form identifies a record in a document's **Design Decisions** section (see [Interview Mode](references/interview-mode.md)); the `_DEC` segment is reserved and never used as a numeric section number.

In code, reference these as comments: `# [C_ACS_03_01] PermissionInterceptor`

## File Organization

All documents live in `docs/` directories. One concept = one file set:

| File | Extension | Example |
|------|-----------|---------|
| Spike | `*.spike.md` | `access_control.spike.md` |
| Concept | `*.concept.md` | `access_control.concept.md` |
| Specification | `*.sp.md` | `access_control.sp.md` |
| Plan | `*.plan.md` | `access_control.plan.md` |
| Epic | `*.epic.md` | `access_management.epic.md` |
| Index | `_index.md` | `docs/_index.md` |
| Glossary | `_glossary.md` | `docs/_glossary.md` |
| Framework map | `_framework.md` | `docs/_framework.md` |

**Spike** is an optional pre-concept investigation artifact, produced by the [research phase](phases/research.md) (`/dev-flow research`). Use it when the problem domain is unclear and you need to explore approaches before committing to a concept. Spikes do not pass through the pipeline gates.

**Epic** is an optional grouping document for a feature spanning 3+ closely related concepts (see onboard Step 7 and [concept phase](phases/concept.md)). Template: [templates/epic.md](templates/epic.md).

When `docs/` has more than 5 documents, maintain an `_index.md` catalog.

**Index format convention:** machine-read catalogues (`.dev_flow/rules/`, `.dev_flow/skills/`, `.dev_flow/roles/`, `.dev_flow/cache/`) use `_index.yaml` — structured entries agents match against. Human-browsed catalogues (`docs/`, `.dev_flow/tasks/`) use `_index.md`. Apply the same split to any new collection.

`docs/_glossary.md` is the project's canonical domain vocabulary (term → definition + aliases to avoid). It is created lazily (during onboard, or when the first cross-concept term is resolved) and, whenever present, is **loaded into context alongside `_index.md`** (independent of the >5-doc threshold that gates `_index.md`) — so authoring uses one canonical term per concept. See [Glossary](references/glossary.md).

`docs/_framework.md` is the project's living **architectural map** (core abstractions · layers · extension points · shared utilities · conventions) — an overview that links *down* to the `.dev_flow/rules/` and `.dev_flow/skills/` holding the enforceable detail; it inlines none of it. It is created/maintained by onboard and by the [`audit code` scope](phases/audit.md#step-9--code-scope-the-whole-codebase-audit) (never hand-authored as part of a feature), and, whenever present, is **loaded into context alongside `_index.md`** on code-touch phases — so implementation and review see the architecture spine. See [Code Audit](references/code-audit.md).

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

dev-flow uses a **collaborative per-task context model** so multiple AI agents can work on the same project in parallel. A task file is a **shared document with multiple contributors**; each contributor owns the parts they add and never rewrites parts authored by others.

**Three memory tiers.** This durable per-task context is **L2** (`.dev_flow/`, survives both compact and restart). Above it sit **L0** live context (the transcript, lost on compact) and **L1** session scratch (survives compact, not durable): the [session working memory](references/cache.md#session-working-memory-l1) of distilled notes/params/reminders (session-keyed, lost on restart) plus the data cache (the project `/tmp` workspace, cleared on reboot). [Experience Capture](references/experience-capture.md) checkpoints promote L1→L2, and [salience markers](phases/status.md#salience-markers) decide what survives a compaction — together keeping durable task state complete so a fresh session can resume from files.

**Use working memory as you work — it is not optional infrastructure.** Set a parameter when you fix a working value (current segment / phase / target), jot a note when you learn a non-obvious fact you'd hate to re-derive, drop a reminder when you defer an action; and re-read the whole area first whenever you've lost the thread (after a compaction, on a subtask switch). The transcript is exactly what a compaction takes from you — these few lines are the cheap insurance that survives it and keep a long, multi-topic task from drifting. Write triggers and rationale: [Session Working Memory](references/cache.md#session-working-memory-l1).

```
.dev_flow/
├── active_context.md          # Dashboard — table of active tasks + recently completed
├── cache/                     # Durable resources (Figma exports, downloads, baselines) + _index.yaml
├── tasks/
│   ├── _index.md              # Catalog of task files (conventions + active/recent lists)
│   ├── task_<ID>.md           # Per-task shared context — multiple contributors
│   └── ...
├── todos/                     # Deferred future work captured by `todo` (homeless ideas) + _index.md
└── session_history/           # Archived completed tasks
```

**Source of truth = the task files.** The two index files (`active_context.md`, `tasks/_index.md`) are derived views — any contributor can rebuild them from `tasks/*.md` when they drift.

**Task file naming:**
- Tied to a traceable doc → `task_C_AUTH.md`, `task_PL_RATE_LIMITER.md`.
- Otherwise → `task_YYYYMMDD_HHMMSS_<slug>.md` (slug = 1–3-word kebab-case).

**Each task file contains:**
- Header: Task ID, Created, Last updated, Status, **Contributors** (list of agent IDs).
- **Current Work Item** — shared metadata (Document, Phase, Traceable ID).
- **Description** — shared, additive paragraphs signed by contributor.
- **Subtasks** — one block per contributor; each block has Author/Status/Goal/ Progress checklist/Activity. The block's author is its sole editor.
- **Coordination Notes** — append-only conversation between contributors.
- **Blocking Issues** — each tagged with the reporter who raised it.
- **Relevant Context** — table; each row tagged with the contributor who added it.
- **Shared Activity Log** — task-level events (subtask added, contributor joined, status changed). Append-only, newest first.

**`active_context.md`** is a lightweight dashboard listing active tasks with phase, status, contributors, and last-updated, plus a thin **Deferred (todos)** pointer (counts + any todo bound to an already-closed plan/task, so a deferral filed by `todo` stays visible at the entry point). **No per-task details live here.**

**Templates:**
- [templates/task_context.md](templates/task_context.md) — task file
- [templates/active_context.md](templates/active_context.md) — dashboard
- [templates/tasks_index.md](templates/tasks_index.md) — `tasks/_index.md`

### Rules for all phases

- **Project-knowledge gate (first).** Read `.dev_flow/rules/_index.yaml` and `.dev_flow/skills/_index.yaml`, load what's relevant to the area you touch, and obey it — see [Project Knowledge Is Binding].
- **Resource gate.** Before an expensive external fetch (Figma export, web document), check `.dev_flow/cache/_index.yaml` and reuse a cached copy (no-op while the directory is absent); an entry past its `valid_until` gets a cheap currency check (ETag, Figma version) before any re-fetch. After an expensive fetch, save the artifact back — a focus-delegated helper *stages* it in the workspace and reports (a task-delegated subagent writes the cache itself per the protocol — see [subtask phase](phases/subtask.md)). A resource fetched from the open internet is saved with `trust: public` and goes through the safety check first. Transient artifacts follow the workspace discipline — `/tmp/{project-slug}/` with timestamped names. See [Resource Cache](references/cache.md).
- At the **start** of any phase:
  - **Continuation** — locate the task via `active_context.md`. If your own Subtask block exists, resume it. If you have no block in this task yet, add a new `### Subtask:` block (you become a Contributor).
  - **New task** — create `tasks/task_<ID>.md` from the template with one Subtask block (yours). Add a row to `active_context.md` and `tasks/_index.md` via a **targeted Edit**.
- After **completing a step**: in your own Subtask block, check off the Progress item, set the next item, append to your Activity bullet list. Refresh the task header's `Last updated`.
- At a **phase boundary**: targeted Edit on the dashboard row (Phase / Status / Contributors / Updated).
- At a **transition** (phase/subtask boundary, task switch) run a **Transition Checkpoint** — distill the closing segment into a `{s:pin}` summary, demote its raw entries, propose any durable lesson, and promote durable working-memory parts to the task file. See [Experience Capture](references/experience-capture.md).
- **Spotted-defect reflex (while working).** If you notice a defect or problem that is **not part of your current task** and whose fix is **deferrable to a later session**, do not derail into it and do not lose it — file an **agent-initiated `todo`**: spawn a **cheap subagent** with the problem description (it runs the [todo phase](phases/todo.md) and reports what it filed), or for a trivial finding needing no analysis, file the register line inline. In-scope or urgent problems are not deferred — fix them or escalate.
- When you **finish your subtask**: set your Subtask's `Status: done`. The task itself stays `in-progress` until all subtasks are done.
- On **task completion** (all subtasks done): any contributor may set the task's overall `Status: done`, move its row from "Active" to "Recently Completed" in the dashboard, and update the catalog. Then **surface queued follow-ups**: scan `.dev_flow/todos/` (and plan backlogs) for `queued` records triggered `after task_<this ID>` and offer to run each next (`/dev-flow do …`) — a suggestion, not an auto-run; the executed work still passes its own gates and commit approval. This is how a fix deferred *because its context overlapped this task* gets picked up the moment the conflict is gone. See [todo phase](phases/todo.md).

### Multi-contributor tolerance

Each contributor edits only their own subtask block and their own tagged entries in shared sections. The dashboard and catalog are updated with **targeted edits** (Edit, single row), never full rewrites.

1. **Own your block, leave others':** each Subtask block has an `Author` tag. Only the author edits it. Reading other contributors' blocks is encouraged — they are shared context.
2. **Additive shared sections:** Description paragraphs, Coordination Notes, Blocking Issues, Relevant Context rows, and Shared Activity Log entries are all tagged with their author. Append your own; do not rewrite others'.
3. **Targeted edits over rewrites:** when updating the dashboard, the catalog, or the task header (Last updated / Contributors / Status), use `Edit` on the specific field/row rather than rewriting the whole file.
4. **Read-before-write:** immediately before writing to a shared file, re-read it. If another contributor's edit has landed, re-apply your edit on the latest content.
5. **Append-only logs:** Coordination Notes and Shared Activity Log entries are added newest-first; never rewrite an existing entry.
6. **Indexes are regenerable:** if `active_context.md` or `_index.md` drift, any contributor may rebuild them from `tasks/*.md` headers.
7. **No exclusive locks, no time-based takeover.** If a contributor's subtask is stale and another wants to continue that line of work, they **add a new subtask block** referencing the original — they do not edit the original block. Coordinate explicitly via Coordination Notes.

### Hygiene

- **Salience-ordered compaction:** when a cap forces eviction, drop `noise`/`superseded` entries first and retain `pin` entries of active tasks, falling back to age-order for the `normal` remainder — markers are task-scoped and go inert on task close (see [Salience Markers](phases/status.md#salience-markers)).
- **Shared Activity Log** per task: keep at most **10 entries** (newest first). Archive overflow to `.dev_flow/session_history/session_YYYY-MM-DD.md`.
- **Per-subtask Activity:** also capped at ~10 entries per block.
- **Task file size:** if a task file exceeds ~300 lines, trigger an archive cycle — archive completed (done) subtask blocks first.
- **Dashboard size:** `active_context.md` stays under ~80 lines. "Recently completed" keeps the latest 5 — older entries go to `session_history/`.
- **No large blobs** in any context file: never store logs, diffs, or verbose narratives — reference a file instead: durable resources from `.dev_flow/cache/`, transient output from the project workspace in `/tmp` (see [Resource Cache](references/cache.md)). Never link a `/tmp` path from a doc or task file.

See [status phase](phases/status.md) for the full read/write protocol, regeneration procedure, and archive flow.

## Project Rules

When `.dev_flow/rules/` exists, all new code MUST comply (binding — see [Project Knowledge Is Binding]). Rules are extracted during onboard and updated during implement/review phases.

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

Severity levels: **must** (blocks review) | **should** (warning) | **prefer** (advisory). Rules apply to new code only — no retroactive refactoring required.

**Auto-discovery of new rules (autonomous self-learning):** During the `implement` and `fix` reflection checkpoints (see [Experience Capture](references/experience-capture.md)), if the implementation plan or fix analysis reveals a coding pattern, constraint, or convention that is not yet captured in `.dev_flow/rules/` — harvest it and **write the rule automatically, without asking permission**. The gate is *structural, not a self-score*. Specifically:
- If a plan phase describes a new architectural constraint, naming convention, or error-handling pattern — create or update the corresponding rule file automatically.
- If a `fix` phase root-cause analysis identifies a violated invariant that has no matching rule — write the rule so the same class of bug is prevented in the future.
- Auto-written rules default to severity **should** (use **prefer** for advisory). **Never auto-write a `must`** — a `must` blocks future code; if the lesson is genuinely `must`-strength, or it contradicts an existing rule, route it to an **independent clean-context review** ([Delegation for Focus](references/delegation.md)) and write only if confirmed.
- The rule phase writes the rule, updates `.dev_flow/rules/_index.yaml` (under its category's `rules` list — see [rule phase → Index Format](phases/rule.md#index-format)), and the new rule is mentioned in the commit message. The developer reviews every auto-written rule in the **commit diff** — the [commit-approval gate](#git-workflow-integration) is untouched, so self-learning never bypasses developer oversight, it just moves it to commit time.

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
- [Research phase](phases/research.md) | [Spike template](templates/spike.md) *(on-demand — time-boxed investigation when knowledge is insufficient; alias `spike`)*
- [Concept phase](phases/concept.md) | [Template](templates/concept.md)
- [Specification phase](phases/specification.md) | [Template](templates/specification.md)
- [Plan phase](phases/plan.md) | [Template](templates/plan.md)
- [Interview Mode](references/interview-mode.md) *(cross-cutting sub-procedure of concept/spec/plan/fix — surface design forks to the developer instead of choosing silently)*
- [Upstream Escalation](references/escalation.md) *(cross-cutting sub-procedure of implement/test/review/verify/fix — when evidence shows the spec/plan/concept is wrong, fix the document, don't bend the code)*
- [Delegation for Focus](references/delegation.md) *(cross-cutting sub-procedure of implement/fix/verify — delegate noisy work to a subagent, keep only the conclusion)*
- [Experience Capture](references/experience-capture.md) *(cross-cutting sub-procedure of all phases — Transition Checkpoint: distill a closing segment into a pinned summary, demote raw turns, propose durable lessons; factual Response Trailer; tiered context-pressure response)*
- [Impact Walk](references/impact.md) *(cross-cutting sub-procedure of ask/do/propagate/review — blast radius of a change: docs, code bindings, active tasks)*
- [Consequence Forecasting](references/consequence-forecasting.md) *(cross-cutting sub-procedure of concept/spec/plan/implement/fix — forecast future needs at the phase's altitude, then YAGNI-gate each: build now / seam+flag / drop+record; advisory, composes Impact Walk + Minimality, routes to Interview/Escalation)*
- [Procedural Skills](references/procedural-skills.md) *(cross-cutting refinement of the skill subsystem — skills are procedural memory with a mandatory applicability boundary + freshness stamp; a current skill outranks the prior, a stale one is re-grounded first; candidate→established promotion; audit curation)*
- [Application Enforcement](references/application-enforcement.md) *(cross-cutting sub-procedure of implement/fix/testing/verify — re-trigger the knowledge gate per action burst via a pointer-only Pre-Action Marker; conformance tripwire + advisory/interface-gate/sampled-verifier tiers; clean-context review realizes the verifier tier; self-attestation banned; runtime-conditional, degrades to advisory)*
- [Resource Cache](references/cache.md) *(cross-cutting — durable resource store `.dev_flow/cache/` with trust levels + `/tmp` workspace discipline; every phase checks it before expensive fetches)*
- [Roles](references/roles.md) *(base vs project-overlay subagent roles — reuse what exists, create new under .dev_flow/roles/ via inherits)*
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
- [Audit phase](phases/audit.md) *(full `.dev_flow/` + `docs/` revision — reconcile, trim, compact + reflect, groom rules/skills, check docs integrity; opt-in `code` scope = whole-codebase audit → refactoring plan)*
- [Code Audit](references/code-audit.md) *(the `audit code` scope's detail — lens registry + per-lens checklists, the bottom-up walk shared with onboard, SOLID/DRY heuristics, antipattern catalogue, refactoring playbook)*
- [Ask phase](phases/ask.md) *(read-only Q&A, no file changes)*
- [Todo phase](phases/todo.md) *(capture future work — find docs, assess feasibility, file a planning record)* | Template: [todo_index](templates/todo_index.md)
- [Subtask phase](phases/subtask.md) *(delegate secondary tasks to subagent)*
- [Do phase](phases/do.md) *(default fallback for freeform requests)*
- [End-to-end example](examples/rate-limiter.md)

## Delegation for Focus (Context Isolation)

The main agent's focus — the plan, the spec, and the task state it holds while working — is the scarce resource, and it erodes when noisy secondary work (test output, build logs, screenshots, wide searches, reproduction traces) floods the same context. The fix is a reflex during implement / fix / verify: hand that work to a subagent and keep only the *conclusion, not the dump*; pick the subagent's model by the task's nature, never by a hardcoded name. The full principle — the core-vs-delegate line, the scenarios table, and the model guidance — lives in **[Delegation for Focus](references/delegation.md)**.

For a whole secondary *task* (not a single noisy step), use the [subtask phase](phases/subtask.md): there the subagent is a **full dev-flow participant with delegated rights** — it assembles its own context from hints, joins the task file as a contributor, persists skills/cache per their owning protocols, escalates real decisions to its initiator, and returns a full report. Commits stay with the main context (the developer-approval chain).

## Subagent Roles (AI-DSL)

Each phase has a specialized **base role** for subagent execution (implementer, tester, reviewer, …), and a project adds its own **overlays** and specializations under `.dev_flow/roles/`, declared by naming the base role(s) in an `inherits:` field. The full catalogue of base roles by phase — and how to find, reuse, and create roles — lives in **[Roles](references/roles.md)**.

Beyond the per-phase roles, a project can grow **specialist focus helpers** — read-only roles that own a recurring noisy step (wide code search, log/trace triage, screenshot analysis). They are **not shipped base roles** (a generic log-analyst can't know your log format); the agent creates them per-project under `.dev_flow/roles/` when the pattern recurs, and the [Delegation routing reflex](references/delegation.md#named-specialists-and-the-routing-reflex) routes matching work to them by description. Each warm-starts from a role-local memory of distilled heuristics and promotes the broadly-useful ones to skills via [Experience Capture](references/experience-capture.md).
