# Test Selection — Which Tests Run, and When

Cross-cutting sub-procedure of [test](../phases/testing.md), [implement](../phases/implement.md), [fix](../phases/fix.md), [review](../phases/review.md), [verify](../phases/verify.md), [do](../phases/do.md) and the [Refactoring Protocol](../phases/plan.md#refactoring-protocol) — every place an agent runs tests. Not a pipeline stage and no command. It is the owner of the `unrun-tests` row kind of the [proportionality block](proportionate-caution.md#proportionality-block).

## Contents

- [Why this exists](#why-this-exists) — a run with no subject spends wall-clock and gives no information
- [The selected set](#the-selected-set) — which tests a change binds; the narrowest runner selector; project overrides
- [Run trigger](#run-trigger) — the entry condition of a run; reusing a green result
- [Cadence](#cadence) — once per finished unit; the failure loop
- [Breadth beyond the set](#breadth-beyond-the-set) — the full suite only on request or rule; the `unrun-tests` row
- [Selection undeterminable](#selection-undeterminable) — widen to the module, then ask
- [Boundaries](#boundaries) — `must`/security fixes, gate criteria, commit sign-off

## Why this exists

A test run is a check, so it has a subject: the code the change touched. A test that does not bind to the change has no subject in it — running it adds wall-clock, not information. On a large suite (thousands of tests) a whole-suite run after each edit turns minutes of work into hours. This reference follows the entry-condition principle of [Verification Economy](verification-economy.md): the condition is an absent subject, not the cost of the run.

## The selected set

The **selected set** of a change is built from the change, never from the suite. It is the union of:

1. **Bound tests** — the tests the [Symbol reach](impact.md#symbol-reach) lists for each changed or added symbol: tests that call it, tests of its callers inside the reach list, and tests of the fixtures and config bound to it.
2. **Named tests** — tests the implemented plan phase's `Verify:` field names, and tests that carry a traceable ID (`[SP_XXX_…]`) of a changed contract.
3. **New and edited tests** — every test the change itself wrote or modified.

Run the set through the **narrowest selector the runner offers** — a file, a class, a test name, a tag, a related-tests mode — never the runner's bare whole-project command (`npm test`, `go test ./...`, `./gradlew test`, `pytest` with no target). A selection command named by a project rule, a skill, or a [project tool](project-tools.md) replaces the hand-built set and the hand-built command.

## Run trigger

A run of the selected set needs one of these facts, read from an artifact, not recalled:

| Trigger | True when | Read the fact from |
|---------|-----------|--------------------|
| `covered-change` | Code the set covers, or a test in the set, changed since the set's last green run — or no green result is recorded for the set | the diff against the recorded tree state; the working-memory record |
| `failure-recheck` | A test of the set failed and a fix for it is now in place | the failing run's exit code + the diff |
| `explicit-request` | The developer asked for the run | the developer's message |
| `rule-required` | A project rule or skill requires the run at this point | the loaded rule or skill |

No trigger → no run. A change to docs, comments, a test-irrelevant file, or code no selected test covers triggers nothing. A green result is recorded in the [session working memory](cache.md#session-working-memory-l1) `params` — the set, the tree state reference, the result — and is reused until `covered-change` fires; elapsed time alone never re-opens it.

## Cadence

- **Once per finished unit of work** — at the end of an implementation burst for one plan phase, at the end of a fix, at the end of a review fix round. Never after each edit inside the unit.
- **The failure loop.** When tests of the set fail, re-run only the failing tests until they pass, then the selected set once.
- **Repeat rounds.** A review or verify fix round re-runs only the part of the set that covers the fix's locations; the rest keeps its recorded green result unless `covered-change` fires for it.
- **Delegation.** A long run goes to a subagent that returns the verdict and the failures, with the full log in the workspace ([Delegation for Focus](delegation.md)).

## Breadth beyond the set

The **full suite**, or any set not derived from the change, runs only on `explicit-request` or `rule-required`. The [verify](../phases/verify.md) regression level widens to the tests of the reach list's dependents outside the changed module — still derived from the change, still never the whole suite by default.

What is left unrun is never silent. The owning phase appends an `unrun-tests` row to the task file's `## Proportionality` — `ref` is the change's scope (module or plan phase), `text` names what was not run (the suite beyond the selected set, its size when known). The row is shown at the commit sign-off; the developer's answer is `run-full-local`, `run-full-ci`, or `skip` ([Proportionate Caution](proportionate-caution.md#proportionality-block)). The phase report carries the same breadth as `unobserved` with the named reason ([Evidence Discipline](evidence-discipline.md)). The answers act as follows: `run-full-local` — run the full suite before the commit (an `explicit-request`); `run-full-ci` — commit and leave the full run to CI, the breadth stays `unobserved` locally; `skip` — no full run. A new set of unrun tests after an answer gets a new row, `#n+1`.

## Selection undeterminable

When the set cannot be computed — no diff, no mapping from the changed code to tests, a runner with no selector — the fallback is **not** the whole suite. Widen to the tests of the changed module (package, directory, test target), and ask the developer before going wider. This is the one place where the Verification Economy default "undeterminable → run in full" does not apply to the whole suite.

## Boundaries

- **`must` and security findings.** After their fix the covering part of the set always re-runs — a recorded green result is never reused for it. Breadth still follows this reference.
- **No gate criterion moves.** "All relevant functional tests pass" and "regression tests pass (if applicable)" read "relevant" and "applicable" through this reference.
- **Commit approval is untouched.** The `unrun-tests` row is shown at the commit sign-off; silence is never consent to skip, and the answer is the developer's.
- **Test authoring is untouched.** The [test phase](../phases/testing.md) rules on covering every contract, error case and invariant stay; this reference governs which tests run, not which tests exist.
