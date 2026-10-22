# Post-Hoc Reports

Every `mosaics` pipeline that produces a scientific catalogue or comparison dataset can render a structured **scireport bundle** describing what the run built, what failed, what had to be type-cast, what it asked of each astronomical archive, and how the resulting dataset passes scientific quality checks.

## The Four Reports

| Report Subcommand                                                                  | Template ID                | Input                                                                                 | Purpose                                                                                                                                                                                                                                                                                                |
|------------------------------------------------------------------------------------|----------------------------|---------------------------------------------------------------------------------------|--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| [`mosaics-report targets`](../cli/reports.md#mosaics-report-targets)               | `mosaics-targets@1`        | `targets_report.yaml` (written automatically by `mosaics-targets run` / `production`) | Complete multi-chapter scientific and operational report on a 5-stage `targets` run: executive summary, overview, per-stage funnel and coverage chapters, scientific assessment (redshift consistency, photometry distributions, sky coverage), conclusion, and reproducibility manifest.              |
| [`mosaics-report crossmatch`](../cli/reports.md#mosaics-report-crossmatch)         | `mosaics-crossmatch@1`     | `<output-stem>.crossmatch.yaml` (written automatically by `mosaics-crossmatch run`)   | Crossmatch report: one chapter per survey with its radius and what it rests on, the overview, input profile, angular separation and multiplicity distributions, MOC footprint filter efficiency, output column definitions, archive request ledger, conclusion, and full remote table schema appendix. |
| [`mosaics-report compare_stamps`](../cli/reports.md#mosaics-report-compare-stamps) | `mosaics-compare-stamps@1` | Two cutout Lance datasets (`--left`, `--right`)                                       | Pixel-for-pixel comparison of two cutout datasets (for example, `hips` service vs `hips --local`, or `lsv` vs `nersc`): eleven composite diagnostic figures with tidy per-stamp metric sidecars.                                                                                                       |
| [`mosaics-report db_constraints`](../cli/reports.md#mosaics-report-db-constraints) | `mosaics-db-constraints@1` | `--config` entrypoint YAML                                                            | Standalone live-archive audit that measures how many rows each SQL constraint in a survey’s `target_selection` query removes.                                                                                                                                                                          |

## Output Layout

When a report renders into an output directory (`report/` by default next to the run’s YAML summary), it leaves four artefacts:

```text
report/
├── <name>.scireport.zip     # Sealed, self-contained scireport bundle (manifest, tables, figures, numbers)
├── md/                      # Markdown chapters and tables for LLM agents and quick inspection
│   ├── index.md
│   └── chapters/
├── pdf/                     # Typeset PDF report for human readers (unless --no-report-pdf is passed)
│   └── <name>.pdf
└── render-manifest.json     # Build metadata and rendered file inventory
```

## Reading Reports with LLM Agents

The `md/` directory inside `report/` contains clean, chapter-by-chapter Markdown where every metric, check verdict, and summary table is rendered in plain text. Coding agents and research assistants can read `report/md/index.md` and the chapter files under `report/md/chapters/` directly without parsing PDFs or large binary catalogues.

## Re-rendering and Exporting with `scireport`

Because `<name>.scireport.zip` is a standard sealed `scireport` bundle, you can re-render it on any machine where `mosaics` (which registers the `mosaics-*` templates) and `scireport` are installed—or re-render a `targets_report.yaml` / `<output-stem>.crossmatch.yaml` file via `mosaics-report`:

```bash
# Re-render a targets run YAML without PDF generation
uv run -m mosaics.cli.report targets -r /data/targets/targets_report.yaml \
  -o /data/targets/report \
  --no-pdf

# Re-render a crossmatch run account
uv run -m mosaics.cli.report crossmatch -r /data/xmatch/matched.crossmatch.yaml \
  -o /data/xmatch/report
```
