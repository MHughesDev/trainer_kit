# ADR-0007: Reserve an Internal Device/Placement Seam for Future Multi-GPU

**Status:** Accepted
**Date:** 2026-06-06
**Deciders:** Project owner

## Context

v1 runs on a single device (CPU or single GPU). But SC-5 promises that moving to multi-GPU/DDP
later will not break the public pipeline API that consuming apps depend on. That only holds if the
engine isolates device/placement decisions now, rather than scattering `.to('cuda')` and loop
assumptions throughout — otherwise "add multi-GPU" becomes a rewrite (FM-5). This resolves Q-7.

## Decision

Route all device and tensor-placement decisions through a single **internal** component (a device/
placement seam) owned by the engine. v1 implements only single-device behavior behind it. **No
public device-strategy API is exposed** in v1; the seam is internal and the public `fit()` surface
(ADR-0011) stays device-agnostic.

## Rationale

Concentrating placement in one internal component means a future DDP/multi-device strategy is a
swap behind a stable boundary, satisfying SC-5 without breaking consumers and directly defusing
FM-5. Keeping the seam *internal* (not a public strategy API) avoids over-building speculative
extension points in v1 — the public contract simply never mentions devices, so it can't be broken
by how devices are later handled. This mirrors the seam-reservation discipline of ADR-0004.

## Consequences

**Positive:**
- Multi-GPU becomes an internal extension, not a public API break (SC-5).
- The public `fit()` API stays clean and device-agnostic.

**Negative:**
- A small upfront discipline cost: device handling must be funneled through one component from the start.
- Until multi-GPU lands, the seam carries only a single-device implementation (minor indirection).

**Neutral:**
- The shape of the eventual strategy API (accelerators, launchers) remains undecided and out of v1 scope.

## Alternatives Considered

### Option A: Full device-strategy abstraction now (Lightning-style accelerators)
Most future-proof, but speculative scope for v1 and risks designing the abstraction before the
multi-GPU requirements are concrete. Rejected as premature.

### Option B: No seam — handle devices inline in the loop
Simplest now, but bakes single-device assumptions throughout, making multi-GPU a rewrite and
breaking SC-5 (the FM-5 failure). Rejected.

## References

- Resolves [open-questions.md](../open-questions.md) Q-7
- Artifact: [SC-5, FM-5](../artifact.md)
- Related: ADR-0004 (seam reservation), ADR-0011 (device-agnostic entry point)
