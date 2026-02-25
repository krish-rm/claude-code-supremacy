# Claude Code vs. Cursor

Cursor is the most direct and serious competitor to Claude Code. Both tools use Claude 3.5 Sonnet as the primary model. The distinction is entirely architectural: Cursor is built as a pair programmer in a custom IDE; Claude Code is built as an autonomous engineering agent in the CLI. The difference in product philosophy produces a substantial difference in capability at the L4–L5 scope.

---

## Philosophy: Identity Matters

The opening lines of each tool's system prompt reveal the intended design:

**Cursor's opening:**
> "You are a powerful agentic AI coding assistant, powered by Claude 3.5 Sonnet. You operate exclusively in Cursor, the world's best IDE. You are pair programming with a USER."

**Claude Code's opening (paraphrased from prompt data):**
> Claude Code is designed to autonomously execute engineering tasks. It has access to a wide range of tools and is expected to complete tasks end-to-end.

"Pair programming with a USER" vs. "autonomously execute engineering tasks." These are meaningfully different product identities. Cursor optimises for real-time collaboration with a human present. Claude Code optimises for autonomous task completion with a human in a supervisory role.

Neither is wrong. They're built for different working styles.

---

## System Prompt Architecture

| Dimension | Claude Code | Cursor |
|---|---|---|
| Prompt structure | 110+ conditional strings | Single ~3,000 token prompt |
| Assembly | Context-dependent, modular | Static at session start |
| Sub-agents | 9+ specialised sub-agents | None documented |
| Token count (approx.) | Variable, 2,000–8,000+ | ~3,000 fixed |
| Version tracking | Piebald-AI tracks 37 versions | x1xhlol (single version) |

The modular architecture of Claude Code's prompt has a practical consequence: the right components are loaded for each task. Exploration uses the Haiku sub-agent prompt (516 tokens). Execution loads the full toolset. Plan Mode restricts tools to read-only. A single monolithic prompt cannot adapt this way — it provides the same instructions regardless of context.

---

## Full Feature Matrix

| Capability | Claude Code | Cursor |
|---|---|---|
| **Plan Mode** (enforced explore-before-modify) | ✅ Two-phase gate | ❌ |
| **Task persistence** (cross-session) | ✅ ~/.claude/tasks/ | ❌ |
| **Task dependencies** (addBlockedBy) | ✅ Full graph | ❌ |
| **Multi-agent coordination** | ✅ CLAUDE_CODE_TASK_LIST_ID | ❌ |
| **Sub-agent spawning** | ✅ Task tool | ❌ |
| **CI/CD integration** | ✅ Read CI output + fix loop | ❌ |
| **Autonomous git workflow** | ✅ HEREDOC commits, Co-Authored-By | ❌ (user commits) |
| **PR creation** | ✅ gh CLI | ❌ |
| **CLAUDE.md / project memory** | ✅ Generated sub-agent | ❌ |
| **Model routing** | ✅ Haiku/Sonnet/Opus per task | ❌ (Sonnet only) |
| **Security hardcodes** | ✅ Defensive-only, explicit | Soft guidelines |
| **Interface** | CLI | IDE (custom VS Code fork) |
| **3-strike linter rule** | ✅ | ✅ (same rule) |
| **Tool count** | 16 native + MCP | 10 |
| **SDLC scope** | L4–L5 | L2–L4 |

---

## The Communication Rules Comparison

Both tools have explicit rules about how the AI should communicate. The differences reveal product philosophy:

**Cursor's rules (from leaked prompt):**
- "It is EXTREMELY important that your generated code can be run immediately by the USER"
- Never lie to the user or make up information
- Do not refer to tool names in responses to the user
- Tool names are hidden from the user
- "Cursor does not use markdown in its LLM responses"

**Claude Code's rules (from Piebald-AI source):**
- Minimal output — do not narrate, do not explain unless asked
- Be explicit about tool invocations (not hidden)
- Escalate on failure rather than silently retry
- HEREDOC format for multi-line shell content

Cursor hides tool names from users — the interaction model assumes users don't need to know what tools are firing. Claude Code makes tool invocations visible — the interaction model assumes users want to see and understand what's happening. This is a direct reflection of the pair-programmer (Cursor) vs. supervisor (Claude Code) identity.

---

## When to Use Which

This is not a zero-sum comparison. Both tools serve real needs.

**Use Claude Code when:**
- The feature requires understanding a large, existing codebase before modification
- You're running overnight or unattended (autonomous execution, human not present)
- You need multi-session continuity (pick up where you left off)
- You want CI to be part of the verification loop, not just a final gate
- The task spans multiple files, services, or days of work
- You need coordination between multiple AI instances on the same task list

**Use Cursor when:**
- You want real-time collaboration with the AI visible in your editor
- The task is bounded: a single file, a specific refactor, a quick fix
- You prefer to commit code yourself and maintain tight commit-level control
- You're comfortable in the IDE and prefer the inline experience
- The task benefits from the AI seeing your cursor position and active context

A sophisticated team might use both: Cursor for immediate, editor-integrated assistance and Claude Code for longer-horizon autonomous task execution.

---

## Where Cursor Genuinely Wins

- **Editor integration**: Cursor's IDE experience is seamless. Inline code suggestions, chat in context, diff views — the UX for small tasks is superior.
- **Discoverability**: The IDE metaphor is familiar. New engineers get productive in Cursor faster.
- **Immediate feedback loop**: Seeing changes appear in your editor as the AI makes them is a valuable flow state for bounded tasks.
- **Ecosystem**: Third-party extensions, community tips, established workflows.

These are not trivial advantages for day-to-day coding work. Cursor's focus on the editor experience is a genuine product strength.

> See [`comparisons/benchmark-matrix.md`](benchmark-matrix.md) for the comprehensive cross-tool capability matrix.
