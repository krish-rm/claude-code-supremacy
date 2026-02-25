# Context Management

One of Claude Code's practical advantages for large projects is its explicit context management system. This document explains how it handles context limits, what it preserves when compressing, and how `CLAUDE.md` functions as persistent project memory.

---

## The Context Limit Problem

All language models have a finite context window — the amount of text the model can "see" at once. In a long coding session involving many file reads, multiple tool calls, and extended task execution, the session context can grow large enough to approach or exceed this limit.

Without explicit management, two things can happen:
1. Older context silently drops out as new content arrives (sliding window)
2. The session fails with a context length error

Claude Code addresses this with a **session compaction** mechanism.

---

## Session Compaction

When the session context approaches the model's limit, the Session Compaction sub-agent fires:

```
Session approaching context limit
         │
         ▼
Session Compaction Sub-Agent (Haiku — cheap, fast)
  Input: Full session history
  Process: Extract and preserve:
    ├── All task states (pending/in_progress/completed)
    ├── Key architectural decisions made
    ├── Files modified and how
    ├── Current error state (if any)
    ├── Next planned steps
    └── Open blockers or questions
  Output: Compressed summary (~1,000-2,000 tokens)
         │
         ▼
Session continues with compressed context + new work
```

The compaction is designed to preserve the **state** of the work, not the **history** of the conversation. Tool call results, file contents already read, and conversational exchanges are dropped or summarised. Task state, decisions, and next steps are preserved.

**Token economics:** Using Haiku for compaction is deliberate. The compaction operation reads a lot of text (full session history) but produces relatively simple output (structured summary). Haiku is well-suited to this transformation at minimal cost.

---

## What Gets Preserved in Compaction

| Information Type | Preserved? | Format |
|---|---|---|
| Active task states | ✅ | From ~/.claude/tasks/ (not in context) |
| Completed task summaries | ✅ | Brief description |
| In-progress task state | ✅ | Current step, what was done |
| Key decisions made | ✅ | Bullet list |
| Files modified (list) | ✅ | File paths + brief change description |
| File contents read | ❌ | Re-read from disk when needed |
| Full conversation history | ❌ | Summarised |
| Tool call results (old) | ❌ | Summarised or dropped |
| Error context (current) | ✅ | Current error state preserved |
| Next planned steps | ✅ | From task list or plan |

Note that **task state is always correct regardless of compaction** because tasks persist to `~/.claude/tasks/` — they're not in the session context. The task list is the source of truth; compaction only affects the conversational context.

---

## CLAUDE.md as Persistent Project Memory

`CLAUDE.md` solves a different but related problem: not context within a session, but context across sessions.

When you close Claude Code and reopen it:
- Session context is gone
- Tasks in `~/.claude/tasks/` remain
- `CLAUDE.md` remains (it's a file in your project)

This means a new session can start with meaningful project context without repeating the Explore phase. The sub-agent reads `CLAUDE.md`, understands the project architecture, and can immediately work productively.

**The compounding effect:** With a good `CLAUDE.md`, the cost of starting a new Claude Code session on an existing project is near-zero. Without it, every session starts the Explore phase from scratch.

```
WITHOUT CLAUDE.md:
New session → Explore phase (read 50+ files) → understand project → productive work
[~10-15 minutes before productive work begins on a large codebase]

WITH CLAUDE.md:
New session → Read CLAUDE.md → selective Explore (only unknowns) → productive work
[~2-3 minutes before productive work begins]
```

---

## Session Resumption Pattern

For multi-session work on a large feature:

1. **Session 1:** Plan Mode → TaskCreate → begin execution (tasks 1-3 completed)
2. **Session 1 ends** (user closes Claude Code, end of work day)
3. **Session 2:** Claude Code reads:
   - `CLAUDE.md` (project context)
   - `~/.claude/tasks/` (task state — tasks 1-3 completed, task 4 pending)
   - Most recent compaction summary (if session was long enough to compact)
4. Session 2 picks up at task 4 without any context recovery work from the user

This is not automatic — the user needs to tell Claude Code to "continue the [feature] work." But the scaffolding for resumption is in place.

---

## Token Economics: Why Haiku for Exploration

The consistent use of Haiku for high-volume, read-heavy operations (Explore sub-agent, Session Compaction) has a meaningful cost impact at scale:

| Operation | If using Sonnet | If using Haiku | Savings |
|---|---|---|---|
| Explore phase (100 file reads) | ~$0.50–2.00 | ~$0.05–0.20 | ~90% |
| Session compaction (per event) | ~$0.10–0.40 | ~$0.01–0.05 | ~90% |
| CLAUDE.md generation | ~$0.05–0.20 | ~$0.01–0.03 | ~85% |

For a team running 10 Claude Code sessions daily with frequent context compaction, model routing can reduce API costs by 40–60% compared to routing all operations through Sonnet.

> See [`architecture/multi-agent-design.md`](multi-agent-design.md) for the full sub-agent breakdown with token counts.
