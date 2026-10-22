# `mosaics-crossmatch` — Match a Table Against Surveys

`mosaics-crossmatch` (`uv run -m mosaics.crossmatch.cli`) matches a table of positions against one or more surveys and writes the table back with each survey’s columns joined on: the nearest counterpart within the survey’s own radius, its separation, and how many counterparts there were. It works from a job file, from survey files named on the command line, or from flags alone.

## Subcommands

- `run`: Match the input table against every survey of the roster (or one ad hoc source), write the joined table, a run account (`<stem>.crossmatch.yaml`), a report (`<stem>.report/`) and, when a task failed, `<stem>.failures.parquet`.
- `moc`: Build a survey’s footprint as a Multi-Order Coverage map from its footprint table, for `run --filter-moc`.
- `jobs`, `kill`, `del`: List, interrupt and delete the server-side asynchronous TAP jobs a survey’s service holds for your credential.

## Examples

```console
$ uv run -m mosaics.crossmatch.cli run --survey surveys/wise surveys/gaia3 -i test/objects_sample_1k.csv -o matched.parquet
$ uv run -m mosaics.crossmatch.cli run -c runs/gissel.yaml
$ uv run -m mosaics.crossmatch.cli run --survey extensions/ls11-datalab -i galaxies.parquet -o galaxies_fov.parquet --fov ls11
$ uv run -m mosaics.crossmatch.cli run -i targets.parquet -o out.parquet --adapter lsst --param release=dp2 --table dp2.Object --radius '1.5 arcsec' --columns objectId coord_ra coord_dec '[ugrizy]_cModelMag'
$ uv run -m mosaics.crossmatch.cli moc --survey surveys/lsstdp2 -o lsstdp2-moc.fits
$ uv run -m mosaics.crossmatch.cli jobs --survey surveys/lsstdp2
```

## What a run does

- **One radius per survey.** The radius is a property of the survey’s depth and astrometry, so each survey file carries its own with the note saying what it rests on (see the `crossmatch` section of the configuration reference). `--radius` overrides it for one survey; a job overrides it with `overrides:`.
- **A chain of left joins.** Every input row keeps its place. Each survey adds its columns behind its prefix (`wise_mag_w1`), then `<prefix>_dist_arcsec` and `<prefix>_nmatch`. `--join inner` keeps the rows matched in every survey; `--all-matches` keeps every pair (one survey only).
- **Only positions leave the machine.** The archive is sent `(key, ra, dec)`; the survey’s columns are joined back here. Rows with a missing or non-finite position are never sent and come back unmatched.
- **Two strategies.** A survey’s `crossmatch.strategy` is `upload` (the default: positions are sent to the archive) or `extract` (for a small catalogue, or a selection that joins tables and cannot be an upload join: the survey’s rows over each occupied HEALPix cell, plus the cells touching it, are fetched once and matched here). An extract that reaches the service’s row cap fails that cell and names the cap; there is no silent fallback to uploading. A spectroscopic survey is matched through its `target_selection` query, so its quality cuts apply and its columns are the query’s aliases.
- **Failures are values.** A task that still fails after its retries leaves its rows empty for that survey, is listed in the failures file and marks the survey incomplete in the report; the other tasks and surveys go on. `--strict` makes it fatal instead. Finished tasks are cached beside the output, so running the same command again repeats only what is missing.
- **Columns.** A survey’s `column_mapping` names the columns and their aliases (patterns allowed). `--columns` replaces it for one survey with remote names, and `--prefix` / `--no-prefix` change the prefix. With no survey and no `--prefix` an ad hoc source is not prefixed.
- **Field of view.** `--fov SURVEY` adds a `fov` column after the join: the edge, in arcseconds, of the square cutout that frames the galaxy, computed from that survey’s `shape_r`, `shape_e1`, `shape_e2` and `mag_r` columns (or `flux_r` with `mw_transmission_r`, under its prefix, or bare when the run has no prefix). The survey must be one of those matched and must add those columns; otherwise the run is refused before any request. `--hle` computes it with the half-light-ellipse method alone instead of the ellipse form with a circle fallback. A row with no usable size keeps an empty `fov`. The same computation is available on any table as [`mosaics-fov`](fov.md), and [Field of view](../concepts/field-of-view.md) explains it.
- **Credentials** are never read from YAML: `--lsst-token` or `LSST_TOKEN` for Rubin, `DATALAB_USER` and `DATALAB_PASSWORD` for the `adl` adapter of Astro Data Lab. The shipped `wise`, `gaia3` and `ukidss11` files use the `adl-healpix` adapter, which needs no login (see [Data Lab without a login](../concepts/datalab-healpix.md)); `--dry-run` prints the queries it would send. To match `ls11` through Data Lab, name `extensions/ls11-datalab` as the survey.
- **More.** [Crossmatching a table of positions](../concepts/crossmatch.md) explains the survey section, the adapters and the failure policy.

