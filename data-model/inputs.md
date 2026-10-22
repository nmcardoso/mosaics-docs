# Input Table Formats & Coordinate Conventions

Every MOSAICS command that takes a target list (`-t` / `--targets` on `mosaics-cutout`, `mosaics-spectrum`, `mosaics-crossmatch run` (as `-i` / `--input`), `mosaics-query ssa`, and `-i` / `--input` on `mosaics-hipscat`) reads it through a unified table reader.

## 1. Supported file formats

Format is detected automatically from the file or directory extension:

| Extension                    | Format                     | Notes                                                                                         |
|------------------------------|----------------------------|-----------------------------------------------------------------------------------------------|
| `.parquet`, `.pq`            | Apache Parquet             | Recommended for large tables ($N \ge 10^5$); preserves Arrow types and metadata.              |
| `.lance`                     | Lance dataset              | Directory format produced by `mosaics-cutout`, `mosaics-spectrum`, and `mosaics-targets`.     |
| `.fits`, `.fit`, `.fts`      | FITS binary or ASCII table | Reads the first table HDU; blank ASCII integer fields are preserved as nulls rather than `0`. |
| `.csv`, `.tsv`               | Delimited text             | Comma- or tab-separated UTF-8 text with a header row.                                         |
| `.feather`, `.arrow`, `.ipc` | Arrow IPC / Feather        | Memory-mapped columnar format.                                                                |

All input columns are preserved in the output dataset along with any new image, spectrum, or crossmatch columns.

## 2. Required columns and automatic inference

Target-based pipelines require three logical columns:

| Logical role        | CLI override flag   | Auto-detected column names (case-insensitive)                                                                                                     | Units / Type                                           |
|---------------------|---------------------|---------------------------------------------------------------------------------------------------------------------------------------------------|--------------------------------------------------------|
| **Unique key**      | `--id-col`          | `id`, `targetid`, `objid`, `source_id`, `ls_id_dr11`, `mosaics_id` (generated as 0-based `mosaics_id` when omitted and no `id` column is present) | Integer or string; **must be unique** across all rows. |
| **Right ascension** | `--ra-col`          | `ra`, `coord_ra`, `raj2000`, `plug_ra`, `mean_fiber_ra`, `ra_icrs`                                                                                | ICRS decimal degrees $[0, 360)$.                       |
| **Declination**     | `--dec-col`         | `dec`, `coord_dec`, `dej2000`, `plug_dec`, `mean_fiber_dec`, `dec_icrs`                                                                           | ICRS decimal degrees $[-90, +90]$.                     |

## 3. Optional per-target field of view (`--fov-col`)

By default, `mosaics-cutout` uses a constant angular field of view (`--fov` and `--fov-unit`, default `0.5 arcmin`) for every stamp. When extracting cutouts from a `mosaics-targets` production catalogue (which computes an object-adaptive `fov` column in arcseconds), pass `--fov-col fov --fov-unit arcsec` so each stamp’s angular width scales with the galaxy’s size while keeping the output pixel grid (`--size`, e.g. `96x96`) fixed. `mosaics-fov` and `mosaics-crossmatch run --fov` make the same column for any table, and [Field of view](../concepts/field-of-view.md) explains how it is computed.

## 4. Sample target files

The repository ships sample target tables under `test/` for dry runs and testing:

- `test/objects_sample_1k.csv`: 1,000 extragalactic sky positions with `id`, `ra`, `dec`, and survey identifiers.
