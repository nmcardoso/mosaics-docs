# `mosaics-report` — Post-Hoc Scientific Reports

`mosaics-report` (`uv run -m mosaics.cli.report` or `uv run -m mosaics.report.<task>`) builds and renders the four MOSAICS `scireport` bundles into chapter-split Markdown (`md/`), a sealed `.scireport.zip` archive, and a print-ready PDF (`pdf/report.pdf`).

## Report tasks

- `targets`: Render a targets pipeline run from `targets_report.yaml` (and optionally the produced catalogue via `--catalog` for photometric closure and zero-point assessment figures).
- `crossmatch`: Render a crossmatch run account (`<output-stem>.crossmatch.yaml`) as a report with one chapter per survey.
- `compare_stamps`: Compare two cutout Lance datasets pixel for pixel (power spectra, residuals vs. flux/radius/gradient, worst/best stamp montages, and significance maps).
- `db_constraints`: Decompose each survey’s `target_selection` SQL query and measure how many rows each `WHERE` constraint removes against the live archive.

## Examples

```console
$ uv run -m mosaics.cli.report targets -r tests/fixtures/targets_report.yaml -o ./out_targets_report --no-pdf
$ uv run -m mosaics.cli.report crossmatch -r tests/fixtures/crossmatch_account.yaml -o ./out_crossmatch_report --no-pdf
$ uv run -m mosaics.cli.report compare_stamps --left left.lance --right right.lance --report ./out_compare_report --no-pdf
$ uv run -m mosaics.cli.report db_constraints --config defs/targets.yaml -o ./out_constraints_report --no-pdf
```

## Command-line reference

<!-- generated:cli-reports -->

**Entry points:** `mosaics-report` or `uv run -m mosaics.cli.report`

### Top-level options

```text
mosaics-report [-h] TASK ...
```

### `mosaics-report targets`

Render a targets-pipeline targets_report.yaml as a report: bundle, Markdown and PDF.

```text
mosaics-report targets [-h] --report REPORT [--output OUTPUT]
                              [--no-pdf] [--dpi DPI] [--catalog CATALOG]
                              [--assess-sample ASSESS_SAMPLE]
                              [--assess-figure-points ASSESS_FIGURE_POINTS]
                              [--log-level {DEBUG,INFO,WARNING,ERROR}]
```

| Option / Argument                               | Default    | Choices                             | Description                                                                                                                                                                                                                                                                                     |
|-------------------------------------------------|------------|-------------------------------------|-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `--report`, `-r` `REPORT`                       | *required* | —                                   | The targets_report.yaml written beside a target catalogue.                                                                                                                                                                                                                                      |
| `--output`, `-o` `OUTPUT`                       | `None`     | —                                   | Directory to write the report into. Default: report/ beside the input.                                                                                                                                                                                                                          |
| `--no-pdf`                                      | `true`     | —                                   | Skip the PDF and write only the bundle and the Markdown.                                                                                                                                                                                                                                        |
| `--dpi` `DPI`                                   | `200`      | —                                   | Resolution of the PNG renditions of the figures mosaics draws itself (the node figures); every other figure is drawn by scireport at its own. Default: 200.                                                                                                                                     |
| `--catalog` `CATALOG`                           | `None`     | —                                   | Catalogue the scientific assessment chapter reads, overriding the path recorded in the report – for a report rendered on a machine where that path has moved. Default: the recorded path.                                                                                                       |
| `--assess-sample` `ASSESS_SAMPLE`               | `None`     | —                                   | Row cap for the scientific assessment chapter’s queries. Default: mosaics.report.targets.photometry.DEFAULT_ASSESS_SAMPLE.                                                                                                                                                                      |
| `--assess-figure-points` `ASSESS_FIGURE_POINTS` | `None`     | —                                   | Points kept in the bundle per point-cloud figure of the scientific assessment: a seeded random sample, so a bundle renders without the catalogue. The tables are computed on the whole assessment sample either way. Default: mosaics.report.targets.chapters.assessment.DEFAULT_FIGURE_POINTS. |
| `--log-level` `LOG_LEVEL`                       | `INFO`     | `DEBUG`, `INFO`, `WARNING`, `ERROR` | Logging verbosity. Default: INFO.                                                                                                                                                                                                                                                               |

### `mosaics-report crossmatch`

Render a crossmatch run account (<stem>.crossmatch.yaml) as a report: bundle, Markdown and PDF.

```text
mosaics-report crossmatch [-h] --report REPORT [--output OUTPUT]
                                 [--no-pdf] [--dpi DPI]
                                 [--log-level {DEBUG,INFO,WARNING,ERROR}]
```

| Option / Argument         | Default    | Choices                             | Description                                                                                            |
|---------------------------|------------|-------------------------------------|--------------------------------------------------------------------------------------------------------|
| `--report`, `-r` `REPORT` | *required* | —                                   | The <stem>.crossmatch.yaml written beside a matched table.                                             |
| `--output`, `-o` `OUTPUT` | `None`     | —                                   | Directory to write the report into. Default: report/ beside the account.                               |
| `--no-pdf`                | `true`     | —                                   | Skip the PDF and write only the bundle and the Markdown.                                               |
| `--dpi` `DPI`             | `200`      | —                                   | Accepted for compatibility; this report draws no figure of its own, so it has no effect. Default: 200. |
| `--log-level` `LOG_LEVEL` | `INFO`     | `DEBUG`, `INFO`, `WARNING`, `ERROR` | Logging verbosity. Default: INFO.                                                                      |

### `mosaics-report compare_stamps`

