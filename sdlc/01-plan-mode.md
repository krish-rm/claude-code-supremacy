# Plan Mode: Discovery and Sprint Planning

Plan Mode is the most structurally significant feature that separates Claude Code from every major competitor. It encodes a two-phase discovery-and-planning protocol directly into the system prompt — not as a prompting suggestion, but as an enforced architectural constraint.

---

## The Two-Phase Gate

```
Phase 1: EXPLORE (Read-Only)
┌────────────────────────────────────────────────┐
│  Haiku Sub-Agent (516 tokens, low-cost)        │
│                                                │
│  Tools allowed: Read, Glob, Grep, LS          │
│  Tools blocked: Write, Edit, Bash (mutating)  │
│                                                │
│  Goal: Understand before proposing             │
└────────────────────────────────────────────────┘
            │
            ▼
Phase 2: PLAN PRESENTATION
┌────────────────────────────────────────────────┐
│  Plan presented to user in structured format   │
│                                                │
│  Includes:                                     │
│  - Files to be modified                        │
│  - Changes to be made per file                 │
│  - New files to be created                     │
│  - Tasks in dependency order                   │
│  - Risk assessment                             │
└────────────────────────────────────────────────┘
            │
            ▼ [USER APPROVAL REQUIRED]
            │
Phase 3: EXECUTION
┌────────────────────────────────────────────────┐
│  Full toolset unlocked                         │
│  TaskCreate fires (if feature is complex)      │
│  Execution Loop begins                         │
└────────────────────────────────────────────────┘
```

The separation between Explore and Execute is not a stylistic choice. It is a guard against the most common failure mode in AI coding: premature writing. An agent that jumps to code generation before understanding the codebase produces changes that are locally correct but structurally wrong — incompatible with existing patterns, missing dependencies, or duplicating functionality that already exists.

---

## The Toolset Restriction in Plan Mode

The system prompt restricts Plan Mode to read-only tools:

| Tool | Allowed in Plan Mode |
|---|---|
| Read | ✅ |
| Glob | ✅ |
| Grep | ✅ |
| LS | ✅ |
| Write | ❌ |
| Edit | ❌ |
| MultiEdit | ❌ |
| Bash (mutating) | ❌ |
| Bash (read-only, e.g. `cat`) | ✅ |

This restriction is enforced at the sub-agent level, not as a suggestion. The Explore sub-agent simply does not have access to mutation tools.

---

## The Haiku Sub-Agent: Why Low-Cost Matters

The Explore phase uses Claude Haiku (516 tokens for the sub-agent prompt), not Sonnet or Opus. This is a deliberate cost/quality trade-off:

- **Codebase reading is cheap** — Understanding file structure, reading imports, mapping function signatures does not require the full capability of Sonnet.
- **Token economics compound** — In a large codebase, the Explore phase might read 50–100 files. Using Haiku here makes the overall operation 5–10x cheaper without sacrificing plan quality.
- **Exploration is broad, not deep** — The goal is coverage, not analysis. Haiku is well-suited to "read everything, understand structure" while Sonnet is reserved for "make complex decisions."

---

## Real Example: JWT Authentication Feature

To make this concrete, here is what Plan Mode produces for a typical mid-complexity feature request: "Add JWT authentication to the existing Express API."

**Explore phase reads:**
- `package.json` — existing dependencies, identifies no JWT library present
- `src/routes/` — all route files, maps which routes exist
- `src/middleware/` — existing middleware, finds auth middleware stub
- `src/models/User.js` — user schema, confirms `passwordHash` field exists
- `.env.example` — environment variable patterns

**Plan presented:**
```
## Implementation Plan: JWT Authentication

### New files:
- src/middleware/auth.js — JWT verification middleware
- src/utils/jwt.js — Token generation/verification helpers

### Modified files:
- package.json — Add jsonwebtoken, bcrypt
- src/routes/auth.js — Add /login and /refresh endpoints
- src/routes/protected.js — Apply auth middleware to protected routes
- .env.example — Add JWT_SECRET, JWT_EXPIRES_IN

### Task order (with dependencies):
1. Install dependencies [no dependencies]
2. Create jwt.js utility [depends on 1]
3. Create auth.js middleware [depends on 2]
4. Add auth routes [depends on 3]
5. Apply middleware to protected routes [depends on 4]
6. Write tests [depends on 5]

### Risk assessment:
- Existing /user routes may conflict with new /auth routes — will check
- bcrypt hashing will increase registration latency ~100ms — acceptable
```

**User approves → Execution begins.**

This plan would have taken a senior engineer 20–30 minutes to produce without specialized knowledge of the codebase. The Explore sub-agent produces it in under 2 minutes.

---

## Agile Mapping

| Agile Concept | Plan Mode Equivalent |
|---|---|
| Story pointing | Explore phase estimates complexity |
| Sprint planning meeting | Plan presentation and discussion |
| Definition of ready | User approval gate |
| Acceptance criteria | Verification steps included in plan |
| Risk log | Risk assessment section |

The key difference from a human sprint planning session: Claude Code's plan is generated *from actual codebase state*, not from memory or assumptions. It reads the code before proposing changes.

---

## The CLAUDE.md Acceleration Effect

When a `CLAUDE.md` file exists in the project root, the Explore phase is significantly faster — and more accurate. `CLAUDE.md` provides:

- Project architecture overview (skips the "what is this codebase?" phase)
- Existing conventions (code style, test patterns, naming)
- Known constraints (do not modify X, always use Y for Z)
- Environment setup

A well-written `CLAUDE.md` can reduce Explore phase time by 50–70% on large codebases by eliminating the need to infer context that should already be explicit.

> See [`prompts/claude-md-guide.md`](../prompts/claude-md-guide.md) for how to write a `CLAUDE.md` that maximally accelerates Plan Mode.

---

## Why Cursor's Lack of This Matters

Cursor's architecture is built around the conversation model: you describe what you want, Cursor responds. There is no enforced Explore phase. Cursor *can* read files when asked, but:

1. There's no sub-agent with a restricted toolset enforcing read-before-write
2. There's no structured plan presentation with a mandatory approval gate
3. There's no cost-optimised Haiku exploration pass

In practice, this means Cursor often begins writing code before fully understanding the codebase. For small, self-contained changes, this works fine. For multi-file features that require understanding existing patterns, it frequently produces changes that are locally correct but structurally inconsistent.

This is not a criticism of Cursor's model — it's optimised for a different use case (pair programming on smaller, bounded tasks). It's a description of a structural consequence of the architectural choice.

> See [`comparisons/vs-cursor.md`](../comparisons/vs-cursor.md) for the full comparison.
