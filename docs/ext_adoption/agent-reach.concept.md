# Agent Reach — Source Analysis

> **Advisory document.** A concept-altitude portrait of an external repository, produced by `/dev-flow adopt`. It carries no traceable ID, passes no validation gate, and authorizes nothing. Its consumer is [agent-reach.md](./agent-reach.md), the adoption document for this project.

| Field | Value |
|-------|-------|
| **Source** | [`ext_repos/agent-reach/`](../../ext_repos/agent-reach) |
| **Upstream** | https://github.com/Panniantong/agent-reach |
| **Analyzed at** | `93ae1d18c37b707dec053c7c4f9d91cd8ef8943d` (2026-08-12) |
| **Analysis date** | 2026-08-15 |
| **Acquisition** | Cloned by this run |
| **Scope** | Whole repository (120 files, ~6.2 kLOC Python + agent-facing markdown) |
| **License** | MIT |

## What the project is

Agent Reach gives an AI agent read and search access to fifteen internet platforms — Twitter/X, Reddit, Facebook, Instagram, YouTube, Bilibili, XiaoHongShu, GitHub, LinkedIn, V2EX, Xueqiu, Xiaoyuzhou Podcast, RSS, web pages, and web search. It is a Python CLI plus an agent skill, distributed as a git repository and installed by pasting one sentence to an agent.

The project's own summary of its position is the sharpest statement of what it is: *"当下最稳的接入方式，替你选好、装好、体检好"* — it picks the most reliable access route available today, installs it, and health-checks it, so the user never tracks which route currently works.

## Concepts

### 1. Capability layer, not a wrapper

The project sits one level above every concrete implementation and owns exactly four responsibilities: **selection, installation, health-checking, routing**. It does not own the reading itself. After installation the agent calls the upstream tool directly — `yt-dlp`, `gh`, `opencli`, `curl` against Jina Reader — with no interposed API.

*Why it exists:* a wrapper would have to track every upstream tool's interface changes and would become the bottleneck the moment an upstream added a capability the wrapper had not exposed. By refusing to be in the data path, the project keeps its own surface small enough to stay current across an upstream ecosystem that churns hard. `CLAUDE.md` encodes this as a rule — *"Agent Reach is a glue layer — only route and call, don't reimagine"* — alongside a prohibition on ever modifying an upstream project's source.

### 2. A capability is an ordered list of interchangeable backends

Each platform is one file declaring `backends` as an ordered candidate list: index 0 is preferred, the rest are fallbacks. Twitter reads `twitter-cli ▸ OpenCLI ▸ bird CLI (legacy)`; Bilibili reads `bili-cli ▸ OpenCLI ▸ search API`.

*Why it exists:* upstream access routes die on the platforms' schedule, not the project's. Making the route a list turns "our Bilibili support broke" from a code change into a **reordering** — the base class states this explicitly: *"Switching backends for a platform means reordering this list (or a user override) — not rewriting code."* A user can force a specific backend through `<channel>_backend` config or a `<CHANNEL>_BACKEND` environment variable; an unrecognized override is ignored rather than honored, so a stale preference can never hide a working backend.

### 3. Availability is proved by execution, never by presence

`agent_reach/probe.py` exists to defeat one specific illusion: `shutil.which()` finding a command is not evidence the command runs. The probe executes a side-effect-free version command and classifies the outcome into `ok` / `missing` / `broken` / `timeout` / `error`, where **broken** names a real and common failure — a stale virtualenv shim left behind by a system Python upgrade, which `which()` resolves happily and `exec` then fails on.

*Why it exists:* the project's whole value is telling the user what actually works right now. An existence check would let it report health it has not observed. The classification is also load-bearing for retry policy: `missing` and `broken` are terminal and return immediately, while `timeout` and `error` may be transient and are worth a second attempt.

### 4. Diagnosis carries a prescription

`agent-reach doctor` reports every channel's status, which backend is serving it **at this moment**, and — for anything not working — the command that fixes it. `probe.reinstall_hint()` returns the two concrete reinstall invocations rather than a description of the problem. The report is available as `--json` so an agent consumes it structurally, and it degrades per item: a channel whose check raises is reported as `error` and the rest of the report survives, because *"a single misbehaving channel must never take the whole report down"*.

The report is also an output boundary — every message passes through credential scrubbing before rendering, since upstream probe output may echo a configured URL containing a token.

