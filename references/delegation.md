# Delegation for Focus — Context Isolation

Shared sub-procedure for the **Implement**, **Fix**, and **Verify** phases (and the mechanism behind **Review** and the **Subtask** command). This is **not** a standalone pipeline stage — it runs *inside* those phases whenever a step is about to flood the main context with secondary, noisy work.

**Two delegation shapes.** This reference covers **focus delegation** — a single noisy *step* inside your own phase run: minimal rights, no `.dev_flow/` writes, stage-and-report, no dialogue. Delegating a whole secondary *task* is the [subtask phase](../phases/subtask.md) — there the subagent is a **full dev-flow participant with delegated rights**: it assembles its own context, joins the task file as a contributor, persists skills/cache per the phase protocols, and can converse with its initiator. The moment a delegated step outgrows one exchange, it is a subtask.

## Why this exists

The main agent is the conductor. It holds the plan, the specification, and the task state, and it makes the calls at the gates. That working focus is the scarce resource — the thing that lets it write code that actually matches the spec and notice when something drifts.

Focus erodes when secondary, noisy work shares the same context: full test output, build logs, screenshots, wide code searches, reproduction traces. None of it is wrong to *produce* — but most of it doesn't need to *stay*. A thousand lines of passing-test output buys one bit of information ("they passed") while pushing the plan and the half-written change out of view. By the time the context is thick with logs, the agent is reconstructing what it was doing instead of doing it.

The cure is to hand that work to a subagent with its own context and keep only the conclusion. The subagent spends its context on the noise; the main agent gets the one bit that mattered and stays on task. This is the same instinct as `handle-uncertainty` and Interview Mode — protect the expensive thing (here, attention) from the cheap thing (here, volume).

## Where the line falls

**Keep in the main context — the core.** Work that needs the full plan + spec + rules, or that *is* the focus:
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

A delegated subagent returns the **conclusion, not the dump**: a verdict plus what's actionable, with the full log written to a file and referenced by path — under the project workspace `/tmp/{project-slug}/` with a timestamped name, per the [Resource Cache](cache.md) workspace discipline. In this focus shape, anything durable among the artifacts is promoted to `.dev_flow/cache/` by the calling agent — the helper only stages. (A task-delegated subagent persists per the phase protocol itself; see the [subtask phase](../phases/subtask.md).)

Skip this and the win evaporates — the noise simply relocates from the tool output into the subagent's report, and the main context is flooded just the same, now with an extra hop. The discipline is the whole point: *the report is the conclusion, not the transcript.*

There is a flip side to keeping only the conclusion: you deliberately did **not** read the raw material, so the helper's framing is all you have — and a plausible-but-wrong verdict rides that trust straight into your work. So when a delegated conclusion is **load-bearing** — it will drive a code change, block a fix, or land in a document — spot-check the *specific* claim against primary evidence before acting: open the one file the verdict turns on, re-derive the one number. That is cheap and targeted, not a re-read of the dump — and it is what stands between a wrong conclusion and a real defect, especially across a **nested** hop where the conclusion is now two levels removed from anyone who saw the evidence.

Review already works this way for a second reason — a clean-context subagent reads the diff without the author's rationalizations, so its judgment is unbiased. Verify and diagnosis share the shape; they just optimize for focus rather than impartiality.

[Experience Capture](experience-capture.md) borrows the same move: distilling a closing segment is a noisy read whose *conclusion* — a short summary plus any durable lesson — is all that comes back, so the checkpoint delegates the read and keeps only the distillation.

## Fit the model to the task — by its nature, not its name

A delegated step can run on a different model than the main agent, chosen by the *character* of the work:
- Mechanical, narrow-output work (run a suite, grep, summarize a log) runs fine on a fast, cheap model.
- Work that needs judgment (untangling a root cause, weighing an architectural option) wants a stronger one.
- When unsure, prefer the stronger one — a wrong result costs more than the tokens saved.

Describe the *nature* of the task and let the orchestrator map it to whatever model is configured. **Never hardcode model names** — they age, and they tie the skill to one provider. Keep the name→tier mapping in the orchestrator's config, the same way the engine resolves providers by convention rather than a whitelist. The choice then survives new models arriving and current ones retiring.

## Scenarios

A quick reference for the recurring steps. "Delegate?" is a default, not a law — a trivial one-test run isn't worth the overhead of a subagent, and a genuinely tangled diagnosis might need the main agent's full context. Use judgment.

