# Test Economy — A Permanent Test Needs a Keep Reason

Cross-cutting sub-procedure of [test](../phases/testing.md), [implement](../phases/implement.md), [plan → Refactoring Protocol](../phases/plan.md#refactoring-protocol), [fix](../phases/fix.md), [specification](../phases/specification.md), [review](../phases/review.md) and [audit](../phases/audit.md). A project rule in `.dev_flow/rules/testing.md` overrides every default in this file ([Project rule](#project-rule)).

## The rule

- A permanent test is maintained code. Create one only with a [keep reason](#keep-reasons). Check all other behaviour with a temporary test, a read, a build or a run.
- Check every verification criterion the change touches. A criterion that is not checked is `unobserved` with its reason ([Evidence Discipline](evidence-discipline.md)). Never drop a criterion to save a test.
- No unit or mock test needs the developer's permission.
- Run only the [task's tests](#task-test-set). The full suite runs only in [verify](../phases/verify.md) or on the developer's request.

## Keep reasons

Closed set. A permanent test without one is not created.

| Reason | True when | Fact from |
|--------|-----------|-----------|
| `spec` | The test checks a section 05 criterion marked `keep` in the `Test` column | The criterion's traceable ID in the test |
| `complex` | The test checks logic of class `complex` ([Logic class](#logic-class)) | The agent's judgement against the class table |
| `regression` | The test reproduces a bug in `complex` logic, or a bug whose cause the `fix` diagnosis loop found | The `fix` report and the `Keep-Test` trailer |
| `rule` | A project rule requires the test; a project CI coverage threshold counts as a rule | The rule ID or the CI configuration |

The phase report names the keep reason of each new permanent test. The commit that adds the test carries one git trailer per test:

```
Keep-Test: <reason> <path>::<test case>
```

The test code does not name the reason.

A test the developer asks for is permanent: mark its criterion `keep`.

## Logic class

| Class | Signs |
|-------|-------|
| `complex` | A defect survives a careful read of the code: calculation, format parsing or serialization, state machine, ordering or merge, concurrency, timing, retries, cache, transactions; or an area declared `Criticality: critical` |
| `trivial` | Field access, constructors, constants and enums, direct delegation and wiring, framework or library behaviour, data-structure shape, linear mapping, logging; a test that only asserts a mock call |

Doubt between the classes → `complex`.

## Choosing the form

For each criterion or behaviour you check in `implement`, `test` or `fix`:

```
if a project rule requires a test       -> permanent, reason rule
elif a 05 row with Test = keep           -> permanent, reason spec
elif logic class = complex               -> permanent, reason complex
else                                     -> temporary test | other check | none
none -> record the criterion unobserved with its reason
```

A temporary test that exposes `complex` behaviour moves into the project tree as a permanent test with reason `complex`.

## Temporary tests

| Place | When | Duty |
|-------|------|------|
| workspace | Default | `/tmp/{project-slug}/scratch/tests/<task-id>/` ([Resource Cache](cache.md#temporary-workspace-tmp-discipline)); run it with the project's runner and an explicit path |
| in-tree | The runner cannot find a test outside the code's package or module | On creation, write the L1 reminder "delete temporary test `<path>`"; delete the test before review |
| rule | A project rule sets another place | As the rule says |

- A temporary test ends with its task. A temporary test in the review diff is a finding.
- Never record a workspace path in a document or a task file.

## Golden capture

A refactor under the [Refactoring Protocol](../phases/plan.md#refactoring-protocol) keeps every output of the code it restructures — what a caller or a user can observe: a return value, printed or written data, a message sent. A `covering` test that passes unchanged proves this only for the outputs it asserts. For each output of the refactored code that no `covering` test asserts, capture a golden from the old code:

1. Before the first edit, run the old code on the inputs its callers pass and on the edge cases the spec names. Save the result as a temporary test whose expected value is that output ([Temporary tests](#temporary-tests)).
2. Write the comparison policy into the Refactor phase's `Goldens:` field ([Refactor Phase Format](../phases/plan.md#refactor-phase-format-in-plan)): exact, or the fields that change between runs (a time, a random ID, the order of an unordered set) with how each is normalized. A field leaves the comparison only with its reason in the policy.
3. After the refactor, run each golden with the task set. A mismatch the policy does not explain means the behaviour changed: restore it, or stop — the change is not a refactor and takes the spec route ([do → Change Classes](../phases/do.md#change-classes)).

A golden ends with its task, like every temporary test; one that exposes complex behaviour moves into the project tree as a permanent test with reason complex ([Choosing the form](#choosing-the-form)). An output that cannot be captured (the old code cannot be run) is `unobserved` with its reason.

## Task test set

| Part | Holds |
|------|-------|
| temp | The task's temporary tests |
| own | Permanent tests created or changed in the task's diff |
| covering | Permanent tests found for the code in the task's diff |

Find `covering` from traceable IDs in tests, code↔test file pairs and imports. No source gives a test → use the tests of the same module. No module → `unobserved` "no tests found". The full suite is never `covering`.

## When to run

| Moment | Runs |
|--------|------|
| After an edit | The task-set tests that cover the edited code; an edit without executable code (comment, docs, formatting) runs nothing |
| `test` phase | The whole task set, once, after the edits are complete |
| `fix` verify step | The whole task set, once, after the fix |

A test that passed after the last change to the code it covers is not run again ([Verification Economy](verification-economy.md)).

Never:

- run the full suite in `implement`, `test` or `fix` without the developer's request;
- run a test again when the code it covers did not change;
- delete a failing test in `implement`, `test` or `fix` to get a pass — restore it, then update the test or report the failure.

## Regression test after a fix

| Condition | Result |
|-----------|--------|
| The bug is in `complex` logic | Permanent test, reason `regression` |
| The `fix` diagnosis loop found the cause | Permanent test, reason `regression` |
| A `keep` criterion or a project rule covers the bug | Permanent test, reason `spec` or `rule` |
| Otherwise | Temporary test that reproduces the bug; it ends with the task |

## The `keep` marker

The last column of the specification tables `05_01`–`05_04` is `Test` ([template](../templates/specification.md)).

| Value | Meaning |
|-------|---------|
| `keep` | A permanent test is required; reason `spec` |
| `—` or empty | A check is required; the agent chooses its form |

Mark `keep` only for `complex` logic or on the developer's request.

## Pruning

Scope `audit tests` ([audit](../phases/audit.md)); part of `all`. Input: the project's test tree, rules, specifications.

**Origin** of a test file, from `git log --follow`:

| Origin | True when |
|--------|-----------|
| `agent` | Every commit that created or changed the file carries an agent attribution: a `Co-Authored-By`, `Assisted-by` or `Generated-by` trailer that names an AI agent or model, or the attribution pattern of a project rule |
| `human` | At least one commit of the file has no agent attribution, an agent commit without an agent attribution trailer included |
| `unknown` | The file is outside git or has no commits |

**Steps:**

1. **Filter** — keep a file only if its origin is `agent`, it is in git and it has no uncommitted changes. Drop all other files without judgement.
2. **Judge** — select a test case only if its logic class is `trivial`, it carries no traceable ID of a `keep` criterion, no `Keep-Test` trailer of a commit of its file names it with reason `spec`, `regression` or `rule`, and no project rule requires it. A `complex` trailer gets no protection: judge the class again.
3. **Delete** — delete the selected cases; delete a file that has no cases left and that no other file uses (imports it or calls a name it defines); delete helper code that only the deleted cases used, only in files that passed step 1.
4. **Control** — run the remaining tests of the changed files and of every test file that uses a changed or deleted file; when a rule or the CI sets a coverage threshold, run the coverage check too. The suite must build and pass. On failure, restore from git the failed test file and every helper file the run changed; list them in the report.
5. **Report** — for each deleted case: file, name, reason `trivial: <sign from Logic class>`.

`--dry-run` runs steps 1, 2 and 5 and deletes nothing.

Never change a file that did not pass step 1. Never delete a `keep` test, a test that a `Keep-Test` trailer protects, or a test a rule requires. A change of this kind is restored from git at once. A rule that disables pruning makes the scope report "disabled by rule `<id>`" and do nothing else.

## Report rows

The `test` phase report has two rows in place of "New tests added":

| Row | Content |
|-----|---------|
| Permanent tests added | Each test with its keep reason |
| Temporary checks | The number of temporary tests and the criteria they checked |

## Review check

Each finding is `should`:

- a new permanent test without a keep reason in the `test` report;
- a temporary test in the diff;
- a test deleted outside `audit tests` while the code it checked stays.

## Project rule

A rule in `.dev_flow/rules/testing.md` may:

- extend reason `rule` to a class of tests (for example, "a test for each public function");
- disable pruning;
- set the temporary-test place;
- set the agent attribution pattern for origin.

The rule wins over the defaults of this file.
