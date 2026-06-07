# Spec: DATA-001 — Config and Result Types

**Spec ID:** DATA-001
**Type:** Data
**Status:** Draft
**Date:** 2026-06-07
**Author:** Agent

## 1. Overview
1.1 Purpose — Defines the TrainConfig, EvalConfig, EpochRecord, and Result dataclasses that form the configuration and output contract of fit() and evaluate().
1.2 Context — ADR-0011 decided that all training configuration is passed as a single typed dataclass (TrainConfig) rather than loose kwargs. A matching EvalConfig serves evaluate(). The Result returned by both entry points gives the caller their outcome without the library logging or persisting anything. These types are the glue between the consumer, the engine, and the logger.
1.3 Related artifacts
   1.3.A ADR-0011: [adr/0011-entry-point.md](../adr/0011-entry-point.md)
   1.3.B ADR-0001: [adr/0001-inject-dependencies-via-protocols.md](../adr/0001-inject-dependencies-via-protocols.md)
   1.3.C ADR-0008: [adr/0008-correctness-strategy.md](../adr/0008-correctness-strategy.md)
   1.3.D Related specs: COMP-002 (Metric, Logger, Checkpointer referenced as field types), FEAT-001 (consumes these types), COMP-003 (engine reads these types)

## 2. Scope
2.1 Goals
   2.1.A Define every field of TrainConfig with its type, default, and validation rule.
   2.1.B Define EvalConfig as the minimal configuration for evaluate().
   2.1.C Define EpochRecord as the structured payload passed to Logger.on_epoch_end().
   2.1.D Define Result as the return type of fit() and evaluate().
2.2 Non-Goals
   2.2.A Does not define how the engine uses these fields — that is COMP-003.
   2.2.B Does not define serialization or persistence of these types.
   2.2.C Does not define the Metric, Logger, or Checkpointer protocols — those are COMP-002.

## 3. Requirements
3.1 Functional requirements
   3.1.A TrainConfig must be a frozen dataclass (immutable after construction) to prevent mid-run mutation.
   3.1.B TrainConfig must validate all fields at __post_init__ time and raise ValueError for any invalid combination (e.g. epochs < 1, val_every_n_epochs < 1).
   3.1.C TrainConfig must accept optimizer_factory as a callable (not a pre-built optimizer) so the engine can construct the optimizer after model.build() — preserving the correct parameter binding.
   3.1.D TrainConfig.device = None must signal auto-detection (prefer CUDA if available, else CPU).
   3.1.E TrainConfig.seed = None must signal non-deterministic execution; any integer value seeds Python/NumPy/torch deterministically (ADR-0008).
   3.1.F TrainConfig.logger defaults to StdoutLogger() when not supplied.
   3.1.G TrainConfig.checkpointer is reserved and ignored by the v1 engine (ADR-0004).
   3.1.H EvalConfig must share the same device, metrics, and logger fields as TrainConfig; it does not include epochs, optimizer_factory, loss_fn, or seed.
   3.1.I EpochRecord must carry: epoch index, split label, mean loss, per-metric scalar values, and elapsed time in seconds.
   3.1.J Result must carry: epochs_completed, train_history (one EpochRecord per trained epoch), val_history (one EpochRecord per validation run), and final_metrics (the last computed metric values keyed by metric name).
3.2 Non-functional requirements
   3.2.A These dataclasses must import only from the Python standard library and from trainer_kit._protocols (for type annotations). They must not import _engine.py.
   3.2.B Metric name keys in EpochRecord.metrics and Result.final_metrics are the string returned by str(metric) or metric.__class__.__name__ — consistent and predictable.

## 4. Interface / Data

### 4.1 TrainConfig

```python
from __future__ import annotations
from dataclasses import dataclass, field
from typing import Callable, Sequence, TYPE_CHECKING
import torch.optim as optim

if TYPE_CHECKING:
    from trainer_kit._protocols import Metric, Logger, Checkpointer

@dataclass(frozen=True)
class TrainConfig:
    # Required fields (no default)
    epochs: int
    optimizer_factory: Callable[[torch.nn.Module], optim.Optimizer]
    loss_fn: Callable[[torch.Tensor, torch.Tensor], torch.Tensor]

    # Optional fields with defaults
    metrics: Sequence[Metric] = field(default_factory=list)
    logger: Logger = field(default_factory=lambda: StdoutLogger())
    device: str | None = None        # None → auto-detect
    seed: int | None = None          # None → non-deterministic
    val_every_n_epochs: int = 1      # run validation every N epochs
    checkpointer: Checkpointer | None = None  # reserved, not called in v1

    def __post_init__(self) -> None:
        if self.epochs < 1:
            raise ValueError(f"epochs must be >= 1, got {self.epochs}")
        if self.val_every_n_epochs < 1:
            raise ValueError(f"val_every_n_epochs must be >= 1, got {self.val_every_n_epochs}")
```

