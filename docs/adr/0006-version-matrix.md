# ADR-0006: Support Python ≥ 3.10 and a Tested PyTorch Range

**Status:** Accepted
**Date:** 2026-06-06
**Deciders:** Project owner

## Context

As a library consumed by many applications (SC-2), `trainer_kit` must declare which Python and
PyTorch versions it supports. The forces: adoption breadth (more apps can adopt a range) vs.
testing/guarantee cost (a range is more to verify), and access to modern language features. This
resolves Q-6.

## Decision

Require **Python ≥ 3.10** and support **PyTorch as a tested range** (target floor `torch >= 2.1`,
`< 3`) rather than pinning a single version. The supported matrix is enforced by CI; versions are
only claimed once they pass tests.

## Rationale

A PyTorch *range* maximizes how many existing applications can adopt the library without forcing a
torch upgrade — directly serving the multi-consumer goal (SC-2). Python 3.10 as the floor gives
modern typing ergonomics (useful for the `Protocol`-based contract in ADR-0001, e.g. better
structural-typing support and cleaner union syntax) without excluding many users. Pinning to one
torch version would be simpler to test but would needlessly fragment adoption across apps on
different torch releases.

## Consequences

**Positive:**
- Broad adoption: apps on any supported torch in range can use the library unchanged.
- Modern typing features available for the contract surface.

**Negative:**
- A version *range* means a CI matrix to maintain (multiple torch versions tested).
- Supporting older torch in the range may occasionally constrain use of the newest torch APIs.

**Neutral:**
- The exact upper bound and tested points evolve as new torch releases are validated in CI.

## Alternatives Considered

### Option A: Pin a single recent PyTorch (e.g. 2.x exact)
Simplest to test and guarantee, but forces every consuming app onto that exact torch — fragmenting
adoption (works against SC-2). Rejected.

### Option B: Support very old Python (≤ 3.8)
Maximizes reach but forfeits modern typing features that make the `Protocol` contract cleaner and
safer. Rejected; 3.10 is a reasonable, widely-available floor.

## References

- Resolves [open-questions.md](../open-questions.md) Q-6
- Artifact: [SC-2](../artifact.md)
- Related: ADR-0001 (typing features), ADR-0005 (packaging)
