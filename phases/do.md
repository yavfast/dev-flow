# Phase: Do — Freeform Intent Routing

## Purpose

Accept any natural-language request and route it to the appropriate dev-flow phase.
The agent interprets intent from context, asks clarifying questions when needed,
and executes the correct phase sequence automatically.

This is the **default command** — any invocation of `/dev-flow` without a recognized
phase keyword falls through to `do`.

## Command

```
/dev-flow do <freeform request>

# or equivalently (default fallback):
/dev-flow <freeform request>
```

### Examples

```
/dev-flow do continue
/dev-flow do додай кнопку Skip на формі авторизації
/dev-flow do the login form needs a password strength indicator
/dev-flow do update the spec for rate limiting
/dev-flow do I need to refactor the auth module — where do I start?
```

## Role Responsible

This command is handled by **DevFlowOrchestrator**:
[roles/dev-flow-orchestrator.ai.md](../roles/dev-flow-orchestrator.ai.md)

## Procedure

### Step 1: Load context

1. Read `.dev_flow/active_context.md` (the dashboard) if it exists. Identify
   which task this request relates to:
   - User said "continue" → resume the most recently updated active task that
     already lists your session in `Contributors`. If only one active task
     exists, resume it.
   - User described a specific item → match against active task titles/IDs.
   - No match → this is a new task; a new `tasks/task_<ID>.md` will be created
     in Step 6.

2. For the chosen task (continuation), read its file `.dev_flow/tasks/task_<ID>.md`:
   - Current Work Item (document, phase, traceable ID)
   - Description
   - **Your own Subtask block** (if you are already a Contributor) — pick up
     from its `Next` step. If you are not yet a contributor on this task, you
     will add a new Subtask block in Step 6.
   - Other contributors' Subtask blocks — read for shared context, do not edit.
   - Recent Coordination Notes — anything addressed to you or that changes the picture.
   - Any open Blocking Issues.

3. If neither the dashboard nor any task file exists, note this — the orchestrator
   will ask the user to describe the goal from scratch.

4. **Skill check (gate).** MUST read `.dev_flow/skills/_index.yaml`; load matching skills
   before routing and pass them as context to the executing phase. Save new research to
   `.dev_flow/skills/` after completion. See [skill phase](skill.md).

5. **Rule check (gate).** When `.dev_flow/rules/` exists, MUST read `.dev_flow/rules/_index.yaml`
   and load rules for the request area; pass them to the executing phase, which MUST comply
   (`must` = blocks). See [rule phase](rule.md).

### Step 2: Interpret the request

Analyze the freeform request against the loaded context to determine:

| Intent type | Indicators | Routed to |
|-------------|-----------|-----------|
| **Continue** | "continue", "resume", "де зупинились", active context has a next step | Resume active task |
| **Add/change feature** | UI element, field, behavior description, "додай", "зміни", "add", "change" | spec → plan → implement |
| **New feature or idea** | Broad new capability, no existing documents match | concept → spec → plan → implement |
| **Fix documentation** | "update docs", "propagate", "оновити специфікацію" | propagate |
| **Fix a bug** | "fix", "виправи", "bug", "баг", "падає", "crash", "NPE", "не працює", error description | fix |
| **Review/validate** | "check", "validate", "чи правильно", "review" | review |
| **Manage rules** | "add rule", "додай правило", "edit rule", "remove rule", "list rules", "show rule" | rule |
| **Manage cached resources** | "cache", "закешуй", "збережи макет/скріншот/документ", "save this export", "find cached" | inline cache management — [Resource Cache](../references/cache.md) |
| **Understand state** | "що зроблено", "status", "де я?" | status |
| **Revise / housekeep context** | "ревізія", "audit", "почисти контекст", "синхронізуй задачі", "оновити індекси", "groom", "tidy up", "compact", "retrospective" | audit |
| **Delegate side task** | "subtask", "delegate", "делегуй", "зроби паралельно", "offload", explicit secondary task during active work | subtask |
| **Question (no changes)** | "how does", "як працює", "can we", "чи можливо", "where is", "де знаходиться", "is it feasible" — answerable from codebase + docs | ask |
| **Research / investigate** | "research", "дослідити", "spike", "compare approaches", "порівняй підходи", "не знаю, який підхід", "what's the best way to" — answer needs external sources, measurements, or an unexplored solution space | research |
| **Plan only** | "plan", "сплануй", no code changes mentioned | plan |

