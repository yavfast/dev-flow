# Role Styles — a Working Manner Chosen by the Kind of Work

Cross-cutting sub-procedure of every phase in the [phase → style map](#phase--style-map). Not a pipeline stage, no command. A role says *what* an agent does and returns; a role style says *how* it works. The main agent sets the style when a phase starts, declares it, records it in the task file, and hands it to subagents in their brief.

## Contents

- [Boundaries](#boundaries) — what a style never is: an obligation, a text register, a role
- [Shipped styles](#shipped-styles) — the five cards: `exploratory`, `deliberate`, `formal`, `investigative`, `meticulous`
- [Phase → style map](#phase--style-map) — the default style of each phase or step; checking and making phases
- [Selecting the style](#selecting-the-style) — the steps at phase start: pin, manner named in the request, candidates, question
- [Declaring the style](#declaring-the-style) — the chat line and the task-file row
- [The developer's choice](#the-developers-choice) — what a named style pins and what the pin covers
- [Applying the style](#applying-the-style) — the card, what wins on a conflict, the result marks
- [Subagents](#subagents) — the `Role style:` line of the brief; the model for `exploratory` work
- [Project styles](#project-styles) — `.dev_flow/role_styles.md`: new styles and overlays

## Boundaries

- **A manner, never an obligation.** Role obligations, project rules, gates, developer checkpoints, the hard floor and the output contract hold under every style; a style governs only what the role leaves open.
- **Not a text register** — that is [Output Styles](output-styles.md). **Not a role** — no inputs, outputs or workflow ([Roles](roles.md)).
- **Never silent.** A style the developer did not see can be neither judged nor corrected.

## Shipped styles

Card fields: **Fits** — the map keys it is the default for · **Signals** — the kind of question or state of the work that makes it a candidate · **Manner** — how to work · **Result shows** — what in the output shows the style.

### exploratory

Ambitious and creative: research, prototypes.

- **Fits:** `research`
- **Signals:** an open solution space; a new idea with no existing design; a prototype.
- **Manner:**
  1. State the ambitious target first: the best result if nothing constrained you, then what stands in its way.
  2. Before you read any source, write down at least three ideas of your own; after reading, add what the sources gave and keep the two lists apart.
  3. One of your ideas must remove the problem instead of solving it; one must transfer a mechanism from a neighbouring domain, which you name.
  4. Test the most promising idea with the cheapest experiment the scope allows instead of reasoning about it at length.
  5. Keep a failed idea in the record with the reason it failed.
  6. Mark each idea of your own as a hypothesis with the check that would confirm it; a found fact carries its source.
  7. Recommend the boldest option that survives the checks, and say what the safe option gives up.
- **Result shows:** the ambitious target, stated before the options · ideas of the agent's own, written before the sources were read, listed apart from found material · one idea that removes the problem and one from a named other domain · each own idea marked a hypothesis with its confirming check.

### deliberate

Weighed design: options, trade-offs, a recorded choice.

- **Fits:** `concept`, `spec`, `plan`, `adopt`, `ask`
- **Signals:** a choice among known options; a trade-off to settle.
- **Manner:**
  1. Lay out the options before choosing; never present the first workable one as the only one.
  2. Name what each option costs and what it rules out.
  3. Choose the minimum that meets the stated goal; cut what has no stated consumer.
  4. Record the choice with its reason and the rejected options.
  5. Say what you did not check.
- **Result shows:** the options considered and why the chosen one won · nothing included without a stated consumer.

### formal

By the approved document: code to plan, propagation.

- **Fits:** `implement`, `propagate`, `onboard`, `fix:implement`
- **Signals:** an approved plan or specification exists for the work and is to be followed.
- **Manner:**
  1. The approved document is the instruction: do what it says, in its order, in its terms.
  2. Add nothing it does not ask for; file an improvement you notice instead of making it.
  3. Before the first edit, read the document for ambiguities and gaps and list them as questions. A question whose answers lead to different results stops that item until it is answered; never pick a reading silently.
  4. Reuse the existing patterns and names; prefer the predictable to the clever.
  5. Report each deviation from the document as a deviation.
- **Result shows:** the document's ambiguities listed as questions before the work, or "none" · every change traceable to a line of the document · deviations listed, or "none".

### investigative

Finding the cause: diagnosis of a defect or an unexplained behaviour.

- **Fits:** `fix:analysis`
- **Signals:** an unexplained behaviour; a cause to find.
- **Manner:**
  1. Observe and reproduce before explaining.
  2. Hold several competing hypotheses of the cause; write them down.
  3. Run the cheapest check that tells the hypotheses apart.
  4. Follow the evidence past the first plausible story to the root cause.
  5. Confirm that the fix removes the cause, not only the symptom.
- **Result shows:** the hypotheses considered and what ruled each out · the root cause with the evidence for it.

### meticulous

Exacting check: test, review, verify, audit.

- **Fits:** `test`, `review`, `verify`, `audit`, `fix:verify`
- **Signals:** a claim, a number or finished work to be checked.
- **Manner:**
  1. Assume a defect is present until the check shows otherwise.
  2. Check each claim against primary evidence — the file, the run — never against the author's description.
  3. Check every item in scope; say so when you sampled.
  4. Give the work no benefit of the doubt: what you could not establish is a finding or `unobserved`, never a pass.
  5. Report each finding with its exact location; do not fix or redesign.
- **Result shows:** each verdict tied to evidence that was read or run · what was not checked, named.

## Phase → style map

Keys are the values a card's `Fits` may name.

| Key | Phase or step | Default |
|-----|---------------|---------|
| `research` | research, any scope, `prototype` included | `exploratory` |
| `concept` | concept | `deliberate` |
| `spec` | specification | `deliberate` |
| `plan` | plan, the Refactoring Protocol's plan included | `deliberate` |
| `adopt` | external repo adoption | `deliberate` |
| `ask` | read-only Q&A | `deliberate` |
| `implement` | implement, a planned refactor included | `formal` |
| `propagate` | propagate | `formal` |
| `onboard` | onboard | `formal` |
| `fix:analysis` | fix — analysis and fix plan | `investigative` |
| `fix:implement` | fix — implementing the fix | `formal` |
| `fix:verify` | fix — verifying the fix | `meticulous` |
| `test` | test | `meticulous` |
| `review` | review | `meticulous` |
| `verify` | verify | `meticulous` |
| `audit` | audit, every scope | `meticulous` |

- `test`, `review`, `verify`, `audit` and the step `fix:verify` are the **checking phases**; the others are the **making phases**.
- Commands outside the map — `do` (routing), `status`, `checkpoint`, `resume`, `rule`, `skill`, `tool`, `todo`, cache management — have no style and declare nothing.
- `subtask` has the style of the phase it runs.

## Selecting the style

The main agent selects when a phase or step of the map starts; a subagent never does.

1. Phase outside the map → no style.
2. A pin for this phase (`<key>=<name>`), or a `task=<name>` pin on a making phase → that style, source `developer`.
3. The request names a manner for this phase ("creative", "креативно", "strictly by the plan") → [the developer's choice](#the-developers-choice), no question.
4. Default: the project card whose `Fits` names the key ([Project styles](#project-styles)), else the map.
5. Candidates: the default — or both project cards when two name the key — plus, **for a making phase only**, every style whose `Signals` the context shows (the kind of question and the state of the work, not the wording).
6. One candidate → take it, source `default`.
7. Several → **one question**: 2–3 options, the recommended first, one line each on what it changes in the result; the answer is a pin for this phase, source `developer`. Under `Autonomy: full` no question: the default — the first of the two project cards when two name the key — source `delegated`.

A named style in no catalogue → list the styles and ask; never invent a card. The question leaves only the pin, no Design Decision record.

Example — research over two named libraries: default `exploratory`, the context shows a choice among known options (`deliberate`) → *"Creative search beyond these two, or a weighed comparison of the two?"*

## Declaring the style

**Chat line** — `chat` register, the developer's language, the style name as it is:

```
Role style: <name> — <phase>, <source>[ — <reason, or the candidate not taken>]
```

Written when the style of the starting phase differs from the style of the task's previous styled phase, when it is the task's first styled phase, and when the developer changed it — not when the style repeats. Under `delegated` it names the candidate not taken.

**Task-file row** — in `## Current Work Item`, written by the main agent together with the `Pipeline phase` row (in the Write that creates a new task file, in one targeted Edit on an existing one); a subagent never writes it:

```
| **Role style** | `<name>` — <source>[; pinned: <pin>, <pin>…] |
```

- `<name>` — the style of the phase in the `Pipeline phase` row; for `fix`, of the step in progress, rewritten when the step changes; `n/a` (no source) for a phase with no style.
- `<source>` — `default`, `developer` or `delegated`.
- `<pin>` — `<key>=<name>` or `task=<name>`; pins stay when the phase changes.

A resumed session reads the row and declares nothing.

## The developer's choice

A style the developer names — in the request or at any later moment — takes effect at once: write the pin, declare with source `developer`. A later choice replaces the pin for the same key; a running subagent keeps the style of its brief.

| The developer names | Pin | Covers |
|---------------------|-----|--------|
| a style and a phase | `<key>=<name>` | that phase — a checking phase included |
| a style and `fix` as a whole | `fix:analysis=<name>`, `fix:implement=<name>` | its two making steps; `fix:verify` only when the developer names the verify step |
| a style "for the whole task" | `task=<name>` | every making phase; no checking phase |
| a style and nothing else | `<key>=<name>` for the current phase; at intake, for the first styled phase of the route | that phase |

A checking phase leaves its default only through a pin that names it: signals are never evaluated for it and a `task=` pin never reaches it.

## Applying the style

1. Read the card — the shipped section, plus the project section for an overlay or a project style.
2. Follow its Manner wherever the role leaves a choice; where a line conflicts with an obligation ([Boundaries](#boundaries)), the obligation holds and the rest of the style stays.
3. Before reporting, check the output against `Result shows`; state an absent mark with its reason.

## Subagents

A subagent whose brief carries `Read first:` ([SKILL.md → Subagents](../SKILL.md#subagents)) and whose phase is in the map gets one more line after it — an order, because a subagent given only the card's location does not open it:

```
Role style: <name> — before your first action read <card file> § <name>; work by it and name the style in your report
```

- `<card file>` — `references/role-styles.md` (relative to the skill root) for a shipped card, `.dev_flow/role_styles.md` (relative to the project root) for a project card, both separated by `; ` for an overlay.
- The subagent [applies the style](#applying-the-style); it never selects one and never asks the developer. No style line → the shipped default of its phase.
- A subagent that runs the whole `fix` gets the line of the step it starts with and works each later step by that step's default; the initiator rewrites the row when it returns.
- A subagent that only searches, reads, or counts gets no line.
- **Model.** `exploratory` work runs on the strongest model the runtime offers, within the profile's budget ([Delegation](delegation.md#fit-the-model-to-the-task--by-its-nature-not-its-name)): a strong model explores unprompted and the card adds the target, the origin of each idea and a bolder recommendation; a fast one follows the directives but runs no experiment. Other styles do not change the model.

## Project styles

`.dev_flow/role_styles.md` — optional, committed, read at the first selection of a session; edited on the developer's request as an ordinary project file. One h1, then one h2 per card named by the style (lowercase kebab-case):

```markdown
# Role Styles

## migration-careful
- **Title:** Data migration: reversible steps, counted rows.
- **Fits:** on request
- **Manner:**
  - Count the rows before and after each step.
  - Write the reverse step before running the forward one.
  - Stop at the first count that does not match.
- **Result shows:**
  - Row counts before and after each step.

## exploratory
- **Manner:**
  - Check the vendor's roadmap before proposing a mechanism the platform may ship.
```

- **New style** (a name that is not shipped): `Title`, `Fits` (map keys, or `on request`), `Manner` (3–7 lines), `Result shows` (1–4 lines); `Signals` optional.
- **Overlay** (a shipped name): `Manner` (1–7 lines) required; its Manner and Result shows lines follow the shipped ones; its `Fits`, when present, replaces the shipped one.
- A project card whose `Fits` names a key replaces the shipped default for that key.
- A card missing a required field, or naming an unknown key in `Fits`, is ignored; one chat line names the card and what is missing. File absent → shipped cards only.
