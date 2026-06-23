# Todos Register

This directory holds **deferred work captured by `/dev-flow todo`** — ideas the project might execute later, each filed with the documentation it touches, a feasibility verdict, and a return trigger. It is the home for a todo only when **no owning plan exists** for the area; when a plan exists, the item lives in that plan's `## Backlog` section instead.

A todo is *deferred*, not committed work — execution happens later via `/dev-flow do <description>`, which **re-runs a full analysis** (the entry below is a head-start, not a trusted final verdict — context drifts) and marks the entry `promoted`. [Audit](../phases/audit.md) grooms this register.

## Conventions

- **ID:** `TD_<YYYYMMDD_HHMMSS>_<slug>` (timestamp + 1–3-word kebab slug), mirroring task naming so concurrent contributors never collide. Immutable once assigned.
- **Status:** `candidate` (deferred/speculative) → `promoted` / `dropped` (YAGNI or declined); `queued` (committed follow-up waiting on a task) → `promoted` / `dropped` only explicitly with a reason — **never** dropped as speculative.
- **Return trigger:** every entry names a condition that brings it back into scope — a **date**, a **named event**, a **task completion** (`after task_<ID>`, for a queued follow-up), or a **revisit cadence** (for a candidate). No triggerless "later".
- **Capture-time analysis is preliminary** — it places the record and leaves a head-start; the full analysis re-runs at execution.
- **Suggested phase:** where pickup should route (`fix` / `implement` / `spec` / …).
- **Heavy notes** spill to a sibling `<ID>.md`; keep the index entry compact and link to it.
- This catalogue is **human-browsed** → `_index.md` (per the index-format split). Regenerable from any spill files; the entries here are the source of truth otherwise.

## Register

| ID | Description | Scope | Phase | Trigger | Status |
|----|-------------|-------|-------|---------|--------|
| TD_20260101_090000_short-slug | short description | Standard | implement | when X happens / by YYYY-MM-DD | candidate |
| TD_20260102_140000_cache-leak | fix noticed during current work | Trivial | fix | after task_C_AUTH | queued |

## Entries

### TD_20260101_090000_short-slug — <short title>

> **Status:** candidate — **Created:** YYYY-MM-DD

- **Idea:** [the future work, in 1–2 sentences]
- **Relevant docs:** [`C_XXX`](../../docs/xxx.concept.md), [`SP_XXX`](../../docs/xxx.sp.md) — or "none yet"
- **Feasibility (at capture):** feasible / with caveats / not feasible — one line; re-checked at execution
- **Scope / suggested phase:** Trivial / Standard / Architectural — `implement` / `fix` / `spec`
- **Context snapshot:** [why it matters + key files/docs — a head-start for the re-analysis]
- **Return trigger:** [date, named event, or revisit cadence — derived from plan/task state, not from how the request was phrased]

### TD_20260102_140000_cache-leak — <queued follow-up title>

> **Status:** queued — **Created:** YYYY-MM-DD — **Waits on:** `task_<ID>`

- **Fix:** [the defect noticed while working on `task_<ID>`, in 1–2 sentences]
- **Why deferred:** context overlaps `task_<ID>` — fixing now would interfere; run it after.
- **Relevant docs / suggested phase:** [links] — `fix`
- **Context snapshot:** [key files + the symptom — a head-start for the re-analysis]
- **Return trigger:** `after task_<ID>`

---

*Created and groomed by `/dev-flow todo` and `/dev-flow audit`. See [../phases/todo.md](../phases/todo.md).*
