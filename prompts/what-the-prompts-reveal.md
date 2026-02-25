# What the Prompts Reveal

Reading the Claude Code and Cursor system prompts side by side, certain structural and philosophical differences emerge that go beyond feature lists. This document analyses the deeper design principles visible in how each prompt is written.

---

## The Opening Lines: Identity at 5 Words

**Cursor:**
> "You are a powerful agentic AI coding assistant, powered by Claude 3.5 Sonnet. You operate exclusively in Cursor, the world's best IDE. You are **pair programming** with a USER."

**Claude Code (paraphrased from prompt data):**
> "You are Claude Code, Anthropic's AI coding agent. Your role is to autonomously complete engineering tasks."

**Pair programming** vs. **autonomously complete**. These aren't just word choices — they're the entire product identity expressed in a single phrase.

"Pair programming" encodes:
- A human is always present
- Collaboration is the mode, not delegation
- Speed and immediacy matter (pair programmers don't disappear for hours)
- The AI is a partner, not a contractor

"Autonomously complete" encodes:
- The human may not be present
- The AI is responsible for the full task
- Verification is the AI's job, not the human's
- Escalation happens when the AI is stuck, not continuously

Every feature difference between the tools follows logically from this identity difference.

---

## "Apologise Less" vs. "Minimal Output"

**Cursor's rule (from prompt):**
The prompt instructs Cursor to avoid excessive apology — "be direct, not overly apologetic."

**Claude Code's equivalent:**
The prompt instructs minimal output — do not narrate, do not explain unless asked, produce less text not more.

These are different solutions to the same problem: AI assistants that over-talk, over-explain, and perform helpfulness through words rather than work.

Cursor's solution is emotional ("apologise less") — appropriate for a tool in a real-time conversation where the AI's tone and relationship with the user matter.

Claude Code's solution is functional ("minimal output") — appropriate for a tool in autonomous mode where the human isn't reading every line as it happens, they're checking results.

---

## Tool Name Hiding: Transparency vs. Simplicity

**Cursor's rule:**
> "Do not refer to tool names in responses to the USER."

**Claude Code:**
Tool invocations are visible to the user. The transparency is intentional.

Cursor hides its tool names because the pair-programmer framing makes internal mechanics irrelevant to the user experience. A human pair doesn't say "I'm now running grep" — they just search. Hiding tool names makes the AI feel more like a colleague and less like a system.

Claude Code exposes tool calls because the supervisor framing makes transparency essential. If you're reviewing an AI's work rather than collaborating in real time, you want to know what it's doing. Visible tool calls build trust through auditability.

This is not a right-or-wrong distinction. It's a consequences-of-framing distinction.

---

## Security Sections: Hardcodes vs. Guidelines

**Claude Code security approach:**
The security rules are hardcodes — they appear in the prompt as absolute prohibitions:
- "Do not perform offensive security operations"
- "Do not expose credentials"
- "Add Co-Authored-By to all commits"

These are stated as facts about what Claude Code does, not suggestions about what it should try to do.

**Cursor security approach (from prompt):**
Security guidance is softer — "always use environment variables for credentials," "handle API keys securely." These are guidelines, not prohibitions.

The difference reflects the risk profile of each product. Claude Code, operating autonomously without constant human supervision, needs harder constraints to limit autonomous damage. Cursor, operating with a human present and reviewing changes in real time, can rely more heavily on the human as a safety layer.

---

## Three Design Principles Visible in the Architecture

Reading both prompts structurally, three distinct design philosophies emerge:

### 1. Framing determines capability ceiling

Cursor was framed as a pair programmer from the beginning. This framing produced an excellent pair-programming tool but imposed an implicit ceiling: pair-programming doesn't include overnight runs, cross-session task persistence, or CI recovery. These features don't fit the mental model.

Claude Code was framed as an autonomous engineering agent. This framing produced features that would be weird in a pair-programmer (task dependency graphs, HEREDOC commits) but are natural for an agent (they're the scaffolding needed for responsible autonomy).

**Lesson:** The framing of an AI system prompt is not cosmetic. It shapes which features feel natural to build and which feel out of scope.

### 2. Human presence is an architectural assumption

Cursor assumes the human is present: code must be immediately runnable, tool names are hidden (the AI does, not narrates), apologise less (tone matters in conversation). All of these are optimised for a real-time collaborative session.

Claude Code assumes the human may not be present: verification is automated, task state persists across sessions, CI failure recovery is autonomous. All of these are optimised for unsupervised execution.

**Lesson:** The assumed human presence level should be explicit in system design. Features built for the wrong presence level will either frustrate users (over-explaining when they want results) or lose them (failing silently when they can't know what went wrong).

### 3. Modularity is a product commitment

Claude Code's 110+ modular strings represent a decision to invest significantly in prompt engineering infrastructure. Maintaining 37 versions of 110+ strings, with diff tracking, requires ongoing effort.

Cursor's single ~3,000 token prompt is easier to maintain but less adaptable. Every context gets the same instructions.

The modular approach enables Claude Code to optimise per-context in a way that's impossible with a single prompt. But it also creates complexity — a bug in one component can have unexpected effects on assembled contexts.

**Lesson:** Modularity in prompt design is an investment with maintenance costs, not just benefits.

---

## The Philosophical Synthesis

Both Cursor and Claude Code build on the same underlying model capability. The difference in what they are — pair programmer vs. engineering agent — comes entirely from how the system prompt frames identity, assumes context, and encodes constraints.

System prompt design is product design. The prompts reveal this with unusual clarity.

> See [`prompts/claude-code-prompt-anatomy.md`](claude-code-prompt-anatomy.md) and [`prompts/cursor-prompt-anatomy.md`](cursor-prompt-anatomy.md) for the detailed breakdowns.