| Step (phase) | Delegate? | Subagent returns | Fits |
|--------------|-----------|------------------|------|
| Write / edit code (implement) | No — core | — | main agent |
| Decisions at gates, task state | No — core | — | main agent |
| Run test suite + parse failures (test, fix) | Yes | failing tests + reasons, log path | fast / cheap |
| Build / verify run (verify) | Yes | pass/fail + failures that matter, log path | fast / cheap |
| Regression / integration / live (verify) | Yes | verdict + failures, artifact paths | fast / cheap |
| Diagnosis: reproduce + instrument (fix) | Yes | confirmed cause + evidence, log path | fast / cheap |
| Spike investigation (research) — one researcher, or a parallel **panel** (one per perspective) for a many-sided topic | Yes | verdict + findings + recommendation; raw material stays in the spike file. A panel fans the same questions across lenses; each returns conclusion-not-dump and the main agent synthesizes — see [research Step 2b](../phases/research.md#step-2b-perspective-panel-optional) | standard / strong — weighing sources and trade-offs needs judgment |
| Broad code search ("where is X used?") | Yes | findings + file:line list | fast / cheap |
| Distill a closing segment at a checkpoint (experience capture) | Yes | segment summary + any proposed lessons | standard — judging what's durable |
| Pre-commit review | Yes (clean ctx) | blocking issues + warnings | standard / strong |
| Untangle a tangled root cause | Borderline | cause + reasoning | stronger |

## Named specialists and the routing reflex

Some noisy steps recur enough in a given project to be worth a **named, reusable helper** rather than an ad-hoc subagent each time — a wide/structural **code search**, triaging a large **log/trace**, analysing a **screenshot/image**. When that pattern appears, create a **project-specific specialist role** under `.dev_flow/roles/` (the "new specialization" path in [Roles](roles.md)) and route to it thereafter. These are **not shipped with the skill**: a generic `log-analyst` cannot know your log format, nor a `screenshot-analyst` your UI — so a specialist is born where that knowledge lives, the project, tailored to its log formats, its screens, its module layout.

A specialist is a **read-only focus helper**: it owns one task-signature, returns the conclusion (not the dump), stages raw output in the workspace by path, and stays out of `.dev_flow/` task context, the contributor list, and commits — its one writable target is its own role-local memory.

**Route by description, not a dispatcher.** Match the noisy step against each specialist role's `description` and send it to the one that fits; there is no central routing table. A trivial one-off (a single-line grep) is not worth delegating — do it inline. But when a matching specialist already exists, prefer it over an inline pass for any *recurring* noisy step — it exists because the step recurs, and it carries the project's accumulated traps; only a genuinely trivial one-off stays inline.

**They accumulate, and they self-compact.** A specialist warm-starts from a small role-local memory of distilled heuristics (`.dev_flow/roles/<name>.memory.md`) so the next run starts ahead of cold; what proves broadly useful is *proposed* as a skill via [Experience Capture](experience-capture.md), never self-written. Crucially they store **only the distillation, never the raw payload** — raw output stays staged in the workspace, referenced by path. Because this memory is self-written **per call and ungated** (only skill promotion passes a gate), a heuristic is a *falsifiable, evidence-scoped observation* ("callers wrapping `.refresh()` in `try/except` **may** depend on the raise — check each"), never a standing verdict or absolute blocker — an over-stated heuristic poisons every later warm-start, and nothing downstream re-reads the evidence to catch it. Whether an instance stays warm across a burst or re-spawns per call follows the work: one that builds an expensive reusable artifact (a repo/symbol map) is kept alive across ≥2 calls, while large call-specific inputs (a log, an image) re-spawn fresh and warm-start from memory.

## How

The spawning mechanics live in the [Subtask phase](../phases/subtask.md): formulate the brief, spawn the subagent (picking type and model), and take back the conclusion. For a focus delegation the brief is a narrow step description; for a task delegation it is "keys, not content" — task + role + context hints, and the subagent assembles the rest. Use delegation not only as the explicit `/dev-flow subtask` command but as a *reflex* during implement / fix / verify — the moment a step is about to dump noise into the main context, delegate it.

Project-specific delegation practice (how *this* project runs its tests, where its live scenarios live, what counts as pass/fail) belongs in a project role overlay under `.dev_flow/roles/`, not here — see [Roles](roles.md) for using and creating them.

These scenarios and rules are **projections, not a formal protocol** — heuristics the agent applies with judgment. When a call is genuinely unclear, it asks the initiator rather than following an algorithm. Don't encode merge rules or rigid MUST/NEVER machinery here; keep the guidance in prose.
