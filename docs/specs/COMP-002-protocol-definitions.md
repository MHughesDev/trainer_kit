# Spec: COMP-002 — Protocol Definitions

**Spec ID:** COMP-002
**Type:** Component
**Status:** Draft
**Date:** 2026-06-07
**Author:** Agent

## 1. Overview
1.1 Purpose — Defines the exact Python signatures and behavioral contracts of all five typing.Protocol interfaces that trainer_kit exposes: the two required contracts (ModelProvider, DataSource) and the three injectable optional contracts (Metric, Logger, Checkpointer).
1.2 Context — All integration between the library and consuming applications flows through these protocols (ADR-0001). The library never inherits from consuming code; structural conformance (duck typing + static type-checking) is the mechanism. This spec is the authoritative source for every protocol's method signatures and invariants.
1.3 Related artifacts
   1.3.A ADR-0001: [adr/0001-inject-dependencies-via-protocols.md](../adr/0001-inject-dependencies-via-protocols.md)
   1.3.B ADR-0009: [adr/0009-stateful-metrics.md](../adr/0009-stateful-metrics.md)
   1.3.C ADR-0010: [adr/0010-logger-protocol.md](../adr/0010-logger-protocol.md)
   1.3.D ADR-0004: [adr/0004-checkpoint-seam.md](../adr/0004-checkpoint-seam.md)
   1.3.E Related specs: DATA-001 (TrainConfig/Result used in Logger hooks), COMP-001 (module location)

## 2. Scope
2.1 Goals
   2.1.A Define the exact method signatures, parameter types, and return types for all five protocols.
   2.1.B Define the behavioral invariants (pre/post-conditions and ordering guarantees) the engine upholds when calling each protocol method.
   2.1.C Define the StdoutLogger default implementation shipped by the library.
2.2 Non-Goals
   2.2.A Does not specify the engine's internal loop logic — that is COMP-003.
   2.2.B Does not specify the timeseries-specific Metric defaults (MAE, MSE, RMSE) — those are COMP-004.
   2.2.C Does not specify how consumers implement these protocols — that is the consumer's concern.
   2.2.D Checkpointer is defined here but its runtime behavior is reserved for a future version (ADR-0004); the v1 engine ignores it.

## 3. Requirements
3.1 Functional requirements
   3.1.A ModelProvider must declare `build() -> torch.nn.Module`. The engine calls it exactly once per fit()/evaluate() call, after seeding and before the first epoch.
   3.1.B DataSource must declare `train_loader() -> torch.utils.data.DataLoader` and `val_loader() -> torch.utils.data.DataLoader`. The engine calls train_loader() once per epoch and val_loader() at the end of each validation step. Both must yield batches of type `tuple[Tensor, Tensor]` (input, target).
   3.1.C Metric must declare `update(preds: Tensor, targets: Tensor) -> None`, `compute() -> Tensor`, and `reset() -> None`. The engine calls update() per batch during validation, compute() once at epoch end, and reset() immediately after compute().
   3.1.D Logger must declare five lifecycle hooks: `on_run_start`, `on_epoch_start`, `on_batch_end`, `on_epoch_end`, `on_run_end`. The engine calls them in that order; no hook may be called out of sequence.
   3.1.E Checkpointer must declare `save(state: dict[str, Any], path: str | Path) -> None` and `load(path: str | Path) -> dict[str, Any]`. The seam is defined but the engine does not call it in v1.
   3.1.F All five protocols must be decorated with @runtime_checkable so isinstance() checks are valid.
   3.1.G The library must ship a StdoutLogger that satisfies the Logger protocol and writes human-readable epoch summaries to sys.stdout. StdoutLogger is the default when TrainConfig.logger is not specified.
3.2 Non-functional requirements
   3.2.A Protocol definitions must not import anything outside of the Python standard library and torch — no circular imports with the engine.
   3.2.B Adding a new optional method to a protocol in a future version must follow the ADR-0003 deprecation cycle (one-minor deprecation period at 1.0+).

## 4. Interface / Data

### 4.1 ModelProvider

```python
from typing import Protocol, runtime_checkable
import torch.nn as nn

@runtime_checkable
class ModelProvider(Protocol):
    def build(self) -> nn.Module:
        """Return a freshly constructed, un-trained nn.Module.
        Called once per fit()/evaluate() invocation after seeding.
        The caller owns the returned module's weights."""
        ...
```

### 4.2 DataSource

```python
from typing import Protocol, runtime_checkable
from torch.utils.data import DataLoader

@runtime_checkable
class DataSource(Protocol):
    def train_loader(self) -> DataLoader:
        """Return a DataLoader yielding (input: Tensor, target: Tensor) batches.
        Windowing and temporal split are the caller's responsibility (ADR-0002)."""
        ...

    def val_loader(self) -> DataLoader:
        """Return a DataLoader yielding (input: Tensor, target: Tensor) batches.
        Must not overlap with train_loader() batches (leakage guard, ADR-0008)."""
        ...
```

### 4.3 Metric

```python
from typing import Protocol, runtime_checkable
from torch import Tensor

@runtime_checkable
class Metric(Protocol):
    def update(self, preds: Tensor, targets: Tensor) -> None:
        """Accumulate state for this batch. Called once per validation batch."""
        ...

    def compute(self) -> Tensor:
        """Return the aggregated scalar metric for the current epoch.
        Does not reset state — the engine calls reset() after compute()."""
        ...

    def reset(self) -> None:
        """Clear accumulated state. Called by the engine after compute()."""
        ...
```