## Command-line reference

<!-- generated:cli-crossmatch -->

**Entry points:** `mosaics-crossmatch` or `uv run -m mosaics.crossmatch.cli`

### Top-level options

```text
mosaics-crossmatch [-h] {run,moc,jobs,kill,del} ...
```

### `mosaics-crossmatch run`

Match a table against surveys.

```text
mosaics-crossmatch run [-h] [--config FILE] [--survey REF [REF ...]]
                              [--exclude-survey REF [REF ...]] [--input INPUT]
                              [--output OUTPUT] [--ra-col RA_COL]
                              [--dec-col DEC_COL] [--overwrite]
                              [--join {left,inner}] [--all-matches]
                              [--radius ANGLE]
                              [--columns NAME_OR_GLOB [NAME_OR_GLOB ...]]
                              [--prefix P] [--no-prefix] [--filter-moc]
                              [--fov SURVEY] [--hle] [--adapter NAME]
                              [--table TABLE] [--service URL]
                              [--remote-ra COL] [--remote-dec COL]
                              [--param KEY=VALUE] [--name LABEL]
                              [--moc MOC.fits] [--chunk-mib CHUNK_MIB]
                              [--use-async] [--workers WORKERS]
                              [--limit-share LIMIT_SHARE]
                              [--lsst-token LSST_TOKEN]
                              [--cache-dir CACHE_DIR] [--overwrite-cache]
                              [--files-dir FILES_DIR] [--no-download]
                              [--strict] [--dry-run] [--no-report-pdf]
                              [--proxy PROXY]
                              [--max-per-origin MAX_PER_ORIGIN]
                              [--retries RETRIES] [--timeout TIMEOUT]
                              [--proxy-stats-url URL]
                              [--log-level {DEBUG,INFO,WARNING,ERROR,CRITICAL}]
```

