# Git Workflow and CI/CD Integration

Claude Code's git integration is not a convenience feature bolted on — it's an encoded workflow with specific patterns, attribution rules, and CI recovery loops. This section documents exactly what the system prompt specifies and what it means in practice.

---

## The HEREDOC Commit Pattern

Claude Code is instructed to write commit messages using shell HEREDOC syntax, not inline string arguments. This is required to avoid shell escaping issues with multi-line messages:

```bash
git commit -m "$(cat <<'EOF'
feat(auth): implement JWT authentication with refresh tokens

- Add jwt.js utility for token generation and verification
- Add auth.js middleware for route protection
- Add /login endpoint with bcrypt password verification
- Add /refresh endpoint for token renewal
- Apply auth middleware to all /api/protected/* routes

Fixes #142

Co-Authored-By: Claude <noreply@anthropic.com>
EOF
)"
```

The HEREDOC pattern (`cat <<'EOF'`) ensures that:
- Multi-line messages are preserved without escaping issues
- Special characters (quotes, backticks) don't break shell parsing
- The commit message is readable in the git log without modification

---

## Co-Authored-By: Always

Every commit Claude Code creates includes:

```
Co-Authored-By: Claude <noreply@anthropic.com>
```

This is a **hardcoded requirement** in the system prompt, not a configurable option. It:

1. **Provides attribution** — the commit history is honest about AI involvement
2. **Creates an audit trail** — teams can query which commits had AI assistance
3. **Sets expectations** — reviewers know to look more carefully at the reasoning, not just the diff
4. **Respects the human contributor** — the engineer who approved the plan is still credited as author

For enterprises with compliance requirements around AI-generated code, this attributation is important. For teams evaluating AI risk, it makes the exposure visible and auditable.

---

## The Interactive Flag Prohibition

The system prompt explicitly prohibits Claude Code from running git commands with interactive flags:

- ❌ `git add -p` (interactive patch mode)
- ❌ `git rebase -i` (interactive rebase)
- ❌ `git commit` without `-m` (opens editor)
- ❌ Any command that expects keyboard input mid-execution

The reason: interactive commands break in CI environments and in non-interactive terminal sessions. Claude Code is designed to work in automated pipelines, overnight runs, and headless environments — interactive commands would cause it to hang indefinitely.

This is not a limitation — it's a deliberate design constraint for autonomous operation.

---

## CI Failure Recovery Loop

```
┌─────────────────────────────────────────────────────────┐
│                 CI RECOVERY LOOP                         │
│                                                         │
│  1. git push (or PR creation via gh CLI)                │
│       │                                                 │
│       ▼                                                 │
│  2. Read CI output (gh cli or webhook)                  │
│       │                                                 │
│       ├──▶ PASS ──▶ Mark task complete, notify human    │
│       │                                                 │
│       └──▶ FAIL ──▶ Parse error output                  │
│                        │                               │
│                        ▼                               │
│                 3. Diagnose failure                     │
│                        │                               │
│                        ▼                               │
│                 4. Apply fix                            │
│                        │                               │
│                        ▼                               │
│                 5. Recommit + repush                    │
│                        │                               │
│                        ├──▶ PASS ──▶ complete           │
│                        └──▶ FAIL ──▶ [3 strikes]        │
│                                          │              │
│                                    Escalate to human    │
└─────────────────────────────────────────────────────────┘
```

The CI recovery loop uses `gh` CLI to read build output. Claude Code reads the raw log, identifies the failing step, and applies a targeted fix. Common CI failures it handles:

- Missing environment variables (detected from error, adds to `.env.example`)
- Dependency version mismatches (updates `package.json` or requirements)
- Test environment setup failures (adds fixture or seed data)
- Docker build failures (fixes Dockerfile steps)
- Lint failures in the CI runner (applies same linter locally and fixes)

What it cannot handle autonomously:
- Secrets or deployment credentials (escalates immediately — security hardcode)
- Infrastructure changes (adds manually as a task for human)
- Network-dependent integration failures in CI (escalates after 3 attempts)

---

## PR Creation via `gh` CLI

When all tasks in a sprint are complete and verified, Claude Code creates a pull request using the `gh` CLI:

```bash
gh pr create \
  --title "feat(auth): JWT authentication system (#142)" \
  --body "$(cat <<'EOF'
## Summary
Implements JWT authentication with refresh token support.

## Changes
- `src/utils/jwt.js` — Token generation and verification
- `src/middleware/auth.js` — Route protection middleware
- `src/routes/auth.js` — Login and refresh endpoints
- Tests: 94% coverage on auth module

## Testing
- Unit tests: all passing
- Integration tests: all passing
- Manual: tested login/refresh/protected route flow

## Notes
- JWT_SECRET must be set in production environment (see .env.example)
- Refresh tokens expire in 30 days by default (configurable via JWT_REFRESH_EXPIRES_IN)

Co-Authored-By: Claude <noreply@anthropic.com>
EOF
)" \
  --base main \
  --draft
```

PRs are created as **draft** by default. This is another explicit human gate: the engineer reviews the PR, confirms it looks correct, and promotes it to ready-for-review. Claude Code does not automatically merge.

---

## Comparison: Git Workflow Across Tools

| Git Capability | Claude Code | Cursor | GitHub Copilot | Gemini Code Assist |
|---|---|---|---|---|
| Autonomous commits | ✅ HEREDOC pattern | ❌ (user commits) | ❌ | ❌ |
| Co-authored attribution | ✅ Hardcoded | ❌ | ❌ | ❌ |
| CI failure recovery | ✅ Read + fix loop | ❌ | ❌ | Partial (PR review) |
| PR creation | ✅ gh CLI | ❌ | Basic | PR review only |
| Branch management | ✅ | ❌ | ❌ | ❌ |
| Interactive flag safety | ✅ Prohibited | N/A | N/A | N/A |
| Commit message standard | ✅ Conventional commits | User's choice | N/A | N/A |

The git integration is where Claude Code's L5 scope becomes clearest: it doesn't just write code, it ships code — through the full pipeline from commit to CI-verified PR.

> See [`comparisons/vs-cursor.md`](../comparisons/vs-cursor.md) for a detailed analysis of why Cursor's omission of this matters.
