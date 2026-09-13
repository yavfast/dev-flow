# Phase: Do — Freeform Intent Routing

## Contents

- [Purpose](#purpose) — Route any freeform request to the right phase; `do` is the default fallback for `/dev-flow` without a phase keyword
- [Command](#command) — `/dev-flow do <request>` and the equivalent bare `/dev-flow <request>` syntax, with example invocations
- [Role Responsible](#role-responsible) — DevFlowOrchestrator role file that handles this command
- [Procedure](#procedure) — Steps 1–7: load context + gates, interpret intent (intent table, Change Classes), clarify, route Scenarios A–D, wrap-up
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
   - No match → this is a new task; a new `tasks/task_<ID>.md` will be created in Step 6.

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
| **Manage rules** | "add rule", "додай правило", "edit rule", "remove rule", "list rules", "show rule" | rule |
| **Manage skills** | "add skill", "update skill", "додай знання", "збережи знання про X", "capture what we learned" | skill |
| **Manage cached resources** | "cache", "закешуй", "збережи макет/скріншот/документ", "save this export", "find cached" | inline cache management — [Resource Cache](../references/cache.md) |
| **Understand state** | "що зроблено", "status", "де я?" | status |
| **Revise / housekeep context** | "ревізія", "audit", "почисти контекст", "синхронізуй задачі", "оновити індекси", "groom", "tidy up", "compact", "retrospective" | audit |
| **Delegate side task** | "subtask", "delegate", "делегуй", "зроби паралельно", "offload", explicit secondary task during active work | subtask |
| **Question (no changes)** | "how does", "як працює", "can we", "чи можливо", "where is", "де знаходиться", "is it feasible" — answerable from codebase + docs | ask |
| **Capture for later** | "todo", "колись", "на майбутнє", "maybe later", "capture this idea", "потім" (deferred); or "виправити після поточної", "fix after this task", "помітив … але не зараз", "after this is done" (queued follow-up noticed mid-task, contexts overlap) — work to file, not execute now | todo |
| **Research / investigate** | "research", "дослідити", "spike", "compare approaches", "порівняй підходи", "не знаю, який підхід", "what's the best way to" — answer needs external sources, measurements, or an unexplored solution space | research |
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
   - Design sign-off — DEC batch, changed docs, files to touch; wait ([SKILL.md → Developer Checkpoints](../SKILL.md#developer-checkpoints))
   - Implement the change
   - Run tests if suite exists
   - Review, then commit sign-off — changed files, review verdict, intent verdict; "Ready to commit?"

#### Scenario C — New feature (no existing documents)

1. Confirm: "No existing concept found for this. I'll start from concept phase."
2. Execute: `/dev-flow concept` → `/dev-flow spec` → `/dev-flow plan` → **design sign-off** → `/dev-flow implement` → … → review → **commit sign-off**
3. Follow all gate checks between phases; the checkpoints are skipped only under `Autonomy: full` in the task header ([SKILL.md → Developer Checkpoints](../SKILL.md#developer-checkpoints)).

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
1. **Intent verdict (on completion).** When reporting the task done or stopping at the commit sign-off, compare the outcome to the task's `## Intent` (Expected result) and state `intent: met / partially met / diverged (+why)` in the report — a divergence is surfaced, never silently absorbed. Skip for a mid-task hand-off. See [Task Intent](../references/task-intent.md).
2. Run the [status write protocol → on task completion](status.md#write-protocol) — Subtask `Status` and wrap-up Activity entry, header `Last updated` (task-level `Status: done` only when every contributor's subtask is done), the dashboard and catalog row move, and the [hygiene caps](status.md#context-hygiene) with overflow archived to `.dev_flow/session_history/`. Optionally add a hand-off Coordination Note (`[your-id] — stepping away, anyone may pick up from <here>`).
3. **Surface queued follow-ups.** If the task became `done`, scan `.dev_flow/todos/` and plan backlogs for `queued` records triggered `after task_<this ID>` (fixes deferred *because their context overlapped this task*). List each and offer to run it next via `/dev-flow do …` — a suggestion, not an auto-run; the executed work passes its own gates and commit approval. See [todo phase](todo.md).

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

The orchestrator answers in the `chat` register and writes documents in the `documentation` register. See [Output Styles](../references/output-styles.md) — it also carries the **Style gate** (`.dev_flow/output_styles.md`) the orchestrator passes to the executing phase alongside rules and skills.