*Why it exists:* a diagnostic that names a problem without naming its fix moves work to the reader. With an agent as the reader, the prescription is directly executable, which closes the loop without a human in it.

### 5. Capabilities are tiered by setup cost, and only the free tier is on by default

Every channel declares a `tier`: 0 = works immediately, 1 = needs a free key or login, 2 = needs real setup. Installation activates the tier-0 set and nothing else; the agent then presents the optional channels as a menu and installs only what the user names. `doctor` groups its report by the same tiers and summarizes unactivated optional channels in a single line rather than listing each as a failure.

*Why it exists:* it separates "not installed" from "broken", which are the same red mark in a naive report. A user who never wanted Instagram should not see Instagram as a problem.

### 6. Safe by default, with an explicit consent flag for mutation

`agent-reach install` performs a read-only environment check and lists what is missing. It touches the machine only under an explicit `--system` flag; `--dry-run` previews what `--system` would do; `--safe` is kept as a compatibility alias for the default. `uninstall` is symmetrically complete — config, tokens, agent skill files, MCP registrations — and also has `--dry-run` plus a `--keep-config` variant for reinstalls.

*Why it exists:* the command is executed by an agent on the user's behalf, so the default has to be the one that is safe to run without asking. Consent is a flag, not a prompt, because a prompt cannot be granted before the agent runs.

### 7. Distribution is a URL pasted to an agent

Installation is one sentence — *"帮我安装 Agent Reach: <raw URL to docs/install.md>"* — and update is the same shape against `docs/update.md`. Both documents are split into a short **For Humans** section (the sentence to copy) and a long **For AI Agents** section that is a procedure: goal, explicit boundaries, numbered steps, verification.

The boundaries section is the notable part. It is written as prohibitions the executing agent must respect: no `sudo` without explicit approval, no modification of system files outside the project's own directory, no installation of packages not listed in the guide, no disabling of security settings, no file creation inside the user's workspace, and — when elevated permission is genuinely needed — tell the user and let them decide.

*Why it exists:* the installer's user is an agent with shell access and a bias toward completing the task. Handing it a procedure with a fence around it is more reliable than handing it a package and hoping.

### 8. The skill is a thin router over on-demand references

`agent_reach/skill/SKILL.md` is roughly 150 lines and contains only: frontmatter with an aggressive `MUST USE` trigger description and an explicit `NOT for` list; five standing rules for the whole session; a routing table mapping user intent to one of seven category files; a handful of zero-config commands; and pointers. Everything specific — per-backend command groups, caveats, retry chains — lives in `references/{search,social,career,dev,web,video,finance}.md` and is read only when the routing table sends the agent there.

The standing rules are behavioral, not informational: health-check before acting on a multi-backend platform, announce which platform and backend you are using before starting, follow the reference's retry chain on failure rather than guessing a command, combine platforms for broad research, and check for updates after a substantial task.

*Why it exists:* the always-loaded part costs context on every session regardless of whether the skill is used. Keeping it to triggers, rules, and routes puts the cost where the value is.

### 9. A hard workspace boundary

Stated identically in `SKILL.md`, `install.md`, and `update.md`: never create files, clone repositories, or run commands inside the agent's workspace. Durable state goes to `~/.agent-reach/` (config at mode 600), upstream tool checkouts to `~/.agent-reach/tools/`, transient output to `/tmp/`, the skill to the agent's own skills directory. The stated reason is cumulative: workspace pollution *"can break their agent over time"*.

### 10. The tool declines authority it could technically take

The most unusual concept in the repository. Agent Reach can read browser cookies, and for some platforms it deliberately does not.

For XiaoHongShu it will not perform the login and will not read browser cookies; OpenCLI may use only a Chrome session the user already has and controls, and `configure xhs-cookies` explicitly does not inject anything into OpenCLI or Chrome. For Twitter, `doctor` refuses to run the upstream `twitter status` at all — because that command silently falls back to reading browser cookies when credentials are missing or invalid, and doctor cannot disable the fallback, so running it would violate the Cookie-Editor-only policy. Saved Twitter credentials serve only doctor's own completeness check; the agent must set them explicitly in a child process environment before calling the upstream tool, and `os.environ` is never mutated.

This forces a distinction the report has to carry: `active_backend: null` means *doctor deliberately did not probe*, not *no backend exists* — and `SKILL.md` tells the agent exactly that, so the null is not read as absence.