| Option / Argument                   | Default   | Choices                                         | Description                                                                                                                                                                                                                                                                                                                                 |
|-------------------------------------|-----------|-------------------------------------------------|---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `--config`, `-c` `FILE`             | `None`    | —                                               | A job: an entrypoint whose keys are the options of this command, plus `surveys:`, `exclude:`, `overrides:` and `extends:`. Required unless –survey or an ad hoc source is given.                                                                                                                                                            |
| `--survey` `REF`                    | `None`    | —                                               | Survey files to run on, replacing the entrypoint’s `surveys:`: library names or paths, `.yaml` optional, globs allowed (`--survey surveys/ls11 surveys/wise`). The entrypoint’s `overrides:` still apply.                                                                                                                                   |
| `--exclude-survey` `REF`            | `None`    | —                                               | Survey files to remove from the run, same syntax as –survey.                                                                                                                                                                                                                                                                                |
| `--input`, `-i` `INPUT`             | `None`    | —                                               | The table of positions, in any format read_table opens. Required, here or in the job.                                                                                                                                                                                                                                                       |
| `--output`, `-o` `OUTPUT`           | `None`    | —                                               | Where to write the joined table (any format write_table writes); the account and report land beside it, named after its stem. Required, here or in the job.                                                                                                                                                                                 |
| `--ra-col` `RA_COL`                 | `None`    | —                                               | RA column of –input. Inferred from the column list when omitted.                                                                                                                                                                                                                                                                            |
| `--dec-col` `DEC_COL`               | `None`    | —                                               | Dec column of –input. Inferred when omitted.                                                                                                                                                                                                                                                                                                |
| `--overwrite`                       | `false`   | —                                               | Replace an existing output, account and report. Never writes over the input.                                                                                                                                                                                                                                                                |
| `--join` `JOIN`                     | `left`    | `left`, `inner`                                 | left keeps every input row; inner keeps the rows matched in every survey. Default: left.                                                                                                                                                                                                                                                    |
| `--all-matches`                     | `false`   | —                                               | Keep every pair inside the radius, not the nearest per row. One survey only.                                                                                                                                                                                                                                                                |
| `--radius` `ANGLE`                  | `None`    | —                                               | One source only: the match radius, e.g. ‘1.5 arcsec’ (a bare number is arcseconds). Overrides the survey’s; required for an ad hoc source.                                                                                                                                                                                                  |
| `--columns` `NAME_OR_GLOB`          | `None`    | —                                               | One source only: remote column names (globs allowed) that replace the survey’s column_mapping. The alias is the remote name, behind the prefix.                                                                                                                                                                                             |
| `--prefix` `P`                      | `None`    | —                                               | One source only: the prefix of its columns instead of the catalog’s.                                                                                                                                                                                                                                                                        |
| `--no-prefix`                       | `false`   | —                                               | Do not prefix the survey columns (collisions between surveys are then refused).                                                                                                                                                                                                                                                             |
| `--filter-moc`                      | `false`   | —                                               | Leave rows outside a survey’s footprint (crossmatch.moc) out of its upload.                                                                                                                                                                                                                                                                 |
| `--fov` `SURVEY`                    | `None`    | —                                               | Add a `fov` column: the edge, in arcseconds, of the cutout that frames the galaxy, computed from this survey’s shape (shape_r, shape_e1, shape_e2) and r magnitude (mag_r, or flux_r with mw_transmission_r) columns. The survey must be one of those matched and must add them; refused before any request otherwise.                      |
| `--hle`                             | `false`   | —                                               | With –fov: use only the half-light-ellipse method (the half-light radius times the circle correction factor of the r magnitude is the semi-major axis of the ellipse that shape_e1 and shape_e2 describe, axis ratio (1-e)/(1+e); the larger side of its bounding box is the edge) in place of the ellipse form with a circle fallback.     |
| `--adapter` `NAME`                  | `None`    | —                                               | The catalog adapter: adl, adl-healpix, crossid, local, lsdb, lsst, nersc, tap.                                                                                                                                                                                                                                                              |
| `--table` `TABLE`                   | `None`    | —                                               | The remote table (the HATS path for lsdb).                                                                                                                                                                                                                                                                                                  |
| `--service` `URL`                   | `None`    | —                                               | The service endpoint, for tap and lsst.                                                                                                                                                                                                                                                                                                     |
| `--remote-ra` `COL`                 | `None`    | —                                               | The remote right ascension column; found from its UCD or name when omitted.                                                                                                                                                                                                                                                                 |
| `--remote-dec` `COL`                | `None`    | —                                               | The remote declination column; given together with –remote-ra.                                                                                                                                                                                                                                                                              |
| `--param` `KEY=VALUE`               | `None`    | —                                               | Any other adapter_params key of the adapter, e.g. release=dp2. Repeatable.                                                                                                                                                                                                                                                                  |
| `--name` `LABEL`                    | `None`    | —                                               | What the source is called in the account and the report. Default: the table name.                                                                                                                                                                                                                                                           |
| `--moc` `MOC.fits`                  | `None`    | —                                               | The source’s footprint, for –filter-moc.                                                                                                                                                                                                                                                                                                    |
| `--chunk-mib` `CHUNK_MIB`           | `None`    | —                                               | Byte budget of one inline upload, in MiB. Default: the survey’s chunk_mib, else 28.                                                                                                                                                                                                                                                         |
| `--use-async`                       | `false`   | —                                               | Run every task as a server-side job instead of deciding from its measured size.                                                                                                                                                                                                                                                             |
| `--workers`, `-j` `WORKERS`         | `4`       | —                                               | Tasks in flight per survey; surveys run one after another. Default: 4.                                                                                                                                                                                                                                                                      |
| `--limit-share` `LIMIT_SHARE`       | `0.85`    | —                                               | The share of an archive’s published limits this run claims. Default: 0.85.                                                                                                                                                                                                                                                                  |
| `--lsst-token` `LSST_TOKEN`         | `None`    | —                                               | Rubin RSP access token; the environment (LSST_TOKEN) otherwise. Not an entrypoint key.                                                                                                                                                                                                                                                      |
| `--cache-dir` `CACHE_DIR`           | `None`    | —                                               | Where task results are kept. Default: .mosaics-crossmatch/<output-stem>/ beside the output, kept so a rerun costs no request.                                                                                                                                                                                                               |
| `--overwrite-cache`                 | `false`   | —                                               | Ignore cached task results.                                                                                                                                                                                                                                                                                                                 |
| `--files-dir` `FILES_DIR`           | `None`    | —                                               | Where a local catalog’s file is downloaded once. Default: $MOSAICS_FILES_DIR, else ~/.cache/mosaics/files.                                                                                                                                                                                                                                  |
| `--no-download`                     | `false`   | —                                               | Refuse to download a catalog file; fail if it is missing.                                                                                                                                                                                                                                                                                   |
| `--strict`                          | `false`   | —                                               | A task that still fails after its retries fails the run (exit 1, no table written). By default the run goes on and lists the failure in <stem>.failures.parquet.                                                                                                                                                                            |
| `--dry-run`                         | `false`   | —                                               | Print per survey the adapter, radius and columns (patterns unexpanded) and the effective configuration in full; send nothing.                                                                                                                                                                                                               |
| `--no-report-pdf`                   | `false`   | —                                               | Skip the PDF; the bundle and the Markdown are still written.                                                                                                                                                                                                                                                                                |
| `--proxy` `PROXY`                   | `None`    | —                                               | Proxy URL, e.g. http://127.0.0.1:8888 to route through a mosaics proxy pool. HTTPS_PROXY is honoured when this is omitted.                                                                                                                                                                                                                  |
| `--max-per-origin` `MAX_PER_ORIGIN` | `3`       | —                                               | In-flight requests per host per source IP, which is what an archive actually throttles on. Behind a proxy pool the real ceiling is this times the number of connected workers, read from –proxy-stats-url. Default: 3.                                                                                                                      |
| `--retries` `RETRIES`               | `5`       | —                                               | Attempts per request. Only transient failures consume attempts; a 404 fails immediately. Default: 5.                                                                                                                                                                                                                                        |
| `--timeout` `TIMEOUT`               | `60.0`    | —                                               | Per-request timeout in seconds. Default: 60.0.                                                                                                                                                                                                                                                                                              |
| `--proxy-stats-url` `URL`           | `None`    | —                                               | URL of a mosaics proxy stats endpoint, e.g. http://127.0.0.1:8889/stats. Read once at startup to learn how many workers are connected, which is what turns –max-per-origin into the real per-host ceiling, and polled per batch for the telemetry panel when –wandb is on. A proxy that cannot be reached is treated as a single source IP. |
| `--log-level` `LOG_LEVEL`           | `INFO`    | `DEBUG`, `INFO`, `WARNING`, `ERROR`, `CRITICAL` | Logging verbosity. Default: INFO.                                                                                                                                                                                                                                                                                                           |

