# Supervising a Delegate — Watch Levels, Progress Record, Watcher, Stall Ladder

Sub-procedure of every phase that starts a subagent; not a pipeline stage, no command. Which vehicle to use — continue, fork, fresh — and how to talk to a running delegate is prose in [Delegation for Focus → Running a delegate](delegation.md#running-a-delegate). This file holds what is fixed: the watch level a run gets, the lines its brief carries, the duties of a delegate, the progress record and inbox it keeps, the watcher, and what an initiator does when a run falls silent. The same contract holds at every level of nesting: a delegate that starts helpers is their initiator.

## Contents

- [Watch levels](#watch-levels) — `none` / `record` / `watched`, chosen by the run's type and complexity
- [The brief](#the-brief) — the `Run:` / `Progress:` / `Inbox:` / `Duties:` lines, the `## Run bounds` block, the files made before the spawn
- [Delegate duties](#delegate-duties) — what a delegate does at each level
- [Run id](#run-id) — the name of a run in the workspace; a subtree is one prefix
- [Progress record](#progress-record) — the event grammar, with an example
- [Inbox](#inbox) — instructions down to a running delegate
- [Silence threshold](#silence-threshold) — when a run counts as silent
- [The watcher](#the-watcher) — its events, how to run it, a bash example
- [Handling a watcher event](#handling-a-watcher-event) — `done`, `note`, `stall`, `expired`, `error`, a finish the harness reports
- [The stall ladder](#the-stall-ladder) — from a look to one re-delegation; which process may be stopped
- [Nesting](#nesting) — every initiator supervises its own children; when an ancestor steps in

## Watch levels

The initiator sets the level when it starts the run:

| Level | What runs | Typical runs |
|-------|-----------|--------------|
| `none` | the run bounds, when the run executes commands | a search, a read, a count, `ask`, a review of a diff, a fork drafting text |
| `record` | + the progress record; the initiator reads it when the run is late or when it checks in for its own reasons | a test pass, a build, a one-command verify, a subtask of a few steps |
| `watched` | + a watcher and the stall ladder | a long suite, a live / device / integration verify, research over the network, a prototype, a multi-step subtask with long commands or helpers of its own |

A run that only searches, reads or counts, and a delegate in a read-only phase (`ask`), stays at `none`, with its helpers. Otherwise: one level up when the run starts helpers, when the initiator has nothing else to do while it waits, or when the same kind of run has stalled before in this project (a lesson recorded by [Experience Capture](experience-capture.md)); one level down when the run is expected to take a couple of minutes. A delegate that starts helpers runs at least at `record`, so its subtree stays visible from above; a delegate at `none` that starts a helper at `record` or above gives it a top-level run id. Nesting depth is not limited.

## The brief

At `record` and `watched` the brief carries its `Skill root:` line — even when the subagent gets no role — and these lines after `Role style:` (or after `Read first:`, or after `Skill root:`, whichever comes last):

```
Run: <run id> · Watch: <record|watched> · Expected: <N>m
Progress: /tmp/{project-slug}/agents/<run id>.progress
Inbox: /tmp/{project-slug}/agents/<run id>.inbox
Duties: before your first action read references/supervision.md § Delegate duties, § Progress record and § Inbox; keep the duties of your watch level
```

`Expected` is the initiator's estimate of the whole run in minutes. At `none` there are no such lines.

A run that executes commands — shell commands beyond a read-only search — gets this block at any level, with `<N>` the same estimate:

```
## Run bounds
- Every command non-interactive — no editor, pager or prompt (GIT_EDITOR=true,
  --no-pager, CI=1, no watch mode) — and under a timeout of its own fitted to
  it (`timeout <s> <cmd>`): a harness call timeout only moves it to the
  background.
- Servers and watch-mode tools start in the background with a readiness check;
  stop them before you end.
- A command the harness moves to the background still runs: wait for it (with
  a progress record: a `run` line if it had none, then a `wait` line; read the
  inbox between checks) or stop it; never end with it running.
- A command asks for input → stop it; a secret or an approval goes to me.
- The same failing action at most twice without a new hypothesis → end
  blocked, with what you tried.
- Past <N> minutes with no end in sight → wrap up with a partial report.
```

**Before the spawn**, at `record` and `watched`, the initiator creates the workspace, checks it, and reserves the run id — the empty record — so that two runs cannot take the same id:

```bash
d=/tmp/{project-slug}/agents; mkdir -p "$d" && [ -O "$d" ] && [ -O "${d%/*}" ] && chmod 700 "$d" && chmod go-w "${d%/*}" || echo unsafe   # unsafe → stop here
(set -C; : > "$d/<run id>.progress") 2>/dev/null || echo taken   # taken → the next suffix (Run id)
: > "$d/<run id>.inbox"                                           # watched only
```

The inbox is an instruction channel and the watcher a script that runs, so the directory is the developer's own, as for a one-time wrapper in the workspace ([Proportionate Caution → Secret input](proportionate-caution.md#secret-input)): before it writes an inbox line, copies the watcher or runs it, the initiator checks again — `[ -O "$d" ] && [ -O "${d%/*}" ] && [ "$(stat -c %a "$d")" = 700 ] && [ -z "$(find "${d%/*}" -maxdepth 0 -perm /022)" ]`: both directories its own, the workspace closed to others, its parent writable by no one else. The workspace cannot be written, or a check fails (`unsafe`) → run the delegate at `none` and say so in the brief. The harness cannot run anything in the background → no watcher; the brief says so, and the run bounds carry the whole protection.

## Delegate duties

| Duty | `none` | `record` | `watched` |
|------|--------|----------|-----------|
| keep the run bounds (when it executes commands) | yes | yes | yes |
| `plan` first, `step` per step, `run` before a command expected to take a minute or more, `spawn` / `wait` per helper, `end` last | — | yes | yes |
| `note` on a wrong premise or a conflict that blocks the rest, then go on with what it does not block; the question also goes into the report's Open items (a focus report: into its verdict) | — | yes | yes |
| read the inbox at each step boundary and between checks of a command it waits on, and obey it | — | when the file is not empty | yes |
| supervise the helpers it starts, by this file | yes | yes | yes |
| name the record in the report as `Progress: <path>` (a full report: under Materials) | — | yes | yes |

## Run id

- Top level: `<name>_<YYYYMMDD>_<HHMMSS>` — the spawn time, per the workspace naming ([Resource Cache → Temporary Workspace](cache.md#temporary-workspace-tmp-discipline)).
- Below: `<parent run id>.<name>`.
- `<name>`: 1–3 kebab-case words, `[a-z0-9-]`, at most 24 characters.
- Unique: the initiator reserves `<run id>.progress` before the spawn ([The brief](#the-brief)); when it is taken, it appends `-2`, `-3`, … to `<name>` and reserves again.
- Assigned by the initiator for `record` and `watched` only; a helper at `none` has none. A continued delegate keeps its run id and writes a new `plan`; a fork gets its own.
- A run id that breaks these rules is used as given and named in the initiator's report; nothing renames files.

The files of a run are `/tmp/{project-slug}/agents/<run id>.progress` and `<run id>.inbox` beside it. A run's subtree is its own record and every record whose name starts with `<run id>.`. No document or task file links these files.

## Progress record

One line per event, appended, never rewritten:

```
<time> <event> [<fields>] -- <text>
```

`<time>` is ISO 8601 with offset and seconds (`date -Iseconds`). `<text>` is one line of at most 160 characters — no secrets, no raw output.

| Event | Fields | Sets the threshold | Meaning |
|-------|--------|--------------------|---------|
| `plan` | `steps=<N> exp=<M>m` | no | first line of a run; again on a continuation or a scope change |
| `step` | `<k>/<N>` | no | step `k` begins |
| `run` | `exp=<M>m` | yes | a command expected to take a minute or more starts; `<text>` is the command, its leading words verbatim, shortened only at the end |
| `spawn` | `<child run id> level=<level> exp=<M>m` | no | a helper started; a helper at `none` is named by its `<name>` |
| `wait` | `<child run id or what> exp=<M>m` | yes | the run waits and does nothing else until it ends |
| `note` | — | no | a wrong premise or a conflict that blocks the rest ([Delegate duties](#delegate-duties)) |
| `end` | `done` \| `blocked` \| `stopped` | — | the last line; the report follows. The initiator writes it for a run it stopped ([The stall ladder](#the-stall-ladder)) |

```
2026-11-16T19:40:05+02:00 plan steps=3 exp=20m -- run the auth suite, triage failures
2026-11-16T19:40:30+02:00 step 1/3 -- build
2026-11-16T19:40:31+02:00 run exp=5m -- ./gradlew assembleDebug
2026-11-16T19:44:02+02:00 step 2/3 -- test suite
2026-11-16T19:44:03+02:00 run exp=12m -- ./gradlew testDebugUnitTest
2026-11-16T19:52:40+02:00 step 3/3 -- triage
2026-11-16T19:52:41+02:00 spawn tests_20261116_194005.triage level=record exp=5m -- 3 failures
2026-11-16T19:52:42+02:00 wait tests_20261116_194005.triage exp=5m -- triage verdict
2026-11-16T19:56:10+02:00 end done -- 3 failures, 2 flaky; report follows
```

A line that does not parse — a time that is not ISO 8601 with its offset, does not convert, or is more than a minute later than the record's last modification (a file's time may lag the clock by a second); an unknown event; a `run` or `wait` without `exp` — is skipped by every reader, the watcher's scan for notes and children included, and the reader takes the last line that parses; the record's modification time still shows the run is active. An `exp` above the brief's `Expected` is allowed. Separators are ASCII so a shell loop parses them.

## Inbox

```
<time> <instruction> from=<run id|main> -- <text>
```

| Instruction | The delegate, when it next reads the inbox |
|-------------|------------------------------------------|
| `wrap-up` | starts no new step, writes the report with what is done, ends `end stopped -- wrapped up on request` |
| `stop` | ends at once: `end stopped`, a report of one line per finished step |
| `info` | takes the fact into account |
| `scope` | re-plans: a new `plan` |

The direct initiator writes it; an ancestor only as [Nesting](#nesting) allows. At `record` the inbox stays empty until one of them writes to it. The delegate reads it after the same check as the initiator ([The brief](#the-brief)); a failed check → it leaves the inbox unread and writes a `note`. The delegate acts on the lines it has not acted on before; a line that does not parse — its time by the record's rule, or an unknown instruction — is skipped. Where the harness delivers a message to a running agent, the initiator may use it instead — it too arrives only when the delegate next takes its turn, so neither reaches a delegate blocked inside a command. Claude Code moves a foreground command to the background when the call's own timeout runs out, at the latest after about three minutes, and gives the delegate its turn back; the command runs on, and the delegate waits for it with checks, reading the inbox between them ([The brief](#the-brief), Run bounds).

## Silence threshold

A run's **child** is a run whose id starts with the run's own id and a dot, and which is not at `none`. Take a run's last event `e` at time `t(e)`, and `last_seen` = the latest of `t(e)`, the record's modification time, and the harness's last activity of the run, where the harness exposes one:

| `e` | The run is silent when |
|-----|------------------------|
| `run`; `wait` on something other than a child | `now − t(e)` > max(1.5 × exp, exp + 2) minutes |
| `wait` on a child that has not ended | the child is silent — the run is judged by the child's record, or by the last row when the child has none |
| `wait` on a child that has ended | as `step`, with the child's `end` in `last_seen`: it is the parent's turn |
| `plan`, `step`, `spawn`, `note` | `now − last_seen` > 10 minutes |
| `end` | never — the run is over; except the watched run's own `end` older than its watcher's floor: a continued run before its new `plan`, judged by the last row from the floor |
| no line that parses yet (the record reserved, empty or unparsable; or missing) — at any depth | `now − last_seen` > 10 minutes, `last_seen` counted from the reservation (a missing record: from the first line that parses in its parent's record and names it, else from the spawn), and the harness does not report the run as starting. A watcher sees a child only through its record or such a line; a child it cannot see is left to its parent's own row |

A `run` or `wait` is measured from the event, not from activity: a delegate inside one long command shows none by design. Silence the initiator has already acted on is not counted twice: a re-armed watcher counts from its floor ([Handling a watcher event](#handling-a-watcher-event)).

## The watcher

For a `watched` run the initiator arms one watcher — a background process with no model — instead of checking in turn after turn. It reads the records of the run's subtree every 30 seconds (and the delegate's activity file, where given), writes exactly one line, and exits:

| Event | When |
|-------|------|
| `done <run id> <outcome>` | the watched run wrote `end`, at or after the floor |
| `note <run id> <time> <line> -- <text>` | a run in the subtree has a `note` after `WATCH_NOTED`; the earliest such note, ordered by time, run id and line number |
| `stall <run id> <minutes>m -- <last line>` | the deepest run of the subtree that has not ended is silent |
| `expired` | the watcher reached its own time limit; the run goes on |
| `error -- <text>` | a setting does not parse; the watcher watched nothing |

One poll with several of them prints the first in this order: `note`, `done`, `stall`.

**Only while the delegate runs.** The initiator arms a watcher, and arms it again, only while the run's delegate runs: the harness, where it reports one, has not reported its finish (in Claude Code, the background agent's notice) since the delegate last started — its spawn or its continuation. Once it has, no watcher is armed for that run until the delegate is continued, whatever event is still unhandled when the notice comes, or comes after it; that event is not acted on — the finish is handled once, and a watcher still running is stopped ([Handling a watcher event](#handling-a-watcher-event)).

A `watched` delegate itself runs in the background — in Claude Code, a background agent — so the initiator is free when the watcher's line comes back; a harness that can only run it in the foreground gets no watcher, as when nothing runs in the background. Copy the example to `/tmp/{project-slug}/agents/watch.sh` once per session and run it through the harness's background facility, so that its one line comes back as a notice — in Claude Code, Bash with `run_in_background`, or `Monitor` with `timeout_ms: 1800000`, its cap. Settings:

- `WATCH_LIMIT`, in whole seconds, defaults to 1740, under that cap, so the watcher ends with `expired` instead of being cut off without a line.
- `WATCH_FROM`, in epoch seconds and not later than now, is the floor: silence before it is not counted.
- `WATCH_NOTED` is the key of the last note the initiator handled — the three fields after `note`: `<run id> <time> <line>`; a time alone covers every note up to that second; unset, every note in the subtree is new.
- A continued delegate keeps its record, so its first watcher gets `WATCH_FROM=$(date +%s)` and `WATCH_NOTED=$(date -Iseconds)`, taken at the continuation: the `end` of the run before is not read as `done`, and its notes are not new.
- A setting that does not parse — `WATCH_LIMIT` or `WATCH_FROM` not a whole number, `WATCH_FROM` later than now, a `WATCH_NOTED` time that is not ISO 8601 with its offset, does not convert or is more than a minute ahead of now, or a line number that is not a whole number — gives `error`; nothing is watched. Empty is the same as unset.
- The optional third argument is a file whose modification time shows the delegate's activity — in Claude Code, its transcript `~/.claude/projects/<project>/<session>/subagents/agent-<agent id>.jsonl`.

The example needs bash, GNU `date` and `stat`, and `awk`; it runs in the C locale, so notes order the same everywhere.

```bash
#!/usr/bin/env bash
# Delegate watcher — references/supervision.md. Prints one line and exits:
#   done <id> <outcome> | note <id> <time> <line> -- <text> | stall <id> <N>m -- <last line> | expired | error -- <text>
# Usage: [WATCH_LIMIT=<s>] [WATCH_FROM=<epoch s>] [WATCH_NOTED=<id time line|time>] watch.sh <agents dir> <run id> [activity file]
export LC_ALL=C
dir=$1 root=$2 act=${3:-} limit=${WATCH_LIMIT:-1740} from=${WATCH_FROM:-0} noted="" armed=$(date +%s)
iso='^[0-9]{4}-[0-9]{2}-[0-9]{2}T[0-9]{2}:[0-9]{2}:[0-9]{2}([+-][0-9]{2}:[0-9]{2}|Z)$' num='^[0-9]{1,10}$'
hasexp=' exp=([0-9]+)m( |$)' isnone=' level=none( |$)'
bad() { echo "error -- $1"; exit 2; }
[[ $limit =~ $num ]] || bad "WATCH_LIMIT is not whole seconds: $limit"
[[ $from =~ $num ]] && (( 10#$from <= armed )) || bad "WATCH_FROM is not epoch seconds up to now: $from"
limit=$(( 10#$limit )) from=$(( 10#$from ))
if [ -n "${WATCH_NOTED:-}" ]; then   # notes up to this key are handled: (time, run id, line)
  read -r a b c <<<"$WATCH_NOTED"; [ -n "$c" ] || { b=$a a=""; }
  [[ $b =~ $iso && ( -z $a || $c =~ $num ) ]] && t=$(date -d "$b" +%s 2>/dev/null) && (( t <= armed + 60 )) ||
    bad "WATCH_NOTED does not parse: $WATCH_NOTED"
  if [ -n "$a" ]; then noted=$(printf '%010d %s %06d' "$t" "$a" "$(( 10#$c ))"); else noted=$(printf '%010d ~' "$t"); fi
fi
parsed() {  # the lines of a record that parse, as "<epoch> <line number> <line>" — the one rule every reader uses
  local m n=0 line ts ev t
  m=$(stat -c %Y "$1" 2>/dev/null) || return 0
  while IFS= read -r line || [ -n "$line" ]; do
    n=$(( n + 1 )); read -r ts ev _ <<<"$line"
    [[ $ts =~ $iso ]] || continue                      # an ISO 8601 time with its offset,
    case $ev in plan|step|spawn|note|end) ;;           # a known event, exp= on run and wait,
      run|wait) [[ ${line%% -- *} =~ $hasexp ]] || continue ;; *) continue ;; esac
    t=$(date -d "$ts" +%s 2>/dev/null) || continue    # a time that converts
    (( t <= m + 60 )) || continue                      # and not later than a minute past the record's last change
    echo "$t $n $line"
  done 2>/dev/null <"$1"
}
newer() { local m; m=$(stat -c %Y "$1" 2>/dev/null) && (( m > since )) && since=$m; }
judge() {  # <run id> <since> <limit> <line shown>: keep the deepest silent run
  local s=$2 dots=${1//[^.]/}
  (( s < from )) && s=$from
  if (( now - s > $3 && ${#dots} > depth )); then depth=${#dots}; stall="stall $1 $(( (now - s) / 60 ))m -- $4"; fi
}
while :; do
  now=$(date +%s); note="" nk="" fin="" stall="" depth=-1
  [ -e "$dir/$root.progress" ] || judge "$root" "$armed" 600 "no record"   # the watched run's record is missing
  for f in "$dir/$root.progress" "$dir/$root".*.progress; do
    [ -e "$f" ] || continue
    id=${f##*/}; id=${id%.progress}; last="" ev="" arg="" since=0 kids="" none=" "
    while read -r t n line; do
      read -r ts e txt <<<"$line"; read -r _ _ a _ <<<"$line"
      last=$line ev=$e arg=$a since=$t
      if [ "$e" = note ]; then   # the earliest note after WATCH_NOTED
        k=$(printf '%010d %s %06d' "$t" "$id" "$n")
        [[ $k > $noted && ( -z $nk || $k < $nk ) ]] && { nk=$k; note="note $id $ts $n $txt"; }
      elif [[ ( $e == spawn || $e == wait ) && $a == "$id".* ]]; then   # a child it names, first time
        if [[ $e == spawn && ${line%% -- *} =~ $isnone ]]; then none+="$a "
        elif [[ $kids != *" $a"$'\n'* ]]; then kids+="$t $a"$'\n'; fi
      fi
    done < <(parsed "$f")
    case $ev in
      "")  last="no record" ;;   # nothing parses yet: from the reservation, the record's modification time below
      end) if [ "$id" != "$root" ] || (( since >= from )); then [ "$id" = "$root" ] && fin="done $root $arg"; continue; fi
           ev="" last="no plan since the continuation" since=0 ;;   # the root's end before the floor: a continued run
    esac
    while read -r ct c; do   # a child it named that has no record: from the first line naming it
      [ -n "$c" ] && [[ $none != *" $c "* ]] && [ ! -e "$dir/$c.progress" ] && judge "$c" "$ct" 600 "no record"
    done <<<"$kids"
    if [[ $ev == wait && $arg == "$id".* && $none != *" $arg "* ]]; then   # a wait on its own child
      [ -e "$dir/$arg.progress" ] || continue            # no record: judged above
      read -r ct _ _ cev _ <<<"$(parsed "$dir/$arg.progress" | tail -n 1)"
      [ "$cev" != end ] && continue                      # a live child: judged by its own record
      ev=step; (( ct > since )) && since=$ct            # it ended: the parent's turn
    fi
    if [[ $ev == run || $ev == wait ]]; then
      [[ ${last%% -- *} =~ $hasexp ]]; exp=$(( 10#${BASH_REMATCH[1]} ))
      lim=$(( exp * 90 > exp * 60 + 120 ? exp * 90 : exp * 60 + 120 ))
    else
      lim=600; newer "$f"
      [ "$id" = "$root" ] && [ -n "$act" ] && [ -e "$act" ] && newer "$act"
    fi
    judge "$id" "$since" "$lim" "$last"
  done
  for line in "$note" "$fin" "$stall"; do [ -n "$line" ] && { echo "$line"; exit; }; done
  (( now - armed > limit )) && { echo expired; exit; }
  sleep 30
done
```

## Handling a watcher event

To **re-arm** is to run the watcher again on the same run, with the `WATCH_FROM` and `WATCH_NOTED` it had — only while its delegate runs ([Only while the delegate runs](#the-watcher)). Two moves change one of them: after the initiator has acted on silence — judged a run "slow but alive", taken ladder step 2 or 3 — `WATCH_FROM=$(date +%s)`, so the silence it has just acted on does not fire again at once (to re-arm **from now**); after it has handled a `note`, `WATCH_NOTED="<run id> <time> <line>"` from that note. A note is not silence: it moves no floor.

- `done` → where the harness reports the finish (in Claude Code, the background agent's notice), wait for that notice — the report comes with it; then read the report: the conclusion, not the dump.
- `note` → read the line; answer through the inbox or a harness message, or let the run go on; re-arm with `WATCH_NOTED` set to its key.
- `stall` → [the stall ladder](#the-stall-ladder).
- `expired` → re-arm.
- `error` → correct the setting it names and arm again.
- The harness reports the run finished → stop its watcher through the harness if one still runs; arm none until the delegate is continued ([Only while the delegate runs](#the-watcher)). The record has no `end` → append `<time> end stopped -- finished without an end line`, then salvage (ladder step 5) before trusting the report.
- The harness reports the delegate stopped with background work of its own still running → a command of the run is still running, whatever the record says. Stop it by the rule of step 2 ([Which process step 2 may stop](#the-stall-ladder)), the leftover command in place of the one step 1 names — or, when its result is still needed, continue the delegate to wait for it; the rule leaves none or several → name the command to the developer instead. Then salvage (ladder step 5) before trusting `done` or the report. No re-arm on the ended run: its old `end` would read as a continuation.

The watcher is the only periodic check: the initiator neither loops over its own turns nor asks a delegate whether it is done.

## The stall ladder

Step 1's verdict picks the first step to take; after it, each step only when the one before it did not resolve the stall:

| Step | Action | Next step when |
|------|--------|----------------|
| 1 Look | the stalled run's last lines, its harness status and activity, the processes of its `run` lines; a noisy look (log tails, a deep subtree) goes to a one-shot diagnostic helper at `none`, which returns the verdict: hung command (which), loop or off track, dead, slow but alive | by the verdict: hung command → 2; loop or off track → 3; dead → 4; slow but alive → none, re-arm from now |
| 2 Hung command | stop the process of the command step 1 named; re-arm from now | the re-armed watcher reports the same run, or no process qualifies |
| 3 Loop or off track | `wrap-up` in the inbox, or a harness message; re-arm from now | the re-armed watcher reports the same run |
| 4 Stop | stop the delegate through the harness, and its watcher; no harness stop → `stop` in the inbox, and the run counts as ended. Append `<time> end stopped -- stopped by <run id\|main>` to its record and to every record of its subtree that has no `end` | — |
| 5 Salvage | collect its record, any report fragment, `git status` and the diff in its scope, its workspace files; record what is done and what is unverified. A stopped subtask's block stays; a new block references it ([status → Collaboration Model](../phases/status.md#collaboration-model-read-first)) | — |
| 6 Re-delegate once | a fresh delegate with the salvage and the cause of the stall in its brief, or finish inline; the same work stalls a second time → stop delegating it and tell the developer the cause | — |

**Which process step 2 may stop.** The match uses the `run` event of the command step 1 named. That command has no `run` line, or its text is empty up to its first quote or shell operator (it begins with one, or there is none) → no process qualifies: an empty text is in every command line. List the processes with their ages and command lines (`LC_ALL=C ps -eo pid,ppid,etimes,args`; a start time is now − `etimes`). A candidate started within a minute of that event, and its command line contains the event's text up to its first quote or shell operator (`'` `"` `$` `` ` `` `|` `&` `;` `<` `>` `(`) — a process's command line shows its arguments with the shell's quotes removed, so `python3 -c 'import time'` is matched by `python3 -c`. Drop every candidate that is an ancestor of another candidate — the shell a harness wraps the command in. Exactly one left, and it is the command step 1 named or the shell wrapping it → stop it and its descendants by process id, deepest first: a script that hands over to another program (`./gradlew` → `java`) or a compound command (`cd x && cmd`) leaves only the wrapper as a candidate, and the hung program is its descendant. Otherwise — none, several, or another command — step 3. Never a pattern kill (`pkill -f`, `killall`): a harness may run the main agent's and every subagent's commands under one parent.

**Who acts.** The initiator, on its own, through step 6; its next message to the developer names the stall and the steps taken.

## Nesting

- Every initiator supervises its own children by this file.
- A parent's `wait` on a child that has not ended is judged by the child — its record, or the no-record row when it has none ([Silence threshold](#silence-threshold)); the watcher reports the deepest silent run.
- A watcher may report a run below the initiator's direct child. The initiator first leaves it to that run's own initiator: it re-arms from now. When the re-armed watcher reports the same run again, the initiator acts on it by the ladder. It writes `from=<its run id or main>` in the inbox of every run it acts on, and an `info` line in its direct child's inbox naming what it did.
- A parent rolls its children's outcomes into its own report: one line per child — run id, outcome, conclusion.
- `status` lists the live runs of the workspace ([status → Present the summary](../phases/status.md#step-4-present-the-summary)).
