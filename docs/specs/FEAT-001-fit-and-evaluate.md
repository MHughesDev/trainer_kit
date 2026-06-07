# Spec: FEAT-001 — fit() and evaluate() Entry Points

**Spec ID:** FEAT-001
**Type:** Feature
**Status:** Draft
**Date:** 2026-06-07
**Author:** Agent

## 1. Overview
1.1 Purpose — Defines the exact signatures, pre-conditions, validation behavior, and return contracts of fit() and evaluate() — the only two public functions in trainer_kit's top-level namespace.
1.2 Context — ADR-0011 decided on a single typed entry point fit(model_provider, data_source, config) → Result, with a matching evaluate() for validation-only runs. These functions are thin, stable wrappers over the engine (COMP-003) that handle argument validation and delegate to _run_fit / _run_evaluate. Their signatures are frozen once 1.0 ships.
1.3 Related artifacts
   1.3.A ADR-0011: [adr/0011-entry-point.md](../adr/0011-entry-point.md)
   1.3.B ADR-0001: [adr/0001-inject-dependencies-via-protocols.md](../adr/0001-inject-dependencies-via-protocols.md)
   1.3.C ADR-0007: [adr/0007-device-seam.md](../adr/0007-device-seam.md)
   1.3.D Related specs: COMP-002 (ModelProvider, DataSource, the protocols accepted as arguments), DATA-001 (TrainConfig, EvalConfig, Result), COMP-003 (engine that does the work)

## 2. Scope
2.1 Goals
   2.1.A Define exact function signatures for fit() and evaluate().
   2.1.B Define pre-condition validation that runs before delegating to the engine.
   2.1.C Specify that these are the only two public functions — no other engine internals are exported.
2.2 Non-Goals
   2.2.A Does not define the loop logic — that is COMP-003.
   2.2.B Does not define checkpointing, scheduling, or AMP — all out of v1 scope.
   2.2.C Does not define a builder pattern or staged API — those are future possibilities that must not break fit()/evaluate() (ADR-0011).

## 3. Requirements
3.1 Functional requirements
   3.1.A fit() must accept exactly three positional arguments: model_provider (ModelProvider), data_source (DataSource), config (TrainConfig). No keyword-only extras in v1.
   3.1.B evaluate() must accept exactly three positional arguments: model_provider (ModelProvider), data_source (DataSource), config (EvalConfig).
   3.1.C Both functions must return a Result.
   3.1.D fit() and evaluate() must validate that model_provider satisfies isinstance(model_provider, ModelProvider) and raise TypeError with a descriptive message before calling the engine if not.
   3.1.E fit() and evaluate() must validate that data_source satisfies isinstance(data_source, DataSource) and raise TypeError before calling the engine if not.
   3.1.F fit() and evaluate() must not catch exceptions raised by the engine or by protocol implementations — all errors propagate to the caller unmodified.
   3.1.G fit() and evaluate() must be importable from the top-level trainer_kit namespace (COMP-001 §3.1.A).
3.2 Non-functional requirements
   3.2.A The function signatures (parameter names, parameter count, return type) must not change in any 0.x release and must follow the ADR-0003 one-minor deprecation cycle before any breaking change at 1.0+.
   3.2.B fit() and evaluate() must remain device-agnostic — they accept no device argument directly; device is a field of TrainConfig / EvalConfig (ADR-0007).

## 4. Interface / Data

### 4.1 fit()

```python
from trainer_kit._protocols import ModelProvider, DataSource
from trainer_kit._config import TrainConfig, Result
from trainer_kit._engine import _run_fit

def fit(
    model_provider: ModelProvider,
    data_source: DataSource,
    config: TrainConfig,
) -> Result:
    """Train a model end-to-end and return the full training history and final metrics.

    Args:
        model_provider: Implements ModelProvider.build() -> nn.Module.
        data_source: Implements DataSource.train_loader() and DataSource.val_loader().
        config: Frozen TrainConfig with all hyperparameters and injected objects.

    Returns:
        Result with train_history, val_history, and final_metrics.

    Raises:
        TypeError: if model_provider or data_source do not satisfy their protocols.
        ValueError: if config fields are invalid (raised by TrainConfig.__post_init__).
    """
    if not isinstance(model_provider, ModelProvider):
        raise TypeError(
            f"model_provider must satisfy the ModelProvider protocol "
            f"(needs a build() -> nn.Module method), got {type(model_provider)}"
        )
    if not isinstance(data_source, DataSource):
        raise TypeError(
            f"data_source must satisfy the DataSource protocol "
            f"(needs train_loader() and val_loader() methods), got {type(data_source)}"
        )
    return _run_fit(model_provider, data_source, config)
```

