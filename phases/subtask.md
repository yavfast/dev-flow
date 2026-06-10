# Phase: Subtask — Delegate a Secondary Task to a Subagent

## Purpose

Offload a secondary task to a subagent so the main conversation keeps its context
and focus on the primary work. The subagent is a **full dev-flow participant with
delegated rights**: the initiator hands it the task, a role, and context *hints* —
the subagent assembles its own working context from there, executes the matching
phase protocol end to end, talks to its initiator when a real decision needs one,
and returns a **full report**.

The economy is the point. The initiator spends a few lines on the brief instead of
pre-reading material into its own context; all the noisy assembly (reading docs,
loading rules/skills, running tools) happens in the subagent's context; what comes
back is the condensed, complete result. Delegation buys focus only if both sides
hold their end: the initiator doesn't pre-chew, the subagent doesn't dump.

> **One contributor model for everyone.** A task-delegated subagent follows the
> same [multi-contributor rules](../SKILL.md#multi-contributor-tolerance) as any
> peer AI session: it owns its own Subtask block and its tagged entries, never
> rewrites others' content, uses targeted edits and read-before-write on shared
> files. The `/dev-flow subtask` command and the "Subtask block" in a task file
> are two ends of the same concept — the command spawns a contributor; the block
> is where that contributor's thread lives.

## Command

```
/dev-flow subtask <task description>
```

The description is freeform, in any language. It should clearly state what
the subagent must accomplish. If the task maps to a dev-flow phase, name it
explicitly (e.g., "fix ...", "write tests for ...", "research ...").

### Examples

```
/dev-flow subtask fix: NPE в UserService.getProfile() коли user.avatar == null
/dev-flow subtask fix: Upload progress bar freezes at 99% on slow connections — see src/upload/ProgressTracker.ts
/dev-flow subtask Напиши unit-тести для класу TokenBucket згідно зі специфікацією docs/rate_limiter.sp.md
/dev-flow subtask ask: які модулі будуть зачеплені, якщо замінити OldClient на NewClient?
/dev-flow subtask research how other projects implement retry backoff for gRPC — summarize top 3 approaches
/dev-flow subtask Зрефактори ініціалізацію логера в src/utils/logger.ts — винеси конфіг в окремий файл
```

## Role Responsible

This command is handled by **SubtaskExecutor**:
[roles/subtask-executor.ai.md](../roles/subtask-executor.ai.md) — combined with
the base role (and project overlay, if any) of the phase being executed.

## Two Delegation Shapes

Not every delegation is a subtask. The rights follow the shape:

| | **Focus delegation** | **Task delegation (this phase)** |
|--|---------------------|----------------------------------|
| What | One noisy step inside the initiator's own phase run (run tests, parse logs, wide search) | A self-contained secondary task (fix, research, tests, propagate, …) |
| Protocol | [Delegation for Focus](../references/delegation.md) | This phase |
| Context writes | None — stage artifacts in the workspace, report paths | Own Subtask block + tagged entries, per the contributor rules |
| `.dev_flow/` writes (skills, cache, rules) | Stage + report; the initiator persists | Directly, by executing the owning phase's protocol (its gates, filters, and index discipline) |
| Dialogue | None — returns a verdict | Escalates real questions to the initiator mid-task |
| Report | Conclusion, not the dump | Full report (below) — complete in coverage, conclusion in style |

Rule of thumb: the moment delegated work outgrows one exchange — multi-step, may
pause on a question, produces durable artifacts — it is a **task delegation** and
the subagent joins as a contributor. A one-shot mechanical step stays focus
delegation and never touches `.dev_flow/`.

## When to Use

- A bug or small defect is noticed during work on a different task (concept, spec, plan)
  and can be fixed without blocking the main flow
- A side task appears during implementation that is **not** part of the current plan phase
- You need research or analysis that would consume significant context
- A peripheral change (tests, docs, config, refactoring) can be described by a goal
  and a few pointers
- The result is needed to continue the main task, but the *process* of getting it is not

## When NOT to Use

- The task **is** the main task — just do it directly
- The task is mostly a conversation with the developer — escalation can relay
  single decisions, but work that is all dialogue belongs in the main context
- The task modifies the same files the main context is actively editing (conflict risk)
- The task is trivial (< 2 minutes of work) — the overhead of delegation is not worth it

## Delegated Rights & Boundaries

A task-delegated subagent has the **same functional rights as the initiating
agent**, bounded by the same protocols — not by blanket bans:

- **Code, tests, docs** — full read/write within the brief's scope, following the
  executed phase's gates. Running the skills/rules/cache checks is the
  *subagent's own* job, not the initiator's.
- **Task context** — joins the initiator's task file as a **Contributor**: adds
  itself to the header's `Contributors`, adds its own `### Subtask:` block, keeps
  it current, appends Coordination Notes and tagged entries in shared sections.
  Dashboard/catalog rows — targeted edits at phase boundaries, per the
  [rules for all phases](../SKILL.md#rules-for-all-phases). Do not create a
  separate task file — the subtask lives in the calling task. A read-only
  subtask skips the join (next bullet).
- **Phase constraints prevail over delegated rights.** A read-only phase keeps
  its guarantees regardless of delegation — an `ask` subtask writes **nothing**,
  context files included. Likewise, a base role read in the executor seat
  contributes its craft (how to investigate, how to test); its *helper-position*
  constraints ("stage and report", "never write `.dev_flow/`") describe focus
  delegation and do **not** bind the phase executor. On any other genuine role
  conflict — ask the initiator (see [Roles](../references/roles.md)).
- **Skills / cache / rules** — written directly by executing the owning phase's
  protocol: the [skill](skill.md) non-triviality filter, the [cache](../references/cache.md)
  worth-caching filter + entry-by-entry index discipline, the [rule](rule.md)
  severity model. Everything persisted is listed in the report.
- **Commits — no.** Not a capability gap but the approval chain: commits require
  the developer's approval, and the developer channel lives with the main
  context. The subagent leaves the work commit-ready; the initiator runs the
  pre-commit review and asks the developer.
- **Scope narrowing** — the brief may still restrict files or areas; this is what
  keeps **parallel** subtasks from colliding. An explicit restriction in the
  brief overrides the defaults above. When the scope conflicts with the task —
  ask the initiator, don't guess.

## Talking to the Initiator

The subagent's escalation channel is its **initiator** — the developer channel
stays with the main context:

- **Escalate what Interview Mode would surface:** a material fork (2+ viable,
  hard-to-reverse options), missing access, a scope conflict, or evidence that
  the brief's premise is wrong ([Upstream Escalation](../references/escalation.md)
  applies inside subtasks too).
- **Same discipline as [Interview Mode](../references/interview-mode.md):**
  exhaust your own context assembly first — anything answerable from code, docs,
  or indexes is homework, not a question. Batch independent questions; attach a
  recommended answer to each.
- **The initiator decides:** answer from the main-task picture, relay to the
  developer what only the developer can decide, or re-scope the task.
- **Checkpoint dialogue is the default.** In most runtimes the end-of-turn
  report IS the channel: finish everything the question does not block, put the
  questions under the report's *Open items* (markered options + recommendation,
  Interview-Mode style); the initiator answers and **continues the same
  subagent** so its assembled context survives. Never stall mid-run waiting for
  an answer. Live mid-task dialogue is the exception, used when the harness
  supports it. A subagent with no continuation channel at all records open
  decisions exactly like a delegated subagent in Interview Mode — options,
  recommendation, resolution trigger.

