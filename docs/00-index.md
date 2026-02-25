# Documentation Index

This repository contains 31 documentation files across 7 sections. This index helps you navigate based on what you're trying to understand.

---

## If you want to understand what Claude Code is

Start here: **[`docs/01-what-is-claude-code.md`](01-what-is-claude-code.md)**

Then: **[`sdlc/00-overview.md`](../sdlc/00-overview.md)** — The SDLC loop explained

---

## If you want to understand the architecture

| Topic | File |
|---|---|
| Multi-agent system (sub-agents, spawning, orchestration) | [`architecture/multi-agent-design.md`](../architecture/multi-agent-design.md) |
| All 16 tools, categorised | [`architecture/tool-taxonomy.md`](../architecture/tool-taxonomy.md) |
| Context management and CLAUDE.md | [`architecture/context-management.md`](../architecture/context-management.md) |
| Security model and hardcodes | [`architecture/security-model.md`](../architecture/security-model.md) |

---

## If you want to understand the SDLC integration

| Phase | File |
|---|---|
| Full overview (all phases) | [`sdlc/00-overview.md`](../sdlc/00-overview.md) |
| Plan Mode (explore + plan + approval) | [`sdlc/01-plan-mode.md`](../sdlc/01-plan-mode.md) |
| Task management (TaskCreate, dependencies) | [`sdlc/02-task-management.md`](../sdlc/02-task-management.md) |
| Execution loop (tools, 3-strike rule) | [`sdlc/03-execution-loop.md`](../sdlc/03-execution-loop.md) |
| QA and verification | [`sdlc/04-qa-and-verification.md`](../sdlc/04-qa-and-verification.md) |
| Git and CI/CD integration | [`sdlc/05-git-and-cicd.md`](../sdlc/05-git-and-cicd.md) |

---

## If you want to understand the system prompt architecture

| Topic | File |
|---|---|
| Why system prompts matter | [`prompts/00-why-prompts-matter.md`](../prompts/00-why-prompts-matter.md) |
| Claude Code's 110+ string anatomy | [`prompts/claude-code-prompt-anatomy.md`](../prompts/claude-code-prompt-anatomy.md) |
| Cursor's single prompt anatomy | [`prompts/cursor-prompt-anatomy.md`](../prompts/cursor-prompt-anatomy.md) |
| What the prompts reveal (analysis) | [`prompts/what-the-prompts-reveal.md`](../prompts/what-the-prompts-reveal.md) |
| How to write a CLAUDE.md | [`prompts/claude-md-guide.md`](../prompts/claude-md-guide.md) |

---

## If you're evaluating Claude Code vs. alternatives

| Comparison | File |
|---|---|
| Methodology and sources | [`comparisons/00-methodology.md`](../comparisons/00-methodology.md) |
| vs. Cursor | [`comparisons/vs-cursor.md`](../comparisons/vs-cursor.md) |
| vs. Gemini Code Assist | [`comparisons/vs-google.md`](../comparisons/vs-google.md) |
| vs. GitHub Copilot | [`comparisons/vs-github-copilot.md`](../comparisons/vs-github-copilot.md) |
| vs. Windsurf | [`comparisons/vs-windsurf.md`](../comparisons/vs-windsurf.md) |
| Full capability matrix (all tools) | [`comparisons/benchmark-matrix.md`](../comparisons/benchmark-matrix.md) |

---

## If you want practical walkthroughs

| Scenario | File |
|---|---|
| When to use Claude Code | [`use-cases/00-overview.md`](../use-cases/00-overview.md) |
| Greenfield project | [`use-cases/greenfield-project.md`](../use-cases/greenfield-project.md) |
| Legacy codebase | [`use-cases/legacy-codebase.md`](../use-cases/legacy-codebase.md) |
| Debugging complex issues | [`use-cases/bug-hunt.md`](../use-cases/bug-hunt.md) |
| Feature sprint | [`use-cases/feature-sprint.md`](../use-cases/feature-sprint.md) |
| Team coordination | [`use-cases/team-coordination.md`](../use-cases/team-coordination.md) |

---

## If you're concerned about the productivity claims

Read: **[`docs/02-the-10x-myth.md`](02-the-10x-myth.md)** — Honest analysis of what "10x" actually means

---

## If you want to understand what changes for engineers

Read: **[`docs/03-human-perspective.md`](03-human-perspective.md)** — Time allocation, new skills, uncomfortable questions

---

## If you want to get started

Read: **[`docs/04-getting-started.md`](04-getting-started.md)** — Installation, setup, first session

---

## Research and Methodology

| Topic | File |
|---|---|
| All sources and reliability ratings | [`research/sources.md`](../research/sources.md) |
| Version changelog | [`research/changelog.md`](../research/changelog.md) |
| What we don't know | [`research/open-questions.md`](../research/open-questions.md) |
