# Verification Economy — Every Check Has an Entry Condition

Cross-cutting sub-procedure of [review](../phases/review.md) and [verify](../phases/verify.md) (round mode), and of [review](../phases/review.md), [propagate](../phases/propagate.md), [audit](../phases/audit.md), [implement](../phases/implement.md) and [fix](../phases/fix.md) (unsolicited checks), and the session working memory in [Resource Cache](cache.md) (re-reads). Not a pipeline stage and no command.

Advisory throughout — no gate criterion changes. What changes is the work done inside a phase.

## Why this exists

Each discipline states a trigger to do **more** and rarely states the condition under which to do **nothing**; the bounds that exist are local and do not generalize to their neighbours. But an unnecessary check is not free caution — it spends the context whose loss is the drift [Application Enforcement](application-enforcement.md) exists to fight, and its findings dilute the real ones. **A check is an action, so it has a precondition:** something must have changed, or be unknown, for the check to have a subject.

## The rule

An entry condition is **computed from a fact outside the acting agent's own prose**: a diff, a file state, an exit code, a ledger entry, a developer message, a running command. "It looks fine to me" is not an entry condition — the same self-attestation ban as [Application Enforcement](application-enforcement.md).

| Verdict | Action |
|---------|--------|
| satisfied | Run the check in full |
| unsatisfied | Do not run it; write a suppression record |
| undeterminable | Run the check in full |

Absence of a fact always favours the check. A project with no vcs, no ledger, or a fresh session runs everything, exactly as before.

**A new check declares its kind.** Any check added to dev-flow names which of the conditions below governs it. One that declares none is unconditional and always runs — the safe default, never a licence to invent a fourth kind.

## Signal — unsolicited comparison

A comparison **nobody asked for** and **outside the current task** runs only on a signal. Closed set:

| Signal | True when | Read the fact from |
|--------|-----------|--------------------|
| `in-diff` | The file is in the current change's diff | vcs or the working tree |
| `referenced-id-changed` | A traceable ID the file references changed | the diff ∩ the ID map |
| `test-failed` | A test or build covering the area failed | the exit code |
| `explicit-request` | The developer asked for it | the developer's message |
| `audit-run` | `/dev-flow audit` is running in the matching scope | the running command |

Any one signal opens the check; signals do not carry weight and do not sum. No signal → suppress, and route a substantive remaining suspicion to a `todo` through the spotted-defect reflex.

Corpus-wide comparison keeps its declared home: [propagate](../phases/propagate.md#when-to-run-drift-detection) and the [`audit`](../phases/audit.md) `docs` scope. The gate returns it there from the individual tasks it spread into.

## Invalidator — re-reading

A re-read needs an invalidator. Closed set:

| Invalidator | True when | Read the fact from |
|-------------|-----------|--------------------|
| `compaction` | The context was compacted after the read | the session event |
| `foreign-write` | The target's state differs from the recorded state and you did not change it | mtime or hash against the ledger entry |
| `own-edit` | You edited the target after reading it | your own action log |
| `task-switch` | You moved to another task or subtask | the task state |
| `partial-read` | You read one fragment and now need another, or the whole file | the ledger entry's extent |
| `explicit-request` | The developer asks for a re-read | the developer's message |

Read every fact from the named artifact. Recalling that nothing changed is not a reading of the fact.

None fired → use what is already in context. Elapsed time alone is not an invalidator.

Boundaries:

- **`read-before-write` is not weakened.** A shared file (task file, index, external ticket) is re-read before **every** write, unconditionally — the ledger never answers for it. The rule exists because other contributors write concurrently, and a state comparison you compute yourself cannot see a same-second write.
- **Knowledge gates are untouched.** `.dev_flow/rules/_index.yaml` and `.dev_flow/skills/_index.yaml` stay mandatory at phase start. Re-triggering knowledge at the moment of action stays pointer-only per [Application Enforcement](application-enforcement.md), so it never re-reads a body either.

The ledger of what was read — target, state reference, time, extent — lives in the [session working memory](cache.md#session-working-memory-l1). No entry means "not read", never "read long ago".

## Discriminator — re-verifying a known fix

[Review Convergence](review-convergence.md) computes a finding's materiality, and `blocking` spawns the next round. That round gets a **mode**. A repeat round is `confirm` when **all** discriminators hold:

| # | Discriminator | Source |
|---|---------------|--------|
| D1 | The finding is not `must` and not a security class | finding fields |
| D2 | The review record names one concrete fix | the finding text, written before the fix existed |
| D3 | The fix diff is contained in the locations the finding named | diff |
| D4 | Nothing outside those locations changed in this round | diff |
| D5 | Tests that were green stayed green | exit code |

`confirm` is a mechanical check in the main context: the diff matches the prescribed fix and the tests that were green are still green. No clean-context reviewer is spawned. Any discriminator false or undeterminable → `full`, the current procedure.

**A confirm round that does not confirm escalates.** If the diff does not do what the finding prescribed, or a previously green test is not green, the round **replays as `full`** with the clean-context reviewer. A `confirm` round has two outcomes only — the finding closes, or the full round runs. It never closes a finding it did not confirm.

D2 lives in the artifact, never in the fixer's opinion: the prescription was written before the fix existed. D3 and D4 close "the fix may have broken something nearby" — a diff outside the named locations fails the discriminator.

The first pre-commit review round is always `full`. The same discriminator set scopes the [verify](../phases/verify.md) fix cycle.

## Suppression is written, never silent

A **suppressed check** — one the signal gate did not open — writes a record: what was not checked, which condition failed, and the **named missing fact**. It enters the phase report as evidence state `unobserved` per [Evidence Discipline](evidence-discipline.md). A reason states the absent fact ("no signal is true for this file"), not an intention ("judged unimportant").

An unrun and unmentioned check is indistinguishable from a completed one.

A reused read and a `confirm` round write **no** suppression record: nothing was skipped there. The re-read was unnecessary, and the round ran.

These cases refuse the suppression outright — run the check instead:

- The `todo` for a substantive remaining suspicion cannot be written, for any reason. Suppressing would lose the suspicion.
- You cannot tell whether the check belongs to the current task. Treat it as in scope.

## Boundaries

- **`must` findings and security findings are never economized** — not by mode, not by signal, not by ledger.
- **Commit approval is untouched.** The economy applies to the size of a check, never to the point where the decision passes to the developer.
- **Clean-context review is not skipped.** `confirm` applies to a **repeat** round of a review already performed.
- **Cost is not an entry condition.** The condition is an absent subject, not an expensive check.
- **No gate criterion moves.** Every transition's criteria list stays byte-identical.
