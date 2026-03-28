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
