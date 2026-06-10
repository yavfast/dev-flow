# Phase: Audit — Full `.dev_flow` Revision & Housekeeping

## Purpose

Context rots. As work proceeds, `.dev_flow/` accumulates drift: the dashboard
fills with narratives of tasks that are long since committed, task headers lag
behind the real state of their documents and the git history, finished tasks
keep their verbose activity logs, and `rules/` and `skills/` grow duplicate or
stale entries faster than their indexes are updated. None of this is caught by
the normal phases, because each phase only touches the slice of context it owns.

Audit is the periodic **whole-directory sweep** that brings `.dev_flow/` back in
line with reality. It reconciles every task's recorded state against ground
truth, trims the dashboard down to what is actually active, compacts and reflects
on closed work so its lessons survive while its noise is archived, and grooms the
`rules/` and `skills/` catalogues — plus the one doc-side artifact it owns for
hygiene, the `docs/_glossary.md` vocabulary. It is the write-heavy cousin of
[status](status.md): `status` *reports* drift, `audit` *resolves* it.

Audit composes existing phases rather than reinventing them — it leans on the
[status](status.md) regeneration procedure for the indexes, the [rule](rule.md)
and [skill](skill.md) phases for catalogue edits, and [propagate](propagate.md)
when a reconciliation reveals that docs and code disagree.

## Command

```
/dev-flow audit [scope] [--dry-run]
```

- `scope` (optional) — limit the sweep to one area. One of:
  `context` (dashboard + task reconciliation + compaction),
  `tasks` (reconcile + compact task files only),
  `rules`, `skills`, or `all` (default). `all` additionally grooms
  `docs/_glossary.md`, sweeps open Design Decisions / plan backlogs / stale
  spikes, and runs docs ↔ code drift detection.
- `--dry-run` — produce the audit report only; make no changes on disk.

### Examples

```
/dev-flow audit                  # full sweep of .dev_flow/
/dev-flow audit --dry-run        # show what would change, touch nothing
/dev-flow audit context          # only reconcile/trim context, skip rules & skills
/dev-flow audit rules            # only groom .dev_flow/rules/
/dev-flow audit skills           # only groom .dev_flow/skills/
```

## When to Run

- Periodically (e.g. end of a milestone, before a release, on a cadence).
- When `/dev-flow status` reports drift, stale tasks, or missing/incorrect indexes.
- When `active_context.md` has grown past its hygiene cap (~80 lines) or carries
  per-task narratives that belong in the task files.
- Before a new contributor or a fresh session needs a trustworthy picture.
- After a burst of commits, when several tasks closed but their headers were
  never moved to `done`.

## Guiding Principles

These shape every step below — read them first, they are the difference between
a clean-up and a data-loss incident.

1. **Reconcile, don't fabricate.** A task's real state is determined by evidence:
   the status of its linked concept/spec/plan in `docs/`, and the git history for
   its traceable ID. Change a recorded state only when evidence supports it. When
   evidence is missing or contradictory, **flag it in the report and leave the
   file unchanged** — never invent a status to make the dashboard look tidy.
2. **Non-destructive by default.** History is moved, never deleted. Overflow logs,
   completed subtask blocks, and closed task files go to
   `.dev_flow/session_history/`, not to `/dev/null`. Indexes are regenerable, so
   they may be rewritten freely; task files and history are not, so they are only
   ever appended to or relocated intact.
3. **Source-of-truth order.** Task files win over the dashboard and catalog (those
   are derived). For task *reality*, `docs/` document status and git win over the
   task header. For rules/skills, the files on disk win over their `_index.yaml`.
4. **Safe under contention.** Audit may run while other contributors hold open
   subtasks. It may fully rewrite *derived* indexes and *closed* task files, but
   it must never rewrite a live contributor's Subtask block or their tagged
   entries in shared sections — those are reconciled only with targeted edits, and
   only the header/status fields. Re-read immediately before each write.
5. **Apply the safe, propose the judgement.** Derived/reversible changes (index
   regen, dashboard trim, archival of done work, header reconciliation backed by
   git) are applied directly and reported. Judgement calls (merging two rules,
   deleting a skill, promoting a reflection into a new rule) are **proposed for
   confirmation**, never auto-applied. This mirrors the rule/skill phases, where
   removal requires explicit intent.

## Procedure

Run the steps in order; later steps assume earlier ones have settled the state
they depend on. Skip steps outside the requested `scope`.

### Step 1 — Inventory & freshness

1. List `.dev_flow/tasks/task_*.md`, read each header (Task ID, Status,
   Contributors, Last updated, Current Work Item → phase + traceable ID, title).
