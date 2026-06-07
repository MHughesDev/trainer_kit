# Spec: COMP-003 — Training Engine

**Spec ID:** COMP-003
**Type:** Component
**Status:** Draft
**Date:** 2026-06-07
**Author:** Agent

## 1. Overview
1.1 Purpose — Defines the internal training engine that executes the train/val loop, manages device placement, seeds randomness, enforces the leakage guard, and orchestrates metric accumulation and logger callbacks.
1.2 Context — The engine is the core of trainer_kit. It is deliberately time-agnostic (ADR-0002): it operates on plain (input, target) batches from the injected DataSource and knows nothing about windowing or temporal structure. All device decisions flow through a single internal seam (ADR-0007) so multi-GPU can be added later without touching the public API. Correctness is guaranteed by seeding, a leakage guard, and golden-run tests (ADR-0008).
1.3 Related artifacts
   1.3.A ADR-0002: [adr/0002-time-agnostic-engine.md](../adr/0002-time-agnostic-engine.md)
   1.3.B ADR-0007: [adr/0007-device-seam.md](../adr/0007-device-seam.md)
   1.3.C ADR-0008: [adr/0008-correctness-strategy.md](../adr/0008-correctness-strategy.md)
   1.3.D Related specs: COMP-002 (all protocols the engine calls), DATA-001 (TrainConfig/Result the engine reads/produces), FEAT-001 (entry points that delegate to the engine)

## 2. Scope
2.1 Goals
   2.1.A Implement the full train/val loop driven by injected ModelProvider and DataSource.
   2.1.B Implement the internal device/placement seam (_device.py) — single-device only in v1.
   2.1.C Implement library-managed seeding (_seed.py) when TrainConfig.seed is set.
   2.1.D Implement the train/val leakage guard.
   2.1.E Produce a Result and call all Logger hooks in the correct order.
2.2 Non-Goals
   2.2.A Does not implement distributed/multi-GPU training — the device seam reserves this but v1 is single-device only (ADR-0007).
   2.2.B Does not implement checkpointing — the Checkpointer seam is defined in COMP-002 but the engine does not call it in v1 (ADR-0004).
   2.2.C Does not implement mixed-precision (AMP), HPO, early stopping, or learning-rate scheduling — all out of v1 scope.
   2.2.D Does not own windowing or temporal splitting — those are the DataSource's responsibility (ADR-0002).

## 3. Requirements
3.1 Functional requirements
   3.1.A The engine must be invokable only via fit() or evaluate() (FEAT-001); it must not expose any public class or function beyond those two entry points.
   3.1.B Before epoch 1, the engine must call model_provider.build() exactly once and move the resulting model to the resolved device.
   3.1.C Before calling build(), the engine must seed Python, NumPy, and torch if config.seed is not None (ADR-0008).
   3.1.D For each training epoch, the engine must: iterate all batches from data_source.train_loader(), compute the loss, call loss.backward(), and call optimizer.step() with optimizer.zero_grad() before each batch.
   3.1.E For each validation run (governed by config.val_every_n_epochs), the engine must: set the model to eval() mode, iterate all batches from data_source.val_loader() under torch.no_grad(), call metric.update() for each metric, then call metric.compute() and metric.reset() for each metric after the loop. The model must be set back to train() mode after validation.
   3.1.F Logger hooks must be called in the order specified by COMP-002 §4.4. Hook exceptions propagate to the caller unmodified.
   3.1.G All tensor placement must flow through the internal _resolve_device() function in _device.py. No inline .to('cuda') or .cuda() calls are permitted outside _device.py (ADR-0007).
   3.1.H The engine must call optimizer_factory(model) to create the optimizer after model.build(); it must never accept a pre-built optimizer (DATA-001 §3.1.C).
