# Use Case: Feature Sprint

A full sprint's worth of work — multiple stories, cross-cutting concerns, frontend and backend, with CI integration — is Claude Code's primary sweet spot. This use case demonstrates the task management and parallel execution capabilities at their most useful.

---

## Scenario

**Sprint goal:** Implement user notifications (in-app, email, push) for an existing SaaS platform.

**Stories:**
1. Notification data model and persistence layer
2. In-app notification API (create, list, mark-read, mark-all-read)
3. Email notification service (Sendgrid integration, templating)
4. Push notification service (Firebase FCM integration)
5. Frontend notification bell + dropdown UI component

**Dependencies:** Frontend stories depend on the API existing. Email and push can be built in parallel. The API depends on the data model.

---

## Claude Code Approach: Step by Step

### Step 1: Plan Mode

```
> Plan Mode: Implement the notifications sprint. The 5 stories are:
  [story descriptions]
  
  The codebase is documented in CLAUDE.md.
  All 5 stories should be part of a single plan. Identify dependencies.
```

**Explore reads:** Existing notification stubs (none), User model (for ownership), existing email service patterns (if any), frontend component structure, CI configuration.

**Plan presented:**
```
## Notifications Sprint Plan

### Story 1: Data Model (no external deps)
- Notification Prisma model: userId, type, title, body, metadata, readAt, createdAt
- Migration: notifications table
- Seed data for development

### Story 2: In-App API (depends on Story 1)
- GET /notifications (paginated, user-owned)
- POST /notifications (internal service call — authenticated separately)
- PATCH /notifications/:id/read
- PATCH /notifications/read-all
- Unit tests for all endpoints

### Story 3: Email Service (depends on Story 1, parallel with Story 4)
- EmailNotificationService wrapping Sendgrid
- Template system (handlebars)
- 3 templates: welcome, password-reset, order-confirmation
- Queue integration (Bull job for each send)
- Unit tests with mocked Sendgrid

### Story 4: Push Service (depends on Story 1, parallel with Story 3)
- PushNotificationService wrapping Firebase Admin SDK
- Device token storage (UserDevice model + migration)
- Fanout logic for multi-device users
- Unit tests with mocked Firebase

### Story 5: Frontend Component (depends on Story 2)
- Notification icon with unread count badge
- Dropdown list with infinite scroll (TanStack Query)
- Mark-read on click, mark-all-read button
- Real-time update via polling (every 30s)
- Component tests

### Integration:
- Connect email + push to event system (notification:created event)
- Full integration tests
- PR with complete feature documentation
```

**User approves → TaskCreate fires.**

### Step 2: TaskCreate Dependency Graph

```
TaskCreate: "Notifications data model + migration"
  priority: critical, no deps

TaskCreate: "In-app notification API"
  priority: high, addBlockedBy: ["Notifications data model"]

TaskCreate: "Email notification service"
  priority: high, addBlockedBy: ["Notifications data model"]

TaskCreate: "Push notification service"
  priority: high, addBlockedBy: ["Notifications data model"]

TaskCreate: "Frontend notification component"
  priority: high, addBlockedBy: ["In-app notification API"]

TaskCreate: "Connect email + push to event system"
  priority: medium, addBlockedBy: [
    "Email notification service",
    "Push notification service"
  ]

TaskCreate: "Integration tests"
  priority: medium, addBlockedBy: [
    "Connect email + push to event system",
    "Frontend notification component"
  ]

TaskCreate: "Create PR with documentation"
  priority: low, addBlockedBy: ["Integration tests"]
```

### Step 3: Parallel Execution

After Story 1 (data model) completes:

```
Stories 3 and 4 are independent → spawn in parallel

Main agent:
├── Task("Build email notification service", tools=[...]) ──▶ Sub-Agent A
└── Task("Build push notification service", tools=[...]) ──▶ Sub-Agent B

Meanwhile:
Story 2 (API) begins in main execution loop [also independent of 3 and 4]
```

**Stories 2, 3, and 4 execute in parallel.** Wall-clock time for three stories: approximately the time for the longest single story, not three sequential stories.

### Step 4: CI Integration

After each set of stories completes and commits are pushed:

1. GitHub Actions runs: lint → type-check → unit tests → integration tests
2. Claude Code reads CI output via `gh run view`
3. CI pass: mark tasks complete
4. CI fail (example: Sendgrid API type mismatch): read error, fix, recommit

---

## Toolchain Used (Complete Sprint)

| Tool | Sprint-Specific Usage |
|---|---|
| TaskCreate | Full dependency graph for 8 tasks |
| Task | Spawn parallel sub-agents for concurrent stories |
| Bash | `npx prisma migrate dev`, `npx jest`, `npm run lint` |
| Bash | `gh run view` — read CI output |
| Write | New service files, migration files, components |
| Edit | Integrate new services into event system |
| Glob | Find all test files to understand pattern |
| WebFetch | Sendgrid and Firebase documentation |

---

## What the Human Does

| When | Action | Time |
|---|---|---|
| Sprint start | Review plan, edit if needed | 20 min |
| After data model | Review migration and model | 10 min |
| During parallel phase | Available for escalations | Async |
| After API complete | Test endpoints in Postman/Thunder | 20 min |
| After email/push | Verify with actual service (sandbox) | 20 min |
| After frontend | UI review in browser | 20 min |
| CI results | Review CI run, approve PR | 30 min |

**Total human time: ~2 hours** for a sprint that would typically take a solo engineer 3–5 days.

---

## Time Estimate

| | With Claude Code | Without (solo engineer) |
|---|---|---|
| Wall-clock time | ~4–6 hours (with parallel) | 3–5 days |
| Human active time | ~2 hours | 3–5 days |
| Time of day | Can run overnight | Only during work hours |

The overnight pattern is meaningful here: kick off the sprint at end of day, review results in the morning. The calendar day burned is close to zero.

---

## Where It Might Struggle

- **Integration between services:** The email/push services need to connect to the event system. If the event system is complex or undocumented, this integration step may require human guidance.
- **Firebase and Sendgrid account setup:** Claude Code writes the integration code but cannot access your actual service accounts. You provide keys and verify connectivity manually.
- **Visual component quality:** The frontend notification component will be functional but may not match your design system precisely. Expect visual polish work.
- **Complex business rules:** If "who gets notified when" involves complex business logic (role hierarchies, subscription tiers), Claude Code will implement what's specified — but the spec needs to be complete.

---

## Contrast: How You'd Do This With Cursor

Cursor handles individual stories well. The advantage of Claude Code appears in the sprint-level coordination: the task graph ensures the right order, parallel sub-agents cut wall-clock time, and CI integration makes the feedback loop automatic. With Cursor, you'd replicate most of this coordination manually.
