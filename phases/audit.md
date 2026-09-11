# Phase: Audit — Full `.dev_flow/` + `docs/` Revision & Housekeeping

## Purpose

`.dev_flow/` accumulates drift the normal phases never catch (each touches only the slice it owns): stale dashboard rows, task headers lagging documents and git, verbose closed tasks, duplicate or stale rules/skills. Audit is the periodic **whole-directory sweep** that reconciles every task's recorded state against ground truth, trims the dashboard to what is actually active, compacts and reflects on closed work so its lessons survive while its noise is archived, grooms the `rules/`, `skills/`, and `cache/` catalogues — and reconciles the integrity of the `docs/` set itself (index, statuses, cross-references, orphans, freshness, glossary, duplicated value sets, docs↔code drift) through the `docs` scope. It is the write-heavy cousin of [status](status.md): `status` *reports* drift, `audit` *resolves* it.

A separate, opt-in **`code` scope** ([Step 9](#step-9--code-scope-the-whole-codebase-audit)) extends the same "reconcile the project to reality" idea to the *source code itself*: it audits the whole codebase through parallel lenses (architecture / SOLID / DRY / security / …), consolidates the findings, and emits a prioritized **refactoring plan** plus a run report — written under `.dev_flow/audit/` with a timestamped name (not into `docs/`) — plus a `docs/_framework.md` map update. Like every other scope it is **non-committing** — it stops at the Plan→Code gate and hands the plan off to the standard pipeline; it never edits source or commits.

Audit is also the **declared home of a corpus-wide comparison**. A sweep with no per-file signal belongs here or in [propagate](propagate.md), not inside an unrelated task — running the command **in the matching scope** is itself the entry condition (`audit-run`) — a `code`-scope run does not open a `docs` sweep. See [Verification Economy](../references/verification-economy.md).

Audit composes existing phases rather than reinventing them — it leans on the [status](status.md) regeneration procedure for the indexes, the [rule](rule.md) and [skill](skill.md) phases for catalogue edits, [propagate](propagate.md) when a reconciliation reveals that docs and code disagree, and (for the `code` scope) the [implement](implement.md)/[review](review.md)/[verify](verify.md) phases to *execute* the refactoring plan it produces.

## Command

```
/dev-flow audit [scope] [--dry-run]
```

- `scope` (optional) — limit the sweep to one area. One of: `context` (dashboard + task reconciliation + compaction), `tasks` (reconcile + compact task files only), `rules`, `skills`, `cache`, `docs` (documentation integrity — index · statuses · cross-refs · orphans · freshness · glossary · duplicated sets · drift), `code` (whole-codebase architecture/SOLID/DRY/security audit → refactoring plan), or `all` (default). `all` runs every workspace + `docs` scope below; **`code` is opt-in and excluded from `all`** — it is the heavy, periodic scope and is invoked explicitly.
- The `code` scope takes **free-form intent**, not flags: `/dev-flow audit code <description of intent>`. The `--dry-run` flag below applies to the workspace/`docs` scopes; `code` expresses "report only, no hand-off" in its intent text (e.g. "preview plan") — see [Step 9](#step-9--code-scope-the-whole-codebase-audit).
- `--dry-run` — produce the audit report only; make no changes on disk.

### Scope → steps

Each scope runs a defined subset of the [procedure](#procedure); `all` runs the workspace + `docs` steps in order (it does **not** run `code`). Skip steps outside the requested scope.

| Scope | Steps | Reconciles |
|-------|-------|------------|
| `context` | 1–4 | dashboard + task reconciliation + compaction |
| `tasks` | 1–3 | task files only (reconcile + compact) |
| `rules` | 6 | `.dev_flow/rules/` catalogue |
| `skills` | 7 | `.dev_flow/skills/` catalogue |
| `cache` | 7d | `.dev_flow/cache/` index ↔ disk |
| `docs` | 7a · 7b · 7c · 7e | `docs/` — glossary, open decisions/backlogs/todos/spikes, docs↔code drift, **index · statuses · cross-refs · orphans · freshness · duplicated sets · scaling** |
| `all` (default) | 1–8 | everything in the workspace **and** docs (**not** `code`) |
| `code` (opt-in, separate) | 9 | the **whole project codebase** — architecture/SOLID/DRY/security/… via lens fan-out → prioritized refactoring plan + run report (timestamped, in `.dev_flow/audit/`) + `docs/_framework.md` update (read-only; hands off to the pipeline, never commits) |

### Examples

```
/dev-flow audit                  # full sweep — workspace + docs (not code)
/dev-flow audit --dry-run        # show what would change, touch nothing
/dev-flow audit context          # only reconcile/trim context, skip rules & skills
/dev-flow audit tasks            # only reconcile + compact the task files
/dev-flow audit rules            # only groom .dev_flow/rules/
/dev-flow audit skills           # only groom .dev_flow/skills/
/dev-flow audit cache            # only groom .dev_flow/cache/
/dev-flow audit docs             # only check docs/ integrity (index, statuses, refs, orphans, freshness, glossary, duplicated sets, drift)
/dev-flow audit code             # whole-codebase audit → refactoring plan (all base lenses)
/dev-flow audit code модуль auth, фокус на architecture та DRY   # scoped + lens-focused (free-form)
/dev-flow audit code focus security                              # security-lens sweep
/dev-flow audit code лише зміни з останнього релізу, прев'ю-план  # incremental + preview-only (no hand-off)
```

## When to Run

- Periodically (e.g. end of a milestone, before a release, on a cadence).
- When `/dev-flow status` reports drift, stale tasks, or missing/incorrect indexes.
- When `active_context.md` has grown past its hygiene cap (~80 lines — caps defined in [status phase → Context Hygiene](status.md#context-hygiene)) or carries per-task narratives that belong in the task files.
- Before a new contributor or a fresh session needs a trustworthy picture.
- After a burst of commits, when several tasks closed but their headers were never moved to `done`.
- After a batch of document edits — when a doc's `Status` lags reality (a plan whose phases are all `[DONE]` but still `in-progress`), `docs/_index.md` has drifted from the files, or `Depends on` / `Used by` links broke.
- **(`code` scope)** Periodically on a longer cadence than the workspace sweep — at a milestone, before a release, or after a burst of changes that each passed `review` locally but may have drifted the system (layer erosion, accreting duplication, an abstraction that never got extracted). The codebase must already be `onboard`-ed (documented architecture/rules) for the full conformance lenses; without them the run drops to **reduced mode** (the conformance lenses — `standards`/`architecture`/`specifications` — are dropped; SOLID/DRY/antipatterns/security/correctness/performance/tests still run) and recommends `onboard` first.

## Guiding Principles

These shape every step below — read them first, they are the difference between a clean-up and a data-loss incident.

1. **Reconcile, don't fabricate.** A task's real state is determined by evidence: the status of its linked concept/spec/plan in `docs/`, and the git history for its traceable ID. Change a recorded state only when evidence supports it. When evidence is missing or contradictory, **flag it in the report and leave the file unchanged** — never invent a status to make the dashboard look tidy.
2. **Non-destructive by default.** History is moved, never deleted. Overflow logs, completed subtask blocks, and closed task files go to `.dev_flow/session_history/`, not to `/dev/null`. Indexes are regenerable, so they may be rewritten freely — with one exception: **`cache/_index.yaml` is data, not a derived view** (its `source` metadata exists nowhere else), so it is only ever reconciled entry by entry, never regenerated (see [Resource Cache](../references/cache.md)). Task files and history are not regenerable either — they are only ever appended to or relocated intact.
3. **Source-of-truth order.** Task files win over the dashboard and catalog (those are derived). For task *reality*, `docs/` document status and git win over the task header. For rules/skills, the files on disk win over their `_index.yaml`. For the cache, disk decides *existence* (a missing file orphans its entry) but the index owns *metadata* (`source`/`summary` cannot be rebuilt from disk) — flag mismatches, never drop or regenerate entries wholesale (Step 7d).
4. **Safe under contention.** Audit may run while other contributors hold open subtasks. It may fully rewrite *derived* indexes and *closed* task files, but it must never rewrite a live contributor's Subtask block or their tagged entries in shared sections — those are reconciled only with targeted edits, and only the header/status fields. Re-read immediately before each write.
5. **Apply the safe, propose the judgement.** Derived/reversible changes (index regen, dashboard trim, archival of done work, header reconciliation backed by git) are applied directly and reported. Judgement calls (merging two rules, deleting a skill, removing a cached resource) are **proposed for confirmation** — this mirrors the rule/skill phases, where removal requires explicit intent. Reflection-harvested lessons follow their own gate instead: written **automatically** through the structural [rule](rule.md)/[skill](skill.md) gate (never an auto-`must`; a contradiction routes to independent review), visible in the commit diff — see Step 3.

## Procedure

Run the steps in order; later steps assume earlier ones have settled the state they depend on. Skip steps outside the requested `scope`.

### Step 1 — Inventory & freshness

1. List `.dev_flow/tasks/task_*.md`, read each header (Task ID, Status, Contributors, Last updated, Current Work Item → phase + traceable ID, title).
2. Read `active_context.md` and `tasks/_index.md` if present.
3. Note freshness per the [status read protocol](status.md#read-protocol): tasks older than 7 days are `stale`; a single subtask far behind its siblings is `possibly-stalled` (informational only — no takeover).
4. Build a working table of *recorded* state. The next step establishes *real* state and diffs the two.

### Step 2 — Reconcile task state with reality

For each task, compare its recorded header `Status` against ground truth:

- **Linked documents** — open the concept/spec/plan named in Current Work Item. A plan marked `completed`, or a concept/spec `active` with all plan phases checked, indicates the task's work is done even if the header still says `in-progress`.
- **Git history** — search commits for the traceable ID (`[C_XXX]`, `[PL_XXX]`, …). A commit that lands the task's deliverable is hard evidence the work reached at least `committed`.

Then:

- If reality is clearly ahead of the header (committed / closed but header lags) → update the header `Status` via a targeted Edit, refresh `Last updated`, and drop a `[your-id]` Coordination Note recording what evidence drove the change. Do **not** touch contributor Subtask blocks.
- If reality and header disagree in a way evidence cannot settle → **flag in the report**, change nothing.
- If a task references a document or traceable ID that no longer exists → flag as an orphan for the report; consider it for archival in Step 3 only if its work is demonstrably finished.

### Step 3 — Compact & reflect on closed tasks

A task is **closed** when every subtask is `done` and its deliverable is committed or otherwise demonstrably finished (per Step 2). For each closed task:

**Compact** — shrink it to durable state, archive the noise:
- Move verbose per-subtask Activity, Coordination Notes overflow, and `done` Subtask blocks into `.dev_flow/session_history/session_YYYY-MM-DD.md` (see the [status archive procedure](status.md#session-history-archive)).
- Order eviction by **effective salience**, not pure age (see [Salience Markers](status.md#salience-markers)): `noise`/`superseded` entries are archived first; a `pin` is retained while its task is still active and goes inert once the task closes — **Reflect** (below) harvests its lesson *before* that demotion, so nothing durable is lost when the pin expires. Flag any task — active or closing — whose `pin` ratio is implausibly high and propose a re-grade (advisory, no hard cap).
- Leave behind a short outcome summary (1 short paragraph) plus links to the commit(s) and the archived history. The task file should read as "what this was and how it ended", not as a journal.

**Reflect** — harvest lessons before they are buried. This is the task-close **Transition Checkpoint** of [Experience Capture](../references/experience-capture.md) — the same auto-apply reflection, not a parallel path; its *harvest-before-demote* ordering is why it runs before any salience demotion on the closing task:
- Distill what the closed task *taught* that is not already captured. A reusable coding constraint or convention → write a rule automatically (apply the [rule](rule.md) phase's category/severity model; default `should`, never auto-`must`). Project-specific technology knowledge, gotchas, or research results → write/update a skill (apply the [skill](skill.md) phase's non-triviality filter — only what a senior dev arriving fresh would not already know). A would-be `must`, or a rule/skill that contradicts an existing one, routes to an independent clean-context review first. A blocker that recurred → note it so the same class is prevented.
- Feed durable insights **forward** into `rules/`/`skills/` so the knowledge outlives the task; let the rest be archived.
- **Optional `/dream` delegation:** if a reflection/`dream` skill is installed in this environment, delegate the reflection to it for a deeper retrospective; if not, perform the inline distillation above. Audit does not depend on `/dream`.

**Retire** — a fully-compacted closed task that is older than the retention window (default ~30 days, per the [tasks index](../templates/tasks_index.md)) is moved in its entirety to `session_history/` and dropped from the catalog.

### Step 4 — Trim the dashboard

Rebuild `active_context.md` as a thin dashboard (the [status regeneration procedure](status.md#regeneration-procedure)):

- **Active Tasks** table = tasks whose status ∈ {in-progress, blocked, review-pending}, one short row each (link + short title, phase, status, contributors, updated). No per-task narrative.
- **Recently Completed** = the latest 5 `done` tasks, one line each; older ones are already archived by Step 3.
- Any prose, progress checklists, or per-commit logs still living in the dashboard are moved to the owning task file (if active) or to `session_history/` (if historical). The dashboard stays under its ~80-line cap.

### Step 5 — Rebuild the catalog

Regenerate `tasks/_index.md` from the task headers: Active and Recently Completed lists mirroring the dashboard, plus the standing Conventions section from [templates/tasks_index.md](../templates/tasks_index.md). If the catalog is missing, create it from the template.

### Step 6 — Revise `rules/`

1. **Index ↔ disk (derived layers, applied)** — regenerate the derived layers from the rule headings per [Knowledge Scaling](../references/knowledge-scaling.md): `rules/_index.yaml` category blocks (`rules[]` = the file's units by id plus `moved_to` aliases; stale `file` / `summary` / `severity`; a unit selector wider than its category or a `phases` value outside the closed set → `index-drift`), each category file's `## Contents` digest above `digest_min` units (`digest-drift`; legacy `## Rule:` units get `(legacy)` lines — `heading-legacy`, the heading itself is never rewritten by audit), a missing `{#id}` anchor added from the heading id (`anchor-coverage`), severity words outside `must` / `should` / `prefer` or outside the heading normalized per the [rule phase](rule.md#severity-normalization) with every rename reported (`severity-vocab`), and a category `summary` over `directive_max` or reading as a list of directives replaced by one scope line (`summary-accumulation`). Defects reported, never auto-fixed: a duplicate anchor, a `moved_to` whose target is missing. Every line carries its measurement or is `unobserved`.
2. **Semantic duplicates** — scan for rules expressing the same constraint under different names or in different category files. Propose a merge: keep the canonical rule (usually the one with the stable ID and more references), deprecate or redirect the other, and report it. **Do not auto-delete.**
3. **Consistency** — check that each rule has a sane category and severity, that cross-references (rule IDs, file paths, traceable IDs) still resolve, and that `must`-severity rules are genuinely mandatory. Flag stale or dangling references.
4. **Ledger reconcile (every entry except `owner_kind: skill`)** — for each `.dev_flow/evidence/ledger.yaml` entry whose `trigger` is due, append one verdict (`improved` / `unchanged` / `regressed` / `still-unobserved`); a `regressed` in history keeps its blocker. `owner_kind: skill` belongs to Step 7.6. Report artifacts still `present` with no `exercised` observation ever recorded as **prune candidates** — *proposed*, never auto-deleted (Step 6.2 policy). Absent the file → no-op. See [Evidence Discipline](../references/evidence-discipline.md).
5. **Knowledge scaling — proposed** ([Knowledge Scaling → Audit verdicts](../references/knowledge-scaling.md#audit-verdicts); content changes, executed by the [rule phase](rule.md) only after confirmation; never a rule-id change): `unit-verbosity` — body over `unit_body_max`, `Why` over `rationale_max`, or a directive over `directive_max` → compaction into `<category>.log.md` (a directive is split or reclassified, never truncated); `kind-mismatch` — the kind test (a reviewer can check it from the artifact alone?) disagrees with the current kind, with the signals named → reclassification rule → skill; `split-planned` / `split-required` — file words over `full_read_limit` / `soft_split` → split into a new category file with its own selector, no umbrella; `knowledge-overload` — the probe: for each phase of the closed set, count the rules (aliases excluded) whose `applies_to.phases` is empty or contains it; over `activation_budget` → the remedy menu (narrow selectors · split the category by selector · reclassify procedural rules · merge duplicates · lower to `prefer` — never a `must`).
6. **Selector coverage (advisory)** — a category without `applies_to` is `always-on`: report `selector-missing` with a selector hint derived from the category name; the hint is never written automatically.

### Step 7 — Revise `skills/`

Curation of **procedural skills** — edit **incrementally, never a bulk rewrite** of the catalogue (avoids context-collapse / brevity-bias rot); see [Procedural Skills → Curation](../references/procedural-skills.md):

1. **Index ↔ disk** — reconcile the root and per-domain `_index.yaml` against the actual skill files (add missing, remove orphaned entries, fix `topics`/`file` fields) using the [skill](skill.md) phase's index format.
2. **Semantic duplicates / overlap** — flag skills covering the same topic or bleeding across domains; propose a merge into the better-placed file.
3. **Staleness** — flag skills whose `updated` date is old, that reference removed code/APIs, or that have decayed into general knowledge no longer worth keeping (re-apply the non-triviality filter). Propose removals; do not auto-delete.
4. **Freshness re-stamp** — where a procedural skill's `written_against` has drifted past the current tool/framework version, mark it `stale` (or re-stamp `current` if re-grounded). This restamp/incremental edit is **applied**; `prune`/`merge` stay **proposed** (destructive → review).
5. **Conflicts** — two skills prescribing different procedures for one surface are **surfaced as an explicit decision**, never silently resolved by picking one. Flag an implausibly high `candidate` ratio for re-grading (advisory).
6. **Evidence & ledger (`owner_kind: skill` entries)** — read `low-hit` from the recorded evidence state rather than estimating it, note skills whose missing `check` keeps them below `exercised` (advisory, never a promotion blocker), and apply the Step 6.4 reconcile to entries with `owner_kind: skill`. See [Evidence Discipline](../references/evidence-discipline.md).
7. **Knowledge scaling — applied** ([Knowledge Scaling](../references/knowledge-scaling.md)): a root `domains[].description` over `directive_max` or carrying inline file lists / history comments → one scope line (`summary-accumulation`); a skill file above `full_read_limit` without `## Contents` → `digest-drift` (the digest is required there; regenerated from its headings), a missing lead summary reported advisory per [Docs Scaling](../references/docs-scaling.md).
8. **Knowledge scaling — proposed** (executed by the [skill phase](skill.md) only after confirmation): `split-planned` / `split-required` for a skill file over `full_read_limit` / `soft_split` (a new file in the same domain, ids unchanged); `kind-mismatch` for a Pitfall that reads as an artifact constraint (`artifact-constraint-in-skill`) → reclassification skill → rule with a one-line pointer left in the skill; `knowledge-overload` per domain (entries over `activation_budget`) — advisory for skills.

### Step 7a — Groom `docs/_glossary.md` (if present)

Runs under scope `docs` (and `all`). Steps 7a–7c and 7e together constitute the **`docs` scope**. Keep the project glossary lean and true (see [Glossary](../references/glossary.md)):

1. **Duplicates** — merge terms that denote the same concept under different names; keep the canonical one, fold the rest into its `_Avoid_` list. Do not auto-delete — propose the merge.
2. **Stale ambiguities** — for each entry under *Flagged ambiguities*, check whether the conflict is now settled in the documents; if so, resolve it (canonical term + `_Avoid_`) and drop the flag.
3. **Boundary** — strip any implementation detail, contract, or relationship-as-behavior that crept in (those belong to a concept, not the glossary), and remove general tech terms that are not project domain vocabulary.

### Step 7b — Sweep open decisions, backlogs, todos & spikes (scope `docs` / under `all`)

Open deferrals are sanctioned only while their trigger lives — this step is what keeps them from quietly becoming permanent (see [Interview Mode → open decisions](../references/interview-mode.md)):

1. **Open Design Decisions** — scan `docs/*.concept.md|*.sp.md|*.plan.md` for `DEC_NN` records with `Status: open`. For each, check the **resolution trigger** against evidence (dates passed; the named event visible in git, docs, or task files). Trigger expired → flag it in the report and **propose the closing route**: a [research](research.md) run if the missing information is researchable, otherwise an interview with the developer. Never resolve a decision yourself; an open record with a still-live trigger is healthy — leave it.
2. **Plan backlogs** — flag backlog items that carry no return trigger/owner or whose trigger has expired; propose converting each to an open `DEC_NN` with a trigger, scheduling it into a plan phase, or dropping it explicitly.
3. **Todos register** — in `.dev_flow/todos/_index.md` (deferred work filed by [todo](todo.md)). A `candidate` simply sitting is a **normal** backlog state — do not nag it for age alone. Act on:
   - **triggerless / expired-trigger** entries → propose re-triggering or explicit drop;
   - **orphaned** entries whose relevant docs no longer exist, and `promoted` entries with no matching task → propose reconciling/dropping;
   - **matured candidates** (stable, repeatedly skipped, clearly belongs somewhere durable) → propose **graduating into documentation** — a concept's future-work/scope section or the owning plan's backlog — so it leaves the transient register for its proper home;
   - **fell-through queued follow-ups** — a `queued` record triggered `after task_<ID>` whose originating task is `done`/archived **or stale/abandoned** but was never picked up → surface for running or explicit drop (a deferred fix must not vanish when its task closed or stalled);
   - **contested clusters** — several `contested` records ([Review Convergence](../references/review-convergence.md)) pointing at one area is a signal that the area's `Criticality` is absent or declared wrong, not that each record needs a verdict. Propose declaring or correcting the criticality in the owning concept/spec; a `contested` record is never dropped as speculative, and a record missing either position is reported as malformed.
   Same discipline as plan backlogs — "later" is never silent.
4. **Spikes** — flag `docs/*.spike.md` stuck in `in-progress` beyond their time-box; propose concluding them (`concluded` / `inconclusive` / `abandoned`).

### Step 7c — Docs ↔ code drift detection (scope `docs` / under `all`)

Run the [drift detection algorithm](propagate.md#drift-detection-algorithm) from the propagate phase: collect traceable IDs from code and docs, cross-reference, check freshness by dates. **Obvious mechanical defects are auto-fixed in place** per [propagate → Auto-fix obvious defects](propagate.md#auto-fix-obvious-defects-no-permission-prompt) (broken `Depends on`/`Used by` links, dangling references/anchors, wrong-format IDs, stale dates) — no permission prompt. Findings that would **change meaning** (rewriting a design document, a substantive status call) are not rewritten — they go into the report and route through [propagate](propagate.md) / [Upstream Escalation](../references/escalation.md); an uncertain case goes to independent review.

### Step 7d — Revise `cache/` (scope `cache` / under `all`)

1. **Index ↔ disk** — reconcile `cache/_index.yaml` against the actual files: flag files with no entry (invisible) and entries pointing to missing files. For a **single-file entry**, the target is its `file`; for a **collection entry**, the targets are every `files[].name` resolved against the entry's `path` — flag a `files[]` sub-entity whose file is missing, and a file under `path` with no matching sub-entity. Unlike other indexes, the cache index carries `source` metadata that exists nowhere else — **reconcile entry by entry (and sub-entity by sub-entity), never regenerate from disk** (see [Resource Cache](../references/cache.md)).
2. **Unreferenced / stale** — flag entries whose `refs` no longer resolve (the doc or task that linked them is gone/closed) and snapshots superseded by newer ones; propose removals. **Do not auto-delete.**
3. **Expired validity** — for entries whose `valid_until` has passed (date) or whose named event has visibly occurred: run a **cheap currency check** where the source supports it (ETag/Last-Modified, Figma version metadata). Confirmed unchanged → extend `valid_until` (evidence-backed — applied directly and reported). Changed or uncheckable → flag for refresh; **audit never re-fetches** — the refresh belongs to the update task that made the resource stale (especially for `reacquire: manual / paid` entries).
4. **Trust & safety** — flag entries missing a `trust` level, and `trust: public` entries missing `checked` (saved without the safety check — see [Resource Cache → Trust & Safety](../references/cache.md)). Audit flags; the check itself runs when the resource is next touched, not from audit.
5. **Hygiene** — flag `/tmp`-style transients that crept in (logs, build output), missing `summary`/`source` fields, and domains grown disproportionately large; propose pruning oldest unreferenced entries first (cheapest `reacquire` first).

### Step 7e — Revise `docs/` integrity (scope `docs` / under `all`)

Grouped with Steps 7a–7c as the **`docs` scope**. Where 7a–7c groom the glossary, sweep deferrals, and detect docs↔code drift, 7e reconciles the documentation set itself. `docs/_index.md` is a **derived** view (regenerable per the [status regeneration procedure](status.md#regeneration-procedure)); the documents are the source of truth.

1. **Index ↔ disk** — reconcile `docs/_index.md` against the actual `docs/*.concept.md|*.sp.md|*.plan.md|*.epic.md`: add missing entries, drop entries for files that no longer exist, fix stale one-line descriptions and a wrong `Status` column. Below the >5-doc threshold where no `_index.md` is required (see [SKILL.md → File Organization](../SKILL.md#file-organization)), skip. The index is derived, so regenerate it freely when it has drifted.
2. **Status lifecycle reconciliation** — reconcile documents whose recorded `Status` disagrees with evidence (mirrors Step 2 for tasks, using the [status vocabulary](../SKILL.md#document-status-vocabulary)). A status that **mechanically contradicts hard evidence** is an obvious defect and is **auto-fixed** (a plan `in-progress` whose phases are all `[DONE]` → `completed`; the index `Status`-column following an evidence-backed reconciliation). A status change that is a genuine **judgement call** (a concept/spec long `draft` with active dependents; a `deprecated` doc still named in an active `Depends on` or referenced by live code IDs → an incomplete migration) is **flagged**, not auto-changed; an uncertain case goes to independent review.
3. **Cross-reference integrity** — check that `Depends on` / `Used by` resolve and are **bidirectional** (if A depends on B, B lists A under `Used by`), that header links (`Specification:`, `Plan:`, `Spike:`) resolve, and that inline `[C_XXX]` / `[SP_XXX]` / `[PL_XXX]` references point at existing documents/sections. An anchor may live in a **child file of a split umbrella set** — resolve against the whole set (corpus search), not only the same-named file, before flagging a reference as dangling (see [Docs Scaling](../references/docs-scaling.md)). Flag dangling or one-directional links; propose the reciprocal fix.
4. **Orphans & completeness** — flag a concept with no spec or plan where one is expected, a spec with no parent concept, a doc file absent from `_index.md`, and an epic referencing a missing concept.
5. **Freshness** — one definition, branched by history medium (see [Docs Scaling](../references/docs-scaling.md)): **vcs branch** — flag a document whose `Updated:` field is older than the file's last change in version control; **in-doc branch** (no vcs in the project) — flag a document whose file-system modification date is newer than its `Updated:` field. Changelog completeness has no mechanical measure where the history policy makes the Changelog optional — assessing it is the agent's judgement during the audit, not a check. Also flag lingering [banned phrases](concept.md#banned-phrases) in an `active` concept/spec.
6. **Duplicated value sets** — where the same set of values (an enum, a criteria list, an option set, a gate checklist) is enumerated in more than one document, diff the copies: each must list the same members. Also diff a set restated **within** one document — a contract's pseudocode-enumerated set against its §03 validation rule. Report divergence as drift and propose reconciliation toward the **owning** document (the spec defining the entity, else the concept); a copy is never treated as the authority.
7. **Documentation scaling** — mechanical measurements per [Docs Scaling](../references/docs-scaling.md) (thresholds and service-section list live there; none of these asks anyone to count by hand):
   - **contents-drift** — the set/titles of substantive h2s disagree with the Contents items, or a plan's Progress links are broken; report, then auto-fixed via [propagate](propagate.md).
   - **anchor-coverage** — a substantive h2/h3 in a concept/spec/plan carries no `{#ID}` anchor (service sections excluded per the closed list; epics excluded entirely — addressed as a whole `E_XXX`), or an anchor has more than one definition site in the corpus. Report only: missing anchors are added at the section's first substantive edit; a duplicate is a defect for manual resolution.
   - **size-trigger** — word count above `soft_split` → verdict `split-planned` (plan the split at the next substantive edit); above `hard_split`, or h3 density above `h3_density` → verdict `split-required` (split before adding new material). Report with the verdict; the split decision stays with the agent/developer, consumed by do/plan.
   - **freshness-branch** — covered by Step 7e.5 above; one definition of freshness, not two.

**Scope exclusion.** `docs/ext_adoption/` is out of scope for 7e.1–7e.5 and 7e.7: adoption documents are advisory, own no traceable ID, carry no pipeline `Status`, and are deliberately absent from `_index.md`'s document tables — flagging them as orphans or status-drift is a false positive. Check only that each pair's internal links resolve, and report an adoption document whose source repository no longer exists at the recorded path. See [External Repo Adoption](../references/repo-adoption.md).

Apply the derived/index fixes directly; **propose** status changes, reciprocal-link fixes, and removals (per *apply-safe / propose-judgement*). Document *content* fixes route through [propagate](propagate.md) — audit reports the drift, it does not rewrite design documents.

### Step 8 — Report

Always end with a structured report (and in `--dry-run`, this is the *only* output — nothing is written). Apply the safe/derived changes directly; list the judgement calls as proposals awaiting confirmation. **Never commit** — present the changes and follow the standard approval rule.

A check that could not be run — a *blocked* path, not the no-op of an absent directory — is reported as `unobserved` with the missing observation boundary; no section claims more than its evidence supports. See [Evidence Discipline](../references/evidence-discipline.md).

### Step 9 — `code` scope: the whole-codebase audit

The opt-in `code` scope is its **own multi-stage procedure**, not a subset of Steps 1–8 — it audits *source code*, where the rest of audit reconciles the `.dev_flow/` + `docs/` workspace. It owns three read-only stages — **RunAnalysis → Consolidate → ProducePlan** — preceded by **ParseIntent** (step 0), and stops at the Plan→Code gate; execution is the standard pipeline (**HandOff**). The full lens menu, per-lens checklists, the shared bottom-up walk, the SOLID/DRY heuristics, the antipattern catalogue, and the refactoring playbook live in **[references/code-audit.md](../references/code-audit.md)** — this step is the orchestration; that reference is the detail.

**Scope guiding constraints** (layered on audit's *non-committing* character):

- **Read-only w.r.t. source through the plan.** ParseIntent + the stages touch no source file and never commit. Code change happens only in HandOff, via the standard gated pipeline. (`git status` shows no source change after ProducePlan.)
- **Conclusions, not dumps.** A lens subagent returns Findings — a one-line `conclusion` + `file:line` locations. Raw search hits/traces/logs stay in the workspace, referenced by path, never inlined ([Delegation](../references/delegation.md)).
- **No silent truncation.** If cost/scope narrows the run (sampled, top-N, incremental), the report names what was excluded.
- **Reduced mode is honest.** Without documented architecture/rules, drop the conformance lenses (`standards`/`architecture`/`specifications`) and say so; recommend `onboard`. Never fabricate a conformance baseline.
- **Respect settled decisions & document faults.** A finding contradicting a settled `DEC_NN` cites it, never proposes blind reversal. A finding tracing to a *wrong document* routes to [Upstream Escalation](../references/escalation.md), not into the refactoring plan.
- **Free-form intent, no flags.** Unparsed fields default and the assumption is reported back.

#### Step 9.0 — ParseIntent + context gates

1. Parse the free-form intent text into working parameters — `scope` (path/glob, or `whole`), `lenses` (default: all base lenses), `depth` (`quick`/`standard`/`deep`), `mode` (`full`/`incremental` + `baseline`), `preview_only`. **Never prompt for flags**; an absent/unparseable field takes its default and the assumption is stated back in the result.
2. **`scope` resolving to nothing is an error (NO_SCOPE)** — state what was looked for; do **not** silently fall back to the whole codebase.
3. Run the standard gates (skill / rule / glossary / cache). Load `docs/_framework.md` (if present), the architecture concepts, `.dev_flow/rules/architecture.md`, and the [SOLID reference](../references/solid-architecture.md) as the conformance baseline (project rules override generic guidance). If none of these exist → mark the run **REDUCED**.
4. **Cost gate** (research-style) for a large `scope`/`deep` depth — may narrow to incremental or sampled (the narrowing is logged in the report). Create/Resume **AuditState** in `.dev_flow/audit/` (L2, mirrors onboard's `.dev_flow/onboard/`): `status`, `scope`/`lenses`/`mode`, `units {done,total}`, timestamps. A free-form "audit code continue" resumes from this state instead of re-analyzing settled units.

#### Step 9.1 — RunAnalysis (lens fan-out, parallel, read-only)

Fan out **one read-only subagent per lens** (the [code-audit-lens role](../roles/code-audit-lens.ai.md)), in parallel — `standards` / `architecture` / `specifications` / `patterns` / `duplication` / `security` / `correctness` / `performance` / `tests` / … (per the parsed `lenses`). Each applies the **[Shared Bottom-Up Analysis](../references/code-audit.md#shared-bottom-up-analysis)** walk through its one [checklist](../references/code-audit.md#per-lens-checklists) and returns `Finding[]` — **conclusions, not raw output**. No lens sees another's picture; that is what Consolidate is for. If `intent.lenses` parses empty, fall back to the base set — **never run zero lenses** (NO_LENS). The `security` lens audits *statically* (no exploit execution). Persist AuditState (`analyzing → consolidating`) as lenses return.

#### Step 9.2 — Consolidate (barrier — needs all Findings)

One agent (single writer) merges every lens's Findings into ConsolidatedFindings: dedup by normalized location + type; cluster cross-module duplication into a `duplication-cluster` / `abstraction-candidate`; detect docs↔code architecture drift (`docs-code-drift`); surface **cross-lens conflicts** (e.g. `standards` vs `architecture`) as their own `cross-lens-conflict` kind — **preserved, never averaged away**. For each, compute `blast_radius` via [Impact Walk](../references/impact.md) over the merged `affected` set and `priority = f(max severity, blast_radius, effort)`; return sorted by priority. This is a **barrier** — it needs the whole picture.

#### Step 9.3 — ProducePlan (the scope stops here)

1. Split consolidated findings into **top-N by priority** (plan items) and the **backlog** (each with a return trigger) — **nothing dropped silently**; the report names anything deferred or excluded.
2. Stamp the run once (`ts = YYYYMMDD_HHMMSS`) and emit, **under `.dev_flow/audit/` — not `docs/`**:
   - the plan **`.dev_flow/audit/<scope>_<ts>.plan.md`** (`Status: in-progress`) — each item carries its change-class (`trivial`/`standard`/`architectural`), affected files, rationale, and **links to ≥1 ConsolidatedFinding** (no orphan refactors). See the [refactoring playbook](../references/code-audit.md#refactoring-playbook).
   - the run report **`.dev_flow/audit/code-audit_<ts>.report.md`** — the [Report Structure](#report-structure) block, persisted (conclusions only; links its sibling plan).
3. Update **`docs/_framework.md`** from the abstraction-candidates — overview + links to the `.dev_flow/rules/`/`.dev_flow/skills/` that hold the detail; **no enforceable detail inlined**. The map stays in `docs/` (loaded as code-touch context).
4. Harvest rules/skills, auto-applied through the structural gate ([Experience Capture](../references/experience-capture.md) — no permission prompt; default `should`, never auto-`must`; would-be `must`/contradictions go to independent review). Route any finding that is really a *wrong document* to [Upstream Escalation](../references/escalation.md), not into the plan.
5. **Fast-track security:** an **exploitable `must`-severity** `security` finding is surfaced for an immediate [fix](fix.md)/escalation, **not** parked in the backlog. Persist AuditState (`planning`).
6. **No source change, no commit.** The developer approves the plan at the **Plan→Code gate** — that approval is what authorizes execution.

#### Step 9.4 — HandOff (execution = the standard pipeline)

If `preview_only`: **stop** — report the plan, no hand-off. Otherwise, on developer approval, run each plan item (by priority) through the standard gated phases for its change-class — [implement](implement.md) → [review](review.md) (clean-context) → [verify](verify.md) → commit (with approval). Audit may *orchestrate/offer* the chain but **edits no source and commits nothing itself** — each back-half stage is its own gated phase. Persist AuditState (`handed-off`).

#### Empty / trivial result

If consolidation yields no actionable findings, the plan is empty, HandOff is a no-op, and the run stays a safe read-only pass — report it as clean.

## Report Structure

Use this template so the result is scannable:

```
━━━ /dev-flow audit — <scope> <(dry-run)?> ━━━

✅ Applied (safe / derived)
   • Reconciled <task> status <old> → <new>  (evidence: commit <sha> / plan completed)
   • Compacted <task>: archived N log entries + M done subtasks → session_history/<file>
   • Trimmed active_context.md (<before> → <after> lines)
   • Rebuilt tasks/_index.md
   • rules/_index.yaml: +<a> entries, −<b> orphans, fixed <c> summaries
   • rules: regenerated <d> digests (<l> legacy lines), added <e> anchors, normalized <f> severity words (<old> → <new> each), de-accumulated <g> category summaries
   • evidence/ledger.yaml: +<v> verdicts appended; <p> prune candidates proposed
   • skills indexes: +<a>/−<b>
   • cache: extended valid_until on <n> entries (cheap check confirmed unchanged)
   • docs/_glossary.md: resolved <b> stale ambiguities (flags dropped)
   • docs: auto-fixed <n> mechanical statuses (plan all-[DONE] → completed)
   • docs/_index.md: rebuilt (+<a>/−<b> entries, fixed <c> status columns)

🔁 Reflection harvested (auto-applied; ⚑ = sent to independent review)
   • From <closed-task>: rule <Name> (<category>/<severity>)
   • From <closed-task>: skill <domain/Name>

🟡 Proposed (need your confirmation)
   • Merge rule <A> into <B> (same constraint: <reason>)
   • Compact rule <id> (<n> words > unit_body_max) → <category>.log.md
   • Reclassify rule <id> → skill <domain/> (kind test: <signals>) / skill <domain/Name> Pitfall → rule
   • Split <category>.md (<n> words > full_read_limit | soft_split): <id…> → <new-category>.md
   • knowledge-overload: phase <phase> activates <n> rules > activation_budget — remedies: <menu>
   • Merge glossary term <A> into <B> (same concept; fold into _Avoid_)
   • Remove stale skill <domain/Name> (references deleted <X>)
   • Remove cached <domain/file> (unreferenced since <date> / superseded by <newer>)
   • Close <DocID>_DEC_NN — trigger expired (<trigger>); route: research / interview
   • Backlog item "<item>" in <plan> has no return trigger; convert to DEC / schedule / drop
   • Re-grade <task>: pin ratio implausibly high (<n> pins) — propose demoting the stale ones
   • Reconcile <doc> status <X> → <Y> (judgement: long-draft with active dependents / deprecated still referenced)
   • Fix one-directional ref <A> → <B> (B missing <A> under Used by)

⚠️ Flagged (evidence insufficient — left unchanged)
   • <task> header says <X> but <Y>; cannot resolve from docs/git
   • Orphan task <task> references missing <doc/ID>
   • Drift: orphaned code ref <ID> at <file:line> / stale doc <doc> (code changed after Updated:)
   • Spike <name.spike.md> in-progress beyond its time-box
   • Cache: <a> files without index entry / <b> entries with missing files (cannot invent `source`)
   • Cache: <file> expired (<valid_until>), source changed or uncheckable — refresh belongs to its update task
   • Docs: orphan <doc> (no spec/plan) / deprecated <doc> still in active Depends on / banned phrase in active <doc>
   • Rules: duplicate anchor {#<id>} in <file-A>, <file-B> / alias <id> → <moved_to> target missing / category <name> has no applies_to (always-on; hint: <selector>)

✔️ Clean — no action: <areas with nothing to do>
```

For the **`code` scope** (Step 9) the output is the refactoring plan itself, summarized:

```
━━━ /dev-flow audit code — <scope> <(preview)?> ━━━

🔍 Run
   • Scope <scope> · lenses {<list>} · depth <d> · mode <full|incremental> · <REDUCED mode? recommend onboard>
   • Assumptions (defaulted, not prompted): <field=value, …>
   • Excluded / sampled (no silent truncation): <what + why>

📋 Refactoring plan → .dev_flow/audit/<scope>_<ts>.plan.md (in-progress)
   • Top-<N> items (priority desc): <id> <one-line> [<change-class>] → <affected> (from <finding-ids>)
   • Backlog: <m> items, each with a return trigger
   • cross-lens-conflict(s): <preserved as explicit decision input>

📝 Run report → .dev_flow/audit/code-audit_<ts>.report.md  (this summary, persisted)

🗺️ Framework map → docs/_framework.md  (stays in docs/ — loaded as code-touch context)
   • <abstractions added / links updated> (overview only — detail stays in rules/skills)

🚨 Fast-tracked (not in backlog)
   • Security <vuln:class> at <file:line> (must, exploitable) → immediate fix/escalation

🔁 Harvested (auto-applied; ⚑ = sent to independent review)
   • rule <Name> (<category>/<severity>) / skill <domain/Name>
   • Upstream escalation: <doc> appears wrong (code diverged because the spec is stale)

⏭️ Next: developer approves the plan at the Plan→Code gate → standard pipeline executes (audit commits nothing).
```

## Safety & Contention

- Audit may run alongside other contributors. It freely rewrites **derived** files (`active_context.md`, `tasks/_index.md`) and **closed** task files, but on **live** task files it makes only targeted header/status edits and appends Coordination Notes — it never rewrites another contributor's Subtask block or tagged entries.
- Re-read each file immediately before writing it; if a row/block moved, re-locate by Task ID / Author tag and re-apply (see [status targeted-edit safety](status.md#targeted-edit-safety)).
- If you suspect another contributor is mid-write on the same task, prefer flagging over rewriting, and coordinate via a Coordination Note.

## Dry-Run

`--dry-run` performs Steps 1–7e as analysis only and emits the Step 8 report without touching disk. Use it to preview a sweep, to review proposed merges and removals before committing to them, or to audit a shared `.dev_flow/` you do not want to mutate. The report distinguishes what *would* be applied from what would be proposed.

The **`code` scope** is read-only w.r.t. source through the plan regardless of any flag — its `--dry-run` equivalent is **preview-only intent** (e.g. "audit code … preview plan"), which runs ParseIntent + RunAnalysis + Consolidate + ProducePlan and then *stops* without offering the hand-off. (ProducePlan still writes the plan + report to `.dev_flow/audit/` and updates the framework map; those are derived artifacts, not source.)

## Relation to Other Phases

| Phase | How audit uses it |
|-------|-------------------|
| [status](status.md) | Reuses the read protocol, regeneration procedure, and archive flow. `status` reports drift; `audit` resolves it. |
| [rule](rule.md) | Catalogue edits and reflection-derived rules follow the rule phase's category/severity model and index format. |
| [skill](skill.md) | Skill index reconciliation, dedupe, and reflection-derived skills follow the skill phase's non-triviality filter and index chain. |
| [cache](../references/cache.md) | Step 7d reconciles the resource cache index ↔ disk and flags unreferenced/stale entries; the cache index is data (carries `source`), never regenerated. |
| [propagate](propagate.md) | When reconciliation reveals docs and code disagree, route the doc fix through propagate. Step 7c runs its drift detection algorithm as part of the sweep. |
| [research](research.md) | The proposed closing route for expired open-decision triggers whose missing information is researchable, and for stale spikes. The `code` scope borrows its cost gate for large/deep runs and its perspective-panel shape for the parallel lens fan-out + single-writer consolidation. |
| [review](review.md) | The `docs` scope (Step 7e) reconciles in bulk what review enforces per-change — Document Index Maintenance and Deprecation hygiene; review checks them on each change, audit periodically across the whole `docs/` set. The `code` scope (Step 9) reuses review's check catalogue as lenses (the Security check → the `security` lens) and review itself as a back-half execution stage. |
| [onboard](onboard.md) | The `code` scope shares onboard's bottom-up layered module walk — both call the single [Shared Bottom-Up Analysis](../references/code-audit.md#shared-bottom-up-analysis); onboard consumes it to *generate* docs, a lens to *audit* against them. |
| [implement](implement.md) / [verify](verify.md) | The back half of the `code` scope: the refactoring plan it produces is *executed* by the standard implement → review → verify → commit pipeline (audit hands off at the Plan→Code gate and changes no code itself). |
