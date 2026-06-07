# Spec: COMP-005 — Test Suite

**Spec ID:** COMP-005
**Type:** Component
**Status:** Draft
**Date:** 2026-06-07
**Author:** Agent

## 1. Overview
1.1 Purpose — Defines the three mandatory test categories that guarantee trainer_kit's correctness contract: golden-run regression tests, determinism tests, and a validation-leakage guard test. Together they convert FM-4 (silent loop-math corruption) from a production incident into a CI failure.
1.2 Context — Every consuming app shares the one training loop. A subtle bug — wrong metric averaging, broken seeding, data leakage — corrupts results for all consumers at once (artifact FM-4). ADR-0008 mandates these three measures. This spec defines exactly what each test must cover, what "pass" looks like, and how to update expected values when a legitimate change alters golden-run outputs.
1.3 Related artifacts
   1.3.A ADR-0008: [adr/0008-correctness-strategy.md](../adr/0008-correctness-strategy.md)
   1.3.B ADR-0009: [adr/0009-stateful-metrics.md](../adr/0009-stateful-metrics.md)
   1.3.C Related specs: COMP-003 (engine behavior these tests verify), COMP-004 (timeseries helpers tested by leakage guard), DATA-001 (Result type inspected by tests), FEAT-001 (entry points invoked by tests)

## 2. Scope
2.1 Goals
   2.1.A Define the golden-run test: a deterministic fit() call whose outputs are pinned to expected values.
   2.1.B Define the determinism test: two identical fit() calls that must produce identical Results.
   2.1.C Define the leakage guard test: a fit() call using a DataSource whose train and val sets overlap must trigger an AssertionError.
   2.1.D Define the update procedure for golden-run expected values when a legitimate loop change occurs.
2.2 Non-Goals
   2.2.A Does not cover integration or end-to-end tests with real datasets or models — those are the consuming app's responsibility.
   2.2.B Does not cover performance or throughput benchmarks.
   2.2.C Does not define tests for the timeseries helpers beyond what the leakage guard requires — timeseries unit tests are separate but recommended.

## 3. Requirements
3.1 Functional requirements
   3.1.A The test suite must run with `pytest` from the repository root with no additional arguments.
   3.1.B The golden-run test must use a tiny, fully synthetic model (e.g. a 2-layer linear net) and synthetic data (torch.randn-generated inputs/targets, seeded). The expected loss and metric values must be stored in the test file as constants and compared with rtol=1e-4, atol=1e-5.
   3.1.C The golden-run test must run on CPU only; no GPU is required to run the test suite.
   3.1.D The determinism test must call fit() twice with seed=42 and assert that train_history[i].loss and val_history[i].metrics are identical across both calls to float32 precision.
   3.1.E The leakage guard test must construct a DataSource whose train_loader and val_loader expose the same underlying dataset indices, call fit(), and assert that an AssertionError is raised (COMP-003 §4.5).
   3.1.F Each test must be independent — no shared mutable state across tests.
   3.1.G When a legitimate loop change requires updating golden-run expected values, a developer must explicitly update the constants in the test file and document the change in a comment next to the constants, noting the commit or PR that changed the loop.
3.2 Non-functional requirements
   3.2.A The full test suite must complete in under 60 seconds on a modern laptop CPU.
   3.2.B Tests must produce clear failure messages that identify which expected value mismatched and by how much.

## 4. Interface / Data

### 4.1 Test file layout

```
tests/
├── conftest.py              ← shared fixtures (tiny model, tiny dataset)
├── test_golden_run.py       ← golden-run regression tests
├── test_determinism.py      ← seeded determinism tests
├── test_leakage_guard.py    ← leakage guard AssertionError tests
├── test_protocols.py        ← isinstance() checks for all protocols
├── test_config.py           ← TrainConfig/EvalConfig validation tests
└── timeseries/
    ├── test_dataset.py      ← SlidingWindowDataset unit tests
    ├── test_split.py        ← temporal_split unit tests
    └── test_metrics.py      ← MAE/MSE/RMSE unit tests
```

### 4.2 Shared fixtures (conftest.py)

