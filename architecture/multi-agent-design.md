# Multi-Agent Architecture

Claude Code's multi-agent system is the feature that most clearly places it at the L5 (project) scope of software engineering. This document describes each sub-agent, how they interact, and how the orchestration logic works.

---

## The Orchestration Model

Claude Code uses a hub-and-spoke model: a primary orchestrating agent (the main Claude Code session) spawns specialised sub-agents via the `Task` tool. Each sub-agent:

1. Receives a targeted prompt (its own mini-system-prompt)
2. Has access to a defined, restricted toolset
3. Operates in an independent context window
4. Returns a structured result to the orchestrating agent

Sub-agents do not communicate with each other directly — all coordination goes through the orchestrating agent. This is a deliberate architectural constraint: it simplifies reasoning about state and prevents cascading agent failures.

```
ORCHESTRATING AGENT (primary session)
         │
         ├── Task("Explore codebase") ──▶ Explore Sub-Agent
         │                                       │
         │                                [reads code]
         │                                       │
         │◀──────────────────── findings ────────┘
         │
         ├── Task("Generate CLAUDE.md") ──▶ CLAUDE.md Creator
         │
         ├── Task("Review PR comments") ──▶ PR Comments Agent
         │
         └── Task("Verify implementation") ──▶ Verification Agent
```

---

## Sub-Agent Inventory

### 1. Explore Sub-Agent
- **Token count:** 516
- **Model:** Claude Haiku (cost-optimised)
- **Toolset:** Read, Glob, Grep, LS (read-only only)
- **Purpose:** Codebase discovery before Plan Mode
- **Inputs:** Project root path, task description
- **Outputs:** File structure summary, relevant file list, architectural observations
- **When spawned:** At Plan Mode entry, before plan creation

The Explore sub-agent's read-only toolset is enforced at the prompt level — write tools simply don't appear in its tool list. This prevents premature modification before understanding exists.

### 2. Plan Mode Enhanced Sub-Agent
- **Token count:** 633
- **Model:** Claude Sonnet
- **Toolset:** Structured output tools
- **Purpose:** Produce a structured plan document from Explore findings
- **Inputs:** Explore sub-agent output, task description, CLAUDE.md content
- **Outputs:** Structured plan (files to modify, changes per file, task order, risk assessment)
- **When spawned:** After Explore completes, before plan presentation to user

### 3. Task Tool Sub-Agent
- **Token count:** 294
- **Model:** Claude Sonnet
- **Toolset:** TaskCreate, TaskGet, TaskUpdate, TaskList
- **Purpose:** Translate a plan into a structured task list with dependencies
- **Inputs:** Approved plan from Plan Mode
- **Outputs:** Task list entries in ~/.claude/tasks/
- **When spawned:** After user approves plan

### 4. Agent Creation Architect
- **Token count:** 1111
- **Model:** Claude Sonnet/Opus
- **Toolset:** Full toolset
- **Purpose:** Design complex multi-agent workflows for large features
- **Inputs:** Complex task requiring multiple parallel workstreams
- **Outputs:** Agent coordination plan, task assignments, dependency structure
- **When spawned:** For L5-scope tasks that require parallel agent coordination

The Agent Creation Architect has the largest token count of any sub-agent (1111 tokens) because designing multi-agent coordination requires substantial context and reasoning.

### 5. CLAUDE.md Creator
- **Token count:** 384
- **Model:** Claude Sonnet
- **Toolset:** Read, Glob, Grep, LS (read-only)
- **Purpose:** Generate project memory documentation
- **Inputs:** Project root path
- **Outputs:** CLAUDE.md draft with project structure, tech stack, conventions
- **When spawned:** First session in new project, or explicit request

### 6. Status Line Sub-Agent
- **Token count:** 993
- **Model:** Claude Sonnet
- **Toolset:** TaskList, TaskGet
- **Purpose:** Generate real-time status display for the terminal interface
- **Inputs:** Current task state, active operations
- **Outputs:** Formatted status string for terminal display
- **When spawned:** Continuously during execution (on status update events)

The Status Line sub-agent's relatively high token count (993) reflects the complexity of generating contextually meaningful status messages from varied task states.

### 7. PR Comments Agent
- **Token count:** 404
- **Model:** Claude Sonnet
- **Toolset:** WebFetch (to read PR), Read (to read code)
- **Purpose:** Review incoming PR comments and generate response code
- **Inputs:** PR URL, comment thread
- **Outputs:** Code changes addressing the comments, response text
- **When spawned:** When user directs Claude Code to address PR review feedback

### 8. Security Review Sub-Agent
- **Token count:** Not documented in source (proprietary)
- **Model:** Claude Sonnet
- **Toolset:** Read, Grep (read-only scan)
- **Purpose:** Security-focused code review for sensitive changes
- **Inputs:** Changed files involving auth, credentials, or elevated permissions
- **Outputs:** Security assessment, flagged issues, remediation suggestions
- **When spawned:** Automatically on changes involving security-sensitive patterns

### 9. Session Compaction Agent
- **Token count:** Variable
- **Model:** Claude Haiku
- **Toolset:** Read, Write (for compaction summary)
- **Purpose:** Compress session context when approaching limits
- **Inputs:** Full session history
- **Outputs:** Compressed summary preserving: task state, key decisions, file changes, next steps
- **When spawned:** When session context approaches the model's context limit

---

## Parallel Execution Pattern

For independent tasks (no dependency relationship), the orchestrating agent can spawn multiple sub-agents simultaneously:

```
Orchestrating Agent
├── Task("Implement payment service")  ──▶ Sub-Agent A [works independently]
└── Task("Implement cart API")         ──▶ Sub-Agent B [works independently]
         │                                        │
         │                                        │
         ◀──────── both complete ─────────────────┘
         │
Task("Implement checkout API") [now unblocked]
```

The task dependency graph (from `addBlockedBy`) determines which tasks can run in parallel. The orchestrating agent reads the graph, identifies independent work items, and spawns sub-agents for each.

---

## Multi-Agent Team Coordination

When two engineers both run Claude Code with `CLAUDE_CODE_TASK_LIST_ID=shared-project`:

```
Engineer A's Session:          Engineer B's Session:
       │                              │
TaskList → picks "Auth API"    TaskList → picks "Cart API"
Status: in_progress            Status: in_progress
[works autonomously]           [works autonomously]
Status: completed              Status: completed
       │                              │
       └──────────────────────────────┘
              Checkout API now unblocked
              First session to pick it gets it
```

The shared task list in `~/.claude/tasks/` is the coordination mechanism. Tasks marked `in_progress` by one session are implicitly "claimed." No additional coordination protocol is needed.

> See [`use-cases/team-coordination.md`](../use-cases/team-coordination.md) for a full walkthrough of this pattern.