## Procedure

### Step 1: Determine the dev-flow phase

Analyze the task description to identify which dev-flow phase protocol the subagent
should follow:

| Task pattern | Phase | Notes |
|-------------|-------|-------|
| Bug report, error, "fix" | `fix` | Full fix protocol: analyze → plan → implement → verify |
| "Write tests", "add tests" | `test` | Follow testing phase protocol |
| "Find out", "where is", "ask" — answerable from the codebase | `ask` | Read-only — the phase's no-write constraints prevail: no contributor join, report only |
| "Research", "investigate", "compare approaches" — needs external sources or experiments | `research` | Full research protocol incl. Step 4 persistence (skills + cache) |
| "Refactor", "extract", "rename" | `implement` | Code changes following plan |
| "Update spec", "update docs" | `propagate` | Documentation updates |
| General / no clear phase | — | Freeform execution with scope boundaries |

The subagent executing a phase holds that phase's "main agent" duties — gates,
verification steps, persistence. Helper roles it spawns inside its own run (a
noisy test pass, a wide search) follow focus delegation, as anywhere else.

### Step 2: Formulate the brief — keys, not content

The brief hands over pointers; the subagent assembles its own context:

1. **Task** — the goal and its done-criteria, in 1–3 sentences.
2. **Role(s)** — the phase's base role plus a project overlay if one exists
   (check `.dev_flow/roles/_index.yaml` first — see [Roles](../references/roles.md)).
