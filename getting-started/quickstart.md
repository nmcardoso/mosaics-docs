# Quickstart

This walk-through runs the three core workflows—image cutouts, 1D spectra, and catalogue queries—using the sample target lists shipped in `test/`.

## 1. Preview and fetch multi-survey image cutouts

Given a table with target positions (`id`, `ra`, `dec`), `mosaics-cutout full` routes your requested surveys and passbands across the enabled cutout backends defined in `defs/full.yaml`.

First, inspect the routing plan without making any network requests (`--plan`):

```console
$ uv run -m mosaics.cli.cutout full -t test/objects_sample_1k.csv -o cutouts.lance --config defs/cutout.yaml --size 96 --fov-unit arcmin --plan
```

Run the extraction (writing batches to `cutouts.lance` as they complete so an interrupted run can be resumed with `--resume`):

```console
$ uv run -m mosaics.cli.cutout full -t test/objects_sample_1k.csv -o cutouts.lance --config defs/cutout.yaml --size 96 --fov-unit arcmin --batch-size 128
```

## 2. Fetch 1D spectra for a target table

Each spectroscopic survey in `defs/surveys/` has its own subcommand under `mosaics-spectrum` (plus `full` to run multiple spectroscopic surveys into one Lance dataset).

For example, to fetch SDSS DR17 coadded spectra for a table carrying `sdss17_plate`, `sdss17_mjd`, `sdss17_fiberid` (or to run all configured spectroscopic adapters with `full`):

```console
$ uv run -m mosaics.cli.spectrum sdss17 -t test/objects_sample_1k.csv -o sdss17_spectra.lance --snr-floor 2.0
$ uv run -m mosaics.cli.spectrum full -t test/objects_sample_1k.csv -o all_spectra.lance --config defs/full.yaml --snr-floor 2.0
```

## 3. Query a remote astronomical catalogue

Use `mosaics-query` to run a cone search, a sky box (`region`), or a custom ADQL/SQL query against any supported provider (`tap`, `adl`, `sciserver`, `crossid`, `local`, `nersc`):

```console
$ uv run -m mosaics.cli.query cone --provider tap --service vizier --table "VII/259/6dfgs" --columns 6dFGS RAJ2000 DEJ2000 cz --ra 180.0 --dec -10.0 --radius 1.0 -o 6dfgs_cone.parquet
$ uv run -m mosaics.cli.query region --provider adl --table gaia_dr3.gaia_source --ra-range 180.0 180.2 --dec-range 0.0 0.2 --columns source_id ra dec phot_g_mean_mag -o gaia_box.parquet
```

## 4. Match a table against surveys, and size its cutouts

`mosaics-crossmatch run` joins the columns of one or more surveys onto a table of positions; with `--fov` it also adds the size of the cutout that frames each galaxy of one of them. The shipped `wise` file needs no login:

```console
$ uv run -m mosaics.crossmatch.cli run --survey surveys/wise -i test/objects_sample_1k.csv -o matched.parquet --dry-run
$ uv run -m mosaics.crossmatch.cli run --survey extensions/ls11-datalab -i test/objects_sample_1k.csv -o galaxies.parquet --fov ls11
```

The first prints what it would send and sends nothing. See [crossmatching](../concepts/crossmatch.md) and [field of view](../concepts/field-of-view.md).

## 5. Build a multi-survey target catalogue

`mosaics-targets` runs the eight-stage target selection and multi-survey crossmatch pipeline over declination strips (`--dec-range`), writing sky-cell stores that resume automatically and producing a homogenised Parquet/Lance catalogue alongside a `scireport` bundle and PDF report:

```console
$ uv run -m mosaics.cli.targets --config defs/targets.yaml -o ./targets_run --dec-range 0.0 1.0 --plan plan.yaml
$ uv run -m mosaics.cli.targets --config defs/targets.yaml -o ./targets_run --dec-range 0.0 1.0
```
