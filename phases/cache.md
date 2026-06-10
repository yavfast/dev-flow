# Phase: Cache — Durable Project Resources & Temp Workspace

## Purpose

Working on a project produces two kinds of non-code artifacts, and they need
opposite treatment:

- **Durable / expensive resources** — Figma layouts and design exports, documents
  downloaded from the internet, baseline application screenshots, data samples.
  Re-acquiring them costs real money, rate-limited API access (e.g. the Figma MCP),
  or manual effort — and anything *linked from docs or task files* must outlive the
  session. These live in **`.dev_flow/cache/`**: a per-project store with a
  hierarchy and an `_index.yaml`, surviving reboots.
- **Transient artifacts** — test/build logs, reproduction dumps, throwaway
  screenshots, spike prototypes. Cheap to regenerate, worthless next week. These
  live in a **project workspace under `/tmp`** with disciplined naming, and are
  deleted without regret.

The boundary in one line: **`/tmp` is staging, the cache is keeping** — anything in
the workspace that turns out durable is *promoted* into the cache, and nothing in
docs or task context ever links into `/tmp`.

## Command

```
/dev-flow cache <request>
```

The request is freeform, in any language. Interpret the intent and apply the
appropriate action to `.dev_flow/cache/`.

### Examples

```
/dev-flow cache save the login-form Figma export for the auth task
/dev-flow cache збережи цей макет дашборда
/dev-flow cache find the cached OAuth RFC download
/dev-flow cache update figma/auth/login-form.png — the design changed
/dev-flow cache remove app/old-onboarding-baseline.png
/dev-flow cache list figma
```

This phase runs **inline** (no subagent), like the [rule](rule.md) and
[skill](skill.md) phases.

## What Belongs in the Cache

Cache a resource when **any** of these hold:

- **Expensive or limited to re-acquire** — rate-limited MCP access (Figma),
  a slow/large download, a manual export sequence, a paid API.
- **Linked from documentation or a task file** — a link in `docs/*` or
  `.dev_flow/tasks/*` must point at something that still exists next month.
  Local link targets MUST live in the cache, never in `/tmp`. For a
  *load-bearing external URL* (a spec the design depends on), cache a snapshot
  copy alongside the link — URLs rot.
- **Needed across sessions** — reference baselines (a screenshot future Verify
  runs compare against), fixtures reused by multiple tasks.

Do **NOT** cache:

- Anything **cheap to regenerate** — logs, build output, test results, transient
  screenshots taken to answer an immediate question. Workspace material.
- **Secrets, credentials, tokens, personal data** — never, in any form.
- Bulky data with **no stated future consumer** — the same minimality bar the
  validation gates apply to documents: "just in case" is not a consumer.

## Directory Layout

```
.dev_flow/cache/
├── _index.yaml      # Catalog — the only place source metadata lives
├── figma/           # Design exports: layouts, frames, component screenshots
├── web/             # Downloaded documents, pages, specs
├── app/             # Application screenshots/recordings kept as baselines
└── data/            # Samples, fixtures, API payloads worth keeping
```

- Inside a domain, group by feature/area subdirectory once a domain exceeds
  ~10 files (`figma/auth/`, `web/payment-provider/`).
- Create a new top-level domain only when none of the four fits.
- **Naming:** kebab-case slug + the original extension (`login-form.png`,
  `oauth2-rfc6749.html`). Snapshots of the same resource over time get a compact
  date suffix — `login-form_20260610.png` — **never** bare numeric indexes.
  (Date-only granularity here; the finer `_YYYYMMDD_HHMMSS` form is for
  workspace transients.)

## The Index (`_index.yaml`)

One entry per cached file:

```yaml
- file: figma/auth/login-form.png
  source: "https://www.figma.com/design/ABC?node-id=12-34"
  summary: "Login form layout — auth flow baseline"
  fetched: 2026-06-10
  valid_until: "auth redesign lands (C_AUTH v2)"   # optional — date or event
  reacquire: rate-limited                           # optional — only when non-trivial
  refs: [docs/auth.concept.md, task_C_AUTH]
```

- `file` — path relative to `.dev_flow/cache/`.
- `source` — where it came from: URL, Figma node, "app screenshot of {screen}",
  or the command that produced it. This is what makes re-fetching possible.
