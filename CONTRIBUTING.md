# Contributing to claude-code-supremacy

Thank you for your interest in improving this repository. This is a research project, and contributions that improve accuracy, add evidence, or correct errors are highly valued.

---

## What We're Looking For

### High-value contributions
- **Corrections to factual claims** — If a prompt excerpt is inaccurate, a feature description is wrong, or a comparison claim has changed due to a tool update, please raise an issue or open a PR with the corrected information and a source link.
- **New version tracking** — Claude Code updates its system prompt regularly. If you've detected a meaningful change (via the Piebald-AI tracker or your own analysis), document it in [`research/changelog.md`](research/changelog.md).
- **Missing comparisons** — If a major AI coding tool is underrepresented or misrepresented, a well-sourced addition to `comparisons/` is welcome.
- **Use case additions** — Concrete, real-world scenarios with specific toolchain walkthroughs that haven't been covered.
- **Open question resolution** — If you have credible data that resolves one of the items in [`research/open-questions.md`](research/open-questions.md), please contribute it.

### Lower-priority contributions
- Stylistic edits (we have a consistent tone — please read existing files before proposing changes)
- Theoretical additions without grounding in system prompt data or documented behaviour

---

## Standards

All contributions must meet the standards defined in the main PRD:

### Accuracy requirements
- Every comparative claim must be tied to a specific prompt excerpt, architectural observation, or documented feature
- Distinguish clearly between "what the system prompt says" and "what we observe in practice"
- When a competitor is genuinely better at something, say so — we do not want a marketing document

### Source requirements
- Primary sources: [x1xhlol/system-prompts-and-models-of-ai-tools](https://github.com/x1xhlol/system-prompts-and-models-of-ai-tools) and [Piebald-AI/claude-code-system-prompts](https://github.com/Piebald-AI/claude-code-system-prompts)
- All other sources should be linked inline and listed in [`research/sources.md`](research/sources.md)
- Do not use sources that are purely speculative, vendor marketing, or uncorroborated social media posts

### Language
- No hype language (see banned phrases list in the repo's writing standards)
- Precise over broad — "The Plan Mode architecture enforces X because..." rather than "Claude Code is much better at planning"
- Tables for structured comparisons, not prose lists

---

## How to Contribute

1. **Open an issue first** for anything beyond a trivial correction. Describe what you're changing and why.
2. **Fork the repository** and create a branch named descriptively: `fix/cursor-tool-count`, `add/devin-comparison`, `update/task-system-v3`.
3. **Write the content** following the standards above.
4. **Update `research/sources.md`** if you're adding new sources.
5. **Update `research/changelog.md`** if you're documenting a version change.
6. **Open a pull request** with a clear description of what changed and why, and what source it's based on.

---

## Version Discipline

This repo is anchored to Claude Code **v2.0.56** (December 1, 2025) as the baseline. If you're contributing data from a newer version:

- Note the version in the relevant file using a `> **Version note:**` blockquote
- Update [`research/changelog.md`](research/changelog.md) with the delta
- If the change invalidates a claim in a comparison file, update that file too

---

## What We Will Not Accept

- Claims based solely on personal experience without corroborating evidence
- Additions that make the repository sound like marketing material
- Prompt excerpts from sources of unknown reliability
- Comparisons that dismiss competitors without engaging with their actual architecture
- Any content that would embarrass a technically sophisticated reader

---

## Questions

Open an issue tagged `question`. We'll respond.
