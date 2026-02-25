# The Human Perspective

What does using Claude Code for meaningful engineering work actually feel like, and what does it actually change about the job? This document is an honest account — not a sales pitch.

---

## The Shift in Time Allocation

The most concrete change is where an engineer's time goes:

| Activity | Typical Before | With Claude Code |
|---|---|---|
| Writing implementation code | ~40% | <5% |
| Writing tests | ~15% | <2% |
| Reading documentation | ~10% | ~3% |
| Debugging | ~20% | ~5% |
| Architectural thinking | ~10% | ~25% |
| Defining intent clearly | ~5% | ~30% |
| Reviewing AI output | 0% | ~20% |
| Admin / human coordination | ~0% | ~10% |

The work doesn't disappear. It shifts from *doing* implementation to *directing and reviewing* implementation. For engineers who find implementation work energising, this change requires genuine adjustment.

---

## The Architect Elevation

The abstraction level at which the engineer operates rises. Instead of asking:

> "How should I implement the pagination cursor in this SQL query?"

They're asking:

> "Should this API use cursor-based or offset pagination? What are the tradeoffs for our access patterns?"

The first question is implementation. The second is architecture. Claude Code handles the first; the engineer can spend more time on the second.

This is an elevation, not a replacement. But it requires the engineer to actually be capable of operating at the architectural level — which not everyone is, and which takes time to develop even for those who are.

---

## New Skills That Matter

The engineers who get the most leverage from Claude Code develop skills that didn't matter as much before:

### 1. Intent Specification

The ability to describe what you want clearly enough that an AI can implement it correctly, completely, and without constant clarification. This is harder than it sounds.

Bad specification: "Add notifications."  
Good specification: "Add a notification system that supports in-app notifications (stored in DB, displayed in header badge), email notifications (sent via Sendgrid), and push notifications (Firebase FCM). Notifications can be read/unread. They can be triggered by: new order, order status change, new follower, and message received."

The second is a complete spec with types, triggers, and delivery channels. Writing specs this precisely is a distinct skill — closer to product management than engineering.

### 2. Plan Evaluation

During Plan Mode, Claude Code presents a plan. The engineer must:
- Evaluate whether the plan is correct and complete
- Identify missing steps or incorrect assumptions
- Spot cases where the agent misunderstood the requirement
- Decide whether to approve, modify, or reject

This requires understanding the codebase and the task well enough to evaluate a plan — which is similar to reviewing another engineer's proposed approach. It requires architectural judgment.

### 3. Codebase Ownership

If you don't understand the codebase deeply, you can't review Claude Code's output effectively. Worse, the illusion that you can is dangerous — you may approve changes that look correct but have subtle structural problems.

Paradoxically, Claude Code requires *more* codebase familiarity for responsible use, not less. Engineers who don't understand what they're reviewing can create slow-building quality debt.

### 4. Taste

Knowing when output is "good enough" and when it needs revision is a judgment call that has no objective answer. Engineers who have developed taste — for clean abstractions, appropriate complexity, code that will be maintainable — use Claude Code more effectively than engineers who haven't.

---

## What Doesn't Change

- **Responsibility.** You review and approve the code. You own it. If it ships a bug, that's your bug.
- **System thinking.** Understanding how components interact, where the boundaries are, what the failure modes are — this is still yours.
- **Domain knowledge.** The AI doesn't know your business rules, your customers, your team's constraints. You provide this context.
- **The hard conversation.** Telling stakeholders that a feature will take longer, or that a previous decision was wrong — this is human work.

---

## The Leverage Math

For a senior engineer using Claude Code effectively on sprint-level work:

- **Direct productivity:** 3–5x per-feature throughput (see [`docs/02-the-10x-myth.md`](02-the-10x-myth.md))
- **Parallel leverage:** 1 engineer running 2 parallel tracks (one with Claude Code overnight, one direct)
- **Quality consistency:** Automated testing on every task reduces regression rate
- **Scope elevation:** Time freed from implementation goes to architectural work that improves future sprints

The net effect: one engineer + Claude Code ≈ previous output of a small team for execution work, *if the engineer is spending their time well* (architecture, review, direction) rather than replicating what the AI already handles.

---

## Team Structure Implications

If one engineer ≈ a small team for execution, the team structure math changes:

- A 5-person engineering team might develop the capability of an 8–10-person team
- Or: a solo technical founder might ship at the pace that previously required a first engineering hire
- Or: an engineering team might need fewer implementation engineers and more architects/reviewers

None of these implications are certain — they depend heavily on the nature of the work (greenfield vs. legacy, L4–L5 vs. L1–L3), the quality of the engineers (reviewing AI output well requires seniority), and the team's investment in tooling infrastructure (CLAUDE.md maintenance, MCP servers, CI quality).

---

## The Uncomfortable Questions

Honest acknowledgment of the questions this raises:

**What happens to junior engineers?**

Junior engineers learn by writing code. If senior engineers use Claude Code to generate implementation and review it, junior engineers who join the team face a learning problem: the work that used to teach them is now done by the AI. This is a real concern without a clear resolution.

**Who is responsible for defects in AI-generated code?**

The engineer who reviewed and approved the code. The `Co-Authored-By: Claude` attribution provides a record, but doesn't reduce responsibility. This has legal and ethical dimensions that engineering teams should think through explicitly.

**Does code quality degrade over time?**

A consistent concern: if AI-generated code is approved without deep understanding, structural quality may degrade below the surface. It may look correct and test well but become increasingly hard to modify. This is a real risk, not a theoretical one.

**Does specialised domain expertise atrophy?**

If engineers stop writing certain types of code, their ability to write it may decline. An engineer who hasn't written a SQL query in 18 months because Claude Code always writes them may have weaker SQL skills than before.

---

## The Optimistic Framing

None of the uncomfortable questions are reasons not to use Claude Code. They're reasons to use it thoughtfully:

- Invest in junior engineer development explicitly (code reviews are a teaching opportunity even for AI-generated code)
- Maintain accountability explicitly (review thoroughly, own the output)
- Monitor quality signals (technical debt metrics, maintenance cost over time)
- Stay hands-on in the areas of the codebase that matter most

The engineers who will thrive in an AI-augmented environment are those who pair the leverage of AI execution with strong architectural judgment, clear intent specification, and the willingness to deeply review what's produced. That's a higher bar than "good programmer." It's the bar for engineering direction.
