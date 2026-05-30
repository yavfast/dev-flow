# Phase: Subtask — Delegate Secondary Work to a Subagent

## Purpose

Offload a secondary task to an independent subagent so the main conversation
stays focused on the primary work. The subagent operates as a full dev-flow
participant — it can follow any phase protocol (fix, implement, test, ask, etc.)
autonomously and returns a concise result report.

The key value: the main context stays clean. Instead of switching to a bug fix
mid-concept and flooding the context with irrelevant code diffs, the subagent
handles it in isolation and reports back only the essential result.

> **Naming note — two related concepts:**
> - **`/dev-flow subtask <description>` (this command)** spawns an isolated
>   subagent that works on a self-contained piece of work and reports back to
>   the main conversation. The subagent never touches `.dev_flow/` context files.
> - **In-task "Subtask blocks"** inside a `tasks/task_<ID>.md` file are how
>   peer AI contributors (separate sessions) divide work on the same task.
>   Each contributor owns one Subtask block.
>
> The command here is about **delegation to a subagent**. In-task Subtasks
> are about **collaboration between peer contributors**.

## Command

```
/dev-flow subtask <task description>
```

The description is freeform, in any language. It should clearly state what
the subagent must accomplish. If the task maps to a dev-flow phase, name it
explicitly (e.g., "fix ...", "write tests for ...").

### Examples

```
/dev-flow subtask fix: NPE в UserService.getProfile() коли user.avatar == null
/dev-flow subtask fix: Upload progress bar freezes at 99% on slow connections — see src/upload/ProgressTracker.ts
/dev-flow subtask Напиши unit-тести для класу TokenBucket згідно зі специфікацією docs/rate_limiter.sp.md
/dev-flow subtask ask: які модулі будуть зачеплені, якщо замінити OldClient на NewClient?
/dev-flow subtask Зрефактори ініціалізацію логера в src/utils/logger.ts — винеси конфіг в окремий файл
/dev-flow subtask Research how other projects implement retry backoff for gRPC — summarize top 3 approaches
```

## Role Responsible

This command is handled by **SubtaskExecutor**:
[roles/subtask-executor.ai.md](../roles/subtask-executor.ai.md)

## When to Use

- A bug or small defect is noticed during work on a different task (concept, spec, plan)
  and can be fixed without blocking the main flow
- A side task appears during implementation that is **not** part of the current plan phase
- You need research or analysis that would consume significant context
- A peripheral change (tests, docs, config, refactoring) can be described self-containedly
- The result is needed to continue the main task, but the *process* of getting it is not

## When NOT to Use

- The task **is** the main task — just do it directly
- The task requires ongoing dialogue with the user — subtasks run autonomously
- The task modifies the same files the main context is actively editing (conflict risk)
- The task is trivial (< 2 minutes of work) — the overhead of delegation is not worth it

## Procedure

### Step 1: Determine the dev-flow phase

Analyze the task description to identify which dev-flow phase protocol the subagent
should follow:

| Task pattern | Phase | Notes |
|-------------|-------|-------|
| Bug report, error, "fix" | `fix` | Full fix protocol: analyze → plan → implement → verify |
| "Write tests", "add tests" | `test` | Follow testing phase protocol |
| "Research", "find out", "ask" | `ask` | Read-only analysis, no file changes |
| "Refactor", "extract", "rename" | `implement` | Code changes following plan |
| "Update spec", "update docs" | `propagate` | Documentation updates |
| General / no clear phase | — | Freeform execution with scope boundaries |

If the task clearly maps to a phase, tell the subagent to follow that phase's protocol.
This ensures the subagent performs proper analysis, verification, and rule compliance
rather than just making ad-hoc changes.

### Step 2: Formulate the subtask brief

Before spawning the subagent, prepare a self-contained brief:

1. **Task description** — what the subagent must accomplish (from the user's command).
2. **Dev-flow phase** — which phase protocol to follow (from Step 1).
3. **Context notes** — extract from the current conversation any information the subagent
   will need: file paths, spec references, architectural constraints, naming conventions,
   relevant rules from `.dev_flow/rules/`, related concept/spec documents.
4. **Output expectations** — what the report should contain: root cause, files changed,
   test results, analysis conclusions, etc.
5. **Scope boundaries** — what the subagent should NOT do (e.g., "do not modify files
   outside `src/utils/`", "do not commit").

The goal is to give the subagent everything it needs to work independently,
without access to the main conversation history.

### Step 3: Spawn the subagent

Use the Agent tool with a prompt structured as:

```
## Subtask
<task description>

## Dev-Flow Phase
Follow the `<phase>` phase protocol from the dev-flow skill.
Skill location: <path to dev-flow SKILL.md>
Phase details: <path to relevant phase file>

## Context
<relevant notes, file paths, spec references, rules, constraints>

## Scope
<boundaries — what NOT to do>
- Do NOT create or modify any file under .dev_flow/tasks/ — the main task is owned by the calling agent
- Do NOT modify .dev_flow/active_context.md or .dev_flow/tasks/_index.md — the main context owns them
- Do NOT commit to git

## Report Format
When done, provide a brief report (under 300 words) containing:
- What was accomplished (root cause for fix, findings for ask)
- Key artifacts created or modified (with file paths)
- Verification result (build/test status if applicable)
- Any issues or follow-up items for the main task
- Raw output — full test logs, build dumps, traces — stays in files referenced by path;
  never paste it into the report. The report is the conclusion, not the dump.
```

Choose the appropriate subagent type:
- **Explore** — for `ask` phase (read-only search and analysis)
- **general-purpose** — for `fix`, `implement`, `test`, `propagate`, and freeform tasks
- Use `run_in_background: true` when the main task can continue without the result

**Pick the model to fit the task** by its nature, not a fixed name — fast/cheap for
mechanical, narrow-output work; stronger when it needs judgment; when unsure, the stronger
one. See **[Delegation for Focus](../references/delegation.md)**.

### Step 4: Receive and relay the report

When the subagent completes:

1. Read the report.
2. Extract the actionable information relevant to the main task.
3. Present a **concise summary** to the main context — what was done, what was found,
   and whether any follow-up is needed.
4. If the subtask produced files or code changes, mention the paths so the main
   context can reference them.
5. Continue the main task using the subtask results.

### Step 5: Update the main task's context (if applicable)

If the main work is tracked in `.dev_flow/tasks/task_<ID>.md`, append a one-line
entry to its Activity Log:

```
- YYYY-MM-DD HH:MM — <agent> — subtask completed (<phase>): <brief description> — <key result>
```

Do **not** create a separate task file for the subtask itself — subtasks share
the calling task's file. The subagent is forbidden from touching context files;
only the calling (main) agent records the subtask outcome here.

## Parallel Subtasks

Multiple subtasks can be spawned simultaneously when they are independent.
Launch them in a single turn using multiple Agent tool calls:

```
/dev-flow subtask fix: NPE in UserService.getProfile()
/dev-flow subtask ask: які модулі зачепить заміна OldClient?
```

The orchestrator should batch these into parallel subagent calls and collect
all reports before continuing.

## Language Policy

The subagent works in the **same language** as the task description.
The report is relayed in the **same language** used in the main conversation.
