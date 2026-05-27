# Phase: Fix — Bug Investigation and Resolution

## Purpose

Investigate a reported problem, plan the fix, implement it, and verify the result.
Unlike the full concept-spec-plan pipeline, `fix` is a streamlined path for bug fixes
and corrections that do not require new concepts or specifications.

## Command

```
/dev-flow fix <problem description>
```

The description is freeform, in any language. It can be a bug report, error message,
user-visible symptom, or a reference to a specific class/method.

### Examples

```
/dev-flow fix Падає NPE при відкритті файлу без розширення
/dev-flow fix Upload progress bar freezes at 99% on slow connections
/dev-flow fix UserService не оновлює кеш після зміни профілю
/dev-flow fix ClassCastException in EventHandler.onInit when payload is null
```

## Procedure

### Step 0: Check skills

Before analyzing the bug, read `.dev_flow/skills/_index.yaml` and check if a skill exists
for the technology area where the bug occurs (e.g., networking, storage, UI, event bus).
If relevant skill(s) found — load them. They may contain known "Pitfalls" that explain
the bug. If research is needed during analysis — save results to `.dev_flow/skills/` after
the fix is complete. See [skill phase](skill.md).

### Step 1: Analyze

1. **Understand the symptom** — parse the problem description.
2. **Locate the code** — use ai-search, grep, or read relevant files to find the
   area of the codebase where the problem likely originates.
