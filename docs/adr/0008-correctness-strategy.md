# ADR-0008: Library-Managed Seeding, Golden-Run Tests, and a Leakage Guard

**Status:** Accepted
**Date:** 2026-06-06
**Deciders:** Project owner

## Context

Every consuming app shares the one training loop, so a subtle bug in it — wrong metric averaging,
broken seeding, validation data leaking into training — corrupts results for *all* consumers at
once (FM-4). The library needs explicit correctness guarantees and tests that prove them, and a
policy for who controls randomness so runs are reproducible. This resolves Q-8.

## Decision

Adopt three measures: (1) **library-managed seeding** via a config knob (the engine seeds Python/
NumPy/torch deterministically when given a seed); (2) **golden-run regression tests** that train a
tiny known model and assert loss/metrics match expected values to tight tolerance; and (3) an
explicit **train/val leakage guard** asserting validation data never overlaps training (paired with
the temporal-split responsibility in ADR-0002).

## Rationale

Golden-run tests catch any silent change to loop math — the most dangerous, hardest-to-spot class of
FM-4 bug — by pinning exact numerical behavior. Library-managed seeding (opt-in via config) makes
those golden runs reproducible and gives consumers deterministic runs for free, rather than leaving
each app to get global seeding right. The leakage guard turns the single most damaging time-series
mistake into a test failure rather than a silent accuracy illusion. Together they convert FM-4 from
"discovered in production across many apps" to "caught in CI once."

## Consequences

**Positive:**
- Loop-math regressions are caught centrally before they reach any consumer (FM-4).
- Reproducible runs out of the box when a seed is provided.

**Negative:**
- Golden-run tests are sensitive: legitimate changes require deliberately updating expected values.
- Full determinism can disable some nondeterministic-but-faster CUDA kernels (a speed/repro trade-off, opt-in).

**Neutral:**
- Seeding is opt-in: callers who pass no seed get normal nondeterministic behavior.

## Alternatives Considered

### Option A: Caller-owned seeding + property/invariant tests only
Cleaner ownership (the app seeds), but weaker regression coverage and leaves reproducibility of the
shared loop to each consumer. Not rejected wholesale — invariant tests (e.g. "val never overlaps
train") are adopted on top — but insufficient alone against FM-4.

### Option B: No determinism guarantees, integration tests only
Lowest effort, but the highest-impact failure mode (silent loop-math corruption) would go uncaught.
Rejected.

## References

- Resolves [open-questions.md](../open-questions.md) Q-8
- Artifact: [FM-4](../artifact.md)
- Related: ADR-0002 (temporal split), ADR-0009 (metric aggregation correctness)
