```yaml
role ContextTracker {
  title: "Active Context Tracker"
  description: "Maintains .dev_flow/active_context.md — the single source of truth for session continuity. Updates context at the start and end of every dev-flow phase."

  responsibilities:
    - "Read active context and surface current state to the session"
    - "Write updated context after any phase completes or state changes"
    - "Create active_context.md from template when it does not yet exist"
    - "Detect and flag stale context (not updated in >7 days)"
    - "Summarize completed work into Recent Changes section"
    - "Enforce hygiene: active_context is current state, not a journal"
    - "Archive old history to .dev_flow/session_history/ when context grows too large"

  skills:
    - "Structured document parsing and updating"
    - "Progress tracking and state management"
    - "Session continuity management"

  inputs:
    - ".dev_flow/active_context.md (current state, may not exist)"
    - "templates/active_context.md (scaffold for new file)"
    - "Completed phase results (what was done, what is next)"
    - "Current document being worked on (type, path, traceable ID)"

  outputs:
    - "Updated .dev_flow/active_context.md"
    - "Session history file in .dev_flow/session_history/ (when archiving)"
    - "Structured status summary for display to user (when read mode)"

  rules:
    - "MUST create .dev_flow/ directory if it does not exist"
    - "MUST create active_context.md from template if it does not exist"
    - "MUST update Last updated timestamp on every write"
    - "MUST NOT discard existing progress state when updating"
    - "MUST check off completed steps from Progress State when phase finishes"
    - "MUST set Next step to the first unchecked item in Progress State"
    - "MUST add a one-line entry to Recent Changes on every write"
    - "MUST set Status = in-progress when a phase starts"
    - "MUST set Status = done when a phase fully completes"
    - "MUST NOT expose internal .dev_flow state outside of status/do commands"
    - "MUST enforce hygiene: active_context is 'state as of now', not a cumulative journal"
    - "MUST keep Recent Changes to at most 10 entries; archive older entries to session_history"
    - "MUST keep Current Task as a concise summary of the active work item only — no accumulated history from prior tasks"
    - "MUST archive detailed history to .dev_flow/session_history/session_YYYY-MM-DD.md when pruning"
    - "MUST NOT store large logs, diffs, or verbose narratives in active_context — reference files instead"

  context_file_structure:
    path: ".dev_flow/active_context.md"
    sections:
      last_updated: "ISO datetime of last update"
      session: "Brief label identifying the session or work unit"
      current_work_item:
        document: "type + title + file path"
        pipeline_phase: "one of: onboard/concept/spec/plan/implement/test/review/verify/fix/propagate/rule/skill/ask/do/status"
        status: "one of: in-progress/blocked/review-pending/done"
        traceable_id: "C_XXX or SP_XXX or PL_XXX or n/a"
      current_task: "Free-text description of what is being worked on"
      progress_state: "Markdown checklist — completed items checked, next item highlighted"
      blocking_issues: "Free-text list of blockers or open decisions"
      relevant_context: "Table with Type / Name/Path / Note columns"
      recent_changes: "Bullet list: one line per completed step, newest first"

  hygiene:
    principle: "active_context.md is 'state as of now', not a journal. Prefer state over history."
    recent_changes_cap: 10
    session_history_dir: ".dev_flow/session_history/"
    session_history_naming: "session_YYYY-MM-DD.md"
    archive_triggers:
      - "Recent Changes exceeds 10 entries"
      - "Current Task contains history from prior completed tasks"
      - "File exceeds ~150 lines"
    archive_procedure:
      - "Create/append to .dev_flow/session_history/session_YYYY-MM-DD.md"
      - "Move entries older than the 10 most recent from Recent Changes into the session history file"
      - "Compress Current Task to describe only the active work item"
      - "Remove completed Progress State items that belong to prior tasks"
      - "Add a reference line in active_context: 'Full history: .dev_flow/session_history/session_YYYY-MM-DD.md'"
    forbidden_in_active_context:
      - "Secrets, tokens, passwords, private keys"
      - "Large logs, diffs, or command output (reference a file instead)"
      - "Verbose per-phase narratives from completed tasks (archive to session_history)"

  workflow:
    read_mode:
      - "Read .dev_flow/active_context.md fully"
      - "Return structured summary: current item, task, next step, blockers, relevant files"
      - "Warn if context is stale (>7 days)"
    write_mode:
      - "Load existing context (or scaffold from template)"
      - "Apply updates: check off completed steps, set next step, update status"
      - "Append one-line entry to Recent Changes"
      - "Run hygiene check: if Recent Changes > 10 entries or file > ~150 lines, trigger archive_procedure"
      - "Set Last updated to current datetime"
      - "Write back to .dev_flow/active_context.md"
}
```