- `summary` — one line; what an agent matches against when looking for a resource.
- `fetched` — date of last acquisition.
- `valid_until` — optional: the date or event after which the resource is suspect
  (`"2026-09-01"`, `"auth redesign lands (C_AUTH v2)"` — same event-or-date
  semantics as a decision's resolution trigger). Absent = valid indefinitely.
  **Set it optimistically**: genuine staleness almost always arrives with its own
  update task (a design refresh, a spec version bump) — paranoid-short terms only
  flood audit with false alarms.
- `reacquire` — optional, only when re-acquisition is non-trivial:
  `rate-limited` / `manual` / `paid`. Guides pruning order (cheap-to-refetch
  entries are proposed for removal first) and removal caution.
- `refs` — optional: docs and task files that link this resource.

> **This index is data, not a derived view.** Unlike the task catalog or
> `docs/_index.md`, the cache index carries `source` metadata that exists nowhere
> else — it cannot be regenerated from the files. Audit *reconciles* it
> (flags mismatches), never rewrites it from scratch.

## Procedures

### Finding (cache-first gate)

**Before any expensive fetch** — a Figma export, a document download — read
`.dev_flow/cache/_index.yaml` and match by `summary`/`source`/`refs`. If a current
entry exists, reuse the file instead of spending limited access. This is the
artifact twin of the skills rule "check skills BEFORE external research".
While `.dev_flow/cache/` is absent, the gate is a no-op (same as the rules/skills gates).

A hit whose `valid_until` has passed is not reused blindly — and not re-fetched
blindly either. When the source offers a **cheap currency check** (HTTP
ETag/Last-Modified, a Figma file's version/last-modified metadata), run it first:
unchanged → keep using the cached copy and extend `valid_until`; changed or
uncheckable → re-fetch via `source`, or flag it — a genuinely stale resource
usually has a dedicated update task behind it (e.g. the design refresh), and the
re-fetch belongs to that task's work.

### Saving

1. Confirm the resource passes the *What Belongs in the Cache* filter.
2. Choose the domain path (ask only if genuinely ambiguous, max 1 question).
3. First save ever? Create `.dev_flow/cache/` with an empty `_index.yaml` and add
   `.dev_flow/cache/` to the project's `.gitignore` (see Git below).
4. Move the file from the workspace (or write it directly), named per the layout rules.
5. Add the `_index.yaml` entry — a cached file without an index entry is
   invisible and therefore wasted.
6. When a doc or task file links the resource, record it in `refs`.

### Updating

**Cheap check before re-fetch.** When the source supports it, verify the resource
actually changed (ETag/Last-Modified, Figma version metadata) before spending the
fetch — unchanged means just extend `valid_until`, no download. When it did
change: re-fetch via `source`, then either replace the file in place (refresh
`fetched`) or add a dated snapshot next to it — keep the old snapshot only while
a doc still references it.

### Removing

On explicit request or an [audit](audit.md) proposal. Check `refs` first — a
resource still linked from a doc or an active task is not removable until the
reference goes. Remove the file *and* its index entry together.

## Auto-Save Triggers

Cache without an explicit command when:

| Trigger | Action |
|---------|--------|
| A Figma export/screenshot is fetched for design or implementation work | Save to `figma/` + index entry — the next fetch of the same node is free |
| A document is downloaded during [research](research.md) | Stage in the workspace; promote what the spike's conclusions rest on at research Step 4 |
| A doc or task file is about to link a local resource | Move the target into the cache first — never link `/tmp` |
| [Verify](verify.md) produces a screenshot worth comparing against later | Promote to `app/` as a named baseline |

When the fetch happens inside a **focus-delegated helper**, "save"/"promote" means:
stage in the workspace and list the path in the report — the cache write itself
belongs to the calling agent (see Workspace Discipline below). A **task-delegated**
subagent ([subtask phase](subtask.md)) writes the cache itself, per this phase.

## Temporary Workspace (`/tmp`) Discipline

All transient artifacts go under **one project root** — `/tmp/{project-slug}/`
(slug = the repository directory name, kebab-case) — never scattered loose in
`/tmp`:

```
/tmp/{project-slug}/
├── logs/         # test / build / run output
├── screenshots/  # transient captures
├── downloads/    # staging for fetched resources (pre-cache)
└── scratch/      # spike prototypes, throwaway harnesses
```

- **Timestamp suffixes, not numeric indexes.** When the same artifact recurs, name
  it `{name}_YYYYMMDD_HHMMSS.{ext}` (`test-run_20260610_143205.log`) — the same
  compact format task files use. Numeric suffixes (`run-1.log`, `run-2.log`)
  collide across sessions, don't sort, and say nothing about when they were made.
- The workspace is **disposable** — a reboot may wipe it at any time. Anything
  that must survive is promoted to the cache the moment that becomes clear.
- **Helper subagents write only here.** A subagent doing one noisy step inside its
  initiator's phase run (researcher, tester — including its Verify duty; see
  [Delegation for Focus](../references/delegation.md)) stages its downloads, logs,
  and captures in the workspace and reports the paths; the **calling agent**
  promotes what is durable to `.dev_flow/cache/`. A **task-delegated** subagent
  ([subtask phase](subtask.md)) executes the owning phase's protocol itself —
  including cache writes with their index discipline — and lists what it
  persisted in its report.

## Git

Add `.dev_flow/cache/` to the project's `.gitignore` by default — it is a
per-machine store of (often binary) artifacts; the goal is surviving reboots, not
versioning. A project may deliberately commit a curated subdirectory (e.g. design
baselines the whole team compares against) — record that decision and narrow the
ignore rule accordingly.

## Relation to Other Phases

| Phase | Relation |
|-------|----------|
| [research](research.md) | Checks the cache before external fetches; its Step 4 promotes durable artifacts here alongside persisting knowledge to skills |
| [skill](skill.md) | Skills capture what you *learned* (text); the cache keeps what you *fetched* (files). A skill may link cache entries |
| [verify](verify.md) | Transient run artifacts in the workspace; promoted baselines in `app/` |
| [fix](fix.md) | Diagnosis artifacts (instrumented logs, repro dumps) live in the workspace; promote only what stays valuable |
| [subtask](subtask.md) | Focus helpers stage in the workspace and report paths — the caller promotes; a task-delegated executor writes the cache itself per this phase |
| [audit](audit.md) | Step 7d reconciles index ↔ disk, flags unreferenced/stale entries and size bloat; removals are proposed, never automatic |

## Anti-Patterns

- Linking a `/tmp` path from a doc or task file (the link dies with the next reboot)
- Re-fetching a Figma node or document that is already cached — check the index first
- Caching logs, build output, or one-off screenshots (workspace material)
- A cached file with no `_index.yaml` entry — invisible, unfindable, wasted
- Numeric suffixes (`screenshot-1.png`, `screenshot-2.png`) instead of timestamps
- Re-downloading a resource when a cheap currency check (ETag/Last-Modified,
  Figma version) would have confirmed it unchanged
- Paranoid-short `valid_until` terms — set them optimistically; genuine staleness
  arrives with its own update task
- Storing secrets, tokens, or personal data in the cache — never
- Scattering project temp files directly in `/tmp` instead of the project workspace
