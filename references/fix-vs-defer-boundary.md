# FIX vs DEFER Boundary

When a finding is **VALID** but not always worth fixing in the current PR.

| Situation | Fix decision |
|-----------|--------------|
| Readability only, no behavior impact | DEFER P3 |
| Small readability fix, 2–3 lines | FIX P3, optional in this PR |
| Large refactor, same behavior | DEFER + follow-up issue |
| Performance: no profile or evidence | DEFER |
| Performance: hot path O(n²) or unbounded growth | FIX P1–P2 |
| Legacy debt, untouched in this PR | DEFER (pre-existing) |
| Legacy debt + this PR extends the area | FIX new issues at minimum |

## ASK question templates

Use when verdict is **NEEDS_CONTEXT**:

- "Is behavior A a product requirement or a temporary implementation?"
- "Will this endpoint be exposed to the public internet?"
- "Does this PR include refactoring scope for module X?"
- "On failure, should the user see silent failure or an explicit error?"

## Fix principles (FIX items only)

1. Minimal diff; no drive-by refactors
2. Match existing repo patterns
3. Every fix traceable to finding #
