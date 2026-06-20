# Phase: Audit — Full `.dev_flow/` + `docs/` Revision & Housekeeping

## Purpose

Context rots. As work proceeds, `.dev_flow/` accumulates drift: the dashboard fills with narratives of tasks that are long since committed, task headers lag behind the real state of their documents and the git history, finished tasks keep their verbose activity logs, and `rules/` and `skills/` grow duplicate or stale entries faster than their indexes are updated. None of this is caught by the normal phases, because each phase only touches the slice of context it owns.

Audit is the periodic **whole-directory sweep** that brings `.dev_flow/` back in line with reality. It reconciles every task's recorded state against ground truth, trims the dashboard down to what is actually active, compacts and reflects on closed work so its lessons survive while its noise is archived, and grooms the `rules/`, `skills/`, and `cache/` catalogues — and reconciles the integrity of the `docs/` documentation set itself (index, statuses, cross-references, orphans, glossary, and docs↔code drift) through the `docs` scope. It is the write-heavy cousin of [status](status.md): `status` *reports* drift, `audit` *resolves* it.

Audit composes existing phases rather than reinventing them — it leans on the [status](status.md) regeneration procedure for the indexes, the [rule](rule.md) and [skill](skill.md) phases for catalogue edits, and [propagate](propagate.md) when a reconciliation reveals that docs and code disagree.

## Command

```
/dev-flow audit [scope] [--dry-run]
```

- `scope` (optional) — limit the sweep to one area. One of: `context` (dashboard + task reconciliation + compaction), `tasks` (reconcile + compact task files only), `rules`, `skills`, `cache`, `docs` (documentation integrity — index · statuses · cross-refs · orphans · glossary · drift), or `all` (default). `all` runs every scope below — the `.dev_flow/` workspace scopes **and** `docs`.
- `--dry-run` — produce the audit report only; make no changes on disk.

### Scope → steps

