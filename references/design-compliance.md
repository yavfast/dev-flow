# Design Compliance — Validating an Implementation Against Its Design Source of Truth

Cross-cutting sub-procedure of the **Verify** phase (and available on demand through `do` — "does this match the design?"). When a change touches user-facing UI that has a design source of truth, a dedicated validator compares the implementation against that source **property by property** and reports every deviation with evidence. It does not fix code — deviations feed the normal Verify fix cycle (fix → Test → Review → Verify).

This reference is **application-type-agnostic**: the same procedure applies to web, mobile (Android/iOS), desktop, and terminal (TUI) applications — only the inspection tooling differs (see Step 3).

## When it runs

- **In Verify**, automatically, when the verified change affects visual UI **and** a design source of truth exists (a design-tool link in the task/spec, design tokens in the repo, mockups referenced by the docs). It is part of live verification, alongside the end-to-end scenarios.
- **On demand**, when the user shares a design link alongside code or asks to check that a build "matches the design".
- **Not** for changes with no design source — then visual checks stay part of ordinary live testing, and this procedure reports "no source of truth" instead of inventing one.

## Execution shape

The check runs as a **read-only, clean-context validator subagent** — the same shape as pre-commit review ([Delegation for Focus](delegation.md)): the diff author's rationalizations don't leak into the comparison, and the noisy material (design exports, screenshots, computed-style dumps) stays out of the main context. The subagent returns the compliance report — conclusion, not dump; raw artifacts go to the project workspace `/tmp/{project-slug}/`, and durable baselines (reference screenshots, design exports) are promoted to `.dev_flow/cache/` per [Resource Cache](cache.md).

