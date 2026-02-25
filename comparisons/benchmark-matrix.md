# Benchmark Matrix: All Tools, All Dimensions

This matrix compares Claude Code, Cursor, GitHub Copilot, Gemini Code Assist, Windsurf, and Devin AI across every major capability dimension. Claims marked with 📝 are based on system prompt data; those marked with 👁 are based on documented features and observed behaviour; those marked with ❓ have lower confidence.

> **Baseline:** Claude Code v2.0.56 (analysed February 2026). See [`comparisons/00-methodology.md`](00-methodology.md) for source reliability ratings.

---

## Capability Matrix

| Capability | Claude Code | Cursor | GitHub Copilot | Gemini Code Assist | Windsurf | Devin AI |
|---|---|---|---|---|---|---|
| **Planning & Discovery** | | | | | | |
| Plan Mode (enforced explore-before-modify) | ✅ 📝 | ❌ | ❌ | ❌ | ❌ | Partial ❓ |
| Structured plan presentation | ✅ 📝 | ❌ | ❌ | ❌ | ❌ | ✅ 👁 |
| User approval gate before execution | ✅ 📝 | ❌ | N/A | N/A | ❌ | ✅ 👁 |
| Read-only exploration sub-agent | ✅ 📝 Haiku | ❌ | ❌ | ❌ | ❌ | ❓ |
| **Task Management** | | | | | | |
| Task creation (structured) | ✅ 📝 TaskCreate | ❌ | ❌ | ❌ | ❌ | ✅ 👁 |
| Task persistence (cross-session) | ✅ 📝 ~/.claude/tasks/ | ❌ | ❌ | ❌ | ❌ | ✅ 👁 |
| Task dependency graphs | ✅ 📝 addBlockedBy | ❌ | ❌ | ❌ | ❌ | Partial 👁 |
| Status lifecycle (pending/in_progress/completed) | ✅ 📝 | ❌ | ❌ | ❌ | ❌ | ✅ 👁 |
| **Multi-Agent & Coordination** | | | | | | |
| Sub-agent spawning (Task tool) | ✅ 📝 9+ agents | ❌ | ❌ | ❌ | ❌ | ✅ 👁 |
| Multi-session task sharing | ✅ 📝 CLAUDE_CODE_TASK_LIST_ID | ❌ | ❌ | ❌ | ❌ | ✅ 👁 |
| Parallel agent workstreams | ✅ 📝 | ❌ | ❌ | ❌ | ❌ | ✅ 👁 |
| Model routing per task type | ✅ 📝 Haiku/Sonnet/Opus | ❌ | ❌ | ❌ | ❌ | ❓ |
| **Execution** | | | | | | |
| Multi-step autonomous execution | ✅ 📝 | Partial 👁 | ❌ | ❌ | ✅ Cascade 👁 | ✅ 👁 |
| File read, write, edit | ✅ 📝 | ✅ 👁 | Limited | Limited | ✅ 👁 | ✅ 👁 |
| Shell command execution (Bash) | ✅ 📝 | ✅ 👁 | ❌ | ❌ | ✅ 👁 | ✅ 👁 |
| Web search/fetch | ✅ 📝 | ❌ | ❌ | ❌ | ❌ | ✅ 👁 |
| Native tool count | 16 📝 | 10 📝 | ~5 👁 | ~5 👁 | ~10 👁 | ~15 👁 |
| MCP server extension | ✅ 📝 | ✅ 👁 | ❌ | ❌ | ❌ | ❌ |
| **Verification & QA** | | | | | | |
| Automated test execution | ✅ 📝 | Manual | ❌ | ❌ | Partial | ✅ 👁 |
| Test-first instructions in prompt | ✅ 📝 | ❌ | ❌ | ❌ | ❌ | ✅ 👁 |
| 3-strike linter escalation | ✅ 📝 | ✅ 📝 (same rule) | ❌ | ❌ | ❌ | ❓ |
| Verify Plan Reminder | ✅ 📝 | ❌ | ❌ | ❌ | ❌ | ❓ |
| Sub-agent verification | ✅ 📝 | ❌ | ❌ | ❌ | ❌ | ✅ 👁 |
| **Git & CI/CD** | | | | | | |
| Autonomous commits (HEREDOC) | ✅ 📝 | ❌ (user) | ❌ | ❌ | ❌ | ✅ 👁 |
| Co-authored attribution | ✅ 📝 hardcoded | ❌ | ❌ | ❌ | ❌ | ✅ 👁 |
| Branch management | ✅ 📝 | ❌ | ❌ | ❌ | ❌ | ✅ 👁 |
| PR creation | ✅ 📝 gh CLI | ❌ | ❌ | PR review only | ❌ | ✅ 👁 |
| CI output reading | ✅ 📝 | ❌ | ❌ | ❌ | ❌ | ✅ 👁 |
| CI failure recovery loop | ✅ 📝 | ❌ | ❌ | ❌ | ❌ | ✅ 👁 |
| **Context & Memory** | | | | | | |
| Project memory file (CLAUDE.md) | ✅ 📝 | ❌ | ❌ | ❌ | ❌ | ❓ |
| Auto-generate project memory | ✅ 📝 sub-agent | ❌ | ❌ | ❌ | ❌ | ❓ |
| Session compaction | ✅ 📝 | ❌ | ❌ | ❌ | ❌ | ❓ |
| Cross-session continuity | ✅ 📝 | ❌ | ❌ | ❌ | ❌ | ✅ 👁 |
| **Security** | | | | | | |
| Hardcoded security rules | ✅ 📝 explicit | Soft guidelines 📝 | ❓ | ❓ | ❓ | ❓ |
| Defensive security only | ✅ 📝 | Partial 📝 | ❓ | ❓ | ❓ | ❓ |
| No credential exposure | ✅ 📝 | ✅ 📝 (implied) | ❓ | ❓ | ❓ | ❓ |
| Security review sub-agent | ✅ 📝 | ❌ | ❌ | ❌ | ❌ | ❓ |
| **Interface & Pricing** | | | | | | |
| Interface type | CLI | IDE (custom) | IDE (VS Code) | IDE plugin | IDE (custom) | Web + CLI |
| Primary model | Claude Sonnet/Opus/Haiku | Claude 3.5 Sonnet ❓ | GPT-4o | Gemini Pro/Ultra | Mixtral/Claude ❓ | Claude ❓ |
| Inline completion | ❌ | ✅ | ✅ | ✅ | ✅ | ❌ |
| Pricing model | API usage | $20–$40/mo | $10–$39/mo/user | Varies (GCP) | Free–$15/mo | $500+/mo |
| Free tier | Limited (API) | ✅ | ✅ | ✅ | ✅ | ❌ |
| **SDLC Scope** | | | | | | |
| SDLC abstraction level | **L4–L5** | L2–L4 | L1–L3 | L2–L3 | L3–L4 | L4–L5 |
| Overnight autonomous runs | ✅ | ❌ | ❌ | ❌ | ❌ | ✅ 👁 |

