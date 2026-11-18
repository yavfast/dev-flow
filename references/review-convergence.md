# Review Convergence — Terminate the Argument, Never the Defect

Cross-cutting sub-procedure of [review](../phases/review.md) (reads), [concept](../phases/concept.md) and [specification](../phases/specification.md) (declare), [todo](../phases/todo.md) (receives), and [audit](../phases/audit.md) (grooms). Not a pipeline stage and no command.

It bounds the `Review → fix → re-Review` loop with a **declared criticality** the reviewer reads instead of judging, a **recurrence limit** on a single finding, a **path rule** for a finding that got past a closed list, and an **incremental round scope**: a repeat round reads only the fixes.

Advisory throughout — the Review→Verify gate criteria are unchanged. What changes is which findings can reach `blocking`.

## Functional criticality — declared upstream, read downstream

Discrete, author-declared, optional. Never a number, never a score, never assigned by the reviewer.

| Value | Means |
|-------|-------|
| `peripheral` | Cosmetics, conveniences, rare paths with no effect on the result |
| `supporting` | Failure degrades convenience, not the result |
| `core` | The functionality the area exists for |
| `critical` | Failure breaks the application or violates security or data integrity |

Declaration levels, each optional:

- **application** — `Criticality:` in the concept header. The weight of this module inside the whole application. One per concept. A spec inherits its concept's line through `Concept:`.
- **module** — `Criticality:` under a concept mechanism section (`§3.x`) or a spec entity/contract section (`§01_xx`, `§02_xx`). The weight of this part inside its own module.

**Effective criticality** = the weakest among the **declared** levels. An undeclared level does not join the comparison — a silent section inherits its document's line. `unstated` only when no level declares anywhere.

Absence is valid, is never a documentation finding, and never lowers a severity.

## Finding materiality — computed, not judged

The reviewer assigns `severity` (`must` / `should` / `prefer`) as before, and does **not** assign materiality. Materiality is a pure function of `severity`, finding `type`, and effective criticality.

Overrides first, in order:

