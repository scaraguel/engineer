# ml4t-engineer

`ml4t-engineer` provides Polars-first feature engineering, labeling, alternative
bars, and leakage-safe dataset preparation for financial machine learning.

## Source orientation

Runtime code lives under `src/ml4t/engineer/`.

- `api.py`, `features/`, and `discovery/` implement registry-backed feature
  computation and discovery.
- `labeling/` implements path-dependent, fixed-horizon, percentile, meta-label,
  and sample-weighting workflows.
- `bars/` converts trade data into tick, volume, dollar, imbalance, and run bars.
- `dataset.py` and `preprocessing.py` implement train/test preparation and
  train-only transformations.
- `core/` and `config/` contain registry metadata, validation, schemas, and
  reusable configuration models.
- `artifacts/`, `relationships/`, `store/`, `logging/`, and `utils/` provide
  supporting services.

Tests are under `tests/`. Executable examples and repository checks are under
`examples/` and `scripts/`.

## Public workflows

Stable entry points include:

- `ml4t.engineer.compute_features`
- `ml4t.engineer.feature_catalog`
- `ml4t.engineer.create_dataset_builder`
- labeling functions under `ml4t.engineer.labeling`
- bar samplers under `ml4t.engineer.bars`

Start with `README.md` and `docs/getting-started/`. Detailed workflow guidance
is under `docs/user-guide/`, and the generated API reference is under
`docs/api/`.

## Engineering constraints

- Python 3.12, 3.13, 3.14, and 3.15 are supported.
- Feature computation must preserve the documented DataFrame or LazyFrame
  return behavior.
- Changes to registered features must keep implementation, registry metadata,
  dependencies, normalization metadata, and lookback behavior consistent.
- Fit preprocessing state only on training data.
- Keep optional dependencies isolated from the base import.
- Shared data-contract changes originate in `ml4t-specs`.

## Quality commands

Run from the repository root:

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

## Maintainer-directed change batching

For our own fixes and agent work directed by the maintainer, use a bounded, coherent PR rather
than one PR per small defect or implementation step. One PR may close several related issues
when the changes serve the same user workflow, subsystem repair or compatibility objective and
can be reviewed and reverted together. Keep implementation, regressions, documentation and
follow-up repairs together; use focused commits and retain evidence for each defect. Do not
create separate issues or PRs merely for individual files, tests, assertions or platform variants.

Split unrelated changes, independently risky or breaking changes, separate ownership boundaries,
and work that needs independent release or rollback decisions. A general maintenance label alone
does not make a batch coherent. Reuse or safely consolidate existing owned submissions instead of
opening overlapping PRs; preserve contributor attribution, recoverable history and other owners'
active work. Do not rewrite or close someone else's work merely to reduce the queue count.

Run focused checks while iterating, then qualify the complete final revision with every applicable
required gate. Group related work before expensive CI; do not weaken checks, bypass review or
claim old-head checks qualify changed code. This maintainer-directed batching rule takes precedence
over a literal one-issue-per-PR reading of contributor intake guidance.

<!-- OPENWIKI:START -->

## OpenWiki

This repository has a generated `openwiki/` evidence index. It is optional just-in-time context, not required startup reading.

- Do not enumerate, preload, or search wikis at task start. Use retrieval when the user asks for it, when unfamiliar architecture or dependency behavior materially affects the task, or when source inspection leaves an important uncertainty. Stop once the question is grounded.
- When those conditions apply and OpenWiki retrieval tools are available, use `openwiki_search` for just-in-time context and `openwiki_read` for the relevant complete sections. If search returns `workspace_required`, ask which listed workspace to use and retry with its ID.
- Use `openwiki_list_workspaces` or `openwiki_list_wikis` when workspace membership itself needs to be discovered.
- If the retrieval tools are unavailable, read `openwiki/quickstart.md` and follow its links to the relevant pages.
- Treat source code and tests as authoritative. A brief's unknowns and review items are verification gaps, not automatic requirements.
- Prefer the narrowest quiet validation that proves the changed behavior. Preserve complete failure output.

Use the repository's documented maintenance process to refresh generated pages after source changes. Check its configuration and completed runs before relying on automatic updates. Do not hand-edit generated OpenWiki pages unless explicitly asked; prefer updating source code/docs and letting OpenWiki regenerate.

<!-- OPENWIKI:END -->
