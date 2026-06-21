# Phase: Fix — Bug Investigation and Resolution

## Purpose

Investigate a reported problem, plan the fix, implement it, and verify the result. Unlike the full concept-spec-plan pipeline, `fix` is a streamlined path for bug fixes and corrections that do not require new concepts or specifications.

## Command

```
/dev-flow fix <problem description>
```

The description is freeform, in any language. It can be a bug report, error message, user-visible symptom, or a reference to a specific class/method.

### Examples

```
/dev-flow fix Падає NPE при відкритті файлу без розширення
/dev-flow fix Upload progress bar freezes at 99% on slow connections
/dev-flow fix UserService не оновлює кеш після зміни профілю
/dev-flow fix ClassCastException in EventHandler.onInit when payload is null
```

## Procedure

### Step 0: Check skills and roles

Before analyzing, MUST read `.dev_flow/skills/_index.yaml` and load any skill for the bug's technology area (networking, storage, UI, event bus, …) — their "Pitfalls" often explain the bug. If research is needed, save results to `.dev_flow/skills/` after the fix. See [skill phase](skill.md).

If you'll delegate the diagnosis loop or verification (see the **Delegation** note below) to a subagent, also check `.dev_flow/roles/_index.yaml` and reuse a fitting role rather than re-deriving one. See [Roles](../references/roles.md).

### Step 1: Analyze

