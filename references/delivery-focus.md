# Delivery Focus — Priorities by Project Stage and a Watch on Delay

Cross-cutting discipline of every phase that plans or does work — [concept](../phases/concept.md), [specification](../phases/specification.md), [plan](../phases/plan.md), [implement](../phases/implement.md), [fix](../phases/fix.md), [test](../phases/testing.md) — of [review](../phases/review.md) and the [code audit](code-audit.md) for one classification, and of the [Transition Checkpoint](experience-capture.md). Not a pipeline stage and no command. A project rule in `.dev_flow/rules/` overrides its defaults.

## Contents

- [Why this exists](#why-this-exists) — work not needed now delays the work that is
- [Project stage](#project-stage) — `prototype` / `working` / `production`, inferred from facts, shown at the design sign-off
- [Needed now](#needed-now) — the check before an item enters a plan or the work; what each stage defers; what is never deferred
- [Security findings and the stage](#security-findings-and-the-stage) — a finding whose attack needs an exposure the stage does not have is deferred
- [Delay watch](#delay-watch) — signals of delay in the agent's own process (searching, costly calls, missing access, wide and slow runs included), the correction, and a delegate's `Friction:` line
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
| `search` | A file or fact the work needs is found only by the third search for it, or not at all |
| `costly-call` | The harness cuts a returned result or saves it aside, or the result is read again to get the rest |
| `fan-out` | The facts of one decision come in more gathering rounds than their dependencies required, or by one call per element where one filtering command answers ([Round-Trip Economy](round-trip-economy.md)) |
| `no-access` | Information the work needs sits behind a credential, a privilege or a resource that no channel or tool reaches |
| `brief-gap` | A delegate's run needed a fact its brief did not carry ([A delegate's run](#a-delegates-run)) |
| `wide-run` | A build or a test run is wider than its question: the whole project where a module answers, a release variant where a debug one answers, a `clean`, a cache wipe or a daemon stop when neither stale output was observed nor the build configuration or the toolchain changed ([Run Economy](run-economy.md#build-scope)) |
| `slow-run` | One build or test command takes 2 minutes or more and runs a third time in the session (its [run record](run-economy.md#run-record): `duration`, `count`) |

On a signal:

1. Write one Activity line in the task file: the signal and its cause.
2. Correct now: narrow the scope, defer the item ([Needed now](#needed-now)), drop a repeated check, or ask the developer when the correction changes what they approved.
3. At the [Transition Checkpoint](experience-capture.md#transition-checkpoint--the-primary-cadence), the cause becomes a lesson through the [owner menu](experience-capture.md#applying-experience--auto-apply-through-structural-gates) (never an auto-`must`): `search` and `brief-gap` → `pointer`, `no-access` → `access`, `wide-run` and `slow-run` → by [Run Economy → Process improvement](run-economy.md#process-improvement), at their first occurrence; any other cause once it is seen in this task and in an earlier one — `costly-call` → `tool`, the rest → `rule` or `tool` by the [checkability rung](experience-capture.md#checkability-rung).

### A delegate's run

A delegate watches its own run for `search`, `costly-call`, `fan-out`, `no-access` and `brief-gap` — the cases the brief's friction request names — and ends its report with a `Friction:` line: one item per signal that fired — the signal and its cause; none fired → `Friction: none`. Every brief carries the request in the words of [Delegation for Focus → How](delegation.md#how), so a delegate that never reads this file keeps it. The initiator writes each item as an Activity line of its own task (step 1); the delegate's causes are lessons of that task.

A report with no `Friction:` line, from a run that ended `blocked` or `stopped`, wrapped up, went down a step of the [stall ladder](supervision.md#the-stall-ladder), passed its run bounds, named a fact or an access it lacked, or was the third run of the same kind in the task → the initiator continues that delegate once and asks for the line. A delegate that cannot be continued → its friction is `unobserved` ([Evidence Discipline](evidence-discipline.md)). On the developer's request the initiator does the same for any run. Neither a new subagent nor the initiator reads the delegate's transcript for this.

## Boundaries

- The stage and the watch change the priority of work. They never change a gate criterion, a developer checkpoint, a project rule, or a `must` check of the current stage.
- A deferral is never silent: every deferred item has its record and its trigger.