### `mosaics-crossmatch moc`

Build a survey’s footprint MOC.

```text
mosaics-crossmatch moc [-h] [--config FILE] [--survey REF]
                              [--exclude-survey REF [REF ...]]
                              [--adapter NAME] [--table TABLE] [--service URL]
                              [--remote-ra COL] [--remote-dec COL]
                              [--param KEY=VALUE] [--name LABEL]
                              [--footprint-table FOOTPRINT_TABLE]
                              [--max-depth MAX_DEPTH] [--use-async] --output
                              OUTPUT [--overwrite] [--limit-share LIMIT_SHARE]
                              [--lsst-token LSST_TOKEN]
                              [--log-level {DEBUG,INFO,WARNING,ERROR,CRITICAL}]
```

| Option / Argument                     | Default    | Choices                                         | Description                                                                                                                                                                            |
|---------------------------------------|------------|-------------------------------------------------|----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `--config`, `-c` `FILE`               | `None`     | —                                               | A job: an entrypoint whose keys are the options of this command, plus `surveys:`, `exclude:`, `overrides:` and `extends:`. Exactly one survey.                                         |
| `--survey` `REF`                      | `None`     | —                                               | The survey file to work on, replacing the entrypoint’s `surveys:`: a library name or a path, `.yaml` optional (`--survey surveys/lsstdp2`). The entrypoint’s `overrides:` still apply. |
| `--exclude-survey` `REF`              | `None`     | —                                               | Survey files to remove from the run, same syntax as –survey.                                                                                                                           |
| `--adapter` `NAME`                    | `None`     | —                                               | The catalog adapter: adl, adl-healpix, crossid, local, lsdb, lsst, nersc, tap.                                                                                                         |
| `--table` `TABLE`                     | `None`     | —                                               | The remote table (the HATS path for lsdb).                                                                                                                                             |
| `--service` `URL`                     | `None`     | —                                               | The service endpoint, for tap and lsst.                                                                                                                                                |
| `--remote-ra` `COL`                   | `None`     | —                                               | The remote right ascension column; found from its UCD or name when omitted.                                                                                                            |
| `--remote-dec` `COL`                  | `None`     | —                                               | The remote declination column; given together with –remote-ra.                                                                                                                         |
| `--param` `KEY=VALUE`                 | `None`     | —                                               | Any other adapter_params key of the adapter, e.g. release=dp2. Repeatable.                                                                                                             |
| `--name` `LABEL`                      | `None`     | —                                               | What the source is called in the account and the report. Default: the table name.                                                                                                      |
| `--footprint-table` `FOOTPRINT_TABLE` | `None`     | —                                               | The table of s_region polygons. Default: the survey’s footprint_table.                                                                                                                 |
| `--max-depth` `MAX_DEPTH`             | `10`       | —                                               | The MOC’s HEALPix resolution. Higher is finer and slower to build. Default: 10.                                                                                                        |
| `--use-async`                         | `false`    | —                                               | Run the s_region query as a server-side job.                                                                                                                                           |
| `--output`, `-o` `OUTPUT`             | *required* | —                                               | Where to write the MOC FITS file.                                                                                                                                                      |
| `--overwrite`                         | `false`    | —                                               | Replace an existing –output.                                                                                                                                                           |
| `--limit-share` `LIMIT_SHARE`         | `0.85`     | —                                               | The share of the archive’s published limits this run claims. Default: 0.85.                                                                                                            |
| `--lsst-token` `LSST_TOKEN`           | `None`     | —                                               | Rubin RSP access token; the environment otherwise.                                                                                                                                     |
| `--log-level` `LOG_LEVEL`             | `INFO`     | `DEBUG`, `INFO`, `WARNING`, `ERROR`, `CRITICAL` | Logging verbosity. Default: INFO.                                                                                                                                                      |

