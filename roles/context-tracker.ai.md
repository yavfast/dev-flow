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

  workflow:
    read_mode:
      - "Read .dev_flow/active_context.md fully"
      - "Return structured summary: current item, task, next step, blockers, relevant files"
      - "Warn if context is stale (>7 days)"
    write_mode:
      - "Load existing context (or scaffold from template)"
      - "Apply updates: check off completed steps, set next step, update status"
      - "Append one-line entry to Recent Changes"
      - "Set Last updated to current datetime"
      - "Write back to .dev_flow/active_context.md"
}
```
