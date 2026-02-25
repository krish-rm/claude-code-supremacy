# The Claude Code SDLC Loop

Most tool comparisons for AI coding focus on code quality: does the output compile? Are variable names sensible? Does it handle edge cases? These are L1–L3 concerns. Claude Code's architectural advantage lives at L4–L5, which is only visible when you look at the full software development lifecycle — not a single autocomplete event.

This section maps Claude Code's built-in architecture to the Agile SDLC. The mapping is deliberate: it's not that Claude Code "can do agile things" with some prompting. The system prompt explicitly encodes each phase as a distinct capability with its own sub-agents, tool restrictions, and verification loops.

---

## The Full Loop

```
┌─────────────────────────────────────────────────────────────────┐
│                    THE CLAUDE CODE SDLC LOOP                    │
│                                                                 │
│  ┌──────────┐    ┌──────────┐    ┌───────────┐    ┌─────────┐  │
│  │  PLAN    │───▶│  TASK    │───▶│  EXECUTE  │───▶│ VERIFY  │  │
│  │  MODE    │    │  CREATE  │    │   LOOP    │    │  LOOP   │  │
│  └──────────┘    └──────────┘    └───────────┘    └────┬────┘  │
│       │               │               ▲               │        │
│  [Explore +      [Dependency      [16 Tools +      [Pass/Fail] │
│   Plan +          Graph +          Sub-Agents +        │        │
│   Approval]       Priorities]      3-Strike]       ┌───▼────┐  │
│                                                    │  GIT   │  │
│                                                    │  & CI  │  │
│                                                    └────────┘  │
└─────────────────────────────────────────────────────────────────┘
```

---

## Agile → Claude Code Equivalence Table

| Agile Phase | Agile Artefact | Claude Code Equivalent | Mechanism |
|---|---|---|---|
| Discovery | User Story Mapping | Plan Mode — Explore phase | Haiku sub-agent reads codebase, generates understanding |
| Sprint Planning | Sprint Board | Plan Mode — Plan presentation | Structured breakdown, user approval gate |
| Backlog | JIRA/Linear tickets | `TaskCreate` with priorities | Persistent JSON to `~/.claude/tasks/` |
| Dependencies | Blocked/Blocking | `addBlockedBy` in TaskCreate | Hard dependency graph, not advisory |
| Sprint Execution | Dev work | Execution Loop (16 tools) | Read → Change → Verify cycle |
| Code Review | PR review | Verify Plan Reminder + sub-agent | System fires verification after execution |
| QA | Test runs | Test execution in Bash | Automatic test run, fix on failure |
| CI/CD | Pipeline | CI failure recovery loop | Push → read CI output → fix → recommit |
| Release | Git tag + PR | `gh` CLI + HEREDOC commits | Attribution, Co-Authored-By, PR creation |
| Retrospective | Team retro | `open-questions.md` equivalent | Not encoded — this is the human layer |

---

## Why SDLC Framing Matters

When engineers evaluate AI coding tools, they typically test: "Can it write a sorting function?" or "Can it refactor this class?" These are valid but insufficient tests. They measure code generation quality — L1 through L3 capability.

The real question for teams is: **can this tool participate in the full lifecycle of shipping software?**

Shipping software requires:
1. Understanding requirements in the context of an existing codebase
2. Breaking requirements into ordered, tracked work items
3. Executing work with verification at each step
4. Integrating with the quality gate infrastructure (tests, CI, linters)
5. Producing a clean, attributed commit history

Claude Code encodes all five. Most AI coding tools encode one or two. That's the structural difference.

---

## What Each Phase Covers

This section is expanded in dedicated files:

| File | Phase | Key Mechanism |
|---|---|---|
| [`01-plan-mode.md`](01-plan-mode.md) | Discovery + Planning | Two-phase gate with Haiku subagent |
| [`02-task-management.md`](02-task-management.md) | Backlog + Dependencies | TaskCreate, persistent JSON, dependency graphs |
| [`03-execution-loop.md`](03-execution-loop.md) | Sprint Execution | 16 tools, 3-strike linter rule, sub-agent routing |
| [`04-qa-and-verification.md`](04-qa-and-verification.md) | QA + Code Review | Verify Plan Reminder, test execution, fix loops |
| [`05-git-and-cicd.md`](05-git-and-cicd.md) | CI/CD + Release | HEREDOC commits, CI recovery, PR creation |

---

## The Abstraction Layer Argument

| Level | Scope | Examples | Claude Code? | Cursor? | Copilot? |
|---|---|---|---|---|---|
| L1 | Token | Next token prediction | ✅ | ✅ | ✅ |
| L2 | Line | Line completion | ✅ | ✅ | ✅ |
| L3 | File | File-scope refactor | ✅ | ✅ | Partial |
| L4 | Feature | Multi-file feature end-to-end | ✅ | Partial | ❌ |
| L5 | Project | Sprint from plan to CI | ✅ | ❌ | ❌ |

Claude Code is the only tool in the major market that operates reliably at L5. This is not a claim about model quality — Cursor also uses Claude 3.5 Sonnet. It's a claim about system architecture: the surrounding structure that makes L5 execution possible.

---

## The Human Role at Each Phase

A common misconception is that Claude Code "does everything." It doesn't — and the system prompt explicitly enforces human gates:

- **Plan Mode**: User must approve the plan before execution begins
- **Task creation**: User can inspect and modify the dependency graph
- **Execution**: User is notified of tool calls (configurable trust levels)
- **QA failure**: After 3 failed linter attempts, Claude Code stops and escalates
- **CI failure**: System attempts recovery, but notifies human if unresolved

The architecture is designed for **human-in-the-loop supervision**, not unsupervised autonomy. This is an explicit design choice visible in the system prompt.

> See [`docs/03-human-perspective.md`](../docs/03-human-perspective.md) for the full analysis of what changes for engineers working with this system.
