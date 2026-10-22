# `mosaics-query` — Catalogue Queries & SSA Discovery

`mosaics-query` (`uv run -m mosaics.cli.query`) queries astronomical catalogue providers (`tap`, `adl`, `sciserver`, `crossid`, `local`, `nersc`) and IVOA Simple Spectral Access (`ssa`) endpoints, writing the resulting tables to Parquet, FITS, CSV, or Feather files.

## Subcommands

- `cone`: Spatial cone search (`--ra`, `--dec`, `--radius`) against a remote or local catalogue table.
- `region`: Rectangular right-ascension and declination box query (`--ra-range`, `--dec-range`).
- `download`: Bulk table download with column projection (`--columns`) and optional `--where` predicate.
- `sql`: Execute a custom ADQL/SQL query from a `.sql` file (`--path`).
- `sweep`: Clean up leftover temporary crossmatch tables (`mosaics_crossmatch_*`) in SciServer MyDB.
- `ssa`: Discover 1D spectra metadata over an IVOA Simple Spectral Access service.

## Examples

```console
$ uv run -m mosaics.cli.query cone --provider tap --service vizier --table "VII/250/2dfgrs" --columns SEQNUM RAJ2000 DEJ2000 z --ra 180.0 --dec 0.0 --radius 0.5 -o 2df_cone.parquet
$ uv run -m mosaics.cli.query region --provider adl --table gaia_dr3.gaia_source --ra-range 180.0 180.5 --dec-range -0.5 0.5 --columns source_id ra dec phot_g_mean_mag -o gaia_box.parquet
$ uv run -m mosaics.cli.query sql --provider tap --service datalab --path defs/surveys/desi1.sql -o desi_sample.parquet
```

## Command-line reference

<!-- generated:cli-query -->

**Entry points:** `mosaics-query` or `uv run -m mosaics.cli.query`

### Top-level options

```text
mosaics.cli.query [-h] {cone,region,download,sql,sweep,ssa} ...
```

### `mosaics-query cone`

Perform a cone search around a sky position.

```text
mosaics.cli.query cone [-h] --table TABLE --columns COLUMNS
                              [COLUMNS ...] --ra CENTER_RA --dec CENTER_DEC
                              --radius RADIUS [--where WHERE]
                              [--ra-col RA_COLUMN] [--dec-col DEC_COLUMN]
                              [--delta DEC_DELTA] --output OUTPUT
                              [--cache CACHE_DIR] [--workers WORKERS]
                              [--retries RETRIES] [--async]
                              [--log-level {DEBUG,INFO,WARNING,ERROR,CRITICAL}]
                              [--provider {adl,crossid,local,lsst,nersc,sciserver,tap}]
                              [--url URL]
                              [--service {datalab,dl,irsa,mast,vizier,lamost}]
                              [--username USERNAME] [--password PASSWORD]
                              [--data-release DATA_RELEASE] [--plan]
```

