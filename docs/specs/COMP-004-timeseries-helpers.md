# Spec: COMP-004 — trainer_kit.timeseries Helpers

**Spec ID:** COMP-004
**Type:** Component
**Status:** Draft
**Date:** 2026-06-07
**Author:** Agent

## 1. Overview
1.1 Purpose — Defines the optional trainer_kit.timeseries sub-package: a SlidingWindowDataset, a temporal_split() function, and three default Metric implementations (MAE, MSE, RMSE) that time-series applications can use behind their DataSource.
1.2 Context — The engine is time-agnostic (ADR-0002). The timeseries module is an opt-in helper layer that apps can use to build their DataSource without re-implementing sliding windows and temporally-correct train/val splits. The engine never imports this module; its absence does not affect any core behavior.
1.3 Related artifacts
   1.3.A ADR-0002: [adr/0002-time-agnostic-engine.md](../adr/0002-time-agnostic-engine.md)
   1.3.B ADR-0009: [adr/0009-stateful-metrics.md](../adr/0009-stateful-metrics.md)
   1.3.C Related specs: COMP-001 (module location under timeseries/), COMP-002 (Metric protocol that MAE/MSE/RMSE implement), COMP-003 (leakage guard that temporal_split is designed to satisfy)

## 2. Scope
2.1 Goals
   2.1.A Implement SlidingWindowDataset to produce (input_window, target_window) pairs from a 1-D or multi-variate time series Tensor.
   2.1.B Implement temporal_split() to partition a SlidingWindowDataset into non-overlapping train and val subsets without shuffling.
   2.1.C Implement MAE, MSE, and RMSE as stateful Metric accumulators compatible with the Metric protocol (COMP-002 §4.3).
2.2 Non-Goals
   2.2.A Does not handle data loading, file I/O, or feature engineering — those are the consuming app's responsibility.
   2.2.B Does not implement multi-step forecast horizon prediction beyond what SlidingWindowDataset naturally produces.
   2.2.C Does not affect the engine in any way — the engine has no dependency on this module (ADR-0002).
   2.2.D Does not implement dataset normalization or scaling.

## 3. Requirements
3.1 Functional requirements
   3.1.A SlidingWindowDataset must accept a Tensor of shape (T, F) or (T,) where T is time steps and F is features, plus input_len, target_len, and stride parameters.
   3.1.B SlidingWindowDataset must produce samples as (input_window: Tensor, target_window: Tensor) pairs, each of shape (input_len, F) and (target_len, F) respectively (or (input_len,) and (target_len,) for univariate).
   3.1.C SlidingWindowDataset.__len__ must return exactly floor((T - input_len - target_len) / stride) + 1.
   3.1.D temporal_split() must accept a SlidingWindowDataset and a val_fraction float in (0, 1), and return a (train_subset, val_subset) tuple where train samples precede val samples in time — no shuffling.
   3.1.E temporal_split() must produce non-overlapping subsets: the last train sample's target window must end before the first val sample's input window begins.
   3.1.F MAE must accumulate sum of absolute errors per element and compute the mean over all elements and all update() calls.
   3.1.G MSE must accumulate sum of squared errors and compute the mean.
   3.1.H RMSE must compute sqrt(MSE.compute()) — it may delegate to MSE internally.
   3.1.I All three metrics must satisfy isinstance(metric, Metric) (COMP-002 §3.1.F).
   3.1.J All three metrics must tolerate reset() called before any update() call.
3.2 Non-functional requirements
   3.2.A trainer_kit.timeseries must not import trainer_kit._engine or any internal engine module.
   3.2.B SlidingWindowDataset must subclass torch.utils.data.Dataset to be compatible with DataLoader.

## 4. Interface / Data

### 4.1 SlidingWindowDataset

```python
from torch import Tensor
from torch.utils.data import Dataset

class SlidingWindowDataset(Dataset):
    def __init__(
        self,
        series: Tensor,         # shape (T,) or (T, F)
        input_len: int,         # length of the input window
        target_len: int,        # length of the target window
        stride: int = 1,        # step between consecutive windows
    ) -> None: ...

    def __len__(self) -> int:
        """Return floor((T - input_len - target_len) / stride) + 1."""
        ...

    def __getitem__(self, idx: int) -> tuple[Tensor, Tensor]:
        """Return (input_window, target_window) for sample at idx."""
        ...
```

### 4.2 temporal_split

```python
from torch.utils.data import Subset

def temporal_split(
    dataset: SlidingWindowDataset,
    val_fraction: float,        # fraction of samples to use for validation (0 < val_fraction < 1)
) -> tuple[Subset, Subset]:
    """Split dataset chronologically into (train_subset, val_subset).
    Train samples occupy the first (1 - val_fraction) of the dataset;
    val samples occupy the remaining val_fraction.
    No samples are shared between subsets.
    """
    ...
```

### 4.3 Metric implementations

