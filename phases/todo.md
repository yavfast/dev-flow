# Phase: Todo — Capture Future Work for Later

## Purpose

Take a free-form description of work to do **later**, find the documentation it would touch, judge whether it is doable, and **file a planning record** with a return trigger. It does not build anything and does not pass through the pipeline gates.

"Later" has two flavors — `todo` handles both:

- **Deferred (speculative)** — work the project *might* do; YAGNI-gated; trigger is a future event or date; may end up dropped. ("колись додамо WebSocket")
- **Queued follow-up (committed)** — a concrete fix/change *noticed while working on the current task* that must wait until that task finishes, because doing it now would interfere (overlapping context corrupts the work in progress). It is **not** speculative and is never YAGNI-dropped; its trigger is the completion of the originating task (`after task_<ID>`). This is the case a parallel [subtask](subtask.md) cannot cover — subtask needs disjoint file scope, but here the contexts overlap, so the work must be *sequenced after*, not parallelized.

Which flavor applies — and how urgent the work is — is **determined by `todo` from context** (the state of the relevant plans and tasks), **not parsed from the request's wording**: the description may carry no timing or task-binding words at all. The analysis may even conclude the work is urgent enough to *not* defer, and recommend running it now instead.

`todo` is invoked two ways: **by the developer** (`/dev-flow todo …`), or **by an agent itself** — when, mid-work, it spots a defect or problem that does *not* belong to its current task and whose fix can wait for a later session, it files an agent-initiated `todo` rather than derailing into it or losing it. One needing analysis runs in a **cheap subagent** (Step 0) so the discovery is captured without eroding focus; a trivial finding that needs no analysis the agent files inline.

`todo` is the deferred-capture sibling of the routing band: `ask` analyzes and writes nothing; `todo` analyzes and files for later; `do` analyzes and acts now.

## Command

```
/dev-flow todo <description>
```

### Examples

```
# deferred (speculative)
/dev-flow todo колись додати WebSocket до gateway без ламання REST
/dev-flow todo maybe later: rate-limit the export endpoint

# queued follow-up (noticed during the current task, run it after)
/dev-flow todo помітив витік у CacheManager.close() — виправити після поточної задачі
/dev-flow todo after this task: the retry helper double-counts attempts, fix it
```

## Role Responsible

This command is handled by **TodoPlanner**: [roles/todo-planner.ai.md](../roles/todo-planner.ai.md)

## Constraints

- **Files only a planning record** — a `[backlog]` item in an owning plan, or an entry in `.dev_flow/todos/`. No code, no concept/spec/plan authoring, no pipeline phases.
- **No commits, no git.**
- **No edits to other contributors' subtask blocks** or their tagged entries.
- **No triggerless deferral** — every record names a return trigger (date, event, task completion, or revisit cadence).
- **Capture-time analysis is preliminary** — it decides *where* to store and gives a head-start; the full analysis happens at execution, because context drifts.

## Procedure

### Step 0: Delegation check

Decide where this `todo` runs, by invocation path:

- **Agent-initiated** (an agent spotted an out-of-scope, deferrable defect during its own work) — if any analysis is needed, spawn a **cheap subagent** with just the problem description to execute this phase ([subtask](subtask.md), `todo-planner` role, low-cost model); the agent does not pause its task. Only the conclusion (what was filed and where) returns. A **trivial** finding that needs no analysis (clear placement, no doc search) the agent files inline — a register line — without a subagent. This is the inverse of [Delegation for Focus](../references/delegation.md): there you delegate noisy *current* work, here an *unrelated discovery* so it is captured without derailing the task in hand.
- **Developer-invoked** — if this session already owns an in-progress task in `.dev_flow/active_context.md`, **delegate to a subagent** for the same focus reason (subtask, or Delegation for Focus when only the verdict is wanted). Otherwise run inline.

Eligibility for an agent-initiated `todo`: the problem is **not part of the current task** *and* its fix is **deferrable to a later session**. If it is part of the current task, or urgent, the normal rules apply — fix it now or escalate ([Upstream Escalation](../references/escalation.md)); don't park real in-scope work as a todo. (A *forecast* about the current change goes to [Consequence-Forecasting](../references/consequence-forecasting.md) `drop+record`; an *observed unrelated defect* goes here.)

### Step 1: Load context (read-only) + gates

1. Read `.dev_flow/active_context.md` and any task file whose area matches — read only.
2. **Skill check (gate).** MUST read `.dev_flow/skills/_index.yaml`; load matching skills — the analysis MUST reflect them. See [skill phase](skill.md).
3. **Rule check (gate).** When `.dev_flow/rules/` exists, MUST read `.dev_flow/rules/_index.yaml`; the feasibility assessment MUST honour loaded rules (flag a `must` the idea would violate). See [rule phase](rule.md).

### Step 2: Find relevant documentation & state

Search `docs/` (concepts `*.concept.md`, specs `*.sp.md`, plans `*.plan.md`), `docs/_glossary.md`, and the code for what the idea touches. Reuse the [Impact Walk](../references/impact.md) to bound the radius — do not re-derive it. Also read the **state** of what it touches: is there an **active plan** for the area (and an open phase/backlog that would absorb it)? Is there an **in-progress task** whose files/area **overlap** the idea?