Compare two cutout Lance datasets built from the same targets.

```text
mosaics-report compare_stamps [-h] --left LEFT --right RIGHT
                                     [--left-label LEFT_LABEL]
                                     [--right-label RIGHT_LABEL]
                                     [--id-col ID_COLUMN]
                                     [--image-col IMAGE_COLUMN] [--report DIR]
                                     [--no-pdf] [--sample-size SAMPLE_SIZE]
                                     [--dpi DPI]
                                     [--log-level {DEBUG,INFO,WARNING,ERROR,CRITICAL}]
```

| Option / Argument             | Default      | Choices                                         | Description                                                                                                                                    |
|-------------------------------|--------------|-------------------------------------------------|------------------------------------------------------------------------------------------------------------------------------------------------|
| `--left` `LEFT`               | *required*   | —                                               | Left dataset directory.                                                                                                                        |
| `--right` `RIGHT`             | *required*   | —                                               | Right dataset directory.                                                                                                                       |
| `--left-label` `LEFT_LABEL`   | `hips`       | —                                               | Label for the left dataset in the report. Default: ‘hips’.                                                                                     |
| `--right-label` `RIGHT_LABEL` | `lsv`        | —                                               | Label for the right dataset in the report. Default: ‘lsv’.                                                                                     |
| `--id-col` `ID_COLUMN`        | `mosaics_id` | —                                               | Name of the shared row-key column. Default: mosaics_id.                                                                                        |
| `--image-col` `IMAGE_COLUMN`  | `image`      | —                                               | Name of the image column. Default: image.                                                                                                      |
| `--report` `DIR`              | `None`       | —                                               | Also write a visual comparison report into this directory: the sealed bundle, Markdown split by section (md/) and a PDF (pdf/).                |
| `--no-pdf`                    | `true`       | —                                               | With –report, skip the PDF and write only the bundle and the Markdown.                                                                         |
| `--sample-size` `SAMPLE_SIZE` | `5000`       | —                                               | Cap on points sampled for the scatter plot and for the O(n^2)-ish metrics (MMD, per-pair KS/Anderson-Darling/Cramer-von Mises). Default: 5000. |
| `--dpi` `DPI`                 | `200`        | —                                               | Resolution of the figures’ PNG renditions. Default: 200.                                                                                       |
| `--log-level` `LOG_LEVEL`     | `INFO`       | `DEBUG`, `INFO`, `WARNING`, `ERROR`, `CRITICAL` | Logging verbosity level. Default: INFO.                                                                                                        |

### `mosaics-report db_constraints`

Count how many rows each constraint of a target query removes.

```text
mosaics-report db_constraints [-h] [--only SLUG [SLUG ...]]
                                     [--output OUTPUT] [--no-pdf]
                                     [--config FILE] [--survey REF [REF ...]]
                                     [--exclude-survey REF [REF ...]]
                                     [--cache PATH] [--timeout SECONDS]
                                     [--budget SECONDS] [--dpi DPI]
                                     [--log-level {DEBUG,INFO,WARNING,ERROR}]
```

| Option / Argument         | Default          | Choices                             | Description                                                                                                                                                                                                                                                                                |
|---------------------------|------------------|-------------------------------------|--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `--only` `SLUG`           | `None`           | —                                   | Restrict to these column prefixes, e.g. ls11 sdss17s.                                                                                                                                                                                                                                      |
| `--output`, `-o` `OUTPUT` | `db_constraints` | —                                   | Directory to write the report into: the sealed bundle, Markdown split by chapter (md/) and a PDF (pdf/). Default: db_constraints.                                                                                                                                                          |
| `--no-pdf`                | `true`           | —                                   | Skip the PDF and write only the bundle and the Markdown.                                                                                                                                                                                                                                   |
| `--config`, `-c` `FILE`   | `None`           | —                                   | Survey definitions naming the queries to analyse and which service answers each. This is the whole of the discovery – pass defs/targets.yaml, or –survey, to scope it to one run. Default: full.yaml under defs/, which is every survey.                                                   |
| `--survey` `REF`          | `None`           | —                                   | Survey files to run on, replacing the entrypoint’s `surveys:`: library names or paths, `.yaml` optional, globs allowed (`--survey surveys/ls11 surveys/wise`). The entrypoint’s `overrides:` still apply.                                                                                  |
| `--exclude-survey` `REF`  | `None`           | —                                   | Survey files to remove from the run, same syntax as –survey.                                                                                                                                                                                                                               |
| `--cache` `PATH`          | `None`           | —                                   | Also write the counts as JSON. They take minutes and a network to produce, so keeping them is what lets one run be compared against the last without asking the archives twice.                                                                                                            |
| `--timeout` `SECONDS`     | `None`           | —                                   | Run each count as a server-side job with this timeout. Unset runs them synchronously, which some unconstrained counts are too slow for.                                                                                                                                                    |
| `--budget` `SECONDS`      | `None`           | —                                   | Wall clock allowed per query before the rest of its constraints are reported as uncounted. Unset waits as long as it takes; 120 is several times what any query that does answer has needed, and bounds the unconstrained count over the DR11 tractor table, which does not answer at all. |
| `--dpi` `DPI`             | `200`            | —                                   | Accepted for compatibility; this report draws no figure of its own, so it has no effect. Default: 200.                                                                                                                                                                                     |
| `--log-level` `LOG_LEVEL` | `INFO`           | `DEBUG`, `INFO`, `WARNING`, `ERROR` | Logging verbosity. Default: INFO.                                                                                                                                                                                                                                                          |
<!-- /generated -->
