# Security Scope — Security Work by Phase and Stage, a Finding by Its Real Risk

Cross-cutting discipline of [concept](../phases/concept.md), [specification](../phases/specification.md), [plan](../phases/plan.md), [implement](../phases/implement.md), [fix](../phases/fix.md), [research](../phases/research.md), [review](../phases/review.md), [todo](../phases/todo.md) and the [code audit](code-audit.md). Not a pipeline stage and no command. A project rule in `.dev_flow/rules/` overrides its defaults: a project rule about security is binding under its own severity, and its finding is a finding of that rule — the Rules compliance check — with no risk argument.

## Contents

- [Why this exists](#why-this-exists) — security work before anything can be attacked delays the product and protects nothing
- [Protect, never restrict](#protect-never-restrict) — the main rule: a security measure never takes away a function
- [Security work by phase and stage](#security-work-by-phase-and-stage) — what each phase does about security, and the stage `prototype`
- [Secure-coding baseline](#secure-coding-baseline) — the rules code follows at every stage
- [Risk argument](#risk-argument) — every security finding names its actor, entry, chain, precondition, exposure and impact
- [Risk — computed, not judged](#risk--computed-not-judged) — `real` blocks, `deferred` waits for its stage, `hypothetical` is a recommendation
- [Boundaries](#boundaries) — what the stage and the risk never change

## Why this exists

A vulnerability exists only where someone outside the team reaches code that does harm. A product that does not exist yet has no such place: a finding about a design document or a prototype attacks nothing, and a blocker built on it stops the work while protecting nothing. A finding that names no path from an actor to the harm is an opinion, not a defect. A protection that disables the function it protects is a defect of its own.

## Protect, never restrict

Security protects the functions of the product; it never restricts them. A security measure — a control the agent writes, a fix a reviewer or an auditor prescribes — never removes, disables or narrows a function the request, the developer, a spec contract or a project rule names, and never adds a confirmation, a limit or a step nobody asked for.

When the only protection found restricts such a function, the choice is the developer's, asked in [Interview Mode](interview-mode.md#4-ask--and-how-to-ask): protect another way (the answer names the way), accept the risk — a decision record in the owning document, the task file when there is none, which closes the finding — or restrict the function. The agent chooses neither the restriction nor the acceptance; until the answer the function stays as it is and the finding keeps its materiality.

## Security work by phase and stage

| Where | Security work |
|-------|---------------|
| Any phase while the stage written in the task file is `prototype`; a [research](../phases/research.md) prototype | None, except keeping a real secret out of the places where secrets never live ([Proportionate Caution → Secrets](proportionate-caution.md#secrets)). Code follows the [baseline](#secure-coding-baseline), which costs nothing; the review opens no Security check, and a code audit's security finding other than a `secret-exposure` is a recommendation. A security mechanism that is itself the idea the prototype checks is that idea's function, built as such |
| [Concept](../phases/concept.md) | Only when the concept's subject is a security mechanism — authentication, authorization, encryption, a sandbox, the handling of secrets — or the request names security; then it is a mechanism like any other |
| [Specification](../phases/specification.md), [plan](../phases/plan.md) | Only as architecture: the trust boundaries the design crosses, who may write or act on each entity, where data must not go, where secrets live. A hypothetical vulnerability of code not yet written is neither a finding nor a gate criterion |
| [Implement](../phases/implement.md), [fix](../phases/fix.md) | The [baseline](#secure-coding-baseline), and the controls the spec's architecture decided |
| [Review](../phases/review.md) | The Security check, opened by the stage; each finding carries its [risk argument](#risk-argument) |
| [Code audit](code-audit.md) — the `security` and `dependencies` lenses | A general standard; each finding carries its risk argument |

The `prototype` row wins over the phase rows. At the stages `working` and `production` the phase rows apply, and [Needed now](delivery-focus.md#needed-now) decides which further security work enters the task.

## Secure-coding baseline

Code follows these at every stage. Each costs nothing when the code is written and restricts no function:

- input reaches a query as a parameter, never spliced into the query text;
- a command runs as an argument list; input never enters a shell string;
- no secret in the source, in a configuration file the vcs tracks, or in a log ([Proportionate Caution → Secrets](proportionate-caution.md#secrets));
- the platform's safe defaults stay on — certificate checks, template escaping, the framework's CSRF protection;
- no evaluation of input as code, and no deserialization of input into arbitrary types;
- cryptography, random tokens and password hashing come from the platform's library, never written by hand.

A function the request names that a safe default blocks — a call to an internal endpoint with a self-signed certificate — is built by [Protect, never restrict](#protect-never-restrict): the agent keeps the protection another way that lets the function run — that endpoint's certificate trusted for that call alone; with no such way, the developer decides; the default is never switched off for the rest of the code.

Not baseline — work the [phase table](#security-work-by-phase-and-stage) and [Needed now](delivery-focus.md#needed-now) decide: threat models, rate limits, abuse handling, audit logs, authentication or authorization layers the spec does not define.

## Risk argument

Every security finding — of the review's Security check, of a security class type (`vuln:*`, `missing-authz`), or of the audit's `security` or `dependencies` lens — carries these fields:

| Field | Holds |
|-------|-------|
| `actor` | Who attacks: a person outside the team — a user, a visitor, the author of a file, a message or a response the product takes in. The developer, the team, and the machines and CI they control are not actors |
| `entry` | Where the actor's input enters the code — `file:line` |
| `chain` | The `file:line` steps from the entry to the harm, with no check on the way that stops it |
| `precondition` | What must hold for the attack besides the exposure — a configuration, a user's action; `none` when nothing must. A deployment others reach is `exposure: external`, never a precondition |
| `exposure` | `external` — the actor reaches the entry only through users outside the team or a deployment others reach; `local` — the actor reaches it without either, at the current stage: a file, a message or a response from outside the team that the product takes in now |
| `impact` | What the actor gains — data read, data changed, code run, a service stopped; one line |

A field is filled from the code or the environment as they are now. A field filled with a supposition — "an attacker could", "if someone later" — is not filled. On a concept, spec or plan, `entry` and `chain` name the document's sections whose decision lets the actor reach the harm — a contract, an access rule, a data flow — and the risk is computed as for the code that follows the document.

A known vulnerability of a dependency: the `chain` starts at the project's call of the vulnerable function with actor-controlled input; an advisory id or a version number alone is not a chain.

`secret-exposure` carries no risk argument: a real secret in a place where secrets never live is the harm itself ([Proportionate Caution → Secrets](proportionate-caution.md#secrets)).

## Risk — computed, not judged

The reviewer or the auditor fills the fields; the risk follows from them, never from a judgement:

| Risk | True when | In review | In a code audit |
|------|-----------|-----------|-----------------|
| `real` | Every field is filled, and the actor reaches the entry at the current stage — `exposure: local`, or `external` while the stage written in the task file is `production` | `blocking` | offered for an immediate fix |
| `deferred` | Every field is filled, and `exposure: external` while the stage written in the task file is not `production` | `deferrable` — a todo of flavor `deferred` with the trigger "before the first external exposure" | a backlog item of the audit plan with that trigger |
| `hypothetical` | A field cannot be filled — no actor, an entry that takes no actor-controlled input, a chain a check stops, a precondition that does not hold, no path at all | `advisory` — a warning with its recommendation and the field it lacks; no round, no todo | a recommendation; no blocker, no immediate fix |

`secret-exposure` of a real secret is `blocking` at every stage, the stage `prototype` included. While the stage is `prototype`, every other security finding counts as `hypothetical`, whatever its fields — in review, where the Security check is not opened, and in a code audit alike ([Security work by phase and stage](#security-work-by-phase-and-stage)). No `Stage` row in the task file → infer and write it now ([Project stage](delivery-focus.md#project-stage)); until it is written, the risk is computed as at `production`. An audit outside a task reads the stage the same way from the project's facts.

**A real-risk security finding** is a security finding of risk `real`, or a `secret-exposure`. Only it keeps the privileges of a `must` finding in review: reported outside the diff and outside the level's checks, it raises the level, never reaches a `confirm` round, never takes the `contested` route, and blocks without a recurrence limit ([Review Convergence](review-convergence.md#finding-materiality--computed-not-judged)). A `deferred` or `hypothetical` finding has none of them. At the stage `prototype` only a `secret-exposure` is one.

**A `deferred` finding.** The todo is always filed: the at-capture YAGNI-gate does not drop it, and the review report names it. Each such deferral appends a `hardening` row (`ref` = the finding identity) to the [proportionality block](proportionate-caution.md#proportionality-block): `build` makes the finding `blocking` again, `decline` keeps the todo. When the stage becomes `production`, the record fires and the finding is `blocking` in that task's review ([The first external exposure](delivery-focus.md#project-stage)).

## Boundaries

- The stage and the risk decide which security findings block. They never change the handling of a real secret and its input route ([Proportionate Caution](proportionate-caution.md#secrets)), the hard floor of the agent's own actions, [recovery points](recovery-points.md), [access channels](access-channels.md), a project rule, a gate criterion or a developer checkpoint.
- A `deferred` or `hypothetical` finding is never silent: the report names it with its trigger or with the field it lacks, or the stage `prototype`.