3.2 Non-functional requirements
   3.2.A The engine must not import trainer_kit.timeseries or any domain-specific module (ADR-0002).
   3.2.B Validation must execute under torch.no_grad() to prevent gradient accumulation during evaluation.
   3.2.C All device placement calls must be isolated in _device.py; replacing that module's implementation must be sufficient to add multi-GPU support (ADR-0007).

## 4. Interface / Data

### 4.1 Internal module: _engine.py

The engine is not instantiated — it exposes two module-level functions used by FEAT-001:

```python
def _run_fit(
    model_provider: ModelProvider,
    data_source: DataSource,
    config: TrainConfig,
) -> Result:
    """Execute the full train/val loop. Called by fit() in FEAT-001."""
    ...

def _run_evaluate(
    model_provider: ModelProvider,
    data_source: DataSource,
    config: EvalConfig,
) -> Result:
    """Execute a single validation pass. Called by evaluate() in FEAT-001."""
    ...
```

### 4.2 Internal module: _device.py

```python
import torch

def _resolve_device(device: str | None) -> torch.device:
    """Return the target device. None → CUDA if available, else CPU."""
    if device is None:
        return torch.device("cuda" if torch.cuda.is_available() else "cpu")
    return torch.device(device)

def _to_device(tensor: torch.Tensor, device: torch.device) -> torch.Tensor:
    """Move a tensor to device. All placement in the engine must call this."""
    return tensor.to(device)
```

### 4.3 Internal module: _seed.py

```python
import random
import numpy as np
import torch

def _seed_all(seed: int) -> None:
    """Seed Python, NumPy, and torch for deterministic execution."""
    random.seed(seed)
    np.random.seed(seed)
    torch.manual_seed(seed)
    if torch.cuda.is_available():
        torch.cuda.manual_seed_all(seed)
```

### 4.4 Training loop pseudocode

```
_run_fit(model_provider, data_source, config):
  device = _resolve_device(config.device)
  if config.seed is not None:
      _seed_all(config.seed)
  model = model_provider.build().to(device)    # §3.1.B, §3.1.C
  optimizer = config.optimizer_factory(model)  # §3.1.H
  train_history = []
  val_history = []
  config.logger.on_run_start(config)

  for epoch in 1..config.epochs:
      config.logger.on_epoch_start(epoch)
      model.train()
      train_loss_sum, train_batches = 0.0, 0

      for step, (inputs, targets) in enumerate(data_source.train_loader()):
          inputs = _to_device(inputs, device)
          targets = _to_device(targets, device)
          optimizer.zero_grad()
          preds = model(inputs)
          loss = config.loss_fn(preds, targets)
          loss.backward()
          optimizer.step()
          config.logger.on_batch_end(step, "train", loss.item())
          train_loss_sum += loss.item()
          train_batches += 1

      train_record = EpochRecord(epoch, "train",
                                 train_loss_sum / train_batches, {}, elapsed)
      train_history.append(train_record)
      config.logger.on_epoch_end(epoch, train_record)

      if epoch % config.val_every_n_epochs == 0:
          model.eval()
          val_loss_sum, val_batches = 0.0, 0

          with torch.no_grad():
              for step, (inputs, targets) in enumerate(data_source.val_loader()):
                  inputs = _to_device(inputs, device)
                  targets = _to_device(targets, device)
                  preds = model(inputs)
                  loss = config.loss_fn(preds, targets)
                  config.logger.on_batch_end(step, "val", loss.item())
                  val_loss_sum += loss.item()
                  val_batches += 1
                  for metric in config.metrics:
                      metric.update(preds, targets)

          metric_values = {}
          for metric in config.metrics:
              metric_values[metric.__class__.__name__] = metric.compute().item()
              metric.reset()

          val_record = EpochRecord(epoch, "val",
                                   val_loss_sum / val_batches, metric_values, elapsed)
          val_history.append(val_record)
          config.logger.on_epoch_end(epoch, val_record)
          model.train()

  final_metrics = val_history[-1].metrics if val_history else {}
  result = Result(config.epochs, train_history, val_history, final_metrics)
  config.logger.on_run_end(result)
  return result
```

