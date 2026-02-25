# Getting Started with Claude Code

This guide covers installation, setup, and your first productive session — from zero to having a Plan Mode session running in your own codebase.

---

## Prerequisites

- Node.js 18 or higher
- An Anthropic API key with access to Claude Sonnet
- npm 8 or higher
- A Unix-like terminal (macOS, Linux, or WSL2 on Windows)

---

## Step 1: Installation

```bash
npm install -g @anthropic-ai/claude-code
```

Verify the installation:

```bash
claude --version
```

---

## Step 2: API Key Setup

```bash
export ANTHROPIC_API_KEY=sk-ant-your-key-here
```

To make this permanent, add it to your shell profile:

```bash
# ~/.zshrc or ~/.bashrc
echo 'export ANTHROPIC_API_KEY=sk-ant-your-key-here' >> ~/.zshrc
source ~/.zshrc
```

**Important:** Treat your API key like a password. Do not commit it to version control.

---

## Step 3: First Run

Navigate to a project directory (any existing codebase works):

```bash
cd /path/to/your/project
claude
```

Claude Code starts, reads the directory, and shows a prompt. If you don't have a `CLAUDE.md` yet, it will offer to generate one.

---

## Step 4: Generate Your First CLAUDE.md

In the Claude Code session:

```
> Generate a CLAUDE.md for this project
```

Claude Code will:
1. Scan your codebase (read-only, using the Explore toolset)
2. Identify your stack, structure, and patterns
3. Generate a `CLAUDE.md` draft and present it to you
4. Save it to your project root on your approval

**Review this carefully before saving.** The auto-generated `CLAUDE.md` is a starting point — add your team's specific conventions, gotchas, and any "do not touch" files that Claude Code couldn't infer from the code.

> See [`prompts/claude-md-guide.md`](../prompts/claude-md-guide.md) for the complete guide to writing a great CLAUDE.md.

---

## Step 5: Your First Plan Mode Session

```
> Plan Mode: [describe what you want to build]
```

Example:

```
> Plan Mode: Add a rate limiting middleware to the Express API.
  Use Redis for the counter store. Limit to 100 requests per user per minute.
  Apply to all /api routes. Add appropriate error responses.
```

Claude Code will:
1. Run the Explore sub-agent to read your codebase
2. Present a structured plan with files to modify and steps to take
3. Wait for your approval before making any changes

Review the plan carefully. Check:
- Are the files it plans to modify the right ones?
- Does it understand your existing middleware pattern?
- Are the steps in the right order?

Type `approve` or provide corrective feedback before execution begins.

---

## Step 6: Your First TaskCreate

For longer work, use the task system:

```
> Create tasks for the rate limiting feature with the plan we just approved
```

Or in a new session, for a full sprint:

```
> Create a task list for this sprint's work:
  1. Rate limiting middleware (Redis-backed, 100 req/user/min)
  2. Request logging (structured JSON, log level configurable)
  3. API key authentication (replace session auth on /api routes)
  4. Swagger documentation for all routes
```

Claude Code creates a `TaskCreate` call for each item, identifies dependencies (e.g., API key auth may need to be done before Swagger docs), and persists the list to `~/.claude/tasks/`.

View the task list at any time:

```
> TaskList
```

Or use `Ctrl+T` to open the task panel.

---

## Common Beginner Mistakes

### 1. Skipping CLAUDE.md

Starting Claude Code without a `CLAUDE.md` means every session rediscovers your codebase from scratch. This is slow and increases the risk of mistakes (getting conventions wrong). Write a good `CLAUDE.md` before your first real task.

### 2. Approving plans without reading them

The plan approval gate exists for a reason. Take 5 minutes to read the plan. Check: do the files it plans to modify match your mental model of where this feature should live? Is anything obviously wrong?

### 3. Using Claude Code for tasks that are too small

If you can implement something in 10 minutes yourself, Claude Code's overhead (Explore → Plan → Approve → Execute) may not be worth it. For small tasks, just use Cursor or your own implementation. Claude Code's value is in L4–L5 tasks.

### 4. Not checking in on CI recovery

When Claude Code is recovering from CI failures autonomously, check in occasionally. If it's been attempting the same fix repeatedly and CI keeps failing, the 3-strike rule will escalate — but you can save time by checking in proactively after 15–20 minutes of autonomous CI recovery.

### 5. Running without configuring API key costs

Claude Code uses API tokens. Without budget monitoring, a long overnight run against a large codebase can generate meaningful API costs. Set up Anthropic usage alerts before running extended autonomous sessions.

---

## Quick Command Reference

| Command | What it does |
|---|---|
| `claude` | Start Claude Code in current directory |
| `claude --model opus` | Start with Claude Opus (for complex architectural work) |
| `claude --no-permissions` | Restricted mode (read-only, no file writes) |
| `CLAUDE_CODE_TASK_LIST_ID=name claude` | Start with a shared task list |
| `Ctrl+T` | Open task panel |
| `Ctrl+C` | Interrupt current operation |

---

## Next Steps

Once you're comfortable with the basics:

1. Read [`sdlc/01-plan-mode.md`](../sdlc/01-plan-mode.md) for a deep dive on Plan Mode and how to write better task descriptions.

2. Read [`prompts/claude-md-guide.md`](../prompts/claude-md-guide.md) for the full guide to building a CLAUDE.md that maximises Claude Code's effectiveness in your codebase.

3. Try the [`use-cases/feature-sprint.md`](../use-cases/feature-sprint.md) walkthrough for your next multi-story sprint.

4. Set up MCP servers if your team uses internal tools or databases that aren't accessible via the standard tool set.