```python
import torch
from torch import Tensor

class MAE:
    """Mean Absolute Error — satisfies the Metric protocol."""
    def __init__(self) -> None:
        self._sum: float = 0.0
        self._count: int = 0

    def update(self, preds: Tensor, targets: Tensor) -> None:
        self._sum += (preds - targets).abs().sum().item()
        self._count += targets.numel()

    def compute(self) -> Tensor:
        if self._count == 0:
            raise RuntimeError("MAE.compute() called before any update()")
        return torch.tensor(self._sum / self._count)

    def reset(self) -> None:
        self._sum = 0.0
        self._count = 0


class MSE:
    """Mean Squared Error — satisfies the Metric protocol."""
    def __init__(self) -> None:
        self._sum: float = 0.0
        self._count: int = 0

    def update(self, preds: Tensor, targets: Tensor) -> None:
        self._sum += ((preds - targets) ** 2).sum().item()
        self._count += targets.numel()

    def compute(self) -> Tensor:
        if self._count == 0:
            raise RuntimeError("MSE.compute() called before any update()")
        return torch.tensor(self._sum / self._count)

    def reset(self) -> None:
        self._sum = 0.0
        self._count = 0


class RMSE:
    """Root Mean Squared Error — satisfies the Metric protocol."""
    def __init__(self) -> None:
        self._mse = MSE()

    def update(self, preds: Tensor, targets: Tensor) -> None:
        self._mse.update(preds, targets)

    def compute(self) -> Tensor:
        return self._mse.compute().sqrt()

    def reset(self) -> None:
        self._mse.reset()
```

### 4.4 trainer_kit/timeseries/__init__.py exports

```python
from trainer_kit.timeseries._dataset import SlidingWindowDataset
from trainer_kit.timeseries._split import temporal_split
from trainer_kit.timeseries._metrics import MAE, MSE, RMSE

__all__ = ["SlidingWindowDataset", "temporal_split", "MAE", "MSE", "RMSE"]
```

## 5. Behavior
5.1 Happy path
   5.1.A SlidingWindowDataset with T=100, input_len=10, target_len=1, stride=1 produces 89 samples.
   5.1.B temporal_split with val_fraction=0.2 on a 100-sample dataset produces 80 train and 20 val samples, with train indices [0..79] and val indices [80..99].
   5.1.C MAE.update() called three times then compute() returns the mean of all accumulated errors, not just the last batch.
5.2 Edge cases
   5.2.A SlidingWindowDataset with stride > 1 produces exactly floor((T - input_len - target_len) / stride) + 1 samples, rounding down.
   5.2.B temporal_split with val_fraction=0.2 on a dataset whose size is not cleanly divisible by 5 uses floor() for train count and ceil() for val count (rounds in favor of training data).
   5.2.C RMSE.reset() followed immediately by RMSE.update() and RMSE.compute() returns a valid scalar.
5.3 Error states
   5.3.A SlidingWindowDataset with input_len + target_len > T raises ValueError at construction time.
   5.3.B SlidingWindowDataset with stride < 1 raises ValueError at construction time.
   5.3.C temporal_split with val_fraction <= 0 or >= 1 raises ValueError.
   5.3.D MAE.compute() or MSE.compute() called before any update() raises RuntimeError.

## 6. Acceptance Criteria
6.1 Criteria
   6.1.A [ ] SlidingWindowDataset(series, input_len=10, target_len=1, stride=1) where series has T=100 returns len() == 89 (verifies §3.1.C) — Verified by: [—]
   6.1.B [ ] temporal_split produces subsets with no overlapping indices (verifies §3.1.E, ADR-0008 leakage guard) — Verified by: [—]
   6.1.C [ ] temporal_split produces chronologically ordered splits: max(train_indices) < min(val_indices) (verifies §3.1.D) — Verified by: [—]
   6.1.D [ ] isinstance(MAE(), Metric) and isinstance(MSE(), Metric) and isinstance(RMSE(), Metric) all return True (verifies §3.1.I, COMP-002 §3.1.F) — Verified by: [—]
   6.1.E [ ] MAE.update(preds, targets) called twice then MAE.compute() returns the element-wise mean of all absolute errors across both calls (verifies §3.1.F) — Verified by: [—]
   6.1.F [ ] `from trainer_kit.timeseries import SlidingWindowDataset, temporal_split, MAE, MSE, RMSE` succeeds without importing trainer_kit core (verifies §3.2.A, COMP-001 §3.1.B) — Verified by: [—]
   6.1.G [ ] SlidingWindowDataset(series, input_len=95, target_len=10, stride=1) where series has T=100 raises ValueError (verifies §5.3.A) — Verified by: [—]

## 7. Open Questions & Assumptions
7.1 Open questions — Q-2 resolved by ADR-0002; see [open-questions.md](../open-questions.md)
7.2 Assumptions
   7.2.A The series Tensor is already in memory; SlidingWindowDataset does not handle lazy loading or memory-mapping of large series — Validated: [—]
   7.2.B Multivariate support (T, F) is provided by making all indexing along the first dimension only; F features are preserved in the output windows unchanged — Validated: [—]
   7.2.C RMSE delegates to MSE internally to avoid code duplication; this is an implementation choice not visible through the protocol — Validated: [—]
