# Interview Mode — Surfacing Design Decisions

Shared sub-procedure for the **Concept**, **Specification**, **Plan**, and **Fix**
phases. Concept and spec link here for design forks; plan for its Technology Decisions;
fix for root-cause and fix-strategy forks (see [Interview Mode in Fix](../phases/fix.md#interview-mode-in-fix)).
This is **not** a standalone pipeline stage — it runs *inside*
those phases whenever a real fork in the road appears.

## Why this exists

Concept and specification documents get large. When an author silently picks one
of several viable approaches and buries that choice three levels deep in a
300-line document, the reviewer almost never catches it — there is too much text,
and the choice doesn't *look* like a choice anymore, it looks like a fact. The
decision then ossifies into code, and by the time anyone notices, reversing it is
expensive.

The author (you, the AI) is **not** the right party to make architectural choices
unilaterally. The developer holds the full project context — business constraints,
roadmap, team skills, things never written down — and carries responsibility for
the outcome. Your job at a fork is not to guess well; it is to make the fork
**visible** and bring the person who owns the decision into it, with enough
framing that they can decide in seconds rather than reverse-engineering your
reasoning later.

So: when authoring surfaces two or more materially different ways forward, **stop
and run an interview** instead of choosing silently. Cheap question now versus
expensive mistake later.

This complements the `handle-uncertainty` principle: silent guesses are the most
expensive class of agent mistake. Interview mode is where that principle gets
operationalised for design documents.

## When to trigger

Enter interview mode the moment you notice a **decision point** while authoring:

A decision point exists when **all** of these hold:
1. There are **two or more viable options** — each one could reasonably be the answer.
2. The choice is **material** — it changes architecture, a contract/data shape, the
   scope boundary, a dependency, a state model, or a technology choice (in plans).
3. The choice is **not cheaply reversible** — undoing it later means touching code,
   migrating data, or breaking a consumer.

If all three hold, the decision goes through the interview. If any fails, decide
it yourself and move on.

### Do NOT interview on

Over-asking is its own failure — it trains the developer to rubber-stamp, and the
real decisions drown in noise. Skip the interview and just decide (mention the
choice in passing) when:

- There is an **obvious default** the developer would not second-guess (naming a
  private field, ordering two independent validation checks).
- The choice is **trivially reversible** — a rename, an internal helper boundary.
- A **project rule, existing concept, or convention already settles it** — follow
  it; don't re-litigate. (Cite the rule.)
- The options are **functionally equivalent** with no downstream consequence.

When in doubt about whether something is material, lean toward surfacing it — but
bundle it efficiently (see batching below) so the cost stays low.

### Interview vs Spike

These are complementary, not the same:
- A **spike** *discovers* options when the solution space is unknown ("can library
  X even do Y?", "what approaches exist?"). It produces candidate options.
- An **interview** *chooses* among options that are already known. It produces a
  decision.

If you don't yet know the options, run a spike first — the
[research phase](../phases/research.md) (`/dev-flow research`) owns that
procedure. Once options exist, interview.

## The procedure

### 0. Gather full context first

An interview is only as good as the context behind it. A half-informed question
wastes the developer's attention and builds the decision on a wrong premise — so
before you surface any fork, make sure you actually understand it:

- **Read the relevant code** — what does the system do *today*? Existing patterns,
  the de-facto answer if one already exists, and the constraints the current
  implementation imposes on the options.
- **Read the relevant docs** — the concept/spec/plan in play (or, in a fix, the
  task file, fix report, and any affected spec/concept), applicable `.dev_flow/rules/`,
  and the **prior Design Decisions (`DEC_NN`)**. A fork may already be settled by an
  earlier decision record — cite it, don't re-open it.
- **Check what already exists** — the option you're about to propose, or the
  abstraction behind it, may already be built or settled by a rule. In the concept
  phase the [Reuse Check](../phases/concept.md) is exactly this search — lean on its
  result, don't re-run it mid-interview.

Anything you could have answered yourself from code or docs is **not** a question
for the developer — it is homework you owe *before* asking. Reserve the interview
for what genuinely cannot be answered from existing artifacts: forks whose right
answer depends on context that lives only in the developer's head — business
priorities, roadmap, team constraints that were never written down.

The boundary is clean: **code and docs tell you everything already known or
decided; the interview is for what isn't.**

### 1. Collect the decision points

While authoring, keep a running list of decision points rather than interrupting at
each one. Finish a coherent chunk of the document, then surface the forks together.
This respects the developer's attention and lets you order dependent decisions.

### 2. Order them

- **Dependent decisions sequentially:** if choice B only makes sense after choice A
  is settled (B's options depend on A's answer), resolve A first, then derive B's
  options from the answer. This is the "послідовно / sequential" part — you cannot
  meaningfully ask both at once.
- **Independent decisions may be batched:** unrelated forks can be asked together.

### 3. Frame each decision — always with a recommendation

Every decision is presented as a **question with 2–4 concrete options**, and you
**always include your own recommended answer**. The recommendation is not optional:
it does the developer's pre-reading for them and turns "analyse this from scratch"
into "confirm or correct my reasoning."

**Give every option a stable, unambiguous marker** — `A`, `B`, `C`, `D`. The marker
is the option's name for the rest of its life: the developer answers by it, you record
it under it in the Design Decisions section, and later documents and code reference it.
Keep the same letters across the question, the recorded record, and any follow-up — do
not renumber. This is what lets the developer reply tersely (`B`) **or compose an
answer out of the options** (`A, but with C's cap`, `B without the queue part`) instead
of re-typing the whole approach. The recommended option is `A` (put it first) and is
marked `(Recommended)`.

For each option give, briefly:
- **Marker + what it is** — `A — {approach}` in one line.
- **Consequence** — what it commits the system to (the downstream cost/benefit that
  makes this a real fork, not a coin flip).

For the recommendation, add **why** you'd choose it — the trade-off you're optimising for.

### 4. Ask — and how to ask

Whichever channel you use, the options carry their markers (§3) so the developer can
answer by marker or compose across them.

**When the `AskUserQuestion` tool is available** (Claude Code), use it — it is built
for exactly this: 2–4 options, recommended option first labelled `(Recommended)`,
and the developer can always choose "Other". **Prefix each option's label with its
marker** (`A — Fixed window (Recommended)`, `B — Sliding window`, …) and put the
consequence in the description. The marker matters even here: when the developer picks
"Other" to amend or combine options, they cite the markers (`like A but cap at 100, per
C`), and your reply is unambiguous. Batch up to 4 *independent* decisions into one call.

**When the tool is not available, the chat IS the dialog** — so reproduce it faithfully.
Present each decision as a clearly marked list the developer can answer against:

```
DEC_01 — {the fork in one sentence}
  A) {approach} — {consequence}   (Recommended: {why})
  B) {approach} — {consequence}
  C) {approach} — {consequence}
Reply with a marker (e.g. "A"), or compose/amend one ("A but with C's cap",
"B, and keep it open until load-test numbers exist").
```

Then **stop and wait** for the developer's answer. Do not proceed past an unresolved
material decision. If several independent decisions are open, list them together (each
with its own `DEC_NN` + lettered options) so the developer can answer them in one pass.
When more than one decision is on the table, the markers are scoped per decision, so the
developer pairs each `DEC_NN` with its letter — e.g. "DEC_01: A, DEC_02: B, DEC_03: C but
with A's default" — never a bare list of letters that can't be mapped back to a decision.

### 5. Resolve — two valid outcomes

Each decision ends in exactly one of two states:

- **Resolved (consensus)** — the developer picked an option (yours or their own).
  This is the default and the goal for production work. Record the chosen option
  **and its rationale**, and note the rejected alternatives so the choice isn't
  silently re-opened later.

- **Open (documented alternatives)** — the developer chooses to keep options open.
  Legitimate for **research / exploratory** work, or when a decision genuinely
  depends on information that doesn't exist yet. This is **not** a license to defer
  by default. An open decision is only acceptable if it records:
  - the candidate options with their trade-offs, **and**
  - a **resolution trigger** — the concrete event or date by which it must be
    closed (e.g. "decide before Plan phase", "resolve once load-test numbers exist",
    "open until C_SYNC ships").

  An open decision **without** a resolution trigger is just a hidden deferral (a
  "TBD" / "for now") — reject it (see the Banned Phrases in the
  [concept](../phases/concept.md) and [specification](../phases/specification.md)
  phases). The resolution trigger is what distinguishes a deliberate, bounded
  deferral from rot.

  **Open decisions have a lifecycle, not just a record.** When the missing
  information is *researchable* (numbers, prior art, library facts), the trigger
  should name the research — and the [research phase](../phases/research.md) is
  the sanctioned way to close it: the spike brings back findings, the developer
  decides, the record flips to `resolved`. The [audit phase](../phases/audit.md)
  sweeps all `DEC_NN` records for **expired triggers** (the named event/date has
  passed) and proposes the closing route — so an open decision cannot silently
  outlive its trigger.

### Who conducts the interview

The interview needs a live developer, so it is conducted by **whoever is talking to
the developer** — normally the orchestrator (the main session that received
`/dev-flow concept|spec|plan`).

A **delegated subagent** (a phase author, or a task-delegated
[subtask](../phases/subtask.md) executor) must **not** resolve a material fork by
guessing — and it must first exhaust the Step 0 context-gathering, since code and
prior `DEC_NN` records often close a fork outright. For whatever genuinely
remains, its channel is the **initiator**, not the developer:

1. **Escalate up the chain.** Present the fork to the initiator in the standard
   form — markered options + a recommended answer, independent questions batched.
   The initiator answers from the main-task picture it holds, or relays to the
   developer what only the developer can decide, and the answer comes back down.
2. **No live channel** (a one-shot run, batch work)? Checkpoint instead: record
   each decision point in Design Decisions as **open** — options, consequences,
   recommended answer with rationale, and a resolution trigger of "resolve with
   the initiator/developer before the gate" — and surface the list in the report.
   The initiator then answers and **continues the same subagent** (its assembled
   context is the asset), or runs the interview with the developer itself and
   flips each record to `resolved`.

Either way the gate (below) will not pass while material decisions are still open
without a trigger — so nothing slips through on the delegated path.

## Recording — the Design Decisions section

Every concept, spec, and plan template carries a **Design Decisions** section. Each
decision is an ADR-style record with a traceable ID so it survives review and can
be referenced from code and from later documents:

- Concept: `{#C_XXX_DEC_NN}`
- Specification: `{#SP_XXX_DEC_NN}`
- Plan: `{#PL_XXX_DEC_NN}`

> **Fix phase exception.** A code-only fix has no design document of its own. Record its
> decision in the **fix report** and the fix's **task file** (`.dev_flow/tasks/`) instead;
> an OPEN decision's resolution trigger must become a tracked item (backlog entry, rule,
> or Blocking Issue). If the fix changes a contract, the decision lives in the affected
> document's Design Decisions section as usual. See [fix phase](../phases/fix.md).

```markdown
### DEC_01 — {short question}  {#C_XXX_DEC_01}

> **Status:** resolved | open
> **Date:** YYYY-MM-DD

**Question:** {the fork, in one sentence}

**Options considered:**
| Option | Consequence |
|--------|-------------|
| A — {approach} | {what it commits us to} |
| B — {approach} | {what it commits us to} |

**Decision:** {chosen option by marker + name, e.g. `B — fork into two files`; or "OPEN — see resolution trigger". If the developer composed an answer, record the composite, e.g. `A with C's 100-item cap`.}
**Rationale:** {why this option / what trade-off it optimises for}
**Rejected because:** {one line per rejected option — why not}
**Resolution trigger:** {open decisions only: the event/date by which this must close}
```

Keep records terse — a decision the developer can re-grasp in fifteen seconds, not a
transcript. When a later change reverses a resolved decision, don't delete the
record: flip a new record's reasoning against it and cross-reference, so the history
of *why* stays intact.

## Worked example

While authoring `notifications.concept.md` you reach delivery semantics. You notice
a fork and surface it:

> **DEC_01 — Delivery guarantee for push notifications**
>
> A) **At-least-once (Recommended)** — retries until acked; consumers must dedupe.
>    Consequence: simple broker, but every consumer needs an idempotency key.
> B) **Exactly-once** — broker-side dedupe window. Consequence: no consumer dedupe,
>    but needs a stateful broker and bounded dedupe window — more infra.
> C) **At-most-once** — fire and forget. Consequence: cheapest, but silent drops on
>    transient failure — unacceptable if notifications are user-facing.
>
> *Recommended A: matches our existing event bus, and consumers already key events by
> ID — B's infra cost isn't justified yet. Reply with a marker, or amend one.*

The developer replies `A` (or, say, `A, but revisit B once we add a stateful broker`)
→ record it `resolved` with that rationale and "rejected B (infra cost), C (drops)".
The downstream consumer decision (DEC_02 — "where does dedupe live?") only has meaning
*after* DEC_01, so it is asked next, sequentially.
