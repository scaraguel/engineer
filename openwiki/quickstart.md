---
type: task-oriented quickstart
title: Quickstart and Task Router
description: Start here to choose the supported ml4t-engineer workflow, identify the contract that owns a proposed change, and route to the relevant architecture, concept, workflow, operations, or testing guide.
tags: [quickstart, task-routing, development, public-api, contracts]
verified:
  - by: openwiki/0.9.0
    at: 2026-11-10T03:14:41.957Z
sources:
  - id: openwiki-source-8037e2358a2c4f9b2c722a11
    resource: repo://AGENTS.md
  - id: openwiki-source-05ccef8d4cf1698187f20464
    resource: repo://pyproject.toml
  - id: openwiki-source-23775c3de52f3ab95a13cb8b
    resource: repo://README.md
  - id: openwiki-source-5f53a0a5a3646af0b613baf7
    resource: repo://src/ml4t/engineer/__init__.py
  - id: openwiki-source-665c524c1477f2c44e244c1a
    resource: repo://src/ml4t/engineer/AGENTS.md
  - id: openwiki-source-64628df0a443ebf90b9f6f81
    resource: repo://src/ml4t/engineer/api.py
  - id: openwiki-source-b9599f123b8324ce4e9e6a29
    resource: repo://src/ml4t/engineer/core/decorators.py
  - id: openwiki-source-dd94cd6d61ff1bce52015c35
    resource: repo://src/ml4t/engineer/dataset.py
  - id: openwiki-source-9e9fdac07c092cd2336846d6
    resource: repo://src/ml4t/engineer/discovery/catalog.py
  - id: openwiki-source-defdc6d61bb07034e84932fa
    resource: repo://src/ml4t/engineer/store/offline.py
  - id: openwiki-source-11519246eac934485315b3cb
    resource: repo://tests/test_api.py
generated: { by: "openwiki/0.9.0", at: "2026-11-10T03:14:41.957Z" }
---

# Quickstart and Task Router

`ml4t-engineer` is a Polars-first, in-process toolkit for feature computation, financial labels, activity-based bars, and leakage-safe model datasets. The main workflow accepts market frames and returns frames or fold data; it is a library, not a service, and the core quickstart needs no credentials or external infrastructure. Begin with the public workflow nearest the proposed change, then follow the task routes below instead of scanning every package module.

## Protect these invariants first

Before editing code, identify whether the change can violate any of these high-risk contracts:

1. **Preserve eager versus lazy behavior.** A public path that accepts `polars.DataFrame` and `polars.LazyFrame` must preserve the documented return kind; do not silently collect lazy input. Grouped rolling and lagged work must remain isolated by asset.
2. **Keep implementation and registration metadata aligned.** A registered feature's callable, defaults, input requirements, dependencies, normalization metadata, and first usable row form one contract. Updating only the numerical function is incomplete.
3. **Fit preprocessing on training observations only.** Each cross-validation fold uses a fresh scaler fitted on that fold's training rows, then applies the fitted state to both train and test data. The simple split follows the same fit-train/transform-test rule.
4. **Keep optional dependencies outside the base import path.** Extra-backed behavior must use the existing guarded or lazy import boundary. `import ml4t.engineer` must continue to work with base dependencies alone, while selecting an unavailable integration should fail with an actionable message or documented fallback.
5. **Originate shared contract changes in `ml4t-specs`.** `FeedSpec`, calendar/window declarations, and shared artifact primitives are cross-library contracts consumed here. Change the owning package first rather than creating a divergent local definition, then qualify the ecosystem impact.

These are review stop conditions, not style preferences. For the detailed temporal rules, read [Temporal and Market-Data Contracts](concepts/temporal-data-contracts.md); for the tests that prove each invariant, read [Testing, Validation, and Release Gates](testing/quality-gates.md).

## Install and run the shortest workflow

Development source supports Python 3.12 through 3.15. From the repository root, install the base package for the core quickstart:

```bash
uv sync --no-dev
```

