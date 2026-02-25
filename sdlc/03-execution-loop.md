# The Execution Loop

Once Plan Mode completes and the user approves, Claude Code enters its execution loop. This is the phase where the abstract plan becomes concrete code, and where the system's architectural depth becomes most visible.

---

## The Core Loop

```
┌─────────────────────────────────────────────────────────┐
│                    EXECUTION LOOP                        │
│                                                         │
│   READ (understand current state)                       │
│      │                                                  │
│      ▼                                                  │
│   CHANGE (apply modification)                           │
│      │                                                  │
│      ▼                                                  │
│   VERIFY (run tests/linter/type-check)                  │
│      │                                                  │
│      ├──▶ PASS ──▶ Next task (TaskUpdate: completed)    │
│      │                                                  │
│      └──▶ FAIL ──▶ Diagnose ──▶ Fix ──▶ VERIFY again   │
│                         │                               │
│                    [3 failures]                         │
│                         │                               │
│                         ▼                               │
│                   Escalate to human                     │
└─────────────────────────────────────────────────────────┘
```

The loop is not executed once per task. It fires at the sub-file level: for each meaningful change, a read-change-verify cycle runs. This is what makes Claude Code's output substantially more reliable than tools that write code without intermediate verification.

---

## The 16 Built-in Tools

The full toolset, categorised by function:

### Codebase Intelligence (Read)
| Tool | Purpose |
|---|---|
| `Read` | Read file contents |
| `Glob` | Find files by pattern |
| `Grep` | Search file contents with regex |
| `LS` | List directory contents |

### Modification
| Tool | Purpose |
|---|---|
| `Write` | Create new files |
| `Edit` | Edit existing files (targeted diff) |
| `MultiEdit` | Multiple edits in one operation |
| `NotebookEdit` | Edit Jupyter notebook cells |
| `NotebookRead` | Read Jupyter notebook cells |

### Execution
| Tool | Purpose |
|---|---|
| `Bash` | Run shell commands (tests, linters, git, etc.) |

### Web
| Tool | Purpose |
|---|---|
| `WebSearch` | Search the web |
| `WebFetch` | Fetch a specific URL |

### Task and Agent
| Tool | Purpose |
|---|---|
| `TodoWrite` | Legacy simple task list (still present) |
| `TaskCreate` / `TaskGet` / `TaskUpdate` / `TaskList` | Full task management system |
| `Task` | Spawn a sub-agent with a specific prompt and toolset |

The `Task` tool is the most structurally important item in the list — it is what gives Claude Code its multi-agent capability. When Claude Code invokes `Task`, it spawns a new agent instance with a focused prompt, a restricted toolset, and an independent context window. Results are returned to the orchestrating agent.

---

## The 3-Strike Linter Rule

The system prompt encodes an explicit stop-and-escalate rule: if a linter, type-checker, or test suite fails **three consecutive times** on the same issue, Claude Code stops attempting to fix it and surfaces the problem to the human.

This exists for a specific reason: AI systems can enter fix loops where each "fix" introduces a new error, cycling indefinitely without making progress. The 3-strike rule breaks this loop and preserves human time. An indefinitely looping AI is worse than an AI that acknowledges it cannot resolve an issue.

The rule applies to:
- Linter errors (ESLint, Pylint, etc.)
- Type errors (TypeScript `tsc`, mypy)
- Test failures (jest, pytest, etc.)

After 3 strikes: Claude Code surfaces the specific error, what was attempted in each of the 3 tries, and its best hypothesis about the root cause. The human can then intervene with information the system doesn't have (e.g., "this is expected — that linter rule is misconfigured").

Cursor has the same 3-retry rule, per its leaked system prompt. The shared pattern suggests this is a converged best practice for agentic coding systems, not a Claude Code-specific innovation.

---

## Sub-Agent Model Routing

Claude Code uses different models for different phases of execution, based on capability requirements:

| Phase | Model | Rationale |
|---|---|---|
| Codebase exploration | Haiku | High volume, read-only, low complexity |
| Standard execution | Sonnet | Primary workhorse — ability vs. cost balance |
| Complex design decisions | Opus | High-stakes architectural choices |
| PR comment review | Sonnet (404-token sub-agent) | Structured output, moderate complexity |
| Status line generation | Sonnet (993-token sub-agent) | Real-time, must be fast |
| Security review | Sonnet (security sub-agent) | Specialised prompt, not maximum model |

The model routing is encoded in the sub-agent definitions, not determined dynamically per-request. This means the routing behaviour is predictable and auditable.

---

## The Verify Plan Reminder

After execution of a significant change, the system prompt fires a "Verify Plan Reminder" — an internal prompt injected into the context that asks: "Have you followed all the steps in the plan? Have you verified the changes against the acceptance criteria?"

This is a mechanical guard against the "I'm done" premature completion bias that AI systems exhibit. The agent must explicitly answer the verification questions before marking a task complete.

The reminder enforces:
1. Test execution (not just code writing)
2. Cross-checking against the original plan
3. Confirmation that acceptance criteria are met

---

## The Read-Before-Write Principle

The system prompt explicitly instructs Claude Code to read files before editing them. This is encoded in the tool descriptions, not just in general instructions:

- Before `Edit`, the agent reads the current file state
- Before `Write`, the agent checks if the file already exists
- Before `Bash`, the agent reasons about what the command will do

This prevents a common failure mode: overwriting a file that has been modified since the plan was created, or writing to a location that already has conflicting content.

---

## MCP Server Extension

The 16 built-in tools can be extended via Model Context Protocol (MCP) servers. An MCP server exposes additional tools that Claude Code can invoke as if they were native. Examples:

- Database introspection tools (read schema directly)
- CI/CD system tools (trigger pipelines, read build logs)
- Third-party API connectors
- Custom internal tooling

This extensibility means the "16 tools" number is a floor, not a ceiling, for teams that invest in MCP configuration.

> See [`architecture/tool-taxonomy.md`](../architecture/tool-taxonomy.md) for the full per-tool breakdown with behavioural details from the system prompt.