*Why it exists:* the project's trust proposition is that credentials stay local and under user control. A capability that would break that proposition is refused even when it would improve the product, and the refusal is made visible rather than silent.

### 11. Route churn is recorded with its evidence and its date

Backend changes are documented as measured events, not as version bumps: *"yt-dlp 被 B站风控 412 封死（2026-06 实测），bili-cli 无登录可搜可读"*, and in the feature table, *"2026-06 实例：yt-dlp 被 B站风控封死 → 已切换 bili-cli，用户零操作"*. Retirement appears in the user-facing selection table with its reason, not only in the changelog. The changelog goes further and keeps a removed channel as struck-through text with the upstream issue link and a stated intent to re-add when upstream recovers.

*Why it exists:* the selection table is the project's central claim, so each row has to carry the evidence for its own currency. A reader can tell a deliberate current choice from an unreviewed default.

### 12. The project watches itself on the user's behalf

`agent-reach watch` is a combined health-and-version check designed for scheduled execution, and `install.md` closes by offering to create a daily cron job with an explicit contract: silent when everything is fine, report only on a problem or a new version. `SKILL.md` adds an in-session version rule — after a substantial task, check for an update, and if one exists append **one line** to the wrap-up with the update sentence; never interrupt the running task, and never mention the same version twice.

*Why it exists:* the project's premise is that access routes decay. A user who has to remember to check is back to the problem the project claims to solve — but a tool that nags about it becomes the problem instead, so the interruption budget is spent deliberately.

### 13. Announce the route before using it

Standing rule 2: say *"using agent-reach, platform X via backend Y"* before starting. A trivial-looking convention that makes the routing layer's decision auditable at the moment it is acted on, rather than reconstructable afterwards.

### 14. Multiple reader-shaped entry points

The repository carries `README.md` (Chinese), `docs/README_{en,ja,ko}.md`, an `llms.txt` that compresses the whole project into about thirty lines for machine ingestion, `CLAUDE.md` as the contributor-agent's operating rules, `SKILL.md` and `SKILL_en.md` for the consuming agent, and `install.md` / `update.md` for the installing agent. Six distinct readers, six documents, minimal overlap.

## Design philosophy

Three commitments run through everything above.

**The layer earns its place by what it refuses to do.** No wrapper, no reimplementation, no modification of upstream, no data path. What remains — choose, install, check, route — is small enough to maintain against an ecosystem that breaks constantly.

**Claims are backed by observation.** Execution over presence in `probe.py`, `active_backend` reporting what is serving right now rather than what is configured, a selection table where each row states the measurement behind it, and an explicit null for "deliberately not checked".

**The agent is the user, and an agent needs fences.** Documents addressed to agents rather than to humans, procedures with prohibitions attached, safe defaults with explicit consent flags, and a declared workspace boundary — all of it because the executing reader is capable, literal, and biased toward finishing.

## What is unusual

- **Refusing a technically available capability on trust grounds** (concept 10), and then making the refusal legible in the machine-readable output so it is not mistaken for a failure.
- **Reordering a list as the primary maintenance operation.** Most projects would express "we switched Bilibili backends" as a code change; here it is data.
- **A skill whose standing rules govern conduct rather than content** — announce your route, verify before acting, follow the retry chain, never guess a command.
- **Recording a decommissioned route with its failure evidence, in the user-facing table** rather than burying it in history.
- **An interruption budget stated in the skill itself** — one line, at wrap-up, never twice for the same version.

## Where this project is weak

Recorded because the adoption document must not borrow a weakness alongside a strength.

- **`cli.py` is 2350 lines** and holds the installer, the configurator, the uninstaller, the update checker, and every per-channel install routine. The channel abstraction is clean; the command layer is not, and it is the file most likely to have to change when a channel changes.
- **The design language is split.** Docstrings and structural comments are English, user-facing strings and several inline comments are Chinese, reference documents are Chinese while the skill that points at them is English. Coherent for the author, a barrier for a contributor.
- **Version identity is duplicated in three places** — `pyproject.toml`, `__init__.py`, `tests/test_cli.py` — with a `CLAUDE.md` rule instructing humans to keep them in sync, where a single source would remove the rule.

## Changelog

| Date | Commit | Change |
|------|--------|--------|
| 2026-08-15 | `93ae1d1` | Initial analysis (`CREATE`) — 14 concepts extracted |