**Engine invariant:** the engine always calls reset() immediately after compute() at epoch end, before the next epoch begins. A Metric must tolerate being reset() before any update() call (i.e., freshly constructed state is valid).

### 4.4 Logger

```python
from typing import Any, Protocol, runtime_checkable

@runtime_checkable
class Logger(Protocol):
    def on_run_start(self, config: "TrainConfig") -> None:
        """Called once before epoch 1. config carries the full TrainConfig."""
        ...

    def on_epoch_start(self, epoch: int) -> None:
        """Called at the start of each epoch (1-indexed)."""
        ...

    def on_batch_end(self, step: int, split: str, loss: float) -> None:
        """Called after every training and validation batch.
        split is 'train' or 'val'. step is the global batch index."""
        ...

    def on_epoch_end(self, epoch: int, record: "EpochRecord") -> None:
        """Called at the end of each epoch with the full epoch summary."""
        ...

    def on_run_end(self, result: "Result") -> None:
        """Called once after the final epoch with the complete Result."""
        ...
```

**Hook call order guarantee (engine):** on_run_start → [on_epoch_start → [on_batch_end*] → on_epoch_end]* → on_run_end. No hook is ever called out of this order.

### 4.5 Checkpointer (v1 seam — not called by the engine in v1)

```python
from pathlib import Path
from typing import Any, Protocol, runtime_checkable

@runtime_checkable
class Checkpointer(Protocol):
    def save(self, state: dict[str, Any], path: str | Path) -> None:
        """Persist state to the caller-supplied path."""
        ...

    def load(self, path: str | Path) -> dict[str, Any]:
        """Restore and return state from the caller-supplied path."""
        ...
```

### 4.6 StdoutLogger (default implementation)

```python
import sys
from trainer_kit._config import EpochRecord, Result, TrainConfig

class StdoutLogger:
    """Default Logger. Writes human-readable summaries to sys.stdout."""

    def on_run_start(self, config: TrainConfig) -> None:
        print(f"[trainer_kit] Starting run: epochs={config.epochs}", file=sys.stdout)

    def on_epoch_start(self, epoch: int) -> None:
        pass  # silent

    def on_batch_end(self, step: int, split: str, loss: float) -> None:
        pass  # silent — epoch-level summary is sufficient

    def on_epoch_end(self, epoch: int, record: EpochRecord) -> None:
        parts = [f"epoch={epoch}", f"split={record.split}", f"loss={record.loss:.4f}"]
        parts += [f"{k}={v:.4f}" for k, v in record.metrics.items()]
        print(f"[trainer_kit] {' | '.join(parts)}", file=sys.stdout)

    def on_run_end(self, result: Result) -> None:
        print(f"[trainer_kit] Run complete: epochs={result.epochs_completed}", file=sys.stdout)
```

## 5. Behavior
5.1 Happy path
   5.1.A A consumer class with the required methods satisfies ModelProvider and DataSource without inheriting from any trainer_kit type.
   5.1.B isinstance(obj, ModelProvider) returns True for any object with a callable build() method.
   5.1.C StdoutLogger.on_epoch_end prints one line per epoch to stdout with loss and metric values.
5.2 Edge cases
   5.2.A A Metric whose reset() is called before any update() must not raise — it must return to the same state as freshly constructed.
   5.2.B Logger hooks must not be called if fit() raises before training begins (e.g. invalid config).
   5.2.C on_batch_end is called for both train and val batches; split distinguishes them.
5.3 Error states
   5.3.A If an object passed as model_provider does not satisfy ModelProvider at runtime, the engine raises TypeError before any training begins (COMP-003 §5.3.B).
   5.3.B If a Logger hook raises, the exception propagates to the caller unmodified — the engine does not catch hook exceptions.

## 6. Acceptance Criteria
6.1 Criteria
   6.1.A [ ] A class with only `def build(self) -> nn.Module` passes `isinstance(obj, ModelProvider)` (verifies §3.1.A, §3.1.F) — Verified by: [—]
   6.1.B [ ] A class with only `train_loader` and `val_loader` methods passes `isinstance(obj, DataSource)` (verifies §3.1.B, §3.1.F) — Verified by: [—]
   6.1.C [ ] The engine calls `Metric.reset()` on every metric after every call to `Metric.compute()` and before the next epoch begins (verifies §3.1.C and §4.3 invariant) — Verified by: [—]
   6.1.D [ ] Logger hooks fire in the guaranteed order: on_run_start → on_epoch_start → on_batch_end* → on_epoch_end → on_run_end (verifies §3.1.D, §4.4 order guarantee) — Verified by: [—]
   6.1.E [ ] StdoutLogger satisfies isinstance(StdoutLogger(), Logger) (verifies §3.1.G, §3.1.F) — Verified by: [—]
   6.1.F [ ] Importing _protocols.py does not import _engine.py, _config.py, or any trainer_kit non-stdlib module (verifies §3.2.A) — Verified by: [—]

## 7. Open Questions & Assumptions
7.1 Open questions — all Q-1, Q-9, Q-10 resolved by ADR-0001, ADR-0009, ADR-0010; see [open-questions.md](../open-questions.md)
7.2 Assumptions
   7.2.A DataLoader batches are always 2-tuples of (input Tensor, target Tensor); the engine does not support non-standard batch shapes in v1 — Validated: [—]
   7.2.B Python 3.10's typing.Protocol with @runtime_checkable supports isinstance() checks for single-method protocols without additional dependencies — Validated: [—]
   7.2.C Checkpointer will not be called by the v1 engine; it is included for type-checking completeness and future readiness (ADR-0004) — Validated: ADR-0004 accepted
