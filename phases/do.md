# Phase: Do — Freeform Intent Routing

## Contents

- [Purpose](#purpose) — Route any freeform request to the right phase; `do` is the default fallback for `/dev-flow` without a phase keyword
- [Command](#command) — `/dev-flow do <request>` and the equivalent bare `/dev-flow <request>` syntax, with example invocations
- [Role Responsible](#role-responsible) — DevFlowOrchestrator role file that handles this command
- [Procedure](#procedure) — Steps 1–7: load context + gates, interpret intent (intent table, Change Classes), clarify, route Scenarios A–D, wrap-up
- [Batch of Tasks](#batch-of-tasks) — two or more tasks in one request: disjoint file sets, one design sign-off, one commit sign-off table, one commit per approved row
- [Routing Decision Tree](#routing-decision-tree) — ASCII tree from request shape to target phase: resume, checkpoint, change, fix, docs, status, catalogues, audit, ask, adopt, todo
- [Output Style](#output-style) — Chat vs documentation registers and the Style gate the orchestrator passes to the executing phase

## Purpose

Accept any natural-language request and route it to the appropriate dev-flow phase.

This is the **default command** — any invocation of `/dev-flow` without a recognized phase keyword falls through to `do`.

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

This command is handled by **DevFlowOrchestrator**: [roles/dev-flow-orchestrator.ai.md](../roles/dev-flow-orchestrator.ai.md)

## Procedure

### Step 1: Load context

1. Read `.dev_flow/active_context.md` (the dashboard) if it exists. Identify which task this request relates to:
   - User said "continue" → resume the most recently updated active task that already lists your session in `Contributors`. If only one active task exists, resume it.
   - User described a specific item → match against active task titles/IDs.
   - No match → this is a new task; a new `tasks/task_<ID>.md` will be created in Step 6, and the [upstream look](status.md#upstream-look) runs before Step 4 routes into the first design or code step.
   - The dashboard shows an open outcome check whose trigger is due → offer to reconcile it ([Outcome check](../references/task-intent.md#outcome-check)); run it on the developer's yes, then continue with the request.

2. For the chosen task (continuation), read its file `.dev_flow/tasks/task_<ID>.md`:
   - Current Work Item (document, phase, traceable ID)
   - Description
   - **Your own Subtask block** (if you are already a Contributor) — pick up from its `Next` step. If you are not yet a contributor on this task, you will add a new Subtask block in Step 6.
   - Other contributors' Subtask blocks — read for shared context, do not edit.
   - Recent Coordination Notes — anything addressed to you or that changes the picture.
   - Any open Blocking Issues.

3. If neither the dashboard nor any task file exists, note this — the orchestrator will ask the user to describe the goal from scratch.

4. **Skill check (gate).** MUST read `.dev_flow/skills/_index.yaml`; load matching skills before routing and pass them as context to the executing phase. Save new research to `.dev_flow/skills/` after completion. See [skill phase](skill.md).

5. **Rule check (gate).** When `.dev_flow/rules/` exists, MUST read `.dev_flow/rules/_index.yaml` and load rules for the request area; pass them to the executing phase, which MUST comply (`must` = blocks). See [rule phase](rule.md).

6. **Ticket check.** If the request **explicitly** names a tracker ticket (a `--ticket PROJ-123` flag, a "Jira PROJ-123"-style mention, or a ticket URL), run [Ticket Tracker Integration](../references/ticket-tracker.md): discover the project's tracker skill/MCP, pull the ticket to seed the task's real requirements, and write the `Ticket:` link into the task file's Current Work Item. A bare `KEY-123`-looking token does **not** activate it; every outward write to the tracker is confirmed first.

### Step 2: Interpret the request

**Capture the intent first.** If the request itself orders an unattended run ("автономно", "без зупинок"), set the task header `Autonomy: full — "<quote>"`; a harness or system autonomy setting does not ([SKILL.md → Developer Checkpoints](../SKILL.md#developer-checkpoints)). Before classifying, extract the [Task Intent](../references/task-intent.md) — the goal (why), target state, and expected result — from the request wording, active context, and any linked ticket. Distinguish the *requested action* from the *underlying goal*; mark inferred parts `(inferred)`. Record it in the task file's `## Intent` section (in Step 6 for a new task). Skip the record for trivial routes with self-evident intent.

**Two or more tasks in one request** → [Batch of Tasks](#batch-of-tasks).

Analyze the freeform request against the loaded context to determine:

| Intent type | Indicators | Routed to |
|-------------|-----------|-----------|
| **Continue / re-enter** | "continue", "resume", "продовжуй", "де зупинились", a fresh session with no new information, active context has a next step | [resume](status.md#resume--re-enter-a-task-in-a-fresh-session) — it establishes state first, then continues via Scenario A |
| **Fix the state before leaving** | "checkpoint", "збережи стан", "зафіксуй контекст", "закінчую сесію", "save the session", "hand this over", "I'm hitting the context limit" | [checkpoint](status.md#checkpoint--fix-the-task-for-a-session-boundary) |
| **Add/change feature** | UI element, field, behavior description, "додай", "зміни", "add", "change" | spec → plan → implement |
| **New feature or idea** | Broad new capability, no existing documents match | concept → spec → plan → implement |
| **Fix documentation** | "update docs", "propagate", "оновити специфікацію" | propagate |
| **Fix a bug** | "fix", "виправи", "bug", "баг", "падає", "crash", "NPE", "не працює", error description | fix |
| **Review/validate** | "check", "validate", "чи правильно", "review" | review |
| **Commit** | "commit", "закоміть", "коміть" | The developer's yes to the commit sign-off — valid when the commit sign-off was held on the current tree; otherwise run what is missing (review, verify) and hold the sign-off; in a [batch](#batch-of-tasks) it answers no row — ask which. It answers the commit question only, not the push |
| **Manage rules** | "add rule", "додай правило", "edit rule", "remove rule", "list rules", "show rule" | rule |
| **Manage skills** | "add skill", "update skill", "додай знання", "збережи знання про X", "capture what we learned" | skill |
| **Manage tools** | "tool", "інструмент", "збережи скрипт як інструмент", "покажи інструменти", "перевір інструменти", "save this script as a tool", "list tools", "verify tools", "regenerate tool" | [tool](tool.md) |
| **Manage cached resources** | "cache", "закешуй", "збережи макет/скріншот/документ", "save this export", "find cached" | inline cache management — [Resource Cache](../references/cache.md) |
| **Understand state** | "що зроблено", "status", "де я?" | status |
| **Revise / housekeep context** | "ревізія", "audit", "почисти контекст", "синхронізуй задачі", "оновити індекси", "groom", "tidy up", "compact", "retrospective", "прибери зайві тести", "prune agent tests" | audit |
| **Delegate side task** | "subtask", "delegate", "делегуй", "зроби паралельно", "offload", explicit secondary task during active work | subtask |
| **Question (no changes)** | "how does", "як працює", "can we", "чи можливо", "where is", "де знаходиться", "is it feasible" — answerable from codebase + docs | ask |
| **Capture for later** | "todo", "колись", "на майбутнє", "maybe later", "capture this idea", "потім" (deferred); or "виправити після поточної", "fix after this task", "помітив … але не зараз", "after this is done" (queued follow-up noticed mid-task, contexts overlap) — work to file, not execute now | todo |
| **Research / investigate** | "research", "дослідити", "spike", "compare approaches", "порівняй підходи", "не знаю, який підхід", "what's the best way to" — answer needs external sources, measurements, or an unexplored solution space; "prototype first", "спершу прототип", "спробуй прототипом" → scope `prototype` ([Prototype in the Project](research.md#prototype-in-the-project)) | research |
| **Learn from an external repo** | "adopt", "analyze repo", "розбери репозиторій", "проаналізуй репо", "що взяти з X", "what can we borrow from", "which ideas from X are useful here" — a *named external repository* (path or URL) is the subject | adopt — [External Repo Adoption](../references/repo-adoption.md) |
| **Plan only** | "plan", "сплануй", no code changes mentioned | plan |

When the intent is **ambiguous**, ask targeted clarifying questions before routing (capped at Step 3). Do **not** start executing before the intent is clear.

If the request requires knowledge nobody has yet (unfamiliar domain, unverified library capability, unknown solution space) — route through [research](research.md) *first*, then continue to the design phases with the findings.

### Change Classes

For change requests (not questions/research), classify the change **before** routing:

| Class | What it is | Route |
|-------|-----------|-------|
| **Trivial** | No behavior change: typo, comment, log message, rename with no contract impact | implement → test (if suite exists) → review → **commit sign-off**. No design sign-off (no design documents). Skip concept/spec/plan edits; pre-commit review stays a clean-context subagent ([review phase](review.md)) — it just has little to read |
| **Standard** | Behavior changes within an existing spec'd area: new field, changed validation, UI element | spec → plan → **design sign-off** → implement → full Test/Review/Verify pipeline → **commit sign-off** |
| **Architectural** | New capability, new entity, changed mechanism or boundary | concept → spec → plan → **design sign-off** → implement → full pipeline → **commit sign-off** |
| **Internal refactor** | Structure changes, contracts identical | [Refactoring Protocol](plan.md#refactoring-protocol) (plan-only workflow — carries its own **design sign-off** and **commit sign-off**) |

When unsure between two classes, take the heavier one. When the class hinges on how far the change reaches, run the [Impact Walk](../references/impact.md) — the radius (docs / code bindings / active tasks) is the evidence. The [propagation matrix](propagate.md#change-type-propagation-matrix) remains the per-document authority on what must be updated; change classes decide where the route *starts*.

### Step 3: Ask clarifying questions (if needed)

Only ask questions that are genuinely blocking routing or execution. Use prior context to fill obvious gaps silently.

**Question patterns:**
- Goal: "What should this achieve / what problem does it solve?" — only when a material routing or design choice depends on an intent you cannot infer ([Task Intent](../references/task-intent.md))
- Scope: "Does this change an existing feature or is it new?"
- Document: "Do you have an existing concept/spec for this? (e.g., `auth.concept.md`)"
- Breaking change: "Is this a breaking change to an existing API/contract?"
- Priority: "Should this go into the current plan phase or start a new plan?"

Limit to **maximum 3 questions** per invocation. If still ambiguous — propose the most reasonable interpretation and ask for confirmation:
> "I'll treat this as: updating `auth.sp.md` and creating a new plan phase for
> the Skip button. Does that sound right?"

### Step 4: Route and execute

After confirming intent, invoke the appropriate dev-flow phase(s) in order:

#### Scenario A — Continue active task

This is also where [`resume`](status.md#resume--re-enter-a-task-in-a-fresh-session) lands once it has established state: resume reconciles and decides, Scenario A executes.

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
   - Design sign-off — DEC batch, changed docs, files to touch, proportionality block; wait ([SKILL.md → Developer Checkpoints](../SKILL.md#developer-checkpoints))
   - Implement the change
   - Run the task's tests if a suite exists ([Test Economy](../references/test-economy.md#when-to-run))
   - Review, then commit sign-off — changed files, review verdict, intent verdict, proportionality block, unpushed commits; "Ready to commit?" and offer the push as its own question ([SKILL.md → Developer Checkpoints](../SKILL.md#developer-checkpoints))

#### Scenario C — New feature (no existing documents)

1. Confirm: "No existing concept found for this. I'll start from concept phase."
2. Execute: `/dev-flow concept` → `/dev-flow spec` → `/dev-flow plan` → **design sign-off** → `/dev-flow implement` → … → review → **commit sign-off**
3. Follow all gate checks between phases; the checkpoints are skipped only under `Autonomy: full` in the task header ([SKILL.md → Developer Checkpoints](../SKILL.md#developer-checkpoints)); a needed [access channel](../references/access-channels.md) sanction or `reauth-required` stops only that line of work.

#### Scenario D — Documentation only

1. Execute `/dev-flow propagate` or `/dev-flow review` as appropriate.

### Step 5: Check documentation impact

After code changes are implemented (Scenarios A, B, C), check the affected artifacts — specs, concepts, plans, tests — against the [change-type propagation matrix](propagate.md#change-type-propagation-matrix).

If updates are needed:
- For small doc changes — apply them as part of the current phase.
- For significant doc changes — run `/dev-flow propagate` before presenting for commit.
- Always report which documents were updated (or flagged for update) in the result.

### Step 6: Update context

For a **new task**, set the header `Autonomy` field and fill the task file's `## Intent` section with what Step 2 captured (goal / target state / expected result) — it is the reference every later check compares against ([Task Intent](../references/task-intent.md)). If the user restates the goal mid-task, update the section and re-check open work against it.

Then follow the [status write protocol](status.md#write-protocol) for step-end and phase-boundary updates — your own Subtask block, the task header, the dashboard and catalog rows, re-reading each index immediately before the targeted edit.

Never rewrite another contributor's Subtask block or their tagged entries in shared sections. To respond to or build on another contributor's work, add your own Coordination Note tagged with your session id.

### Step 7: Session wrap-up

If the user ends the session (or after completing a full phase chain):
1. **Intent verdict (on completion).** When reporting the task done or stopping at the commit sign-off, compare the outcome to the task's `## Intent` (Expected result) and state `intent: met / partially met / diverged (+why)` in the report — a divergence is surfaced, never silently absorbed. At the sign-off of the task's last commit, an intent with an [Outcome check](../references/task-intent.md#outcome-check) adds `— outcome pending (<trigger>)`. Skip for a mid-task hand-off. See [Task Intent](../references/task-intent.md).
2. Run the [status write protocol → on task completion](status.md#write-protocol) — Subtask `Status` and wrap-up Activity entry, header `Last updated` (task-level `Status: done` only when every contributor's subtask is done), the dashboard and catalog row move, the `unverified` tools named with the `/dev-flow audit tools` offer, the outcome entry of a task whose intent carries an Outcome check, and the [hygiene caps](status.md#context-hygiene) with overflow archived to `.dev_flow/session_history/`. Optionally add a hand-off Coordination Note (`[your-id] — stepping away, anyone may pick up from <here>`).
3. **Surface queued follow-ups.** If the task became `done`, scan `.dev_flow/todos/` and plan backlogs for `queued` records triggered `after task_<this ID>` (fixes deferred *because their context overlapped this task*). List each and offer to run it next via `/dev-flow do …` — a suggestion, not an auto-run; the executed work passes its own gates and commit approval. See [todo phase](todo.md).

## Batch of Tasks

A request that names two or more tasks forms a batch: each checkpoint is held once for all members, and every commit stays separate.

**Form.**

1. Open each named task as its own task — file, Intent, change class, route.
2. Establish each member's file set: from its design documents and the [Impact Walk](../references/impact.md); for a Trivial change, the files to edit.
3. Two members' file sets intersect → the task named later in the request leaves the batch and runs after it. An intersection found during the work → the task that reached the shared file second leaves.
4. Fewer than two members remain → no batch.
5. Record the members in each member's Coordination Notes. The [upstream look](status.md#upstream-look) runs once for the batch; the overlap is reported per member.
6. Run the members in sequence, or as [parallel subtasks](subtask.md#parallel-subtasks) where they are independent.

**Design sign-off** — once, before the first code edit of any member. Present each member's decisions, documents, files, and proportionality rows; a member with no design documents presents nothing and starts without an answer, and a batch with no design documents holds no design stop. The developer answers per member; any other member with no answer does not start.

**Commit sign-off** — once, when each member has passed review or is blocked. A blocked member is not in the table and gets its own sign-off later. One row per member: task · files · review verdict with open advisories · intent verdict · open proportionality rows. The developer answers per row; "all" is valid only when the developer says it. The unpushed commits are shown once, and the push is offered once, as its own question, with the table; it runs after the approved rows are committed ([SKILL.md → Git Workflow Integration](../SKILL.md#git-workflow-integration)).

- approved row → one commit, with its own message and IDs, staged from its own file list;
- declined row → stays uncommitted and returns to its own loop;
- row with no answer → no commit;
- two approved rows share a file → neither is committed; run the two tasks in sequence.

Never one commit for several tasks; never an inferred yes. Under `Autonomy: full` the batch holds no stops. A subagent holds no checkpoint and makes no commit.

## Routing Decision Tree

```
User request received
│
├─ "continue" / "resume" / no new info
│   └─ resume → read dashboard + task file, reconcile against the tree,
│              continue when unambiguous, else ask; nothing active → offer work
│
├─ "checkpoint" / "save the state" / "ending the session" / context limit reached
│   └─ checkpoint → distil, close doc drift, satisfy the readiness set,
│                  write the handoff record into the dashboard
│
├─ Describes UI/API/behavior change
│   ├─ Small (affects 1–2 spec sections)
│   │   └─ spec → plan → design sign-off → implement → … → commit sign-off
│   └─ Large (affects architecture)
│       └─ concept → spec → plan → design sign-off → implement → … → commit sign-off
│
├─ Reports a defect ("fix", "виправи", "падає", crash, error description)
│   └─ fix
│
├─ "commit" / "закоміть"
│   └─ commit sign-off held on the current tree → commit; else run what is
│      missing (review, verify), then hold the sign-off; batch → ask which row
│
├─ Describes documentation update
│   └─ propagate / review
│
├─ Question about state / what's done
│   └─ status
│
├─ Manage a project catalogue (coding rules / knowledge skills / project tools / cached resources)
│   └─ rule / skill / [tool](tool.md) / inline cache management ([Resource Cache](../references/cache.md))
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
├─ A named external repository is the subject ("what's worth taking from X")
│   └─ adopt
│
├─ Work to file for later (deferred idea, or a fix noticed mid-task to run after it)
│   └─ todo
│
├─ Knowledge missing to even start a concept
│   └─ research → then concept with the findings
│
├─ Side task during active work
│   └─ subtask (delegate to subagent)
│
└─ Unclear
    └─ Ask clarifying questions (Step 3 cap), then re-route
```

## Output Style

The orchestrator answers in the `chat` register and writes documents in the `documentation` register. See [Output Styles](../references/output-styles.md) — it also carries the **Style gate** (`.dev_flow/output_styles.md`) the orchestrator passes to the executing phase alongside rules, skills and the [profile](../references/adaptation-profile.md).
