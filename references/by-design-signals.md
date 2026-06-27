# By-Design Signals

Patterns that often indicate **BY_DESIGN → SKIP** (optional comment, not logic change).

- Documented in `AGENTS.md`, `CLAUDE.md`, architecture docs, or tech specs
- Matches established pattern in a mature module in the same repo
- Explicit MVP or scope reduction (missing feature ≠ bug)
- Compatibility layer, feature flag, or migration-period dual-write
- Product-required unconventional UX (can point to requirement source)

If by-design but undocumented: suggest a short comment instead of changing logic.