### 4.2 evaluate()

```python
from trainer_kit._config import EvalConfig

def evaluate(
    model_provider: ModelProvider,
    data_source: DataSource,
    config: EvalConfig,
) -> Result:
    """Run a single validation pass and return metrics.

    Args:
        model_provider: Implements ModelProvider.build() -> nn.Module.
        data_source: Implements DataSource.val_loader() (train_loader not called).
        config: EvalConfig with device, metrics, and logger.

    Returns:
        Result with val_history and final_metrics; train_history is empty.

    Raises:
        TypeError: if model_provider or data_source do not satisfy their protocols.
    """
    if not isinstance(model_provider, ModelProvider):
        raise TypeError(
            f"model_provider must satisfy the ModelProvider protocol, got {type(model_provider)}"
        )
    if not isinstance(data_source, DataSource):
        raise TypeError(
            f"data_source must satisfy the DataSource protocol, got {type(data_source)}"
        )
    return _run_evaluate(model_provider, data_source, config)
```

## 5. Behavior
5.1 Happy path
   5.1.A fit() called with valid arguments completes all epochs and returns a Result with len(train_history) == config.epochs.
   5.1.B evaluate() called with valid arguments runs one validation pass and returns a Result with len(train_history) == 0 and len(val_history) == 1.
   5.1.C fit() and evaluate() are importable: `from trainer_kit import fit, evaluate`.
5.2 Edge cases
   5.2.A A class that has build(), train_loader(), and val_loader() methods but does not explicitly inherit from any trainer_kit type satisfies both protocols and passes the isinstance() checks.
   5.2.B evaluate() calls data_source.val_loader() only; data_source.train_loader() is never called.
5.3 Error states
   5.3.A fit(None, data_source, config) raises TypeError before the engine is invoked.
   5.3.B fit(model_provider, data_source, config) where config.epochs=0 raises ValueError from TrainConfig.__post_init__ before fit() is called (the config is invalid at construction time, not at fit() time).
   5.3.C Any exception from the engine or from a protocol method propagates to the caller without wrapping.

## 6. Acceptance Criteria
6.1 Criteria
   6.1.A [ ] `from trainer_kit import fit, evaluate` succeeds in a clean install (verifies §3.1.G) — Verified by: [—]
   6.1.B [ ] fit(None, data_source, config) raises TypeError with a message naming ModelProvider (verifies §3.1.D) — Verified by: [—]
   6.1.C [ ] fit(model_provider, None, config) raises TypeError with a message naming DataSource (verifies §3.1.E) — Verified by: [—]
   6.1.D [ ] evaluate() result has train_history == [] (verifies §5.1.B) — Verified by: [—]
   6.1.E [ ] A duck-typed object (no trainer_kit inheritance) satisfying ModelProvider and DataSource passes both isinstance checks (verifies §5.2.A) — Verified by: [—]
   6.1.F [ ] fit() and evaluate() have identical parameter names across all 0.x releases; any future change is documented in a CHANGELOG entry with a deprecation notice (verifies §3.2.A) — Verified by: [—]

## 7. Open Questions & Assumptions
7.1 Open questions — Q-11 resolved by ADR-0011; see [open-questions.md](../open-questions.md)
7.2 Assumptions
   7.2.A isinstance() checks against @runtime_checkable Protocols check only the presence of the required methods, not their signatures. mypy/pyright performs full signature checking at static analysis time — Validated: [—]
   7.2.B evaluate() calling only val_loader() and not train_loader() is sufficient; DataSource implementations are expected to handle this gracefully — Validated: [—]
