# Evidence Discipline — Name What Fired, Not What Exists

Cross-cutting sub-procedure of [audit](../phases/audit.md) (`rules`/`skills` scopes) and the harvest step of [Experience Capture](experience-capture.md), which runs in every phase. Not a pipeline stage and no command. It supplies one vocabulary — **evidence state** — plus the consumers that speak it (`C_EVD_03_01`).

Advisory throughout — no consumer adds a blocking gate (`C_EVD_DEC_02`).

## Evidence state

Discrete values on a project-knowledge artifact (rule, skill), ordered by strength. Recorded as an **optional** `evidence:` key on the artifact's `_index.yaml` entry; a missing key reads as `unobserved`.

| State | Means | Admissible source |
|-------|-------|-------------------|
| `unobserved` | Observation boundary cannot decide | no source; runtime cannot observe the action; conditional phase never activated |
| `present` | Artifact exists in the catalogue | the write itself |
| `wired` | A relevant action can reach it | its name entered a Pre-Action Marker ([Application Enforcement](application-enforcement.md)) |
| `exercised` | It fired and a result was retained | `tripwire` by-product · `gate-result` · a skill's `check` applied |
| `outcome-supported` | A later comparable window confirmed the benefit | `improved` verdict from ledger reconciliation |

Record fields: `state` · `source` · `observed_at` · `ref` — `source` is one token of `catalogue-write` \| `pre-action-marker` \| `tripwire` \| `gate-result` \| `skill-check` \| `ledger-verdict`. A state other than `unobserved` **requires** non-empty `source` + `ref` (`SP_EVD_03_06`).

Never lower a state; `RecordEvidence` returns unwritten instead. Ageing belongs to `freshness` in [Procedural Skills](procedural-skills.md). `outcome-supported` is reachable only from reconciliation.

Standing rules:

- **Configured ≠ used.** Presence proves `present` and nothing more.
- **Count ≠ conclusion.** Numbers of rules, skills, or roles route inspection only — never a quality argument, never grounds for a finding.

Orthogonal to `promotion`/`freshness` in [Procedural Skills](procedural-skills.md); its "second convergent use" threshold reads as two recorded `exercised` observations.

## Consumer 1 — the coverage rung

Runs inside [Experience Capture](experience-capture.md)'s harvest, before a durable artifact is written. Rungs: **covered** (name the covering artifact) → **extend** (an existing artifact absorbs it) → **create** → **needs-evidence**.

Directive, not blocking: it recommends and requires a recorded reason; the caller may still create the artifact (`C_EVD_DEC_02`). `covered` without a named covering artifact is invalid.

Owner menu (`SP_EVD_03_04`): `rule` · `skill` · `drop` · `none` (covered) · `needs-evidence`. The first three mirror `SP_EXC_01_04.kind` plus its DROP branch; the last two are non-targets. **Narrowest durable owner wins; defaulting to `skill` is forbidden** — a `rule` carries a constraint, a `skill` a procedure, `drop` a narrow one-off.

## Consumer 2 — the skill `check` field

Additive **optional** field on a procedural skill, beside the [Procedural Skills](procedural-skills.md) set: **`check`** — the signal that proves the procedure fired (command, check result, observable artifact). It is the only source of `exercised` for a skill; without it that state stays out of reach (`C_EVD_DEC_07`).

**No invented evidence** (`SP_EVD_03_03`). A command, path, port, version, flag, or filename in `check` is admissible only when its source was **opened in this session** or **explicitly supplied by the user**. Otherwise record an explicit no-evidence marker, never an ecosystem default (`npm test`, `pytest`, `make build`). An L1 working-memory note counts when it records the opening with a reference; a bare mention of the name does not.

## Consumer 3 — the intervention ledger

`.dev_flow/evidence/ledger.yaml` (`SP_EVD_DEC_01`). One entry per harvest that created a durable artifact: `id` · `lesson` · `artifact` · `owner_kind` · `problem_class` · `trigger` · `state` · `verdicts` (append-only).

An entry with empty `verdicts` is **dormant** — it supports no claim of benefit. A ledger restored in a new session preserves continuity, never synthesizes a verdict. Opening an entry requires a named `trigger`; a second open entry for the same `artifact` + `problem_class` is rejected.

**Reconcile** in [audit](../phases/audit.md) `rules`/`skills` scope: every entry whose trigger is due gets one appended verdict — `improved` · `unchanged` · `regressed` · `still-unobserved`. `state` rises to `outcome-supported` only on `improved`, with no `regressed` in history, and only when the artifact is already `exercised`; a `regressed` in history keeps its blocker permanently. An artifact still `present` with no `exercised` observation ever recorded is reported as a **prune candidate** for [Procedural Skills](procedural-skills.md) curation.

Absent `.dev_flow/evidence/` the reconcile step is a no-op.

## Degradation

Where the runtime cannot observe an action, no source for `exercised` exists and the state stays `unobserved`; never simulate the observation ([Application Enforcement](application-enforcement.md) degrades the same way). A conditional phase that never activated is not an observation either: the artifacts it would have exercised stay `unobserved`, never read as clean.

Opening a ledger entry can fail (`EVD_NO_TRIGGER`, `EVD_DUPLICATE_ENTRY`) inside a harvest that is otherwise ungated: on either, skip the ledger entry and let the harvest write proceed — the write is never blocked by ledger bookkeeping.

## Where it is wired

| Touchpoint | What it carries |
|------------|-----------------|
| [Experience Capture](experience-capture.md) harvest | The coverage rung, owner menu, and ledger entry opening |
| [Procedural Skills](procedural-skills.md) · [skill phase](../phases/skill.md) | The `check` field and its no-invented-evidence rule |
| [Application Enforcement](application-enforcement.md) | Its tripwire by-product is the primary `exercised` source |
| [rule phase](../phases/rule.md) · [skill phase](../phases/skill.md) index format | The optional `evidence:` key |
| [audit](../phases/audit.md) `rules`/`skills` scope | Ledger reconciliation and prune candidates |

## Boundaries

- **No numeric scores.** States are discrete; a confidence or maturity number is rejected.
- **One owner for the vocabulary.** Defined here, cited everywhere; never redefined locally.
- **Scoped to the delta** (`C_EVD_DEC_03`).
- **Same-window validation proves repair state, never later effectiveness.**
