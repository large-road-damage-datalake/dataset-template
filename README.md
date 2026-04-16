# Dataset Package Template

This template mirrors the current publication contract used by the Large Road Damage Detection Benchmark website.

## What This Template Covers

- Publication docs: `README.md`, `ABSTRACT.md`, `SUMMARY.md`, `CITATION.*`, `DOWNLOAD.md`, `LICENSE.md`
- Metadata contract: `METADATA.json`
- Generated analytics contract: `stats/*` and `visualizations/*`
- Package-level validation helper: `python -m src.main --repo-root .`

## End-to-End Workflow

1. Copy this template to a working location (or clone your template repo).
2. Create a dataset config using `config.example.yaml` as a guide.
3. Run the pipeline from this template's scripts:
   - `python scripts/run_pipeline.py --config <config-path> --output <dataset-package-path>`
4. Optionally apply dataset-specific patch logic for that dataset family.
5. Re-run strict validation (executed by `run_pipeline.py` unless `--no-strict` is passed).
6. Validate package contract from inside the generated package:
   - `python -m src.main --repo-root .`
7. Commit and push the generated dataset package repository.

## Script Location Policy

- Keep publication-critical scripts in `scripts/` inside this template.
- This includes orchestration and enrichment modules such as:
  - `run_pipeline.py`
  - `build_artifacts.py`
  - `generate_visualizations.py`
  - `enrich_metadata.py`
  - `validate_dataset_package.py`
  - `task_classification.py`
  - `select_diverse_samples.py`
  - `compute_stats.py`, `stats_core.py`, `loaders.py`
- If you maintain a separate local workspace copy, treat this template copy as the source that is pushed and periodically sync local changes into it.

## Website-Consumed Artifacts

At minimum, keep these files committed:

- `METADATA.json`
- `stats/stats.json`
- `stats/manifest.json`
- `stats/samples_summary.json`
- `visualizations/manifest.json`
- `visualizations/samples/samples_manifest.json`
- Markdown publication docs (`README.md`, `ABSTRACT.md`, `SUMMARY.md`, `CITATION.md`, `DOWNLOAD.md`, `LICENSE.md`)

## Class Label Taxonomy

To improve web-side filtering and cross-dataset comparability, keep class metadata normalized:

- `project_context.damage_types`: high-level canonical families only (for example: `crack`, `pothole`, `patch_repair`).
- `annotation_schema[].classes[]`: annotation-centric entries only:
   - Keep dataset label fields such as `id`, `name`, `description`, `color`, and optional `instances`.
   - Keep `canonical_name` as the taxonomy reference key.
- `class_taxonomy`: canonical taxonomy source used for indexing/filtering and semantic metadata (`display_name`, `damage_family`, `damage_subtype`, `role`, `is_damage_target`, `aliases`, `source_labels`, `taxonomy_source`, `taxonomy_unresolved`).

The enrichment script (`scripts/enrich_metadata.py`) now uses `scripts/label_taxonomy_registry.json` as the authoritative alias registry, then applies strict fallback behavior:

- If label matches registry alias: map directly to canonical class.
- If label looks like a code (for example `D00`) and class `description` exists: try description-based canonical mapping.
- If still unresolved and label is a code: mark as `role=code_legacy` with `damage_family=unknown`.
- Otherwise: mark as `role=unknown` and flag for taxonomy review.

For datasets with code labels, add explicit overrides in `config.yaml` via `label_codebook`.
The strict validator enforces taxonomy completeness and consistency, and ensures every `annotation_schema[].classes[]` entry references an existing `class_taxonomy.classes[]` entry by `canonical_name`.

## Notes

- Do not ignore generated `stats/` and `visualizations/` outputs in Git.
- Keep metadata keys stable to avoid frontend regressions.
- Use UTF-8 for all text files.
