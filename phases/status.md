# Phase: Status — Restore Session Context

## Purpose

Load the active development context into the session so you can quickly resume where you (or other contributors) left off, without re-reading all documents from scratch.

Status also defines the **read/write protocol** that every other phase follows when touching the context files — read this whenever you need to update task state safely under multiple AI contributors.

For the periodic whole-directory revision — reconciling task state with reality, trimming the dashboard, compacting and reflecting on closed tasks, and grooming `rules/`/`skills/` — see the [audit phase](audit.md). Where `status` *reports* drift, `audit` *resolves* it, building on this protocol's regeneration and archive procedures.

## Command

```
/dev-flow status [task_id]
```

- No argument — show all active tasks (dashboard summary).
- Optional `task_id` — show the detailed state for one specific task file.

## Context Files

```
.dev_flow/
├── active_context.md          # Dashboard — table of active tasks + recently completed
├── tasks/
│   ├── _index.md              # Catalog of task files
│   ├── task_<ID>.md           # Per-task shared context (multiple contributors)
│   └── ...
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

**Re-attention first (after a compaction or subtask switch).** Before re-reading the task files, re-read the **session working memory** area — the L1 notes / parameters / reminders that survive a compaction (see [Resource Cache → Session Working Memory](../references/cache.md#session-working-memory-l1)). It is small and whole-re-readable, and it restores the working focus the dropped transcript held. The dashboard and task files (below) remain the durable source of truth; working memory just rebuilds *attention* cheaply.

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

### Step 5: Offer continuation

After displaying a summary, ask the user:

> "Would you like to continue from where you left off,
> or start something new? (continue / new)"

If the user says **continue** — proceed as `/dev-flow do continue` on the chosen task. The orchestrator will resume your own Subtask block if you have one, or add a new one if not.

## Write Protocol

Every dev-flow command MUST update context using this protocol. The goal is **tolerance to other contributors**: an agent's writes must not destroy state another contributor has written.

**Working memory → durable (promote at a checkpoint).** Transient working state — the L1 notes / parameters / reminders — is *not* written to task files on every step. At a checkpoint (a phase or subtask boundary, or an [Experience Capture](../references/experience-capture.md) checkpoint) **promote** the durable part — a settled decision, a confirmed parameter, a harvested lesson — into the task file (or an auto-applied rule/skill). The rest stays L1 scratch, acceptably lost on restart. See [Resource Cache → Session Working Memory](../references/cache.md#session-working-memory-l1).

### When a phase starts on a task

1. **Identify the task.** Is this a new task or continuation?
   - **New** — generate a Task ID:
     - If a traceable doc ID is known → `task_<TRACEABLE_ID>` (e.g. `task_C_AUTH`).
     - Otherwise → `task_YYYYMMDD_HHMMSS_<slug>`.
   - **Continuation** — read `active_context.md` to find the existing Task ID.
2. **Open or create the task file.**
   - New → write `tasks/task_<ID>.md` from the [task template](../templates/task_context.md). Add yourself to `Contributors`. Add one `### Subtask:` block with you as `Author` and `Status: in-progress`. Set `Last updated` to now.
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

If two contributors update the same row at nearly the same time (e.g. both update Last updated in a header), the later write wins on that single field — that is acceptable because the field's content is interchangeable. The contributors' own subtask blocks and tagged entries are untouched.

### Append-only logs

Inside a task file, never rewrite Coordination Notes, Shared Activity Log, or per-subtask Activity. Always append (newest at top for log/notes, newest at bottom for per-subtask Activity within a block). This means a contributor that mistakenly opens a stale copy and appends an entry still produces a valid log — at worst with one duplicated line, never with lost history.

### How a contributor leaves a task

There is no "abandon" or "release ownership" operation. To leave gracefully:
1. Set your Subtask's `Status` to `done` (if finished) or `blocked` (if you cannot continue) or leave it `in-progress` (others may pick up).
2. Optionally add a Coordination Note: `HH:MM [your-id] — stepping away, <next-contributor-or-anyone> can continue from <here>`.
3. Do not remove yourself from `Contributors` — the historical record stays.

## Regeneration Procedure

When the indexes look wrong (missing rows, stale entries, conflict markers, file corrupted), any contributor may rebuild them:

1. List `tasks/task_*.md`.
2. For each file, read the header (Task ID, Contributors, Status, Last updated, Current Work Item.Pipeline phase, title from `# Task: …`).
3. Rewrite `active_context.md`:
   - Active table from files with `Status` ∈ {in-progress, blocked, review-pending}.
   - Recently Completed from files with `Status: done`, sorted by Last updated desc, keeping the latest 5.
   - **Deferred (todos)** section from `.dev_flow/todos/_index.md` + plan backlogs: counts of `candidate`/`queued`, plus a flag line for any record bound to an already-closed plan/task. Omit the section if there are no todos.
4. Rewrite `tasks/_index.md` the same way.
5. Append a regeneration entry to the Shared Activity Log of one of your active task files (`HH:MM [your-id] — regenerated indexes`), or to a coordination task if you have none.

Regeneration is a full rewrite — only do it when targeted edits cannot recover the state. If you suspect another contributor is mid-write, prefer a targeted fix or coordinate first.

## Salience Markers

Compaction is where context is lost, and dev-flow archives by **age** — oldest first, whether or not it still matters. Salience markers let the author of an entry record *how much it matters*, so a dev-flow-owned compaction keeps the signal and sheds the noise instead of going purely by age.

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

So `pin` means "survive while this task is active" — not "survive forever". When the task closes or loses focus its markers go inert (effective `normal`); this is what stops completed tasks clogging the active context. A durable lesson does **not** rely on a `pin` surviving closure — it is harvested into a rule/skill *before* the task closes (see [Experience Capture](../references/experience-capture.md), whose harvest-before-demote ordering runs the reflection before markers are demoted).

### How the protocol treats markers

- **Tag at write-time.** When you add a log line / note / row that is clearly durable (a settled decision, a segment summary) or clearly disposable (a routine service entry), append the matching token. Most entries stay `normal` — mark **sparingly**.
- **Re-grade by appending, never editing.** Logs are append-only, so you do not edit an entry to change its marker. To supersede an entry, append a new tagged note that references the original (`{s:superseded→…}` points at the successor).
- **`superseded` keeps the trail.** Never evict a `superseded` entry whose successor would also be gone — the successor must survive.
- **Compaction honours effective salience.** When this protocol's archiving or [audit](audit.md) Step 3 reduces a set of entries, a `pin` (of an active task) is retained, `noise`/`superseded` are evicted first, and only then does the existing age rule apply to the `normal` remainder. Evicted entries are **archived to `session_history/`, never lost** (the non-destructive rule still holds).
- **Over-pinning is self-correcting.** There is no hard cap on `pin` count; [audit](audit.md) flags a task whose `pin` ratio is implausibly high and proposes a re-grade (advisory).

**dev-flow compaction only.** Markers are honoured by dev-flow's *own* compaction (this protocol, audit). The **runtime's** context-summary is not a dev-flow-owned event — there a `pin` is at most conveyed as in-context phrasing (advisory). The durable copy in the task file is what dev-flow compaction honours.

## Context Hygiene

**Canonical hygiene caps.** This section is the single source of truth for the context-hygiene limits (activity-log entries, task-file/dashboard sizes). The [audit phase](audit.md) enforces them and SKILL.md summarizes them — both defer to the numbers here.

**Principle:** task files are "shared state as of now", not journals. Per-section logs are the only history kept in-place, and they are capped. When a cap forces eviction, eviction order is by **effective salience** first (see [Salience Markers](#salience-markers)) — `noise`/`superseded` go before `normal`, a `pin` of an active task is retained — and only then by age within the `normal` remainder.

### Hygiene rules

1. **Shared Activity Log:** keep at most **10 entries** per task file (newest first). When over the cap, evict `noise`/`superseded` first and then the oldest `normal` entries; a `pin` on an active task is kept. Move the evicted overflow to a session history file before appending.
2. **Per-subtask Activity:** same cap — ~10 entries per subtask block. When exceeded, archive that block's older entries.
3. **Description:** describes the active understanding. New paragraphs are additive (signed by contributor). When the description grows past ~3 paragraphs, consider consolidating into one paragraph in a Coordination Note discussion first.
4. **Subtask blocks:** completed (done) subtask blocks may be archived once the task file exceeds ~300 lines.
5. **Dashboard size:** if `active_context.md` exceeds ~80 lines, prune Recently Completed and archive overflow.
6. **No large blobs:** never store logs, diffs, full command output, or verbose narratives in any context file. Reference a file instead.

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

## Roles

- Each phase role updates its own subtask block as it runs.
- **ContextTracker** ([context-tracker.ai.md](../roles/context-tracker.ai.md)) is the dedicated read/write/regenerate worker; invoke it when context needs refreshing without executing a phase.
