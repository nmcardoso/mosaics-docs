# Installation & Credentials

> [!IMPORTANT]
> Access to the private `nmcardoso/mosaics` GitHub repository is required to clone and install MOSAICS. Always run MOSAICS from a repository checkout so that the survey definitions in `defs/` and SQL queries in `defs/surveys/*.sql` resolve relative to the checkout root.

## 1. Clone and install with `uv`

MOSAICS requires Python $\ge 3.12$ and is managed with [uv](https://docs.astral.sh/uv/):

```bash
git clone git@github.com:nmcardoso/mosaics.git
cd mosaics
uv sync
```

`uv sync` installs `mosaics` in editable mode inside `.venv` along with all runtime dependencies (`pyarrow`, `pylance`, `duckdb`, `lsdb`, `astropy`, `reproject`, `scireport[astro,pdf]`, etc.) and registers nine console scripts (`mosaics-cutout`, `mosaics-query`, `mosaics-spectrum`, `mosaics-targets`, `mosaics-crossmatch`, `mosaics-fov`, `mosaics-hipscat`, `mosaics-proxy`, `mosaics-report`).

Confirm the installation:

```console
$ uv run -m mosaics.cli.cutout --help
$ uv run -m mosaics.cli.targets --help
```

## 2. System packages for PDF reports

Every pipeline report (`mosaics-targets`, `mosaics-crossmatch`, `mosaics-report`) always writes a sealed `.scireport.zip` bundle and chapter-split Markdown files (`report/md/*.md`). Rendering the print-ready PDF (`report/pdf/report.pdf`) uses WeasyPrint, which requires the system Pango and HarfBuzz shared libraries:

```bash
# Ubuntu / Debian
sudo apt-get update && sudo apt-get install -y libpango-1.0-0 libpangoft2-1.0-0 libharfbuzz-subset0
```

If Pango is unavailable on a headless cluster node, pass `--no-report-pdf` (or `--no-pdf` on `mosaics-report`) to produce the `.scireport.zip` bundle and Markdown chapters without invoking WeasyPrint.

## 3. Archive credentials

Most astronomical archives queried by MOSAICS (CDS VizieR, CDS HiPS/hips2fits, NERSC Legacy Survey sweeps and bricks, STScI MAST, LAMOST, WFAU SIA/VSA, LINCC/CDS HATS servers, SDSS SAS) answer anonymously. Three services support or require account credentials, read from environment variables or a `.env` file in the checkout root:

| Archive / Provider                     | Environment variables                                                                    | When needed                                                                                                                                                     |
|----------------------------------------|------------------------------------------------------------------------------------------|-----------------------------------------------------------------------------------------------------------------------------------------------------------------|
| **Rubin Science Platform (`lsst`)**    | `LSST_TOKEN` (or `LSST_PASSWORD` / `LSST_PASS`), overridable via `--lsst-token`          | Required for the Rubin surveys (`lsstdp1`, `lsstdp2`) in every `mosaics-crossmatch` command (`run`, `moc`, `jobs`, `kill`, `del`). Never read from a YAML file. |
| **NSF NOIRLab Astro Data Lab (`adl`)** | `DATALAB_USERNAME`, `DATALAB_PASSWORD` (overridable via `--username` / `--password`)     | Optional for public tables (`gaia_dr3.gaia_source`, `allwise.source`, `ukidss_dr11plus.lassource`); required for proprietary access or extended MyDB storage.   |
| **SciServer CasJobs (`sciserver`)**    | `SCISERVER_USERNAME`, `SCISERVER_PASSWORD` (overridable via `--username` / `--password`) | Required when querying SciServer CasJobs (`--provider sciserver`) or cleaning up MyDB tables (`mosaics-query sweep`).                                           |
| **Weights & Biases (`--wandb`)**       | `WANDB_API_KEY`                                                                          | Optional; used only when `--wandb` telemetry is enabled on `cutout`, `spectrum`, or `targets`.                                                                  |

## 4. Download cache directories

Field-based cutout backends (`hips --local`, `nersc`, `sia`, `vsa`) and the `nersc` catalogue provider cache downloaded tiles, bricks, and sweep slices on disk under `$XDG_CACHE_HOME/mosaics` (default `~/.cache/mosaics`), capped by `--cache-size-gb` (default `50.0` GB LRU). You can point individual caches at fast scratch storage with environment variables:

| Environment variable    | Default path               | What it stores                                                         |
|-------------------------|----------------------------|------------------------------------------------------------------------|
| `MOSAICS_HIPS_CACHE`    | `~/.cache/mosaics/hips`    | Downloaded HiPS tiles for `hips --local`.                              |
| `MOSAICS_BRICK_CACHE`   | `~/.cache/mosaics/bricks`  | Legacy Survey coadd bricks from NERSC (`cutout nersc`).                |
| `MOSAICS_FIELD_CACHE`   | `~/.cache/mosaics/fields`  | SIA and VSA survey frames before local WCS reprojection.               |
| `MOSAICS_TRACTOR_CACHE` | `~/.cache/mosaics/tractor` | Pruned NERSC sweep and tractor Parquet slices (`catalog` / `targets`). |
