# Open Questions

Honest accounting of what this repository does not know, cannot verify, or has insufficient evidence to claim. This list exists to prevent false confidence and to invite contributions from people who have better data.

---

## Architecture Unknowns

### 1. Server-side prompt components

Claude Code's npm package exposes the client-side prompt strings. It is plausible — even likely — that additional system prompt components are applied server-side, before the client-assembled prompt reaches the model.

If server-side components exist, they could:
- Add or modify security rules
- Adjust model behaviour for specific contexts
- Include additional tool definitions or restrictions

**What we know:** Everything in the npm package (documented in this repo).  
**What we don't know:** Anything additional applied server-side.  
**Why this matters:** Claims about security hardcodes and tool restrictions are based on what's visible client-side. There may be additional layers.

### 2. Model routing reality

This repository documents the *intended* model routing (Haiku for Explore, Sonnet for execution, Opus for complex design) based on sub-agent definitions.

**What we don't know:** Whether the documented routing is applied consistently in production, whether it varies by API tier, or whether Anthropic overrides routing server-side.

### 3. Fine-tuning on any model

Cursor uses Claude 3.5 Sonnet — or possibly a fine-tuned version. If Cursor fine-tunes the model for pair-programming behaviour, the comparison is not purely "same model, different prompts" — it's "same architecture, different model."

**What we don't know:** Whether Cursor, Windsurf, or any competitor uses a fine-tuned model rather than the base model.

---

## Competitor Unknowns

### 4. Whether Cursor has unrevealed orchestration layers

The Cursor system prompt (from x1xhlol) doesn't document any multi-agent or sub-agent system. But this doesn't mean one doesn't exist — it might be implemented differently from how Claude Code implements it (not in the system prompt itself).

**What we don't know:** Whether Cursor has server-side orchestration not visible in the extracted system prompt.

### 5. Windsurf's full Cascade architecture

Windsurf's Cascade feature shows agentic behaviour. The precise architecture — what triggers multi-step execution, how failures are handled, whether task state persists — is not fully documented from available sources.

### 6. Gemini Code Assist's internal prompt structure

Google has not published system prompt details for Gemini Code Assist, and extraction efforts via x1xhlol have been limited compared to Claude Code and Cursor. The Gemini comparison in this repository has lower confidence than the Cursor comparison.

---

## Performance Unknowns

### 7. Rigorous performance benchmarks

This repository makes no direct claims about output quality (which model produces better code). That question requires:
- Controlled, reproducible tasks
- Multiple runs with identical prompts
- Blind evaluation by qualified engineers
- Coverage across domains (web, systems, data science, etc.)

**What exists:** Some community benchmarks (HumanEval, SWE-Bench) that score models on coding tasks. These measure L1–L3 capability, not L4–L5 task completion.  
**What doesn't exist:** Rigorous L4–L5 benchmarks measuring plan quality, task coordination, and sprint-level throughput.

### 8. Long-term code quality effects

The concern: AI-generated code looks correct in testing but degrades in maintainability over time. Does using Claude Code for implementation work increase technical debt at a rate that eventually offsets the productivity gain?

**What we don't know:** There is no rigorous longitudinal study covering this. The concern is real; the magnitude is unknown.

---

## Human Effects Unknowns

### 9. Junior engineer development impact

If senior engineers use Claude Code to handle implementation work that would previously have been mentoring opportunities for junior engineers, does the junior engineer pipeline suffer?

**What we don't know:** Whether this concern materialises at scale, how teams can mitigate it, and whether the new review-and-direction skills offset the lost implementation practice.

### 10. Domain expertise atrophy

If engineers stop writing certain types of code (SQL, CSS, infrastructure scripts) because Claude Code handles it, do they lose fluency in those areas? Does this reduce the quality of their reviews?

**What we don't know:** The rate of atrophy, which domains are most at risk, and whether deliberate practice can counteract it.

### 11. Responsibility and accountability norms

When AI-generated code ships a bug, how should responsibility be allocated? Currently, the `Co-Authored-By: Claude` attribution creates a record but no legal clarity.

**What we don't know:** How courts, regulators, and professional engineering organisations will treat AI-assisted code over time.

---

## Invitation to Contribute

If you have data, research, or observations that address any of these open questions:

1. Open an issue describing what you know and its source
2. If adding content to this file (splitting into subfiles, adding context to a question), follow the contribution guidelines in [`CONTRIBUTING.md`](../CONTRIBUTING.md)
3. If resolving a question (closing it with evidence), propose a move to a "resolved" section with the source

The point of a research repository is to get closer to ground truth over time. Honest open questions are how we get there.
