# ADR-0005: Distribute via VCS Pre-1.0, PyPI at 1.0

**Status:** Accepted
**Date:** 2026-06-06
**Deciders:** Project owner

## Context

Reuse across applications means each app installs `trainer_kit` with `pip` (SC-2). The question is
*from where* during the pre-1.0 period, when interfaces are still moving (ADR-0003). The forces:
release friction during rapid interface iteration vs. ease of consumption, and whether the package
is published publicly before the contract is stable. This resolves Q-5.

## Decision

While pre-`1.0`, distribute via **VCS/git install** (apps pin a tag or commit) or a private index.
At **`1.0`**, publish to **PyPI** (public or private — that sub-choice is deferred until 1.0). The
distribution milestone is aligned with the versioning milestone in ADR-0003.

## Rationale

Git/tag installs let interfaces iterate without per-change PyPI release ceremony, while still giving
consumers a pinnable, reproducible reference (SC-2 holds: every app can install the same ref). Once
ADR-0003's `1.0` stability promise lands, PyPI publishing gives frictionless installation to match
a now-stable contract. Coupling the two milestones keeps "stable contract" and "easy to install"
arriving together rather than promising one without the other.

## Consequences

**Positive:**
- No release overhead during the high-churn pre-1.0 phase.
- Consumers still get reproducible installs via pinned refs.

**Negative:**
- VCS installs are slightly less convenient (need repo access / explicit ref) than a PyPI name.
- Deferring the public-vs-private PyPI choice leaves one sub-decision open until 1.0.

**Neutral:**
- Packaging metadata (`pyproject.toml`) is set up from the start regardless of index.

## Alternatives Considered

### Option A: Public PyPI from day one
Frictionless installs immediately, but publishes an unstable, churning contract under a permanent
package name and invites adoption before SC-2 stability exists. Rejected until 1.0.

### Option B: Private index only, indefinitely
Fine for internal reuse, but forecloses public distribution prematurely. Rejected in favor of
deciding public-vs-private at 1.0 when the need is clearer.

## References

- Resolves [open-questions.md](../open-questions.md) Q-5
- Artifact: [SC-2](../artifact.md)
- Related: ADR-0003 (1.0 milestone), ADR-0006 (packaging version matrix)
