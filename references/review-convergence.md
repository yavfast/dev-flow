# Review Convergence — Terminate the Argument, Never the Defect

Cross-cutting sub-procedure of [review](../phases/review.md) (reads), [concept](../phases/concept.md) and [specification](../phases/specification.md) (declare), [todo](../phases/todo.md) (receives), and [audit](../phases/audit.md) (grooms). Not a pipeline stage and no command.

It bounds the `Review → fix → re-Review` loop with a **declared criticality** the reviewer reads instead of judging, a **recurrence limit** on a single finding, and an **incremental round scope**.

Advisory throughout — the Review→Verify gate criteria are unchanged. What changes is which findings can reach `blocking`.

## Functional criticality — declared upstream, read downstream

Discrete, author-declared, optional. Never a number, never a score, never assigned by the reviewer.

| Value | Means |
|-------|-------|
| `peripheral` | Cosmetics, conveniences, rare paths with no effect on the result |
| `supporting` | Failure degrades convenience, not the result |
| `core` | The functionality the area exists for |
| `critical` | Failure breaks the application or violates security or data integrity |

Declaration levels, both optional:

- **application** — `Criticality:` in the concept header. The weight of this module inside the whole application. One per concept. A spec inherits its concept's line through `Concept:`.
- **module** — `Criticality:` under a concept mechanism section (`§3.x`) or a spec entity/contract section (`§01_xx`, `§02_xx`). The weight of this part inside its own module.

**Effective criticality** = the weakest among the **declared** levels. An undeclared level does not join the comparison — a silent section inherits its document's line. `unstated` only when no level declares anywhere.

Absence is valid, is never a documentation finding, and never lowers a severity.

## Finding materiality — computed, not judged

The reviewer assigns `severity` (`must` / `should` / `prefer`) as before, and does **not** assign materiality. Materiality is a pure function of `severity`, finding `type`, and effective criticality.

Overrides first, in order:

1. `type` in a security class (`vuln:*`, `secret-exposure`, `missing-authz`) → `blocking`
2. `severity: must` → `blocking`

Then the table (so `severity` is `should` or `prefer`):

| severity ↓ \ criticality → | `critical` | `core` | `supporting` | `peripheral` | unstated |
|---|---|---|---|---|---|
| `should` | blocking | blocking | advisory | deferrable | blocking |
| `prefer` | advisory | advisory | advisory | deferrable | advisory |

- **blocking** — spawns the next round. The round's **mode** (`full` / `confirm`) is computed from the fix, not from the finding — see [Verification Economy](verification-economy.md).
- **advisory** — reported to the developer, spawns no round.
- **deferrable** — routes to the todo register at once, spawns no round.

The unstated column equals the `core` column: silence never lowers a severity. `critical` differs from `core` only in round scope, below.

**A reviewer who holds the declaration wrong does not argue and does not lower it** — that is an evidence ↔ doc conflict for [Upstream Escalation](escalation.md).

## Recurrence limit — the `review-non-convergence` tripwire

Finding identity is `(normalized location + type)` — the dedup key of the audit `Finding`, reused unchanged.

- `must` and security findings block **without limit**. They must be fixed; no exit exists.
- Any other `blocking` finding gets at most two rounds. The same identity raised a second time unresolved fires the tripwire.
- `deferrable` spawns no round; it routes immediately.

Symmetric twin of the `verifier-rubber-stamp` tripwire in [Application Enforcement](application-enforcement.md) — same deterministic form, opposite failure direction. Both fire together when both hold; neither cancels the other.

Round count is bounded by (distinct `must` + security findings, each of which someone must fix) + 2.

## The `contested` route

A tripped or `deferrable` finding leaves the loop as a todo record of flavor `contested` — the third beside `candidate` and `queued`. It carries **both** readings, the reviewer's and the author's, so the next session does not re-derive the argument.

- Not speculative — the YAGNI-gate does not drop it.
- Does not surface when the originating task completes — that would reopen the argument right after the commit.
- Trigger = the next deliberate change to the owning file.

**Refuse the route** when: an override fired (`must` / security), or a `blocking` finding has not tripped the tripwire, or either position is empty. A finding stays blocking when the register cannot be written.

Name every routed record in the review report. Silent deferral is a report-ceiling violation ([Evidence Discipline](evidence-discipline.md)).

## Incremental round scope

Round state is written into the task file at the **close** of a round: `baseline` (tree state), `carry_over` (open blocking findings + their recurrence), `always_in_scope`.

Round 1, and any round with a full reason, reads everything. Full reasons: no reachable baseline · upstream spec/plan/rules changed · change class `architectural`.

Otherwise the round reads **delta ∪ carry-over ∪ always-in-scope**:

- **delta** — everything changed since the baseline, so a regression introduced into an already-reviewed file stays in scope.
- **carry-over** — previously open findings, whether or not their file changed, so moving code cannot close a finding.
- **always-in-scope** — every area declared `critical`, whether or not it changed.

An empty delta with a non-empty carry-over is a legitimate round. Every part empty means no round is needed.

The report names `scope_mode`, the round mode, and the reason for a full round. A narrowed scope with no named reason violates the no-silent-truncation rule.

Scope and mode are independent: scope says **what the round reads**, mode says **who performs it**. A `confirm` round reads the fix diff against the prescribed fix; `must` and security findings never reach it (discriminator D1).

## Degradation

A project that declares nothing runs every area as `unstated`: criticality and materiality become no-ops, and the recurrence limit alone carries convergence. The procedure uses documentation when it exists; it does not require it.
