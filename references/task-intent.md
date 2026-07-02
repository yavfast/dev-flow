# Task Intent — Executing for the Goal, Not the Letter

Shared sub-procedure for the **Do**, **Plan**, **Implement**, **Fix**, **Verify**, and **Subtask** phases. It is **not** a standalone pipeline stage and has no command — it activates whenever a task starts (capture) and at the moments listed below (check). The **Task Intent** is the *why* behind a request: the goal it serves, the target state it aims at, and the result the user will judge it by.

## Why this exists

A request states an *action*; the *reason* for it usually stays in the user's head. An agent optimizing for the literal action can complete it perfectly and still miss the point — or follow the letter into a result the user never wanted. Captured once at intake, the intent makes every later decision checkable against the goal instead of the wording, and makes "done" mean "the expected result exists", not "the described steps were performed".

## The Intent record

Three fields, captured in the task file (see the [task template](../templates/task_context.md) → `## Intent`):

| Field | Answers |
|-------|---------|
| **Goal (why)** | What problem this solves / why the user wants it — the purpose the task serves |
| **Target state** | How the system/docs/behavior should look when the task is done |
| **Expected result** | The observable outcome the user will check — the success criteria |

Rules for the record:

- **User's terms, not paraphrase drift.** Quote or closely restate the request; keep the user's vocabulary.
- **Stated vs inferred.** What the user said is fact; what you deduced is marked `(inferred)`. An inference the execution materially depends on and that you cannot make confidently → one targeted intake question (within `do`'s max-3 budget), not a guess.
- **Sources:** the request wording, the active task context, prior conversation, a linked ticket's description/acceptance criteria ([Ticket Tracker Integration](ticket-tracker.md)).
- **It changes only by the user's word.** A recorded intent is not re-litigated mid-task; when the user restates the goal, update the record and re-check open work against it (a widened goal may re-class the change — see [do → Change Classes](../phases/do.md#change-classes)).

For pipeline-scale work the intent flows downstream into the documents: the concept's *Philosophy / Core Principle* ("why this feature exists") and the plan's *Goal* section are its durable homes. The task-file record is what exists for **every** task — including fixes and trivial routes that never touch `docs/`.

## Capture — at intake

Whoever opens the task captures the intent: `do` while interpreting the request (before routing), or any directly-invoked phase when it creates the task file. Distinguish the **requested action** from the **underlying goal** — "add an index to this table" is the action; "the search page is too slow" may be the goal, and it changes what a good solution is.

## Check — the four moments

1. **Before a material decision** (implement / fix / plan). Ask: *does this option serve the recorded intent?* This is the concrete target of [Consequence Forecasting](consequence-forecasting.md)'s upward glance at implement altitude — glance at the record, not at a remembered impression of it.
2. **Letter-vs-spirit conflict.** Evidence shows that executing the request *as literally stated* will not achieve the recorded goal (or actively defeats it) → **stop and surface**: present the conflict with marked options ([Interview Mode](interview-mode.md) style — follow the letter / adjust toward the goal / clarify), with your recommendation. Never silently follow the letter off the cliff, and never silently substitute your own reinterpretation.
3. **At completion — before reporting done / commit approval.** Compare the outcome to **Expected result** and state the verdict in the result report: `intent: met / partially met / diverged (+why)`. In [Verify](../phases/verify.md), the bar is "the expected result is observable", not merely "tests pass".
4. **On delegation.** The intent travels with the work: a [subtask](../phases/subtask.md) brief carries the Goal/Expected result alongside the done-criteria; even a one-line focus delegation states what the conclusion is *for* — so subagents optimize for the same goal as the initiator.

## When NOT to fire

- **Trivial route with self-evident intent** (typo, comment, log message) — no record, no questions; the action *is* the intent.
- **Don't interrogate.** Infer first; ask only when a material decision depends on an inference you cannot make. Routine goal-quizzing trains the user to answer noise.
- **Intent is not a scope license.** Serving the goal does not authorize work beyond the request — a "while we're at it" that the goal merely *suggests* routes through the YAGNI-gate ([Consequence Forecasting](consequence-forecasting.md)) or an agent-initiated [todo](../phases/todo.md), not into the task.

## Where it is consumed — thin pointers

| Consumer | Use |
|----------|-----|
| [do](../phases/do.md) | Capture at intake (Step 2), record into the task file, verdict at wrap-up |
| [plan](../phases/plan.md) | The plan's Goal section restates the recorded intent for its scope |
| [implement](../phases/implement.md) | Intent check in the pre-code gate; check-moment 1 & 2 during work |
| [fix](../phases/fix.md) | Expected behavior captured in Step 1 analysis; verdict in the Step 4 report |
| [verify](../phases/verify.md) | Expected-result check beside the `Verify:` criteria; verdict row in the report |
| [subtask](../phases/subtask.md) | Intent handed down in the brief (Step 2 / prompt `## Subtask`) |
| task context ([template](../templates/task_context.md)) | The `## Intent` section is the record's home |
