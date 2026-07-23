# Documentation Formatting — Conventions for All Generated Doc Files

Cross-cutting formatting rules for **every documentation file the pipeline produces or edits** — spikes, concepts, specs, plans, epics, task files, indexes, rules, skills. Apply them whenever a phase writes Markdown; they are conventions of the *source text*, independent of how any renderer displays it.

## The core rule — no hard wraps inside a sentence or paragraph

**One paragraph = one logical line.** Never insert a line break in the middle of a sentence or paragraph to keep lines under some column width (80, 100, 120 — any). Let long lines stay long; editors soft-wrap them. A line break in a doc file appears only *between* block elements: paragraphs, list items, table rows, headings, code fences.

**Why this must be stated explicitly.** LLMs inherit an ~80-column hard-wrap habit from their training corpora (man pages, mailing lists, docstrings, commit messages) and will reintroduce it unless instructed otherwise. Rendered Markdown hides the problem — renderers merge single newlines into flowing text — while soft-wrap editors and diffs expose it: editing one word re-flows every following line of the paragraph, turning a one-word change into a multi-line diff that buries the real edit. This reference is the standing override; do not "helpfully" re-wrap for readability.

```md
<!-- WRONG — hard-wrapped mid-sentence -->
The cache index is checked before every expensive fetch, and a cached
copy is reused whenever its currency check passes, so repeated runs
stay cheap.

<!-- RIGHT — one paragraph, one line -->
The cache index is checked before every expensive fetch, and a cached copy is reused whenever its currency check passes, so repeated runs stay cheap.
```

## Where line breaks ARE correct

- **Between block elements** — a blank line separates paragraphs, headings, lists, tables, code fences, blockquotes.
- **Inside code blocks** — fenced code preserves newlines by design; format code by the code's own conventions.
- **ASCII diagrams** — line structure *is* the content; keep it exactly.
- **YAML frontmatter and `_index.yaml` entries** — structured per YAML syntax, one key per line.
- **Table rows** — one row per line (see below).

**Out of scope:** commit messages keep their own conventions (subject ≤ 72 chars, wrapped body) — those rules apply to git, not to doc files.

## Lists, tables, headings

- **One list item = one line**, however long it gets. Do not wrap a long item onto continuation lines; if an item is truly too long, that is a signal to split it into sub-items, not to wrap it.
- **One table row = one line.** Never break a cell across lines; keep cell text tight instead.
- **Blank line between block elements** — before/after every heading, list, table, and code fence.
- **Consistent heading hierarchy** — no skipped levels; structural headings stay exactly as the phase templates define them (they carry traceable anchors like `{#C_XXX_01}`).

## No wholesale reformatting

Apply these rules to **new and edited text only**. Never reflow an entire existing file just to fix its wrapping — a whitespace-only diff buries the real changes and pollutes blame/history. Fix wrapping opportunistically in the paragraphs you are already editing; reformat a whole file only when the user explicitly asks for it.

## Language consistency

The documentation **body** follows the project's chosen working language (mirroring each phase's Language Policy); structural section headers stay in the exact form the templates define them, whatever the body language. Do not mix languages within one document body.

## Quick checklist (before saving any doc file)

- [ ] No line break inside any sentence or paragraph
- [ ] Blank line between all block elements
- [ ] Each list item and table row on a single line
- [ ] Template-defined headings and anchors untouched
- [ ] Only touched text re-formatted — no file-wide reflow
