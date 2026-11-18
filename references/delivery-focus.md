# Delivery Focus — Priorities by Project Stage and a Watch on Delay

Cross-cutting discipline of every phase that plans or does work — [concept](../phases/concept.md), [specification](../phases/specification.md), [plan](../phases/plan.md), [implement](../phases/implement.md), [fix](../phases/fix.md), [test](../phases/testing.md) — of [review](../phases/review.md) and the [code audit](code-audit.md) for one classification, and of the [Transition Checkpoint](experience-capture.md). Not a pipeline stage and no command. A project rule in `.dev_flow/rules/` overrides its defaults.

## Contents

- [Why this exists](#why-this-exists) — work not needed now delays the work that is
- [Project stage](#project-stage) — `prototype` / `working` / `production`, inferred from facts, shown at the design sign-off
- [Needed now](#needed-now) — the check before an item enters a plan or the work; what each stage defers; what is never deferred
- [Security findings and the stage](#security-findings-and-the-stage) — a finding whose attack needs an exposure the stage does not have is deferred
- [Delay watch](#delay-watch) — signals of delay in the agent's own process and the correction
- [Boundaries](#boundaries) — what the stage and the watch never change

## Why this exists

Work that is not needed now is not free caution: it delays the work that is. A threat model for a product that does not run yet, a test for a case nobody has met, a third review round over a detail that changes nothing — each spends the developer's time before the product exists. The agent sets the priorities of its own work and checks its own process for delay; it does not wait to be told.

## Project stage

| Stage | True when |
|-------|-----------|
| `prototype` | The main use case does not yet run end to end, and nothing in the `production` row holds |
| `working` | The main use cases run; nobody outside the team uses the product, and it is not deployed where others reach it |
| `production` | The product is deployed where others reach it, people outside the team use it, or the task's goal or diff does either |

Check `production` first: an exposure decides the stage whatever runs. The agent that creates the task file infers the stage from facts — a run of the main use case, a deployment, its users — writes it in the task file's `Stage` row with the fact, and shows it at the design sign-off. A stage the developer states wins. Doubt between two stages → the later one. A diff that deploys the product where others reach it or opens it to people outside the team, or such a fact that arose after the row was written, makes the stage `production`: rewrite the `Stage` row with that fact before the review scopes its round ([review](../phases/review.md) step 0); when the design sign-off has passed, list the records of the first external exposure at the commit sign-off.

**The first external exposure.** While the stage written in the task file is `production`, every open record with the trigger "before the first external exposure" — in `.dev_flow/todos/` and the plan backlogs — has fired. List them at the task's design sign-off, or as a stop of its own when the task has no design documents, before any deploying step. A deferred security finding among them is `blocking` in that task's review, inside or outside its diff ([review](../phases/review.md) step 3). [Review](../phases/review.md) step 3 marks such a record `promoted`.

## Needed now

Before an item enters a concept, a spec, a plan phase or the work itself, ask: does the task's goal need it at the current stage?

- **Yes** → do it, scaled by the [change class](../phases/do.md#change-classes).
- **No** → do not plan it and do not build it. Record it once with a stage trigger — "before the first external exposure", "before the release", "at <N> users" — in the owning plan's backlog or as a [todo](../phases/todo.md) of flavor `deferred`.

Usually not needed yet:

| Stage | Defer |
|-------|-------|
| `prototype` | Threat models and hardening against attackers outside the team; rate limits and abuse handling; scale and performance tuning; migrations and backward compatibility; polish |
| `working` | Hardening against attackers the product cannot meet yet; scale beyond the expected load; polish |
| `production` | What the [YAGNI-gate](consequence-forecasting.md#the-yagni-gate) drops |

Tests are decided by [What gets a test](test-economy.md#what-gets-a-test) at every stage: an exotic case gets no test and no record.

Never deferred, at any stage:

- what the request, the developer, a project rule or a spec contract names;
- error handling that prevents the loss of real data;
- the handling of a real secret the work touches ([Proportionate Caution](proportionate-caution.md#secrets));
- a defect that breaks a main use case;
- the [never-cut set](consequence-forecasting.md#the-yagni-gate), except the controls it lets wait for the stage `production`.

## Security findings and the stage

A security finding of review — the Security check, or a type `vuln:*` or `missing-authz` — carries an `exposure`:

| Exposure | True when |
|----------|-----------|
| `external` | The attack needs users outside the team or a deployment others reach |
| `local` | The attack works from what already reaches the code at the current stage: the developer's machine, CI, a dependency, a file the product opens |

No `exposure` → `local`. No `Stage` row in the task file → infer and write it now ([Project stage](#project-stage)); until it is written, the finding blocks.

A finding with `exposure: external` while the stage written in the task file is not `production` is `deferrable`: it leaves the loop as a todo of flavor `deferred` with the trigger "before the first external exposure" ([Review Convergence](review-convergence.md#finding-materiality--computed-not-judged)). The todo is always filed: the at-capture YAGNI-gate does not drop it, and the review report names it. Each such deferral appends a `hardening` row (`ref` = the finding identity) to the [proportionality block](proportionate-caution.md#proportionality-block): `build` makes the finding `blocking` again, `decline` keeps the todo. A `local` finding and a `secret-exposure` finding block at every stage.

## Delay watch

The agent watches its own process for what delays the result.

| Signal | True when |
|--------|-----------|
| `review-rounds` | A third review round of the same task starts |
| `repeat-check` | A check runs again with no change to its subject |
| `rework` | A phase's output is redone after its gate passed |
| `oversized` | A document, a test or a check costs more than the change it serves |
| `off-goal` | An item serves no part of the task's goal at the current stage |
| `waiting` | The work waits on something the goal does not need |

On a signal:

1. Write one Activity line in the task file: the signal and its cause.
2. Correct now: narrow the scope, defer the item ([Needed now](#needed-now)), drop a repeated check, or ask the developer when the correction changes what they approved.
3. At the [Transition Checkpoint](experience-capture.md#transition-checkpoint--the-primary-cadence), a cause seen in this task and in an earlier one becomes a project rule through the structural gate (default `should`, never an auto-`must`).

## Boundaries

- The stage and the watch change the priority of work. They never change a gate criterion, a developer checkpoint, a project rule, or a `must` check of the current stage.
- A deferral is never silent: every deferred item has its record and its trigger.