### 4.5 Leakage guard

Before the first validation batch, the engine asserts that the val_loader's dataset indices do not overlap with the train_loader's dataset indices when both loaders expose a dataset attribute with indices. If either loader does not expose indices, the guard is skipped with a warning. This guard is active during all validation runs, not only the first.

```python
def _assert_no_leakage(train_loader, val_loader) -> None:
    """Raise AssertionError if train and val datasets share any sample indices."""
    ...
```

## 5. Behavior
5.1 Happy path
   5.1.A seed=42 set → Python/NumPy/torch are seeded before build(); two consecutive fit() calls with identical inputs produce identical Results.
   5.1.B seed=None → training executes normally without seeding; results may differ across runs.
   5.1.C val_every_n_epochs=2, epochs=6 → validation runs after epochs 2, 4, 6; val_history has 3 entries.
   5.1.D model is in eval() mode during all validation batches and back in train() mode before the next training epoch.
5.2 Edge cases
   5.2.A config.metrics=[] → EpochRecord.metrics={} for all val records; no metric.update/compute/reset calls.
   5.2.B val_every_n_epochs > epochs → no validation runs, val_history=[], final_metrics={}.
   5.2.C Single-batch dataset → loop executes correctly with one step per epoch.
5.3 Error states
   5.3.A If model_provider.build() raises, the exception propagates before any training begins and on_run_start is never called.
   5.3.B If data_source.train_loader() or val_loader() yields a batch that is not a 2-tuple of Tensors, a TypeError or shape error propagates to the caller unmodified.
   5.3.C Leakage guard fires → AssertionError with a message identifying the overlapping indices.
   5.3.D config.device="cuda" on a machine without CUDA → torch raises a RuntimeError; the engine does not catch it.

## 6. Acceptance Criteria
6.1 Criteria
   6.1.A [ ] Two fit() calls with identical inputs and seed=42 produce Results with identical train_history loss values to float precision (verifies §3.1.C, §5.1.A, ADR-0008) — Verified by: [—]
   6.1.B [ ] model.training is False during all val_loader() iterations and True before any train_loader() iteration in the next epoch (verifies §3.1.E) — Verified by: [—]
   6.1.C [ ] torch.no_grad() is active during all val_loader() iterations; no grad_fn is attached to val loss tensors (verifies §3.2.B) — Verified by: [—]
   6.1.D [ ] metric.reset() is called for each metric immediately after metric.compute() at each validation epoch end (verifies §3.1.E) — Verified by: [—]
   6.1.E [ ] Logger hooks fire in order: on_run_start → on_epoch_start → on_batch_end* → on_epoch_end → on_run_end (verifies §3.1.F, COMP-002 §4.4) — Verified by: [—]
   6.1.F [ ] All tensor placement uses _to_device(); no .cuda() or .to('cuda') calls exist outside _device.py (verifies §3.1.G, §3.2.C) — Verified by: [—]
   6.1.G [ ] Leakage guard raises AssertionError when train and val datasets share indices (verifies §4.5) — Verified by: [—]
   6.1.H [ ] _engine.py has no import of trainer_kit.timeseries (verifies §3.2.A) — Verified by: [—]

## 7. Open Questions & Assumptions
7.1 Open questions — Q-2, Q-7, Q-8 resolved by ADR-0002, ADR-0007, ADR-0008; see [open-questions.md](../open-questions.md)
7.2 Assumptions
   7.2.A Loss is computed per batch and averaged over batches for EpochRecord.loss; this is batch-count-weighted averaging, which is exact when all batches are the same size and an approximation otherwise — Validated: [—]
   7.2.B The leakage guard checks indices only when both loaders expose a .dataset attribute with a valid index set; loaders without this attribute are not checked (guard is best-effort) — Validated: [—]
   7.2.C timing for elapsed_seconds in EpochRecord is wall-clock time measured with time.perf_counter() — Validated: [—]
