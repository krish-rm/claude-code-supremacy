# The CLAUDE.md Guide

`CLAUDE.md` is Claude Code's project memory system. It's a markdown file in your project root that persists context across sessions, accelerates Plan Mode exploration, and communicates project conventions to every sub-agent that runs.

A well-written `CLAUDE.md` is arguably the highest-leverage configuration step you can take when adopting Claude Code. It can reduce Plan Mode exploration time by 50–70% on large codebases.

---

## What CLAUDE.md Is

`CLAUDE.md` is read at the start of every Claude Code session and injected into the context of sub-agents that need it. It serves as:

1. **Project architecture documentation** — "this is what this codebase is and how it's structured"
2. **Convention reference** — "we name things this way, we test this way, we don't do X"
3. **Environment documentation** — "here's how to run this locally, here's the CI setup"
4. **Known constraints** — "don't modify these files, always use this library for that purpose"

The Explore sub-agent reads `CLAUDE.md` before scanning the codebase. A good `CLAUDE.md` means the Explore phase starts with high confidence context rather than having to infer everything from the code.

---

## The CLAUDE.md Creation Sub-Agent

Claude Code includes a dedicated sub-agent for generating `CLAUDE.md` files (384 tokens, Sonnet). When you run Claude Code in a new project, or explicitly request a `CLAUDE.md` refresh:

1. The sub-agent scans the codebase (Explore toolset: read, glob, grep, ls)
2. Identifies project structure, frameworks, test setup, CI configuration
3. Infers conventions from existing code patterns
4. Generates a `CLAUDE.md` draft
5. Presents it to you for review and editing before saving

