# ADR-0003: Start in 0.x, Commit to SemVer at 1.0 with a Deprecation Cycle

**Status:** Accepted
**Date:** 2026-06-06
**Deciders:** Project owner

## Context

Multiple applications depend on `trainer_kit` as a shared library (SC-2). Frequent breaking changes
to the injected interfaces would force every consumer to rewrite its integration, defeating the
"one trusted dependency" goal (FM-3). But the interface set (ADR-0001) is new and will need to
flex as the first real consumers exercise it. We need a versioning promise that is honest about
that early instability without abandoning stability later. This resolves Q-3.

## Decision

Release in the **`0.x`** range while the interfaces stabilize, signalling that minor versions may
break. On reaching **`1.0`**, adopt **strict Semantic Versioning**: breaking interface changes only
on major bumps, with deprecated APIs emitting warnings for at least one minor version before removal.

## Rationale

`0.x` sets accurate expectations during the period when ADR-0001's contract is most likely to move,
avoiding a false stability promise. Committing to SemVer plus a deprecation-warning cycle at `1.0`
gives downstream apps the predictable upgrade path that FM-3 demands, once the contract has earned
that promise. This is the conventional Python-ecosystem expectation, so consumers already understand
the contract implied by the version number.

## Consequences

**Positive:**
- Early iteration on interfaces without pretending stability we can't yet guarantee.
- A clear, conventional stability contract from 1.0 onward (defends FM-3).

**Negative:**
- Consumers adopting during 0.x accept possible breaking minors and must pin versions.
- A deprecation cycle adds release-management overhead post-1.0 (keeping shims around for a cycle).

**Neutral:**
- The 0.x → 1.0 transition becomes a deliberate milestone gated on interface confidence.

## Alternatives Considered

### Option A: Strict SemVer from 1.0 immediately
A stronger trust signal up front, but premature: it would either lock in an unproven contract or
force early major bumps. Rejected given ADR-0001 is freshly minted.

### Option B: Permanent 0.x / "ship from main"
Maximum freedom, but never gives consumers a stability guarantee — directly the FM-3 failure mode.
Rejected.

## References

- Resolves [open-questions.md](../open-questions.md) Q-3
- Artifact: [SC-2, FM-3](../artifact.md)
- Related: ADR-0005 (distribution milestone aligns with 1.0)
