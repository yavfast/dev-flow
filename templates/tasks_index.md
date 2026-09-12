# Tasks Directory Index

<!-- Instantiated as `.dev_flow/tasks/_index.md`; relative links below resolve from that directory. -->

Each file here is the shared working space for one research topic, feature, or fix; several agents may contribute in parallel.

The parent file [`../active_context.md`](../active_context.md) is a thin dashboard derived from these files; if it ever drifts, this directory wins.

## Conventions

**Filename:**
- `task_<TRACEABLE_ID>.md` when the task is tied to a concept/spec/plan (e.g. `task_C_AUTH.md`, `task_SP_RATE_LIMITER.md`, `task_PL_PAYMENTS.md`).
- `task_YYYYMMDD_HHMMSS_<slug>.md` otherwise. The slug is 1–3 kebab-case words (e.g. `task_20260520_143022_refactor-auth.md`).

**Collaboration model:** a task is a shared file. Each contributor owns one **Subtask block** and its own tagged entries, appends to shared sections, and **must not rewrite another contributor's content**. Coordination happens via the file's **Coordination Notes**. Full rules: `phases/status.md` → Collaboration Model (dev-flow skill).

**Lifecycle:**
- A task is created with one Subtask (the creator's work).
- Other contributors join by adding a new Subtask block. They are also added to the task's `Contributors` field.
- Each Subtask is marked `done` by its author when finished.
- When all Subtasks are `done`, any contributor may set the task's overall `Status: done`.
- Completed tasks stay here for ~30 days, then move to `../session_history/`.

**No exclusive locks** and no time-based takeover. To continue a stale line of work, add a *new* Subtask block referencing the original; the original stays untouched as a historical record.

## Active Tasks

<!-- Mirror of active rows in active_context.md. Add via targeted edit. -->

- [task_…](task_….md) — `<contributors>` — `<phase>` — short title

## Recently Completed

<!-- Latest 5. Older ones archive to ../session_history/. -->

- [task_…](task_….md) — done YYYY-MM-DD — short outcome

---

*Tasks index is regenerable. Any dev-flow command may rebuild it by listing `task_*.md` files in this directory. See `phases/status.md` (dev-flow skill).*
