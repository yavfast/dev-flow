# Documentation Formatting — Conventions for All Generated Doc Files

Cross-cutting formatting rules for **every documentation file the pipeline produces or edits** — spikes, concepts, specs, plans, epics, task files, indexes, rules, skills. Apply them whenever a phase writes Markdown; they are conventions of the *source text*, independent of how any renderer displays it. The rules this reference delegates to [Docs Scaling](docs-scaling.md) carry that reference's own scope, which exempts spikes and `ext_adoption/`.

**Scope boundary.** This reference owns the *mechanics* — line breaks, list and table shape, heading hierarchy. Its counterpart [Output Styles](output-styles.md) owns the *register* — which language, how complex, and how a pointer to another document is carried.

## The core rule — no hard wraps inside a sentence or paragraph

**One paragraph = one logical line.** Never insert a line break in the middle of a sentence or paragraph to keep lines under some column width (80, 100, 120 — any). Let long lines stay long; editors soft-wrap them. A line break in a doc file appears only *between* block elements: paragraphs, list items, table rows, headings, code fences.

**Standing override.** LLMs re-introduce ~80-column hard wraps by habit and renderers hide the result; do not "helpfully" re-wrap for readability. A re-flowed paragraph turns a one-word change into a multi-line diff that buries the real edit.

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

## Paragraph size and prose value

- **A long paragraph splits into several.** One paragraph carries one statement; a paragraph longer than `paragraph_max` (value in [Docs Scaling](docs-scaling.md)) is divided into several paragraphs within the same section. Reading tools silently truncate overlong lines, so an oversized logical line loses its tail unseen. Legacy oversized paragraphs are fixed when that paragraph is edited, never by a mass campaign.
- **No prose without informative or semantic value in pipeline documents.** Filler, restatements, and ceremony are not written; the rule is applied by the author at writing time and by the reviewer as an ordinary style finding — it is not an audit check.
- **Self-sufficient headings** — word each heading so it is understandable without its parents; the rule lives in [Docs Scaling](docs-scaling.md).

## Lists, tables, headings

- **One list item = one line**, however long it gets. Do not wrap a long item onto continuation lines; if an item is truly too long, that is a signal to split it into sub-items, not to wrap it.
- **One table row = one line.** Never break a cell across lines; keep cell text tight instead.
- **Blank line between block elements** — before/after every heading, list, table, and code fence; a list or table may follow its lead-in line (the one ending in `:`) directly.
- **Consistent heading hierarchy** — no skipped levels; structural headings stay exactly as the phase templates define them (they carry traceable anchors like `{#C_XXX_01}`).

## No wholesale reformatting

Apply these rules to **new and edited text only**. Never reflow an entire existing file just to fix its wrapping — a whitespace-only diff buries the real changes and pollutes blame/history. Fix wrapping opportunistically in the paragraphs you are already editing; reformat a whole file only when the user explicitly asks for it.

## Language consistency

The documentation **body** follows the project's documentation language (set by the `documentation` register — see [Output Styles](output-styles.md), which may differ from the chat language); structural section headers stay in the exact form the templates define them, whatever the body language. Do not mix languages within one document body.

## Quick checklist (before saving any doc file)

- [ ] No line break inside any sentence or paragraph
- [ ] Blank line between all block elements (a lead-in line may abut its list or table)
- [ ] Each list item and table row on a single line
- [ ] Template-defined headings and anchors untouched
- [ ] Only touched text re-formatted — no file-wide reflow
- [ ] New/edited paragraphs each carry one statement, none overlong (see [Docs Scaling](docs-scaling.md))
- [ ] No filler prose; headings self-sufficient (see [Docs Scaling](docs-scaling.md))
