# Use Case: Greenfield Project

Starting a project from scratch is where Claude Code's full architecture can be applied from day one — before technical debt accumulates, before conventions ossify, before the codebase develops dark corners. This use case walks through building a complete web application with Claude Code from initial setup to first PR.

---

## Scenario

**Task:** Build a REST API with JWT authentication, a React frontend, PostgreSQL database, Docker containerisation, and GitHub Actions CI.

**Starting point:** Empty repository, `README.md` only.  
**Human time available:** Review sessions (15–30 min each), not continuous presence.

---

## Claude Code Approach: Step by Step

### Step 1: Generate CLAUDE.md

The first action in any new project should be creating a `CLAUDE.md` — but on a greenfield project, there's no codebase to read. Instead, you provide the project specification and ask Claude Code to generate a forward-looking `CLAUDE.md`:

```
> I'm starting a new project: a REST API with JWT auth, React frontend,
  PostgreSQL, Docker, and GitHub Actions CI. The tech stack will be:
  - Backend: Node.js + Express + Prisma ORM + TypeScript
  - Frontend: React 18 + Vite + TanStack Query + TypeScript
  - DB: PostgreSQL 16
  Can you generate a CLAUDE.md for this project?
```

The CLAUDE.md Creator sub-agent generates a complete project memory file covering the stack, conventions it will follow, testing approach, and CI setup. **Review this carefully** — these conventions will be applied consistently throughout the entire project.

### Step 2: Plan Mode

With `CLAUDE.md` established:

```
> Plan Mode: Build the full project according to CLAUDE.md specs.
  Start with the foundation (Docker, DB, project structure) and
  work up to the frontend. Use a feature from stories in STORIES.md.
```

**Explore phase:** Reads CLAUDE.md (the primary source), checks for any existing files.

**Plan presented:**
```
## Project Build Plan

### Phase 1: Foundation (no dependencies)
1. Initialize Node.js + TypeScript backend project
2. Configure Docker + docker-compose for app + PostgreSQL
3. Set up GitHub Actions CI pipeline skeleton

### Phase 2: Database Layer (depends on Phase 1)
4. Set up Prisma with User, Session schemas
5. Create and test initial migration
6. Add seeding for development

### Phase 3: Backend Core (depends on Phase 2)
7. Implement JWT utilities (generation, verification, refresh)
8. Implement auth middleware
9. Add /auth/register, /auth/login, /auth/refresh endpoints
10. Add user management endpoints (profile, update)
11. Integration tests for all endpoints

### Phase 4: Frontend Foundation (can run parallel with Phase 3)
12. Initialize React + Vite + TypeScript frontend
13. Set up routing (React Router v6)
14. Set up TanStack Query + Axios client
15. Build auth context and protected routes

### Phase 5: Integration (depends on Phases 3 and 4)
16. Connect frontend auth flow to backend
17. Build user profile page
18. End-to-end tests (Playwright)
19. Complete CI pipeline with all tests
```

**User approves → TaskCreate fires.**

### Step 3: TaskCreate with Dependency Graph

```
Task 1: "Initialize backend project" [no deps]
Task 2: "Configure Docker" [no deps]
Task 3: "GitHub Actions skeleton" [no deps]
Task 4: "Set up Prisma schema" [deps: 1, 2]
Task 5: "Create initial migration" [deps: 4]
Task 6: "Add development seed data" [deps: 5]
Task 7: "Implement JWT utilities" [deps: 4]
Task 8: "Implement auth middleware" [deps: 7]
Task 9: "Add auth endpoints" [deps: 6, 8]
Task 10: "Add user management endpoints" [deps: 8]
Task 11: "Integration tests — backend" [deps: 9, 10]
Task 12: "Initialize frontend" [deps: 1] — can parallel with Tasks 4-10
Task 13: "Set up routing and auth context" [deps: 12]
Task 14: "Connect frontend to backend auth" [deps: 9, 13]
Task 15: "Build user profile page" [deps: 14]
Task 16: "End-to-end tests" [deps: 11, 15]
Task 17: "Complete CI pipeline" [deps: 16]
Task 18: "Create PR with documentation" [deps: 17]
```

### Step 4: Execution

**Tasks 1, 2, 3:** Claude Code runs in parallel — scaffolding the project.

**Toolchain used:**
- `Bash`: `npm init`, `npx tsc --init`, `npx prisma init`
- `Write`: `docker-compose.yml`, `Dockerfile`, `.github/workflows/ci.yml`
- `Bash`: Verify Docker containers start, migrations run

**Task 7 (JWT utilities):**
- `Read`: Any relevant existing utility patterns (none in greenfield)
- `Write`: `src/utils/jwt.ts` with generation/verification/refresh functions
- `Write`: `src/utils/jwt.test.ts` (test-first approach)
- `Bash`: `npx jest jwt.test.ts` — verify tests pass

**Pattern repeats** for each task.

---

## Toolchain Used (Complete)

| Tool | When Used |
|---|---|
| Bash | npm/npx commands, docker, git, test runner, linter, type checker |
| Write | New files (utilities, components, config) |
| Edit | Configuration updates, import additions |
| MultiEdit | Multiple related changes in the same file |
| Glob | Find all test files, find all route files |
| Read | Verify file contents before editing |
| TaskCreate/Update | Task lifecycle management |

---

## What the Human Does

| When | Action | Time |
|---|---|---|
| Start | Describe the project, review CLAUDE.md | 20 min |
| After Plan | Review and approve the plan | 15 min |
| After Task 3 | Check Docker setup works locally | 5 min |
| After Tasks 9–10 | Review backend code, run integration tests locally | 30 min |
| After Tasks 14–15 | Review frontend, test in browser | 30 min |
| After Task 17 | Review complete CI run, approve PR | 20 min |

**Total human time: ~2 hours.** Estimated equivalent without Claude Code: 3–5 days for a solo engineer.

---

## Time Estimate

| Task | With Claude Code | Without |
|---|---|---|
| Project setup + Docker | 20 min | 2–3 hours |
| Database schema + migrations | 30 min | 2–4 hours |
| JWT + auth backend | 45 min | 4–6 hours |
| User management endpoints | 30 min | 2–4 hours |
| Frontend + React setup | 40 min | 3–5 hours |
| Integration + E2E tests | 40 min | 4–6 hours |
| CI pipeline | 20 min | 1–3 hours |
| **Total** | **~3.5 hours wall clock** | **18–31 hours** |

*Wall clock includes execution time, not just human time.*

---

## Where It Might Struggle

- **Your specific conventions:** If you have company-specific conventions not in CLAUDE.md, they won't be applied. Invest time in CLAUDE.md before starting.
- **Novel architectural decisions:** If the architecture itself requires design invention, Claude Code will pick a reasonable default that may not match your intent.
- **Database schema evolution:** For complex, highly normalised schemas, the auto-generated Prisma schema may require meaningful manual review.
- **CSS/visual polish:** The UI will be functional but not necessarily polished. Treat Claude Code's frontend output as a starting point for visual work.

---

## Contrast: How You'd Do This With Cursor

With Cursor, you'd work file-by-file, side-by-side with the AI. Each step requires you to be present, reviewing updates in the IDE. The total elapsed *calendar time* is similar to the human time — you can't walk away. For a greenfield project, this produces equivalent quality but takes significantly more human time.
