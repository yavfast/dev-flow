# Docs Scaling — Reading, Navigating, and Splitting Large Documentation

Cross-cutting conventions that keep the cost of finding and reading a fragment constant as the `docs/` corpus grows: a grep-first reading protocol, in-document navigation (Contents + lead summary), an adaptive history policy, a split policy for documents that outgrow one file, and the corpus mechanics they rest on (anchors, index router, self-sufficient headings). Every phase that reads or writes pipeline documents applies these conventions; the mechanical checks run in the audit phase (report only) and the mechanical fixes in the propagate phase. Advisory: no gate criterion and no verdict set changes. Scope: pipeline documents in `docs/` — spikes and `ext_adoption/` are exempt; the knowledge catalogues `.dev_flow/rules/` and `.dev_flow/skills/` consume the named constants through [Knowledge Scaling](knowledge-scaling.md).

## Reading protocol (grep-first)

A pipeline document is consumed in fragments, not whole. Choose the path by goal:

1. **Whole-file operation** (review, gate check, propagate over the file) — read the full file. Always permitted.
2. **File at or under `full_read_limit`** — read the full file; the protocol adds nothing on small files.
3. **Known traceable ID** — string-search the file for the anchor pattern `{#ID}` with positions, then read only from the hit to the section end (the next heading of the same or higher level, or end of file). Do not delegate a known-ID lookup to a subagent — it costs more than the direct search.
   - No hit in the file: the section may have moved to a child file of a split set — search the corpus for the anchor before treating the ID as wrong.
   - More than one definition site in the corpus: a corpus defect (duplicate anchor) — report it, do not guess.
4. **Topic, no known ID** — list the file's h2/h3 headings with positions (this listing *is* the current table of contents and never goes stale), pick the section, read its range.

Delegate searching to a subagent only for fan-out questions across many files. A legacy document without anchors degrades to searching heading text; the audit reports the coverage gap, and anchors are added at the section's first substantive edit.

## Thresholds

The single source of numeric thresholds in this skill. Other files name a constant and point here — never restate a value.

| Constant | Value | Metric |
|----------|-------|--------|
| `soft_split` | 8 000 words | words = whitespace-separated tokens |
| `hard_split` | 12 000 words | words |
| `h3_density` | 25 h3 headings inside one h2 section | heading count between adjacent h2s |
| `full_read_limit` | 4 000 words | words |
| `paragraph_max` | 1 500 characters | length of one logical line (one paragraph = one line) |
| `lead_lines` | 3–5 logical lines | lead summary length |
| `changelog_overflow` | 30 entries | Changelog table rows |
| `directive_max` | 160 characters | length of a knowledge unit's heading directive / a category `summary` / a skills domain `description` ([Knowledge Scaling](knowledge-scaling.md)) |
| `rationale_max` | 3 logical lines | lines of a knowledge unit's `Why` section |
| `unit_body_max` | 400 words | words of a knowledge unit's body, heading to next h2 |
| `digest_min` | 5 units | h2 units in a knowledge file above which a digest is required |
| `activation_budget` | 20 units | items in one relevant set at a knowledge gate; per phase in the audit overload probe |
| `consolidation_min` | 5 rules | rules of one category sharing an effective selector; above it audit reports a `skill-candidate` cluster |
| `stale_after` | 3 months | age of a document's `Updated` date above which it is treated as stale (review, plan staleness, removal candidacy) |
| `phase_stalled_after` | 2 months | age of a plan phase held at `[IN PROGRESS]` above which it is reassessed as abandoned |

Consumption rule: each constant is read only by a cheap mechanical count inside the audit checks or the reading protocol above. Authors and reviewers never count words, characters, sections, or entries by hand — they respond to report verdicts and write by habit ("one paragraph — one statement"; "a section you cannot scan is a split candidate"). Boundary semantics: a trigger fires strictly above its threshold, never at equality. A numeric review finding ("exceeds N words") requires the mechanical measurement attached as provenance; without it the claim is `unobserved`, not a severity finding. A threshold whose cheap measurement is unavailable degrades to judgment without the number — work never stops for a recount.

## Contents and lead summary

**Lead summary** — the descriptive paragraph of the header blockquote, after the metadata fields, within `lead_lines`: what the document or module is, who needs it in which situation, what parts it consists of (for an umbrella — the children). Reading the header alone must suffice to decide "read on or not". The lead answers *why*, Contents answers *where* — they do not duplicate each other.

