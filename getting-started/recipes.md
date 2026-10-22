# End-to-End Recipes

Each recipe below is self-contained: it shows the input file or YAML configuration, the exact CLI command, and the resulting output files and columns.

---

## Recipe 1: Extract calibrated optical + NIR cutouts from a custom survey subset

**Goal:** Given a CSV of galaxy coordinates (`id,ra,dec`), extract $128 \times 128$ pixel cutouts in Legacy Survey DR11 ($g, r, z$ via NERSC bricks) and SkyMapper DR4 ($g, r, i$ via local HiPS tile rendering) into a single Lance dataset.

**1. Input table (`targets.csv`):**

```text
id,ra,dec
1001,180.1234,-5.4321
1002,180.2345,-5.3210
```

**2. Entrypoint configuration (`my_cutouts.yaml`):**

```yaml
surveys:
  - surveys/ls11
  - surveys/skymapper4

overrides:
  surveys/ls11:
    cutout:
      - adapter: nersc
        adapter_params:
          bands: grz
          data_release: dr11
        bands: [g, r, z]
```

Relative paths in `surveys:` resolve against the file’s directory, then `$CWD`, then `defs/`. `surveys:` lists the survey files the run covers and `overrides:` changes the ones its key names.

**3. Command:**

```console
$ uv run -m mosaics.cli.cutout full -t test/objects_sample_1k.csv -o cutouts_opt.lance --config defs/cutout.yaml --size 128 --fov-unit arcmin
```

**4. Output:**

- `cutouts_opt.lance/`: Lance dataset containing `id`, `ra`, `dec`, and fixed-shape `float32` tensor columns `cutout_ls11_g`, `cutout_ls11_r`, `cutout_ls11_z`, `cutout_skymapper4_g`, `cutout_skymapper4_r`, `cutout_skymapper4_i` of shape `(1, 128, 128)`, with per-column calibration stored in the Arrow field metadata (`mosaics_unit`, `mosaics_scale`, `mosaics_offset`).
- `cutouts_opt.failures.parquet`: sidecar listing any target/service pair that failed, ready for `--retry-failed`.

---

## Recipe 2: Resume an interrupted cutout or spectrum run and retry transient archive errors

**Goal:** Continue a long extraction after a network interruption or `Ctrl-C`, then re-attempt only the rows that failed during the initial pass.

**Commands:**

```console
$ uv run -m mosaics.cli.cutout full -t test/objects_sample_1k.csv -o cutouts_opt.lance --config defs/cutout.yaml --resume
$ uv run -m mosaics.cli.cutout full -t test/objects_sample_1k.csv -o cutouts_opt.lance --config defs/cutout.yaml --resume --retry-failed --fill-missing rewrite
```

- `--resume` reads the existing key column (`id`) from `cutouts_opt.lance` and skips every row already committed.
- `--retry-failed` reads `cutouts_opt.failures.parquet`, re-fetches only the failed `(id, service)` pairs, and with `--fill-missing rewrite` patches the recovered tensors back into `cutouts_opt.lance` in place.

---

## Recipe 3: Crossmatch a local table against Rubin/LSST Data Preview 2 (`dp2.Object`)

**Goal:** Match a local Parquet catalogue against `dp2.Object` on the Rubin Science Platform, filtering out positions outside the release footprint beforehand so only in-footprint targets cross the wire.

**Commands:**

```console
$ uv run -m mosaics.crossmatch.cli moc --survey surveys/lsstdp2 -o lsst-dp2-moc.fits
$ uv run -m mosaics.crossmatch.cli run --survey surveys/lsstdp2 -i test/objects_sample_1k.csv -o matched_lsst.parquet --filter-moc
```

The radius, the columns and the release come from the shipped `lsstdp2` survey file. Name the footprint under `overrides:` in a job file (`crossmatch: {moc: lsst-dp2-moc.fits}`) so `--filter-moc` finds it, or describe the source on the command line with no survey file at all:

```console
$ uv run -m mosaics.crossmatch.cli run -i test/objects_sample_1k.csv -o matched_lsst.parquet --adapter lsst --param release=dp2 --table dp2.Object --radius "1 arcsec" --columns objectId coord_ra coord_dec "[ugrizy]_cModelMag" --moc lsst-dp2-moc.fits --filter-moc
```

**Output:**

- `lsst-dp2-moc.fits`: Multi-Order Coverage map built once from `dp2.CoaddPatches`.
- `matched_lsst.parquet`: All input columns preserved plus the matched `dp2.Object` columns, the separation and the number of counterparts.
- `matched_lsst.crossmatch.yaml` and `matched_lsst.report/`: Machine-readable run account, sealed `crossmatch.scireport.zip` bundle, chapter Markdown files in `md/`, and `pdf/report.pdf`.

**Several surveys at once.** A job file lists them and may change any survey inline; every key is an option of `run`:

```yaml
# runs/two-surveys.yaml
surveys: [surveys/wise, surveys/gaia3]
input: test/objects_sample_1k.csv
output: matched.parquet
overrides:
  surveys/wise:
    crossmatch:
      radius: 2 arcsec
```

```console
$ uv run -m mosaics.crossmatch.cli run -c runs/two-surveys.yaml
```

**Extending a shipped survey.** The shipped survey files are a library you extend rather than edit. To match `wise` at 2 arcseconds against your own copy of the table, write a survey file that extends the shipped one and name it:

```yaml
# my_wise.yaml
extends: surveys/wise.yaml
crossmatch:
  radius: 2 arcsec
catalog:
  - name: wise
    type: photometric
    adapter: tap
    adapter_params:
      service: https://example.org/tap
      table: my.allwise_copy
```

```console
$ uv run -m mosaics.crossmatch.cli run --survey my_wise -i test/objects_sample_1k.csv -o matched_wise.parquet
```

The slug stays `wise`, so the columns keep their `wise_` prefix. The file is found next to where you run the command, and `extends:` finds `surveys/wise.yaml` in the package’s `defs/`.

**Adding the cutout size.** `--fov SURVEY` adds the `fov` column from the shape and magnitude columns that survey brought, and `--hle` sizes it with the half-light ellipse alone (see [Field of view](../concepts/field-of-view.md)). `mosaics-cutout` then needs `--fov-unit arcsec`:

```console
$ uv run -m mosaics.crossmatch.cli run --survey extensions/ls11-datalab -i test/objects_sample_1k.csv -o matched_ls11.parquet --fov ls11 --hle
$ uv run -m mosaics.cli.cutout lsv -t matched_ls11.parquet -o lsv_grz.lance --layer ls-dr11 --bands grz --size 96 --fov-unit arcsec
```

---

## Recipe 4: Run a multi-node extraction across SSH worker machines

**Goal:** Spread archive requests across multiple machines (`node1.example.org`, `node2.example.org`) to multiply per-IP archive rate limits while keeping all orchestration and storage on the main node.

**Commands:**

```console
$ uv run -m mosaics.cli.proxy keygen
$ uv run -m mosaics.cli.proxy server --proxy-port 8888 --tunnel-port 8022 --stats-port 8889
$ uv run -m mosaics.cli.targets --config defs/targets.yaml -o ./targets_run --dec-range -5.0 5.0 --proxy http://127.0.0.1:8888
```

**Output:**

- The proxy server deploys and heartbeats lightweight worker processes over SSH, load-balancing per-origin requests by live worker goodput and latency.
- When `mosaics-targets` finishes, it queries the proxy stats endpoint, merges per-node throughput and traffic into `targets_report.yaml`, and renders the multi-node execution chapter in the report.
