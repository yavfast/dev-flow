# Task Intent — Executing for the Goal, Not the Letter

Shared sub-procedure for the **Do**, **Plan**, **Implement**, **Fix**, **Verify**, and **Subtask** phases. It is **not** a standalone pipeline stage and has no command — it activates whenever a task starts (capture) and at the moments listed below (check). The **Task Intent** is the *why* behind a request: the goal it serves, the target state it aims at, and the result the user will judge it by.

## The Intent record

Captured in the task file (see the [task template](../templates/task_context.md) → `## Intent`):

| Field | Answers |
|-------|---------|
| **Goal (why)** | What problem this solves / why the user wants it — the purpose the task serves |
| **Target state** | How the system/docs/behavior should look when the task is done |
| **Expected result** | The observable outcome the user will check — the success criteria |
| **Outcome check** *(optional)* | The signal and the return trigger of a result that shows only after the commit — [Outcome check](#outcome-check) |

Rules for the record:

- **User's terms, not paraphrase drift.** Quote or closely restate the request; keep the user's vocabulary.
- **Stated vs inferred.** What the user said is fact; what you deduced is marked `(inferred)`. An inference the execution materially depends on and that you cannot make confidently → one targeted intake question (within `do`'s max-3 budget), not a guess.
- **Sources:** the request wording, the active task context, prior conversation, a linked ticket's description/acceptance criteria ([Ticket Tracker Integration](ticket-tracker.md)).
- **It changes only by the user's word.** A recorded intent is not re-litigated mid-task; when the user restates the goal, update the record and re-check open work against it (a widened goal may re-class the change — see [do → Change Classes](../phases/do.md#change-classes)).

For pipeline-scale work the intent also lands in the concept's *Philosophy / Core Principle* and the plan's *Goal*. Every other task carries only the task-file record — fixes included — except the trivial route below.

## Capture — at intake

Whoever opens the task captures the intent: `do` while interpreting the request (before routing), or any directly-invoked phase when it creates the task file. Distinguish the **requested action** from the **underlying goal** — "add an index to this table" is the action; "the search page is too slow" may be the goal, and it changes what a good solution is.

## Check — the moments

1. **Before a material decision** (implement / fix / plan). Ask: *does this option serve the recorded intent?* This is the concrete target of [Consequence Forecasting](consequence-forecasting.md)'s upward glance at implement altitude — glance at the record, not at a remembered impression of it.
2. **Letter-vs-spirit conflict.** Evidence shows that executing the request *as literally stated* will not achieve the recorded goal (or actively defeats it) → **stop and surface**: present the conflict with marked options ([Interview Mode](interview-mode.md) style — follow the letter / adjust toward the goal / clarify), with your recommendation. Never silently follow the letter off the cliff, and never silently substitute your own reinterpretation.
3. **At completion — before reporting done / commit approval.** Compare the outcome to **Expected result** and state the verdict in the result report: `intent: met / partially met / diverged (+why)`. In [Verify](../phases/verify.md), the bar is "the expected result is observable", not merely "tests pass". An [Outcome check](#outcome-check) extends the verdict.
4. **On delegation.** The intent travels with the work: a [subtask](../phases/subtask.md) brief carries the Goal/Expected result alongside the done-criteria; even a one-line focus delegation states what the conclusion is *for* — so subagents optimize for the same goal as the initiator.

## Outcome check

An optional line of the record, after Expected result: `- **Outcome check:** <signal> — trigger: <trigger>`. **Signal** — what to observe and where (command, file, metric, place in the product). **Trigger** — a date, or an event visible in git, the docs, or the task files. The line is whole or absent, never `n/a`.

**Capture** — with the record. Write no line when one of these is true: the task has no Intent record; a test, gate, run, or look at the diff shows the Expected result before the commit; no signal can be named; no trigger can be named. Show the line to the developer when it is written; the developer changes or removes it by their word.

**Verdict form** — at the sign-off of the task's last commit: `intent: <verdict> — outcome pending (<trigger>)`.

**Open** — when the task becomes `done`, the contributor that sets it appends one entry to `outcomes:` in the ledger ([Evidence Discipline](evidence-discipline.md#outcome-entries)) and refreshes the dashboard's outcome-checks line. One entry per task: a task that has an entry gets no second one.

**Reconcile** — when the trigger of an open entry is due: in [audit](../phases/audit.md) Step 7b, at `do` intake as an offer that runs on the developer's yes, or on the developer's request. Append one verdict:

- the developer states the outcome → their word; `ref` is `developer`, and `note` holds the quote;
- the signal is observed and leaves a non-model by-product → `met` · `partially met` · `diverged`; `ref` is the by-product;
- neither → `still-unobserved` with the reason and no `ref`. A verdict other than `still-unobserved` is never written without a `ref`.

Then:

- a verdict other than `still-unobserved` closes the entry;
- a second consecutive `still-unobserved` goes to the developer: a new trigger, or the entry becomes `dropped`;
- `partially met` and `diverged` come with an offered follow-up ([fix](../phases/fix.md) or [todo](../phases/todo.md)), never an auto-run;
- the verdict goes into the report of the run that wrote it, and the dashboard's outcome-checks line is refreshed;
- the developer's word overrides the agent's verdict: append the new verdict, keep the old one;
- `audit --dry-run` lists the due entries and appends nothing.

## When NOT to fire

- **Trivial route with self-evident intent** (typo, comment, log message) — no record, no questions; the action *is* the intent.
- **Don't interrogate.** Infer first; ask only when a material decision depends on an inference you cannot make. Routine goal-quizzing trains the user to answer noise.
- **Intent is not a scope license.** Serving the goal does not authorize work beyond the request — a "while we're at it" that the goal merely *suggests* routes through the YAGNI-gate ([Consequence Forecasting](consequence-forecasting.md)) or an agent-initiated [todo](../phases/todo.md), not into the task.

## Where it is consumed — thin pointers

| Consumer | Use |
|----------|-----|
| [do](../phases/do.md) | Capture at intake (Step 2), record into the task file, verdict at wrap-up; due outcome checks offered at intake |
| [plan](../phases/plan.md) | The plan's Goal section restates the recorded intent for its scope |
| [implement](../phases/implement.md) | Intent check in the pre-code gate; check-moment 1 & 2 during work |
| [fix](../phases/fix.md) | Expected behavior captured in Step 1 analysis; verdict in the Step 4 report |
| [verify](../phases/verify.md) | Expected-result check beside the `Verify:` criteria; verdict row in the report |
| [subtask](../phases/subtask.md) | Intent handed down in the brief (Step 2 / prompt `## Subtask`) |
| [status](../phases/status.md) | Outcome entry opened on task completion; the dashboard line of open outcome checks |
| [audit](../phases/audit.md) | Due outcome checks reconciled in Step 7b |
| task context ([template](../templates/task_context.md)) | The `## Intent` section is the record's home |
