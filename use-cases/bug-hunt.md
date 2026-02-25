# Use Case: Bug Hunt

Debugging complex issues — particularly intermittent failures in distributed systems — is a scenario where Claude Code provides meaningful leverage, but with important limitations. This use case is honest about both.

---

## Scenario

**Task:** An intermittent production failure in a distributed Node.js/Redis/PostgreSQL system. The symptom: order processing occasionally fails with "duplicate key violation" on the `orders` table, but only under load. Partial logs are available. The failure rate is ~0.3% — reliably reproducible under load testing but not in standard testing.

**Context:** Complex codebase, multiple services, async job queue (Bull/Redis).

---

## Claude Code Approach: Step by Step

### Step 1: Log Analysis and Hypothesis Generation

```
> We have an intermittent production bug: duplicate key violation on orders table,
  ~0.3% failure rate under load. Here are the logs from the last 3 occurrences:
  [paste logs]
  
  Start by analysing the logs. Don't touch any code yet.
```

**Explore phase reads:**
- The stack trace in each log entry — identifies the exact insertion call
- `src/services/order-service.js` (identified from stack trace)
- `src/jobs/payment-processor.js` (job queue processor)
- `src/models/Order.js` — Sequelize model, any unique constraints
- `src/utils/id-generator.js` — how order IDs are generated
- Queue configuration (job concurrency settings)
- Database schema (migration files — how the unique constraint is defined)

**Hypothesis generation:**

```
Based on log analysis and codebase read:

Primary hypothesis:
  Race condition in order creation. Two queue workers can receive the same
  payment event within the queue deduplication window and both attempt to
  create an order with the same external reference ID.

Evidence:
  - Log timestamps show two concurrent requests ~40ms apart (within Bull's
    5-minute dedup window if hash not set correctly)
  - order-service.js line 847: no database-level idempotency check before insert
  - id-generator.js uses timestamp + random: theoretical collision under high load
  - Payment processor job does not use Bull's jobId for deduplication

Secondary hypothesis:
  Webhook retry creating duplicate: Stripe may retry a webhook before the
  first one completes processing.

Files to examine further:
  - webhooks/stripe-handler.js
  - config/bull-config.js (deduplication settings)
```

### Step 2: Root Cause Verification

```
> Investigate the primary hypothesis: race condition in order creation.
  Read the relevant files and trace the complete code path.
```

**Claude Code reads and traces:**

1. `webhooks/stripe-handler.js` — receives webhook, dispatches job
2. `config/bull-config.js` — job deduplication settings
3. `src/jobs/payment-processor.js` — job handler
4. `src/services/order-service.js` — order creation logic
5. `src/models/Order.js` — constraints

**Finding:**

```
Root cause confirmed:

In bull-config.js:
  defaultOptions: {
    attempts: 3,
    backoff: { type: 'exponential', delay: 1000 }
  }
  // No jobId set — deduplication is not configured

In stripe-handler.js (line 34):
  queue.add('process-payment', { stripeEventId, paymentIntent });
  // stripeEventId is not used as the jobId — jobs are not deduplicated

In payment-processor.js (line 89):
  const order = await OrderService.createFromPaymentIntent(paymentIntent);
  // No check if order already exists for this payment intent

In order-service.js (line 847):
  const order = await Order.create({ paymentIntentId, ... });
  // Assumes no existing order — no findOrCreate pattern

Under load: Stripe sends webhook → queued as job A. Within Bull's window,
a retry arrives → queued as job B (different Bull ID). Both workers run
concurrently. Both reach Order.create with the same paymentIntentId.
First one succeeds, second hits duplicate key violation.
```

### Step 3: Fix

```
> Apply the fix. The approach should be:
  1. Use stripeEventId as Bull jobId (deduplication at queue level)
  2. Add findOrCreate pattern in order-service as a database-level safety net
  3. Add a unique index on paymentIntentId if not already present
  4. Write a test that simulates concurrent job execution
```

**Claude Code implements:**
- Modifies `stripe-handler.js` — adds `{ jobId: stripeEventId }` to Bull `add()`
- Modifies `order-service.js` — replaces `Order.create()` with `Order.findOrCreate()`
- Reads migration files, adds new migration for `paymentIntentId` unique index
- Writes test that loads the payment processor with the same event 5 times concurrently
- Runs test suite, verifies no duplicate key errors under simulated concurrency

---

## Toolchain Used

| Tool | Debugging-Specific Usage |
|---|---|
| Explore sub-agent | Codebase mapping before any hypothesis |
| Grep | Find all locations that create orders |
| Grep | Find all queue job definitions |
| Read | Trace the complete code path |
| Bash | `git log --oneline src/services/order-service.js` — history of this file |
| Bash | Run load test simulation with concurrent events |
| Bash | Verify fix with stress test |

---

## What the Human Does

| When | Action | Time |
|---|---|---|
| Start | Provide logs, describe the symptom | 10 min |
| After hypothesis | Validate the hypothesis makes sense | 15 min |
| After root cause | Confirm interpretation is correct | 10 min |
| After fix | Review the diff carefully | 20 min |
| After test | Verify the stress test actually tests the right thing | 15 min |

**Total human time: ~1 hour.** A senior engineer debugging this manually might take 2–8 hours (the intermittent nature makes reproduction tricky).

---

## Where It Might Struggle

**Intermittent failures that can't be reproduced:** If the bug only appears in production under specific conditions (database state, network timing, specific data patterns), Claude Code cannot reproduce it locally. It can reason about hypotheses but cannot verify them.

**External system dependencies:** If the bug involves Stripe's webhook timing, Redis clustering behaviour, or PostgreSQL transaction isolation specifics that differ between environments, Claude Code can read documentation but cannot observe the actual external system behaviour.

**Memory leaks and performance degradation:** These require profiling tools and observation over time. Claude Code can add profiling instrumentation but not interpret live profiler output directly.

**Statistical debugging:** A 0.3% failure rate requires reproducing the condition many times. Claude Code can run tests but not run them 10,000 times to get statistically significant results efficiently.

---

## Contrast: How You'd Do This With Cursor

Cursor is genuinely useful for debugging — you can paste logs, ask questions, and navigate code in the IDE. The primary advantage of Claude Code here is the structured Explore → Hypothesis → Root Cause flow, where the agent reads the entire code path systematically rather than following the engineer's navigation choices. For experienced engineers who know the codebase, Cursor may be equally effective. For engineers unfamiliar with the codebase, Claude Code's systematic approach has an advantage.