**Dedup.** Check the existing `.dev_flow/todos/` register, plan backlogs, and `.dev_flow/rules/` for a matching item. If one exists, update/merge it instead of filing a duplicate; if it is already a known rule, do not file at all.

Output: the linked relevant docs, the owning-plan question, the overlap question, and any existing record to merge into.

### Step 3: Assess for placement, then determine urgency, binding & trigger

A **quick at-capture assessment** — enough to place the record and leave a head-start, **not** the execution analysis (that re-runs at pickup, because context drifts):

- **Feasibility (at capture)** — feasible / with caveats / not feasible, in a line; flag a `must`-rule it would violate.
- **Scope estimate** — likely [Change Class](do.md#change-classes) (Trivial / Standard / Architectural).
- **Suggested phase** — where pickup should route (`fix` / `implement` / `spec` / …).
- **Context snapshot** — one or two lines of why-it-matters + the key files/docs, to speed the re-analysis later.

Then **derive timing and binding from context — never parse them from the request's wording** (the description may carry no timing words at all). Classify from the Step 2 state:

| Context found | Flavor / outcome | Trigger |
|---------------|------------------|---------|
| **Overlaps an in-progress task** — doing it now would corrupt that work | **queued** follow-up; record the originating task and why it must wait | `after task_<ID>` |
| **An active plan owns the area** with an open phase/backlog | bind to that plan (file into its backlog); urgency follows the plan's schedule | the plan event/phase that returns it to scope |
| **Docs/tasks signal it is urgent** — a `must`-rule violation, a blocking defect, a plan phase already waiting on it | do **not** defer — surface a recommendation to run it now (`/dev-flow do …` or `fix`) instead of filing | n/a (act now) |
| **None of the above** — a free-standing concept/idea | a **candidate** deferred to preserve focus on current work | a soft revisit cadence (next planning / when bandwidth frees); sitting in the register is a normal backlog state — audit may graduate a matured candidate into documentation |

Trigger discipline: **never a bare "later"**, but the trigger need not be a hard date — a context-derived event, a task completion, or a revisit cadence all qualify. For a *candidate*, run the [Consequence-Forecasting](../references/consequence-forecasting.md) YAGNI-gate (`build now` / `seam+flag` / `drop+record`) to decide the trigger and whether to file at all. A *queued* follow-up skips the gate — it is committed work, not speculation.

### Step 4: File the planning record

If Step 3 concluded the work is **urgent (act now)**, do not file a record — report the recommendation to run it now and stop. Otherwise file one record, picking the home by one rule:

| Condition | Where the record goes |
|-----------|-----------------------|
| An **owning plan exists** for the area | Append a `[backlog]` item to that plan's `## Backlog` section (see [plan phase → Backlog](plan.md)), carrying the return trigger. |
| **No owning plan** (concept/spec may or may not exist) | Add an entry to `.dev_flow/todos/_index.md` (create from [templates/todo_index.md](../templates/todo_index.md) on first use), cross-linking any related concept/spec. Spill heavy notes to `<ID>.md`. |

**Record id** — `TD_<YYYYMMDD_HHMMSS>_<slug>` (timestamp + 1–3-word kebab slug), mirroring task naming so concurrent contributors never collide. No sequential numbering.

Every record carries: id, description, relevant-doc links, at-capture feasibility, scope estimate, **suggested phase**, **context snapshot**, return trigger, and a status — `candidate` (deferred) or `queued` (committed follow-up; also records the originating task it waits behind) → later `promoted` / `dropped`.

**Leave a dashboard trace.** Targeted-edit `.dev_flow/active_context.md`'s **Deferred (todos)** section (create it from the [template](../templates/active_context.md) on first use) so the fact of the addition is visible at the entry point — refresh the `candidate · queued` counts. **Always** add an explicit flag line when the record bound to an already-**closed** plan or task (a backlog item in a `completed` plan, or a change to a `done`/archived task's area): that target is off the active view, so without this line the deferral would be invisible. Per-item detail stays in the register; the dashboard carries only counts + closed-target flags.

### Step 5: Report

State what was filed and where (the plan backlog item, or the `TD_…` register entry), note the dashboard trace, and suggest the command that would execute it later:

> "Filed as `TD_20260623_143000_cache-leak` in `.dev_flow/todos/` and noted on the dashboard. When ready, run `/dev-flow do <description>` to pick it up."

Promotion is **out of scope** of `todo` — a later `do` or `plan` run picks the record up and marks it `promoted`. On pickup it **re-runs a full analysis** (the record's at-capture assessment + context snapshot is a head-start, not a trusted final verdict — context may have drifted). A **`queued` follow-up surfaces automatically when its originating task completes**: the task-completion step lists every record triggered `after task_<ID>` and offers to run it next (a suggestion, not an auto-run — the executed fix still goes through its own gates and commit approval). See the task-completion surfacing in [do phase → Session wrap-up](do.md#step-7-session-wrap-up) and [status phase](status.md). [Audit](audit.md) grooms the register — see [audit Step 7b](audit.md).

## Language Policy

Respond in the **same language** the user used in their request.