The human review step is important — automatically generated `CLAUDE.md` will have gaps (what the code does isn't always what it *should* do, and conventions should be prescriptive, not just descriptive).

---

## Template: All Sections

```markdown
# CLAUDE.md

## Project Overview
[2-3 sentence description of what this project is and what it does.
Include: the domain, the primary user or customer, and the team size if relevant.]

## Architecture
[How is the project structured? What are the major modules/services?
Include a brief directory tree if helpful.]

## Technology Stack
- **Language:** [Primary language and version, e.g. TypeScript 5.x]
- **Runtime/Framework:** [e.g. Node.js 20, React 18, Django 4.x]
- **Database:** [e.g. PostgreSQL 16 with Prisma ORM]
- **Infrastructure:** [e.g. Docker, AWS ECS, GitHub Actions CI]
- **Testing:** [e.g. Jest + React Testing Library, pytest]

## Development Setup
```bash
# Clone and install
git clone ...
npm install

# Environment setup
cp .env.example .env
# Edit .env — see .env.example for required variables

# Database
npm run db:migrate
npm run db:seed

# Run locally
npm run dev
```

## Testing
```bash
# Unit tests
npm test

# Integration tests
npm run test:integration

# Coverage
npm run test:coverage
```

## CI/CD
[Describe the CI setup briefly. What does the pipeline check?
What environments exist (dev/staging/prod)?]

## Conventions

### Naming
- Files: kebab-case (e.g. `user-service.ts`)
- Classes: PascalCase (e.g. `UserService`)
- Functions/variables: camelCase (e.g. `getUserById`)
- Database tables: snake_case (e.g. `user_profiles`)

### Code Style
- Linter: ESLint with [config name]
- Formatter: Prettier (see `.prettierrc`)
- TypeScript: strict mode enabled, no `any` types
- Imports: absolute imports from `src/`, no relative `../../`

### Testing
- Test files: alongside source (`*.test.ts`)
- Unit tests for: all service-layer functions
- Integration tests for: all API endpoints
- Minimum coverage: 80% (enforced in CI)

### Git
- Branch naming: `feat/`, `fix/`, `chore/` prefixes
- Commit format: Conventional Commits (`feat:`, `fix:`, `docs:`)
- PRs: require at least 1 reviewer, all CI checks passing

## Do Not Touch
[List files or directories that should not be modified without explicit discussion.
Examples: generated files, migration files if using a specific workflow, etc.]

- `src/generated/` — auto-generated from GraphQL schema, regenerate with `npm run codegen`
- `migrations/` — never edit migration files, create new ones instead

## Known Issues / Gotchas
[Anything that would confuse a developer (or AI agent) who doesn't know the history.
Examples: technical debt decisions, workarounds, planned refactors.]

- The `legacy-auth.ts` file is to be replaced in Q1 — don't add new functionality there
- Environment variable `FEATURE_FLAG_X` must be set to `true` in dev (default is false)
```

---

## Real Example: TypeScript/Python Full-Stack App

```markdown
# CLAUDE.md — Acme Invoice Platform

## Project Overview
A B2B SaaS platform for invoice management and automated AR workflows.
Serves 500+ enterprise customers. Core team: 3 engineers.

## Architecture
Monorepo with three packages:
- `apps/web` — React 18 frontend (TypeScript)
- `apps/api` — FastAPI backend (Python 3.11)
- `apps/jobs` — Celery task worker (Python 3.11)

Shared packages in `packages/` (TypeScript utilities, shared types).

## Technology Stack
- **Frontend:** TypeScript 5.x, React 18, Vite 5, TanStack Query
- **Backend:** Python 3.11, FastAPI, SQLModel, Alembic
- **Database:** PostgreSQL 16 (production), SQLite (tests)
- **Queue:** Redis + Celery
- **Infrastructure:** Docker Compose (local), AWS ECS + RDS (production)
- **CI:** GitHub Actions (`/.github/workflows/`)

## Development Setup
```bash
# Start everything
docker compose up -d

# Frontend (hot reload on :5173)
cd apps/web && npm install && npm run dev

# Backend (auto-reload on :8000)
cd apps/api && pip install -r requirements.txt
uvicorn main:app --reload

# Run database migrations
cd apps/api && alembic upgrade head
```

## Testing
```bash
# Frontend
cd apps/web && npm test        # unit
npm run test:e2e               # Playwright e2e (requires running backend)

# Backend
cd apps/api && pytest          # uses SQLite, no Docker needed
pytest --cov=app tests/        # with coverage
```

## Conventions

### Code Style
- Frontend: ESLint + Prettier, strict TypeScript (no any)
- Backend: ruff for linting, black for formatting, mypy for types
- Backend types: always use Pydantic models for request/response; SQLModel for DB models

### API Design
- REST: `GET /invoices`, `POST /invoices`, `PATCH /invoices/{id}`, `DELETE /invoices/{id}`
- Never use plural for sub-resources: `GET /invoices/{id}/line-items` not `.../line_items`
- All responses: `{"data": ..., "meta": {...}}` wrapper format (see `app/schemas/base.py`)

### Testing
- Backend: tests in `tests/`, mirrors `app/` structure
- Frontend: test files alongside source (`*.test.tsx`)
- Mocks: use `tests/factories.py` for backend test data (don't create raw dicts)

## Do Not Touch
- `apps/api/migrations/` — never edit existing files, create new with alembic
- `packages/types/generated/` — regenerate with `npm run types:generate`

## Gotchas
- Celery workers don't hot-reload; restart with `docker compose restart worker` after changes
- `STRIPE_WEBHOOK_SECRET` must match the actual Stripe webhook secret; use the Stripe CLI locally
- Email sending is disabled in dev by default (`EMAIL_ENABLED=false`); set to `true` to test
```

---

## Best Practices

**Do include:**
- Actual commands that work (test them before committing `CLAUDE.md`)
- Gotchas that cost time to discover (these are the most valuable entries)
- The "do not touch" list — this prevents expensive mistakes
- Your actual naming convention rules, not aspirational ones

**Do not include:**
- Documentation that duplicates what's in README for humans (keep `CLAUDE.md` agent-focused)
- Overly broad constraints ("always write clean code") — be specific
- Outdated information — stale `CLAUDE.md` is worse than no `CLAUDE.md`
- Sensitive data, credentials, or internal URLs

**Keep it current:** Update `CLAUDE.md` when architecture changes meaningfully. A `CLAUDE.md` that's 3 major refactors out of date will actively mislead sub-agents.
