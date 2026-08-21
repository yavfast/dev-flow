# Task: [title]

> **Task ID:** `task_<ID>` (filename without `.md`)
> **Created:** YYYY-MM-DD HH:MM
> **Last updated:** YYYY-MM-DD HH:MM
> **Status:** `in-progress` (one of: `in-progress` / `blocked` / `review-pending` / `done`)
> **Contributors:** `<agent-id-1>`, `<agent-id-2>`, ...

## Current Work Item

| Field | Value |
|-------|-------|
| **Document** | `[type: concept/spec/plan/code]` — [title and file path] |
| **Pipeline phase** | `[onboard / research / concept / spec / plan / implement / test / review / verify / propagate / fix / rule / skill / ask / todo / do / subtask / status / audit / adopt]` |
| **Traceable ID** | `[C_XXX / SP_XXX / PL_XXX / E_XXX or n/a]` |
| **Ticket** | `[PROJ-123 (external tracker) or n/a]` — present only when the task is explicitly tied to a tracker ticket; see `references/ticket-tracker.md` (dev-flow skill) |

## Intent

<!-- Captured at intake by whoever opens the task; user's terms, inferred parts
     marked `(inferred)`. Changes only when the user restates the goal.
     Skip for trivial routes with self-evident intent.
     See `references/task-intent.md` (dev-flow skill). -->

- **Goal (why):** [what problem this solves / why the user wants it]
- **Target state:** [how things should look when done]
- **Expected result:** [the observable outcome the user will check]

## Description

[Shared. What this task is about overall, written as the team understands it. Each contributor may append a paragraph signed `— <agent-id>` to extend the description. **Do not rewrite paragraphs another contributor wrote.**]

## Subtasks

A subtask is a thread of work claimed by one contributor. To join this task, add a new `### Subtask:` block below. **Only the block's author modifies it.** Reading other contributors' blocks is encouraged — they are shared context.

### Subtask: <short title>
> Author: `<agent-id>` — Created: HH:MM — Last updated: HH:MM — Status: `in-progress`

**Goal:** [what this subtask covers in 1–2 sentences]

**Progress:**
- [x] Completed step
- [ ] **Next:** [what to do next — specific and actionable]
- [ ] Remaining step

**Activity:**
<!-- Content filter (see references/docs-scaling.md): only incidents, ambiguous decisions,
     and structural events. A success report or interim status is not written (TRIVIAL_ENTRY) —
     progress lives in the Progress checklist above. -->
- HH:MM — [what changed in one line]

<!-- Add more `### Subtask:` blocks below as other contributors join.
     If a subtask becomes stale and another contributor wants to continue
     that line of work, they add a *new* subtask block — they do NOT edit
     the original block. -->

## Review Rounds

<!-- Written by the review phase at the CLOSE of each round, so the next round can
     scope from it instead of re-reading the whole diff. Omit the section until a
     first round has closed. See `references/review-convergence.md` (dev-flow skill). -->

| Round | Scope | Baseline | Verdict | Carry-over (identity → recurrence) |
|-------|-------|----------|---------|-------------------------------------|
| 1 | full (no-baseline) | `<tree-ref>` | FAIL | `src/x.ts:41#logic:null` → 1 |

**Always in scope:** [areas declared `Criticality: critical` — read every round regardless of the delta, or "—"]

## Coordination Notes

<!-- Append-only conversation between contributors. Prefix each note with
     [agent-id]. Newest at top. Use for handoffs, pings, decisions. -->

- HH:MM [agent-id] — [note, e.g. "starting on validator extraction"]

## Blocking Issues

<!-- Each issue tagged with [reporter-agent-id]. Only the reporter marks
     their own issue resolved. Other contributors may comment via
     Coordination Notes. -->

[No blockers yet.]

## Relevant Context

| Type | Name / Path | Note (added by) |
|------|-------------|-----------------|
| Concept | `docs/xxx.concept.md` | [why it's relevant — `<agent-id>`] |
| Spec | `docs/xxx.sp.md` | [relevant sections — `<agent-id>`] |
| Cache | `.dev_flow/cache/figma/xxx.png` | [what it shows — `<agent-id>`] |

<!-- Each row tagged with the contributor who added it. Rows are additive;
     do not remove or rewrite rows added by others. -->

## Shared Activity Log

<!-- Task-level events: subtask created/done, status changed, contributor
     joined, regenerated. Newest first. Cap at 10 entries — archive overflow
     to .dev_flow/session_history/session_YYYY-MM-DD.md.
     Content filter (see references/docs-scaling.md): only incidents, ambiguous
     decisions, and structural events — a trivial progress report is not written. -->

- HH:MM [agent-id] — created task

---

*This is a shared file. Each contributor owns their own subtask block and their own tagged entries in shared sections (Description paragraphs, Coordination Notes, Blocking Issues, Relevant Context rows, Activity Log entries). Do not refactor others' content. Coordinate via Coordination Notes. See `phases/status.md` (dev-flow skill) for the full protocol.*