```python
import pytest
import torch
import torch.nn as nn
from torch.utils.data import DataLoader, TensorDataset

SEED = 42
EPOCHS = 3

@pytest.fixture
def tiny_model_provider():
    class _Provider:
        def build(self):
            torch.manual_seed(SEED)
            return nn.Sequential(nn.Linear(4, 8), nn.ReLU(), nn.Linear(8, 1))
    return _Provider()

@pytest.fixture
def tiny_data_source():
    torch.manual_seed(SEED)
    inputs = torch.randn(50, 4)
    targets = torch.randn(50, 1)
    train_ds = TensorDataset(inputs[:40], targets[:40])
    val_ds = TensorDataset(inputs[40:], targets[40:])

    class _Source:
        def train_loader(self):
            return DataLoader(train_ds, batch_size=10, shuffle=False)
        def val_loader(self):
            return DataLoader(val_ds, batch_size=10, shuffle=False)
    return _Source()

@pytest.fixture
def basic_config():
    import torch.optim as optim
    from trainer_kit import TrainConfig
    return TrainConfig(
        epochs=EPOCHS,
        optimizer_factory=lambda model: optim.SGD(model.parameters(), lr=0.01),
        loss_fn=nn.MSELoss(),
        device="cpu",
        seed=SEED,
    )
```

### 4.3 Golden-run expected values (test_golden_run.py)

```python
# Expected values generated by running the test suite once with a known-good engine.
# To update: run the test with --update-golden flag (or manually re-run and paste outputs),
# then document the reason and commit SHA in this comment block.
# Last updated: [initial values — set after first clean implementation]
EXPECTED_TRAIN_LOSS_EPOCH_1 = None   # fill after first clean run
EXPECTED_TRAIN_LOSS_EPOCH_3 = None
EXPECTED_VAL_LOSS_EPOCH_3 = None
TOLERANCE = dict(rtol=1e-4, atol=1e-5)
```

### 4.4 Leakage guard test fixture

```python
@pytest.fixture
def leaking_data_source():
    """A DataSource whose train and val loaders share the same underlying dataset indices."""
    torch.manual_seed(SEED)
    shared_ds = TensorDataset(torch.randn(20, 4), torch.randn(20, 1))

    class _LeakingSource:
        def train_loader(self):
            return DataLoader(shared_ds, batch_size=5)
        def val_loader(self):
            return DataLoader(shared_ds, batch_size=5)  # same dataset — leaks!
    return _LeakingSource()
```

## 5. Behavior
5.1 Happy path
   5.1.A pytest passes with exit code 0 on a clean implementation.
   5.1.B Golden-run constants are populated after the first clean run; subsequent runs match to tolerance.
   5.1.C Determinism test: result_a.train_history[i].loss == result_b.train_history[i].loss for all i.
5.2 Edge cases
   5.2.A Tests pass on CPU with no CUDA device available.
   5.2.B A legitimate engine change (e.g. optimizer update rule) causes the golden-run test to fail; the developer updates the constants and commits with a clear message explaining the change.
5.3 Error states
   5.3.A Any test that calls fit() with an invalid config must see a ValueError (not a crash or silent wrong result).
   5.3.B The leakage guard test must fail (pytest.raises(AssertionError)) when the shared-index DataSource is used; if the guard is not implemented, this test must fail.

## 6. Acceptance Criteria
6.1 Criteria
   6.1.A [ ] `pytest tests/` exits 0 on a correct implementation with a CPU-only environment (verifies §3.1.A, §3.3.A) — Verified by: [—]
   6.1.B [ ] Golden-run test asserts train_history[2].loss matches EXPECTED_TRAIN_LOSS_EPOCH_3 within TOLERANCE on two separate runs (verifies §3.1.B) — Verified by: [—]
   6.1.C [ ] Determinism test: result_a and result_b produced with seed=42 have identical train and val loss values to float32 precision (verifies §3.1.D) — Verified by: [—]
   6.1.D [ ] Leakage guard test: fit() with leaking_data_source raises AssertionError before epoch 1 completes (verifies §3.1.E, COMP-003 §4.5) — Verified by: [—]
   6.1.E [ ] Full test suite completes in under 60 seconds on a modern laptop CPU (verifies §3.2.A) — Verified by: [—]
   6.1.F [ ] Manually corrupting _engine.py (e.g. removing optimizer.zero_grad()) causes the golden-run test to fail (verifies that the test actually catches regressions) — Verified by: [—]

## 7. Open Questions & Assumptions
7.1 Open questions — Q-8 resolved by ADR-0008; see [open-questions.md](../open-questions.md)
7.2 Assumptions
   7.2.A Golden-run expected values are floating-point constants that must be generated after the first correct implementation; this spec intentionally leaves them as None placeholders — Validated: by design
   7.2.B The test suite uses pytest as the test runner; no other test framework is required — Validated: [—]
   7.2.C torch.manual_seed(SEED) in conftest.py is sufficient for CPU-only determinism; CUDA kernels may require additional settings (torch.use_deterministic_algorithms(True)) which are out of v1 scope — Validated: [—]
