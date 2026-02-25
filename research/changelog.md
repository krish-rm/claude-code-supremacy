# Changelog

Version tracking for Claude Code system prompt changes and their impact on the claims in this repository. Entries are added when a meaningful prompt change is detected via the Piebald-AI tracker or community analysis.

**Format:** Date | Claude Code Version | What Changed | Impact on This Repository

---

## 2026

### February 2026 | v2.0.56 (baseline) | Initial analysis

**Context:** This repository was created with Claude Code v2.0.56 (December 1, 2025) as the baseline. All architectural claims, sub-agent token counts, and feature descriptions are anchored to this version.

**Key facts at this version:**
- 110+ conditional prompt strings
- 9+ sub-agent types documented
- TaskCreate system (upgraded from TodoWrite on January 23, 2025)
- 16 built-in tools
- Sub-agent token counts: Explore (516), Plan Mode Enhanced (633), Task Tool (294), Agent Creation Architect (1111), CLAUDE.md Creation (384), Status Line (993), PR Comments (404)

**Impact:** No previous baseline to compare against. This is the initial state.

---

## 2025

### January 23, 2025 | ~v1.5.x | TodoWrite → TaskCreate upgrade

**What changed:** The simple `TodoWrite` in-session task list was replaced with the full `TaskCreate/Get/Update/List` system with:
- Persistent storage to `~/.claude/tasks/`
- Dependency graphs via `addBlockedBy`
- Status lifecycle (pending → in_progress → completed)
- Multi-session sharing via `CLAUDE_CODE_TASK_LIST_ID`
- `Ctrl+T` keyboard shortcut for task panel

**Impact on this repo:** The task management section ([`sdlc/02-task-management.md`](../sdlc/02-task-management.md)) fully reflects the post-upgrade architecture. Claims about TodoWrite refer to the pre-January 2025 baseline only.

**Claims affected:**
- All task management comparisons (no competitor has matched this capability)
- The multi-agent team coordination use case ([`use-cases/team-coordination.md`](../use-cases/team-coordination.md))
- The feature sprint use case ([`use-cases/feature-sprint.md`](../use-cases/feature-sprint.md))

---

## Template for Future Entries

```
### [Date] | [Claude Code Version] | [Brief description]

**What changed:** [Specific prompt change — what string was added/removed/modified]

**Impact on this repo:** [Which claims are affected, which files need updating]

**Claims affected:**
- [List specific claims that changed, with file links]

**Updated files:**
- [List files in this repo that were updated to reflect the change]
```

---

## How to Detect Prompt Changes

1. **Monitor the Piebald-AI tracker:** [https://github.com/Piebald-AI/claude-code-system-prompts](https://github.com/Piebald-AI/claude-code-system-prompts) — they diff each version automatically.

2. **Check npm release notes:** `npm view @anthropic-ai/claude-code versions` lists all published versions.

3. **Monitor Claude Code release notes:** Anthropic publishes release notes that sometimes describe prompt-level changes (but not always).

4. **Community monitoring:** r/ClaudeAI and the Anthropic Discord often surface prompt changes when users notice behaviour changes.

---

## Impact Classification

For each changelog entry, classify the impact:

| Impact Level | Meaning |
|---|---|
| **Critical** | A claim in a comparison file is now incorrect |
| **High** | A key architectural fact has changed |
| **Medium** | A minor detail changed but the overall claim still holds |
| **Low** | A cosmetic or wording change with no substantive effect |
| **Informational** | A new feature added that doesn't change existing comparisons |