---

## Notes

**On Devin AI:** Devin operates at L4–L5 similar to Claude Code, but with a different model — it is more autonomous (less human-in-the-loop) and runs as a cloud service with a browser-based interface. The price point ($500+/month) reflects full-autonomy positioning for enterprise teams. This comparison does not include a dedicated `vs-devin.md` file, but Devin is worth evaluating for teams willing to reduce human supervision further.

**On model identities:** Several tools (Cursor, Windsurf) have not publicly confirmed exact model versions or fine-tuning status. Entries marked ❓ reflect this uncertainty.

**On pricing:** Pricing changes frequently. Use this for relative positioning only. Verify current pricing on each vendor's website before making decisions.

---

## Summary: Where Each Tool Wins

| Tool | Best for |
|---|---|
| **Claude Code** | Full SDLC autonomous execution, multi-session features, sprint-level work |
| **Cursor** | Real-time editor collaboration, bounded features, familiar IDE UX |
| **GitHub Copilot** | Inline completion, Microsoft/GitHub ecosystem, enterprise governance |
| **Gemini Code Assist** | GCP-integrated teams, PR review at scale, enterprise Google Workspace |
| **Windsurf** | Agentic IDE mid-point, good UX, competitive pricing |
| **Devin AI** | Full autonomy, enterprise teams willing to reduce human oversight, large budgets |