When the intent is **ambiguous**, ask 1–2 targeted clarifying questions before routing.
Do **not** start executing before the intent is clear.

If the request requires knowledge nobody has yet (unfamiliar domain, unverified
library capability, unknown solution space) — route through [research](research.md)
*first*, then continue to the design phases with the findings.

### Change Classes

For change requests (not questions/research), classify the change **before** routing —
this is what scales the pipeline ceremony to the size of the change, in one explicit
decision instead of per-phase skip rules:

| Class | What it is | Route |
|-------|-----------|-------|
| **Trivial** | No behavior change: typo, comment, log message, rename with no contract impact | implement → test (if suite exists) → review → commit approval. Skip concept/spec/plan edits; pre-commit review stays a clean-context subagent ([review phase](review.md)) — it just has little to read |
| **Standard** | Behavior changes within an existing spec'd area: new field, changed validation, UI element | spec → plan → implement → full Test/Review/Verify pipeline |
| **Architectural** | New capability, new entity, changed mechanism or boundary | concept → spec → plan → implement → full pipeline |
| **Internal refactor** | Structure changes, contracts identical | [Refactoring Protocol](plan.md#refactoring-protocol) (plan-only workflow) |

When unsure between two classes, take the heavier one — under-classifying is how
drift starts. When the class hinges on how far the change reaches, run the
[Impact Walk](../references/impact.md) — the radius (docs / code bindings / active
tasks) is the evidence. The [propagation matrix](propagate.md#change-type-propagation-matrix)
remains the per-document authority on what must be updated; change classes decide
where the route *starts*.

### Step 3: Ask clarifying questions (if needed)

Only ask questions that are genuinely blocking routing or execution.
Use prior context to fill obvious gaps silently.

**Question patterns:**
- Scope: "Does this change an existing feature or is it new?" 
- Document: "Do you have an existing concept/spec for this? (e.g., `auth.concept.md`)"
- Breaking change: "Is this a breaking change to an existing API/contract?"
- Priority: "Should this go into the current plan phase or start a new plan?"

Limit to **maximum 3 questions** per invocation. If still ambiguous — propose the
most reasonable interpretation and ask for confirmation:
> "I'll treat this as: updating `auth.sp.md` and creating a new plan phase for
> the Skip button. Does that sound right?"

### Step 4: Route and execute

After confirming intent, invoke the appropriate dev-flow phase(s) in order:

#### Scenario A — Continue active task

1. Re-read the active task (document + next step from context).
2. Resume execution: load the relevant documents and continue from **Next step**.
3. Update context after completing each step.

#### Scenario B — Small targeted change (one area of pipeline)

1. Identify which pipeline layer needs updating (concept / spec / plan / code).
2. Execute phases from that layer forward, following normal gate checks.
3. Example: "add Skip button to auth form" →
   - Read `auth.sp.md` (ask user to confirm or provide path)
   - Ask: does concept need to change? (usually no for small UI tweaks)
   - Update `auth.sp.md` — add Skip button interaction contract
   - Gate: spec → plan
   - Update `auth.plan.md` — add task for Skip button
   - Gate: plan → implement
   - Implement the change
   - Run tests if suite exists
   - Present for commit approval

#### Scenario C — New feature (no existing documents)

1. Confirm: "No existing concept found for this. I'll start from concept phase."
2. Execute: `/dev-flow concept` → `/dev-flow spec` → `/dev-flow plan` → `/dev-flow implement`
3. Follow all gate checks between phases.

#### Scenario D — Documentation only

1. Execute `/dev-flow propagate` or `/dev-flow review` as appropriate.

### Step 5: Check documentation impact

After code changes are implemented (in Scenarios A, B, or C), verify whether
related documentation artifacts need updating:

1. **Specifications** — does the change alter behavior described in any `*.sp.md`?
   (new fields, changed contracts, different error handling, new UI interactions).
2. **Concepts** — does the change affect architectural assumptions in any `*.concept.md`?
   (new integration points, changed mechanisms, expanded scope).
3. **Plans** — does the change complete or invalidate tasks in any `*.plan.md`?
4. **Tests** — do existing tests need updating, or should new test cases be added
   to cover the new/changed behavior?

If updates are needed:
- For small doc changes — apply them as part of the current phase.
- For significant doc changes — run `/dev-flow propagate` before presenting for commit.
- Always report which documents were updated (or flagged for update) in the result.

### Step 6: Update context

After every phase step, in `.dev_flow/tasks/task_<ID>.md`:
- In **your own Subtask block** (the one whose `Author` is your session):
  check off completed steps in Progress, set the next step, append a one-line
  entry to that block's Activity bullet list.
- In the **task header**: refresh `Last updated` (targeted Edit on that field).

If this step also crosses a **phase boundary** (e.g. spec completed, plan starts):
- Update your Subtask's `Status` (e.g. `done`, `review-pending`).
- Targeted Edit on `.dev_flow/active_context.md` and `.dev_flow/tasks/_index.md`
  to update your task's row (Phase, Status, Contributors, Updated).
- Append a Shared Activity Log entry tagged `[your-id] — <event>`.

Re-read each index file immediately before editing it — see
[status phase: Targeted-edit safety](status.md#targeted-edit-safety).

Never rewrite another contributor's Subtask block or their tagged entries in
shared sections. To respond to or build on another contributor's work, add
your own Coordination Note tagged with your session id.

### Step 7: Session wrap-up

If the user ends the session (or after completing a full phase chain):
1. In **your own Subtask block** — set `Status` (`review-pending` / `done` /
   `blocked`), append a wrap-up entry to its Activity list. Optionally add a
   Coordination Note about hand-off (`[your-id] — stepping away, anyone may
   pick up from <here>`).
2. In the task **header** — refresh `Last updated`. If all Subtasks across all
   contributors are `done`, set the task-level `Status: done`.
3. Targeted Edit on the dashboard and catalog:
   - If task-level `Status: done` → move your task's row from "Active Tasks"
     to "Recently Completed".
   - Otherwise → update Status / Updated columns in place.
4. Run hygiene checks (Shared Activity Log cap, per-subtask Activity cap, file
   size) and archive overflow to `.dev_flow/session_history/` if triggered —
   see [status phase](status.md).

## Routing Decision Tree

```
User request received
│
├─ "continue" / "resume" / no new info
│   └─ Read dashboard → open task file → resume your own Subtask block
│
├─ Describes UI/API/behavior change
│   ├─ Small (affects 1–2 spec sections)
│   │   └─ spec → plan → implement
│   └─ Large (affects architecture)
│       └─ concept → spec → plan → implement
│
├─ Describes documentation update
│   └─ propagate / review
│
├─ Question about state / what's done
│   └─ status
│
├─ Manage a project catalogue (coding rules / knowledge skills / cached resources)
│   └─ rule / skill / inline cache management ([Resource Cache](../references/cache.md))
│
├─ Revise / clean up .dev_flow (reconcile state, trim context, dedupe rules/skills/cache)
│   └─ audit
│
├─ Question about code / feasibility (no changes requested)
│   ├─ Answerable from codebase + docs
│   │   └─ ask
│   └─ Needs external sources / experiments / unknown solution space
│       └─ research (time-boxed spike)
│
├─ Knowledge missing to even start a concept
│   └─ research → then concept with the findings
│
├─ Side task during active work
│   └─ subtask (delegate to subagent)
│
└─ Unclear
    └─ Ask 1–2 clarifying questions, then re-route
```

## Language Policy

The orchestrator responds in the **same language** the user used in their request.
It does not impose a language — if the user writes in Ukrainian, respond in Ukrainian.
