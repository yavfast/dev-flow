# End-to-End Example: Rate Limiter

A minimal walkthrough showing the full concept-driven development pipeline.

## Step 1 — Concept (`docs/rate_limiter.concept.md`)

```markdown
# Rate Limiter  {#C_RLM}

> **Code:** C_RLM
> **Status:** active
> **Created:** 2026-03-24
> **Updated:** 2026-03-24
> **Author:** developer
>
> **Depends on:** [C_ACS](./access_control.concept.md)
> **Used by:** —
> **Specification:** [SP_RLM](./rate_limiter.sp.md)
> **Plan:** [rate_limiter.plan.md](./rate_limiter.plan.md)
>
> Prevents agents from overwhelming external APIs by enforcing per-agent call rate limits.

## 1. Philosophy  {#C_RLM_01}

### 1.1. Core Principle  {#C_RLM_01_01}
Each agent has a maximum number of LLM calls per time window.
Exceeding the limit causes the call to be delayed, not rejected.

### 1.2. Design Constraints  {#C_RLM_01_02}
- Limits are per-agent, not global.
- The limiter is transparent — agents do not know they are being throttled.

## 2. Domain Model  {#C_RLM_02}

### 2.1. Key Entities  {#C_RLM_02_01}

    Agent ──has──→ RateBucket
    RateBucket: { capacity, tokens, refill_rate }

### 2.2. Data Flows  {#C_RLM_02_02}

    Agent calls LLM → Limiter checks bucket → tokens available? → proceed / wait

## Changelog

| Date | Change |
|------|--------|
| 2026-03-24 | Initial version |
```

**Gate check:** No conflicts with existing concepts. C_ACS dependency valid. Scope clear. Proceed to spec.

---

## Step 2 — Specification (`docs/rate_limiter.sp.md`)

```markdown
# Rate Limiter — Specification  {#SP_RLM}

> **Code:** SP_RLM
> **Status:** active
> **Created:** 2026-03-24
> **Updated:** 2026-03-24
>
> **Concept:** [C_RLM](./rate_limiter.concept.md)

## 01. Data Structures  {#SP_RLM_01}

### 01_01. RateBucket  {#SP_RLM_01_01}

| Field | Type | Required | Default | Constraints | Description |
|-------|------|----------|---------|-------------|-------------|
| agent_id | string | yes | — | valid agent ID | Owner agent |
| capacity | integer | yes | 10 | 1..1000 | Max tokens |
| tokens | float | yes | capacity | 0..capacity | Current tokens |
| refill_rate | float | yes | 1.0 | 0.1..100.0 | Tokens per second |
| last_refill | timestamp | yes | now | ISO 8601 | Last refill time |

## 02. Contracts  {#SP_RLM_02}

### 02_01. AcquireToken  {#SP_RLM_02_01}

Input: agent_id (string)
Output: wait_seconds (float) — 0 if token acquired immediately, >0 if delayed.

Processing:
    FUNCTION AcquireToken(agent_id):
        bucket = GET_OR_CREATE(agent_id)
        REFILL(bucket)
        IF bucket.tokens >= 1:
            bucket.tokens -= 1
            RETURN 0.0
        ELSE:
            wait = (1 - bucket.tokens) / bucket.refill_rate
            SLEEP(wait)
            bucket.tokens -= 1
            RETURN wait
```

**Gate check:** All fields typed. Contract has input/output. Pseudocode clear. Proceed to plan.

---

## Step 3 — Plan (`docs/rate_limiter.plan.md`)

```markdown
# Implementation Plan: Rate Limiter  {#PL_RLM}

> **Code:** PL_RLM
> **Status:** in-progress
> **Created:** 2026-03-24
> **Updated:** 2026-03-24
>
> **Concept:** [C_RLM](./rate_limiter.concept.md)
> **Specification:** [SP_RLM](./rate_limiter.sp.md)

## Goal
Implement token-bucket rate limiting for per-agent LLM calls.

## Technology Decisions
| Decision | Choice | Rationale |
|----------|--------|-----------|
| Language | Python 3.12 | Project standard |
| Algorithm | Token bucket | Simple, well-understood |

## Progress
- [ ] Phase 1 — RateBucket data model
- [ ] Phase 2 — Integration with LLMRouter

## Phases

### Phase 1 — RateBucket (`engine/rate_limiter.py`) [TODO]
**Implements:** [SP_RLM_01](./rate_limiter.sp.md#SP_RLM_01)

### Phase 2 — LLMRouter Integration [TODO]
**Depends on:** Phase 1
**Implements:** [SP_RLM_02](./rate_limiter.sp.md#SP_RLM_02)
```

**Gate check:** Both spec sections covered. Dependencies stated. Proceed to code.

---

## Step 4 — Code

Implement `engine/rate_limiter.py` following the plan.
Add `# [C_RLM_02_01]` comments to link code back to concept sections.
