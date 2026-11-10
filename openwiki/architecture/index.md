# Files

- [Alternative Bar Sampling](bar-sampling.md) - How trade-level inputs become activity-driven tick, volume, dollar, imbalance, and run bars, including thresholds, adaptation, state transitions, and output guarantees.
- [Leakage-Safe Dataset and Preprocessing Pipeline](dataset-preprocessing.md) - How aligned Polars features, labels, and dates become leakage-safe train/test or cross-validation partitions, with train-only fitted preprocessing and NumPy interoperability.
- [Registry-Driven Feature Computation](feature-computation.md) - End-to-end architecture of compute_features, from configuration parsing and temporal validation through registry dispatch, dependency ordering, grouped execution, output naming, and discovery.
- [Labeling Runtime and Outcome Semantics](labeling-runtime.md) - Architecture of fixed-horizon, percentile, barrier, trend, meta-labeling, uniqueness, and bootstrap workflows, with explicit calendar, horizon, censoring, and output contracts.
- [System Map and Public Surfaces](system-map.md) - Maps the Polars-first feature, labeling, bar-sampling, dataset, discovery, configuration, artifact, relationship-analysis, and storage domains, and distinguishes primary APIs from support and optional integration surfaces.