Then compute registry-backed features on a Polars frame:

```python
from datetime import date, timedelta

import polars as pl
from ml4t.engineer import compute_features

close = [100.0 + i * 0.1 + (i % 7) * 0.2 for i in range(100)]
ohlcv = pl.DataFrame(
    {
        "timestamp": [date(2024, 1, 1) + timedelta(days=i) for i in range(100)],
        "open": close,
        "high": [price + 1.0 for price in close],
        "low": [price - 1.0 for price in close],
        "close": close,
        "volume": [100_000 + i * 100 for i in range(100)],
    }
)

features = compute_features(ohlcv, ["rsi", "macd", "atr"], calendar=None)

assert {"rsi", "macd", "atr"} <= set(features.columns)
assert features.height == ohlcv.height
```

`compute_features()` appends requested outputs to the input columns. Explicit `calendar=None` selects observation-row mode: periods count rows in input order and schedule validation is disabled. This is appropriate for the self-contained example, but it is not interchangeable with a declared calendar, expected-bar, session, or elapsed-time contract. Route any change involving timestamps, grids, holidays, feed declarations, or window units through the temporal contract guide before changing feature or label logic.

For a development environment with the extras exercised by repository qualification, use:

```bash
uv sync --dev --extra docs --extra ta --extra store --extra viz
```

The other published extras are `calendars`, `stats`, and `ml`. Do not promote one of their dependencies into unconditional imports merely because the full development environment contains it.

## Choose the public surface before the implementation file

Use imports that express ownership:

| Goal | Public starting point | What owns the behavior |
|---|---|---|
| Compute registered features | `from ml4t.engineer import compute_features` | Registry-driven parsing, validation, dependency ordering, and execution |
| Search feature metadata | `from ml4t.engineer import FeatureCatalog, feature_catalog` | The same feature registry used by computation |
| Build leakage-safe model data | `from ml4t.engineer import create_dataset_builder` | Dataset alignment, temporal splitting, and train-only preprocessing |
| Create labels or sample weights | `from ml4t.engineer.labeling import ...` | The labeling subpackage; label functions are not flattened into the root namespace |
| Build alternative bars | `from ml4t.engineer.bars import ...` | The bars subpackage; samplers are direct-submodule APIs |
| Persist features locally | `from ml4t.engineer.store import OfflineFeatureStore` | Optional DuckDB-backed storage |

The package root deliberately exports the main compute, discovery, dataset, and preprocessing symbols plus selected submodules; it does not flatten every labeler, sampler, configuration model, artifact type, relationship function, or store class. Prefer an owning package's `__init__.py` export over an implementation-file import. See [System Map and Public Surfaces](architecture/system-map.md) for the complete boundary map, including configuration, artifacts, relationship analysis, and adjacent libraries.

## Task router

