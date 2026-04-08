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
## 03. Verification Criteria  {#SP_RLM_03}

### 03_01. Functional Expectations  {#SP_RLM_03_01}

| Contract | Scenario | Input | Expected outcome |
|----------|----------|-------|------------------|
| AcquireToken | Tokens available | agent with full bucket | Returns 0.0, tokens decremented by 1 |
| AcquireToken | Bucket empty | agent with 0 tokens | Returns wait_seconds > 0, call delayed |
| AcquireToken | New agent | unknown agent_id | Creates bucket with default capacity, returns 0.0 |

### 03_02. Invariant Checks  {#SP_RLM_03_02}

| Invariant | Verification method |
|-----------|-------------------|
| tokens <= capacity | Drain bucket, wait for refill, verify tokens never exceed capacity |
| tokens >= 0 | Rapid concurrent calls, verify tokens don't go negative |

### 03_03. Integration Scenarios  {#SP_RLM_03_03}

| Scenario | Preconditions | Steps | Expected result |
|----------|--------------|-------|-----------------|
| LLM calls throttled | LLMRouter configured with RateLimiter | Send capacity+1 rapid calls | First N calls immediate, N+1 delayed by ~1/refill_rate seconds |

### 03_04. Edge Cases and Boundaries  {#SP_RLM_03_04}

| Case | Input | Expected behavior |
|------|-------|-------------------|
| Min capacity | capacity = 1 | Only 1 call before throttling |
| Max capacity | capacity = 1000 | 1000 calls before throttling |
| Min refill rate | refill_rate = 0.1 | 10 seconds to refill 1 token |
| Concurrent agents | 100 agents simultaneously | Each has independent bucket, no cross-interference |
```

**Gate check:** All fields typed. Contract has input/output. Pseudocode clear. Verification criteria defined. Proceed to plan.

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
Follow SOLID principles (see [solid-architecture reference](../references/solid-architecture.md))
unless project rules define alternatives.

```python
# engine/rate_limiter.py

import time
from dataclasses import dataclass, field

# [SP_RLM_01_01] RateBucket — token bucket data model
@dataclass
class RateBucket:
    agent_id: str
    capacity: int = 10
    tokens: float = field(default=None)
    refill_rate: float = 1.0
    last_refill: float = field(default_factory=time.time)

    def __post_init__(self):
        if self.tokens is None:
            self.tokens = float(self.capacity)

# [C_RLM_02_01] RateLimiter — per-agent token bucket management
# [SP_RLM_02_01] AcquireToken contract
class RateLimiter:
    """Rate limiter using token bucket algorithm (DIP: depends on abstract storage interface)."""

    def __init__(self, storage: "BucketStorage"):
        self._storage = storage

    def _refill(self, bucket: RateBucket) -> None:
        now = time.time()
        elapsed = now - bucket.last_refill
        bucket.tokens = min(bucket.capacity, bucket.tokens + elapsed * bucket.refill_rate)
        bucket.last_refill = now

    def acquire_token(self, agent_id: str) -> float:
        bucket = self._storage.get_or_create(agent_id)
        self._refill(bucket)
        if bucket.tokens >= 1:
            bucket.tokens -= 1
            return 0.0
        wait = (1 - bucket.tokens) / bucket.refill_rate
        time.sleep(wait)
        self._refill(bucket)
        bucket.tokens -= 1
        return wait
```

Update plan: Phase 1 `[TODO]` → `[DONE]`.

---

## Step 5 — Test (Functional)

Run only relevant unit and mock tests covering the changed code.
No permission needed — functional tests are safe.

```
$ pytest tests/test_rate_limiter.py -v

tests/test_rate_limiter.py::test_acquire_token_immediate       PASSED
tests/test_rate_limiter.py::test_acquire_token_rate_limited     PASSED
tests/test_rate_limiter.py::test_refill_restores_tokens         PASSED
tests/test_rate_limiter.py::test_bucket_defaults                PASSED
tests/test_rate_limiter.py::test_acquire_token_unknown_agent    PASSED

5 passed in 0.12s
```

Example test with traceable ID:

```python
def test_acquire_token_rate_limited():
    """[SP_RLM_02_01] AcquireToken returns wait_seconds when bucket is empty."""
    storage = InMemoryBucketStorage()
    limiter = RateLimiter(storage)
    # Drain all tokens
    for _ in range(10):
        limiter.acquire_token("agent-1")
    # Next call should return wait > 0
    wait = limiter.acquire_token("agent-1")
    assert wait > 0
```

**Gate check:** All spec contracts tested. Error cases covered. Proceed to Review.

---

## Step 6 — Review (Pre-Commit)

Launch a reviewer subagent with a clean context. The subagent receives:
- Git diff of all changes
- `rate_limiter.sp.md` (spec)
- `rate_limiter.plan.md` (plan)
- `.dev_flow/rules/` (if exists)
- `references/solid-architecture.md` (SOLID principles)

```
## Pre-Commit Review

**Result:** PASS

### Blocking Issues
(none)

### Warnings
- Consider adding `BucketStorage` abstract base class to make DIP explicit — engine/rate_limiter.py:20

### SOLID Compliance
- SRP: RateBucket (data) and RateLimiter (logic) are separated ✓
- OCP: Storage is injectable, new storage backends don't require changes ✓
- DIP: RateLimiter depends on abstract BucketStorage, not concrete class ✓

### Summary
Implementation matches spec contracts. SOLID principles followed.
One minor suggestion about making the storage interface explicit.
```

**Gate check:** No blocking issues. Warning acknowledged. Proceed to Verify.

---

## Step 7 — Verify (Regression / Integration / Live)

After review passes, run broader verification. Ask user permission before
creating new integration or live test scenarios.

### Regression

```
$ pytest tests/ -v --timeout=60

182 passed, 0 failed in 3.41s
```

No regressions introduced.

### Integration (if applicable)

If `RateLimiter` integrates with a real LLM router — verify the integration
using a test account or sandbox environment:

```
$ pytest tests/integration/test_llm_router.py -v -k rate_limit

tests/integration/test_llm_router.py::test_rate_limit_delays_calls   PASSED

1 passed in 2.15s
```

### Live / Manual Verification (if applicable)

When no automated live tests exist, provide a manual checklist:

```
## Manual Verification

Prerequisites:
- Dev server running: `python -m app serve --dev`

Steps:
1. Send 10 rapid requests to POST /agents/agent-1/execute
2. Expected: first 10 succeed immediately (status 200)
3. Send 11th request immediately
4. Expected: response delayed by ~1 second (rate limiting active)
5. Wait 10 seconds, send another request
6. Expected: immediate response (tokens refilled)
```

**Gate check:** Regression tests pass. Integration verified. Proceed to Commit.

---

## Step 8 — Commit

Ask the user for explicit commit approval before committing.

```
All checks passed:
- ✓ Functional tests (5/5)
- ✓ Pre-commit review (PASS, 1 warning acknowledged)
- ✓ Regression tests (182/182)
- ✓ Integration tests (1/1)

Ready to commit?
```

After user confirms:

```
git add engine/rate_limiter.py tests/test_rate_limiter.py docs/rate_limiter.plan.md
git commit -m "[SP_RLM] Implement token-bucket rate limiter with per-agent limits"
```