Visual comparison is judgment-heavy — pick the subagent model by that nature, never by a hardcoded name ([Delegation → model fit](delegation.md#fit-the-model-to-the-task--by-its-nature-not-its-name)). Use whatever inspection tools the runtime actually offers (design-tool MCP, browser automation, emulator/simulator control, screenshot capture); when a tool is missing, degrade from rendered inspection to source + tokens (Step 3) and state so in Coverage. If the check recurs in a project, grow a **specialist role** under `.dev_flow/roles/` that knows its screens and design system ([Named specialists](delegation.md#named-specialists-and-the-routing-reflex)).

The validator runs to completion on its own: it asks its initiator only when genuinely blocked (e.g. no design source can be located at all).

## Step 1 — Establish the design source of truth

Find what the implementation is supposed to match, in order of preference:

1. A **design-tool link** (Figma URL/node or equivalent) provided in the task, spec, or task file — the expected default. Extract exact specs through the available design-tool MCP: layout, spacing, color, typography, effects, and design variables/tokens. Check `.dev_flow/cache/_index.yaml` before fetching and save expensive exports back ([Resource Cache](cache.md)).
2. A **design tokens / theme definition in the repo** — treat these as canonical values. Per platform: web — `tokens.json`, Tailwind config, CSS custom properties, theme files; Android — resource XML (`colors.xml`, `dimens.xml`, `styles.xml`) or Compose theme objects; iOS — asset catalogs, SwiftUI theme/environment values; desktop — Qt/GTK/XAML theme resources; TUI — the style/palette map of the TUI framework.
3. **Project documents** — UI interaction contracts in the relevant `*.sp.md`, a style guide or design-system doc in the repo.
4. **Static mockups/screenshots** referenced by the task or docs — read them visually.
5. **Platform guidelines** (Material, Apple HIG, etc.) — only when the task or spec explicitly names them as the standard, never as an assumed default.

If no source of truth exists, stop and report that compliance cannot be validated, naming exactly what was looked for. A conflict *between* sources (e.g. the spec says one value, the design file another) is a fork — surface it via [Upstream Escalation](escalation.md), never resolve it silently.

## Step 1b — Enumerate the FULL component: every variant and state (mandatory)

The node, mockup, or screen you were handed is usually a **single frame or a demo instance**, NOT the complete component. It shows only the states someone happened to place there. Treating the visible states as the full set is the single most common way this check misses defects (e.g. hover/pressed decorations).

Before comparing, discover every variant and state the design defines:

- **Trace instances back to their master component.** If the handed node contains instances, the real spec lives in the main component/component set — often in a shared library, possibly not instantiated in the node you were given.
- **Enumerate the component set.** Search the design system/library by the component's name to find the component set AND any sibling state components (`Tab Hover`, `Tab Pressed`, `Tab Disabled`, `… / Focus`) — a design may model states as separate components rather than one variant set; find them all. When the source of truth is tokens or a style guide rather than a design file, enumerate the states *those* define (state-suffixed tokens, interaction sections of the guide).
- **Cover the platform's state axes**, where the design defines them: interaction states (hover, pressed/ripple, focus, selected, disabled, drag), content states (loading, empty, error), color schemes (light/dark themes), text scaling / dynamic type, orientation and size classes / breakpoints, RTL.
- For each state found, pull its spec and record what visually distinguishes it from the resting state — **especially decorative elements** (indicator lines, underlines, border changes, background tints, icons), not just text color.

Produce an explicit inventory of states to validate. This inventory drives Step 4.

## Step 2 — Identify the implementation

Determine the changed/target code (prefer `git diff HEAD`, else the files named in the task). Read the components, styles, and token usage in whatever UI technology the project uses — HTML/CSS/JS components, Compose/SwiftUI/XML views, QML/XAML/widget code, TUI widget definitions.

## Step 3 — Inspect the rendered result when possible

Compare **rendered** values, not just source, whenever the app can be run — per application type:

| App type | Rendered inspection |
|----------|--------------------|
| Web | Serve the app, open the screen via browser automation, read the DOM and computed styles (`getComputedStyle`), check the required breakpoints |
| Mobile | Build to an emulator/simulator or device, capture screenshots per state, use the layout inspector / view-hierarchy dump where available |
| Desktop | Launch the app, capture window screenshots, read the accessibility/widget tree where the toolkit exposes one |
| TUI / CLI | Run in a captured terminal (fixed size), capture the screen buffer, compare glyphs, colors, and layout at the required terminal sizes |

Launching and driving the app follows the Verify phase's [Safe Testing Principle](../phases/verify.md#safe-testing-principle) — test accounts, test entities, sandbox environments. If the app cannot be run, validate against source + tokens and say so explicitly in Coverage.

## Step 4 — Compare, property by property

For each element in the design, verify the implementation against these properties:

- **Layout & spacing**: dimensions, insets/padding, margins, gaps, alignment, stack/grid structure.
- **Color**: fills, text, borders, per state — resolved to concrete values; confirm tokens are used, not hardcoded near-matches.
- **Typography**: family, size, weight, line-height, letter-spacing, transform (and dynamic-type / font-scaling behavior where the platform defines it).
- **Borders & radius**, **shadows/elevation/effects**, **opacity**.
- **Iconography & assets**: correct asset, size, color, density/scale variants.
- **States**: validate EVERY state from the Step 1b inventory. For each, map the design state to the implementation's corresponding mechanism — CSS pseudo-classes/classes on web, interaction modifiers / state callbacks in Compose/SwiftUI, state lists/selectors in Android XML, widget state styles in desktop/TUI toolkits — and diff **every distinguishing feature of that state, not just text color**: a missing hover/pressed indicator line, underline, border, or background tint is a deviation. A state the design defines but the code omits entirely is a Major-or-worse deviation, not an "N/A".
- **Adaptive behavior**: matches the design across the specified breakpoints / size classes / orientations / terminal sizes, and across defined themes (light/dark) and RTL where applicable.
- **Structure/naming**: the component maps to the design's component/variant intent (and to the spec's UI contract naming, where one exists).

Allow a small tolerance (≤1px / ≤1dp, or documented rounding) and note it; anything larger is a deviation. Distinguish "design token available but not used" from "value simply wrong".

## Output — Compliance report

Lead with an overall verdict — **PASS**, **PASS WITH DEVIATIONS**, or **FAIL** — plus a count of matched vs. deviating properties.

Then a table or list of deviations, most severe first. Each entry:

- **Property** (e.g. "Button / horizontal padding")
- **Expected** (from design/token) vs. **Actual** (from code/rendered)
- **Location**: `file:line` (plus DOM selector / view id / widget path when rendered)
- **Severity**: Critical (breaks design intent) / Major (clearly off) / Minor (small/rounding)
- **Fix**: the exact value or token to use.

Finish with a **Coverage** note: what was verified (source-only vs. rendered), which states/breakpoints/themes were checked, and anything that could not be validated and why. Report only deviations confirmed against evidence — never guess. If everything matches, say so plainly.

**Routing the results.** Deviations enter the Verify [fix cycle](../phases/verify.md#fix-cycle); a deviation whose *expected* value traces to a defect or conflict in the spec/design itself goes through [Upstream Escalation](escalation.md) instead of a code bend. Recurring deviation classes (e.g. "hardcoded values where tokens exist") are harvested as rules at the Verify reflection checkpoint ([Experience Capture](experience-capture.md)).

## Never assert an unverified absence

Do not write "the design has no hover state", "there is no X variant", or "no responsive spec exists" unless you have enumerated the component set (Step 1b) and confirmed it. If you did not or could not check, say "**not verified**" and list it under Coverage as a gap — an unchecked state is an open question, never a silent PASS. Distinguish "confirmed absent (enumerated the variants)" from "not present in the node/mockup I was given".
