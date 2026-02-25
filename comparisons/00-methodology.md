# Comparison Methodology

All comparisons in this section are grounded in system prompt data from reverse-engineered sources. This document explains how comparisons were made, what sources were used, and where the methodology has limits.

---

## Primary Sources

### 1. x1xhlol/system-prompts-and-models-of-ai-tools
**Repository:** [https://github.com/x1xhlol/system-prompts-and-models-of-ai-tools](https://github.com/x1xhlol/system-prompts-and-models-of-ai-tools)

This repository contains reverse-engineered system prompts for 30+ AI tools, including Claude Code, Cursor, Gemini Code Assist, GitHub Copilot, Windsurf, Devin, and others. Prompts are extracted through a combination of direct prompt injection attacks, npm package analysis, and community collaboration.

**Reliability rating: HIGH** for tools where the npm package contains the prompt (Claude Code), **MODERATE** for tools where extraction required more indirect methods.

### 2. Piebald-AI/claude-code-system-prompts
**Repository:** [https://github.com/Piebald-AI/claude-code-system-prompts](https://github.com/Piebald-AI/claude-code-system-prompts)

The most comprehensive breakdown of Claude Code's prompt architecture specifically. Tracks 110+ prompt strings across 37 versions, with token counts per component and version diffs. The baseline for this repository is **Claude Code v2.0.56 (December 1, 2025)**.

**Reliability rating: VERY HIGH** — Claude Code's npm package contains the prompt strings directly, making extraction deterministic and verifiable.

---

## What "Reverse-Engineered" Means

For different tools, "reverse-engineered" means different things:

| Tool | Extraction Method | Confidence |
|---|---|---|
| Claude Code | npm package analysis (prompts in JS bundle) | Very High |
| Cursor | Community prompt injection + leaked confirmation | Moderate |
| GitHub Copilot | API metadata + community extraction | Moderate |
| Gemini Code Assist | Limited public disclosure + community | Low-Moderate |
| Windsurf | npm/binary analysis + community | Moderate |
| Devin AI | Limited — mostly from public docs and demos | Low |

For tools with lower extraction confidence, comparisons explicitly note where claims are based on observed behaviour rather than prompt text.

---

## Reliability Ratings Per Claim Type

| Claim Type | Reliability | Example |
|---|---|---|
| "Claude Code's system prompt says X" | Very High | Direct quote from Piebald-AI source |
| "Cursor's prompt says X" | Moderate | From x1xhlol repo, corroborated by others |
| "Tool X has/lacks feature Y" | High | Verified by direct use and community testing |
| "Tool X performs better at Y" | Low | Requires controlled benchmarks we don't have |

We deliberately avoid performance comparisons without rigorous benchmarks. All comparisons in this section are about architecture and documented features, not raw output quality.

---

## Honesty Section: What We Can't Verify

1. **Server-side prompt components** — Claude Code's npm package exposes the client-side prompt strings. There may be server-side system prompt components not visible here.

2. **Real-time prompt changes** — System prompts can be updated without a product release. The Piebald-AI tracker monitors versions, but real-time comparisons may be stale.

3. **Tool-specific model fine-tuning** — Cursor may be running a fine-tuned version of Claude 3.5 Sonnet with adjustments not visible in the system prompt. We cannot verify this.

4. **Actual feature coverage** — A feature described in a system prompt may not work as described in all cases. System prompts express intent, not guaranteed behaviour.

5. **Competitor roadmaps** — Cursor, Windsurf, and others are actively developing. Claims about "missing" features may be outdated within months.

---

## Comparison Scope

Each `vs-*.md` file covers:
- Product philosophy and identity
- System prompt architecture (structure, not just content)
- Specific feature matrix (15+ dimensions)
- Honest assessment of where the competitor wins
- Appropriate use cases for each tool

Claims about competitive advantage are stated mechanically: "Claude Code can do X because its system prompt includes Y; Cursor cannot do X because [architectural reason]." Conclusions follow from mechanisms, not from preferences.

---

## Version Anchoring

**This comparison is anchored at:**
- Claude Code: v2.0.56 (December 1, 2025), analysed February 2026
- Cursor: System prompt version from x1xhlol repo, dated late 2025
- GitHub Copilot: Based on 2025 public documentation and prompt extractions
- Gemini Code Assist: Based on 2025 public documentation and limited extraction

If you find a specific comparison that is outdated, please follow the process in [`CONTRIBUTING.md`](../CONTRIBUTING.md) to submit a correction with a versioned source.

> See [`research/changelog.md`](../research/changelog.md) for version tracking.