### `mosaics-crossmatch jobs`

List the async TAP jobs a survey’s service holds.

```text
mosaics-crossmatch jobs [-h] [--config FILE] [--survey REF]
                               [--exclude-survey REF [REF ...]]
                               [--adapter NAME] [--table TABLE]
                               [--service URL] [--remote-ra COL]
                               [--remote-dec COL] [--param KEY=VALUE]
                               [--name LABEL] [--limit-share LIMIT_SHARE]
                               [--lsst-token LSST_TOKEN]
                               [--log-level {DEBUG,INFO,WARNING,ERROR,CRITICAL}]
```

| Option / Argument             | Default   | Choices                                         | Description                                                                                                                                                                            |
|-------------------------------|-----------|-------------------------------------------------|----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `--config`, `-c` `FILE`       | `None`    | —                                               | A job: an entrypoint whose keys are the options of this command, plus `surveys:`, `exclude:`, `overrides:` and `extends:`. Exactly one survey, whose TAP service holds the jobs.       |
| `--survey` `REF`              | `None`    | —                                               | The survey file to work on, replacing the entrypoint’s `surveys:`: a library name or a path, `.yaml` optional (`--survey surveys/lsstdp2`). The entrypoint’s `overrides:` still apply. |
| `--exclude-survey` `REF`      | `None`    | —                                               | Survey files to remove from the run, same syntax as –survey.                                                                                                                           |
| `--adapter` `NAME`            | `None`    | —                                               | The catalog adapter: adl, adl-healpix, crossid, local, lsdb, lsst, nersc, tap.                                                                                                         |
| `--table` `TABLE`             | `None`    | —                                               | The remote table (the HATS path for lsdb).                                                                                                                                             |
| `--service` `URL`             | `None`    | —                                               | The service endpoint, for tap and lsst.                                                                                                                                                |
| `--remote-ra` `COL`           | `None`    | —                                               | The remote right ascension column; found from its UCD or name when omitted.                                                                                                            |
| `--remote-dec` `COL`          | `None`    | —                                               | The remote declination column; given together with –remote-ra.                                                                                                                         |
| `--param` `KEY=VALUE`         | `None`    | —                                               | Any other adapter_params key of the adapter, e.g. release=dp2. Repeatable.                                                                                                             |
| `--name` `LABEL`              | `None`    | —                                               | What the source is called in the account and the report. Default: the table name.                                                                                                      |
| `--limit-share` `LIMIT_SHARE` | `0.85`    | —                                               | The share of the archive’s published limits this run claims. Default: 0.85.                                                                                                            |
| `--lsst-token` `LSST_TOKEN`   | `None`    | —                                               | Rubin RSP access token; it must be the one the run that started the jobs used. The environment otherwise.                                                                              |
| `--log-level` `LOG_LEVEL`     | `INFO`    | `DEBUG`, `INFO`, `WARNING`, `ERROR`, `CRITICAL` | Logging verbosity. Default: INFO.                                                                                                                                                      |

