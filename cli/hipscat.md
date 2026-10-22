# `mosaics-hipscat` — Aladin HiPS Catalogue Builder

`mosaics-hipscat` (`uv run -m mosaics.cli.hipscat`) converts a produced target catalogue or cutout dataset carrying `ra`, `dec`, and `fov` into a standard HiPS progressive catalogue directory (`properties`, `Moc.fits`, and `NorderK/DirD/NpixN.tsv` tiles) with an `s_region` square footprint column that Aladin Desktop or Aladin Lite renders directly on the sky.

## Examples

```console
$ uv run -m mosaics.cli.hipscat -d targets.lance -o ./hips_catalog --fov-unit arcsec --title "MOSAICS Targets Framing"
```

## Command-line reference

<!-- generated:cli-hipscat -->

**Entry points:** `mosaics-hipscat` or `uv run -m mosaics.cli.hipscat`

### Top-level options

```text
mosaics.cli.hipscat [-h] --dataset DATASET
                           [--fov-unit {arcmin,arcsec,deg}]
                           [--id-col ID_COLUMN] [--limit LIMIT] --output
                           OUTPUT [--order ORDER] [--title TITLE]
                           [--log-level {DEBUG,INFO,WARNING,ERROR,CRITICAL}]
```

| Option / Argument           | Default    | Choices                                         | Description                                                                                                                                                                                                                                     |
|-----------------------------|------------|-------------------------------------------------|-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `--dataset`, `-d` `DATASET` | *required* | —                                               | Lance dataset to read. Needs ra, dec and fov columns.                                                                                                                                                                                           |
| `--fov-unit` `FOV_UNIT`     | `arcsec`   | `arcmin`, `arcsec`, `deg`                       | Unit of the fov column, which is the *edge* of the square rather than a radius. Default: arcsec, which is what the production catalogue writes – note mosaics.cli.cutout defaults to arcmin instead, so a table built for that needs this flag. |
| `--id-col` `ID_COLUMN`      | `None`     | —                                               | Column holding each row key, shown as the row name in a client. Defaults to mosaics_id, else the first string column.                                                                                                                           |
| `--limit` `LIMIT`           | `None`     | —                                               | Read at most this many rows. Useful for a first look at a large run.                                                                                                                                                                            |
| `--output`, `-o` `OUTPUT`   | *required* | —                                               | Directory to write the catalogue into. Created if absent.                                                                                                                                                                                       |
| `--order` `ORDER`           | `None`     | —                                               | HEALPix order of the tiling. Default: chosen from the row count so a tile holds roughly ten thousand rows, which is a tile a client draws without stalling.                                                                                     |
| `--title` `TITLE`           | `None`     | —                                               | Human label for the catalogue. Default: the dataset name.                                                                                                                                                                                       |
| `--log-level` `LOG_LEVEL`   | `INFO`     | `DEBUG`, `INFO`, `WARNING`, `ERROR`, `CRITICAL` | Logging verbosity level. Default: INFO.                                                                                                                                                                                                         |
<!-- /generated -->