1. **Understand the symptom** — parse the problem description.
2. **Locate the code** — use ai-search, grep, or read relevant files to find the area of the codebase where the problem likely originates.
3. **Confirm the cause from the evidence** — `fix` is normally invoked with evidence already in hand (a description, screenshot, stack trace, error message). Trace the execution path and check whether that evidence **corroborates** a specific located cause (does the stack / screenshot line up with the code site?). The goal here is confirmation from what you already have — not reproduction.
4. **Identify root cause** — distinguish root cause from symptoms. If the root cause is unclear, list hypotheses ranked by probability. If two or more hypotheses are *materially different* — they lead to different fixes — and the developer likely holds the deciding context, surface them as marked options instead of betting on the top-ranked one (see [Interview Mode in Fix](#interview-mode-in-fix)).
5. **Check related code** — look for the same pattern elsewhere that may have the same bug (if it's a pattern-level issue, not a one-off).

**Confidence gate — diagnose only when you must.** After analysis:
- **Confident, located cause** corroborated by the provided evidence → go straight to Step 2. Do **not** reproduce or build a feedback loop — the evidence already establishes the bug, so reproduction here only burns time and tokens. (Correctness is still confirmed at Step 4 Verify.)
- **No confident cause** — you cannot locate it, cannot tell a real bug from expected behavior, or several causes stay equally plausible → enter the optional [Diagnosis](#diagnosis-optional) sub-step.

### Step 2: Plan the fix

1. **Describe the root cause** — one paragraph explaining what goes wrong and why.
2. **Propose the fix** — list specific changes (files, methods, what changes and why). If the same root cause admits *materially different* strategies — a quick **band-aid**, a **structural** fix, a **workaround** — with different long-term cost, surface them as marked options with a recommendation, not a single silent pick (see [Interview Mode in Fix](#interview-mode-in-fix)).
3. **Assess impact** — note if the fix touches shared code that other features depend on.
4. **Check rules (gate).** When `.dev_flow/rules/` exists, MUST read `.dev_flow/rules/_index.yaml` and load rules for the code you'll change; the fix MUST comply (`must` = blocks).
5. **Present to user** — show the analysis and the fix option(s). Wait for approval (or the developer's chosen/composed option) before proceeding to implementation.

### Step 3: Implement

1. Apply the fix following the approved plan.
2. Comply with `.dev_flow/rules/` (`must` = blocks).
3. If the fix reveals a pattern-level issue, fix all occurrences found in Step 1.
4. Add comments referencing the problem only if the fix is non-obvious.
5. **Auto-extract rules** — if the problem description contains an explicit directive (see Rule Detection below), add or update the corresponding rule in `.dev_flow/rules/` using the [rule phase](rule.md) procedure. Report the added rule in the result.

### Step 4: Verify

Verify runs on **every** fix — including the fast path that skipped Diagnosis. A confident cause is not proof the fix is correct; build/test/observe must still confirm it. Where no automated test exists (e.g. a UI symptom), "observe" can be a cheap re-check — re-run and confirm the symptom is gone (an adb screenshot, a log line).

1. **Build** — run the project's build command and confirm success. Use whatever build tool the project uses (e.g., `make`, `npm run build`, `./gradlew build`, `go build ./...`, `cargo build`, `python -m build`). If the fix is in a specific module, build that module.

2. **Test** — if applicable tests exist, run them using the project's test runner (e.g., `pytest`, `npm test`, `./gradlew test`, `go test ./...`, `cargo test`).

3. **Review the diff** — re-read all changed files to check for regressions, missed edge cases, or rule violations. If a Diagnosis sub-step added `[DEBUG-xxxx]` instrumentation or a throwaway harness, remove it now — one `grep` on the tag — and confirm none survives.

4. **Report result** — summarize to the user:
   - Root cause (one sentence)
   - What was changed (files and methods)
   - Verification result (build/test status)
   - Any remaining concerns or follow-up items

### Step 5: Check documentation impact

After the fix is verified, check whether the code changes require updates to related documentation artifacts:

1. **Specifications** — does the fix change behavior described in any `*.sp.md`? (e.g., different error handling, changed contract, new edge case).
2. **Concepts** — does the fix affect architectural assumptions in any `*.concept.md`? (rare for bug fixes, but possible if the root cause reveals a design flaw).
3. **Plans** — does the fix invalidate or complete tasks in any `*.plan.md`?
4. **Tests** — do existing tests need updating, or should new test cases be added to cover the fixed scenario?
5. **Rules** — does the fix expose a missing rule? (already covered in Step 3 Rule Detection).

If any updates are needed, either apply them immediately (for small changes) or flag them explicitly in the result report for the user to decide.

When root-cause analysis shows the **document itself is defective** — the spec is ambiguous or contradicts reality, not merely lagging behind the fix — do not patch it silently as "impact": follow [Upstream Escalation](../references/escalation.md) (surface the correction or fork to the developer, update the owning document, re-pass its gate).

### Step 6: Reflect — harvest the lesson

A fix is the highest-yield **rule auto-discovery** moment: a bug just proved a gap. At the close of the fix (the [Transition Checkpoint](../references/experience-capture.md)), harvest what the root cause taught so the same bug *class* is prevented next time:

- **Rule from a violated invariant.** If the root cause was a violated invariant or convention with no matching rule in `.dev_flow/rules/` — write the rule automatically (default `should`), even when the problem text carried no explicit directive. (That explicit-directive path is Step 3's [Rule Detection](#rule-detection); this one fires on what the *analysis* revealed.) See [Project Rules → Auto-discovery](../SKILL.md#project-rules).
- **Skill from a non-obvious diagnosis.** If diagnosis produced broadly-useful, non-trivial knowledge — a stack-specific pitfall, a debugging technique that paid off — write/update a skill through the [skill phase](skill.md) non-triviality filter.

**Auto-apply, no permission prompt** — written through the structural rule/skill gate; **never auto-write a `must`**, and route a would-be `must` or a contradiction with an existing rule to an [independent clean-context review](../references/delegation.md) first. Every write is visible in the commit diff. Then distill the segment, demote the raw diagnosis turns, and promote durable parts to the fix's task file. See **[Experience Capture](../references/experience-capture.md)**. (For a code-only fix, the auto-written rule and any open decision are recorded as in [Interview Mode in Fix](#interview-mode-in-fix).)

## Diagnosis (optional)

An optional sub-step of **Step 1**, entered only when the confidence gate fails — you have no confident, evidence-corroborated cause. It imports the discipline of the `diagnose` skill, but **gated**: most fixes arrive with enough evidence to skip it.

**Why optional.** An earlier mandatory pre-diagnosis step was removed because reproduction was sometimes costlier than the fix. So diagnosis is opt-in — skipping it is the default, *entering* it is the justified choice.

**Before diagnosing — get context and a plan.** Do not start reproducing blindly. First gather the context the diagnosis needs and decide *how* you will obtain a pass/fail signal and roughly what it will cost. Then apply the cost gate.

**Cost gate (surface as a decision — [Interview Mode](../references/interview-mode.md)).** When a reliable signal is expensive or uncertain to build, present the options instead of silently sinking tokens into reproduction:
- `A` — **Diagnose (Recommended when feasible)** — run the loop below; when a signal is cheap to build.
- `B` — **Fix on the best hypothesis** and rely on Step 4 Verify — when reproduction is clearly costlier than the fix and the top hypothesis is well-supported.
- `C` — **Request an artifact** from the developer — repro steps, a HAR/log/crash dump, a timestamped screen recording, or environment access — rather than reconstructing it. (For a manual/UI repro, a structured human-in-the-loop script keeps the signal usable.)

**The diagnosis loop (path A)** — adapted from `diagnose`:
1. **Build a feedback loop** — a fast, deterministic pass/fail signal for the bug. It need **not** be a unit test: a failing test, a CLI/curl diff, a replayed trace, a throwaway harness, or — where a human must act — an adb/live check are all valid. Build the right loop and the bug is most of the way fixed.
2. **Reproduce** — run the loop; confirm it shows the **user's** symptom, not a nearby one (wrong bug → wrong fix). For flaky bugs, raise the reproduction *rate* until it is debuggable rather than chasing a clean repro.
3. **Falsifiable hypotheses** — 3–5, ranked, each stating a prediction ("if X is the cause, changing Y makes it disappear"). A hypothesis with no prediction is a vibe — sharpen or drop it. Show the ranked list to the developer when they hold deciding context (a cheap checkpoint); proceed on your ranking if they are away.
4. **Instrument** — one variable at a time; debugger / REPL over logs, logs over "log everything and grep". Tag every temporary log with a unique prefix (`[DEBUG-a4f2]`) so cleanup is a single `grep` (removed at Step 4). For performance regressions, measure first (baseline / profiler), then fix.

**No correct test seam is itself a finding.** If the bug cannot be locked down because the code has no seam exercising the real bug pattern (tangled callers, hidden coupling), do not fake it with a shallow test that gives false confidence — **record the missing seam** as a finding and route it upward (Step 5 → a `*.concept.md` note or a rule), the same way an architectural decision would be surfaced.

**Delegation.** The diagnosis loop (reproduce, instrument, parse output) and the build/test in Step 4 are the noisy work that erodes focus — delegate either to a subagent and keep the *fix decision* in the main context. See **[Delegation for Focus](../references/delegation.md)**.

**Diagnosis artifacts** (instrumented logs, repro dumps, screen captures) live in the project workspace — `/tmp/{project-slug}/logs/`, timestamped names — and die there. Promote to `.dev_flow/cache/` only what stays valuable past the fix (e.g. a HAR or recording a document references). See [Resource Cache](../references/cache.md).

## Interview Mode in Fix

A bug fix hides two forks that are easy to resolve silently and expensive to get wrong. When either is *material* — the options lead to genuinely different code or different long-term cost — do **not** pick one quietly. Surface it with marked options (`A`/`B`/`C`) and a recommended answer, per [Interview Mode](../references/interview-mode.md). Most bugs have one obvious fix, so the over-asking discipline applies hard here: trigger this only on a real fork.

- **Root cause (Step 1).** When several plausible causes lead to *different* fixes and the developer likely holds the deciding context ("does it happen only after re-login?"), present the ranked hypotheses as options and let them confirm — rather than betting on the top-ranked one.
- **Fix strategy (Step 2).** The same cause often admits a quick **band-aid**, a **structural** fix, and a **workaround**, with very different long-term cost. Surface them; recommend one.

**The sanctioned stop-gap.** The Banned Phrases rule forbids silent "temporary" fixes — but sometimes you genuinely must ship a stop-gap *now* (prod is down). Interview Mode's **open decision with a resolution trigger** is exactly how: record the band-aid as the chosen action **and** the proper fix as OPEN with a concrete trigger ("resolve by #123 / next sprint"). That converts a silent band-aid — which rots into permanence — into a tracked, owned decision.

**Where the decision is recorded** (a code fix usually has no design document of its own):
- Fix **changes a contract** → it propagates (Step 5) to the affected `*.sp.md` / `*.concept.md`; record the decision in that document's **Design Decisions** section.
- **Code-only** fix → record it in the **fix report** (Step 4) and the fix's **task file** under `.dev_flow/tasks/`. An OPEN decision's resolution trigger MUST become a tracked item (a plan/backlog entry, a rule, or a Blocking Issue in the task context) so it cannot quietly expire.

**Forecast check (advisory).** Forecast the consequences of the fix at *implement/fix altitude* — what else this change touches — and keep the free one-step check on the diagnosis loop (does the next probe/edit undo the last?). The gate is **strict**: expanding the fix's scope to speculative nearby improvements with no trigger defaults to `drop + record` (a backlog note), not into this fix. See [Consequence Forecasting](../references/consequence-forecasting.md).

## Rule Detection

When the problem description contains an explicit coding directive — not just a bug symptom — automatically extract it as a project rule.

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

When the directive matches an existing rule semantically, update that rule instead of creating a duplicate.

## When NOT to Use Fix

Route to the full pipeline instead when:
- The "fix" is actually a new feature → `/dev-flow do <description>`
- The fix requires changing data structures or contracts → update spec first
- The fix requires architectural changes → concept → spec → plan → implement

## Relation to Other Phases

- Documentation impact is checked in **Step 5** — propagate is triggered automatically when specs or concepts need updating.
- If the fix reveals an undocumented pattern, suggest adding a **rule**.
- The fix does NOT touch any file under `.dev_flow/tasks/` or the dashboard `active_context.md` unless this agent is already a contributor on the task being worked on (i.e., it has its own Subtask block there). If you are running `/dev-flow fix` as a standalone command, create a fresh task file for the fix and add yourself as the first contributor.
