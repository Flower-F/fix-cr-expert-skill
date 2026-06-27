# Fix CR Expert

Expert skill for AI agents to validate code review findings and fix only confirmed issues. Re-verifies each CR item against the codebase, separates real defects from false positives and by-design choices, and implements fixes only after user confirmation.

## Installation

```bash
npx skills add Flower-F/fix-cr-expert-skill --skill fix-cr-expert
```

## Why use

Code review—whether from humans, bots, or AI skills—tends to **over-report**. Findings are heuristics, not verdicts. Applying every suggestion leads to:

- **Wasted effort** on false positives and style nits
- **Scope creep** from refactors that were never required for this PR
- **Regressions** when "fixes" misunderstand context, guards, or by-design tradeoffs
- **Merge paralysis** when you cannot tell which items actually block shipping

`fix-cr-expert` adds a **validation gate** before any code changes:

1. **Re-check each finding** against real code, call chains, and project conventions—not just the review text
2. **Label every item** with a verdict (is it real?) and a fix decision (FIX / SKIP / DEFER / ASK)
3. **Produce evidence** so skip and defer choices are auditable, not vibes
4. **Recommend a minimum fix set** (usually P0/P1) instead of treating the whole CR list as mandatory
5. **Fix only after you confirm**—no silent auto-fix of every CR comment

Use it when you have review output in hand and need to answer: *what is actually wrong, what should we fix in this PR, and what can we ignore or defer?*

Pair with `code-review-expert` for a full loop: wide scan → validate → fix on confirm. You can also run `fix-cr-expert` alone on pasted PR comments, bot output, or audit notes.

## Features

- **Finding validation** - Re-verify each CR item with code facts and call chains
- **Verdict classification** - VALID, INVALID, BY_DESIGN, DEFER, ALREADY_FIXED, NEEDS_CONTEXT
- **Fix decisions** - FIX, SKIP, DEFER, ASK for every item
- **Priority adjustment** - Re-rank P0–P3 after validation
- **False positive detection** - Common reviewer mistakes and missed context
- **By-design recognition** - Intentional tradeoffs vs real defects
- **Minimum fix set** - Recommend the smallest necessary change set for merge
- **Fix-after-confirm** - Report first, implement only with explicit user approval

## Usage

After CR findings are available, run:

```
/fix-cr-expert
```

### Input sources

The skill accepts findings from any of these (paste or point the agent at them):

| Source | How to use |
|--------|------------|
| **AI review output** | Output from `code-review-expert` or similar skills |
| **PR review comments** | Paste thread text, or ask agent to read via `gh pr view <N> --comments` |
| **Bot reviews** | CodeRabbit, Copilot, Greptile, BugBot, CodeQL, etc. |
| **Markdown audit files** | `review.md`, `audit.md`, `feedback.md` in the repo |
| **Human review notes** | Email, chat, or inline notes from a teammate |

If input is unstructured, the skill normalizes it into a numbered list before validation.

## Example output

Abbreviated report after validating two findings from a prior review:

```markdown
## CR Fix Decision Summary

**Source**: code-review-expert
**Change scope**: 3 files, 84 lines
**Findings total**: 2

| Fix decision | Count |
|--------------|-------|
| FIX | 1 |
| SKIP | 1 |

**Recommended for this PR**: P0: 0, P1: 1, P2: 0

---

### #1 [orig P1] `api/handler.ts:58` — Missing auth check on delete

- **Verdict**: VALID
- **Fix decision**: FIX
- **Adjusted priority**: P1
- **Evidence**: `deleteRecord` reads `id` from params and calls `service.delete` with no tenant or user check; other handlers in the same file use `requireAuth` first.

### #2 [orig P2] `api/handler.ts:12` — Should split handler into smaller modules

- **Verdict**: BY_DESIGN
- **Fix decision**: SKIP
- **Evidence**: File matches size and layout of `user/handler.ts` and `order/handler.ts`; split is refactor-only, not required for this bugfix PR.
- **Skip rationale**: Scope reduction for MVP; defer split to follow-up if team wants consistency pass.
```

After this report, the agent asks for confirmation before implementing **#1 only**.

## Recommended pairing with code-review-expert

This skill works best as the **second step** after [code-review-expert](https://github.com/sanyuan0704/sanyuan-skills/tree/main/skills/code-review-expert):

```
/code-review-expert   →   /fix-cr-expert   →   user confirms   →   apply FIX items
```

| Step | Skill | Output |
|------|-------|--------|
| 1 | `code-review-expert` | Structured findings (P0–P3) |
| 2 | `fix-cr-expert` | Verdict + FIX/SKIP/DEFER/ASK per item |
| 3 | User confirms | Only FIX items (or selected #) are implemented |

Install code-review-expert:

```bash
npx skills add sanyuan0704/sanyuan-skills --path skills/code-review-expert
```

## Workflow

1. **Preflight** - Collect findings and scope changes via `git diff`
2. **Validate + decide** - Verify each item; assign verdict and fix decision
3. **Output** - Fix/skip/defer report with evidence
4. **Synthesis** - Signal-to-noise summary and minimum fix set
5. **Confirmation** - Ask user before implementing fixes

## Fix Decisions

| Decision | Meaning | Action |
|----------|---------|--------|
| FIX | Confirmed issue, should address | Fix in this PR (by priority) |
| SKIP | Do not fix | False positive, by design, or already fixed |
| DEFER | Valid but not now | Track follow-up |
| ASK | Needs user input | Ask, then re-decide |

## Verdicts

| Verdict | Meaning |
|---------|---------|
| VALID | Confirmed defect or violation |
| INVALID | False positive |
| BY_DESIGN | Intentional design |
| DEFER | Valid but out of PR scope |
| ALREADY_FIXED | Already in diff |
| NEEDS_CONTEXT | Missing decision context |

## Structure

Repository layout (for `npx skills add`):

```
fix-cr-expert-skill/              # GitHub repo root
├── README.md
└── skills/
    └── fix-cr-expert/
        ├── LICENSE
        ├── SKILL.md              # Main skill definition
        ├── agents/
        │   └── agent.yaml
        └── references/
            ├── fix-decision-checklist.md
            ├── false-positive-patterns.md
            ├── by-design-signals.md
            └── fix-vs-defer-boundary.md
```

## References

Each checklist covers a distinct part of the validation workflow:

- **fix-decision-checklist.md** - Per-item checklist before deciding
- **false-positive-patterns.md** - Common INVALID cases and over-skip warnings
- **by-design-signals.md** - Intentional tradeoffs vs real defects
- **fix-vs-defer-boundary.md** - When to FIX vs DEFER, ASK prompts, fix principles

## License

MIT — see [LICENSE](skills/fix-cr-expert/LICENSE).