### `mosaics-crossmatch kill`

Interrupt async TAP jobs.

```text
mosaics-crossmatch kill [-h] [--config FILE] [--survey REF]
                               [--exclude-survey REF [REF ...]]
                               [--adapter NAME] [--table TABLE]
                               [--service URL] [--remote-ra COL]
                               [--remote-dec COL] [--param KEY=VALUE]
                               [--name LABEL] [--limit-share LIMIT_SHARE]
                               [--lsst-token LSST_TOKEN]
                               [--log-level {DEBUG,INFO,WARNING,ERROR,CRITICAL}]
                               [JOB_ID ...]
```

| Option / Argument             | Default    | Choices                                         | Description                                                                                                                                                                            |
|-------------------------------|------------|-------------------------------------------------|----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `--config`, `-c` `FILE`       | `None`     | —                                               | A job: an entrypoint whose keys are the options of this command, plus `surveys:`, `exclude:`, `overrides:` and `extends:`. Exactly one survey, whose TAP service holds the jobs.       |
| `--survey` `REF`              | `None`     | —                                               | The survey file to work on, replacing the entrypoint’s `surveys:`: a library name or a path, `.yaml` optional (`--survey surveys/lsstdp2`). The entrypoint’s `overrides:` still apply. |
| `--exclude-survey` `REF`      | `None`     | —                                               | Survey files to remove from the run, same syntax as –survey.                                                                                                                           |
| `--adapter` `NAME`            | `None`     | —                                               | The catalog adapter: adl, adl-healpix, crossid, local, lsdb, lsst, nersc, tap.                                                                                                         |
| `--table` `TABLE`             | `None`     | —                                               | The remote table (the HATS path for lsdb).                                                                                                                                             |
| `--service` `URL`             | `None`     | —                                               | The service endpoint, for tap and lsst.                                                                                                                                                |
| `--remote-ra` `COL`           | `None`     | —                                               | The remote right ascension column; found from its UCD or name when omitted.                                                                                                            |
| `--remote-dec` `COL`          | `None`     | —                                               | The remote declination column; given together with –remote-ra.                                                                                                                         |
| `--param` `KEY=VALUE`         | `None`     | —                                               | Any other adapter_params key of the adapter, e.g. release=dp2. Repeatable.                                                                                                             |
| `--name` `LABEL`              | `None`     | —                                               | What the source is called in the account and the report. Default: the table name.                                                                                                      |
| `job_ids` `JOB_ID`            | *required* | —                                               | The job ids. Omit for every job the operation applies to.                                                                                                                              |
| `--limit-share` `LIMIT_SHARE` | `0.85`     | —                                               | The share of the archive’s published limits this run claims. Default: 0.85.                                                                                                            |
| `--lsst-token` `LSST_TOKEN`   | `None`     | —                                               | Rubin RSP access token; it must be the one the run that started the jobs used. The environment otherwise.                                                                              |
| `--log-level` `LOG_LEVEL`     | `INFO`     | `DEBUG`, `INFO`, `WARNING`, `ERROR`, `CRITICAL` | Logging verbosity. Default: INFO.                                                                                                                                                      |