| Option / Argument                     | Default     | Choices                                                        | Description                                                                                                                                                                |
|---------------------------------------|-------------|----------------------------------------------------------------|----------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `--table`, `-t` `TABLE`               | *required*  | —                                                              | Remote table name.                                                                                                                                                         |
| `--columns`, `-c` `COLUMNS`           | *required*  | —                                                              | Remote table column names to retrieve (space-separated).                                                                                                                   |
| `--ra` `CENTER_RA`                    | *required*  | —                                                              | RA of the cone center (degrees).                                                                                                                                           |
| `--dec` `CENTER_DEC`                  | *required*  | —                                                              | DEC of the cone center (degrees).                                                                                                                                          |
| `--radius` `RADIUS`                   | *required*  | —                                                              | Angular radius of the cone search (degrees).                                                                                                                               |
| `--where`, `-w` `WHERE`               | `None`      | —                                                              | Additional SQL WHERE clause fragment (appended with AND).                                                                                                                  |
| `--ra-col`, `--rc` `RA_COLUMN`        | `None`      | —                                                              | RA column name; auto-inferred if omitted.                                                                                                                                  |
| `--dec-col`, `--dc` `DEC_COLUMN`      | `None`      | —                                                              | DEC column name; auto-inferred if omitted.                                                                                                                                 |
| `--delta`, `-d` `DEC_DELTA`           | `10 arcmin` | —                                                              | Declination slice size used to split the query into batches. If no unit is specified, arcmin is considered. Default is 10 arcmin.                                          |
| `--output`, `-o` `OUTPUT`             | *required*  | —                                                              | Destination file path. Format inferred from extension.                                                                                                                     |
| `--cache` `CACHE_DIR`                 | `None`      | —                                                              | Directory for intermediate parquet files. A temporary directory is created (and cleaned up) if omitted.                                                                    |
| `--workers`, `-j` `WORKERS`           | `4`         | —                                                              | Maximum number of concurrent provider jobs. Default: 4.                                                                                                                    |
| `--retries` `RETRIES`                 | `5`         | —                                                              | Per-batch retry count with exponential backoff. Default: 5.                                                                                                                |
| `--async`                             | `false`     | —                                                              | Execute each batch as a server-side async job instead of a synchronous request. Async jobs can queue on the server and take substantially longer per batch. Default: sync. |
| `--log-level` `LOG_LEVEL`             | `INFO`      | `DEBUG`, `INFO`, `WARNING`, `ERROR`, `CRITICAL`                | Logging verbosity level. Default: INFO.                                                                                                                                    |
| `--provider`, `-p` `PROVIDER`         | `tap`       | `adl`, `crossid`, `local`, `lsst`, `nersc`, `sciserver`, `tap` | Catalog provider backend to use. Default: tap.                                                                                                                             |
| `--url`, `-u` `URL`                   | `None`      | —                                                              | [tap] Base URL of the TAP service (e.g. https://tap.gaia.unige.ch/tap).                                                                                                    |
| `--service`, `-s` `SERVICE`           | `None`      | `datalab`, `dl`, `irsa`, `mast`, `vizier`, `lamost`            | [tap] Predefined TAP service to use.                                                                                                                                       |
| `--username`, `-U` `USERNAME`         | `None`      | —                                                              | [adl, sciserver] Account username. Falls back to DATALAB_USERNAME or SCISERVER_USERNAME.                                                                                   |
| `--password`, `-P` `PASSWORD`         | `None`      | —                                                              | [adl, sciserver] Account password. Falls back to DATALAB_PASSWORD or SCISERVER_PASSWORD.                                                                                   |
| `--data-release`, `-R` `DATA_RELEASE` | `DR20`      | —                                                              | [sciserver] SDSS release to query, e.g. DR20. Default: DR20.                                                                                                               |
| `--plan`                              | `false`     | —                                                              | Preview the spatial segmentation planning and exit. Displays a plot of the search region with bounding-box overlays instead of downloading data.                           |

### `mosaics-query region`

Download all rows within a rectangular sky region.

```text
mosaics.cli.query region [-h] --table TABLE --columns COLUMNS
                                [COLUMNS ...] --ra-range RA_MIN RA_MAX
                                --dec-range DEC_MIN DEC_MAX [--where WHERE]
                                [--ra-col RA_COLUMN] [--dec-col DEC_COLUMN]
                                [--delta DEC_DELTA] --output OUTPUT
                                [--cache CACHE_DIR] [--workers WORKERS]
                                [--retries RETRIES] [--async]
                                [--log-level {DEBUG,INFO,WARNING,ERROR,CRITICAL}]
                                [--provider {adl,crossid,local,lsst,nersc,sciserver,tap}]
                                [--url URL]
                                [--service {datalab,dl,irsa,mast,vizier,lamost}]
                                [--username USERNAME] [--password PASSWORD]
                                [--data-release DATA_RELEASE] [--plan]
```

| Option / Argument                     | Default     | Choices                                                        | Description                                                                                                                                                                |
|---------------------------------------|-------------|----------------------------------------------------------------|----------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `--table`, `-t` `TABLE`               | *required*  | —                                                              | Remote table name.                                                                                                                                                         |
| `--columns`, `-c` `COLUMNS`           | *required*  | —                                                              | Remote table column names to retrieve (space-separated).                                                                                                                   |
| `--ra-range` `RA_MIN RA_MAX`          | *required*  | —                                                              | Right ascension range [min max] in degrees.                                                                                                                                |
| `--dec-range` `DEC_MIN DEC_MAX`       | *required*  | —                                                              | Declination range [min max] in degrees.                                                                                                                                    |
| `--where`, `-w` `WHERE`               | `None`      | —                                                              | Additional SQL WHERE clause fragment (appended with AND).                                                                                                                  |
| `--ra-col`, `--rc` `RA_COLUMN`        | `None`      | —                                                              | RA column name; auto-inferred if omitted.                                                                                                                                  |
| `--dec-col`, `--dc` `DEC_COLUMN`      | `None`      | —                                                              | DEC column name; auto-inferred if omitted.                                                                                                                                 |
| `--delta`, `-d` `DEC_DELTA`           | `10 arcmin` | —                                                              | Declination slice size used to split the query into batches. If no unit is specified, arcmin is considered. Default is 10 arcmin.                                          |
| `--output`, `-o` `OUTPUT`             | *required*  | —                                                              | Destination file path. Format inferred from extension.                                                                                                                     |
| `--cache` `CACHE_DIR`                 | `None`      | —                                                              | Directory for intermediate parquet files. A temporary directory is created (and cleaned up) if omitted.                                                                    |
| `--workers`, `-j` `WORKERS`           | `4`         | —                                                              | Maximum number of concurrent provider jobs. Default: 4.                                                                                                                    |
| `--retries` `RETRIES`                 | `5`         | —                                                              | Per-batch retry count with exponential backoff. Default: 5.                                                                                                                |
| `--async`                             | `false`     | —                                                              | Execute each batch as a server-side async job instead of a synchronous request. Async jobs can queue on the server and take substantially longer per batch. Default: sync. |
| `--log-level` `LOG_LEVEL`             | `INFO`      | `DEBUG`, `INFO`, `WARNING`, `ERROR`, `CRITICAL`                | Logging verbosity level. Default: INFO.                                                                                                                                    |
| `--provider`, `-p` `PROVIDER`         | `tap`       | `adl`, `crossid`, `local`, `lsst`, `nersc`, `sciserver`, `tap` | Catalog provider backend to use. Default: tap.                                                                                                                             |
| `--url`, `-u` `URL`                   | `None`      | —                                                              | [tap] Base URL of the TAP service (e.g. https://tap.gaia.unige.ch/tap).                                                                                                    |
| `--service`, `-s` `SERVICE`           | `None`      | `datalab`, `dl`, `irsa`, `mast`, `vizier`, `lamost`            | [tap] Predefined TAP service to use.                                                                                                                                       |
| `--username`, `-U` `USERNAME`         | `None`      | —                                                              | [adl, sciserver] Account username. Falls back to DATALAB_USERNAME or SCISERVER_USERNAME.                                                                                   |
| `--password`, `-P` `PASSWORD`         | `None`      | —                                                              | [adl, sciserver] Account password. Falls back to DATALAB_PASSWORD or SCISERVER_PASSWORD.                                                                                   |
| `--data-release`, `-R` `DATA_RELEASE` | `DR20`      | —                                                              | [sciserver] SDSS release to query, e.g. DR20. Default: DR20.                                                                                                               |
| `--plan`                              | `false`     | —                                                              | Preview the spatial segmentation planning and exit. Displays a plot of the search region with bounding-box overlays instead of downloading data.                           |

### `mosaics-query download`

Download all rows of a remote table.

```text
mosaics.cli.query download [-h] --table TABLE --columns COLUMNS
                                  [COLUMNS ...] [--where WHERE]
                                  [--ra-col RA_COLUMN] [--dec-col DEC_COLUMN]
                                  [--delta DEC_DELTA] --output OUTPUT
                                  [--cache CACHE_DIR] [--workers WORKERS]
                                  [--retries RETRIES] [--async]
                                  [--log-level {DEBUG,INFO,WARNING,ERROR,CRITICAL}]
                                  [--provider {adl,crossid,local,lsst,nersc,sciserver,tap}]
                                  [--url URL]
                                  [--service {datalab,dl,irsa,mast,vizier,lamost}]
                                  [--username USERNAME] [--password PASSWORD]
                                  [--data-release DATA_RELEASE] [--plan]
```

| Option / Argument                     | Default     | Choices                                                        | Description                                                                                                                                                                |
|---------------------------------------|-------------|----------------------------------------------------------------|----------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `--table`, `-t` `TABLE`               | *required*  | —                                                              | Remote table name.                                                                                                                                                         |
| `--columns`, `-c` `COLUMNS`           | *required*  | —                                                              | Remote table column names to retrieve (space-separated).                                                                                                                   |
| `--where`, `-w` `WHERE`               | `None`      | —                                                              | Additional SQL WHERE clause fragment (appended with AND).                                                                                                                  |
| `--ra-col`, `--rc` `RA_COLUMN`        | `None`      | —                                                              | RA column name; auto-inferred if omitted.                                                                                                                                  |
| `--dec-col`, `--dc` `DEC_COLUMN`      | `None`      | —                                                              | DEC column name; auto-inferred if omitted.                                                                                                                                 |
| `--delta`, `-d` `DEC_DELTA`           | `10 arcmin` | —                                                              | Declination slice size used to split the query into batches. If no unit is specified, arcmin is considered. Default is 10 arcmin.                                          |
| `--output`, `-o` `OUTPUT`             | *required*  | —                                                              | Destination file path. Format inferred from extension.                                                                                                                     |
| `--cache` `CACHE_DIR`                 | `None`      | —                                                              | Directory for intermediate parquet files. A temporary directory is created (and cleaned up) if omitted.                                                                    |
| `--workers`, `-j` `WORKERS`           | `4`         | —                                                              | Maximum number of concurrent provider jobs. Default: 4.                                                                                                                    |
| `--retries` `RETRIES`                 | `5`         | —                                                              | Per-batch retry count with exponential backoff. Default: 5.                                                                                                                |
| `--async`                             | `false`     | —                                                              | Execute each batch as a server-side async job instead of a synchronous request. Async jobs can queue on the server and take substantially longer per batch. Default: sync. |
| `--log-level` `LOG_LEVEL`             | `INFO`      | `DEBUG`, `INFO`, `WARNING`, `ERROR`, `CRITICAL`                | Logging verbosity level. Default: INFO.                                                                                                                                    |
| `--provider`, `-p` `PROVIDER`         | `tap`       | `adl`, `crossid`, `local`, `lsst`, `nersc`, `sciserver`, `tap` | Catalog provider backend to use. Default: tap.                                                                                                                             |
| `--url`, `-u` `URL`                   | `None`      | —                                                              | [tap] Base URL of the TAP service (e.g. https://tap.gaia.unige.ch/tap).                                                                                                    |
| `--service`, `-s` `SERVICE`           | `None`      | `datalab`, `dl`, `irsa`, `mast`, `vizier`, `lamost`            | [tap] Predefined TAP service to use.                                                                                                                                       |
| `--username`, `-U` `USERNAME`         | `None`      | —                                                              | [adl, sciserver] Account username. Falls back to DATALAB_USERNAME or SCISERVER_USERNAME.                                                                                   |
| `--password`, `-P` `PASSWORD`         | `None`      | —                                                              | [adl, sciserver] Account password. Falls back to DATALAB_PASSWORD or SCISERVER_PASSWORD.                                                                                   |
| `--data-release`, `-R` `DATA_RELEASE` | `DR20`      | —                                                              | [sciserver] SDSS release to query, e.g. DR20. Default: DR20.                                                                                                               |
| `--plan`                              | `false`     | —                                                              | Preview the spatial segmentation planning and exit. Displays a plot of the search region with bounding-box overlays instead of downloading data.                           |

### `mosaics-query sql`

Execute a custom SQL query against the catalog provider.

```text
mosaics.cli.query sql [-h] --path PATH [--where WHERE]
                             [--ra-col RA_COLUMN] [--dec-col DEC_COLUMN]
                             [--delta DEC_DELTA] --output OUTPUT
                             [--cache CACHE_DIR] [--workers WORKERS]
                             [--retries RETRIES] [--async]
                             [--log-level {DEBUG,INFO,WARNING,ERROR,CRITICAL}]
                             [--provider {adl,crossid,local,lsst,nersc,sciserver,tap}]
                             [--url URL]
                             [--service {datalab,dl,irsa,mast,vizier,lamost}]
                             [--username USERNAME] [--password PASSWORD]
                             [--data-release DATA_RELEASE]
```

| Option / Argument                     | Default     | Choices                                                        | Description                                                                                                                                                                |
|---------------------------------------|-------------|----------------------------------------------------------------|----------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `--path` `PATH`                       | *required*  | —                                                              | The path to the SQL query to execute.                                                                                                                                      |
| `--where`, `-w` `WHERE`               | `None`      | —                                                              | Additional SQL WHERE clause fragment (appended with AND).                                                                                                                  |
| `--ra-col`, `--rc` `RA_COLUMN`        | `None`      | —                                                              | RA column name; auto-inferred if omitted.                                                                                                                                  |
| `--dec-col`, `--dc` `DEC_COLUMN`      | `None`      | —                                                              | DEC column name; auto-inferred if omitted.                                                                                                                                 |
| `--delta`, `-d` `DEC_DELTA`           | `10 arcmin` | —                                                              | Declination slice size used to split the query into batches. If no unit is specified, arcmin is considered. Default is 10 arcmin.                                          |
| `--output`, `-o` `OUTPUT`             | *required*  | —                                                              | Destination file path. Format inferred from extension.                                                                                                                     |
| `--cache` `CACHE_DIR`                 | `None`      | —                                                              | Directory for intermediate parquet files. A temporary directory is created (and cleaned up) if omitted.                                                                    |
| `--workers`, `-j` `WORKERS`           | `4`         | —                                                              | Maximum number of concurrent provider jobs. Default: 4.                                                                                                                    |
| `--retries` `RETRIES`                 | `5`         | —                                                              | Per-batch retry count with exponential backoff. Default: 5.                                                                                                                |
| `--async`                             | `false`     | —                                                              | Execute each batch as a server-side async job instead of a synchronous request. Async jobs can queue on the server and take substantially longer per batch. Default: sync. |
| `--log-level` `LOG_LEVEL`             | `INFO`      | `DEBUG`, `INFO`, `WARNING`, `ERROR`, `CRITICAL`                | Logging verbosity level. Default: INFO.                                                                                                                                    |
| `--provider`, `-p` `PROVIDER`         | `tap`       | `adl`, `crossid`, `local`, `lsst`, `nersc`, `sciserver`, `tap` | Catalog provider backend to use. Default: tap.                                                                                                                             |
| `--url`, `-u` `URL`                   | `None`      | —                                                              | [tap] Base URL of the TAP service (e.g. https://tap.gaia.unige.ch/tap).                                                                                                    |
| `--service`, `-s` `SERVICE`           | `None`      | `datalab`, `dl`, `irsa`, `mast`, `vizier`, `lamost`            | [tap] Predefined TAP service to use.                                                                                                                                       |
| `--username`, `-U` `USERNAME`         | `None`      | —                                                              | [adl, sciserver] Account username. Falls back to DATALAB_USERNAME or SCISERVER_USERNAME.                                                                                   |
| `--password`, `-P` `PASSWORD`         | `None`      | —                                                              | [adl, sciserver] Account password. Falls back to DATALAB_PASSWORD or SCISERVER_PASSWORD.                                                                                   |
| `--data-release`, `-R` `DATA_RELEASE` | `DR20`      | —                                                              | [sciserver] SDSS release to query, e.g. DR20. Default: DR20.                                                                                                               |

### `mosaics-query sweep`

Drop temporary crossmatch tables a previous run left behind.

```text
mosaics.cli.query sweep [-h]
                               [--provider {adl,crossid,local,lsst,nersc,sciserver,tap}]
                               [--url URL]
                               [--service {datalab,dl,irsa,mast,vizier,lamost}]
                               [--username USERNAME] [--password PASSWORD]
                               [--data-release DATA_RELEASE] [--dry-run]
                               [--log-level {DEBUG,INFO,WARNING,ERROR,CRITICAL}]
```

| Option / Argument                     | Default   | Choices                                                        | Description                                                                              |
|---------------------------------------|-----------|----------------------------------------------------------------|------------------------------------------------------------------------------------------|
| `--provider`, `-p` `PROVIDER`         | `tap`     | `adl`, `crossid`, `local`, `lsst`, `nersc`, `sciserver`, `tap` | Catalog provider backend to use. Default: tap.                                           |
| `--url`, `-u` `URL`                   | `None`    | —                                                              | [tap] Base URL of the TAP service (e.g. https://tap.gaia.unige.ch/tap).                  |
| `--service`, `-s` `SERVICE`           | `None`    | `datalab`, `dl`, `irsa`, `mast`, `vizier`, `lamost`            | [tap] Predefined TAP service to use.                                                     |
| `--username`, `-U` `USERNAME`         | `None`    | —                                                              | [adl, sciserver] Account username. Falls back to DATALAB_USERNAME or SCISERVER_USERNAME. |
| `--password`, `-P` `PASSWORD`         | `None`    | —                                                              | [adl, sciserver] Account password. Falls back to DATALAB_PASSWORD or SCISERVER_PASSWORD. |
| `--data-release`, `-R` `DATA_RELEASE` | `DR20`    | —                                                              | [sciserver] SDSS release to query, e.g. DR20. Default: DR20.                             |
| `--dry-run`                           | `false`   | —                                                              | List the orphaned tables without dropping them.                                          |
| `--log-level` `LOG_LEVEL`             | `INFO`    | `DEBUG`, `INFO`, `WARNING`, `ERROR`, `CRITICAL`                | Logging verbosity level. Default: INFO.                                                  |

### `mosaics-query ssa`

List available spectra from a Simple Spectral Access service.

```text
mosaics.cli.query ssa [-h] --url URL [--ra CENTER_RA]
                             [--dec CENTER_DEC] [--size SIZE]
                             [--collection COLLECTION] [--param KEY=VALUE]
                             --output OUTPUT [--cache CACHE_DIR]
                             [--workers WORKERS] [--retries RETRIES] [--async]
                             [--log-level {DEBUG,INFO,WARNING,ERROR,CRITICAL}]
```

| Option / Argument           | Default    | Choices                                         | Description                                                                                                                                                                |
|-----------------------------|------------|-------------------------------------------------|----------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `--url`, `-u` `URL`         | *required* | —                                               | SSA endpoint, either a full URL or one of: 6dfgs, euclid, lamost.                                                                                                          |
| `--ra` `CENTER_RA`          | `None`     | —                                               | Search centre right ascension in degrees. Omit to request everything.                                                                                                      |
| `--dec` `CENTER_DEC`        | `None`     | —                                               | Search centre declination in degrees. Omit to request everything.                                                                                                          |
| `--size` `SIZE`             | `0.1`      | —                                               | Search diameter in degrees. Default: 0.1.                                                                                                                                  |
| `--collection` `COLLECTION` | `None`     | —                                               | COLLECTION parameter some services require. Implied for: euclid.                                                                                                           |
| `--param` `KEY=VALUE`       | `[]`       | —                                               | Extra query parameter, repeatable.                                                                                                                                         |
| `--output`, `-o` `OUTPUT`   | *required* | —                                               | Destination file path. Format inferred from extension.                                                                                                                     |
| `--cache` `CACHE_DIR`       | `None`     | —                                               | Directory for intermediate parquet files. A temporary directory is created (and cleaned up) if omitted.                                                                    |
| `--workers`, `-j` `WORKERS` | `4`        | —                                               | Maximum number of concurrent provider jobs. Default: 4.                                                                                                                    |
| `--retries` `RETRIES`       | `5`        | —                                               | Per-batch retry count with exponential backoff. Default: 5.                                                                                                                |
| `--async`                   | `false`    | —                                               | Execute each batch as a server-side async job instead of a synchronous request. Async jobs can queue on the server and take substantially longer per batch. Default: sync. |
| `--log-level` `LOG_LEVEL`   | `INFO`     | `DEBUG`, `INFO`, `WARNING`, `ERROR`, `CRITICAL` | Logging verbosity level. Default: INFO.                                                                                                                                    |
<!-- /generated -->
