# Use Case: Team Coordination

Two engineers, each running their own Claude Code instance, working on the same codebase simultaneously. This scenario demonstrates the `CLAUDE_CODE_TASK_LIST_ID` coordination mechanism and the practical rules for avoiding conflicts.

---

## Scenario

**Team:** Two engineers (Alice and Bob), working on the same e-commerce platform.

**Sprint:** Fifth sprint of the quarter. The sprint backlog has 7 tasks:
1. Implement returns and refunds flow
2. Add shipping carrier integration (FedEx API)
3. Add real-time order tracking page
4. Build seller dashboard analytics
5. Implement discount code system
6. Add frontend for returns
7. Write integration tests and prepare PR

**Dependencies:**
- Task 3 (order tracking) depends on Task 2 (shipping integration)
- Task 6 (returns frontend) depends on Task 1 (returns flow)
- Task 7 (integration tests) depends on all others

**Both engineers have a Claude Code session running.**

---

## Setup: Shared Task List

```bash
# Alice's terminal
CLAUDE_CODE_TASK_LIST_ID=sprint-5 claude

# Bob's terminal
CLAUDE_CODE_TASK_LIST_ID=sprint-5 claude
```

Both sessions share `~/.claude/tasks/sprint-5.json` (keyed by the ID). The task list is visible to both:

```
[Sprint 5]
Status: pending    Priority: critical   "Implement returns and refunds flow"
Status: pending    Priority: high       "Add shipping carrier integration"
Status: pending    Priority: high       "Add real-time order tracking page"
Status: pending    Priority: medium     "Build seller dashboard analytics"
Status: pending    Priority: medium     "Implement discount code system"
Status: pending    Priority: medium     "Add frontend for returns"
Status: pending    Priority: low        "Integration tests + PR"
```

---

## How Claiming Works

**Alice's session picks up Task 1:**
```
TaskUpdate("Implement returns and refunds flow", status: in_progress)
```

The task list now shows:
```
Status: in_progress  "Implement returns and refunds flow"  [Alice's session]
```

**Bob's session looks for work:**
```
TaskList(status: pending)
```
Returns Tasks 2, 4, 5 (Tasks 1 is in_progress; Tasks 3 and 6 are blocked; Task 7 is blocked).

**Bob picks Task 2:**
```
TaskUpdate("Add shipping carrier integration", status: in_progress)
```

**Now executing in parallel:**
```
Alice's Claude Code session:           Bob's Claude Code session:
Task 1: Returns flow                   Task 2: FedEx integration
├── Write refund service               ├── Read FedEx API docs (WebFetch)
├── Add refund endpoint                ├── Write FedEx client service
├── Handle refund states               ├── Write shipping calculation logic
├── Tests                              ├── Tests
└── TaskUpdate: completed              └── TaskUpdate: completed
```

---

## The Dependency Resolution Flow

When Task 1 completes (returns flow), Task 6 (returns frontend) is automatically unblocked. When Task 2 completes (FedEx integration), Task 3 (real-time tracking) is unblocked.

Both engineers check `TaskList(status: pending)` and see new available work:
- Task 3 unblocked (tracking page)
- Task 4 available (seller analytics — no dep)
- Task 5 available (discount codes — no dep)
- Task 6 unblocked (returns frontend)

**Alice picks Task 6, Bob picks Task 3.** They run in parallel again.

---

## Merge Conflict Avoidance

The task map is designed to minimise overlap:

| Task | Primary files involved |
|---|---|
| Task 1 (returns) | `src/services/returns/`, `src/routes/returns.js` |
| Task 2 (FedEx) | `src/services/shipping/fedex.js`, `src/config/carriers.js` |
| Task 3 (tracking) | `src/pages/tracking/`, `src/services/tracking/` |
| Task 4 (analytics) | `src/services/analytics/`, `src/pages/seller-dashboard/` |
| Task 5 (discounts) | `src/services/discounts/`, `src/routes/coupons.js` |
| Task 6 (returns UI) | `src/pages/returns/`, `src/components/returns/` |

When tasks touch shared files (e.g., both Task 1 and Task 6 might touch the Order model), the dependency graph ensures they run sequentially, not in parallel. This is the value of expressing dependencies explicitly — it prevents the coordination overhead of conflict resolution downstream.

---

## Toolchain Used

| Tool | Coordination-Specific Usage |
|---|---|
| TaskCreate | Full sprint task graph created once, shared |
| TaskUpdate | Claiming (in_progress) and completion (completed) |
| TaskList | Check available work, see what's blocked |
| CLAUDE_CODE_TASK_LIST_ID | Environment variable pointing both sessions to same list |
| Bash | `git pull --rebase` before starting each task |

---

## What the Humans Do

| When | Action |
|---|---|
| Sprint start | One engineer creates the task graph (15 min) |
| Each morning | Review task list state, note what completed overnight |
| Task completion | Review the diff for each completed task (10–20 min each) |
| Merge conflicts | Handle manually (should be rare with good task decomposition) |
| End of sprint | Review integration tests, approve PR |

---

## Communication Protocol

A critical rule for this workflow: **communicate task boundaries clearly in the task description.** Each task description should include:

- The files it will primarily touch
- What it will NOT touch (explicit boundary)
- The interface it produces (API shape, component props, etc.)

This allows the dependent task's agent to know what to expect without needing to read the implementation.

Example task description:
```
"Add shipping carrier integration (FedEx API)

Produces:
- ShippingService class in src/services/shipping/ShippingService.js
- Public methods: calculateRate(from, to, weight, dimensions), createShipment(order), getTracking(trackingNumber)
- Does NOT modify Order model or any existing routes
```

---

## Realistic Limitations

**This workflow requires good task decomposition.** If tasks are poorly scoped and touch overlapping files, merge conflicts will occur. The coordination mechanism assumes the human did the decomposition work correctly.

**Both sessions see the same task list, but not each other's work in progress.** If Alice's Task 1 takes an approach that conflicts with Bob's Task 6 assumptions, they'll discover this at merge time — not earlier. Regular sync between engineers (not just agents) is still important.

**The claiming mechanism is advisory, not enforced by lock.** If two sessions simultaneously call `TaskUpdate` on the same task, a race condition can occur. Realistic mitigation: engineers agree who picks which class of tasks, or check in before picking up work.

**Overnight coordination:** Running overnight with two sessions is possible but requires clear boundaries. If both sessions pick up work in the same area at 2am, conflict resolution happens in the morning.

---

## Contrast: How You'd Do This Without Claude Code

Traditional coordination requires explicit assignment (JIRA tickets, standup discussion, Slack), manual status tracking, and the merge conflict resolution overhead of truly parallel work. The Claude Code shared task list reduces — but does not eliminate — this coordination overhead.
