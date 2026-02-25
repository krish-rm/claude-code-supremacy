# Security Model

Claude Code's security model is encoded directly in the system prompt as a set of hardcoded prohibitions and default behaviours. This document describes what the security rules are, why they exist, and what they mean for teams evaluating Claude Code for sensitive codebases.

---

## Security Hardcodes: Always Applied

These rules are absolute — they cannot be overridden by user instruction, project configuration, or conversational pressure:

### 1. Defensive Security Only

Claude Code will assist with:
- Security code review (identifying vulnerabilities)
- Implementing authentication and authorisation
- Setting up security scanning tools (SAST, DAST)
- Hardening configurations
- Writing security-focused tests

Claude Code will **not** assist with:
- Writing exploit code or proof-of-concept attacks
- Crafting malware, ransomware, or destructive payloads
- Bypassing authentication or authorisation systems
- Network intrusion or enumeration tools
- Social engineering scripts

This is an unconditional constraint. The system prompt states "defensive security only" as a design identity, not a guideline. User instructions to "help me understand how an attacker would..." in the context of building a real exploit are refused.

### 2. Credential Handling

- Do not write secrets, API keys, or passwords directly into code files
- Always use environment variables for credentials
- Do not log sensitive values
- Do not transmit credentials via insecure channels
- Flag existing credential exposure when detected in code

When Claude Code encounters hardcoded credentials in an existing codebase during Explore, it will flag them in its findings before proceeding with the requested task.

### 3. Destructive Operation Guards

Before executing commands with irreversible effects:
- `rm -rf` on non-temporary directories
- Database drops or truncations
- Mass file overwrites
- Production deployment changes

Claude Code will stop and request explicit confirmation, with a clear description of what will be destroyed.

### 4. Commit Attribution (Always)

`Co-Authored-By: Claude <noreply@anthropic.com>` is added to every commit. This cannot be disabled. It provides:
- An audit trail of AI involvement
- Transparency for code reviewers
- Traceability for enterprise compliance requirements

---

## The Security Review Sub-Agent

For changes involving security-sensitive code, a dedicated sub-agent fires automatically. It applies a security-focused review:

**Triggers:**
- Changes to authentication or authorisation logic
- Changes involving credential handling
- File system operations writing outside the project root
- Shell command patterns that could be injection risks
- Network request code changes

**The sub-agent produces:**
- Identified security concerns (with severity rating)
- Recommended mitigations
- References to relevant security best practices

**Important:** The security sub-agent **flags but does not block**. The final decision remains with the human. This is a deliberate design choice — the agent's role is to surface information, not to enforce policy.

---

## Comparison: Claude Code vs. Cursor Security Approach

| Security Aspect | Claude Code | Cursor |
|---|---|---|
| Offensive security prohibition | ✅ Hardcoded | Soft guideline 📝 |
| Credential handling rules | ✅ Hardcoded | ✅ Guideline 📝 |
| Destructive operation guard | ✅ Explicit confirmation | ❓ Not documented |
| Security review sub-agent | ✅ | ❌ |
| Commit attribution | ✅ Required | ❌ Not required |
| "Defensive security only" | ✅ Explicit in prompt | ❌ Not explicit |

Cursor's security approach is softer — it includes guidelines about credential handling but does not encode the "defensive security only" constraint explicitly. This may or may not reflect different model training; it's a difference in what the system prompt explicitly states.

---

## What This Means for Enterprise Adoption

### Strengths
The hardcoded security model reduces several enterprise concerns:
- **Accidental secret exposure:** The credential handling rules reduce the risk of AI-assisted code accidentally committing API keys
- **Security audit trail:** The `Co-Authored-By` requirement makes AI involvement visible and auditable
- **Reduced attack surface:** The defensive-only constraint prevents Claude Code from being used as a security attack tool even if an internal user attempts it

### Remaining Concerns
The security model does not address all enterprise requirements:
- **Data residency:** API calls go to Anthropic's infrastructure. Teams with strict data residency requirements need to evaluate this separately.
- **Code exposure:** Code sent to the API is governed by Anthropic's data policy. Verify the current policy for sensitive IP.
- **The model's security knowledge is not exhaustive:** The security sub-agent identifies common patterns but is not a substitute for a professional security review.
- **Server-side prompt components:** The security rules visible in the npm package may not represent all security constraints. Some may be applied server-side.

### Recommendation
For teams with strict security requirements: treat Claude Code's built-in security model as a first layer, not a complete security programme. Use it alongside dedicated SAST tools, professional security review for sensitive systems, and your organisation's standard security controls.

> See [`architecture/context-management.md`](context-management.md) for how context limits interact with security-sensitive session content.
