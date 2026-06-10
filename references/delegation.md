# Delegation for Focus — Context Isolation

Shared sub-procedure for the **Implement**, **Fix**, and **Verify** phases (and the
mechanism behind **Review** and the **Subtask** command). This is **not** a standalone
pipeline stage — it runs *inside* those phases whenever a step is about to flood the main
context with secondary, noisy work.

> **Two delegation shapes.** This reference covers **focus delegation** — a single
> noisy *step* inside your own phase run: minimal rights, no `.dev_flow/` writes,
> stage-and-report, no dialogue. Delegating a whole secondary *task* is the
> [subtask phase](../phases/subtask.md) — there the subagent is a **full dev-flow
> participant with delegated rights**: it assembles its own context, joins the task
> file as a contributor, persists skills/cache per the phase protocols, and can
> converse with its initiator. The moment a delegated step outgrows one exchange,
> it is a subtask.

## Why this exists

The main agent is the conductor. It holds the plan, the specification, and the task
state, and it makes the calls at the gates. That working focus is the scarce resource —
the thing that lets it write code that actually matches the spec and notice when something
drifts.

Focus erodes when secondary, noisy work shares the same context: full test output, build
logs, screenshots, wide code searches, reproduction traces. None of it is wrong to
*produce* — but most of it doesn't need to *stay*. A thousand lines of passing-test output
buys one bit of information ("they passed") while pushing the plan and the half-written
change out of view. By the time the context is thick with logs, the agent is
reconstructing what it was doing instead of doing it.

The cure is to hand that work to a subagent with its own context and keep only the
conclusion. The subagent spends its context on the noise; the main agent gets the one bit
that mattered and stays on task. This is the same instinct as `handle-uncertainty` and
Interview Mode — protect the expensive thing (here, attention) from the cheap thing
(here, volume).

## Where the line falls

**Keep in the main context — the core.** Work that needs the full plan + spec + rules, or
that *is* the focus:
- Writing and editing code.
- The decisions at the gates.
- The task state (what's done, what's next).

Handing these off hands off the focus itself — there's nothing left to protect.

**Delegate — the noise.** Work that emits a lot and needs little of it back:
- Running tests and parsing failures.
- Build and verify runs.
- Reproduction and instrumentation during diagnosis.
- Broad code searches ("where is X used?").
- "Read this large output / file and tell me the verdict."

## The habit that makes it pay off

A delegated subagent returns the **conclusion, not the dump**: a verdict plus what's
actionable, with the full log written to a file and referenced by path — under the
project workspace `/tmp/{project-slug}/` with a timestamped name, per the
[cache phase](../phases/cache.md) workspace discipline. In this focus shape, anything
durable among the artifacts is promoted to `.dev_flow/cache/` by the calling agent —
the helper only stages. (A task-delegated subagent persists per the phase protocol
itself; see the [subtask phase](../phases/subtask.md).)

Skip this and the win evaporates — the noise simply relocates from the tool output into
the subagent's report, and the main context is flooded just the same, now with an extra
hop. The discipline is the whole point: *the report is the conclusion, not the transcript.*

Review already works this way for a second reason — a clean-context subagent reads the
diff without the author's rationalizations, so its judgment is unbiased. Verify and
diagnosis share the shape; they just optimize for focus rather than impartiality.

## Fit the model to the task — by its nature, not its name

A delegated step can run on a different model than the main agent, chosen by the
*character* of the work:
- Mechanical, narrow-output work (run a suite, grep, summarize a log) runs fine on a
  fast, cheap model.
- Work that needs judgment (untangling a root cause, weighing an architectural option)
  wants a stronger one.
- When unsure, prefer the stronger one — a wrong result costs more than the tokens saved.

Describe the *nature* of the task and let the orchestrator map it to whatever model is
configured. **Never hardcode model names** — they age, and they tie the skill to one
provider. Keep the name→tier mapping in the orchestrator's config, the same way the engine
resolves providers by convention rather than a whitelist. The choice then survives new
models arriving and current ones retiring.

## Scenarios

A quick reference for the recurring steps. "Delegate?" is a default, not a law — a trivial
one-test run isn't worth the overhead of a subagent, and a genuinely tangled diagnosis
might need the main agent's full context. Use judgment.

| Step (phase) | Delegate? | Subagent returns | Fits |
|--------------|-----------|------------------|------|
| Write / edit code (implement) | No — core | — | main agent |
| Decisions at gates, task state | No — core | — | main agent |
| Run test suite + parse failures (test, fix) | Yes | failing tests + reasons, log path | fast / cheap |
| Build / verify run (verify) | Yes | pass/fail + failures that matter, log path | fast / cheap |
| Regression / integration / live (verify) | Yes | verdict + failures, artifact paths | fast / cheap |
| Diagnosis: reproduce + instrument (fix) | Yes | confirmed cause + evidence, log path | fast / cheap |
| Spike investigation (research) | Yes | verdict + findings + recommendation; raw material stays in the spike file | standard / strong — weighing sources and trade-offs needs judgment |
| Broad code search ("where is X used?") | Yes | findings + file:line list | fast / cheap |
| Pre-commit review | Yes (clean ctx) | blocking issues + warnings | standard / strong |
| Untangle a tangled root cause | Borderline | cause + reasoning | stronger |

## How

The spawning mechanics live in the [Subtask phase](../phases/subtask.md): formulate the
brief, spawn the subagent (picking type and model), and take back the conclusion. For a
focus delegation the brief is a narrow step description; for a task delegation it is
"keys, not content" — task + role + context hints, and the subagent assembles the rest.
Use delegation not only as the explicit `/dev-flow subtask` command but as a *reflex*
during implement / fix / verify — the moment a step is about to dump noise into the main
context, delegate it.

Project-specific delegation practice (how *this* project runs its tests, where its live
scenarios live, what counts as pass/fail) belongs in a project role overlay under
`.dev_flow/roles/`, not here — see [Roles](roles.md) for using and creating them.

These scenarios and rules are **projections, not a formal protocol** — heuristics the
agent applies with judgment. When a call is genuinely unclear, it asks the initiator
rather than following an algorithm. Don't encode merge rules or rigid MUST/NEVER machinery
here; keep the guidance in prose.