2. Read `active_context.md` and `tasks/_index.md` if present.
3. Note freshness per the [status read protocol](status.md#read-protocol): tasks
   older than 7 days are `stale`; a single subtask far behind its siblings is
   `possibly-stalled` (informational only — no takeover).
4. Build a working table of *recorded* state. The next step establishes *real*
   state and diffs the two.

### Step 2 — Reconcile task state with reality

For each task, compare its recorded header `Status` against ground truth:

- **Linked documents** — open the concept/spec/plan named in Current Work Item.
  A plan marked `completed`, or a concept/spec `active` with all plan phases
  checked, indicates the task's work is done even if the header still says
  `in-progress`.
- **Git history** — search commits for the traceable ID (`[C_XXX]`, `[PL_XXX]`,
  …). A commit that lands the task's deliverable is hard evidence the work
  reached at least `committed`.

Then:

- If reality is clearly ahead of the header (committed / closed but header lags)
  → update the header `Status` via a targeted Edit, refresh `Last updated`, and
  drop a `[your-id]` Coordination Note recording what evidence drove the change.
  Do **not** touch contributor Subtask blocks.
- If reality and header disagree in a way evidence cannot settle → **flag in the
  report**, change nothing.
- If a task references a document or traceable ID that no longer exists → flag as
  an orphan for the report; consider it for archival in Step 3 only if its work is
  demonstrably finished.

### Step 3 — Compact & reflect on closed tasks

A task is **closed** when every subtask is `done` and its deliverable is committed
or otherwise demonstrably finished (per Step 2). For each closed task:

**Compact** — shrink it to durable state, archive the noise:
- Move verbose per-subtask Activity, Coordination Notes overflow, and `done`
  Subtask blocks into `.dev_flow/session_history/session_YYYY-MM-DD.md` (see the
  [status archive procedure](status.md#session-history-archive)).
- Leave behind a short outcome summary (1 short paragraph) plus links to the
  commit(s) and the archived history. The task file should read as "what this
  was and how it ended", not as a journal.

**Reflect** — harvest lessons before they are buried:
- Distill what the closed task *taught* that is not already captured. A reusable
  coding constraint or convention → propose a rule (apply the [rule](rule.md)
  phase's category/severity model). Project-specific technology knowledge,
  gotchas, or research results → propose a skill (apply the [skill](skill.md)
  phase's non-triviality filter — only what a senior dev arriving fresh would not
  already know). A blocker that recurred → note it so the same class is prevented.
- Feed durable insights **forward** into `rules/`/`skills/` so the knowledge
  outlives the task; let the rest be archived.
- **Optional `/dream` delegation:** if a reflection/`dream` skill is installed in
  this environment, delegate the reflection to it for a deeper retrospective; if
  not, perform the inline distillation above. Audit does not depend on `/dream`.

**Retire** — a fully-compacted closed task that is older than the retention window
(default ~30 days, per the [tasks index](../templates/tasks_index.md)) is moved
in its entirety to `session_history/` and dropped from the catalog.

### Step 4 — Trim the dashboard

Rebuild `active_context.md` as a thin dashboard (the
[status regeneration procedure](status.md#regeneration-procedure)):

- **Active Tasks** table = tasks whose status ∈ {in-progress, blocked,
  review-pending}, one short row each (link + short title, phase, status,
  contributors, updated). No per-task narrative.
- **Recently Completed** = the latest 5 `done` tasks, one line each; older ones
  are already archived by Step 3.
- Any prose, progress checklists, or per-commit logs still living in the dashboard
  are moved to the owning task file (if active) or to `session_history/`
  (if historical). The dashboard stays under its ~80-line cap.

### Step 5 — Rebuild the catalog

Regenerate `tasks/_index.md` from the task headers: Active and Recently Completed
lists mirroring the dashboard, plus the standing Conventions section from
[templates/tasks_index.md](../templates/tasks_index.md). If the catalog is missing,
create it from the template.

### Step 6 — Revise `rules/`

1. **Index ↔ disk** — reconcile `rules/_index.yaml` against the actual files:
   add entries for rule files missing from the index, remove entries pointing to
   files that no longer exist, fix `summary`/`file` fields that have drifted.
2. **Semantic duplicates** — scan for rules expressing the same constraint under
   different names or in different category files. Propose a merge: keep the
   canonical rule (usually the one with the stable ID and more references),
   deprecate or redirect the other, and report it. **Do not auto-delete.**
3. **Consistency** — check that each rule has a sane category and severity, that
   cross-references (rule IDs, file paths, traceable IDs) still resolve, and that
   `must`-severity rules are genuinely mandatory. Flag stale or dangling
   references.

### Step 7 — Revise `skills/`

1. **Index ↔ disk** — reconcile the root and per-domain `_index.yaml` against the
   actual skill files (add missing, remove orphaned entries, fix `topics`/`file`
   fields) using the [skill](skill.md) phase's index format.
2. **Semantic duplicates / overlap** — flag skills covering the same topic or
   bleeding across domains; propose a merge into the better-placed file.
3. **Staleness** — flag skills whose `updated` date is old, that reference removed
   code/APIs, or that have decayed into general knowledge no longer worth keeping
   (re-apply the non-triviality filter). Propose removals; do not auto-delete.

### Step 7a — Groom `docs/_glossary.md` (if present)

Runs under `all`. Keep the project glossary lean and true (see
[Glossary](../references/glossary.md)):

1. **Duplicates** — merge terms that denote the same concept under different names;
   keep the canonical one, fold the rest into its `_Avoid_` list. Do not auto-delete —
   propose the merge.
2. **Stale ambiguities** — for each entry under *Flagged ambiguities*, check whether the
   conflict is now settled in the documents; if so, resolve it (canonical term + `_Avoid_`)
   and drop the flag.
3. **Boundary** — strip any implementation detail, contract, or relationship-as-behavior
   that crept in (those belong to a concept, not the glossary), and remove general
   tech terms that are not project domain vocabulary.

### Step 7b — Sweep open decisions, backlogs & spikes (under `all`)

Open deferrals are sanctioned only while their trigger lives — this step is what
keeps them from quietly becoming permanent (see
[Interview Mode → open decisions](../references/interview-mode.md)):

1. **Open Design Decisions** — scan `docs/*.concept.md|*.sp.md|*.plan.md` for
   `DEC_NN` records with `Status: open`. For each, check the **resolution
   trigger** against evidence (dates passed; the named event visible in git,
   docs, or task files). Trigger expired → flag it in the report and **propose
   the closing route**: a [research](research.md) run if the missing information
   is researchable, otherwise an interview with the developer. Never resolve a
   decision yourself; an open record with a still-live trigger is healthy — leave it.
2. **Plan backlogs** — flag backlog items that carry no return trigger/owner or
   whose trigger has expired; propose converting each to an open `DEC_NN` with a
   trigger, scheduling it into a plan phase, or dropping it explicitly.
3. **Spikes** — flag `docs/*.spike.md` stuck in `in-progress` beyond their
   time-box; propose concluding them (`concluded` / `inconclusive` / `abandoned`).

### Step 7c — Docs ↔ code drift detection (under `all`)

Run the [drift detection algorithm](propagate.md#drift-detection-algorithm) from
the propagate phase: collect traceable IDs from code and docs, cross-reference,
check freshness by dates. Findings (orphaned references, unreferenced sections,
stale documents, broken `Depends on`/`Used by` links) go into the report; actual
doc fixes are routed through [propagate](propagate.md) as proposals — audit
reports the drift, it does not rewrite design documents.

### Step 8 — Report

Always end with a structured report (and in `--dry-run`, this is the *only*
output — nothing is written). Apply the safe/derived changes directly; list the
judgement calls as proposals awaiting confirmation. **Never commit** — present the
changes and follow the standard approval rule.

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
   • docs/_glossary.md: merged <a> duplicate terms, resolved <b> stale ambiguities

🔁 Reflection harvested
   • From <closed-task>: proposed rule <Name> (<category>/<severity>)
   • From <closed-task>: proposed skill <domain/Name>

🟡 Proposed (need your confirmation)
   • Merge rule <A> into <B> (same constraint: <reason>)
   • Remove stale skill <domain/Name> (references deleted <X>)
   • Close <DocID>_DEC_NN — trigger expired (<trigger>); route: research / interview
   • Backlog item "<item>" in <plan> has no return trigger; convert to DEC / schedule / drop

⚠️ Flagged (evidence insufficient — left unchanged)
   • <task> header says <X> but <Y>; cannot resolve from docs/git
   • Orphan task <task> references missing <doc/ID>
   • Drift: orphaned code ref <ID> at <file:line> / stale doc <doc> (code changed after Updated:)
   • Spike <name.spike.md> in-progress beyond its time-box

✔️ Clean — no action: <areas with nothing to do>
```

## Safety & Contention

- Audit may run alongside other contributors. It freely rewrites **derived**
  files (`active_context.md`, `tasks/_index.md`) and **closed** task files, but on
  **live** task files it makes only targeted header/status edits and appends
  Coordination Notes — it never rewrites another contributor's Subtask block or
  tagged entries.
- Re-read each file immediately before writing it; if a row/block moved, re-locate
  by Task ID / Author tag and re-apply (see
  [status targeted-edit safety](status.md#targeted-edit-safety)).
- If you suspect another contributor is mid-write on the same task, prefer
  flagging over rewriting, and coordinate via a Coordination Note.

## Dry-Run

`--dry-run` performs Steps 1–7c as analysis only and emits the Step 8 report
without touching disk. Use it to preview a sweep, to review proposed merges and
removals before committing to them, or to audit a shared `.dev_flow/` you do not
want to mutate. The report distinguishes what *would* be applied from what would
be proposed.

## Relation to Other Phases

| Phase | How audit uses it |
|-------|-------------------|
| [status](status.md) | Reuses the read protocol, regeneration procedure, and archive flow. `status` reports drift; `audit` resolves it. |
| [rule](rule.md) | Catalogue edits and reflection-derived rules follow the rule phase's category/severity model and index format. |
| [skill](skill.md) | Skill index reconciliation, dedupe, and reflection-derived skills follow the skill phase's non-triviality filter and index chain. |
| [propagate](propagate.md) | When reconciliation reveals docs and code disagree, route the doc fix through propagate. Step 7c runs its drift detection algorithm as part of the sweep. |
| [research](research.md) | The proposed closing route for expired open-decision triggers whose missing information is researchable, and for stale spikes. |
