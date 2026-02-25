# Task Management: From Backlog to Dependency Graph

On January 23, 2025, Claude Code upgraded its task system from `TodoWrite` (a simple in-session task list) to a full `TaskCreate` system with persistence, dependency graphs, and multi-agent coordination. This is arguably the most significant architectural upgrade in Claude Code's history, and it's the feature that most clearly differentiates it from every other AI coding tool.

No other major AI coding tool — not Cursor, not GitHub Copilot, not Gemini Code Assist — has anything architecturally equivalent.

---

## The Four Task Tools

| Tool | Purpose | Key Parameters |
|---|---|---|
| `TaskCreate` | Create a new task | `title`, `description`, `priority`, `addBlockedBy` |
| `TaskGet` | Retrieve a specific task | `taskId` |
| `TaskUpdate` | Update status or properties | `taskId`, `status` |
| `TaskList` | List all tasks with filters | `status`, `priority` |

### Status Lifecycle

```
pending ──▶ in_progress ──▶ completed
                │
                ▼
            blocked (if dependency unresolved)
```

A task is `blocked` when a task in its `addBlockedBy` list is not yet `completed`. The system enforces this — it will not begin execution on a blocked task until its dependencies are resolved.

---

## Persistence: Where Tasks Live

Tasks persist to `~/.claude/tasks/` in JSON format. This is the **home directory**, not the project directory.

This has two important consequences:

1. **Multi-session continuity**: Close Claude Code, reopen it, and tasks are still there. No other AI coding tool provides this.
2. **Project isolation**: Tasks are keyed by project, but stored centrally. You can run multiple Claude Code sessions against different projects without task lists colliding.

The persistence model resembles a lightweight project management system with a CLI interface, not a conversation-scoped to-do list.

---

## Dependency Graphs: Real Example

Here is a `TaskCreate` invocation for a realistic e-commerce checkout feature:

```
TaskCreate: "Set up database schema"
  priority: high
  description: "Create orders, order_items, payment_intents tables with migrations"

TaskCreate: "Implement payment service"
  priority: high
  addBlockedBy: ["Set up database schema"]
  description: "Stripe payment intent creation, confirmation, webhook handling"

TaskCreate: "Build cart API endpoints"
  priority: high
  addBlockedBy: ["Set up database schema"]
  description: "GET /cart, POST /cart/items, DELETE /cart/items/:id"

TaskCreate: "Build checkout API endpoint"
  priority: high
  addBlockedBy: ["Implement payment service", "Build cart API endpoints"]
  description: "POST /checkout — validates cart, creates payment intent, creates order"

TaskCreate: "Update frontend checkout flow"
  priority: medium
  addBlockedBy: ["Build checkout API endpoint"]
  description: "Replace mock checkout with real API calls, add loading states, error handling"

TaskCreate: "Write integration tests"
  priority: medium
  addBlockedBy: ["Build checkout API endpoint"]
  description: "End-to-end tests for checkout flow including payment failure scenarios"

TaskCreate: "Create PR and documentation"
  priority: low
  addBlockedBy: ["Update frontend checkout flow", "Write integration tests"]
  description: "Draft PR with summary, update API docs, update CHANGELOG"
```

This produces the following dependency graph:

```
Database Schema
├──▶ Payment Service ──────────────────────┐
└──▶ Cart API Endpoints ───────────────────┤
                                           ▼
                                  Checkout API Endpoint
                                  ├──▶ Frontend Checkout Flow ──┐
                                  └──▶ Integration Tests ────────┤
                                                                 ▼
                                                          PR + Documentation
```

Claude Code will not begin "Checkout API Endpoint" until both "Payment Service" and "Cart API Endpoints" are `completed`. This is not advisory — it's enforced by the system.

---

## Multi-Agent Coordination via Task Lists

The `CLAUDE_CODE_TASK_LIST_ID` environment variable enables multiple Claude Code sessions to share a single task list:

```bash
CLAUDE_CODE_TASK_LIST_ID=ecommerce-checkout-sprint claude
```

When two engineers run this command:

- Both sessions see the same `TaskList`
- A task marked `in_progress` by one session is effectively "claimed" — the other session will not pick it up
- Completed tasks are visible to all sessions immediately

This enables parallel execution of independent work streams without requiring external coordination:

```
Session 1 (Engineer A):                Session 2 (Engineer B):
┌──────────────────────┐              ┌──────────────────────┐
│ TaskList → pending   │              │ TaskList → pending   │
│ Picks: Payment Svc   │              │ Picks: Cart API       │
│ Status: in_progress  │              │ Status: in_progress  │
│                      │              │                      │
│ [works on payment]   │              │ [works on cart]       │
│ Status: completed    │              │ Status: completed    │
└──────────────────────┘              └──────────────────────┘
         │                                     │
         └──────────────┬──────────────────────┘
                        ▼
              Checkout API now unblocked
              Either session can pick it up
```

The `Ctrl+T` keyboard shortcut opens the task panel for visual inspection at any time.

---

## The TodoWrite Baseline (Pre-January 2025)

Before the TaskCreate upgrade, Claude Code used `TodoWrite` — a simple list structure that:
- Stored tasks as a flat array in the current session context
- Did not persist across sessions
- Had no dependency graph
- Had no multi-agent coordination capability

The upgrade to `TaskCreate` is the difference between a sticky note and a project management system. Both serve the purpose of "tracking what needs to be done," but they enable fundamentally different workflows.

---

## Comparison: Other Tools

| Capability | Claude Code | Cursor | GitHub Copilot | Gemini Code Assist | Windsurf |
|---|---|---|---|---|---|
| Task creation | ✅ TaskCreate | ❌ | ❌ | ❌ | ❌ |
| Task persistence | ✅ Cross-session | ❌ | ❌ | ❌ | ❌ |
| Dependency graphs | ✅ addBlockedBy | ❌ | ❌ | ❌ | ❌ |
| Status lifecycle | ✅ pending/in_progress/completed | ❌ | ❌ | ❌ | ❌ |
| Multi-agent sharing | ✅ CLAUDE_CODE_TASK_LIST_ID | ❌ | ❌ | ❌ | ❌ |
| Visual task panel | ✅ Ctrl+T | ❌ | ❌ | ❌ | ❌ |

No competitor has announced a roadmap feature that closes this gap. As of the research date (February 2026), this remains Claude Code's most structurally differentiated capability.

> See [`sdlc/05-git-and-cicd.md`](05-git-and-cicd.md) for how completed tasks translate into commits and PRs.
