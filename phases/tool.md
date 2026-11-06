# Phase: Tool — Manage Project Tools

## Contents

- [Purpose](#purpose) — what the command is for and how it relates to the procedures agents call directly
- [Command](#command) — `/dev-flow tool <request>` syntax and examples
- [Procedure](#procedure) — knowledge gate, intent → procedure table, refusals, reporting
- [Relation to Other Phases](#relation-to-other-phases) — who else runs the same procedures

## Purpose

List, find, create, adapt, regenerate, verify, or retire the project tools in `.dev_flow/tools/` — small executables that agents write for agents, each with a header contract, acceptance samples, a card and a binding. This is the developer's direct route to the procedures of [Project Tools](../references/project-tools.md); agents keep calling the same procedures from their phases (promotion at the reflection checkpoint, the trust check before a run, verification in [audit](audit.md)). The command opens no path with other rules.

Tools differ from rules, skills and the cache:
- **Rules** (`.dev_flow/rules/`) — how to write code in this project
- **Skills** (`.dev_flow/skills/`) — what to know about a technology or pattern
- **Tools** (`.dev_flow/tools/`) — what to run: a verified executable that replaces a hand-rolled check or operation
- **Cache** (`.dev_flow/cache/`) — fetched artifacts

## Command

```
/dev-flow tool <request>
```

The request is freeform, in any language. Interpret the intent and run the matching procedure.

### Examples

```
/dev-flow tool покажи інструменти
/dev-flow tool find a check for dangling doc anchors
/dev-flow tool збережи скрипт /tmp/my-project/tools/check-anchors.py як інструмент
/dev-flow tool extend dangling-anchors to cover HTML id attributes
/dev-flow tool fix dangling-anchors — reports clean on docs/api.md although two links are broken
/dev-flow tool port dangling-anchors to this machine
/dev-flow tool regenerate dangling-anchors
/dev-flow tool verify dangling-anchors
/dev-flow tool retire old-link-check — replaced by dangling-anchors
```

## Procedure

1. **Knowledge gate.** Read `.dev_flow/tools/_index.yaml` (the router) and, for the tools the request names, their cards and usage files. Absent catalogue → only `create` applies; any other intent reports that no tools exist.
2. **Interpret the request** and run one procedure:

| Intent | Procedure |
|--------|-----------|
| list / show | Router read: per tool `regime`, `state`, `purpose`; from the usage file the evidence state, run counts and recent failures; bypass counts from the ledger. Writes nothing |
| find | Match the need against each tool's `purpose`, `range_of_use` and `not_for`; answer with the fitting tools and their [trust check](../references/project-tools.md#trust-check) verdict. Writes nothing |
| create | [Promote](../references/project-tools.md#promote) — from a script in the session workspace, or from a description (write the script in the session workspace first, run it on real material, then promote) |
| fix / extend / port / repurpose | [Adapt](../references/project-tools.md#adapt) with that class |
| regenerate | [Regenerate](../references/project-tools.md#regenerate) from the card alone |
| verify | The audit tools step ([audit](audit.md) Step 7g) for the named tools, all when none is named; `--dry-run` as in audit |
| retire | [Adapt](../references/project-tools.md#adapt) `retire` — through the independent review |

3. **Refusals.**
   - The request maps to more than one intent or tool (`PTL_AMBIGUOUS`) → ask with marked options and a recommendation ([Interview Mode](../references/interview-mode.md)); run nothing until answered.
   - Any intent other than list / show / find on a `developer-word` entry (the privileged runner) → refused (`PTL_RUNNER`) with a pointer to [Proportionate Caution](../references/proportionate-caution.md#privileged-commands).
   - A procedure error (`PTL_*` in [Project Tools](../references/project-tools.md)) → report it with its guidance; a failed promotion or adaptation writes nothing.
4. **Report.** Name every tool created, changed, verified or retired, its new state, and what awaits audit; offer `/dev-flow tool verify <names>` or `/dev-flow audit tools` for tools left `unverified`. `.dev_flow/` may be ignored by git — name the tools in the commit message when the change is committed.
5. **Task context.** A command that writes runs inside a task like any other phase (task file, Subtask block, activity); list / show / find need none.

## Relation to Other Phases

| Phase | Runs the same procedure |
|-------|-------------------------|
| [implement](implement.md) · [fix](fix.md) · [verify](verify.md) reflection | Promote at the reflection checkpoint |
| Every phase with a tool step | Trust check and run, through the knowledge gate and the Pre-Action Marker |
| [audit](audit.md) Step 7g | Verification, independent review, binding — `verify` here is the same step |
| [do](do.md) | Routes a freeform tool request here |
