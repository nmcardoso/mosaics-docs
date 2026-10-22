# `mosaics-fov` — Cutout Size for a Galaxy Table

`mosaics-fov` (`uv run -m mosaics.fov.cli`) adds a `fov` column to a table of Legacy Survey galaxies: the edge, in arcseconds, of the square cutout that frames each galaxy. It is the same computation the targets pipeline writes into the catalogue’s `fov` column and that `mosaics-crossmatch run --fov` adds after a match, so a table made by any of the three has the same sizes.

## What it needs

The table needs the Legacy Survey model of each galaxy, as the survey files alias it:

| Column                 | Meaning                                                                                                                                                                                                                   |
|------------------------|---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `shape_r`              | half-light radius of the galaxy model, arcseconds                                                                                                                                                                         |
| `shape_e1`, `shape_e2` | the two ellipticity components of the galaxy model                                                                                                                                                                        |
| `mag_r`                | extinction-corrected `r` magnitude; or `flux_r` (model flux, nanomaggy) together with `mw_transmission_r` (Galactic transmission, linear 0 to 1), from which it is made as `22.5 − 2.5 log10(flux_r / mw_transmission_r)` |

`--prefix P` looks for `P_shape_r`, `P_shape_e1`, and so on (`--prefix ls11` for the catalogue and for a crossmatch with `ls11`); with no prefix the bare names are used. Any column can be named outright with its own option. A missing column is refused, with its name, before the table is read.

## How the size is chosen

The [field-of-view page](../concepts/field-of-view.md) has the equations, the correction table and the reasoning; in short:

- **Ellipse (preferred).** The ellipticity components give the axis ratio and angle of the galaxy; the larger side of the ellipse’s bounding box, times a correction that depends on the `r` magnitude, is the size. The half-light radius `shape_r` plays no part in it; the size is set by the shape and the magnitude alone. The ellipticity is capped at `--max-e` (0.6 by default) so a near-degenerate fit cannot set the size.
- **Circle (fallback).** A galaxy the model fitted without an ellipticity gets twice its half-light radius times a correction.
- **Half-light ellipse (only with `--hle`).** The half-light radius times the circle’s correction factor is the semi-major axis of the ellipse that the ellipticity components describe (axis ratio $(1-e)/(1+e)$), and the larger side of its bounding box is the size. A round galaxy gets exactly the circle’s size. With the flag this is the **only** method used: a row without an ellipticity gets no field of view instead of the circle’s.
- **None.** With no usable input, without a magnitude, or with a size of zero (a half-light radius of zero and no ellipse), the row has no field of view: it keeps an empty value, or is left out with `--drop-missing`.

The correction is read from a table of factors by `r` magnitude, from 8 to 19; outside that range the factor of the nearest end is used. `--interp linear` (the default) interpolates between the bins’ middles; `step` uses the bin’s own value. The log says how many rows took each form.

## Examples

`fov` is in arcseconds; give `--fov-unit arcsec` to `mosaics-cutout` when you cut out from the result.

```console
$ uv run -m mosaics.fov.cli -i matched.parquet -o with_fov.parquet --prefix ls11
$ uv run -m mosaics.fov.cli -i matched.parquet -o with_fov_hle.parquet --prefix ls11 --hle
$ uv run -m mosaics.fov.cli -i galaxies.csv -o galaxies_fov.csv --shape-r r_half --shape-e1 e1 --shape-e2 e2 --mag-r mag --drop-missing
```

## Command-line reference

<!-- generated:cli-fov -->

**Entry points:** `mosaics-fov` or `uv run -m mosaics.fov.cli`

### Top-level options

```text
mosaics-fov [-h] --input INPUT --output OUTPUT [--overwrite]
                   [--prefix P] [--shape-r COL] [--shape-e1 COL]
                   [--shape-e2 COL] [--mag-r COL] [--flux-r COL]
                   [--mw-transmission-r COL] [--interp {linear,step}]
                   [--max-e MAX_E] [--hle] [--column COLUMN] [--drop-missing]
                   [--log-level {DEBUG,INFO,WARNING,ERROR,CRITICAL}]
```

| Option / Argument           | Default    | Choices                                         | Description                                                                                                                                                                                                                                                                                                                                                         |
|-----------------------------|------------|-------------------------------------------------|---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `--input`, `-i` `INPUT`     | *required* | —                                               | The table, in any format read_table opens.                                                                                                                                                                                                                                                                                                                          |
| `--output`, `-o` `OUTPUT`   | *required* | —                                               | Where to write the table with the added column (any format write_table writes). Never the input.                                                                                                                                                                                                                                                                    |
| `--overwrite`               | `false`    | —                                               | Replace an existing output.                                                                                                                                                                                                                                                                                                                                         |
| `--prefix` `P`              | `None`     | —                                               | The prefix of the survey columns, e.g. ls11. Default: none.                                                                                                                                                                                                                                                                                                         |
| `--shape-r` `COL`           | `None`     | —                                               | The half-light radius of the galaxy model, arcsec.                                                                                                                                                                                                                                                                                                                  |
| `--shape-e1` `COL`          | `None`     | —                                               | The first ellipticity component of the galaxy model.                                                                                                                                                                                                                                                                                                                |
| `--shape-e2` `COL`          | `None`     | —                                               | The second ellipticity component of the galaxy model.                                                                                                                                                                                                                                                                                                               |
| `--mag-r` `COL`             | `None`     | —                                               | The extinction-corrected r magnitude.                                                                                                                                                                                                                                                                                                                               |
| `--flux-r` `COL`            | `None`     | —                                               | The model flux in r, nanomaggy; with –mw-transmission-r, in place of a magnitude.                                                                                                                                                                                                                                                                                   |
| `--mw-transmission-r` `COL` | `None`     | —                                               | The Galactic transmission in r, linear from 0 to 1.                                                                                                                                                                                                                                                                                                                 |
| `--interp` `INTERP`         | `linear`   | `linear`, `step`                                | How the correction table is interpolated in magnitude. Default: linear.                                                                                                                                                                                                                                                                                             |
| `--max-e` `MAX_E`           | `0.6`      | —                                               | Cap on the ellipticity of the elliptical and half-light-ellipse forms. Default: 0.6.                                                                                                                                                                                                                                                                                |
| `--hle`                     | `false`    | —                                               | Use only the half-light-ellipse method: the half-light radius times the circle correction factor of the r magnitude is the semi-major axis of the ellipse that shape_e1 and shape_e2 describe (axis ratio (1-e)/(1+e)), and the edge is the larger side of its bounding box. By default the ellipse form is used, and the circle form where the ellipse is missing. |
| `--column` `COLUMN`         | `fov`      | —                                               | Name of the added column. Default: fov.                                                                                                                                                                                                                                                                                                                             |
| `--drop-missing`            | `false`    | —                                               | Leave out the rows that have no field of view (an input is missing, or the size is zero). By default they are kept with a null.                                                                                                                                                                                                                                     |
| `--log-level` `LOG_LEVEL`   | `INFO`     | `DEBUG`, `INFO`, `WARNING`, `ERROR`, `CRITICAL` | Logging verbosity. Default: INFO.                                                                                                                                                                                                                                                                                                                                   |
<!-- /generated -->
