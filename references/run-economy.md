# Run Economy — A Build or Test Runs Once per State, at the Narrowest Scope and the Cheapest Variant

Cross-cutting sub-procedure of [implement](../phases/implement.md), [test](../phases/testing.md), [fix](../phases/fix.md), [review](../phases/review.md) (the `confirm` round, the fix of a failed round), [verify](../phases/verify.md) and of every delegate that builds or tests. Not a pipeline stage and no command. Which tests run is [Test Economy](test-economy.md#task-test-set); this file decides when a build or a test run repeats, how wide a build is and which variant it builds. A project rule in `.dev_flow/rules/` overrides its defaults.

## Contents

- [Why this exists](#why-this-exists) — a run on an unchanged state repeats a known answer; a full build answers a question nobody asked
- [The rule](#the-rule) — once per state, the narrowest scope, the cheapest variant, a record read in place of a run
- [Run record](#run-record) — what each run writes to the session working memory
- [Reuse](#reuse) — when a record answers in place of a run; gates, sign-offs and delegates read it; a failure is not re-run on the same state
- [Build scope](#build-scope) — the question picks the build; when the full or release build runs
- [Fast variant](#fast-variant) — what a test or debug build turns off, and where those checks run
- [Run map](#run-map) — the project's commands per module, a procedural project skill
- [Local switches](#local-switches) — speed switches the agent turns on without asking, and what is not one
- [Process improvement](#process-improvement) — a wide or slow run becomes a run-map line, a tool, or an `improvement` for the developer
- [Boundaries](#boundaries) — what the economy never skips

## Why this exists

A build or a test run costs the developer minutes and the context the tokens of its output. A run on a state that has already passed repeats a known answer. A full or release build after a change in one module answers a question the change did not ask. Neither adds quality; both delay the result.

## The rule

- A build or a test runs only when no result is recorded, something the run reads changed since its last result, or the outside cause a failed result named changed ([Reuse](#reuse)).
- A run answers one question: take the narrowest scope that answers it ([Build scope](#build-scope)) and the cheapest variant ([Fast variant](#fast-variant)).
- Each run writes a [run record](#run-record). A gate, a sign-off, a report, a `confirm` round and a delegate's claim read the record; none of them is a reason to run again.
- A run the economy leaves out is named where it runs instead (verify, CI) or written `unobserved` with its reason ([Evidence Discipline](evidence-discipline.md)).

## Run record

One row per run key in the `runs` file of the [session working memory](cache.md#session-working-memory-l1), replaced by key — one JSON object per line, the fields below as its keys (`result` as `{exit, passed, failed, cause}`, `null` for what does not apply), so a tool reads it without guessing; a line that does not parse is reported, the other lines still read:

| Field | Holds |
|-------|-------|
| `key` | The command line and its working directory |
| `inputs` | The paths the run reads: the module directories it builds or tests and the modules they depend on, the build files and build logic of the root and of those modules (settings, version catalogs, `buildSrc`, included builds), the test files it runs |
| `state` | The working-tree snapshot taken right before the run — the tree hash of [Incremental round scope](review-convergence.md#incremental-round-scope) |
| `variant` | `type-check` / `debug` / `release`; the switches that turned checks off; the [local switches](#local-switches) on; the toolchain version |
| `result` | The exit code; for tests, the counts passed and failed; for a failure whose log names a cause outside the inputs — a device or a service down, the network, a local switch — that cause |
| `duration` | Wall time |
| `count` | How many times this key ran in the session; carried over when the row is replaced |
| `log` | The full output, in `/tmp/{project-slug}/logs/` ([Resource Cache](cache.md)) |

Run with the output written to the `log` path; read the exit code and the failing lines, never the whole log ([Delegation for Focus](delegation.md#the-habit-that-makes-it-pay-off)). Write the row right after the run, pass or fail. A delegate that builds or tests returns the row of each run in its report; the initiator writes them into its own working memory ([subtask → Step 5](../phases/subtask.md#step-5-receive-the-full-report-and-integrate)).

## Reuse

Before a build or a test run, take a snapshot and look up the record of the same key:

| Verdict | True when | Action |
|---------|-----------|--------|
| `reuse` | The record passed, nothing in the `run` row below holds, and every path `git diff --name-only <record state> <snapshot>` lists lies outside the run's reach — in a module the run neither builds, tests nor depends on, and outside the root build logic — or holds no executable code: comments, docs, formatting ([When to run](test-economy.md#when-to-run)). The no-executable-code exemption holds only for a compile or a test run, whose result cannot depend on them; any other run (lint, formatting, static analysis, documentation generation) reuses only when no path in its reach changed. A path neither placed outside the reach nor exempted counts as a change | Do not run; cite the record — key, state, log |
| `known-failure` | The record failed and nothing in the `run` row holds | Do not run; the failure stands — fix it; a failure suspected flaky goes to the [diagnosis loop](../phases/fix.md#diagnosis-optional) (see [Boundaries](#boundaries)) |
| `run` | No record; a path in the run's reach changed that the `reuse` row does not exempt; a git-ignored or out-of-tree file the run reads, an environment variable, a local switch or the toolchain changed since the record — the snapshot does not hold them; or the outside cause a failed record names changed | Run; write the record |
| `undeterminable` | No git, or the snapshot fails | Run — absence of a fact favours the run ([Verification Economy](verification-economy.md#the-rule)) |

A passing record of a wider run on the same state — the module's whole test task, the full suite — answers a narrower one too.

**Where a record answers in place of a run.** The gate criteria that tests pass ([Test -> Review](../phases/testing.md#gate-check-test---review), [Verify -> Commit](../phases/verify.md#gate-check-verify---commit)), the D5 discriminator of a `confirm` round ([Verification Economy](verification-economy.md#discriminator--re-verifying-a-known-fix)), the phase reports and the commit sign-off read the passing record whose verdict for the current state is `reuse`. A delegate's claim is checked against its record — its `state` equals the initiator's snapshot and its `log` shows the exit — never by a run of the initiator's own.

**A failed run is not repeated on the same state** — unless the cause outside the inputs its record names changed since. A result that changes with no change of its inputs or of such a cause is a flaky test or build: report it and file a [todo](../phases/todo.md); never re-run until green.

## Build scope

The question picks the build. Take the first row that answers it:

| Question | Build |
|----------|-------|
| Does the edited code compile? | The compile or type-check task of the touched modules — `cargo check -p <crate>`, `tsc --noEmit -p <package>`, `./gradlew :<module>:compileDebugKotlin`, `go build ./<package>/...`, `mvn -pl <module> compile` |
| Do the task's tests pass? | The test task of the touched modules with explicit test paths; the runner builds what they need |
| Does the changed behaviour work in the running app? | The debug variant of the app module |
| Does the change break the modules that use a changed shared module? | The build of its dependents — `./gradlew :<module>:buildDependents`, `mvn -pl <module> -amd` — in verify, at the regression level |
| Does the shipped artifact work? | The full or release build, under [When the full build runs](#when-the-full-build-runs) |

Builds are incremental: no `clean`, no cache wipe, no daemon stop — unless the failure observed is stale output, or the build configuration or the toolchain changed.

Never in `implement` or `test`, nor in `fix` outside [its Verify step](../phases/fix.md#step-4-verify), without the developer's request: a build of the whole project where a module build answers, of the dependents, or of a release variant.

### When the full build runs

Once per task, in [verify](../phases/verify.md) — on the `fix` route, in [its Verify step](../phases/fix.md#step-4-verify) — after the last code change, when the diff touches what the full or release build changes:

- the build configuration, the dependencies, the toolchain version;
- shrinking or obfuscation rules;
- code reached by name — reflection, serialization, dependency injection by string, native bindings;
- code or resources of the release variant only.

Or when the developer asks. Otherwise the verify report — on the `fix` route, its result report — names CI as its place; with no CI it is `unobserved`: "release build not run: the diff touches none of what it changes".

## Fast variant

A test or debug build turns off what its question does not need:

- shrinking, obfuscation and release optimization;
- release signing;
- lint, static analysis, documentation generation;
- variants, flavors and ABIs the check does not use.

Use what the build already offers — a debug variant, a one-flavor task, a task exclusion (`-x lint`), a property the build reads — or a [local switch](#local-switches).

**Checks turned off run after the last code edit.** Lint and static analysis run once on the touched modules, after the last code edit and before the review; a failure is fixed like a failing test. A fix of a review or a verify finding that edits code runs them again on the modules it touched, unless their record's [Reuse](#reuse) verdict is `reuse`. Shrinking and obfuscation run with the full build when [it runs](#when-the-full-build-runs). A check with no run is named in the verify report — on the `fix` route, its result report — with CI as its place, or `unobserved` with its reason.

## Run map

The project's commands, so that no task works them out again: a procedural project skill `.dev_flow/skills/build/run-map.md` ([Procedural Skills](procedural-skills.md)), domain `build/` in `.dev_flow/skills/_index.yaml` with topics `build`, `test`, `run` and `applies_to: { phases: [implement, test, fix, verify] }`. It is project-specific usage, not build-tool basics ([Non-Triviality Filter](../phases/skill.md#non-triviality-filter)). One row per module:

| Column | Holds |
|--------|-------|
| `module` | Its path |
| `check` | The compile or type-check command |
| `build` | The narrow build command, with the fast-variant switches |
| `test` | The test command, with a placeholder for the test paths |
| `full` | The full or release build command |
| `off` | The checks the fast variant turns off, and where each runs |
| `time` | The duration of each command from the run records, with the date |

Its `version_or_context_scope` names the build tool and its version.

- **Read** before the first build or test run of a phase; the skills gate finds it by its phases.
- **Write** at the [Transition Checkpoint](experience-capture.md#transition-checkpoint--the-primary-cadence), without asking, as owner `skill`: a command narrower or faster than the map holds, or the first one for a module; a newer duration replaces an older one; a command that fails on the current tree is fixed or removed.
- **Absent** → the first task that builds or tests writes it from the commands it ran; no separate pass.

## Local switches

A switch that changes only the speed of the agent's own runs — never what the committed build produces or checks when CI or another person runs it — is turned on without asking:

- a flag or an environment variable of the run: build cache, parallel execution, configuration cache, a kept daemon, offline mode while the dependencies are unchanged, memory;
- a line in a project-local file git ignores (`local.properties`);
- a line in a user-level build configuration (`~/.gradle/gradle.properties`) — it changes every project on the machine.

A switch written to a file is recorded as a `software` entry of `.dev_flow/profile/environment.md` — the file and the key, the new value and the old one, the measured effect ([Adaptation Profile](adaptation-profile.md#entry-form)) — and named in the phase report, so the developer can revert it.

Not a local switch: a change of a committed file or of the CI configuration — that is an [`improvement`](#process-improvement). A flag of the agent's own run that skips a check (`-x lint`) is the [fast variant](#fast-variant); the check runs as that section says.

## Process improvement

The run records measure the process. Two [Delay watch](delivery-focus.md#delay-watch) signals read them, `wide-run` and `slow-run`:

| Signal | Correction now | Lesson at the Transition Checkpoint |
|--------|----------------|--------------------------------------|
| `wide-run` | Narrow the run by [Build scope](#build-scope) and [Fast variant](#fast-variant) | The narrow command → the run map |
| `slow-run` | Look for a faster route the build offers — a narrower task, a fast variant, a [local switch](#local-switches); found → use it | Found → the run map; changed paths mapped to commands by hand in this task and an earlier one → a `tool` of kind `operation` ([Promote](project-tools.md#promote)); a route that needs a committed change → `improvement` |

**`improvement`** — a change of the project's committed build, test or CI setup that removes a measured delay: a debug variant without obfuscation, module-level tasks, a build cache, test sharding. A todo names the change, its measured cost (duration × `count`, from the records) and the expected saving; the phase report and the commit sign-off name it ([SKILL.md → Developer Checkpoints](../SKILL.md#developer-checkpoints)). It is built only after the developer's yes, as its own task through the pipeline.

## Boundaries

- A run with a path in its reach changed that the `reuse` row does not exempt is never skipped; a run a project rule requires runs; no gate criterion and no developer checkpoint moves.
- The scope follows the question, never the cost: a question the narrow run cannot answer gets the wider run.
- A missing fact favours the run: a fresh session with no records runs once.
- A [diagnosis loop](../phases/fix.md#diagnosis-optional) runs its reproduction on one state as often as the diagnosis needs — the reuse verdict and the failed-run rule do not apply to it.
