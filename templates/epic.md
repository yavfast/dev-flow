# Epic: {Feature Name}  {#E_XXX}

> **Code:** E_XXX
> **Status:** draft
> **Created:** YYYY-MM-DD
> **Updated:** YYYY-MM-DD
> **Author:** {author}
>
> {Lead summary, within `lead_lines` (see `references/docs-scaling.md`, dev-flow skill): the feature and its business goal, who reads this epic in which situation, which concepts compose it. The header alone must suffice to decide "read on or not".}

## Stakeholders

| Role | Who | Interest |
|------|-----|----------|
| {Owner / Requester / Consumer / ...} | {name or team} | {what they care about} |

## Concepts

| Order | Code | Name | Status | Description |
|-------|------|------|--------|-------------|
| 1 | C_XXX | {Name} | draft | {description} |
| 2 | C_YYY | {Name} | draft | {description} |
| 3 | C_ZZZ | {Name} | draft | {description} |

## Concept Dependencies

    C_XXX ──→ C_YYY ──→ C_ZZZ

## Implementation Order

1. C_XXX — {why first}
2. C_YYY — {depends on what}
3. C_ZZZ — {can start after what}

## Acceptance Criteria

Epic-level criteria (beyond individual concept success):

- [ ] {Cross-concept integration works end-to-end}
- [ ] {Performance / scalability target met}
- [ ] {User-facing workflow is complete}
- [ ] {Documentation updated}

## Risks & Mitigations

| # | Risk | Impact | Probability | Mitigation |
|---|------|--------|-------------|------------|
| 1 | {risk description} | high / medium / low | high / medium / low | {mitigation strategy} |
| 2 | {risk description} | {impact} | {probability} | {mitigation} |

## Success Criteria

- All concepts have `Status: active` and corresponding specs/plans.
- Integration points between concepts are tested.
- No cross-concept conflicts remain.
- All epic-level acceptance criteria are met.

## Changelog

<!-- Optional when vcs is the primary history medium — see the History policy in `references/docs-scaling.md` (dev-flow skill).
     Entry classes only: incident / ambiguous-decision / structural-event; an event outside these
     classes is not written (TRIVIAL_ENTRY refusal — progress lives in the concept table and criteria).
     One entry = one logical line. -->

| Date | Change |
|------|--------|
| YYYY-MM-DD | Initial version |
