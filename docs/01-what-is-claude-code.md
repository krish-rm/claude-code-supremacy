# What Is Claude Code?

Claude Code is an AI coding agent published by Anthropic as an npm package (`@anthropic-ai/claude-code`). It runs in the terminal, not in an IDE. It uses Claude as its underlying model and is configured primarily through a 110+ string system prompt embedded in the npm package.

Understanding what Claude Code *is* — at the architectural level — is necessary before evaluating whether it's useful.

---

## The CLI-First Design Choice

Most AI coding tools are built as IDE extensions: they live inside VS Code, JetBrains, or a custom editor fork. Claude Code is built as a CLI tool.

This is not a product limitation — it's an architectural decision with specific consequences:

**CLI-first enables:**
- Headless execution (servers, CI pipelines, overnight runs)
- IDE independence (works with any editor, or no editor)
- Scripting and automation (`CLAUDE_CODE_TASK_LIST_ID=sprint claude`)
- Process supervision (run as a background job, pipe output)
- SSH-natural (run on remote machines over SSH without GUI)

**CLI-first limits:**
- No inline code completions while typing
- No visual file tree integration
- No diff view rendered in an editor
- Steeper learning curve for engineers accustomed to GUI tools

The CLI choice reflects the autonomous-agent identity: an agent that runs unattended doesn't need a GUI. An agent that pairs with a human in real time does.

---

## The npm Package Architecture

Publishing as an npm package has an unintended consequence that benefits the research community: the system prompt is embedded in the JavaScript bundle.

```bash
npm install -g @anthropic-ai/claude-code
# The package installed at:
# ~/.npm/lib/node_modules/@anthropic-ai/claude-code/dist/

# The prompt strings are visible in:
# dist/index.js (minified but extractable)
```

The Piebald-AI project extracts and tracks these strings across versions. This is what makes system prompt analysis of Claude Code significantly more reliable than analysis of tools that deliver their prompts exclusively server-side.

---

## How Claude Code Differs from API Usage of Claude

Using the Claude API directly (via the Anthropic SDK) gives you the base model. Claude Code adds:

1. **System prompt** — 110+ strings encoding SDLC behaviour, tool instructions, and security rules
2. **Tool definitions** — 16 built-in tools with their schemas
3. **Sub-agent system** — 9+ specialised agents with their own prompts
4. **Task persistence** — `~/.claude/tasks/` JSON store
5. **CLAUDE.md integration** — automatic project memory loading
6. **Context compaction** — automatic session compression at context limits

Claude Code is Claude + an engineering system built around it. The model is the same; the scaffolding is the product.

---

## The Project Memory System: CLAUDE.md + Session + Task State

Claude Code maintains three distinct layers of state:

```
PERSISTENT (outlives sessions)
├── CLAUDE.md — project conventions, architecture, gotchas
└── ~/.claude/tasks/ — task list, status, dependency graph

SESSION (within one Claude Code session)
├── Session context — conversation history, file reads, tool results
└── Session compaction summary — compressed state after context limits

EPHEMERAL (within one task execution)
└── Sub-agent context — isolated per-agent context windows
```

This layered state model is what allows Claude Code to resume work across sessions, coordinate agents, and maintain project context without requiring the user to re-establish it each time.

---

## The "Chat Interface" vs. "Engineering Agent" Model

Understanding the distinction between these two interaction models helps clarify where Claude Code sits:

**Chat interface model (ChatGPT, Cursor chat):**
- User sends a message
- AI responds with text (and possibly code)
- User reads, evaluates, copies useful parts
- Repeat

**Engineering agent model (Claude Code):**
- User defines a task
- Agent reads the codebase, forms a plan, gets approval
- Agent executes the plan using tools (read files, write code, run tests)
- Agent verifies results and escalates on failure
- Agent commits, pushes, creates PR
- User reviews the output

The chat model optimises for conversation quality. The agent model optimises for task completion. Claude Code is unambiguously in the second category.

---

## Version History and Stability

Claude Code versions are tracked by the Piebald-AI project across 37 versions. Key milestones:

| Date | Version | Change |
|---|---|---|
| January 23, 2025 | ~v1.5.x | TodoWrite → TaskCreate system upgrade |
| December 1, 2025 | v2.0.56 | Current analysis baseline |
| February 2026 | — | This document written |

The prompt has evolved significantly. Comparisons with Claude Code based on pre-v2.0 prompt data are likely outdated, particularly for task management claims.

> See [`research/changelog.md`](../research/changelog.md) for the version tracking table.  
> See [`architecture/multi-agent-design.md`](../architecture/multi-agent-design.md) for the full sub-agent inventory.  
> See [`sdlc/00-overview.md`](../sdlc/00-overview.md) for the SDLC mapping.
