---
name: format-ai-review-report
description: >
  Output format specification for AI review reports. Defines the standard table-based
  structure used by the ai-reviewer agent when posting review findings to GitHub PRs,
  Redmine issues, or local files. Not a standalone skill — imported by agents that
  produce review artifacts.
---

# Review Report Format

This document defines the canonical output format for all AI review reports. Any agent that produces a review (skill audit, PR review, code review, prompt review) MUST follow this format.

---

## Report Structure

Every report has these sections in order:

1. **Header** — title, reviewer, date
2. **Critical** — blocking issues (if any)
3. **Warnings** — quality issues that should be fixed (if any)
4. **Suggestions** — nice-to-have improvements (if any)
5. **Strengths** — what was done well
6. **Scorecard** — category ratings
7. **Footer** — attribution line

If a severity section has no findings, omit it entirely. Do not print empty tables.

---

## Template

```markdown
## <Review Type>: `<artifact name>`

**Reviewer:** <agent name>
**Date:** <YYYY-MM-DD>

---

### Critical

| # | Issue | Location | Details |
|---|-------|----------|---------|
| 1 | Short issue title | File path, line, or section | Concrete description of what is wrong and how to fix it. |

### Warnings

| # | Issue | Location | Details |
|---|-------|----------|---------|
| 1 | Short issue title | Location reference | Concrete description and fix suggestion. |

### Suggestions

| # | Suggestion |
|---|------------|
| 1 | Actionable improvement with enough context to implement. |

### Strengths

| Area | Notes |
|------|-------|
| Area name | What was done well and why it matters. |

### Scorecard

| Category | Score |
|----------|-------|
| Category Name | N/5 |

---

Generated with [Claude Code](https://claude.com/claude-code)
```

---

## Formatting Rules

1. **Tables only** — findings go in tables, not bullet lists or blockquotes. Tables are scannable, compact, and render well on GitHub and Redmine.

2. **No decorative emojis** — no icons in headers, severity labels, or table cells. Status symbols (per project convention) are limited to `✅`, `❌`, `⚠️` and only in chat output, not in persistent reports.

3. **One row per finding** — do not merge cells or nest tables. If a finding needs extended explanation, keep the Details cell concise and link to a line number or file.

4. **Consistent column widths** — keep Issue/Area columns short (3-6 words). Put the substance in Details/Notes.

5. **Location references** — use the format `Line N`, `Lines N-M`, `Section title`, or `File:Line`. Do not use full file paths when the file is obvious from the report header.

6. **Scorecard categories** — use ONLY the categories relevant to the artifact type being reviewed:

   | Artifact type | Categories |
   |---------------|------------|
   | CLAUDE.md files | Structure and Format, Content Quality, Security and Safety, Maintainability, Team Standards |
   | Skills (SKILL.md) | Structure and Format, Prompt Quality, Security and Safety, Architecture and Design, Team Standards |
   | Subagent definitions | Structure and Format, Prompt Quality, Security and Safety, Tool Permissions, Team Standards |
   | Hooks and settings | Security and Safety, Architecture and Design, Team Standards |
   | Code reviews | Correctness, Security, Performance, Readability, Test Coverage |

   Do NOT include categories that are irrelevant to the artifact. For example, "CLAUDE.md Quality" only applies when reviewing CLAUDE.md files themselves — not skills or agents.

7. **Header fields** — Reviewer is the agent name (e.g., `Claude Code (ai-reviewer agent)`). Date is ISO format.

8. **Footer** — always end with `Generated with [Claude Code](https://claude.com/claude-code)`.

9. **Language** — report body is always in English regardless of the conversation language.

---

## Severity Definitions

| Severity | When to use |
|----------|-------------|
| Critical | Security risk, data leak, broken functionality. Must fix before merge. |
| Warning | Quality issue, suboptimal pattern, potential problem. Should fix. |
| Suggestion | Improvement opportunity, style preference, nice-to-have. |
| Strength | Well-done pattern worth keeping or replicating across the project. |

---

## Adaptation by Context

- **GitHub PR comment**: Use the full template as-is. Prefix the report title with `## `.
- **Redmine issue note**: Convert markdown tables to Textile. Use `|` pipe tables (Redmine supports them natively).
- **Local file**: Save as `.md` in the relevant `docs/local/` directory.
- **Chat output**: Print a brief summary (1-3 sentences) after posting the full report elsewhere. Do not duplicate the full report in chat.