3. **Reproduce mentally** — trace the execution path to understand how the symptom occurs.
4. **Identify root cause** — distinguish root cause from symptoms. If the root cause
   is unclear, list hypotheses ranked by probability. If two or more hypotheses are
   *materially different* — they lead to different fixes — and the developer likely
   holds the deciding context, surface them as marked options instead of betting on
   the top-ranked one (see [Interview Mode in Fix](#interview-mode-in-fix)).
5. **Check related code** — look for the same pattern elsewhere that may have
   the same bug (if it's a pattern-level issue, not a one-off).

If context is insufficient to identify the root cause, ask the user targeted questions
(max 3). Examples: "Does this happen on first launch or only after re-login?",
"Which runtime/environment version?", "Is there a stack trace?"

### Step 2: Plan the fix

1. **Describe the root cause** — one paragraph explaining what goes wrong and why.
2. **Propose the fix** — list specific changes (files, methods, what changes and why).
   If the same root cause admits *materially different* strategies — a quick
   **band-aid**, a **structural** fix, a **workaround** — with different long-term
   cost, surface them as marked options with a recommendation, not a single silent
   pick (see [Interview Mode in Fix](#interview-mode-in-fix)).
3. **Assess impact** — note if the fix touches shared code that other features depend on.
4. **Check rules** — read `.dev_flow/rules/_index.yaml` (if exists) to ensure the fix
   will comply with project coding rules.
5. **Present to user** — show the analysis and the fix option(s). Wait for approval
   (or the developer's chosen/composed option) before proceeding to implementation.

### Step 3: Implement

1. Apply the fix following the approved plan.
2. Comply with `.dev_flow/rules/` (if exists).
3. If the fix reveals a pattern-level issue, fix all occurrences found in Step 1.
4. Add comments referencing the problem only if the fix is non-obvious.
5. **Auto-extract rules** — if the problem description contains an explicit directive
   (see Rule Detection below), add or update the corresponding rule in `.dev_flow/rules/`
   using the [rule phase](rule.md) procedure. Report the added rule in the result.

### Step 4: Verify

1. **Build** — run the project's build command and confirm success.
   Use whatever build tool the project uses (e.g., `make`, `npm run build`,
   `./gradlew build`, `go build ./...`, `cargo build`, `python -m build`).
   If the fix is in a specific module, build that module.

2. **Test** — if applicable tests exist, run them using the project's test runner
   (e.g., `pytest`, `npm test`, `./gradlew test`, `go test ./...`, `cargo test`).

3. **Review the diff** — re-read all changed files to check for regressions,
   missed edge cases, or rule violations.

4. **Report result** — summarize to the user:
   - Root cause (one sentence)
   - What was changed (files and methods)
   - Verification result (build/test status)
   - Any remaining concerns or follow-up items

### Step 5: Check documentation impact

After the fix is verified, check whether the code changes require updates to
related documentation artifacts:

1. **Specifications** — does the fix change behavior described in any `*.sp.md`?
   (e.g., different error handling, changed contract, new edge case).
2. **Concepts** — does the fix affect architectural assumptions in any `*.concept.md`?
   (rare for bug fixes, but possible if the root cause reveals a design flaw).
3. **Plans** — does the fix invalidate or complete tasks in any `*.plan.md`?
4. **Tests** — do existing tests need updating, or should new test cases be added
   to cover the fixed scenario?
5. **Rules** — does the fix expose a missing rule? (already covered in Step 3 Rule Detection).

If any updates are needed, either apply them immediately (for small changes)
or flag them explicitly in the result report for the user to decide.

## Interview Mode in Fix

A bug fix hides two forks that are easy to resolve silently and expensive to get
wrong. When either is *material* — the options lead to genuinely different code or
different long-term cost — do **not** pick one quietly. Surface it with marked
options (`A`/`B`/`C`) and a recommended answer, per
[Interview Mode](../references/interview-mode.md). Most bugs have one obvious fix, so
the over-asking discipline applies hard here: trigger this only on a real fork.

- **Root cause (Step 1).** When several plausible causes lead to *different* fixes and
  the developer likely holds the deciding context ("does it happen only after
  re-login?"), present the ranked hypotheses as options and let them confirm — rather
  than betting on the top-ranked one.
- **Fix strategy (Step 2).** The same cause often admits a quick **band-aid**, a
  **structural** fix, and a **workaround**, with very different long-term cost. Surface
  them; recommend one.

**The sanctioned stop-gap.** The Banned Phrases rule forbids silent "temporary" fixes —
but sometimes you genuinely must ship a stop-gap *now* (prod is down). Interview Mode's
**open decision with a resolution trigger** is exactly how: record the band-aid as the
chosen action **and** the proper fix as OPEN with a concrete trigger ("resolve by #123 /
next sprint"). That converts a silent band-aid — which rots into permanence — into a
tracked, owned decision.

**Where the decision is recorded** (a code fix usually has no design document of its own):
- Fix **changes a contract** → it propagates (Step 5) to the affected `*.sp.md` /
  `*.concept.md`; record the decision in that document's **Design Decisions** section.
- **Code-only** fix → record it in the **fix report** (Step 4) and the fix's **task
  file** under `.dev_flow/tasks/`. An OPEN decision's resolution trigger MUST become a
  tracked item (a plan/backlog entry, a rule, or a Blocking Issue in the task context)
  so it cannot quietly expire.

## Rule Detection

When the problem description contains an explicit coding directive — not just a bug
symptom — automatically extract it as a project rule.

**Trigger words (any language):**

| Language | Prohibition | Recommendation | Preference |
|----------|-------------|----------------|------------|
| UK | "заборонено", "не можна", "ніколи" | "рекомендується", "слід", "потрібно" | "бажано", "краще", "пріоритетно" |
| EN | "forbidden", "never", "must not", "prohibited" | "should", "recommended", "required" | "prefer", "better to", "advisable" |

**Severity mapping:**
- Prohibition → `must`
- Recommendation → `should`
- Preference → `prefer`

**Examples:**

```
/dev-flow fix використовувати `IOException ignored` заборонено, завжди логувати через Log.w
→ Fix: знайти та виправити всі `catch (IOException ignored)`
→ Rule added: NoIgnoredException (must, error-handling)

/dev-flow fix бажано використовувати вбудовані колекційні методи замість ручних циклів для перетворення списків
→ Fix: замінити ручні цикли на стандартні map/filter/reduce
→ Rule added: PreferCollectionMethods (prefer, style) — merged with existing CollectionOperationsStyle
```

When the directive matches an existing rule semantically, update that rule instead
of creating a duplicate.

## When NOT to Use Fix

Route to the full pipeline instead when:
- The "fix" is actually a new feature → `/dev-flow do <description>`
- The fix requires changing data structures or contracts → update spec first
- The fix requires architectural changes → concept → spec → plan → implement

## Relation to Other Phases

- Documentation impact is checked in **Step 5** — propagate is triggered
  automatically when specs or concepts need updating.
- If the fix reveals an undocumented pattern, suggest adding a **rule**.
- The fix does NOT touch any file under `.dev_flow/tasks/` or the dashboard
  `active_context.md` unless this agent is already a contributor on the task
  being worked on (i.e., it has its own Subtask block there). If you are
  running `/dev-flow fix` as a standalone command, create a fresh task file
  for the fix and add yourself as the first contributor.
