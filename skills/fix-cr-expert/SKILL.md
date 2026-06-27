---
name: fix-cr-expert
description: "Expert at validating CR findings and fixing only confirmed issues. Verifies each item against the codebase, outputs fix/skip/defer decisions, implements after user confirmation. Use when triaging CR comments, disputing review findings, or deciding what to fix before merge."
---

# Fix CR Expert

## Overview

Validate existing code review findings against real code, decide whether each item should be **fixed**, **skipped**, **deferred**, or **escalated to the user**, then implement only confirmed fixes after explicit user approval. Default to report-only output unless the user authorizes implementation.

**Core principle**: Code review is heuristic, not verdict. Every finding must be re-verified with code facts, call chains, and project conventions before a fix decision is made.

If no CR input is provided, ask the user to paste findings or review comments. Do not invent review items.

## Verdict + Fix Decision

Each finding gets a **verdict** (is it real?) and a **fix decision** (should we act?):

| Verdict | Meaning | Fix decision |
|---------|---------|--------------|
| **VALID** | Confirmed defect or convention violation | **FIX** (by adjusted priority) |
| **INVALID** | False positive | **SKIP** |
| **BY_DESIGN** | Intentional tradeoff or accepted pattern | **SKIP** (may suggest a comment) |
| **DEFER** | Valid but out of scope for this PR | **DEFER** (track follow-up) |
| **ALREADY_FIXED** | Already addressed in the diff | **SKIP** |
| **NEEDS_CONTEXT** | Missing product/architecture context | **ASK** (ask user, then decide) |

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
- Use `git status -sb`, `git diff --stat`, and `git diff` to scope the change set.
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

Load reference checklists for validation heuristics:

- `references/fix-decision-checklist.md` — per-item checklist
- `references/false-positive-patterns.md` — common false positives
- `references/by-design-signals.md` — intentional tradeoffs
- `references/fix-vs-defer-boundary.md` — FIX vs DEFER, ASK prompts, fix principles

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
- **Verdict**: VALID | INVALID | BY_DESIGN | …
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