Each scope runs a defined subset of the [procedure](#procedure); `all` runs them all, in order. Skip steps outside the requested scope.

| Scope | Steps | Reconciles |
|-------|-------|------------|
| `context` | 1–4 | dashboard + task reconciliation + compaction |
| `tasks` | 1–3 | task files only (reconcile + compact) |
| `rules` | 6 | `.dev_flow/rules/` catalogue |
| `skills` | 7 | `.dev_flow/skills/` catalogue |
| `cache` | 7d | `.dev_flow/cache/` index ↔ disk |
| `docs` | 7a · 7b · 7c · 7e | `docs/` — glossary, open decisions/backlogs/spikes, docs↔code drift, **index · statuses · cross-refs · orphans · freshness** |
| `all` (default) | 1–8 | everything above (workspace **and** docs) |

### Examples

```
/dev-flow audit                  # full sweep — workspace + docs
/dev-flow audit --dry-run        # show what would change, touch nothing
/dev-flow audit context          # only reconcile/trim context, skip rules & skills
/dev-flow audit rules            # only groom .dev_flow/rules/
/dev-flow audit skills           # only groom .dev_flow/skills/
/dev-flow audit cache            # only groom .dev_flow/cache/
/dev-flow audit docs             # only check docs/ integrity (index, statuses, refs, glossary, drift)
```

## When to Run

- Periodically (e.g. end of a milestone, before a release, on a cadence).
- When `/dev-flow status` reports drift, stale tasks, or missing/incorrect indexes.
- When `active_context.md` has grown past its hygiene cap (~80 lines — caps defined in [status phase → Context Hygiene](status.md#context-hygiene)) or carries per-task narratives that belong in the task files.
- Before a new contributor or a fresh session needs a trustworthy picture.
- After a burst of commits, when several tasks closed but their headers were never moved to `done`.
- After a batch of document edits — when a doc's `Status` lags reality (a plan whose phases are all `[DONE]` but still `in-progress`), `docs/_index.md` has drifted from the files, or `Depends on` / `Used by` links broke.

## Guiding Principles

These shape every step below — read them first, they are the difference between a clean-up and a data-loss incident.

1. **Reconcile, don't fabricate.** A task's real state is determined by evidence: the status of its linked concept/spec/plan in `docs/`, and the git history for its traceable ID. Change a recorded state only when evidence supports it. When evidence is missing or contradictory, **flag it in the report and leave the file unchanged** — never invent a status to make the dashboard look tidy.
2. **Non-destructive by default.** History is moved, never deleted. Overflow logs, completed subtask blocks, and closed task files go to `.dev_flow/session_history/`, not to `/dev/null`. Indexes are regenerable, so they may be rewritten freely — with one exception: **`cache/_index.yaml` is data, not a derived view** (its `source` metadata exists nowhere else), so it is only ever reconciled entry by entry, never regenerated (see [Resource Cache](../references/cache.md)). Task files and history are not regenerable either — they are only ever appended to or relocated intact.
3. **Source-of-truth order.** Task files win over the dashboard and catalog (those are derived). For task *reality*, `docs/` document status and git win over the task header. For rules/skills, the files on disk win over their `_index.yaml`. For the cache, disk decides *existence* (a missing file orphans its entry) but the index owns *metadata* (`source`/`summary` cannot be rebuilt from disk) — flag mismatches, never drop or regenerate entries wholesale (Step 7d).
4. **Safe under contention.** Audit may run while other contributors hold open subtasks. It may fully rewrite *derived* indexes and *closed* task files, but it must never rewrite a live contributor's Subtask block or their tagged entries in shared sections — those are reconciled only with targeted edits, and only the header/status fields. Re-read immediately before each write.
5. **Apply the safe, propose the judgement.** Derived/reversible changes (index regen, dashboard trim, archival of done work, header reconciliation backed by git) are applied directly and reported. Judgement calls (merging two rules, deleting a skill, promoting a reflection into a new rule) are **proposed for confirmation**, never auto-applied. This mirrors the rule/skill phases, where removal requires explicit intent.

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

**Reflect** — harvest lessons before they are buried. This is the task-close **Transition Checkpoint** of [Experience Capture](../references/experience-capture.md) — the same propose-not-apply reflection, not a parallel path; its *harvest-before-demote* ordering is why it runs before any salience demotion on the closing task:
- Distill what the closed task *taught* that is not already captured. A reusable coding constraint or convention → propose a rule (apply the [rule](rule.md) phase's category/severity model). Project-specific technology knowledge, gotchas, or research results → propose a skill (apply the [skill](skill.md) phase's non-triviality filter — only what a senior dev arriving fresh would not already know). A blocker that recurred → note it so the same class is prevented.
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

1. **Index ↔ disk** — reconcile `rules/_index.yaml` against the actual files: add entries for rule files missing from the index, remove entries pointing to files that no longer exist, fix `summary`/`file` fields that have drifted.
2. **Semantic duplicates** — scan for rules expressing the same constraint under different names or in different category files. Propose a merge: keep the canonical rule (usually the one with the stable ID and more references), deprecate or redirect the other, and report it. **Do not auto-delete.**
3. **Consistency** — check that each rule has a sane category and severity, that cross-references (rule IDs, file paths, traceable IDs) still resolve, and that `must`-severity rules are genuinely mandatory. Flag stale or dangling references.

### Step 7 — Revise `skills/`

1. **Index ↔ disk** — reconcile the root and per-domain `_index.yaml` against the actual skill files (add missing, remove orphaned entries, fix `topics`/`file` fields) using the [skill](skill.md) phase's index format.
2. **Semantic duplicates / overlap** — flag skills covering the same topic or bleeding across domains; propose a merge into the better-placed file.
3. **Staleness** — flag skills whose `updated` date is old, that reference removed code/APIs, or that have decayed into general knowledge no longer worth keeping (re-apply the non-triviality filter). Propose removals; do not auto-delete.

### Step 7a — Groom `docs/_glossary.md` (if present)

Runs under scope `docs` (and `all`). Steps 7a–7c and 7e together constitute the **`docs` scope**. Keep the project glossary lean and true (see [Glossary](../references/glossary.md)):

1. **Duplicates** — merge terms that denote the same concept under different names; keep the canonical one, fold the rest into its `_Avoid_` list. Do not auto-delete — propose the merge.
2. **Stale ambiguities** — for each entry under *Flagged ambiguities*, check whether the conflict is now settled in the documents; if so, resolve it (canonical term + `_Avoid_`) and drop the flag.
3. **Boundary** — strip any implementation detail, contract, or relationship-as-behavior that crept in (those belong to a concept, not the glossary), and remove general tech terms that are not project domain vocabulary.

### Step 7b — Sweep open decisions, backlogs & spikes (scope `docs` / under `all`)

Open deferrals are sanctioned only while their trigger lives — this step is what keeps them from quietly becoming permanent (see [Interview Mode → open decisions](../references/interview-mode.md)):

1. **Open Design Decisions** — scan `docs/*.concept.md|*.sp.md|*.plan.md` for
   `DEC_NN` records with `Status: open`. For each, check the **resolution trigger** against evidence (dates passed; the named event visible in git, docs, or task files). Trigger expired → flag it in the report and **propose the closing route**: a [research](research.md) run if the missing information is researchable, otherwise an interview with the developer. Never resolve a decision yourself; an open record with a still-live trigger is healthy — leave it.
2. **Plan backlogs** — flag backlog items that carry no return trigger/owner or whose trigger has expired; propose converting each to an open `DEC_NN` with a trigger, scheduling it into a plan phase, or dropping it explicitly.
3. **Spikes** — flag `docs/*.spike.md` stuck in `in-progress` beyond their time-box; propose concluding them (`concluded` / `inconclusive` / `abandoned`).

### Step 7c — Docs ↔ code drift detection (scope `docs` / under `all`)

Run the [drift detection algorithm](propagate.md#drift-detection-algorithm) from the propagate phase: collect traceable IDs from code and docs, cross-reference, check freshness by dates. Findings (orphaned references, unreferenced sections, stale documents, broken `Depends on`/`Used by` links) go into the report; actual doc fixes are routed through [propagate](propagate.md) as proposals — audit reports the drift, it does not rewrite design documents.

### Step 7d — Revise `cache/` (scope `cache` / under `all`)

1. **Index ↔ disk** — reconcile `cache/_index.yaml` against the actual files: flag files with no entry (invisible) and entries pointing to missing files. For a **single-file entry**, the target is its `file`; for a **collection entry**, the targets are every `files[].name` resolved against the entry's `path` — flag a `files[]` sub-entity whose file is missing, and a file under `path` with no matching sub-entity. Unlike other indexes, the cache index carries `source` metadata that exists nowhere else — **reconcile entry by entry (and sub-entity by sub-entity), never regenerate from disk** (see [Resource Cache](../references/cache.md)).
2. **Unreferenced / stale** — flag entries whose `refs` no longer resolve (the doc or task that linked them is gone/closed) and snapshots superseded by newer ones; propose removals. **Do not auto-delete.**
3. **Expired validity** — for entries whose `valid_until` has passed (date) or whose named event has visibly occurred: run a **cheap currency check** where the source supports it (ETag/Last-Modified, Figma version metadata). Confirmed unchanged → extend `valid_until` (evidence-backed — applied directly and reported). Changed or uncheckable → flag for refresh; **audit never re-fetches** — the refresh belongs to the update task that made the resource stale (especially for `reacquire: manual / paid` entries).
4. **Trust & safety** — flag entries missing a `trust` level, and `trust: public` entries missing `checked` (saved without the safety check — see [Resource Cache → Trust & Safety](../references/cache.md)). Audit flags; the check itself runs when the resource is next touched, not from audit.
5. **Hygiene** — flag `/tmp`-style transients that crept in (logs, build output), missing `summary`/`source` fields, and domains grown disproportionately large; propose pruning oldest unreferenced entries first (cheapest `reacquire` first).

### Step 7e — Revise `docs/` integrity (scope `docs` / under `all`)

Grouped with Steps 7a–7c as the **`docs` scope**. Where 7a–7c groom the glossary, sweep deferrals, and detect docs↔code drift, 7e reconciles the documentation set itself. `docs/_index.md` is a **derived** view (regenerable per the [status regeneration procedure](status.md#regeneration-procedure)); the documents are the source of truth.

1. **Index ↔ disk** — reconcile `docs/_index.md` against the actual `docs/*.concept.md|*.sp.md|*.plan.md|*.epic.md`: add missing entries, drop entries for files that no longer exist, fix stale one-line descriptions and a wrong `Status` column. Below the >5-doc threshold where no `_index.md` is required (see [SKILL.md → File Organization](../SKILL.md#file-organization)), skip. The index is derived, so regenerate it freely when it has drifted.
2. **Status lifecycle reconciliation** — flag documents whose recorded `Status` disagrees with evidence (mirrors Step 2 for tasks, using the [status vocabulary](../SKILL.md#document-status-vocabulary)): a plan `in-progress` whose phases are all `[DONE]` → propose `completed`; a concept/spec long `draft` with active dependents → flag; a `deprecated` doc still named in an active document's `Depends on`, or still referenced by live code IDs → flag the incomplete migration. **Propose** status changes (they are judgement calls); only the index `Status`-column fix that follows an evidence-backed task/plan reconciliation is applied directly.
3. **Cross-reference integrity** — check that `Depends on` / `Used by` resolve and are **bidirectional** (if A depends on B, B lists A under `Used by`), that header links (`Specification:`, `Plan:`, `Spike:`) resolve, and that inline `[C_XXX]` / `[SP_XXX]` / `[PL_XXX]` references point at existing documents/sections. Flag dangling or one-directional links; propose the reciprocal fix.
4. **Orphans & completeness** — flag a concept with no spec or plan where one is expected, a spec with no parent concept, a doc file absent from `_index.md`, and an epic referencing a missing concept.
5. **Freshness** — flag active concepts/specs/plans with significant edits but no matching `Changelog` row, an `Updated` date older than commits that touched the document's IDs, and lingering [banned phrases](concept.md#banned-phrases) in an `active` concept/spec.

Apply the derived/index fixes directly; **propose** status changes, reciprocal-link fixes, and removals (per *apply-safe / propose-judgement*). Document *content* fixes route through [propagate](propagate.md) — audit reports the drift, it does not rewrite design documents.

### Step 8 — Report

Always end with a structured report (and in `--dry-run`, this is the *only* output — nothing is written). Apply the safe/derived changes directly; list the judgement calls as proposals awaiting confirmation. **Never commit** — present the changes and follow the standard approval rule.

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
   • skills indexes: +<a>/−<b>
   • cache: extended valid_until on <n> entries (cheap check confirmed unchanged)
   • docs/_glossary.md: merged <a> duplicate terms, resolved <b> stale ambiguities
   • docs/_index.md: rebuilt (+<a>/−<b> entries, fixed <c> status columns)

🔁 Reflection harvested
   • From <closed-task>: proposed rule <Name> (<category>/<severity>)
   • From <closed-task>: proposed skill <domain/Name>

🟡 Proposed (need your confirmation)
   • Merge rule <A> into <B> (same constraint: <reason>)
   • Remove stale skill <domain/Name> (references deleted <X>)
   • Remove cached <domain/file> (unreferenced since <date> / superseded by <newer>)
   • Close <DocID>_DEC_NN — trigger expired (<trigger>); route: research / interview
   • Backlog item "<item>" in <plan> has no return trigger; convert to DEC / schedule / drop
   • Re-grade <task>: pin ratio implausibly high (<n> pins) — propose demoting the stale ones
   • Reconcile <doc> status <X> → <Y> (evidence: plan phases all done / active dependents)
   • Fix one-directional ref <A> → <B> (B missing <A> under Used by)

⚠️ Flagged (evidence insufficient — left unchanged)
   • <task> header says <X> but <Y>; cannot resolve from docs/git
   • Orphan task <task> references missing <doc/ID>
   • Drift: orphaned code ref <ID> at <file:line> / stale doc <doc> (code changed after Updated:)
   • Spike <name.spike.md> in-progress beyond its time-box
   • Cache: <a> files without index entry / <b> entries with missing files (cannot invent `source`)
   • Cache: <file> expired (<valid_until>), source changed or uncheckable — refresh belongs to its update task
   • Docs: orphan <doc> (no spec/plan) / deprecated <doc> still in active Depends on / banned phrase in active <doc>

✔️ Clean — no action: <areas with nothing to do>
```

## Safety & Contention

- Audit may run alongside other contributors. It freely rewrites **derived** files (`active_context.md`, `tasks/_index.md`) and **closed** task files, but on **live** task files it makes only targeted header/status edits and appends Coordination Notes — it never rewrites another contributor's Subtask block or tagged entries.
- Re-read each file immediately before writing it; if a row/block moved, re-locate by Task ID / Author tag and re-apply (see [status targeted-edit safety](status.md#targeted-edit-safety)).
- If you suspect another contributor is mid-write on the same task, prefer flagging over rewriting, and coordinate via a Coordination Note.

## Dry-Run

`--dry-run` performs Steps 1–7e as analysis only and emits the Step 8 report without touching disk. Use it to preview a sweep, to review proposed merges and removals before committing to them, or to audit a shared `.dev_flow/` you do not want to mutate. The report distinguishes what *would* be applied from what would be proposed.

## Relation to Other Phases

| Phase | How audit uses it |
|-------|-------------------|
| [status](status.md) | Reuses the read protocol, regeneration procedure, and archive flow. `status` reports drift; `audit` resolves it. |
| [rule](rule.md) | Catalogue edits and reflection-derived rules follow the rule phase's category/severity model and index format. |
| [skill](skill.md) | Skill index reconciliation, dedupe, and reflection-derived skills follow the skill phase's non-triviality filter and index chain. |
| [cache](../references/cache.md) | Step 7d reconciles the resource cache index ↔ disk and flags unreferenced/stale entries; the cache index is data (carries `source`), never regenerated. |
| [propagate](propagate.md) | When reconciliation reveals docs and code disagree, route the doc fix through propagate. Step 7c runs its drift detection algorithm as part of the sweep. |
| [research](research.md) | The proposed closing route for expired open-decision triggers whose missing information is researchable, and for stale spikes. |
| [review](review.md) | The `docs` scope (Step 7e) reconciles in bulk what review enforces per-change — Document Index Maintenance and Deprecation hygiene; review checks them on each change, audit periodically across the whole `docs/` set. |
