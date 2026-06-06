# ADR-0001: Inject Dependencies via Typing Protocols

**Status:** Accepted
**Date:** 2026-06-06
**Deciders:** Project owner

## Context

`trainer_kit` must run train/validation pipelines without owning any model or data (artifact
SC-1, SC-3). Consuming applications therefore have to *hand in* their model and data, and the
shape of that hand-off is the library's entire integration contract. If the contract is too
generic, apps write as much glue as a hand-rolled loop and the reuse value collapses
(artifact FM-1); if it is too prescriptive, it will not fit the second application unchanged
(SC-2). We also had to choose between nominal contracts (`abc.ABC`, explicit inheritance) and
structural ones (`typing.Protocol`, duck-typed conformance). This resolves open question Q-1.

## Decision

Define the integration contract as a **small set of `typing.Protocol` interfaces**. The required
set is kept to two — `ModelProvider` (`build() -> torch.nn.Module`) and `DataSource`
(`train_loader()`, `val_loader()`). Everything else (optimizer, scheduler, loss, metrics, logger)
is passed in as plain injected objects or factories rather than as additional mandatory protocols.

## Rationale

A minimal, structural contract is the strongest available defense against FM-1: apps implement
two small protocols and inject the rest as ordinary objects they already construct. `Protocol`
(structural) over `ABC` (nominal) means an app's existing class conforms without inheriting from
`trainer_kit` at all, minimizing coupling and keeping the library a true peripheral dependency.
Keeping the *required* surface to two protocols, with the remainder injected as objects, holds the
contract small enough to fit many apps unchanged (SC-2).

## Consequences

**Positive:**
- Lowest possible coupling — consumers don't inherit library base classes.
- Smallest contract surface to learn and to keep stable, directly serving SC-1/SC-2.
- Easy to test with lightweight fakes that merely satisfy the protocol.

**Negative:**
- Structural typing surfaces conformance errors later (at call time) than nominal inheritance.
- Less discoverable than ABCs — consumers rely on documentation and type-checkers to learn the shape.

**Neutral:**
- Conformance enforcement leans on static type-checking (mypy/pyright) rather than runtime checks.

## Alternatives Considered

### Option A: `abc.ABC` base classes for the same set
Enforced conformance and clearer error messages, but forces consumers to inherit from
`trainer_kit` types, raising coupling and making the library harder to slot into an existing class
hierarchy. Rejected to keep the dependency peripheral.

### Option B: Many fine-grained protocols (separate optimizer/scheduler/loss/metric/logger contracts)
More explicit and composable, but multiplies what every app must implement, increasing the glue
burden the contract is meant to remove (FM-1). Rejected in favor of injecting those as objects.

## References

- Resolves [open-questions.md](../open-questions.md) Q-1
- Artifact: [SC-1, SC-2, FM-1](../artifact.md)
- Related: ADR-0009 (Metric), ADR-0010 (Logger), ADR-0011 (entry point)
