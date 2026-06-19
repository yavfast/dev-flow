# Phase 7: Verify — Regression, Integration & Live Testing

## Purpose

After functional tests pass and code review is approved, verify the changes at a broader scope: regression testing, integration testing, and live verification (launching the app/service to check end-to-end behavior).

This phase catches issues that unit/mock tests cannot — broken integrations, configuration problems, UI regressions, and real-world service interactions.

## Delegation

This is the noisiest phase in the pipeline — regression suites, integration logs, live runs, screenshots — and almost none of that output needs to reach the main context. Run verification through a subagent (the clean-context shape Review uses): it returns a verdict plus the failures that matter, with full logs in a file referenced by path. See **[Delegation for Focus](../references/delegation.md)**.

**Artifacts.** Run output and screenshots go to the project workspace — `/tmp/{project-slug}/logs/` and `/tmp/{project-slug}/screenshots/`, timestamped (`{name}_YYYYMMDD_HHMMSS.{ext}`), never numeric suffixes. A capture worth keeping across sessions (e.g. a reference screenshot future runs compare against) is promoted to `.dev_flow/cache/app/` by the agent running this phase (focus helpers stage and report). See [Resource Cache](../references/cache.md).

## Command

```
/dev-flow verify [target]
```

The target is optional. Without it, verification scope is determined from the current active context and the nature of the changes.

### Examples

```
/dev-flow verify                           # Verify current active task
/dev-flow verify auth module               # Verify auth module integration
/dev-flow verify integration sync          # Run integration tests for sync
/dev-flow verify live api endpoints        # Run live tests against real API
/dev-flow verify regression                # Run full regression test suite
```

## Activation Condition

This phase activates when **at least one** of the following is true:
1. The project has integration or end-to-end tests.
2. The project can be launched locally (dev server, app build, service start).
3. The changes affect user-facing behavior, external integrations, or configurations.

If none apply — skip directly to commit approval.

## Verification Categories

| Level | Type | Purpose | When to run |
|-------|------|---------|-------------|
| 1 | **Regression tests** | Run the full test suite (or a broader subset) to catch regressions | When changes affect shared code, utilities, or core modules |
| 2 | **Integration tests** | Verify interaction between real components | When contracts between modules change |
| 3 | **Live tests** | Launch the app/service and verify changed behavior end-to-end | When user-facing behavior, integrations, or configurations change |

### Live Tests Include

- Running requests against a real API and verifying responses
- Launching the application or service and executing a user scenario
- Starting a dev server and checking the changed UI/behavior manually or with automation
- Running database migrations against a real database instance
- Verifying deployment configurations or environment-specific behavior

## Context Loading

