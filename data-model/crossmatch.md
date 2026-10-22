# Crossmatch Output Schema

`mosaics-crossmatch run` joins a table of positions against one or more surveys. What it writes is the joined table, a run account, a report and, when a task failed, a failures file. Every file is named after the output’s stem, so two runs in one folder do not overwrite each other.

## 1. The joined table (`-o` / `--output`)

Only `(key, ra, dec)` is sent to an archive. When the tasks finish, each survey’s columns are joined back onto the **whole input table** in memory and the result is written to `--output` (Parquet, FITS, VOTable, CSV, …):

1. **Every input column**, in its original order and type. Input rows keep their place: a row that was not matched, had no usable position, or whose task failed is still there.
2. **For each survey, in the order they were matched**, the columns its definition names, then two the run adds:
   - `<prefix>_<alias>` for each column of the survey’s `column_mapping` (or `--columns`), where the prefix is the survey’s own (`wise_`, `lsstdp2_`, `euclidq1p_`), `--prefix`’s, or none (`--no-prefix`, and an ad hoc source given no prefix).
   - `<prefix>_dist_arcsec`: the great-circle separation to the counterpart, in arcseconds (`dist_arcsec` with no prefix).
   - `<prefix>_nmatch`: how many counterparts lay inside the radius before the nearest was kept (`nmatch` with no prefix).
3. **Join semantics (`--join`):**
   - `left` (default): keeps every input row, with a survey’s columns empty where it found nothing.
   - `inner`: keeps the rows matched in every survey.
4. **Multiplicity (`--all-matches`, one survey only):** by default the nearest counterpart is kept per input row (ties broken by the counterpart’s position, so the answer does not depend on the order responses arrived in); with `--all-matches` every pair inside the radius is kept, one output row per pair.
5. **Field of view (`--fov SURVEY`):** one more column at the end, `fov` (`float32`, arcseconds): the edge of the square cutout that frames the galaxy, computed from that survey’s shape and `r` magnitude columns (see [`mosaics-fov`](../cli/fov.md)). It is empty where the survey found no counterpart, where the counterpart has neither a usable ellipticity nor a half-light radius, and where its size comes out as zero; the row is kept. With `--hle` the value is the half-light-ellipse method’s alone, and a counterpart with a radius but no ellipticity gets none. A FITS or VOTable output carries its unit. See [Field of view](../concepts/field-of-view.md).

Two surveys, or a survey and the input, may not claim the same output name: that is refused before any request is sent. Integer columns that gain empty values are written as nullable integers, so a 19-digit identifier is never rounded through a float. A FITS or VOTable output carries each remote column’s unit, content label and description as the archive wrote them.

## 2. The run account (`<output-stem>.crossmatch.yaml`)

The complete machine-readable summary of the run, and the only thing the report reads:

| Key                            | Content                                                                                                                                                                                                                                                                                                                                                                                                                               |
|--------------------------------|---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `schema`                       | `mosaics-crossmatch/1`                                                                                                                                                                                                                                                                                                                                                                                                                |
| `config`                       | the job and survey files in play, every argument with where its value came from (default, flag or a file), the definitions after the overrides, the flags in force, and which environment variable each credential came from (never its value)                                                                                                                                                                                        |
| `input`                        | rows, columns, the position columns used, and the rows without a usable position                                                                                                                                                                                                                                                                                                                                                      |
| `surveys.<label>`              | per survey: adapter and strategy, service and table, prefix, the radius with its origin (library, override or flag) and the note it rests on, the columns requested and returned with the archive’s description of each, tasks planned/done/cached/failed, rows uploaded/matched/failed, pairs, the separation tally, the multiplicity tally, the footprint filter’s counts and sky maps, errors, time, requests and the rate limiter |
| `fov`                          | only with `--fov`: the survey it came from, the column added, the columns it read, the `method` (`ellipse-or-circle` or `half-light-ellipse`), and how many rows took each form: `ellipse`, `circle`, `half-light ellipse` and `none`                                                                                                                                                                                                 |
| `counts`, `output`, `failures` | rows in and out, the columns added, the failed tasks in total                                                                                                                                                                                                                                                                                                                                                                         |
| `tables`                       | a column profile and sky map of the input and of the output                                                                                                                                                                                                                                                                                                                                                                           |
| `http`, `timing`, `provenance` | requests per service, wall clock, and the software version, revision and host                                                                                                                                                                                                                                                                                                                                                         |

## 3. The report (`<output-stem>.report/`)

- `crossmatch.scireport.zip`: sealed `scireport` bundle built with template `mosaics-crossmatch@1`.
- `md/`: chapter-split Markdown (`index.md`, `summary.md`, `overview.md`, `input.md`, one `survey-<label>.md` per survey, `output.md`, `network.md`, `conclusion.md`, `appendix.md`).
- `pdf/report.pdf`: rendered PDF report (unless `--no-report-pdf` was passed).

## 4. The failures file (`<output-stem>.failures.parquet`)

Written whenever a task still failed after its retries. One row per failed task: `survey`, `task`, `rows` (how many input rows it owned), `keys` (their row positions in the input, from 0) and `error`. Running the same command again repeats only the failed tasks, because finished ones are cached under `.mosaics-crossmatch/<output-stem>/` beside the output.