### `mosaics-crossmatch del`

Delete async TAP jobs.

```text
mosaics-crossmatch del [-h] [--config FILE] [--survey REF]
                              [--exclude-survey REF [REF ...]]
                              [--adapter NAME] [--table TABLE] [--service URL]
                              [--remote-ra COL] [--remote-dec COL]
                              [--param KEY=VALUE] [--name LABEL]
                              [--limit-share LIMIT_SHARE]
                              [--lsst-token LSST_TOKEN]
                              [--log-level {DEBUG,INFO,WARNING,ERROR,CRITICAL}]
                              [JOB_ID ...]
```

| Option / Argument             | Default    | Choices                                         | Description                                                                                                                                                                            |
|-------------------------------|------------|-------------------------------------------------|----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `--config`, `-c` `FILE`       | `None`     | —                                               | A job: an entrypoint whose keys are the options of this command, plus `surveys:`, `exclude:`, `overrides:` and `extends:`. Exactly one survey, whose TAP service holds the jobs.       |
| `--survey` `REF`              | `None`     | —                                               | The survey file to work on, replacing the entrypoint’s `surveys:`: a library name or a path, `.yaml` optional (`--survey surveys/lsstdp2`). The entrypoint’s `overrides:` still apply. |
| `--exclude-survey` `REF`      | `None`     | —                                               | Survey files to remove from the run, same syntax as –survey.                                                                                                                           |
| `--adapter` `NAME`            | `None`     | —                                               | The catalog adapter: adl, adl-healpix, crossid, local, lsdb, lsst, nersc, tap.                                                                                                         |
| `--table` `TABLE`             | `None`     | —                                               | The remote table (the HATS path for lsdb).                                                                                                                                             |
| `--service` `URL`             | `None`     | —                                               | The service endpoint, for tap and lsst.                                                                                                                                                |
| `--remote-ra` `COL`           | `None`     | —                                               | The remote right ascension column; found from its UCD or name when omitted.                                                                                                            |
| `--remote-dec` `COL`          | `None`     | —                                               | The remote declination column; given together with –remote-ra.                                                                                                                         |
| `--param` `KEY=VALUE`         | `None`     | —                                               | Any other adapter_params key of the adapter, e.g. release=dp2. Repeatable.                                                                                                             |
| `--name` `LABEL`              | `None`     | —                                               | What the source is called in the account and the report. Default: the table name.                                                                                                      |
| `job_ids` `JOB_ID`            | *required* | —                                               | The job ids. Omit for every job the operation applies to.                                                                                                                              |
| `--limit-share` `LIMIT_SHARE` | `0.85`     | —                                               | The share of the archive’s published limits this run claims. Default: 0.85.                                                                                                            |
| `--lsst-token` `LSST_TOKEN`   | `None`     | —                                               | Rubin RSP access token; it must be the one the run that started the jobs used. The environment otherwise.                                                                              |
| `--log-level` `LOG_LEVEL`     | `INFO`     | `DEBUG`, `INFO`, `WARNING`, `ERROR`, `CRITICAL` | Logging verbosity. Default: INFO.                                                                                                                                                      |
<!-- /generated -->