3. **Context hints** — the task file path (for the contributor join), relevant
   doc/spec traceable IDs, file paths, known constraints, applicable
   `.dev_flow/rules/` / `skills/` / `cache/` entries worth starting from.
   *Hints, not excerpts* — do not paste what the subagent can read itself.
4. **Scope** — only the genuine restrictions: parallel-conflict areas, forbidden
   targets, plus the standing boundary (no commits).
5. **Report expectations** — anything needed beyond the standard full report.

### Step 3: Spawn the subagent

Use the Agent tool with a prompt structured as:

```
## Subtask
<task + done-criteria>

## Role
Skill root: <absolute path to the dev-flow skill> — skill paths below are
relative to it; expand them yourself (your cwd is the project, not the skill).
Follow the `<phase>` phase protocol (phases/<phase>.md); act as
<role file path(s)>. You are a full dev-flow participant with delegated
rights — see phases/subtask.md → Delegated Rights & Boundaries. A base role's
helper-position constraints do not bind you as the phase executor; a read-only
phase's no-write constraints DO prevail.

## Context assembly (yours — do first)
- Join the task file as a Contributor (skip for read-only phases like `ask`):
  <.dev_flow/tasks/task_<ID>.md> — add yourself to `Contributors`, add your own
  `### Subtask:` block; never edit other contributors' content.
- Run the phase's own gates: .dev_flow/rules/, skills/, cache/ indexes,
  docs/_glossary.md, the docs named below.
- Read whatever else the task needs from the codebase — do not ask for what
  you can read.

## Context hints
<paths, traceable IDs, constraints — pointers, not excerpts>

## Scope
<genuine restrictions for this run; parallel-conflict areas>
- Do NOT commit — leave the work commit-ready; the initiator owns the approval chain.

## Communication
Material fork / missing access / wrong premise → escalate to me: markered
options + your recommendation, independent questions batched. The default
channel is the checkpoint: finish what the question does not block, put the
questions under "Open items" in your report — I will answer and continue you.
Never stall mid-run waiting for an answer.

## Report
Full report per phases/subtask.md → Full Report Contract. Raw output stays in
the project workspace (/tmp/{project-slug}/, timestamped names), referenced by path.
```

Choose the appropriate subagent type:
- **Explore** — for `ask` (read-only search and analysis)
- **general-purpose** — for everything that writes: `fix`, `implement`, `test`,
  `propagate`, `research`, freeform

**Pick the model to fit the task** by its nature, not a fixed name — fast/cheap for
mechanical, narrow-output work; stronger when it needs judgment; when unsure, the stronger
one. See **[Delegation for Focus](../references/delegation.md)**.

### Step 4: Mid-task exchanges

When the subagent escalates, answer with the same care as an Interview-Mode
reply: pick a marker, amend, or relay the question to the developer. Continue the
same subagent rather than respawning — its assembled context is the asset.

### Step 5: Receive the full report and integrate

1. Read the report; spot-check the changes it claims (diff, key files).
2. The subagent has already updated **its own** Subtask block and any shared
   entries it owns — verify the task file reflects reality; do not rewrite the
   subagent's block. (A read-only subtask made no context writes — record its
   outcome yourself in step 3.)
3. Record the outcome in **your own** Subtask block / Shared Activity Log:
   `subtask completed (<phase>): <one line> — <key result>`.
4. Carry *Open items* into the main flow: answer, escalate to the developer, or
   convert to tracked items (backlog with trigger, Blocking Issue, DEC record).
5. Continue the main task using the results.

## Full Report Contract

Complete in **coverage**, conclusion in **style** — every section present, raw
material referenced by path, never inlined:

```
## Result        — outcome vs the task goal (met / partially / blocked + why)
## Decisions     — choices made (incl. DEC records added), escalations and answers
## Changes       — files created/modified: code, tests, docs; own Subtask block
                   (if joined); persisted skills/cache/rules entries (paths)
## Verification  — build / test / review status, per the phase's protocol
## Open items    — questions (markered options + recommendation), follow-ups, risks
## Materials     — workspace paths to raw logs, dumps, captures
```

## Parallel Subtasks

Multiple subtasks can be spawned simultaneously when they are **independent**.
Launch them in a single turn using multiple Agent tool calls.

Shared `.dev_flow/` files tolerate this by design (contributor rules:
read-before-write, own blocks, targeted edits). Source files do not — use the
brief's **Scope** to give parallel subtasks disjoint file areas, and keep work
that touches the initiator's active files out of delegation entirely.

## Language Policy

The subagent works in the **same language** as the task description.
The report is relayed in the **same language** used in the main conversation.