### 4.2 EvalConfig

```python
@dataclass(frozen=True)
class EvalConfig:
    metrics: Sequence[Metric] = field(default_factory=list)
    logger: Logger = field(default_factory=lambda: StdoutLogger())
    device: str | None = None
```

### 4.3 EpochRecord

```python
@dataclass(frozen=True)
class EpochRecord:
    epoch: int            # 1-indexed
    split: str            # "train" or "val"
    loss: float           # mean loss over all batches in this epoch/split
    metrics: dict[str, float]   # {metric_name: scalar_value}
    elapsed_seconds: float
```

### 4.4 Result

```python
@dataclass(frozen=True)
class Result:
    epochs_completed: int
    train_history: list[EpochRecord]   # one entry per epoch
    val_history: list[EpochRecord]     # one entry per validation run
    final_metrics: dict[str, float]    # last val metrics; empty if no metrics configured
```

## 5. Behavior
5.1 Happy path
   5.1.A Constructing TrainConfig(epochs=10, optimizer_factory=..., loss_fn=...) succeeds and all optional fields take their defaults.
   5.1.B After fit() completes, result.train_history has exactly config.epochs entries and result.val_history has exactly ceil(config.epochs / config.val_every_n_epochs) entries.
   5.1.C result.final_metrics matches the metric values in result.val_history[-1].metrics.
5.2 Edge cases
   5.2.A TrainConfig with metrics=[] results in EpochRecord.metrics = {} and Result.final_metrics = {}.
   5.2.B TrainConfig with val_every_n_epochs > epochs results in val_history = [] (no validation runs).
   5.2.C TrainConfig with seed=0 is valid (0 is a legitimate seed; None means no seeding).
5.3 Error states
   5.3.A TrainConfig(epochs=0, ...) raises ValueError at construction time (not at fit() call time).
   5.3.B TrainConfig(val_every_n_epochs=0, ...) raises ValueError at construction time.
   5.3.C Attempting to mutate a field on a frozen TrainConfig instance raises FrozenInstanceError (dataclass behavior).

## 6. Acceptance Criteria
6.1 Criteria
   6.1.A [ ] TrainConfig(epochs=0, ...) raises ValueError before any training begins (verifies §3.1.B, §5.3.A) — Verified by: [—]
   6.1.B [ ] TrainConfig constructed without logger uses StdoutLogger as default (verifies §3.1.F) — Verified by: [—]
   6.1.C [ ] result.train_history length equals config.epochs after a successful fit() (verifies §3.1.J, §5.1.B) — Verified by: [—]
   6.1.D [ ] result.val_history length equals ceil(epochs / val_every_n_epochs) after a successful fit() (verifies §5.1.B) — Verified by: [—]
   6.1.E [ ] EpochRecord.metrics keys match metric.__class__.__name__ for each configured Metric (verifies §3.2.B) — Verified by: [—]
   6.1.F [ ] Importing _config.py does not import _engine.py (verifies §3.2.A) — Verified by: [—]

## 7. Open Questions & Assumptions
7.1 Open questions — Q-11 resolved by ADR-0011; see [open-questions.md](../open-questions.md)
7.2 Assumptions
   7.2.A optimizer_factory is a Callable that accepts an nn.Module and returns an Optimizer; this is the idiomatic PyTorch pattern for deferred optimizer construction — Validated: [—]
   7.2.B Metric names are derived from __class__.__name__; if two metrics of the same class are used, callers must wrap them in a thin subclass with a distinct name — Validated: [—]
   7.2.C frozen=True on TrainConfig is safe because Sequence[Metric] and Logger are reference types; the frozen constraint prevents field reassignment but not mutation of the contained objects — Validated: [—]
