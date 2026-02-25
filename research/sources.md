# Sources and Methodology

All comparative claims in this repository are grounded in documented sources. This file lists those sources, their reliability ratings, and the methodology used to draw conclusions from them.

---

## Primary Sources

### 1. x1xhlol/system-prompts-and-models-of-ai-tools

**URL:** [https://github.com/x1xhlol/system-prompts-and-models-of-ai-tools](https://github.com/x1xhlol/system-prompts-and-models-of-ai-tools)  
**Type:** Community-maintained repository of reverse-engineered AI system prompts  
**Coverage:** 30+ AI tools including Claude Code, Cursor, GitHub Copilot, Gemini Code Assist, Windsurf, Devin, Manus, and others  
**Reliability rating:** MODERATE for most tools; HIGH for tools where npm package extraction was used  
**Used for:** Cursor prompt anatomy, Windsurf prompt analysis, cross-tool comparison grounding

**Methodology:** The repository uses several extraction techniques:
- Direct system prompt injection (asking the AI to reveal its prompt)
- npm/binary package analysis (for tools that bundle their prompts client-side)
- Community corroboration (multiple independent extractions compared)

The reliability varies by tool. Tools with npm-bundled prompts (Claude Code) have near-deterministic extraction. Tools that require prompt injection have lower confidence.

### 2. Piebald-AI/claude-code-system-prompts

**URL:** [https://github.com/Piebald-AI/claude-code-system-prompts](https://github.com/Piebald-AI/claude-code-system-prompts)  
**Type:** Structured analysis of Claude Code's npm-bundled system prompt strings  
**Coverage:** Claude Code specifically; 110+ prompt strings across 37 tracked versions  
**Reliability rating:** VERY HIGH — extraction from npm package is deterministic  
**Baseline version used:** Claude Code v2.0.56 (December 1, 2025)  
**Used for:** All Claude Code architectural claims, sub-agent token counts, prompt component categories

**Methodology:** Claude Code's npm package (`@anthropic-ai/claude-code`) contains the system prompt strings embedded in its JavaScript bundle. These are extractable by inspecting the distributed package. The Piebald-AI project automates this extraction and tracks version diffs. Sub-agent token counts are measured directly from the extracted strings.

---

## Secondary Sources

### 3. Anthropic Official Documentation

**URL:** [https://docs.anthropic.com/claude/docs/claude-code](https://docs.anthropic.com/claude/docs/claude-code)  
**Type:** Official product documentation  
**Reliability rating:** HIGH for documented features; does not cover system prompt details  
**Used for:** Tool descriptions, installation instructions, official feature descriptions

**Caveat:** Official documentation describes intended behaviour, not observed behaviour. Where documentation and prompt data conflict, prompt data is treated as higher confidence.

### 4. GitHub Copilot Documentation

**URL:** [https://docs.github.com/en/copilot](https://docs.github.com/en/copilot)  
**Type:** Official product documentation  
**Reliability rating:** HIGH for documented features  
**Used for:** Copilot capability descriptions, enterprise features, pricing

### 5. Community Analysis

**Sources:** GitHub discussions, engineering blogs, Reddit (r/ClaudeAI, r/ChatGPT), Hacker News threads  
**Reliability rating:** LOW — treated as corroborating evidence only, not primary claims  
**Used for:** Real-world usage patterns, qualitative assessments, edge cases

---

## What "Reverse-Engineered" Means

When this repository says a system prompt was "reverse-engineered," it means one of:

1. **npm package extraction:** The tool distributes its system prompt in a client-side JavaScript bundle. The bundle is deobfuscated and prompt strings are identified. This is deterministic — the same package yields the same strings. Used for Claude Code.

2. **Prompt injection:** The tool is asked (directly or through indirect techniques) to reveal its instructions. Results depend on the model's willingness to comply and the extraction technique. Reliability varies significantly. Used for Cursor and others.

3. **Observed behaviour + documentation:** The tool's behaviour is documented without direct prompt access. Lower confidence. Used for Gemini Code Assist and some GitHub Copilot analysis.

---

## Reliability Ratings System

| Rating | Meaning | Used when |
|---|---|---|
| VERY HIGH | Deterministic extraction from client-side source | Claude Code npm package |
| HIGH | Multiple corroborated extractions or official documentation | Cursor (confirmed by community), Copilot official docs |
| MODERATE | Single extraction with some corroboration | Windsurf, some Gemini analysis |
| LOW | Behavioural inference or single uncorroborated source | Devin AI, some older extractions |
| ❓ | Unknown — insufficient data | Many prompt specifics for non-primary tools |

---

## Version Tracking

This repository uses the following version baseline:

| Tool | Version | Source | Date analysed |
|---|---|---|---|
| Claude Code | v2.0.56 | Piebald-AI | December 2025, analysed February 2026 |
| Cursor | Late 2025 extraction | x1xhlol repo | Analysed February 2026 |
| GitHub Copilot | 2025 platform | Official docs | Analysed February 2026 |
| Gemini Code Assist | 2025 platform | Official docs + community | Analysed February 2026 |
| Windsurf | Late 2025 extraction | x1xhlol repo | Analysed February 2026 |

All tools update continuously. Claims about capabilities may be outdated. Check [`research/changelog.md`](changelog.md) for documented updates.

---

## Contributing New Sources

If you have:
- A more recent or more reliable extraction of any tool's system prompt
- Official documentation that contradicts a claim in this repo
- A credible analysis with a different conclusion

Please follow the process in [`CONTRIBUTING.md`](../CONTRIBUTING.md). Include the source URL, extraction method, reliability justification, and the specific claim being updated.
