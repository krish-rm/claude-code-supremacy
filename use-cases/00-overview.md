# When to Use Claude Code

Claude Code is not a universal tool. Understanding which scenarios it's built for — and which it isn't — is the first step to using it effectively.

---

## The Right Question

Instead of "can Claude Code do X?", ask "is X the kind of task where Claude Code's architecture provides a meaningful advantage?"

The architecture provides advantage when:
- The task requires understanding an existing codebase before working on it
- The task involves coordinated changes across multiple files
- Verification (testing, linting, CI) is naturally part of the task
- You want to run the task autonomously and review results, rather than collaborate step-by-step
- The task spans multiple sessions or involves multiple engineers

The architecture provides less advantage when:
- The task is a single, self-contained change (Cursor or Copilot may be faster to start)
- You want to be in the editor, seeing changes in real time as a collaborator
- The task is highly exploratory (research, prototyping with frequent pivots)
- You need inline code completion while writing code yourself

---

## Scenario Overview

| Use Case | Complexity | Claude Code's Primary Advantage | File |
|---|---|---|---|
| Greenfield project | High | Full SDLC from Day 1, architecture coherence | [`greenfield-project.md`](greenfield-project.md) |
| Legacy codebase | High | Explore sub-agent maps complexity before changes | [`legacy-codebase.md`](legacy-codebase.md) |
| Bug hunt | Medium | Root-cause analysis, log reading, hypothesis testing | [`bug-hunt.md`](bug-hunt.md) |
| Feature sprint | Very High | Task management, parallel execution, CI integration | [`feature-sprint.md`](feature-sprint.md) |
| Team coordination | Very High | Shared task list, parallel AI agents, no conflicts | [`team-coordination.md`](team-coordination.md) |

---

## The "How Long Will This Take?" Heuristic

A rough guide for evaluating whether Claude Code is the right tool for a given task:

| Task duration (human estimate) | Claude Code recommendation |
|---|---|
| < 5 minutes | Probably faster to do it yourself or use Copilot |
| 5–30 minutes | Cursor or Claude Code, depending on whether you want to code alongside or delegate |
| 30 min – 4 hours | Claude Code strong choice — task complexity justifies the planning overhead |
| 4 hours – 1 sprint | Claude Code's primary sweet spot — Plan + Tasks + CI integration |
| > 1 sprint | Claude Code with multi-agent coordination — this is where the task system shines |

---

## Honest Limitations

Before diving into use cases, these are scenarios where Claude Code genuinely struggles:

**Highly novel algorithmic work:** If the task requires inventing a new algorithm or solving a problem with no reference implementation, Claude Code will produce reasonable-looking code that may be subtly wrong. Novel algorithms require deep domain expertise and verification that goes beyond automated testing.

**Rapidly changing requirements:** If you pivot your requirements frequently, the Plan → Task → Execute workflow creates overhead. For true exploration, a chat interface or no AI is more appropriate.

**Infrastructure and deployment with real consequences:** While Claude Code can write Terraform, Kubernetes manifests, and deployment scripts, executing them against production infrastructure autonomously is inadvisable. The consequences of a mistake are too severe. Use Claude Code to write and review; humans to apply.

**Highly visual UI work:** Producing and evaluating UI that looks good requires visual iteration. Claude Code can write the code, but verifying it looks right requires a human with a browser. The verification loop for visual work is slower than for logic-heavy work.

**Security-critical implementation from scratch:** Authentication, cryptography, and security-sensitive systems should be reviewed by security professionals regardless of how they were written. Claude Code's defensive-only posture is a floor, not a ceiling for security review.

---

## The Overnight Run Pattern

One of Claude Code's most distinctive use patterns: assign a substantial feature before the end of the workday, let it run overnight, review results the next morning.

This is only possible with:
1. A clear, well-specified task (Plan Mode approved)
2. A complete test suite (CI is the verification oracle)
3. Cross-session persistence (task list doesn't reset)
4. A CI pipeline Claude Code can read and recover from

When all four conditions are met, a senior engineer can effectively run two parallel tracks of development: their own during the day, Claude Code's overnight.

> The detailed walkthroughs for each scenario follow in the individual use-case files.