1. A security finding — the Security check, or a type `vuln:*` or `missing-authz` — with `exposure: external` while the stage written in the task file is not `production` → `deferrable`, routed as a todo of flavor `deferred` with the trigger "before the first external exposure", not the `contested` route ([Delivery Focus](delivery-focus.md#security-findings-and-the-stage)). `secret-exposure` never takes this override.
2. `type` in a security class (`vuln:*`, `secret-exposure`, `missing-authz`) → `blocking` (`secret-exposure` is only a secret in a place where secrets never live — [Proportionate Caution](proportionate-caution.md#secrets))
3. `severity: must` → `blocking`

Then the table (so `severity` is `should` or `prefer`):

| severity ↓ \ criticality → | `critical` | `core` | `supporting` | `peripheral` | unstated |
|---|---|---|---|---|---|
| `should` | blocking | blocking | advisory | deferrable | blocking |
| `prefer` | advisory | advisory | advisory | deferrable | advisory |

- **blocking** — spawns the next round. The round's **mode** (`full` / `confirm`) is computed from the fix, not from the finding — see [Verification Economy](verification-economy.md).
- **advisory** — reported to the developer, spawns no round.
- **deferrable** — routes to the todo register at once, spawns no round.

The unstated column equals the `core` column: silence never lowers a severity. `critical` reads like `core` here; it differs in [Test Economy](test-economy.md#logic-class), where a `critical` area counts as `complex` logic.

**A reviewer who holds the declaration wrong does not argue and does not lower it** — that is an evidence ↔ doc conflict for [Upstream Escalation](escalation.md).

## Recurrence limit — the `review-non-convergence` tripwire

Finding identity is `(normalized location + type)` — the dedup key of the audit `Finding`, reused unchanged.

- `must` and security findings block **without limit**. They must be fixed; no exit exists — except a security finding override 1 defers by the project stage.
- Any other `blocking` finding gets at most two rounds. The same identity raised a second time unresolved fires the tripwire.
- `deferrable` spawns no round; it routes immediately.

Symmetric twin of the `verifier-rubber-stamp` tripwire in [Application Enforcement](application-enforcement.md), opposite failure direction. Both fire together when both hold; neither cancels the other.

Round count is bounded by (distinct `must` + security findings, each of which someone must fix) + 2.

## List bypass — fix the path, not the list

A finding where a case got past a closed list — allow-list, no-confirm operation list, pattern, class signs — names that list in `bypassed_list` (name + `file:line` of its definition).

The fix the reviewer prescribes, and the only fix that closes the finding, **changes the path**:

- the case takes a route where the list is not the defence — a secret through an [input route](proportionate-caution.md#secret-input), a privileged command through the [project runner](proportionate-caution.md#privileged-commands); or
- the decision derives from the operation itself (`argv`, target, actual content), not from a declared field, and a case the list does not recognise falls into the stricter branch.

A fix whose only change to the decision is a member added to or removed from the list does not close the finding: it stays open under the same identity. A member beside a path change is fine — the list stays a signal. No path outside the list → [Upstream Escalation](escalation.md): the developer changes the design or declares the list the defence in a decision record of the owning document (the task file when there is none); only then does a fix that adds the member covering the case close the finding.

## The `contested` route

A tripped or `deferrable` finding — except one override 1 defers — leaves the loop as a todo record of flavor `contested`, beside `candidate` and `queued`. It carries **both** readings, the reviewer's and the author's, so the next session does not re-derive the argument.

- Not speculative — the YAGNI-gate does not drop it.
- Does not surface when the originating task completes — that would reopen the argument right after the commit.
- Trigger = the next deliberate change to the owning file.

**Refuse the route** when: an override fired (`must` / security), or a `blocking` finding has not tripped the tripwire, or either position is empty. A finding stays blocking when the register cannot be written.

Name every routed record in the review report. Silent deferral is a report-ceiling violation ([Evidence Discipline](evidence-discipline.md)).

## Incremental round scope

A repeat round reviews the fixes, not the whole task again.

**Baseline.** Take a snapshot of the working tree when the round's input is captured, before the reviewer starts. Round 1 and a full round read `git diff HEAD <snapshot>`, untracked files included (no commit yet: `git diff $(git hash-object -t tree /dev/null) <snapshot>`). At the **close** of the round, write that snapshot into the task file as `baseline`, with `carry_over` — the open blocking findings, each with its recurrence, prescribed fix and bypassed list. The snapshot is a git tree object; taking it changes no index, branch or working-tree file (it writes only unreachable objects into `.git`):

```
( d="$(mktemp -d)" || exit 1; trap 'rm -rf "$d"' EXIT
  cp "$(git rev-parse --git-path index)" "$d/index" 2>/dev/null
  GIT_INDEX_FILE="$d/index" git add -A && GIT_INDEX_FILE="$d/index" git write-tree )
```

It prints the tree hash; untracked files are in it, ignored files are not. The next round takes its own snapshot and reads the delta as `git diff <baseline> <snapshot>`.

Round 1 reads everything, and so does a round with no reachable baseline (no git, or the baseline object is gone) — its one full reason.

Otherwise the round reads **delta ∪ carry-over**:

- **delta** — everything changed since the baseline: the fixes and any other edit since, so a regression introduced into an already-reviewed file stays in scope. A changed spec, plan or rule is in the delta like any other file; one the snapshot does not hold (a git-ignored `docs/` or `.dev_flow/`) that changed since the baseline is named in the brief with its changed sections, and code it breaks is in scope like code a changed function breaks.
- **carry-over** — previously open findings, whether or not their file changed, so moving code cannot close a finding.

The reviewer reads unchanged code only as the context of the delta. It raises a finding only in the delta, or in unchanged code the delta breaks (a caller of a changed function). One exception: a `must` or security defect it sees outside the delta is still reported, and classified as usual.

An empty delta with a non-empty carry-over is a legitimate round. Every part empty means no round is needed.

The report names the round's scope — `delta` with its baseline, or `full` with its reason — and the round mode. A full round with no named reason, or a delta round with no recorded baseline, violates the no-silent-truncation rule.

Scope and mode are independent: scope says **what the round reads**, mode says **who performs it**. A `confirm` round reads the fix diff against the prescribed fix; `must` and security findings never reach it (discriminator D1).

## Degradation

A project that declares nothing runs every area as `unstated`: criticality and materiality become no-ops, and the recurrence limit alone carries convergence. The procedure uses documentation when it exists; it does not require it.
