# Use Case: Legacy Codebase

Navigating an unfamiliar or legacy codebase is one of the highest-value scenarios for Claude Code. The Explore sub-agent's ability to map a large codebase before making any changes directly addresses the most dangerous failure mode in legacy work: modifying code based on incomplete understanding.

---

## Scenario

**Task:** You've inherited a 100,000 LOC Node.js monolith (3+ years old, mixed TypeScript/JavaScript, partially documented). You need to add a payments feature — Stripe integration, webhook handling, and a payment history page — without breaking existing functionality.

**Context:** No existing `CLAUDE.md`. Incomplete documentation. Some tests, but inconsistent coverage. CI exists but is flaky.

---

## Claude Code Approach: Step by Step

### Step 1: Initial Exploration

```
> This is an inherited codebase. Before we do anything, explore the project
  structure, understand how it's organised, and then generate a CLAUDE.md.
  Don't make any changes yet.
```

The **Explore sub-agent** (Haiku, read-only) conducts a broad scan:

**What it reads:**
- `package.json` — dependency inventory (Express 4.x, Mongoose, various utilities)
- Top-level directory structure (`src/`, `tests/`, `config/`, `scripts/`)
- `src/routes/` — all route files (maps the API surface)
- `src/models/` — all model files (understands data schema)
- `src/middleware/` — existing middleware
- Existing test patterns (`tests/**/*.test.js`) — infers test approach
- CI configuration (`.travis.yml` — old CI, flaky flag noted)
- README, any documentation files
- A sample of the largest/most complex files to understand code style

**Output:** A codebase map with:
- Module structure overview
- Existing patterns (error handling, auth approach, response format)
- Test coverage estimate (rough)
- Identified dark corners (files with no tests, TODO counts)
- Potential landmines (hardcoded config values, undocumented side effects)

### Step 2: CLAUDE.md Generation

From the Explore output, the CLAUDE.md Creator sub-agent generates a project memory file. Key sections that matter for a legacy codebase:

```markdown
## Gotchas and Known Issues
- `src/utils/currency.js` has no tests and complex rounding logic — treat carefully
- Payment-related routes don't currently exist (to be implemented)
- Tests use a mix of Mocha (older) and Jest (newer) — don't mix test runners
- `.travis.yml` CI is being replaced — don't rely on it

## Do Not Touch
- `src/legacy/` — marked for deprecation, don't add dependencies to these files
- `migrations/` — the team uses a locked migration pattern

## Code Conventions
- Error handling: always use `next(error)` pattern, never `res.status().json()` directly
- Responses: use `res.success(data)` helper (see `src/middleware/response-helper.js`)
- Authentication: JWT in Authorization header, use `requireAuth` middleware
```

**Human reviews and edits `CLAUDE.md` before continuing.** This review is critical — the auto-generated version will capture patterns but will miss institutional knowledge.

### Step 3: Scoped Plan Mode

```
> Plan Mode: Add Stripe payments integration including:
  - Stripe checkout session creation
  - Webhook handling (payment succeeded, failed, refunded)
  - Payment model in the database
  - Payment history API endpoint
  - Payment history page in the frontend (React, in /frontend)
```

**Plan Mode Explore phase (now aided by CLAUDE.md):**
- Reads `src/models/` to understand User model (for relating payments)
- Reads `src/routes/` to find the right place to add payment routes
- Reads `src/middleware/response-helper.js` (referenced in CLAUDE.md)
- Reads existing auth patterns to apply `requireAuth` correctly
- Reads `.env.example` to understand environment variable pattern
- Reads `tests/` to understand test patterns

**Plan output:**
```
## Payments Integration Plan

### New files:
- src/models/Payment.js — Stripe payment record model
- src/routes/payments.js — CRUD endpoints + Stripe checkout
- src/webhooks/stripe.js — Webhook handler (separate from routes for security)
- tests/payments.test.js — Unit tests for payment logic
- tests/webhooks.test.js — Webhook handler tests
- frontend/pages/PaymentHistory.jsx — Payment history UI component

### Modified files:
- src/routes/index.js — Register /payments and /webhooks routes
- .env.example — Add STRIPE_SECRET_KEY, STRIPE_WEBHOOK_SECRET
- package.json — Add stripe package

### Risk assessment:
- Webhook endpoint must be exempt from CSRF middleware (standard requirement)
- Currency rounding in src/utils/currency.js is untested — verify before using
- CI is flaky — consider running tests 3x locally before relying on CI pass
```

**User approves.**

### Step 4: Execution with Extra Caution

Legacy codebase execution has a different character than greenfield:

**Minimal-diff approach:** Every edit is targeted. No refactoring of existing code unless it directly blocks the feature. When Claude Code encounters a pattern that "could be better," it notes it but doesn't change it.

**Heavy verification:** After each change, tests run against the existing test suite to ensure nothing broke. The 3-strike rule applies extra conservatively — ambiguous failures escalate quickly.

**Dark corner avoidance:** The CLAUDE.md `Do Not Touch` list prevents Claude Code from touching `legacy/` or migration files. The Explore findings about `currency.js` mean that file is read carefully before any usage.

---

## Toolchain Used

| Tool | Legacy-Specific Usage |
|---|---|
| Explore sub-agent | Full codebase scan before any modification |
| Glob | Find all test files to understand patterns |
| Grep | Find all usages of modified modules (impact analysis) |
| Read | Every file before editing — no assumptions |
| Bash | `git grep "requireAuth"` — find all auth patterns before adding more |
| Bash | Run full test suite after every meaningful change |

---

## What the Human Does

| When | Action | Time |
|---|---|---|
| Start | Prompt the explore, review the codebase map | 30 min |
| After Explore | Review and heavily edit CLAUDE.md | 45 min |
| After Plan | Review plan, check for missing context | 20 min |
| During execution | Review each new file before any tests | 30 min |
| After completion | Full regression test review, manual testing | 1 hour |

**Total human time: ~2.5–3 hours.** Estimated equivalent without Claude Code: 2–3 days (for an engineer unfamiliar with the codebase).

---

## Where It Might Struggle

- **Undocumented side effects:** A function that sends an email or calls a third-party API as a side effect may not be visible from its signature. The Explore phase will miss these if they're not documented or tested.
- **Implicit contracts:** Code that relies on shared state, undocumented ordering, or implicit caller conventions is risky to modify without knowing the full context.
- **Flaky CI:** If CI is flaky (as described in this scenario), the CI recovery loop cannot reliably distinguish "my change broke something" from "the CI is having a bad day." Human judgment required.
- **Very old JavaScript patterns:** Pre-ES6 code with callback-heavy patterns can confuse modern pattern matching in Explore.

---

## Contrast: How You'd Do This With Cursor

With Cursor, you'd navigate the codebase manually — reading files, following references, asking the AI questions as you go. The AI would be helpful for answering specific questions ("how does auth middleware work here?") but the exploration burden is mostly yours. For a 100k LOC codebase, this exploration step can take days. Claude Code's Explore sub-agent compresses it to minutes.
