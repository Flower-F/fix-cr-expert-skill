# Fix Decision Checklist

Per-item checklist for validating a review finding and deciding fix/skip/defer.

Copy and complete for each finding:

```
Finding #___: _________________________

Context
- [ ] Read at least one full function/component around the cited line
- [ ] Confirmed the line is in this diff (or marked pre-existing)
- [ ] Traced caller or callee at least one level
- [ ] Checked existing patterns in the same file/module

Facts
- [ ] Can state actual behavior, not ideal behavior
- [ ] Separated compile/type vs runtime vs maintainability issues
- [ ] Separated this-PR change vs legacy debt

Verdict (is it real?)
- [ ] Assigned exactly one verdict
- [ ] Wrote 1–2 sentences of verifiable evidence

Fix decision (should we act?)
- [ ] Assigned FIX / SKIP / DEFER / ASK
- [ ] SKIP/DEFER includes skip rationale or follow-up
- [ ] FIX/DEFER includes adjusted P-level
```

See also:

- `false-positive-patterns.md` — common INVALID / SKIP cases
- `by-design-signals.md` — intentional tradeoffs vs defects
- `fix-vs-defer-boundary.md` — FIX vs DEFER, ASK prompts, fix principles