Loading project knowledge is a **gate** (see [Project Knowledge Is Binding](../SKILL.md#project-knowledge-is-binding)):

**Skill check (gate).** MUST read `.dev_flow/skills/_index.yaml` and load skills for the verified functionality — integration specifics, environment setup, known pitfalls. See [skill phase](skill.md).

**Rule check (gate).** When `.dev_flow/rules/` exists, MUST read `.dev_flow/rules/_index.yaml` and load testing rules (test environments, CI conventions); verification MUST follow them. See [rule phase](rule.md).

## Safe Testing Principle

Integration and live tests operate on real data and real services. The primary rule: **never damage or delete user data during verification.**

### Guidelines

1. **Use a dedicated test account.** If the project supports multiple accounts or tenants, always run destructive operations (delete, modify, reset) under a test account, never a real user account.

2. **Use test entities.** For operations like deletion, parameter changes, or state transitions:
   - Use existing test entities if they are available (e.g., items marked as test data).
   - If none exist — create new entities with an explicit test marker (e.g., prefix `[TEST]`, tag `test`, or a dedicated flag) so they are clearly distinguishable from real data.
   - After verification, clean up test entities unless they serve as fixtures.

3. **Ask before touching real data.** If a verification scenario requires interaction with real user data and there is no safe alternative — stop and ask the user for explicit confirmation before proceeding. Describe exactly which data will be affected and what the operation does.

4. **Prefer read-only checks first.** When verifying a feature, start with read-only operations (list, get, search) before attempting write operations (create, update, delete).

5. **Sandbox environments.** If the project has a staging, sandbox, or dev environment — prefer it over production for all live tests.

### What counts as a destructive operation

- Deleting records, files, or resources
- Modifying user settings, permissions, or credentials
- Sending real notifications (email, SMS, push)
- Triggering payments, subscriptions, or billing events
- Altering shared state (queues, caches, feature flags)

When in doubt, treat the operation as destructive.

## Verification Workflow

```
1. Determine which verification levels are needed based on the changes
2. Ask user permission before creating new integration/live test scenarios
3. Run verification level by level (regression → integration → live)
4. If any verification fails:
   a. Analyze the failure — a code bug, config issue, environment problem,
      or a spec/plan defect? (the latter → escalate upstream first,
      see Upstream Escalation: ../references/escalation.md)
   b. Fix the root cause
   c. Re-run functional tests (Test phase) on the fix
   d. Re-run code review (Review phase) on the fix
   e. Re-run the failed verification level and all subsequent levels
5. All verification passes → proceed to commit approval
```

### Fix Cycle

When verification finds an issue that requires a code change:

```
Verify fails
  → Fix code
  → Re-run functional tests (unit + mock) on the fix
  → Re-run code review on the fix
  → Re-run verification from the failed level
```

This cycle repeats until all verification passes. Each iteration is smaller because the fix is targeted.

**When the failure points upstream.** Verify is often the first place reality pushes back on the *documents*: a spec'd limit that does not survive a live run, an integration contract that cannot hold, a plan technology decision that fails in practice. Bending the code there would make it correct per document and broken per reality. Escalate instead — fix the owning document (interview if it is a fork), re-pass its gate, then resume this cycle. See **[Upstream Escalation](../references/escalation.md)**.

### Manual Verification

When automated verification is not available, provide the user with a step-by-step checklist of actions to perform:

1. **Prerequisites** — running services, test data, environment setup
2. **Step-by-step actions** the user should perform
3. **Expected results** at each step
4. **What to look for** if something goes wrong

**Example:**
```
## Manual Verification

Prerequisites:
- Database running on localhost:5432 with test data loaded

Steps:
1. Start the dev server: `npm run dev`
2. Open http://localhost:3000/login
3. Enter valid credentials and click "Login"
4. Expected: redirect to /dashboard, no console errors
5. Click "Settings" in the sidebar
6. Expected: the new "Notifications" tab is visible
7. Toggle a notification setting
8. Expected: success toast, setting persisted after page refresh
```

### Running Complex Test Scenarios

For integration and live tests that require setup:

1. **Check prerequisites** — required services, test databases, API keys.
2. **Prepare test environment** — fixtures, seed data, mock servers.
3. **Run the scenario** — capture output, logs, timing.
4. **Verify results** — compare against spec expectations.
5. **Clean up** — tear down test fixtures, restore state.

## Verification Result Reporting

After running verification, report:

| Item | Detail |
|------|--------|
| Verification levels run | List of levels executed |
| Regression tests | Total count / passed / failed |
| Integration tests | Total count / passed / failed |
| Live tests | Scenario descriptions + pass/fail |
| Manual verification | Steps provided to user (if applicable) |
| Fix cycles | Number of fix → test → review → verify iterations |

## Gate Check (Verify -> Commit)

Before proceeding to commit:
- [ ] All regression tests pass (if applicable)
- [ ] All integration tests pass (if applicable)
- [ ] Live tests pass or manual verification completed by user
- [ ] No fix cycles remain incomplete
- [ ] Ask the user for explicit commit approval

## Anti-Patterns

- Skipping verification for "small" changes that affect shared code
- Running integration/live tests before functional tests and review pass
- Fixing a verification failure without re-running functional tests + review
- Bending code to a spec the failure just disproved — escalate upstream instead (see [Upstream Escalation](../references/escalation.md))
- Not providing manual verification steps when no automated tests exist
- Creating new integration/live test scenarios without asking the user first
- Committing after review without running verification
