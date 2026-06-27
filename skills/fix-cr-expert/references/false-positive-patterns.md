# False Positive Patterns

Common CR claims that often become **INVALID → SKIP** after code verification.

| CR claim | Re-check before deciding |
|----------|---------------------------|
| "Null not handled" | Upstream guarantees it; entry guard exists |
| "Should extract/split" | Matches nearby module size; split adds coupling |
| "N+1 / performance issue" | Data bounded; batch/cache already present |
| "Missing try/catch" | Framework handles it; error should propagate |
| "SRP violation" | Matches documented module boundary |
| "Race condition" | Single-threaded UI; lock already present |
| "Magic number/string" | Same pattern nearby; from named constant/enum |
| "Needs tests" | Equivalent test exists; pure styling |

## Bias toward FIX (do not over-skip)

Re-verify before SKIP when the claim involves:

- Missing or bypassable auth/tenant/identity checks
- Secrets, tokens, or PII in logs or client code
- Async work with no terminal state; SSE/polling never completes
- Injection, XSS, path traversal, SSRF
- Local-state assumptions in multi-instance deployment
- Logic bugs, off-by-one, unit mistakes
- Fire-and-forget async with no error handler or terminal failure state
