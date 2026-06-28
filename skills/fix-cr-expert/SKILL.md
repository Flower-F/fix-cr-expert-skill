---
name: fix-cr-expert
description: "Validate code review findings and PR review comments before fixing. Use for CodeRabbit, Copilot, CodeQL, human review notes, or AI review output when deciding whether findings are real, false positives, by design, deferred, or worth fixing. Produces FIX/SKIP/DEFER/ASK decisions with evidence and edits only after explicit user confirmation."
---

# Fix CR Expert

## Overview

Validate existing code review findings against real code, decide whether each item should be **fixed**, **skipped**, **deferred**, or **escalated to the user**, then implement only confirmed fixes after explicit user approval. Default to report-only output unless the user authorizes implementation.

**Core principle**: Code review is heuristic, not verdict. Every finding must be re-verified with code facts, call chains, and project conventions before a fix decision is made.

If no CR input is provided, ask the user to paste findings or review comments. Do not invent review items.

## Verdict + Fix Decision

Each finding gets two separate labels:

1. **Verdict**: what the code facts show.
2. **Fix decision**: what to do in this PR.

Verdict:

| Verdict | Meaning |
|---------|---------|
| **VALID** | Confirmed defect or convention violation |
| **INVALID** | False positive |
| **BY_DESIGN** | Intentional tradeoff or accepted pattern |
| **ALREADY_FIXED** | Already addressed in the diff |
| **NEEDS_CONTEXT** | Missing product/architecture context |

Fix decision:

| Decision | Meaning | Typical action |
|----------|---------|----------------|
| **FIX** | Confirmed issue worth addressing now | Implement by adjusted priority |
| **SKIP** | No code change | False positive, by design, or already fixed |
| **DEFER** | Valid issue outside current PR scope | Track follow-up |
| **ASK** | Needs user/product/architecture input | Ask, then re-decide |

Adjusted priority (for FIX / DEFER only):

| Level | Description | Action |
|-------|-------------|--------|
| **P0** | Critical | Must fix in this PR |
| **P1** | High | Strongly recommended in this PR |
| **P2** | Medium | Fix in this PR or follow-up |
| **P3** | Low | Optional improvement |
| **—** | N/A | SKIP / pending ASK |

## Workflow

### 1) Preflight context

- Collect CR findings from pasted output, structured review reports, or PR comments.
- If input is unstructured, normalize into a numbered list (original severity, file, line, description).
- Use `git status -sb` to identify the branch and local state.
- Determine the comparison scope before evaluating findings:
  - If the user provides a PR number/link, read the PR diff/comments when available.
  - If working on a feature branch, identify the base branch and use `git diff <base>...HEAD --stat` plus targeted diffs.
  - If only working-tree changes are available, use `git diff --stat` and `git diff`, and state that scope limitation in the report.
- For large diffs (>500 lines), process in batches by module or feature area.

**Edge cases:**
- **No CR input**: Ask the user to paste findings. Do not proceed with invented items.
- **Finding points to deleted code**: Mark ALREADY_FIXED → SKIP.
- **Finding in untouched legacy code**: Prefer DEFER with `pre-existing` note.
- **Duplicate findings**: Merge into one item and note duplicate sources.
- **CR scope mismatches diff**: Call out the mismatch before evaluating items.

### 2) Validate + decide (required for every item)

Before editing code, for each finding:

1. **Locate**: Read the cited file/line plus the full function or component, not just the single line.
2. **Trace**: Follow call chains, data flow, and error paths 1–2 levels up/down (`rg`, semantic search).
3. **Check conventions**: Read relevant `AGENTS.md`, nearby patterns, types, and interface contracts.
4. **Test assumptions**: What did the reviewer assume? Does the code support it?
5. **Decide**: Assign verdict + fix decision + 1–2 sentences of verifiable evidence.

Load references only when they are useful:

- Always use `references/fix-decision-checklist.md` for the per-item validation shape.
- Use `references/false-positive-patterns.md` when a reviewer may have missed guards, existing tests, framework behavior, or project constraints.
- Use `references/by-design-signals.md` when a finding conflicts with documented or established design.
- Use `references/fix-vs-defer-boundary.md` when a VALID item may be outside PR scope or needs an ASK prompt.

### 3) Output format

Structure your report as follows:

```markdown
## CR Fix Decision Summary

**Source**: [user paste / PR #N / …]
**Change scope**: X files, Y lines
**Findings total**: N

| Fix decision | Count |
|--------------|-------|
| FIX | |
| SKIP | |
| DEFER | |
| ASK | |

**Recommended for this PR**: P0: _, P1: _, P2: _
**Recommended skip**: _

---

## Item-by-item decisions

### #1 [orig P1] `path/file.ts:42` — Original title

- **Original CR claim**: …
- **Verdict**: VALID | INVALID | BY_DESIGN | ALREADY_FIXED | NEEDS_CONTEXT
- **Fix decision**: FIX | SKIP | DEFER | ASK
- **Adjusted priority**: P0 / P1 / P2 / P3 / —
- **Evidence**: …
- **Skip/defer rationale or follow-up**: … (required for SKIP/DEFER)

### #2 …

---

## Needs your input (ASK)

1. …

## Follow-up backlog (DEFER)

- [ ] …
```

**Inline comments**: Use this format for file-specific decisions:
```
::code-comment{file="path/to/file.ts" line="42" severity="P1"}
Verdict: INVALID. Skip rationale: upstream guard already handles null.
::
```

**Clean report**: If every item is SKIP, explicitly state:
- What was verified
- Why the CR signal-to-noise ratio is high/low
- Residual risks or recommended follow-up tests

### 4) Synthesis

Provide a short objective summary:

- CR signal-to-noise ratio (high / moderate / low)
- Whether merge recommendation should change (e.g. REQUEST_CHANGES → only P3 left)
- **Minimum necessary fix set** for this PR (usually FIX at P0/P1)

### 5) Next steps confirmation

After presenting the report, ask the user how to proceed:

```markdown
---

## Next Steps

Evaluation complete: _ items to fix, _ to skip, _ awaiting your input.

**How would you like to proceed?**

1. **Fix FIX P0/P1 only** - Address high-priority confirmed issues
2. **Fix specific items** - Tell me which # to fix
3. **Fix all FIX items** - Include P2/P3
4. **No changes** - Report only, no implementation

Please choose an option or provide specific instructions (including whether BY_DESIGN items need comments).
```

**Important**: Do NOT implement any changes until the user explicitly confirms. After confirmation, fix only FIX items (or user-specified numbers).

### 6) Implement confirmed fixes

After the user confirms implementation:

1. Restate the finding numbers and priorities that will be fixed.
2. Edit only confirmed FIX items or user-specified item numbers. Do not touch SKIP/DEFER items.
3. Keep each code change traceable to a finding number.
4. Match existing repo patterns; avoid drive-by refactors.
5. Run the smallest relevant validation for the touched area.
6. Final response must list fixed items, unchanged SKIP/DEFER/ASK items, and validation results.

## Decision heuristics

- **Bias toward FIX**: Security, auth, data consistency, multi-instance/concurrency, user-visible failures, fire-and-forget without terminal state.
- **Bias toward SKIP (INVALID)**: Unreachable paths, existing guards/tests, reviewer missed context.
- **Bias toward SKIP (BY_DESIGN)**: Matches AGENTS.md, tech spec, or established module patterns; explicit scope cut.
- **Bias toward DEFER**: Style-only, large refactors, performance without evidence, legacy debt outside this PR.
- **When uncertain**: ASK. Do not force FIX or SKIP.

## Resources

### references/

| File | Purpose |
|------|---------|
| `fix-decision-checklist.md` | Per-item validation checklist |
| `false-positive-patterns.md` | Common false positives and over-skip warnings |
| `by-design-signals.md` | Intentional tradeoffs vs defects |
| `fix-vs-defer-boundary.md` | FIX vs DEFER boundaries, ASK prompts, fix principles |
