# Phase: Status — Restore Session Context

## Purpose

Load the active development context into the session so you can quickly resume
where the previous session left off, without re-reading all documents from scratch.

## Command

```
/dev-flow status
```

No arguments. Reads `.dev_flow/active_context.md` and presents a structured
session-resume summary.

## Active Context File

All dev-flow commands maintain a persistent state file at:

```
.dev_flow/active_context.md
```

This file is the single source of truth for session continuity. It is:
- **Written** at the end of every dev-flow phase (after completing any command).
- **Read** at the beginning of `status` and `do` commands.
- **Created from template** when it does not yet exist (first run).

Template: [templates/active_context.md](../templates/active_context.md).

## Procedure

### Step 1: Read active context

1. Check if `.dev_flow/active_context.md` exists.
   - If **not** — inform the user: no active context found. Suggest running
     `/dev-flow onboard` (existing project) or starting with `/dev-flow concept`.
     Stop here.
   - If **yes** — read the file completely.

### Step 2: Validate context freshness

- Check the **Last updated** timestamp.
- If the file is older than 7 days, add a `⚠️ Context may be stale (last updated: <date>)` warning.

### Step 3: Present the summary

Output a structured, human-readable summary:

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
 dev-flow status
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

📌 Current work item
   [Document name] — [pipeline phase] — [status]
   Traceable ID: [ID]

📝 Task
   [Current task description]

✅ Done
   [x] completed steps from Progress State

⏭ Next step
   [first unchecked item from Progress State]

🔗 Relevant files
   [list of linked documents/files from context]

⚠️  Blocking issues (if any)
   [items from Blocking Issues section]
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

### Step 4: Offer continuation

After displaying the summary, ask the user:

> "Would you like to continue from where you left off,
> or start something new? (continue / new)"

If the user says **continue** — proceed as `/dev-flow do continue`
(the orchestrator reads context and resumes the active task).

## Context Update Rules

Every dev-flow command MUST update `.dev_flow/active_context.md` upon completion:

| When | What to update |
|------|---------------|
| Start of any phase | Set **Current Work Item**, **Pipeline phase**, **Status** = in-progress |
| After completing a task step | Check off the step in **Progress State** |
| When blocked | Add entry to **Blocking Issues** |
| After completing a phase | Set **Status** = done, update **Next** step |
| End of session | Set **Last updated** timestamp |

Roles responsible for updating context:
- **Each phase role** updates the relevant fields when it runs.
- **ContextTracker** ([context-tracker.ai.md](../roles/context-tracker.ai.md)) can be
  used as a standalone updater when context needs refreshing without executing a phase.

## Context Hygiene

**Principle:** `active_context.md` is "state as of now", not a cumulative journal.
After a task is completed, keep a short summary and remove unnecessary details.

### Hygiene rules

1. **Recent Changes:** keep at most **10 entries** (newest first). When adding a new
   entry would exceed 10, move the oldest entries to a session history file before adding.
2. **Current Task:** must describe only the **active** work item. When a task completes
   and a new one starts, compress the old task description to a one-line summary
   in Recent Changes — do not accumulate prior task narratives in Current Task.
3. **Progress State:** when switching to a new task, archive completed progress
   from the prior task. Only the current task's checklist should be in Progress State.
4. **File size:** if `active_context.md` exceeds ~150 lines, trigger an archive cycle.
5. **No large blobs:** never store logs, diffs, full command output, or verbose
   per-phase narratives directly. Reference a file instead.

### Session history archive

When pruning active_context, move detailed history to:

```
.dev_flow/session_history/session_YYYY-MM-DD.md
```

**Session history file format:**

```markdown
# Session History YYYY-MM-DD

## Archived from active_context

[Moved entries, with dates preserved]

## Session Notes

[Optional: decisions, blockers resolved, lessons learned]
```

**Archive procedure:**

1. Create `.dev_flow/session_history/` directory if it does not exist.
2. Create or append to `session_YYYY-MM-DD.md` (today's date).
3. Move pruned Recent Changes entries into the file under `## Archived from active_context`.
4. Compress Current Task if it contains history from completed tasks.
5. Add a reference in active_context: `Full history: .dev_flow/session_history/session_YYYY-MM-DD.md`
6. Update `Last updated` timestamp.

**Do not** delete session history files automatically. They serve as an audit trail.
