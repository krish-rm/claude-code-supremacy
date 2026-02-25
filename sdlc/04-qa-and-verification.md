# QA and Verification

Verification is not a separate phase in Claude Code — it's a property of the execution loop itself. Every meaningful change triggers a verification pass. This section documents the verification architecture in detail.

---

## System-Level Verification Constructs

Claude Code's system prompt encodes verification at three levels:

### 1. The Verify Plan Reminder

After executing a set of changes, an internal prompt fires that requires the agent to explicitly check:
- Were all plan steps completed?
- Were acceptance criteria met?
- Were tests written and run?
- Were edge cases handled?

This is a prompted self-review, not a rule. The agent must produce an answer to each question before proceeding. If the self-review reveals an incomplete step, the execution loop continues.

### 2. The 3-Strike Fix Loop

When verification fails (test failure, linter error, type error):
1. Claude Code diagnoses the issue
2. Applies a fix
3. Re-runs verification

If this fails three consecutive times on the same issue, the agent escalates to the human with:
- The specific error message
- What was attempted in each of the three fixes
- The agent's best hypothesis about root cause
- A request for information that the agent doesn't have

### 3. Test-First Instructions

The system prompt includes instructions to write tests before or alongside implementation (test-first approach), not after. This is encoded as a behavioural instruction, not merely a suggestion. The consequence: Claude Code's output typically includes working tests as part of the deliverable, not as an afterthought.

---

## Sub-Agent Independent Verification

For complex features where output correctness is critical, Claude Code can spawn a verification sub-agent — a fresh agent instance with no knowledge of how the code was written, tasked only with testing whether it works correctly.

This is structurally similar to the software engineering practice of having a different engineer review code than wrote it. A sub-agent that wasn't involved in the implementation will find issues that the implementing agent missed through confirmation bias.

The verification sub-agent pattern:
```
Main Agent (implementation complete)
         │
         ▼ Task("Verify this implementation")
         │
Verification Sub-Agent
  - Fresh context (no implementation history)
  - Reads implementation from disk
  - Runs tests independently
  - Checks against acceptance criteria
  - Returns: pass/fail + specific issues
         │
         ▼
Main Agent receives result
  - Pass → TaskUpdate: completed
  - Fail → Fix loop begins
```

---

## How Verification Compares Across Tools

| Verification Capability | Claude Code | Cursor | GitHub Copilot | Gemini Code Assist |
|---|---|---|---|---|
| Automated test execution | ✅ Bash tool | Manual | ❌ | ❌ |
| Test-first instructions | ✅ In prompt | ❌ | ❌ | ❌ |
| Fix loop on failure | ✅ 3-strike | 3-retry (same) | ❌ | ❌ |
| Verify Plan Reminder | ✅ System-level | ❌ | ❌ | ❌ |
| Sub-agent verification | ✅ Task tool | ❌ | ❌ | ❌ |
| Human escalation on loop | ✅ Encoded | Encoded | N/A | N/A |
| Linter integration | ✅ Bash | ✅ Native | Partial | Partial |
| CI feedback loop | ✅ Push + read | ❌ | ❌ | ❌ |

---

## What Happens on Verification Failure

### Scenario: Test failure during feature execution

```
State: Task "Build checkout API endpoint" in_progress

1. Code written, tests run:
   → jest: FAIL src/checkout/checkout.test.js
   → Error: "Order total calculation incorrect for discounted items"

2. Strike 1:
   → Diagnose: off-by-one in discount application
   → Fix applied, tests re-run
   → FAIL: "Edge case: multiple discounts not handled"

3. Strike 2:
   → Diagnose: discount stacking logic missing
   → Fix applied, tests re-run
   → FAIL: "Integration test: payment intent amount mismatch"

4. Strike 3:
   → FAIL: different root cause detected

5. Escalation to human:
   "I've attempted 3 fixes for the test failures in checkout.test.js.
    The core issue appears to be in how discounts compose with the
    payment intent calculation. I'm missing information about the
    intended discount stacking behaviour. Specifically:
    - Should multiple discount codes stack additively or use the highest?
    - Does the payment intent amount include or exclude tax?
    Please clarify, and I'll continue."
```

This escalation pattern is essential for team trust. An AI that loops indefinitely is more dangerous than one that asks for help.

---

## The Security Review Sub-Agent

Claude Code includes a dedicated security review sub-agent that fires on certain types of changes:

- Changes involving authentication or authorisation
- File system operations that write outside the project root
- Network requests
- Credential handling
- Shell command execution patterns

The security sub-agent applies "defensive security only" hardcodes from the system prompt. It will flag, but not block, suspicious patterns — the final decision remains with the human.

> See [`architecture/security-model.md`](../architecture/security-model.md) for the full list of security hardcodes.

---

## CI/CD as External Verification

Claude Code treats CI/CD pipelines as an external verification oracle. After pushing code:

1. Push to branch via `gh` CLI or `git push`
2. Read CI build output (via Bash + `gh` CLI or webhook integration)
3. If CI fails: parse error output, apply fix, recommit
4. If fix fails 3 times: escalate to human with CI logs

This means CI is not just a gate for humans to check — it's an input signal that Claude Code actively reads and responds to. The pipeline becomes part of the verification loop, not a separate manual process.

> Detailed in [`sdlc/05-git-and-cicd.md`](05-git-and-cicd.md).
