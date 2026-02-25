# Cursor System Prompt Anatomy

Cursor's system prompt was extracted and published in the [x1xhlol/system-prompts-and-models-of-ai-tools](https://github.com/x1xhlol/system-prompts-and-models-of-ai-tools) repository through community analysis, including direct prompt injection testing. This document provides a structural breakdown of what was found.

**Confidence rating: MODERATE** — The extraction was corroborated by community testing but not from a deterministic source like an npm package.

---

## The Single Prompt Structure

Unlike Claude Code's 110+ modular strings, Cursor uses a single system prompt assembled at session start. Approximate length: ~3,000 tokens. Structure:

```
Cursor System Prompt (~3,000 tokens)
├── 1. Identity and Context
├── 2. Communication Rules
├── 3. Tool Usage Guidelines
├── 4. Code Change Instructions
├── 5. Debugging Instructions
├── 6. External API Instructions
└── 7. Available Tools (10 tools)
```

---

## Section 1: Identity and Context

The opening establishes Cursor's identity and operating context:

> "You are a powerful agentic AI coding assistant, powered by Claude 3.5 Sonnet. You operate exclusively in Cursor, the world's best IDE. You are pair programming with a USER to solve their coding task."

Three key design decisions visible in this opening:
1. **Model identity disclosed** — "powered by Claude 3.5 Sonnet" (at time of extraction)
2. **IDE-locked** — "operates exclusively in Cursor" — environmental constraint
3. **Pair programmer framing** — "pair programming with a USER" — collaboration model, not autonomy

---

## Section 2: Communication Rules

Cursor's communication rules are notable for what they prohibit:

- **Never lie to the user or make up information**
- **Do not refer to tool names in responses to the USER** — tool names are hidden
- **Cursor does not use markdown** in LLM responses (plain text output)
- Be direct, not overly apologetic ("apologise less")
- Keep explanations focused; verbose explanation is discouraged

The "do not refer to tool names" rule is architecturally significant. Cursor actively hides its internal mechanics from the user. This is consistent with the pair-programmer framing — a human pair doesn't narrate their IDE actions, they just produce results. The tradeoff: less transparency about what's happening internally.

---

## Section 3: Tool Usage Guidelines

How and when to use tools, with specific ordering guidance. Key instruction:

> "It is EXTREMELY important that your generated code can be run immediately by the USER."

This instruction (emphasis in original) reveals the primary quality criterion for Cursor: immediacy. Code that runs right now, in the user's environment. This is the right criterion for a pair programmer working alongside a human who wants to see results quickly.

---

## Section 4: Code Change Instructions

Guidance on how to make code modifications:

- Prefer targeted edits over full rewrites
- Never leave placeholder comments ("// TODO: implement this")
- Ensure imports are present and correct
- Handle edge cases explicitly
- Code must be complete and immediately runnable

These rules are oriented around the user's immediate experience: they want to take the code, paste it or apply it, and have it work without additional steps.

---

## Section 5: Debugging Instructions

Cursor's debugging instructions emphasise:

- Identify the root cause, not just the symptom
- Check for related issues that might surface after fixing the primary one
- The 3-retry rule: if linter/type errors persist after 3 attempts, escalate to the user

The 3-retry rule is the same as Claude Code's 3-strike rule — converged best practice for agentic systems.

---

## Section 6: External API Instructions

Instructions for handling third-party API integrations:

- Always check official documentation first
- Handle rate limiting and error responses explicitly
- Do not hardcode credentials — always use environment variables

The credential handling rule is consistent with Claude Code, though less structurally enforced (advisory vs. hardcoded).

---

## Section 7: Available Tools (10 Tools)

Cursor's documented tool set:

| Tool | Description |
|---|---|
| `codebase_search` | Semantic search across the codebase |
| `read_file` | Read file contents |
| `run_terminal_cmd` | Execute terminal commands |
| `list_dir` | List directory contents |
| `grep_search` | Regex search in files |
| `file_search` | Find files by name pattern |
| `delete_file` | Delete a file |
| `reapply` | Reapply a previous edit (retry) |
| `edit_file` | Edit file contents |
| `web_search` | Search the web |

Compared to Claude Code's 16 tools:
- No `Task` tool (no sub-agent spawning)
- No `TaskCreate/Get/Update/List` (no task management)
- No `NotebookEdit/Read` (no Jupyter support)
- No `MultiEdit` (single-edit operations only)
- `reapply` is unique to Cursor (retry a failed edit)

---

## Philosophy: Optimised for Real-Time Collaboration

The entire Cursor prompt is optimised for one scenario: a developer is actively working in the IDE, the AI is their pair, and results need to be visible and runnable immediately.

This is the right optimisation for that scenario. The prompt does not need to handle overnight runs, cross-session persistence, or CI recovery — because those scenarios assume the human is not present, which is the opposite of Cursor's design.

Understanding this framing makes the "missing features" list comprehensible: they're not missing, they're out of scope. Cursor is not trying to be an autonomous engineering agent. It's trying to be an excellent pair programmer.

> See [`comparisons/vs-cursor.md`](../comparisons/vs-cursor.md) for the full feature comparison.  
> See [`prompts/what-the-prompts-reveal.md`](what-the-prompts-reveal.md) for what the structural differences mean.