| If the proposed change concerns… | Read first | Then use |
|---|---|---|
| Public imports, domain ownership, config/artifact boundaries, relationship analysis, or where a new capability belongs | [System Map and Public Surfaces](architecture/system-map.md) | The owning subpackage's `AGENTS.md` and public `__init__.py` |
| `FeedSpec`, calendar precedence, bar grids, timestamps, DST, observation/expected-bar/session/elapsed windows, or panel isolation | [Temporal and Market-Data Contracts](concepts/temporal-data-contracts.md) | [Feature Computation](architecture/feature-computation.md) or [Labeling Runtime](architecture/labeling-runtime.md), depending on the caller |
| How `compute_features()` parses, validates, orders dependencies, groups assets, dispatches windows, or names outputs | [Registry-Driven Feature Computation](architecture/feature-computation.md) | [Adding or Changing a Registered Feature](workflows/adding-or-changing-features.md) when implementation or metadata changes |
| Adding an indicator, changing defaults/lookback/dependencies, exposing it, or choosing feature tests | [Adding or Changing a Registered Feature](workflows/adding-or-changing-features.md) | [Testing, Validation, and Release Gates](testing/quality-gates.md) for focused and final checks |
| Forward returns, barriers, trend scanning, percentiles, meta-labels, uniqueness, or bootstrap weights | [Labeling Runtime and Outcome Semantics](architecture/labeling-runtime.md) | [Temporal and Market-Data Contracts](concepts/temporal-data-contracts.md) whenever a horizon or calendar changes |
| Tick, volume, dollar, imbalance, or run bars; side classification; thresholds; partial bars | [Alternative Bar Sampling](architecture/bar-sampling.md) | [Testing, Validation, and Release Gates](testing/quality-gates.md) for bar and grid regressions |
| Feature/label alignment, feature selection, train/test splitting, external CV, scaler lifecycle, or NumPy interop | [Leakage-Safe Dataset and Preprocessing Pipeline](architecture/dataset-preprocessing.md) | Focused dataset/preprocessing tests from [Quality Gates](testing/quality-gates.md) |
| DuckDB save/load/append/replace, table validation, connection lifecycle, or point-in-time joins | [Offline Feature Store Operations](operations/offline-feature-store.md) | Install the `store` extra and run its focused tests; preserve the missing-dependency path |
| Optional dependencies, numerical parity, performance, packaging, CI, docs, wheel behavior, security, or release readiness | [Testing, Validation, and Release Gates](testing/quality-gates.md) | Start narrow while iterating, then run all applicable final gates |

If a change crosses rows—for example, a new calendar-aware registered feature—follow every relevant route. The ownership boundary determines where the contract changes; the quality guide determines how the combined change is proven.

## Source-of-truth order

When prose, metadata, and behavior appear to disagree, use this order to plan the fix:

1. **Shared ecosystem contracts:** `ml4t-specs` owns shared feed/time and artifact primitives; this repository pins the compatible runtime range in `pyproject.toml` and adapts those contracts locally.
2. **Public behavior:** exported APIs, their validated inputs/outputs, and the tests at those entrypoints define observable package behavior.
3. **Registered feature contract:** the implementation, `@feature` metadata, authoritative lookback calculation, registry/discovery view, and integration tests must agree. The decorator rejects unknown metadata parameters and metadata defaults that differ from callable defaults, but tests are still needed for numerical output and first usable rows.
4. **Repository operations:** `pyproject.toml`, workflow files, and scripts own supported Python versions, dependency groups, test markers, static checks, builds, and release qualification.
5. **Guides:** root and scoped `AGENTS.md` files route agent work; README and docs explain supported user workflows. Source and tests remain authoritative if a guide is stale.

The root-exported `get_agent_docs()` can locate packaged `AGENTS.md` guides by logical area. Use it for navigation, not as a runtime feature or contract registry.

## Development loop and qualification

Run the narrowest relevant regression while iterating—for example:

```bash
uv run pytest tests/test_api.py -q
uv run pytest tests/test_registry_lookbacks.py -q
uv run pytest tests/test_labeling_time_contract.py -q
uv run pytest tests/store/test_offline.py -q
```

Then qualify the complete revision with every applicable local gate:

```bash
uv sync --dev --extra docs --extra ta --extra store --extra viz
uv run ruff check src/ tests/ examples/ scripts/
uv run ruff format --check src/ tests/ examples/ scripts/
uv run ty check
uv run pytest tests/ -q
uv build
uv run python -c "import ml4t.engineer"
uv run mkdocs build --strict
```

The default pytest configuration excludes tests marked `perf`; run an affected performance path explicitly as documented in [Quality Gates](testing/quality-gates.md). Optional suites may skip when an extra is absent, so a skip does not prove that integration. Local checks are preflight rather than a substitute for the clean-wheel, supported-OS/Python, security, and ecosystem gates used by CI.

## Primary repository references

- [Repository agent guide](../AGENTS.md)
- [README and user quickstart](../README.md)
- [Package and tool configuration](../pyproject.toml)
- [Package-scoped agent guide](../src/ml4t/engineer/AGENTS.md)
- [Root public exports and agent navigation](../src/ml4t/engineer/__init__.py)
