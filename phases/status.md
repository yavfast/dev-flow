# Phase: Status — Session Context, Checkpoint & Resume

## Contents

- [Purpose](#purpose) — What this phase loads, fixes, and re-enters; owns the read/write protocol every phase follows; reports drift, audit resolves it
- [Command](#command) — The three invocations: `status [task_id]` reports, `checkpoint [note]` fixes a session boundary, `resume [task_id]` re-enters
- [Roles](#roles) — Each phase role updates its own subtask block; ContextTracker is the dedicated read/write/regenerate/checkpoint/resume worker
- [Context Files](#context-files) — `.dev_flow/` layout, source-of-truth rule (task files win over derived indexes), templates, legacy single-file migration
- [Collaboration Model (read first)](#collaboration-model-read-first) — Table of who may edit each region of a shared task file; no exclusive locking, no time-based takeover
- [Read Protocol](#read-protocol) — Re-attention first, then Steps 1–5: read dashboard, read task file, validate freshness, output templates, continuation
- [Write Protocol](#write-protocol) — Working-memory promotion; updates at phase start / step end / phase end / completion; targeted-edit safety, append-only
- [Checkpoint — Fix the Task for a Session Boundary](#checkpoint--fix-the-task-for-a-session-boundary) — On-demand fixation at a session boundary: four movements, readiness set R1–R7, the handoff record, what it must not do
- [Resume — Re-enter a Task in a Fresh Session](#resume--re-enter-a-task-in-a-fresh-session) — Re-entry in a fresh session: reconcile against the tree, the unambiguous test, the brief, the work offer
- [Regeneration Procedure](#regeneration-procedure) — How any contributor rebuilds `active_context.md` and `tasks/_index.md` from task headers, incl. Deferred (todos)
- [Salience Markers](#salience-markers) — `{s:pin|noise|superseded→}` vocabulary, written form, task-scoped expiry, how dev-flow compaction honours salience
- [Context Hygiene](#context-hygiene) — Canonical caps (10 log entries, ~300-line task, ~80-line dashboard), activity content filter, session history archive

## Purpose

Load the active development context into the session so you can quickly resume where you (or other contributors) left off, without re-reading all documents from scratch.

This phase owns the whole **session boundary**, not just the report: `status` tells you where things stand, [`checkpoint`](#checkpoint--fix-the-task-for-a-session-boundary) fixes a task into durable state before a session ends, and [`resume`](#resume--re-enter-a-task-in-a-fresh-session) re-enters it in a fresh one. The three share one protocol, so a task written by any of them is readable by all.

Status also defines the **read/write protocol** that every other phase follows when touching the context files — read this whenever you need to update task state safely under multiple AI contributors.

For the periodic whole-directory revision see the [audit phase](audit.md): `status` *reports* drift, `audit` *resolves* it, building on this protocol's regeneration and archive procedures.

## Command

```
/dev-flow status [task_id]
/dev-flow checkpoint [note]
/dev-flow resume [task_id]
```

- **`status`** — report state. No argument shows all active tasks (dashboard summary); an optional `task_id` shows the detailed state for one task file. Reports only — it changes nothing.
- **`checkpoint`** — fix the caller's current task into durable state at a session boundary, with an optional one-line note recording why the boundary happened. See [Checkpoint](#checkpoint--fix-the-task-for-a-session-boundary).
- **`resume`** — establish where the work stands, continue it when the picture is unambiguous, or offer the next work. An optional `task_id` selects the task. See [Resume](#resume--re-enter-a-task-in-a-fresh-session).

## Roles

- Each phase role updates its own subtask block as it runs.
- **ContextTracker** ([context-tracker.ai.md](../roles/context-tracker.ai.md)) is the dedicated worker for every mode of this phase — read, write, regenerate, checkpoint, resume; invoke it when context needs refreshing without executing a phase.

## Context Files

```
.dev_flow/
├── active_context.md          # Dashboard — table of active tasks + recently completed (+ thin Deferred pointer)
├── output_styles.md           # Project style profiles — documentation + chat registers (Output Styles); absent → shipped defaults
├── cache/                     # Durable resources (Figma exports, downloads, baselines) + _index.yaml (Resource Cache)
├── evidence/                  # ledger.yaml — intervention ledger (Evidence Discipline); absent → reconcile is a no-op
├── tasks/
│   ├── _index.md              # Catalog of task files
│   ├── task_<ID>.md           # Per-task shared context (multiple contributors)
│   └── ...
├── todos/                     # Deferred future work filed by `todo` + _index.md
└── session_history/           # Archived sessions
```

**Source of truth:** the individual task files under `tasks/`. The dashboard (`active_context.md`) and the catalog (`tasks/_index.md`) are **derived views** — if they conflict with reality, the task files win and the indexes are rebuilt.

Templates:
- [templates/task_context.md](../templates/task_context.md)
- [templates/active_context.md](../templates/active_context.md)
- [templates/tasks_index.md](../templates/tasks_index.md)

### Legacy single-file `active_context.md` (manual migration)

Projects created under the previous single-file model have a monolithic `.dev_flow/active_context.md` with Current Work Item / Progress State / Recent Changes sections and no `tasks/` directory. There is **no automatic migration**. When you encounter such a project:

1. Rename the old file: `mv .dev_flow/active_context.md .dev_flow/legacy_context.md` (keep it as reference; do not delete).
2. Create `.dev_flow/tasks/` and seed it with `_index.md` from [templates/tasks_index.md](../templates/tasks_index.md).
3. Create a fresh `.dev_flow/active_context.md` from [templates/active_context.md](../templates/active_context.md).
4. For each in-progress work item described in the legacy file, create one `tasks/task_<ID>.md` from [templates/task_context.md](../templates/task_context.md), copying the relevant Current Work Item, Progress State, and Blocking Issues into your own Subtask block. Add yourself as the first contributor.
5. Add a row in the new `active_context.md` and `tasks/_index.md` for each task.
6. When all in-progress work items are migrated, archive `.dev_flow/legacy_context.md` to `.dev_flow/session_history/legacy_pre_multi_contributor.md` (or delete if it has no historical value).

## Collaboration Model (read first)

A task file is **shared** between multiple AI contributors. Each contributor appears in the file's `Contributors` field and owns at least one **Subtask block** plus their tagged entries in shared sections.

| Region of the file | Who edits it | Rule |
|--------------------|--------------|------|
| Header (Status, Last updated, Contributors) | Any contributor | Targeted Edit on the specific field; re-read before write |
| Current Work Item | Any contributor (rare changes) | Targeted Edit on a row; re-read before write |
| Description | Any contributor | **Append-only** paragraphs, each signed `— <agent-id>`. Never rewrite others' paragraphs |
| Subtask blocks | The block's `Author` only | The author is the sole editor of their block |
| Coordination Notes | Any contributor | Append-only entries, each prefixed `[agent-id]` |
| Blocking Issues | Reporter only edits/resolves their own | Other contributors may comment in Coordination Notes |
| Relevant Context table | Any contributor | Append rows tagged with the contributor; don't rewrite existing rows |
| Shared Activity Log | Any contributor | Append-only, newest first, tagged `[agent-id]` |

**There is no exclusive locking and no time-based takeover.** If a subtask is stalled, a contributor who wants to push it forward adds a *new* Subtask block referencing the stale one — never rewrites the stale block.

## Read Protocol

**Re-attention first (after a compaction or subtask switch).** Before re-reading the task files, re-read the **session working memory** area — the L1 notes / parameters / reminders / reads that survive a compaction (see [Resource Cache → Session Working Memory](../references/cache.md#session-working-memory-l1)).

### Step 1: Read the dashboard

1. Check if `.dev_flow/active_context.md` exists.
   - If **not** — inform the user: no active context found. Suggest running `/dev-flow onboard` (existing project) or starting with `/dev-flow concept`. Stop here.
   - If **yes** — read it. Pull the list of active tasks (table rows).

2. If a `task_id` argument was supplied, jump straight to that task file.

### Step 2: Read the relevant task file

For each task you want to inspect, read `tasks/task_<ID>.md`. The task file holds the detailed state: Current Work Item, Description, Subtask blocks, Coordination Notes, Blocking Issues, Relevant Context, Shared Activity Log.

When resuming work, focus on:
- Your own Subtask block (if you are already a contributor).
- Coordination Notes — anything addressed to you, or anything that changes the picture.
- The other contributors' subtask blocks — for shared context, read-only.

### Step 3: Validate freshness

For each task, compare the header `Last updated` against the current time:
- Older than **7 days** → add `⚠️ stale (last updated <date>)`.
- A single Subtask whose own `Last updated` is older than the rest by a long margin → flag as **possibly-stalled** (informational only — no takeover).

### Step 4: Present the summary

For `/dev-flow status` (no argument):

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
 dev-flow status — N active tasks
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

📋 Active
   • task_C_AUTH — concept — in-progress
     contributors: session-abc, session-def
   • task_20260520_143022_fix-login — fix — review-pending
     contributors: session-ghi
   ...

🕘 Recently completed (last 5)
   • task_SP_RATE_LIMITER — done 2026-05-18 — Spec finalized

📥 Deferred (todos) — 3 candidate
   • .dev_flow/todos/ — run `/dev-flow audit docs` to groom

⚠️ Warnings
   • task_… — stale (>7 days)
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

If `.dev_flow/todos/_index.md` exists, include the **Deferred (todos)** line with the count of `candidate` entries (omit the line when the register is absent or empty). `queued` follow-ups are not counted here — they surface at their originating task's completion, not in this view. It is a pointer, not a listing — `todo` files them, `audit` grooms them.

For `/dev-flow status <task_id>`:

```
📌 Current work item
   [Document name] — [pipeline phase] — [status]
   Traceable ID: [ID] — Contributors: [list] — Updated: [ts]

📝 Description (head)
   [first paragraph]

👥 Subtasks
   • [session-abc] Extract validator — in-progress — Next: Write tests
   • [session-def] Update tests — done

💬 Recent coordination
   • HH:MM [session-abc] — blocked on PasswordValidator interface
   • HH:MM [session-def] — will mock validator

⚠️ Blocking issues
   • [session-abc] need decision on public API breakage

🔗 Relevant context
   • Concept docs/auth.concept.md — added by session-abc
   • Spec docs/auth.sp.md — added by session-def
```

### Step 5: Offer continuation (`status` only)

This step belongs to the `status` report. [`resume`](#resume--re-enter-a-task-in-a-fresh-session) uses Steps 1–3 and then runs its **own** decision — it never falls into this step, or the two would argue over whether to ask.

After displaying a summary, ask the user:

> "Would you like to continue from where you left off,
> or start something new? (continue / new)"

If the user says **continue** — hand off to [`resume`](#resume--re-enter-a-task-in-a-fresh-session) on the chosen task (`/dev-flow resume <task_id>`): it reconciles the recorded state against the working tree before any work starts, and joins the task under the [write protocol](#when-a-phase-starts-on-a-task).

## Write Protocol

Every dev-flow command MUST update context using this protocol. The goal is **tolerance to other contributors**: an agent's writes must not destroy state another contributor has written.

**Working memory → durable (promote at a checkpoint).** Transient working state — the L1 notes / parameters / reminders / reads — is *not* written to task files on every step. At a checkpoint (a phase or subtask boundary, or an [Experience Capture](../references/experience-capture.md) checkpoint) **promote** the durable part — a settled decision, a confirmed parameter, a harvested lesson — into the task file (or an auto-applied rule/skill). The rest stays L1 scratch, acceptably lost on restart. See [Resource Cache → Session Working Memory](../references/cache.md#session-working-memory-l1).

### When a phase starts on a task

1. **Identify the task.** Is this a new task or continuation?
   - **New** — generate a Task ID:
     - If a traceable doc ID is known → `task_<TRACEABLE_ID>` (e.g. `task_C_AUTH`).
     - Otherwise → `task_YYYYMMDD_HHMMSS_<slug>`.
   - **Continuation** — read `active_context.md` to find the existing Task ID.
2. **Open or create the task file.**
   - New → write `tasks/task_<ID>.md` from the [task template](../templates/task_context.md). Add yourself to `Contributors`. Fill `## Intent` (goal / target state / expected result — [Task Intent](../references/task-intent.md)) and the header's `Autonomy` field — `full — "<quote>"` only when the request itself orders it, otherwise `checkpoints` ([SKILL.md → Developer Checkpoints](../SKILL.md#developer-checkpoints)). Add one `### Subtask:` block with you as `Author` and `Status: in-progress`. Set `Last updated` to now.
   - Existing → read the file. Then:
     - If your `<agent-id>` is already in `Contributors` and you have a Subtask block → resume it.
     - If you are not yet a contributor → add yourself to `Contributors` (targeted Edit on the header), append a new `### Subtask:` block at the end of the Subtasks section, and add a Coordination Notes entry like `HH:MM [your-id] — joined, starting on <goal>`.
3. **Update the dashboard.** Use `Edit` (not `Write`) on `active_context.md`:
   - If your task's row exists → update Phase / Status / Contributors / Updated.
   - If the row is missing (new task) → insert a new row in the Active Tasks table.
4. **Update the catalog** (`tasks/_index.md`) the same way: targeted `Edit`, single row.

### After completing a step

1. **In your own Subtask block:** check off the completed item in Progress, set the next item, append a one-line entry to your block's Activity list.
2. **In the task header:** targeted Edit on `Last updated`.
3. **Do not touch** the dashboard or catalog yet — step-level updates only live in the task file.

### After completing a phase

1. **In your own Subtask block:** update Status (`review-pending` / `done`), set final Activity entry.
2. **In the task header:** targeted Edit on `Status` (use the most active subtask's status, or "in-progress" if any subtask is still running).
3. **In the dashboard:** targeted `Edit` of your task's row — update Phase, Status, Updated.
4. **Append a Shared Activity Log entry** in the task file: `HH:MM [your-id] — subtask "<goal>" → <status>`.

### On task completion (all subtasks done)

1. **In the task file:** set the task-level `Status: done`. Append a final Shared Activity Log entry.
2. **In the dashboard:** targeted `Edit` — move your task's row from "Active Tasks" to "Recently Completed". If that list now exceeds 5 entries, move the oldest into `session_history/session_YYYY-MM-DD.md` and remove the row from the dashboard.
3. **In the catalog:** same — move from Active to Recently Completed.
4. **Surface queued follow-ups:** scan `.dev_flow/todos/` and plan backlogs for `queued` records triggered `after task_<this ID>` and offer to run each next (`/dev-flow do …`) — a suggestion, not an auto-run; the executed work still passes its own gates and commit approval. See [todo phase](todo.md).

### Targeted-edit safety

Before any `Edit` on `active_context.md`, `tasks/_index.md`, or shared sections of a task file:
1. **Re-read the file** (always, even if you read it a minute ago).
2. Locate your row / your block / your entry by its unique key (Task ID for index rows; Author tag for Subtask blocks; timestamp for log entries).
3. Apply the targeted edit (insert / update / remove your own content).
4. If the row or block you expect is not where you expect — the file changed. Re-locate by Task ID / Author tag and re-apply.

If two contributors update the same row at nearly the same time, the later write wins on that field; their subtask blocks and tagged entries are untouched.

### Append-only logs

Inside a task file, never rewrite Coordination Notes, Shared Activity Log, or per-subtask Activity. Always append (newest at top for log/notes, newest at bottom for per-subtask Activity within a block).

### How a contributor leaves a task

There is no "abandon" or "release ownership" operation. To leave gracefully:
1. Set your Subtask's `Status` to `done` (if finished) or `blocked` (if you cannot continue) or leave it `in-progress` (others may pick up).
2. Optionally add a Coordination Note: `HH:MM [your-id] — stepping away, <next-contributor-or-anyone> can continue from <here>`.
3. Do not remove yourself from `Contributors` — the historical record stays.

## Checkpoint — Fix the Task for a Session Boundary

A session ends for reasons that have nothing to do with the work: a context window fills, a usage limit is reached, a machine is closed, an agent is replaced. `checkpoint` is the developer-invoked act that makes the current task answerable **without the transcript**, so the next session restarts from files instead of from memory.

It is the [Transition Checkpoint](../references/experience-capture.md#transition-checkpoint--the-primary-cadence) run out of band — the developer's invocation *is* the transition — plus the two things a structural boundary does not do: it makes the documentation truthful, and it writes down where to start.

**"Checkpoint" in three senses.** The *command* is this section. The *Transition Checkpoint* is the reflection procedure it invokes in movement 1 — also fired at ordinary phase and subtask boundaries by the [write protocol](#write-protocol). A *developer checkpoint* is neither: it is a stop where the agent waits for the developer ([SKILL.md → Developer Checkpoints](../SKILL.md#developer-checkpoints)).

**When to run it.** Before ending a session with work still open; when [context pressure](../references/experience-capture.md#context-pressure--a-tiered-proxy-driven-trigger) reaches `recommend-handoff`; before handing a task to another contributor; before any long interruption.

**Before the movements.** If the dashboard is unparseable, run the [regeneration procedure](#regeneration-procedure) first; if the task file is over its line cap, run the [archive cycle](#session-history-archive) first; then checkpoint.

### The four movements

Run them in order — each one assumes the previous has landed.

1. **Distil.** Run the [Transition Checkpoint](../references/experience-capture.md#transition-checkpoint--the-primary-cadence) over the open segment — that reference owns the steps and their ordering.
2. **Reconcile the documentation.** Run the [propagate](propagate.md) drift check over the artifacts this task touched. Close the mechanical gaps in place; record anything larger as a Blocking Issue rather than leaving it for the next session to discover. A checkpoint never changes a document's lifecycle status.
3. **Make the task handoff-ready.** Walk the readiness set below and repair what is missing, by targeted edit inside your own regions only.
4. **Record the handoff.** Write the instruction line into the dashboard's `## Resume` section, refresh the derived rows (`Last updated`, dashboard, catalog), and echo the same line to the developer so the boundary is visible in the conversation as well as on disk. When a `note` was given, also append it verbatim as a Coordination Note tagged with your agent-id.

### Readiness set

The task file must answer *what / where / why / next* on its own. Walk these seven elements; a repair that needs a decision you cannot make becomes a Blocking Issue instead of a guess.

| # | Element | Where | Satisfied when | Repair |
|---|---------|-------|----------------|--------|
| R1 | Current work item | `## Current Work Item` | Document, pipeline phase, and traceable ID match what is actually in progress | Targeted Edit on the row |
| R2 | Intent | `## Intent` | Goal, target state, expected result present — or the task is a trivial route that legitimately has none | Fill from the request wording ([Task Intent](../references/task-intent.md)) |
| R3 | Next action | your Subtask `Progress` | Exactly one unchecked item is marked `**Next:**` and reads as an **action**, not a topic | Rewrite the item as an action |
| R4 | Blockers | `## Blocking Issues` | Every live blocker recorded; every resolved one marked resolved | Add or resolve your own entries |
| R5 | Decisions | task file or the affected document | No material fork unrecorded; `proposed` / `open` records name their state and trigger ([Interview Mode](../references/interview-mode.md)) | Record the fork; never resolve it by your own hand |
| R6 | Pointer set | `## Relevant Context` | Every document and file the next session must open is listed | Append rows tagged with your agent-id |
| R7 | Tree disposition | your Subtask `Activity` or a Coordination Note | Uncommitted work described, or the tree stated clean | One line naming branch and dirty/clean |

### The handoff record

One markdown list item per checkpointed active task, in a `## Resume` section of `.dev_flow/active_context.md` placed directly after the lead paragraph and before `## Active Tasks`. The section is omitted entirely when no active task has been checkpointed.

```markdown
- `/dev-flow resume task_C_AUTH` — `implement` — next: extract PasswordValidator from the login handler — branch `feat/auth`, 3 files modified — 2026-05-20 14:40
```

Order of parts: the exact invocation, the pipeline phase, the next **action**, the working tree as the checkpoint saw it, the developer's note when one was given, the timestamp.

Rules:
- **One record per Task ID** — a second checkpoint of the same task **replaces** its line; records never accumulate.
- **A resume does not consume it** — the record stays until a later checkpoint replaces it or [regeneration](#regeneration-procedure) drops it with its closed task. Re-entering a task twice reads the same record twice.
- The record is **derived data**: it repeats nothing that the task file plus the working tree do not already say, and it is rebuilt by the [regeneration procedure](#regeneration-procedure), never treated as a source of truth.
- It carries no secrets, no diffs, no command output, and no path under the `/tmp` workspace.

### What a checkpoint must not do

- **Never writes the working tree** — no commit, stash, checkout, or branch switch. The tree is read (branch + porcelain status) and described.
- **Never marks work done.** A checkpoint does not set a subtask or a task `done`, and does not advance a document's lifecycle status — finishing work is the finishing phase's act.
- **Writes only** inside `.dev_flow/` and the documentation artifacts movement 2's drift check names as stale. Anything outside that set is reported, not edited.
- **Never touches another contributor's** subtask block or tagged entries; a checkpoint of a task you do not own is announced in Coordination Notes, not written into their block.

### Checkpoint errors

| Code | Condition | What to do |
|------|-----------|-----------|
| `NO_CONTEXT` | `.dev_flow/` absent | Create it from the templates, proceed, and report this as the first checkpoint |
| `NO_OPEN_TASK` | No active task, and none owned by the caller | Report it, name the route that would open one, write nothing |
| `AMBIGUOUS_TASK` | Caller owns no subtask and several tasks are active | Ask which task to checkpoint; write nothing until answered |
| `UNREPAIRABLE` | A readiness element needs a decision the caller cannot make | Record it as a Blocking Issue, continue with the rest, name it in the report |

### Report

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
 checkpoint — task_C_AUTH
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
✏️  Repaired    R3 next action, R6 pointer set
📄  Docs        auth.sp.md §02 synced; concept drift → blocker [session-abc]
🎓  Harvested   rule naming/validator-suffix (should)
🌳  Tree        branch feat/auth — 3 files modified, nothing committed
▶️  Record      `/dev-flow resume task_C_AUTH` — `implement` — next: extract PasswordValidator
                from the login handler — branch `feat/auth`, 3 files modified — 2026-05-20 14:40
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

## Resume — Re-enter a Task in a Fresh Session

`resume` is the first action of a context that has no other information. It establishes where the work stands, continues it when the picture is unambiguous, and offers the next work when nothing is active.

It composes what this file already defines — the [read protocol](#read-protocol) for state and the [write protocol](#write-protocol) for joining — and hands continuation to the `do` phase's [Scenario A](do.md#scenario-a--continue-active-task). It is legal mid-session, not only in a fresh one: there it re-reads state and reports.

### The three movements

1. **Re-attend, then read.** Re-read the [session working memory](../references/cache.md#session-working-memory-l1) area first — it survives a compaction — then the dashboard's `## Resume` section, the dashboard tables, and the relevant task files through the [read protocol](#read-protocol), **Steps 1–3 only** — Step 4's summary block and Step 5's continuation question belong to `status`, and resume runs its own decision below instead.
2. **Reconcile against reality.** Compare the recorded state with the working tree: branch, uncommitted changes, and whether the artifacts the task claims to have produced exist. The record is a **claim**; the tree is the **evidence**. Emit one verdict — `matches` · `diverged (<what>)` · `unverifiable (<why>)` — and never silently trust a claim the tree contradicts.
3. **Decide and act.** Take exactly one branch:

| State | Branch | Action |
|-------|--------|--------|
| No `.dev_flow/` | — | Report that there is no context; suggest [onboard](onboard.md) or [concept](concept.md); write nothing |
| One task, unambiguous | `continued` | Continue it, announcing what was picked up |
| One task, ambiguous or diverged | `chosen` | Present the brief with the verdict, ask, then act on the answer |
| Several tasks | `chosen` | Present the briefs, ask which, then act on the answer |
| `task_id` names a closed task | `chosen` | Show it as closed (status, close date); offer to reopen or to start follow-up work explicitly; never auto-continue |
| No active task | `offered` | Present the work offer — offers, never an auto-run |
| No active task and nothing to offer | `empty` | Say so; suggest opening new work; invent nothing |

### The unambiguous test

Continue **without asking** only when *all* of these hold; otherwise present the brief and ask:

- Exactly one task is selected — one active task, or an explicit `task_id`.
- The task has a `Next` item that names an action.
- The task has **no live Blocking Issue**.
- The header's `Last updated` is inside the [freshness threshold](#step-3-validate-freshness).
- The reconciliation verdict is `matches`.

`Autonomy: full` does **not** bypass this test — autonomy governs [developer checkpoints](../SKILL.md#developer-checkpoints), not whether the recorded premise is still true.

### The resume brief

Bounded by construction: the readiness set read back, plus the verdict. Never the previous transcript, never a document's full text.

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
 resume — task_C_AUTH (continuing)
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
📌  Work item   auth.sp.md — spec — SP_AUTH
🎯  Intent      let users skip 2FA enrolment on first login
▶️  Next        extract PasswordValidator from the login handler
⚠️  Blockers    (rendered only when live — a task with one is never auto-continued)
🔗  Pointers    docs/auth.sp.md · docs/auth.plan.md · src/auth/login.ts
🌳  Tree        matches the record — branch feat/auth, 3 files modified
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

Continuation itself follows the [write protocol](#when-a-phase-starts-on-a-task): resume your own Subtask block if you have one, otherwise join the task as a new contributor with a fresh block and a Coordination Note.

### The work offer

When nothing is active, assemble the offer from two sources and present it **ordered**:

1. `queued` records in `.dev_flow/todos/` (and plan backlogs) whose trigger has passed — deferred *because* their context overlapped a task that has since closed.
2. Unfinished phases of plans whose `Status` is `in-progress` — work already started.
3. `candidate` records in `.dev_flow/todos/` — parked intent.

At most five entries; state the total when the list truncates. Each entry names its source, its reference, what the work is, the trigger it was waiting for (for a `queued` record — and whether it has passed), and the invocation that would start it. Nothing in the offer runs until the developer picks it — the executed work then passes its own gates and commit approval. See [todo](todo.md).

### Resume errors

| Code | Condition | What to do |
|------|-----------|-----------|
| `NO_CONTEXT` | `.dev_flow/` absent | Report it, suggest onboard or concept, write nothing |
| `UNKNOWN_TASK` | `task_id` given, no such file | List the active tasks and ask |
| `TASK_CLOSED` | `task_id` names a task with `Status: done` or one under `session_history/` | Show it as closed; offer reopen or follow-up work; never auto-continue |
| `STATE_MISMATCH` | The record contradicts the working tree | Present both, record a Blocking Issue, ask before acting |
| `TASK_BLOCKED` | The selected task has live Blocking Issues | Present them; never auto-continue |

## Regeneration Procedure

When the indexes look wrong (missing rows, stale entries, conflict markers, file corrupted), any contributor may rebuild them:

1. List `tasks/task_*.md`.
2. For each file, read the header (Task ID, Contributors, Status, Last updated, Current Work Item.Pipeline phase, title from `# Task: …`).
3. Rewrite `active_context.md`:
   - Active table from files with `Status` ∈ {in-progress, blocked, review-pending}.
   - Recently Completed from files with `Status: done`, sorted by Last updated desc, keeping the latest 5.
   - **`## Resume`** section from the existing handoff records: keep a record only while its task is in the rebuilt Active table, keep the latest record per Task ID, drop the rest. A record is never *invented* here — regeneration reproduces what a [checkpoint](#checkpoint--fix-the-task-for-a-session-boundary) wrote or omits the section entirely.
   - **Deferred (todos)** section from `.dev_flow/todos/_index.md` + plan backlogs: counts of `candidate`/`queued`, plus a flag line for any record bound to an already-closed plan/task. Omit the section if there are no todos.
4. Rewrite `tasks/_index.md` the same way.
5. Append a regeneration entry to the Shared Activity Log of one of your active task files (`HH:MM [your-id] — regenerated indexes`), or to a coordination task if you have none.

Regeneration is a full rewrite — only do it when targeted edits cannot recover the state. If you suspect another contributor is mid-write, prefer a targeted fix or coordinate first.

## Salience Markers

A salience marker lets the author of an entry record how much it matters, so dev-flow-owned compaction evicts by salience before age.

A marker attaches to a single **entry** — a Shared Activity Log line, a Coordination Note, a Relevant Context row, or a per-subtask Activity entry — never to a whole document (that axis is the document `Status`).

### Vocabulary

A marker is exactly one of a closed set; the unmarked majority is `normal`:

| Value | Meaning | Compaction effect |
|-------|---------|-------------------|
| `pin` | Must survive while the owning task is active | Never the first dropped; retained |
| `superseded` | Was relevant, now replaced by a later entry | Evicted first; carries a `→` pointer to its successor |
| `noise` | Service detail, never important | Evicted first |
| `normal` | Default — the unmarked majority | Age-ordered, exactly as before |

### Written form

A non-`normal` marker is a compact trailing token appended to the entry text:

- `{s:pin}` · `{s:noise}` · `{s:superseded→<entry-ref>}`
- `<entry-ref>` is a short human-resolvable locator (a dated log line's date + first words, or a section anchor).
- **Absence of any `{s:…}` token = `normal`** — `normal` is never written explicitly.
- At most **one** token per entry. The token is greppable (`{s:`) so compaction finds marked entries mechanically.

### Scope and expiry — markers are task-scoped

A marker has **no weight of its own**; its weight is conditioned on its owning task (the task file the entry lives in). The **effective salience** of an entry is:

- `normal` if the entry has no token, **or** if the owning task is closed / off the active dashboard / out of focus;
- otherwise the token's value.

So `pin` means "survive while this task is active" — not "survive forever". When the task closes or loses focus its markers go inert (effective `normal`). A durable lesson does **not** rely on a `pin` surviving closure — it is harvested into a rule/skill *before* the task closes (see [Experience Capture](../references/experience-capture.md), whose harvest-before-demote ordering runs the reflection before markers are demoted).

### How the protocol treats markers

- **Tag at write-time.** When you add a log line / note / row that is clearly durable (a settled decision, a segment summary) or clearly disposable (a routine service entry), append the matching token. Most entries stay `normal` — mark **sparingly**.
- **Re-grade by appending, never editing.** Logs are append-only, so you do not edit an entry to change its marker. To supersede an entry, append a new tagged note that references the original (`{s:superseded→…}` points at the successor).
- **`superseded` keeps the trail.** Never evict a `superseded` entry whose successor would also be gone — the successor must survive.
- **Compaction honours effective salience.** When this protocol's archiving or [audit](audit.md) Step 3 reduces a set of entries, a `pin` (of an active task) is retained, `noise`/`superseded` are evicted first, and only then does the existing age rule apply to the `normal` remainder. Evicted entries are **archived to `session_history/`, never lost** (the non-destructive rule still holds).
- **Over-pinning.** No hard cap on `pin` count; [audit](audit.md) flags a task whose `pin` ratio is implausibly high and proposes a re-grade (advisory).

**dev-flow compaction only.** Markers are honoured by dev-flow's *own* compaction (this protocol, audit). The **runtime's** context-summary is not a dev-flow-owned event — there a `pin` is at most conveyed as in-context phrasing (advisory). The durable copy in the task file is what dev-flow compaction honours.

## Context Hygiene

**Canonical hygiene caps.** This section is the single source of truth for the context-hygiene limits (activity-log entries, task-file/dashboard sizes). The [audit phase](audit.md) enforces them and SKILL.md summarizes them — both defer to the numbers here.

**Principle:** task files are "shared state as of now", not journals. Per-section logs are the only history kept in-place, and they are capped. When a cap forces eviction, eviction order is by **effective salience** first (see [Salience Markers](#salience-markers)) — `noise`/`superseded` go before `normal`, a `pin` of an active task is retained — and only then by age within the `normal` remainder.

### Hygiene rules

1. **Shared Activity Log:** keep at most **10 entries** per task file (newest first). When over the cap, evict `noise`/`superseded` first and then the oldest `normal` entries; a `pin` on an active task is kept. Move the evicted overflow to a session history file before appending.
2. **Per-subtask Activity:** same cap — ~10 entries per subtask block. When exceeded, archive that block's older entries.
3. **Activity content filter (canonical):** a Shared Activity Log or per-subtask Activity entry records only `incident` / `ambiguous-decision` / `structural-event` events (classes defined in [Docs Scaling](../references/docs-scaling.md)); an event outside these classes is not written (the TRIVIAL_ENTRY refusal) — progress lives in the Progress checklists.
4. **Description:** describes the active understanding. New paragraphs are additive (signed by contributor). When the description grows past ~3 paragraphs, consider consolidating into one paragraph in a Coordination Note discussion first.
5. **Subtask blocks:** completed (done) subtask blocks may be archived once the task file exceeds ~300 lines.
6. **Dashboard size:** if `active_context.md` exceeds ~80 lines, prune Recently Completed and archive overflow. Handoff records in `## Resume` count against the cap like any other content — one line per interrupted task, and the section disappears when the last one is dropped.
7. **No large blobs:** never store logs, diffs, full command output, or verbose narratives in any context file. Reference a file instead.

### Session history archive

When pruning, move the overflow into:

```
.dev_flow/session_history/session_YYYY-MM-DD.md
```

**Session history file format:**

```markdown
# Session History YYYY-MM-DD

## Archived task files

[Full content of completed task files moved here, separated by --- with the original filename.]

## Archived subtask blocks

[Completed subtask blocks moved here from active task files: which task,
the block content, the timestamp of archiving.]

## Archived log entries

[Activity log / coordination note overflow from active task files: which task,
which log, original timestamp, entry.]

## Session notes

[Optional: decisions, blockers resolved, lessons learned.]
```

**Archive procedure:**

1. Create `.dev_flow/session_history/` if needed.
2. Create or append to `session_YYYY-MM-DD.md` (today's date).
3. Move pruned content into the file under the appropriate section.
4. Add a reference at the bottom of the affected task file: `Full history: ../session_history/session_YYYY-MM-DD.md`.
5. Set the file's `Last updated` timestamp.

**Do not** delete session history files automatically — they are the audit trail.
