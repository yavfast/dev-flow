# Dev-Flow Active Context

This file is a **dashboard** — a thin index over the task files in
[`tasks/`](tasks/). Detailed per-task state lives in those files; this file
only lists what is active and what has recently completed.

Any contributor may rebuild this dashboard from `tasks/*.md` if it becomes
stale or inconsistent. See [phases/status.md](../phases/status.md).

## Active Tasks

<!-- Targeted edits only — add/update/remove a single row at a time,
     keyed by Task ID. Sort by Updated, newest first. -->

| Task | Phase | Status | Contributors | Updated |
|------|-------|--------|--------------|---------|
| [task_…](tasks/task_….md) — short title | concept | in-progress | session-abc, session-def | YYYY-MM-DD HH:MM |

## Recently Completed

<!-- Keep the latest 5. Older completed tasks move to .dev_flow/session_history/. -->

| Task | Phase | Completed | Contributors | Result |
|------|-------|-----------|--------------|--------|
| [task_…](tasks/task_….md) — short title | review | YYYY-MM-DD | session-abc | short outcome |

## Notes

[Cross-task observations, coordination notes that span multiple tasks,
or session-wide blockers. Leave empty if none.]

---

*Dashboard maintained by dev-flow commands. Each contributor updates only
their own row context (e.g., adds itself to Contributors when joining a task).
Hygiene: keep under ~80 lines; rebuild from `tasks/` when in doubt.*