**Contents** — the first h2 of the body, immediately after the header blockquote, titled `## Contents`, carrying no anchor. Each item is one line:

    - [NN. Section Title](#SP_XXX_NN) — one-line annotation of what the section defines

- Items list **substantive h2 sections only** — never h3, never service sections. The link text matches the heading text verbatim; item order matches section order; each substantive h2 has exactly one item; the annotation is non-empty and states content, not a restatement of the title.
- **Plans have no Contents section** — the linked Progress list serves that role: each Progress item links to its phase anchor (`- [ ] [P1 — Name](#PL_XXX_P1)`).
- Maintenance: the edit that adds, renames, or removes an h2 (or a plan phase) updates the matching Contents (Progress) item **in the same edit**. New documents get Contents from the template. Existing documents get it lazily at the first substantive edit; a one-off backfill applies only to files above `hard_split`.

**Service sections** (closed list, matched by literal heading) carry no anchor, get no Contents item, and are excluded from the anchor-coverage check: for all document types — `Contents`, `Changelog`; in plans additionally — `Goal`, `Technology Decisions`, `Required Knowledge`, `Progress`, `Phases` (the h2 container itself; the phase h3s inside are substantive), `Backlog`. Extending the list is an edit to this reference. Epics are outside the anchor convention entirely — an epic is addressed as a whole (`E_XXX`); anchors on its sections are permitted, not required.

## History policy

**Content filter — unconditional.** Applies wherever history is kept: document Changelogs and task Activity entries alike. A history entry must belong to one of these classes:

| Class | Meaning | Example |
|-------|---------|---------|
| `incident` | A failure, defect, or violated invariant and its resolution | "split broke 3 references; fixed by auto-fix" |
| `ambiguous-decision` | A choice between options with non-trivial rationale (usually a pointer to a DEC) | "chose option B — see DEC_02" |
| `structural-event` | A lifecycle event invisible to file-system history: deprecate, v2, a section moved by a split, a status change through a gate | "02_05 moved to name.audio.sp.md" |

An event outside these classes — a success report, an interim status, a retelling of work done — is **not written anywhere** (the TRIVIAL_ENTRY refusal: a silent no-op, no error raised). Progress is expressed by plan and task checklists, not by history entries.

**Medium selection** — the agent's call from project context:

| vcs present | churn | Medium | Consequence |
|-------------|-------|--------|-------------|
| yes | normal | `vcs-primary` | History = commits with traceable IDs + task files; the document Changelog is optional and thin |
| yes | high | `dense` | As `vcs-primary`, but the agent deliberately keeps a denser in-doc Changelog (still within the content filter) |
| no | any | `in-doc-primary` | The in-doc Changelog is mandatory — the only durable medium |

**Immutable core** under any medium: the `Updated:` field, DEC records, and `structural-event` entries stay in the document — the only history vcs cannot express.

Format: one entry = one logical line (subject to `paragraph_max`); extended narrative lives in the task file or in `name.log.md`, never in the document body. When a Changelog exceeds `changelog_overflow` rows, move the older rows to an adjacent `name.log.md` (outside the audit integrity checks) and leave a pointer row.

**Medium switching:** vcs disappears from the project → the Changelog becomes mandatory from that moment, the past is not reconstructed; vcs appears → the in-doc Changelog is frozen as an archive and new history follows the vcs branch; the developer or agent declares the project high-churn → switch to `dense`.

## Split policy

**Triggers** — derived from measurement at every audit run, never stored in the document:

| State | Condition | Verdict |
|-------|-----------|---------|
| split-planned | WORD_COUNT > `soft_split` | Plan the split at the next substantive edit |
| split-required | WORD_COUNT > `hard_split`, or h3 density > `h3_density` | Split before adding new material; do/plan consume the verdict |

The audit reports the verdict; the decision to split stays with the agent and developer. A child file that itself crosses a threshold goes through the same cycle recursively.

**Patterns:** plan → per-phase (children `name.p<NN>.plan.md`); spec → per-module (children `name.<module>.sp.md`); custom — with developer agreement.

**Umbrella set.** The umbrella keeps the original file name (`name.plan.md`, `name.sp.md`) and is the canonical entry point — only umbrellas are listed in `_index.md`. The umbrella holds: the lead summary, Contents (with links into children for moved sections), shared sections, a child table (link + what it holds + its Status), DEC records, and the Changelog per the history policy; a plan umbrella also holds Goal, Technology Decisions, and the linked Progress. The umbrella aggregates the children's statuses — a plan umbrella is `completed` only when every child is. When all children are closed or deprecated, the umbrella expresses that in its own status; no separate collapse procedure exists.

**ID invariant (the ID_MUTATION refusal).** No split operation changes, renames, or renumbers any traceable ID: sections move to child files verbatim, anchors intact. A split plan that would touch any `{#ID}` is rejected and redone. Verification: the set of corpus anchors before and after the split is equal. Versioning is orthogonal: v2 = a new ID space; a split = the same ID space in a different host file.

**Split procedure:** propose the split (which sections → which children) → assert no anchor changes → create children and move sections verbatim → rewrite the original as the umbrella → fix the **path part** of every corpus reference to a moved anchor (the ID part never changes; propagate auto-fix) → append a one-line `structural-event` entry to the umbrella and to each child → update `_index.md` to list only the umbrella.

Rollback: an umbrella set merges back into one file without loss — the anchors are verbatim the same, and references are repaired by the same path fix in reverse.

## Index router

`_index.md` is a router, not a catalogue of sections. Per pipeline file: a link (umbrellas only for split sets) + a one-line annotation — what is inside, enough to pick the file without opening it — plus a map of ID prefix → file (`SP_AUD_PLAY` → `audio.player.sp.md`).

- Section listings (a file's anchor list) are forbidden in the index.
- Prose in the index header falls under the history content filter above.
- A `docs/` subdirectory may carry its own `_index.md`; the more specific index wins for its files.
- Index format split: a machine-read catalogue (`.dev_flow/rules/`, `skills/`, `roles/`, `cache/`) uses `_index.yaml` — structured entries agents match against; a human-browsed catalogue (`docs/`, `.dev_flow/tasks/`, `.dev_flow/todos/`) uses `_index.md`. Apply the same split to any new collection.

## Self-sufficient headings

Word a section heading so it is understandable without its parent headings — the reader usually meets it in isolation, in a grep hit or a headings listing, not under its h2. "Validation — split operations" locates itself; "Rules" does not.
