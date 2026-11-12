# Project Tools — Contracts, Trust, Adaptation

A **project tool** is a small executable that agents write for agents and keep in `.dev_flow/tools/` of one project: a check that confirms a claim, a converter, a generator of a derived file, a client of a service the project works with. A tool is reused when its documentation is a contract that can be checked faster than the code can be read: a **header contract** that tells an agent seeing the tool for the first time what it is for, how to call it and how to read its answer; **acceptance samples** that pin its behaviour; a **binding**, written by [audit](../phases/audit.md), that proves the code on disk is the code that passed verification and an independent review. With these, a later agent runs the tool without reading it, adapts it instead of forking it, or writes it anew from its **card**. This file owns the tool contract and lifecycle; [Experience Capture](experience-capture.md) owns the harvest trigger, [Evidence Discipline](evidence-discipline.md) owns the evidence vocabulary. Read it before writing, running, changing or revising a tool.

## Contents

- [Catalogue](#catalogue) — the files under `.dev_flow/tools/` and the three regimes
- [Header contract](#header-contract) — the block in the tool header: markers, fields, the exit-code contract, an example
- [Acceptance samples](#acceptance-samples) — the samples manifest; real material for good/bad, constructed only for error and review findings
- [Card, binding, router, usage file](#card-binding-router-usage-file) — the durable copy, the verification record, the derived index, the run-time data
- [Lifecycle](#lifecycle) — states, transitions, and who may move them
- [Promote](#promote) — keeping a script from the session workspace; running a sample
- [Verification in audit](#verification-in-audit) — what audit runs, the independent review, the binding
- [Trust check](#trust-check) — `run`, `read-first` or `unfit`, without writing anything
- [Run](#run) — calling a tool and reading its answer by the contract
- [Adapt](#adapt) — `fix`, `extend`, `port`, `repurpose`, `retire`
- [Regenerate](#regenerate) — a new realization from the card alone
- [Use accounting](#use-accounting) — evidence, run statistics, recent failures, bypasses
- [Legacy tools and the privileged runner](#legacy-tools-and-the-privileged-runner) — what stays outside this lifecycle
- [Edge cases](#edge-cases) — situations with a fixed answer

## Catalogue

```
.dev_flow/tools/
├── _index.yaml                 # tool router — derived from cards (+ legacy and runner entries)
├── <name>.<ext>                # realization; its header carries the contract block
├── samples/<name>/
│   ├── samples.yaml            # samples manifest
│   └── …                       # sample material referenced by the manifest
├── cards/<name>.yaml           # tool card
├── usage/<name>.yaml           # usage file — evidence state, run statistics, recent failures
└── prev/<name>.<ext>           # last bound realization, kept only while an adaptation awaits audit
```

Each router entry has a `regime`: `contract` — a tool with a header contract, governed by this file; `harvested` — a tool kept before this file, governed as before (see [Legacy tools](#legacy-tools-and-the-privileged-runner)); `developer-word` — the privileged runner, owned by [Proportionate Caution](proportionate-caution.md#privileged-commands).

A one-off script is not a project tool: it lives and disappears in the session workspace `/tmp/{project-slug}/tools/` ([Resource Cache](cache.md)). The project's own scripts and build tooling are project code and go through the pipeline.

What a tool may do — its access, and which of its actions need the developer's confirmation — is set by the project rules and the agent's system instructions, as for any other action of the agent. This file defines the process; the declared effects in the contract describe what the tool does, so that an agent can choose it and a reviewer can check the code against it.

## Header contract

The contract is a delimited block of YAML inside comment lines at the top of the realization. It is written with the code, every change of the tool starts with it, and the card is a mechanical copy of it.

- The **begin line** contains `dev-flow:tool-contract v1` and nothing after it but whitespace; everything before the marker on that line is the **prefix** — the comment leader with its trailing space (`# `, `// `, `-- `, `REM `).
- The begin line lies within the first 40 lines (a shebang or encoding line may precede it).
- Every line up to the **end line** (prefix + `dev-flow:end`) starts with the prefix; a blank contract line may carry the prefix without its trailing space.
- Removing the prefix from every line between begin and end yields one YAML mapping — the **contract**. One block per file.

| Field | Type | Req. | Constraints | Meaning |
|-------|------|------|-------------|---------|
| `name` | string | yes | `^[a-z0-9]+(-[a-z0-9]+)*$`; the file name without extension; unique in the router | Identity |
| `purpose` | string | yes | one line, ≤ 160 characters | What the tool does; becomes the router's `when_to_use` |
| `kind` | `check` \| `operation` | yes | — | Selects the exit-code contract |
| `range_of_use` | list of `{case, description}` | yes | ≥ 1; `case` matches `^[a-z0-9-]+$`, unique | The cases the tool is for; each pinned by ≥ 1 sample |
| `not_for` | list of string | yes | may be empty | Needs the tool must not be used for |
| `applies_to` | `{paths: [glob], phases: [phase]}` | no | absent = always-on; `phases` are dev-flow phase names (`implement`, `fix`, `testing`, `verify`, `review`, `research`, `propagate`, `audit`, …) | Selector the knowledge gate matches, as for a rule |
| `limits` | list of string | yes | may be empty | What the tool does not see or cannot decide |
| `inputs` | list of Input | yes | may be empty; names unique | Named typed inputs |
| `invocation` | list of string | yes | argv array, never a shell string; the realization path is relative to the project root; placeholders `{<input>}` only for declared inputs | How to call it |
| `output` | `{format: text\|lines\|json\|files, shape, size_class: small\|large}` | yes | `small` = read in context; `large` = may exceed ~50 lines, always redirected | How to read the answer |
| `behavior` | string | yes | precise enough to write a new realization from it and the samples | What the tool computes |
| `effects` | `{reads, writes, network, env, replay_input?}` | yes | `reads`: globs relative to the project root or a stated description, may be empty; `writes`: globs relative to the project root, empty = writes nothing; `network`: hosts or services, empty = none; `env`: variable names; `replay_input`: required when `network` is non-empty — a declared `path` input through which a sample supplies a recorded response or dry-run material instead of the network | What the tool touches |
| `environment` | `{platforms: [linux\|darwin\|windows\|any], requires: [{program, version_cmd, min?}]}` | yes | `platforms` ≥ 1; `version_cmd` is an argv that prints the version | Where it runs |
| `provenance` | `{task, promoted}` | yes | `task` = task file path relative to `.dev_flow/tasks/`; `promoted` = ISO date | Where it was kept |

**Input:** `name` (`^[a-z][a-z0-9_]*$`) · `type` (`string` \| `path` \| `path-list` \| `int` \| `bool` \| `enum`) · `required` · `default` (only when not required) · `values` (for `enum`, ≥ 2) · `description` (one line). Every required input appears as a placeholder in `invocation`; every placeholder names a declared input; an omitted optional input without a default removes the whole `invocation` element that holds its placeholder — so a flag and its value go into one element (`--exclude={exclude}`), not two.

**Exit-code contract**, fixed by `kind`:

| kind | `0` | `1` | `2` | other non-zero |
|------|-----|-----|-----|----------------|
| `check` | clean | findings reported | cannot decide | failure of the tool |
| `operation` | done | — | cannot decide; nothing written | failure of the tool |

A "cannot decide" exit `2` comes with a stderr line `cannot-decide: <reason>`; an exit `2` without that line is a failure of the tool — interpreters and argument parsers also exit `2`, so a missing file or a usage error can never pass as a deliberate refusal. A `check` writes nothing: its `effects.writes` is empty.

Example (a Python check; the same block works behind `//`, `--` or any other comment leader):

```
#!/usr/bin/env python3
# dev-flow:tool-contract v1
# name: dangling-anchors
# purpose: Report links to {#ID} anchors that the target document does not define
# kind: check
# range_of_use:
#   - {case: md-local, description: "links of the form (#ID) inside one file"}
#   - {case: md-cross, description: "links of the form (./file.md#ID) between files"}
# not_for: ["HTML anchors", "links to external URLs"]
# applies_to: {paths: ["docs/**"], phases: [review, propagate]}
# limits: ["anchors generated by a renderer from headings are not seen"]
# inputs:
#   - {name: root, type: path, required: true, description: "directory to scan"}
# invocation: [python3, .dev_flow/tools/dangling-anchors.py, "{root}"]
# output: {format: lines, shape: "<file>:<line>: <link>", size_class: small}
# behavior: >
#   Collect every {#ID} defined in *.md under root; report each (#ID) or
#   (./f.md#ID) link whose ID is not defined in the target file. Exit 2 when
#   root holds no *.md file, printing "cannot-decide: no markdown under root" to stderr.
# effects: {reads: ["{root}/**/*.md"], writes: [], network: [], env: []}
# environment: {platforms: [any], requires: [{program: python3, version_cmd: [python3, --version]}]}
# provenance: {task: task_C_PTL.md, promoted: 2026-11-06}
# dev-flow:end
```

## Acceptance samples

`samples/<name>/samples.yaml` holds one key, `samples`, a list; the material the samples use lies beside it, so the samples hash covers expectations and material together.

| Field | Req. | Constraints | Meaning |
|-------|------|-------------|---------|
| `id` | yes | `^[a-z0-9-]+$`, unique | Identity |
| `case` | yes | a `range_of_use[].case` | The case it pins |
| `kind` | yes | `good` \| `bad` \| `error` | `good` — clean / done; `bad` — the expected report; `error` — input the tool cannot decide on, which it must say instead of reporting clean |
| `inputs` | yes | every required input; `path` / `path-list` values relative to `samples/<name>/` | The call |
| `expect` | yes | `{exit, output_contains: [], writes: []}`; `exit` 0 for `good`, 1 for `bad`, 2 for `error`; `writes` relative to the work copy — in a sample run the tool writes only inside the work copy, because its inputs point there; `effects.writes` is checked on real runs ([Run](#run)) | Expected result |
| `source` | yes | names the real material it was reduced from; `constructed — <why>` only for an `error` sample or a sample pinning a review finding | Provenance |

- A `check` has ≥ 1 `good`, ≥ 1 `bad`, ≥ 1 `error`; an `operation` has ≥ 1 `good`, ≥ 1 `error`, no `bad`.
- Every `range_of_use` case is the `case` of ≥ 1 sample. A case without a sample lies outside the range of use.
- `good` and `bad` samples are real project material reduced to the minimal case — never invented from the author's model of the input. An `error` sample, and a sample that pins a review finding, may be constructed when no real material exists; it says so in `source`.
- A tool with `effects.network` reaches no network in any sample: every sample sets the input named by `effects.replay_input`; what a replay cannot show goes into `limits`.

## Card, binding, router, usage file

**Card** — `cards/<name>.yaml`, the durable copy that outlives the realization; never edited by hand.

| Field | Meaning |
|-------|---------|
| `card_version` | `1` |
| `name` | equals `contract.name` |
| `state` | `unverified` \| `active` \| `failing` \| `retired` ([Lifecycle](#lifecycle)) |
| `contract` | copy of the header contract at the last copy — the parsed YAML mapping |
| `samples` | copy of the `samples` list of `samples.yaml` at the last copy — parsed |
| `binding` | Binding or `null`; written only by audit |
| `failure` | `{date, reason, environment, ref, realization_sha256, samples_sha256}` while `failing` — the hashes of the failing realization and samples — else `null` |
| `retired` | `{date, reason, replaced_by, review}` once `retired`, else `null` |

`state: active` always has a binding. An `unverified` card keeps its previous binding, if any, until audit replaces it — the diff review needs it. "Header ≠ card" and "manifest ≠ card" compare the parsed YAML, never the text, so reformatting alone is not a change.

**Binding** — the verification record: `realization` (path relative to the project root) · `realization_sha256` · `samples_sha256` · `environments` (≥ 1) · `verified_at` (ISO date) · `commit` (project HEAD, `null` outside git — the code state the tool was verified against) · `samples_passed` (every sample id) · `review` (the Review Verdict, `pass`).

- **Hash** — sha256, lowercase hex (`sha256sum` or `shasum -a 256`).
- **Samples hash** — sha256 of the text built from every regular file under `samples/<name>/`: one line per file, `<sha256>  <path relative to samples/<name>/, '/'-separated, no './'>`, sorted bytewise by path, each line ending in `\n`.
- **Environment** — `platform` · `arch` (as `uname -m` prints it; `platform.machine()` elsewhere) · `programs` (each `requires[].program` → the version its `version_cmd` prints).
- **Environment match** — the current environment matches a recorded one when `platform` and `arch` are equal and every program has the same major version (the first run of digits in its version output — `3` in `Python 3.14.7`).

**Router** — `_index.yaml`, one key `tools`, rebuilt from the cards: for each card, `name` · `when_to_use` (= `purpose`) · `applies_to` · `run` (= `invocation`) · `regime: contract` · `state` · `card: cards/<name>.yaml`. Entries of the `harvested` and `developer-word` regimes without a card are carried over untouched; their shape is `name` · `when_to_use` · `applies_to` (absent = always-on) · `run` · `regime` · `self_test: {good, bad, expect}` (`null` for the runner) · `provenance`. A legacy entry whose tool has received a card is replaced by the card's entry, and a legacy `evidence:` field, if any, moves to the tool's usage file. Entries are sorted by name. A `retired` card keeps its router entry (with `state: retired`, no realization), so the name stays reserved and the gate skips it. **Every write of a card is followed by a router rebuild**, so the router's `state` always equals the card's. The router holds no run-time data, so it can always be rebuilt.

**Usage file** — `usage/<name>.yaml`, the run-time data of one tool. Outside trust: never hashed, never bound, never read by the trust check. Created on the first use event; kept with the card on retire. Read by audit, by `fix`, and by `/dev-flow tool`.

| Field | Meaning |
|-------|---------|
| `name` | the tool |
| `evidence` | the evidence record of [Evidence Discipline](evidence-discipline.md) — `state` · `source` · `observed_at` · `ref`; missing = `unobserved` |
| `runs` | `{total, by_verdict: {<run verdict>: count}, last_run}` |
| `recent_failures` | the newest 5, newest first: `{date, task, verdict, exit, environment, inputs, excerpt}` — `verdict` ∈ `cannot-decide` \| `tool-failure` \| `undeclared-write` \| `implausible`; `exit` is `null` when the run did not start; a value the agent knows to be a secret is written `<redacted>`; `excerpt` is ≤ 20 lines of the output or error tail, verbatim — never a session-workspace path, which does not outlive the session |

Re-read the usage file immediately before writing it; append a failure, then trim to 5. A lost counter increment between two sessions is tolerated.

## Lifecycle

| From | To | When | Side effects |
|------|----|------|--------------|
| — | `unverified` | [Promote](#promote) succeeds | realization, samples, card, router entry, ledger entry |
| `unverified` | `active` | audit: samples pass in a clean environment and the independent review passes | binding written; card re-copied; `prev/` deleted |
| `unverified` | `failing` | audit: contract invalid, a sample fails, or the review fails | `failure` written |
| `active` | `unverified` | [Adapt](#adapt) `fix` / `extend` / `port`, [Regenerate](#regenerate), or audit finds the header ≠ card or a hash ≠ binding | `prev/` keeps the last bound realization (Adapt) |
| `active` | `active` | audit: the current environment is new and the samples pass in it | environment appended; no review — the code is unchanged |
| `active` | `failing` | audit: a forced verification (`/dev-flow tool verify <name>`) fails; or a [run](#run) changes the tree outside `effects.writes` | `failure` written |
| `failing` | `unverified` | Adapt or Regenerate | `failure` cleared at the next audit pass |
| any but `retired` | `retired` | [retire](#adapt) confirmed by the independent review | realization and `prev/` deleted; card, samples and usage file kept |

`retired` is terminal; a retired name is never reused.

**Write discipline.**

- **Trust is raised only by audit.** Only the audit tools step writes a binding, sets `active`, or appends an environment.
- **Anyone in the lifecycle may lower trust.** Promote, Adapt and Regenerate set `unverified`; a run with an undeclared write sets `failing`.
- The card's `contract` and `samples` are only ever copies of the header and the manifest.
- Only Adapt (`fix`, `extend`, `retire`) and Regenerate change or delete a realization; audit never does.
- `prev/<name>.<ext>` is written only by Adapt `fix` / `extend`, and deleted only by audit on binding, by `retire`, or by Regenerate (including `port`), whose review is always `full`.
- `.dev_flow/` may be ignored by git: every Promote, Adapt, Regenerate and retire names the tool in the phase report and the commit message.

## Promote

Every tool starts as a script in the session workspace, written for the current task. A one-off stays there. **Promote** it when you judge it worth reusing; you **must consider** promoting it when the structural trigger of [Experience Capture](experience-capture.md#applying-experience--auto-apply-through-structural-gates) fires — the same check or operation hand-rolled in at least two tasks, or hand-rolled wrong (name those records).

1. **Search first** ([Code Reuse](code-reuse.md)). The need lies in a router entry's `range_of_use` → use that tool (`PTL_COVERED`), nothing is written. A neighbouring case that changes for the same reason → [Adapt](#adapt) `extend` that tool (`PTL_NEAR_MATCH`).
2. **Generalize** to the stated range of use and no further: hard-coded paths and values become inputs, the cases met in the session become samples.
3. **Write the header contract** — valid per [Header contract](#header-contract); `provenance.task` = this task (`PTL_CONTRACT_INVALID` otherwise). The name must be new, retired names included (`PTL_NAME_TAKEN`).
4. **Write the samples** — `good` and `bad` from real runs; an `error` sample may be constructed (`source: constructed — <why>`) when no real material exists — valid per [Acceptance samples](#acceptance-samples) (`PTL_SAMPLES_INVALID` otherwise).
5. **Run every sample yourself** ([Running a sample](#running-a-sample)) with the draft in the session workspace: the invocation's realization path is replaced by the draft's path, nothing else changes. A failure → fix the script (`PTL_SAMPLES_FAIL`). This run is part of writing; it gives no trust to other sessions.
6. **Write** the realization, `samples/<name>/`, the card (`state: unverified`, `binding: null`); rebuild the router; open a ledger entry (`id: tool-<name>` · `lesson`: the `purpose` · `artifact`: the realization path · `owner_kind: tool` · `problem_class: tool:<name>` · `trigger`: the ISO date 30 days after promotion, read by the first `/dev-flow audit tools` on or after it · `state: present`) — a ledger failure skips the entry, never the write.
7. **Name** the tool in the phase report and the commit message, and add a row to the task file's Relevant Context (`Tool` · `.dev_flow/tools/<name>` · `unverified — awaiting audit`); the task-completion report offers `/dev-flow audit tools`.

Any failure writes nothing to `.dev_flow/tools/` and records the reason in the task file. The authoring session may use the tool before audit — it knows the code; other sessions read it first ([Trust check](#trust-check)).

### Running a sample

Shared by the author's own run and by audit:

1. Copy `samples/<name>/` to a fresh work copy under `/tmp/{project-slug}/tools/`.
2. Substitute the sample's inputs into `invocation`; sample-relative paths resolve inside the work copy.
3. Snapshot the project tree and the work copy: every file except under `.git/`, ignored files included, as path · size · mtime · type. (`git status --porcelain` alone misses writes to ignored paths, `.dev_flow/` among them, and a second change to an already dirty file.)
4. Run from the project root with a process environment holding only `PATH`, `HOME`, `LC_ALL=C.UTF-8` (`C` where that locale is missing), on Windows also `SystemRoot` and `TEMP`, and the variables in `effects.env` — the literal invocation, in that clean environment.
5. Pass ⇔ the exit equals `expect.exit`, every `output_contains` string appears in stdout or stderr, the work copy changed exactly in `expect.writes`, and the project tree is unchanged.
6. Remove the work copy.

## Verification in audit

Trust is established in one place: the tools step of [audit](../phases/audit.md) (Step 7g, scope `all` or `tools`), also run by `/dev-flow tool verify <names>`. A session that needs an unverified tool mid-task reads it first or offers `/dev-flow audit tools` to the developer; it does not verify the tool itself.

For each `contract`-regime tool not `retired`, compute what is **due**:

| Condition | Due |
|-----------|-----|
| realization missing (and not `retired`) | report row; [Regenerate](#regenerate) candidate; nothing else runs |
| `unverified` | full verification |
| `failing` and the realization or samples hash differs from the one recorded in `failure` | full verification |
| `active` and the realization or samples hash ≠ binding, or the header ≠ `card.contract`, or the manifest ≠ `card.samples` | full verification (treated as an adaptation) — first write `state: unverified` (not under `--dry-run`) |
| `active` and the current environment matches none in the binding | new-environment verification |
| `active`, named in `/dev-flow tool verify <name>` | full verification of that tool (the developer's explicit request is the entry condition) |
| otherwise | nothing — no sample runs, no review ([Verification Economy](verification-economy.md)) |

**Full verification:**

1. Validate the header contract and the manifest; invalid → `failing`.
2. Run every sample ([Running a sample](#running-a-sample)); any failure → `failing`.
3. **Independent review** — `diff` scope when `prev/<name>.<ext>` exists and its sha256 equals the binding's `realization_sha256`, else `full`. A hash mismatch with no `prev/` base is reviewed `full`; passing samples alone never re-bind a tool, because samples cannot show an undeclared effect.
4. Pass → re-copy the card from the header and manifest, write the binding (keep the recorded environments when the realization is unchanged, else the current one), `state: active`, clear `failure`, delete `prev/<name>.<ext>`. Fail → `state: failing`, `failure` with the reason, a report row.

**New-environment verification** — run every sample in this environment: pass → append it to `binding.environments`; fail → report it, leave the state and the binding as they are (that environment stays `read-first`; the row names a `port` candidate). No review: the code is unchanged.

**Independent review.** A clean-context subagent in the reviewer role's tool-review mode ([reviewer role](../roles/reviewer.ai.md)) — never the session that wrote the realization, never the auditor's own context. It receives only the realization, its header contract, the samples manifest and material, and for `diff` the `prev/` realization; framing is adversarial. Four checks:

| Check | Finding when |
|-------|------------|
| `effects` | any read, write, network call, environment read or child process the contract does not declare |
| `contract` | an agent seeing the tool for the first time could not choose it, call it, or read its answer from the header alone |
| `range` | a `range_of_use` case without a sample, or code serving cases outside the range |
| `behavior` | the code departs from `behavior` on inputs the samples do not pin |

**Severity.** `blocking` — any `effects` finding; a `contract` an agent could not use; a `range_of_use` case without a sample; a departure from `behavior` on input inside the range of use. `limit` — code that serves cases outside the range; behaviour on input outside the range of use, or that no sample can pin (special files, permissions, encodings). Trust covers the declared range of use; robustness outside it is documented, not required.

Review Verdict: `{date, scope: full|diff, verdict: pass|fail, findings: [{check, severity, detail}]}`; `pass` ⇔ no `blocking` finding. On pass it is stored as `binding.review`, and its `limit` findings are read with `contract.limits` as known limits; the audit report proposes them for the next `fix`, which moves each into the header's `limits` or pins it by a sample. On fail the verdict goes into `failure.ref`. Repeated fail rounds follow the recurrence limit of [Review Convergence](review-convergence.md#recurrence-limit--the-review-non-convergence-tripwire) — the same blocking finding raised a second time unresolved fires the tripwire; past it the tool stays `failing` and the developer decides. A runtime that cannot spawn a subagent → the tool keeps its state and the row is `unobserved` (`PTL_REVIEW_UNAVAILABLE`).

**Also in the step:** router ↔ cards ↔ realizations ↔ samples directories — a router entry without its card or (unless `retired`) its realization, or a card, realization or samples directory without a router entry, is a report row (`PTL_ORPHAN`), never deleted. Ledger entries with `owner_kind: tool` whose trigger is due are reconciled here ([Evidence Discipline](evidence-discipline.md#consumer-3--the-intervention-ledger)). `harvested` tools: run their self-test as before and propose a contract through Adapt (`PTL_LEGACY`). `developer-word` entries: not run, not reviewed (`PTL_RUNNER`). Report retire candidates (no `exercised` for ~30 days — neither in the usage file nor as an `exercised` observation of the tool's ledger entry), recent failures per tool, and bypass reasons per tool. Under `--dry-run` nothing runs and nothing is written: each due action is listed as `unobserved` (`PTL_DRY_RUN`). After the per-tool rows, rebuild the router from the cards.

Audit never edits, moves, or deletes a realization or a sample.

## Trust check

Before the first use of a tool in a session decide, without writing anything, whether it can run without its code being read. The verdict lives in session working memory ([Resource Cache](cache.md#session-working-memory-l1)) keyed by the realization hash, the samples hash and the environment; it holds until one of the three changes.

| Step | Answer |
|------|--------|
| Router entry is not `contract` regime | `read-first` (`legacy`); for `developer-word`, [Proportionate Caution](proportionate-caution.md#privileged-commands) decides |
| Card `state: retired` | `unfit` (`retired`) |
| Realization missing | `unfit` (`missing-realization`) — Regenerate it, or hand-roll |
| The need matches a `not_for` item | `unfit` (`not-for`) |
| The need fits no `range_of_use` case | `unfit` (`out-of-range`) |
| A required input is not available | `unfit` (`missing-input`) |
| `unverified` / `failing` | `read-first` (`state-unverified` / `state-failing`) |
| Realization or samples hash ≠ binding | `read-first` (`hash-mismatch`) |
| Current environment matches none in the binding | `read-first` (`new-environment`) |
| Otherwise | `run` (`ok`) |

`run` — call the tool without reading its code. `read-first` — read the realization, then use it as before this file existed; the discrepancy waits for the next audit. `unfit` — do not use the tool for this need. The trust check verifies nothing — it only reads the binding audit wrote; whether the tool's action is allowed is decided by the project rules and the agent's system instructions.

## Run

1. Build argv from `invocation` and the input values — an argument array, never a shell string.
2. A tool with `effects.writes` → snapshot the project tree first.
3. `output.size_class: large` → redirect the output to `/tmp/{project-slug}/tools/<name>_YYYYMMDD_HHMMSS.out` and read it by path.
4. Read the exit by the [exit-code contract](#header-contract). The run verdict is `clean` \| `findings` \| `done` \| `cannot-decide` \| `tool-failure`, or `undeclared-write` when step 5 fails.
5. A tool with `effects.writes` → the paths changed by the run must all match `effects.writes`; anything else → the run failed, set the card `state: failing` with `failure`, report it to the developer (`PTL_UNDECLARED_WRITE`).
6. Record the run ([Use accounting](#use-accounting)).

**Reading the answer.**

- Exit `2` with a `cannot-decide:` stderr line is "cannot decide" (`PTL_CANNOT_DECIDE`) — never read as clean; exit `2` without that line is `tool-failure`. Record a `broken` bypass and do the check another way, or, when the refusal was right, add the input as an `error` sample through `fix`.
- An empty or clean result on material whose content you have already seen is a claim about the tool, not about the material (`PTL_IMPLAUSIBLE`): stop, confirm another way; if the tool was wrong, record an `implausible` failure and `fix` it with that material as a sample.
- An exit outside the contract → `tool-failure` (`PTL_TOOL_FAILURE`): record it and a `broken` bypass; hand-roll the check if the work needs it.
- The program a tool needs is missing → the run does not start: record an `unavailable` bypass; nothing is installed — a new dependency is a plan decision; when the need recurs, `port`.

## Adapt

Every class except `retire` starts with the header contract and ends with the card re-copied from the header and manifest and `state: unverified`; audit verifies and reviews it. `retire` ends in `state: retired`. Before a `fix` or `extend` changes a realization that has a binding, copy it to `prev/<name>.<ext>` when no `prev/` copy exists and its hash equals `binding.realization_sha256` — the copy always holds the last bound version; otherwise audit reviews `full`.

| Class | When | Contract change | Samples | Review at audit |
|-------|------|-----------------|---------|-----------------|
| `fix` | a case inside the range gets a wrong answer | `limits`, `behavior` wording only | add the failing real case (the usage file's `recent_failures` is the first place to look); it fails before the change (`PTL_FIX_NOT_REPRODUCED` otherwise) and passes after — both runs noted in the task file; no sample removed or changed. A `fix` that changes only `limits` or `behavior` wording, not the realization, needs no new sample | `diff` |
| `extend` | a near case: a new input, file type, option or output field | additive only — a new case, optional input (with its own `invocation` element), enum value or output field (`PTL_NOT_ADDITIVE` otherwise → `repurpose`) | ≥ 1 new sample per new case and per new optional input; every old sample unchanged and passing — the proof that existing users are unaffected | `diff` |
| `port` | the realization does not run here: a missing program, another platform | `environment` and `invocation` only | unchanged; all pass in the target environment | `full` |
| `repurpose` | the purpose changes; old samples no longer express the intent | — a new tool | Promote under a new name; retire the old one or leave it | `full` |
| `retire` | replaced, merged, or no `exercised` in the retention window (~30 days; usage file or ledger observation) | — | kept | independent review of the deletion |

Code Reuse restraint: a near case that changes for a different reason than the tool is not `extend` but a new tool (`PTL_DIFFERENT_REASON`).

**Retire.** The deletion is a doubtful write ([Experience Capture](experience-capture.md#applying-experience--auto-apply-through-structural-gates)): an independent clean-context review of the reason. Confirmed → delete the realization and any `prev/` copy, set `state: retired` and `retired` (`replaced_by` when replaced), keep the card, samples and usage file, rebuild the router; the ledger entry stays and gets no further verdicts. Not confirmed → nothing deleted, report the reviewer's reason (`PTL_RETIRE_REJECTED`).

Two sessions adapting the same tool: the contract, samples and card change together at the end of each adaptation; a session that sees the tool change mid-task runs the trust check again (the hash changed, so its cached verdict no longer holds).

## Regenerate

When the realization is missing (lost or deleted while the card remains), broken beyond `fix`, or unfit for this machine (`port`), write a new one **from the card alone** — `contract` and `samples` plus the sample material — without reading the old code. Its header block is the card's `contract` (for `port`, with the changed `environment` / `invocation`). Run every sample yourself; delete any `prev/<name>.<ext>` copy; the tool becomes `unverified`; audit accepts it with a `full` review and writes a new binding.

- No card (a `harvested` tool) → not regenerable; give it a contract through Adapt first (`PTL_NO_CARD`).
- List in the task file every behaviour you had to guess because the card does not state it. The new realization fails a sample, or the list is not empty → the contract is incomplete (`PTL_CONTRACT_INCOMPLETE`): close each gap in the header — `behavior`, `limits`, or a new sample — before the new realization lands; the card is re-copied from that header, and audit reviews the regenerated tool `full` together with the contract change.

The card is complete when a fresh agent, given only the card and the samples, writes a realization that passes them; every regeneration tests exactly that.

## Use accounting

| Event | Write |
|-------|-------|
| The tool's name enters a Pre-Action Marker ([Application Enforcement](application-enforcement.md)) | usage file `evidence` → `wired` (when lower), source `pre-action-marker` |
| A run returns | usage file `evidence` → `exercised`, source `tool-run` (the exit code is a non-model by-product), `observed_at` refreshed; `runs` counters; on `cannot-decide`, `tool-failure`, `undeclared-write` or `implausible`, a `recent_failures` entry |
| The Transition Checkpoint's coverage rung ([Evidence Discipline](evidence-discipline.md#consumer-1--the-coverage-rung)) finds a hand-rolled check whose covering artifact is a tool | a **bypass** appended to the tool's ledger entry, unless a run of the tool already recorded one for this check ([Run](#run)) |

**Bypass** — `{date, task, reason, note}` in the ledger entry's append-only `bypasses` list; a tool without a ledger entry gets one on its first bypass, with the same `artifact`, `problem_class` and `trigger` as at Promote. A bypass is the entry's `recurred` observation — the hand-rolled check repeated its `problem_class` — so it is recorded once, as the bypass, and reconciliation reads it as `recurred` — a `broken` bypass can give `regressed`, any other at most `unchanged` ([Evidence Discipline](evidence-discipline.md#consumer-3--the-intervention-ledger)). The reason names the adaptation it points to:

| Reason | Means | Leads to |
|--------|-------|----------|
| `unknown` | the tool was not found | reword `purpose`, `applies_to` or `range_of_use` |
| `unfit` | the case lies outside the range | `extend` |
| `untrusted` | the trust check answered `read-first` and reading did not pay | audit, or more samples |
| `broken` | wrong answer or cannot decide | `fix` |
| `unavailable` | does not run on this machine | `port` |

The bypass record is both the measurement of reuse and the work list that raises it. A failed write of the usage file or the ledger is noted in the task file and never blocks the work.

## Legacy tools and the privileged runner

- **`harvested` tools** — kept before this file, without a header contract: read first, then trust, as before; audit keeps running their self-test and proposes a contract; they receive one through Adapt.
- **`developer-word` — the privileged runner** — stays outside this lifecycle and is owned by [Proportionate Caution](proportionate-caution.md#privileged-commands). The trust check answers `read-first (legacy)` and defers to it; audit does not run or review it; Adapt, Regenerate and `/dev-flow tool` refuse it except for list / show / find (`PTL_RUNNER`).

## Edge cases

| Case | Answer |
|------|--------|
| Realization edited outside Adapt, `.dev_flow/` ignored by git | trust check `read-first (hash-mismatch)`; audit reviews it `full` as an adaptation; never re-binds on passing samples alone |
| Header edited outside Adapt | hash mismatch → `read-first`; audit sees header ≠ card and reviews the change as an adaptation |
| Tool needed by another session before audit | `read-first (state-unverified)`; read it, or offer `/dev-flow audit tools`; do not verify it mid-task |
| Tool reaches an external service | every sample sets `effects.replay_input`; what a replay cannot show is in `limits` |
| Samples pass but the tool is wrong on a case without a sample | the samples are the contract: add the case (`fix`); if it lies outside the purpose, record it in `limits` |
| Usage file missing or unreadable | treated as `unobserved` with zero runs; recreated on the next event; trust unaffected |
| Promote with a retired name | `PTL_NAME_TAKEN` |
