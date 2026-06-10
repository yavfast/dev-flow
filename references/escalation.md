# Upstream Escalation — When the Document Is Wrong, Not the Code

Shared sub-procedure for the **Implement**, **Test**, **Review**, **Verify**, and **Fix**
phases. It is **not** a standalone pipeline stage — it runs *inside* a downstream phase
the moment evidence shows the defect lives in an upstream document (spec, plan, concept)
rather than in the code.

## Why this exists

The pipeline's working rule is "code must satisfy the spec, not the other way
around" — and for code bugs that is exactly right. But the rule has a blind spot:
sometimes the downstream phase is the first place where reality pushes back on the
*document*. A live test shows a spec'd limit is unworkable; implementation reveals
two defensible readings of a contract; a plan's technology decision turns out
infeasible. Without a sanctioned path, an agent in that position does one of two
bad things:

- **bends the code** to a spec it has evidence is wrong (correct per document,
  broken per reality), or
- **silently edits the spec** to match the code — exactly the drift the pipeline
  exists to prevent.

Escalation is the third path: stop, fix the owning document through its own
discipline, re-pass its gate, then resume. [Propagate](../phases/propagate.md)
flows an *already-made* decision top-down; escalation is the bottom-up route that
*creates* that decision mid-pipeline. The two compose: escalate up, then propagate
down.

## When to trigger

Escalate when evidence from a downstream phase points at an upstream artifact:

| Signal | Likely owner |
|--------|--------------|
| Two defensible readings of a contract — tests or implementation could justify either | Specification (ambiguity) |
| Observed reality contradicts a spec'd constraint, limit, or behavior (e.g. a live test shows the contract cannot hold) | Specification (wrong) |
| A technology decision does not work as assumed (library can't do it, platform constraint) | Plan |
| A domain assumption is falsified — the mechanism or boundary itself is wrong | Concept |
| A "fix" would change a published contract's meaning | Specification (this is a change, not a fix) |

Do **not** escalate for code bugs (code disagrees with a clear, correct spec — just
fix the code) or for personal disagreement with a settled, working design decision
(that is a change request: raise it with the developer, route through
[propagate](../phases/propagate.md) if accepted).

## The procedure

1. **Stop the affected work item.** Do not write code against a document you have
   evidence is wrong or ambiguous, and do not pick a reading silently — a silent
   guess here buries the defect one layer deeper.
2. **Locate the owning layer.** Walk up only as far as the evidence requires:
   spec first, plan if the issue is a technology decision, concept only if the
   domain assumption itself fails. Fix at the *lowest* wrong layer.
3. **Surface it.** This is developer-facing by default:
   - If the correction is a **material fork** (two or more viable ways to fix the
     document) — run [Interview Mode](interview-mode.md) with options and a
     recommendation, citing the downstream evidence.
   - If it is a **plain factual correction** (the spec says 1000, reality needs
     10 000 and nothing else changes) — propose the edit with the evidence and get
     confirmation.
   - If the missing piece is **researchable** (need numbers, prior art, library
     facts before the document can be fixed) — run [research](../phases/research.md)
     first, then return here.
4. **Update the owning document** through its own phase discipline: Changelog entry,
   `Updated:` date, Design Decisions record for the fork, versioning if the change
   is breaking (see [SKILL.md → Versioning](../SKILL.md#versioning)).
5. **Re-pass the affected gate** (Concept→Spec, Spec→Plan, or Plan→Code) for the
   changed document, and apply the
   [propagation matrix](../phases/propagate.md#change-type-propagation-matrix) to
   cascade the change downstream.
6. **Resume the downstream phase** where it stopped, re-running whatever the
   document change invalidated (the standard fix cycle: Test → Review → Verify on
   the affected area).

## Recording

Escalations are decisions — they leave the same trail as any other:

- The document fix lands with a **Changelog** entry naming the downstream evidence
  ("live test showed X", "implementation surfaced two readings of Y").
- A fork goes into the document's **Design Decisions** (`DEC_NN`) section as usual.
- The task file gets an Activity entry: what was escalated, to which document, and
  the outcome — so the pause is visible in the session record.

## Anti-patterns

- Bending code to satisfy a spec that evidence says is wrong ("the document is
  always right" — it is authoritative, not infallible)
- Silently editing the spec to match the code ("drift with extra steps")
- Picking one of two defensible contract readings without surfacing the ambiguity
- Escalating straight to the concept when the spec is the wrong layer (over-escalation)
- Treating a settled-but-disliked design decision as an "error" to escalate
