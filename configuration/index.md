# Survey Configuration Reference

Every survey in `mosaics` is defined once in a declarative YAML file under `defs/surveys/<slug>.yaml`. Entrypoint files under `defs/*.yaml` list the surveys a run covers (`surveys:`), change some of them (`overrides:`) and are passed to CLI commands via `--config`.

This page documents every root section, every section key, and every adapter parameter accepted by the configuration loader, followed by complete worked examples that pass strict loader validation.

## File Structure & Composition

There are two kinds of YAML file, and a file is one or the other:

- A **survey file** defines one survey. The files in `defs/surveys/` are the *library*: read-only for users, who extend them rather than edit them.
- An **entrypoint** maps the arguments of one command and says which survey files it runs on. `defs/*.yaml` are the shipped entrypoints.

A **survey file** (`defs/surveys/<slug>.yaml`) declares `survey_slug`, `name`, and whichever of the ten capability and metadata sections apply to that survey (`catalog`, `catalog_filters`, `cutout`, `spectra`, `photometric_system`, `spectroscopic_system`, `target_selection`, `provides`, `column_mapping`, `crossmatch`). It is unfolded: keys at the top, no `<slug>:` wrapper.

An **entrypoint** declares `surveys:` (the roster: a string or list of survey files or glob patterns), an optional `exclude:` (files to leave out of the roster), an optional `overrides:` mapping of changes to roster surveys, and an optional `extends:` of other entrypoints:

```yaml
surveys:
  - surveys/sdss20
  - surveys/ps1
  - surveys/viking

exclude:
  - surveys/viking

overrides:
  surveys/ps1:
    cutout: []
```

`overrides:` keys are globs over a roster survey’s name (its path under `defs/`, without `.yaml`, e.g. `surveys/ps1`) or its slug (`ps1`). Every key that matches a survey is merged over it, deeply, before validation; the less specific key goes first, so the specific key wins a conflict and the general one still supplies what it leaves unset. A key that matches no roster survey logs a warning.

On the command line, `--survey REF …` replaces the entrypoint’s roster (the entrypoint’s `overrides:` still apply), `--exclude-survey REF …` adds to `exclude:`, and every run logs the **effective configuration**: the files in play, the arguments with where each value came from, and for each survey every key an override changed, as `library value -> effective value`. The complete definitions are logged at `debug`.

To change a library survey for good in your own runs, write a **survey file that extends it** and list that file instead of the library one:

```yaml
extends: surveys/ps1.yaml
name: Pan-STARRS1 DR1 (group copy)
cutout: []
```

When `extends:` is evaluated:

- It is a plain dictionary default: the extended file supplies defaults and the extending file overrides them. It never adds surveys to a run; that is `surveys:`.
- An entrypoint extends entrypoints and a survey file extends survey files. Crossing raises an error that shows the new spelling.
- Each entry expands `~` and environment variables. Absolute paths are used as given, and the `.yaml` suffix is optional everywhere (`surveys/ps1`).
- Relative paths are resolved against the naming file’s directory, then `$CWD`, then `defs/`, deduplicated by resolved path. The first base with a match wins; if a relative path exists in multiple distinct bases, a warning is logged naming the winner and shadowed paths. `surveys:` and `exclude:` use the same lookup.
- Recursive extension is supported; circular references are detected and raise an error with the full chain.
- Maps are deep-merged left to right and the extending file wins; lists replace the inherited list, and `key: null` removes the key.
- An extension takes its slug from the merged result, so a roster that lists both an extension and the file it extends raises.
- The root `--config` option resolves relative paths against `$CWD`, then `defs/` (e.g. `--config targets.yaml` works from anywhere, and so does the repository-root spelling `--config defs/targets.yaml`).
- Validation is **strict**: any unknown root key, unknown section key, unknown `adapter_params` key for the declared `(section, adapter)` pair, invalid enum value, or unprefixed column alias raises an immediate validation error before any archive is contacted.

### Column Prefix Rule

A survey’s output column prefix is derived automatically from its `survey_slug`:

- If a survey defines **either** a photometric catalog (`type: photometric`) **or** a spectroscopic catalog (`type: spectroscopic`), its columns take the bare slug prefix `{slug}_`.
- If a survey defines **both** a photometric and a spectroscopic catalog in the same file (for example, `euclidq1`), the photometric catalog uses `{slug}p_` and the spectroscopic catalog uses `{slug}s_` so their coordinate and identifier columns (`{prefix}ra`, `{prefix}dec`) never collide.

---

## Configuration Sections

<!-- generated:config-sections -->

### Entrypoint top-level keys (`defs/*.yaml`)

| Key         | Type                                              | Required   | Default   | Allowed values                                                             | Description                                                                                                                                                                                                                                                  |
|-------------|---------------------------------------------------|------------|-----------|----------------------------------------------------------------------------|--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `extends`   | str or list of str                                | no         | []        | relative or absolute paths, ~ or globs; .yaml optional                     | Files whose keys are the defaults of this one, merged left to right (later wins, this file last). An entrypoint extends entrypoints and a survey file extends survey files; relative paths search the naming file’s directory, then $CWD, then defs/.        |
| `surveys`   | str or list of str                                | no         | []        | survey files by relative or absolute path, ~ or globs; .yaml optional      | The roster, the survey files a run covers, in declaration order (globs sorted, a file named twice counts once). `--survey` on the command line replaces it.                                                                                                  |
| `exclude`   | str or list of str                                | no         | []        | relative or absolute paths, ~ or globs; .yaml optional                     | Files to remove from the roster; an entry matching no files is allowed and logged at info. `--exclude-survey` adds to it.                                                                                                                                    |
| `overrides` | mapping (survey key -> partial survey definition) | no         | {}        | keys are globs over a survey file’s name (`surveys/wise`) or slug (`wise`) | Partial survey definitions merged deeply over each roster survey the key matches, after its own `extends` chain and before validation; less specific keys apply first. Any other top-level key of an entrypoint is an argument of the command that reads it. |

### Survey definition root keys (`defs/surveys/<slug>.yaml`)

| Key                    | Type                                       | Required   | Default   | Allowed values                                                                                                                                | Description                                                                                                                                                                                                                  |
|------------------------|--------------------------------------------|------------|-----------|-----------------------------------------------------------------------------------------------------------------------------------------------|------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `survey_slug`          | str                                        | yes        | —         | must equal the YAML file stem                                                                                                                 | Canonical lowercase identifier for the survey and default column prefix.                                                                                                                                                     |
| `name`                 | str                                        | yes        | —         | non-empty string                                                                                                                              | Human-readable survey and data-release name used in logs, plans, and reports.                                                                                                                                                |
| `catalog`              | list of catalog entries                    | no         | []        | at most one entry per `type` (`photometric`, `spectroscopic`)                                                                                 | Remote or local catalogues queried for target extraction or crossmatching.                                                                                                                                                   |
| `catalog_filters`      | mapping (prefix -> SQL expression or note) | no         | {}        | `<prefix>` and `<prefix>_note`                                                                                                                | Post-hoc DuckDB SQL quality/depth/redshift cuts applied during production (nulls failing survey columns without dropping the row).                                                                                           |
| `cutout`               | list of cutout entries                     | no         | []        | see Cutout entry keys                                                                                                                         | Image cutout backends (`hips`, `lsv`, `nersc`, `sia`, `vsa`) serving this survey’s passbands.                                                                                                                                |
| `spectra`              | list of spectra entries                    | no         | []        | see Spectra entry keys                                                                                                                        | 1D spectrum download backends (`url`, `template`, `sparcl`, `desi-healpix`).                                                                                                                                                 |
| `photometric_system`   | mapping                                    | no         | None      | see Photometric system keys                                                                                                                   | Passband SVO filter identifiers, flux/magnitude homogenisation rules, and image pixel calibration.                                                                                                                           |
| `spectroscopic_system` | mapping                                    | no         | None      | see Spectroscopic system keys                                                                                                                 | Spectrograph fibre diameter, disperser, redshift tiebreak rank, and flux calibration facts.                                                                                                                                  |
| `target_selection`     | mapping                                    | no         | None      | see Target selection keys                                                                                                                     | How this survey seeds or joins the candidate table in the targets pipeline.                                                                                                                                                  |
| `provides`             | mapping (prefix -> photometric system)     | no         | {}        | see Provides keys                                                                                                                             | Another survey’s photometry already carried inside this survey’s extraction query (e.g. `gaia3` and `unwise` inside `ls11`).                                                                                                 |
| `column_mapping`       | mapping (remote_column -> alias)           | no         | {}        | alias without prefix (an alias that starts with `<prefix>_` raises); glob keys with `{1}` captures; a mapping value names a `provides` prefix | Projection and column renaming applied when crossmatching against this survey’s photometric catalogue. The loader adds the catalog’s prefix to every alias, or the hosted survey’s for the columns under its `provides` key. |
| `crossmatch`           | mapping                                    | no         | None      | see Crossmatch keys                                                                                                                           | How a table of positions is matched against this survey. Required for a survey with an `lsdb` catalog; `crossmatch` null removes it, and a survey without it takes no part in `mosaics-crossmatch`.                          |

### `catalog[]` entry keys

| Key              | Type    | Required   | Default   | Allowed values                                                                              | Description                                                                              |
|------------------|---------|------------|-----------|---------------------------------------------------------------------------------------------|------------------------------------------------------------------------------------------|
| `name`           | str     | yes        | —         | `{slug}`, or `{slug}p` / `{slug}s` when both photometric and spectroscopic catalogues exist | Column prefix and catalogue identifier for this entry.                                   |
| `type`           | str     | yes        | —         | `photometric`, `spectroscopic`                                                              | Whether this catalogue provides broadband photometry or spectroscopic redshifts/targets. |
| `adapter`        | str     | yes        | —         | `adl`, `tap`, `crossid`, `local`, `nersc`, `lsdb`                                           | Catalogue query provider backend.                                                        |
| `adapter_params` | mapping | no         | {}        | see Adapter parameters by catalog adapter                                                   | Provider-specific connection, table, coordinate, and chunking settings.                  |
| `enabled`        | bool    | no         | true      | `true`, `false`                                                                             | When `false`, the catalogue entry is skipped by all pipelines.                           |
| `unflagged`      | str     | no         | None      | free text explanation                                                                       | Reason why the archive provides no usable quality flag for this catalogue’s photometry.  |

### `cutout[]` entry keys

| Key               | Type        | Required   | Default                                       | Allowed values                                                                              | Description                                                                                  |
|-------------------|-------------|------------|-----------------------------------------------|---------------------------------------------------------------------------------------------|----------------------------------------------------------------------------------------------|
| `adapter`         | str         | yes        | —                                             | `hips`, `lsv`, `nersc`, `sia`, `vsa`                                                        | Cutout retrieval backend.                                                                    |
| `adapter_params`  | mapping     | no         | {}                                            | backend-specific parameters (`hips_id`, `hips_ids`, `layer`, `bands`, `service_url`, `fov`) | Backend-specific parameters passed to the cutout adapter.                                    |
| `bands`           | list of str | yes        | —                                             | lowercase band tokens (e.g. `[g, r, i, z]`) or `[color]`                                    | Output band tokens determining the Lance dataset column names (`cutout_<slug>_<band>`).      |
| `enabled`         | bool        | no         | true                                          | `true`, `false`                                                                             | Whether `cutout full` considers this backend entry.                                          |
| `pixel_unit`      | str         | no         | None (inherits photometric_system.pixel_unit) | `nanomaggy`, `ab_zeropoint`, `counts_per_s`, `unknown`                                      | Per-entry pixel calibration unit override.                                                   |
| `zeropoint`       | float       | no         | None (inherits photometric_system.zeropoint)  | finite float                                                                                | Per-entry AB magnitude zero point override.                                                  |
| `pixel_unit_note` | str         | no         | None                                          | free text                                                                                   | Provenance note explaining the per-entry pixel calibration.                                  |
| `cost`            | mapping     | no         | None                                          | see Cutout cost keys                                                                        | Optional cost-model overrides used when `cutout full` routes bands across multiple backends. |
| `label`           | str         | no         | `<slug>:<adapter>:<band_or_stack>`            | unique string within the run                                                                | Identifier recorded in the `.failures.parquet` sidecar for this cutout service.              |

### `cutout[].cost` keys

| Key                      | Type   | Required   | Default         | Allowed values   | Description                                                           |
|--------------------------|--------|------------|-----------------|------------------|-----------------------------------------------------------------------|
| `latency_s`              | float  | no         | adapter default | >= 0             | Estimated per-request HTTP latency in seconds.                        |
| `bandwidth_bps`          | float  | no         | adapter default | > 0              | Estimated download throughput in bytes per second.                    |
| `concurrency`            | int    | no         | adapter default | >= 1             | Effective concurrent requests allowed by the service.                 |
| `render_seconds`         | float  | no         | adapter default | >= 0             | Server-side or local cutout rendering CPU time per target in seconds. |
| `bytes_penalty_s_per_gb` | float  | no         | adapter default | >= 0             | Disk/network amortisation penalty in seconds per gigabyte downloaded. |

### `spectra[]` entry keys

| Key              | Type        | Required   | Default   | Allowed values                                                                                              | Description                                                                                  |
|------------------|-------------|------------|-----------|-------------------------------------------------------------------------------------------------------------|----------------------------------------------------------------------------------------------|
| `adapter`        | str         | yes        | —         | `url`, `template`, `sparcl`, `desi-healpix`                                                                 | Spectrum retrieval backend.                                                                  |
| `adapter_params` | mapping     | no         | {}        | backend-specific parameters (`url_template`, `data_release`, `specprod`, `coadd_type`, `survey`, `program`) | Parameters forwarded to the spectrum adapter.                                                |
| `format`         | str         | no         | None      | `sdss`, `lamost-mrs`, `2dfgrs`, `6dfgs`, `hasp`, `euclid-q1`, `sparcl`, `desi-healpix`                      | FITS/VOTable decoder name in `mosaics.spectrum.decode`.                                      |
| `arrays`         | list of str | no         | []        | subset of `flux`, `error`, `wavelength`, `mask`                                                             | Parallel spectrum arrays published by this archive and written to the output Lance dataset.  |
| `enabled`        | bool        | no         | true      | `true`, `false`                                                                                             | Whether this spectrum backend is active (two enabled backends may not fill the same column). |

### `photometric_system` keys

| Key                | Type                            | Required   | Default   | Allowed values                                         | Description                                                                                                                              |
|--------------------|---------------------------------|------------|-----------|--------------------------------------------------------|------------------------------------------------------------------------------------------------------------------------------------------|
| `filters`          | mapping (band -> SVO filter ID) | no         | {}        | lowercase band token -> `Facility/Instrument.Filter`   | Maps each band token to its SVO Filter Profile Service identifier; its keys define the survey’s band list.                               |
| `filter_variants`  | mapping                         | no         | None      | see Filter variant keys                                | Per-row filter-profile override when a survey combines multiple instruments or hemispheres (e.g. `ls11_hemisphere`).                     |
| `vega_offsets`     | mapping (band -> float)         | no         | {}        | declared band tokens -> `m_AB - m_Vega`                | Additive AB minus Vega zero-point offsets used to convert `vega_mag` columns to AB magnitudes and microjanskys.                          |
| `measurements`     | list of measurement mappings    | no         | []        | see Photometric measurement keys                       | Native photometric measurement systems, apertures, and uncertainty encodings published by the catalogue.                                 |
| `detection_column` | str                             | no         | None      | local column alias starting with `<prefix>_`           | Column whose non-null presence marks a target as detected by this survey.                                                                |
| `pixel_unit`       | str                             | no         | `unknown` | `nanomaggy`, `ab_zeropoint`, `counts_per_s`, `unknown` | Physical or calibration unit of the survey’s cutout image pixels.                                                                        |
| `zeropoint`        | float                           | no         | None      | finite float                                           | Survey-wide AB magnitude zero point for cutout pixels when `pixel_unit` is `ab_zeropoint` or `counts_per_s`.                             |
| `pixel_unit_note`  | str                             | no         | None      | free text                                              | Provenance explanation and citation for the image pixel calibration.                                                                     |
| `borrow_filters`   | str                             | no         | None      | another survey’s `survey_slug`                         | Borrows the passband `filters` dictionary from another survey definition (used by cutout-only releases such as `ls10` borrowing `ls11`). |

### `photometric_system.filter_variants` keys

| Key      | Type                                                     | Required   | Default   | Allowed values                                        | Description                                                                                           |
|----------|----------------------------------------------------------|------------|-----------|-------------------------------------------------------|-------------------------------------------------------------------------------------------------------|
| `column` | str                                                      | yes        | —         | column name in the catalogue (e.g. `ls11_hemisphere`) | Catalogue column whose value selects which instrument filter mapping applies to a row.                |
| `cases`  | mapping (case_value -> mapping of band -> SVO filter ID) | yes        | —         | non-empty mapping covering declared bands             | Maps each distinct value of `column` (such as `'N'` or `'S'`) to its per-band SVO filter identifiers. |

### `photometric_system.measurements[]` keys

| Key          | Type                             | Required   | Default               | Allowed values                                                              | Description                                                                                                                |
|--------------|----------------------------------|------------|-----------------------|-----------------------------------------------------------------------------|----------------------------------------------------------------------------------------------------------------------------|
| `system`     | str                              | yes        | —                     | `ab_mag`, `vega_mag`, `asinh_mag`, `linear`                                 | Native photometric unit family of the raw catalogue columns before homogenisation to AB magnitudes and microjansky fluxes. |
| `bands`      | list of str                      | no         | all keys of `filters` | subset of declared band tokens                                              | Bands covered by this measurement block.                                                                                   |
| `apertures`  | list of str                      | no         | `[default]`           | aperture tokens (e.g. `default`, `aper3`, `petro`, `psf`, `cmodel`, `kron`) | Photometric aperture or model suffixes present in the column aliases (`<prefix>_<kind>_<aperture>_<band>`).                |
| `scale`      | float or mapping (band -> float) | no         | 1.0                   | positive float(s)                                                           | Multiplicative factor converting native linear flux units into microjanskys (`uJy`), required when `system: linear`.       |
| `error_kind` | str                              | no         | `linear`              | `linear`, `ivar`                                                            | Whether native linear flux uncertainties are 1-sigma errors (`linear`) or inverse variances (`ivar`).                      |
| `errors`     | bool                             | no         | true                  | `true`, `false`                                                             | Set to `false` when the catalogue publishes no uncertainty column for this measurement block.                              |
| `offsets`    | mapping (band -> float)          | no         | {}                    | declared band tokens -> float                                               | Per-band additive residual offsets onto the AB magnitude scale.                                                            |
| `softening`  | mapping (band -> float)          | no         | {}                    | declared band tokens -> positive float `b`                                  | Lupton asinh softening parameters `b` per band, required when `system: asinh_mag`.                                         |

### `spectroscopic_system` keys

| Key                | Type          | Required   | Default   | Allowed values                                      | Description                                                                                                                                         |
|--------------------|---------------|------------|-----------|-----------------------------------------------------|-----------------------------------------------------------------------------------------------------------------------------------------------------|
| `fiber_diameter`   | float or str  | no         | None      | positive float (arcseconds), `'per_row'`, or `None` | Entrance fibre or slit width in arcseconds, or `'per_row'` when selected as a column by the SQL query.                                              |
| `disperser`        | str           | no         | None      | `grating`, `grism`, `prism`, `echelle`, `mixed`     | Spectrograph dispersing element type.                                                                                                               |
| `redshift_rank`    | int           | no         | None      | positive integer (smaller wins)                     | Priority rank when multiple spectroscopic surveys provide a redshift for the same target; `None` excludes the survey from tiebreaking.              |
| `velocity_columns` | list of 2 str | no         | None      | `[velocity_col, velocity_err_col]`                  | Heliocentric recession velocity and uncertainty columns (in km/s) converted to redshift `z = v / c`.                                                |
| `null_sentinel`    | float         | no         | None      | finite float (e.g. `-9999.0`)                       | Archive sentinel value replaced with `NULL` before redshift/velocity conversion.                                                                    |
| `flux_scale`       | float         | no         | None      | positive float                                      | Multiplier converting native spectrum flux values to `10^-17 erg s^-1 cm^-2 Angstrom^-1`.                                                           |
| `flux_calibration` | str           | no         | `unknown` | `absolute`, `relative`, `none`, `unknown`           | Whether published 1D spectra are flux-calibrated in physical units (`absolute`), relative throughput (`relative`), or uncalibrated counts (`none`). |

### `target_selection` keys

| Key            | Type                                   | Required   | Default            | Allowed values                                                                          | Description                                                                                                                        |
|----------------|----------------------------------------|------------|--------------------|-----------------------------------------------------------------------------------------|------------------------------------------------------------------------------------------------------------------------------------|
| `catalog_name` | str                                    | yes        | —                  | must match a `catalog[].name` in the same file                                          | Selects which catalogue entry in `catalog:` backs this target selection.                                                           |
| `query`        | str (relative path)                    | no         | None               | `.sql` filename next to the survey YAML (mutually exclusive with `columns` + `filters`) | SQL query file executed per declination cell to extract candidate targets.                                                         |
| `ra_column`    | str                                    | yes        | —                  | remote column name                                                                      | Right ascension column name in the remote table or HATS catalogue (used for spatial sky-cell filtering).                           |
| `dec_column`   | str                                    | yes        | —                  | remote column name                                                                      | Declination column name in the remote table or HATS catalogue (used for spatial sky-cell filtering).                               |
| `name`         | str                                    | no         | `<prefix>.parquet` | filename ending in `.parquet`                                                           | Output artifact filename for the extracted table inside the stage directory.                                                       |
| `rescue_where` | str                                    | no         | None               | SQL boolean expression (photometric selections only)                                    | Relaxed SQL filter used during the photometric rescue stage for spectroscopic targets missed by the primary photometric selection. |
| `columns`      | mapping (remote_column -> local_alias) | no         | {}                 | `lsdb` adapter only; must include `<prefix>_id`, `<prefix>_ra`, `<prefix>_dec`          | Column projection and alias map for HATS/LSDB target extraction (used instead of a SQL `query`).                                   |
| `filters`      | list of `[column, operator, value]`    | no         | []                 | `lsdb` adapter only; operators `==`, `=`, `!=`, `<`, `<=`, `>`, `>=`, `in`, `not in`    | Parquet pushdown row filters applied during HATS/LSDB target extraction.                                                           |

### `provides.<prefix>` keys

| Key                | Type                            | Required   | Default   | Allowed values                                       | Description                                                               |
|--------------------|---------------------------------|------------|-----------|------------------------------------------------------|---------------------------------------------------------------------------|
| `filters`          | mapping (band -> SVO filter ID) | no         | {}        | lowercase band token -> `Facility/Instrument.Filter` | Passband SVO filter identifiers for the hosted survey prefix.             |
| `filter_variants`  | mapping                         | no         | None      | see Filter variant keys                              | Per-row filter override for the hosted survey prefix.                     |
| `vega_offsets`     | mapping (band -> float)         | no         | {}        | declared band tokens -> float                        | Additive `m_AB - m_Vega` offsets for the hosted survey prefix.            |
| `measurements`     | list of measurement mappings    | no         | []        | see Photometric measurement keys                     | Photometric measurement blocks carried for the hosted survey prefix.      |
| `detection_column` | str                             | no         | None      | column alias starting with `<provides_prefix>_`      | Column whose non-null value marks a row as detected in the hosted survey. |

### Crossmatch keys (`crossmatch:`)

| Key           | Type   | Required   | Default                                                             | Allowed values                                                                        | Description                                                                                                                                                                                                             |
|---------------|--------|------------|---------------------------------------------------------------------|---------------------------------------------------------------------------------------|-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `radius`      | angle  | yes        | —                                                                   | a number (arcseconds) or a string such as `1.5 arcsec`; above 0 and at most 60 arcsec | The survey’s match radius, the largest separation at which a catalogue object counts as a counterpart. It is a property of the survey’s depth and astrometry, so it lives here and not in an adapter’s transport knobs. |
| `radius_note` | str    | no         | None                                                                | non-empty string                                                                      | What the radius rests on (the instrument facts and where they were read). A test requires it of every file shipped under `defs/surveys/`; a user’s own file may omit it.                                                |
| `catalog`     | str    | no         | the enabled photometric catalog, else the enabled spectroscopic one | the `name` of an enabled `catalog` entry of this survey                               | Which catalog the radius and the strategy apply to.                                                                                                                                                                     |
| `moc`         | path   | no         | None                                                                | a MOC FITS file; a relative path is relative to the file that names it                | The survey’s footprint, for `--filter-moc`. Build it with `mosaics-crossmatch moc`; its existence is checked when a run uses it, not when the file loads.                                                               |
| `strategy`    | str    | no         | `upload`                                                            | `upload`, `extract` (refused for `crossid` and `lsdb`)                                | `upload` sends the positions to the archive; `extract` fetches the survey’s rows for the sky the input occupies and matches them here.                                                                                  |

### `catalog[].adapter_params.file` keys (`local` adapter)

| Key     | Type   | Required   | Default   | Allowed values                              | Description                                                                                           |
|---------|--------|------------|-----------|---------------------------------------------|-------------------------------------------------------------------------------------------------------|
| `url`   | str    | yes        | —         | HTTP/HTTPS URL to a FITS or Parquet table   | Archive URL of the bulk catalogue file downloaded once and converted to a local DuckDB/Parquet store. |
| `table` | str    | yes        | —         | lowercase SQL identifier `[a-z_][a-z0-9_]*` | Local table name under which SQL queries reference the converted catalogue file.                      |
<!-- /generated -->

---

## Adapter Parameters (`adapter_params`)

Each `catalog`, `cutout`, and `spectra` entry accepts an `adapter_params` mapping whose allowed keys depend on the chosen `adapter`.

<!-- generated:config-adapters -->

### `adl` (NSF NOIRLab Astro Data Lab TAP)

| Key                 | Type   | Required   | Default   | Description                                                                                       |
|---------------------|--------|------------|-----------|---------------------------------------------------------------------------------------------------|
| `table`             | str    | yes        | —         | Fully qualified TAP table name on Astro Data Lab (e.g. `gaia_dr3.gaia_source`, `allwise.source`). |
| `mode`              | str    | no         | `spatial` | Crossmatch join mode (`spatial` cone join or `relational` key join).                              |
| `key_column`        | str    | no         | None      | Local candidate column holding the pre-matched foreign key when `mode: relational`.               |
| `remote_key_column` | str    | no         | None      | Remote table primary key column matched against `key_column` when `mode: relational`.             |
| `remote_ra_column`  | str    | no         | `ra`      | Right ascension column in the remote table used for spatial cone joins.                           |
| `remote_dec_column` | str    | no         | `dec`     | Declination column in the remote table used for spatial cone joins.                               |
| `chunk_size`        | int    | no         | 50000     | Number of target positions uploaded per TAP crossmatch batch.                                     |
| `use_async`         | bool   | no         | false     | Submit queries as asynchronous UWS TAP jobs instead of synchronous requests.                      |
| `dialect`           | str    | no         | `top`     | Row-limiting SQL syntax (`top` for ADQL `SELECT TOP n`, `limit` for PostgreSQL `LIMIT n`).        |

### `crossid` (Astro Data Lab Cross-ID service)

| Key                 | Type   | Required   | Default   | Description                                              |
|---------------------|--------|------------|-----------|----------------------------------------------------------|
| `table`             | str    | yes        | —         | Remote table name queried through the Cross-ID endpoint. |
| `mode`              | str    | no         | `spatial` | Join mode (`spatial` or `relational`).                   |
| `key_column`        | str    | no         | None      | Local key column when `mode: relational`.                |
| `remote_key_column` | str    | no         | None      | Remote key column when `mode: relational`.               |
| `remote_ra_column`  | str    | no         | `ra`      | Remote right ascension column name.                      |
| `remote_dec_column` | str    | no         | `dec`     | Remote declination column name.                          |
| `chunk_size`        | int    | no         | 50000     | Positions uploaded per Cross-ID request chunk.           |
| `use_async`         | bool   | no         | false     | Whether to run queries asynchronously.                   |
| `dialect`           | str    | no         | `top`     | SQL row-limit dialect (`top` or `limit`).                |

### `tap` (Generic IVOA TAP service)

| Key                 | Type   | Required   | Default                                 | Description                                                                                                                                  |
|---------------------|--------|------------|-----------------------------------------|----------------------------------------------------------------------------------------------------------------------------------------------|
| `service`           | str    | yes        | —                                       | Base URL of the IVOA TAP endpoint (e.g. `http://tapvizier.cds.unistra.fr/TAPVizieR/tap`, `https://mast.stsci.edu/vo-tap/api/v0.1/hasp/tap`). |
| `table`             | str    | no         | None (read from SQL query)              | Remote TAP table name (quoted when required by VizieR, e.g. `"VII/250/2dfgrs"`).                                                             |
| `mode`              | str    | no         | `spatial`                               | Crossmatch join mode (`spatial` or `relational`).                                                                                            |
| `key_column`        | str    | no         | None                                    | Local key column when `mode: relational`.                                                                                                    |
| `remote_key_column` | str    | no         | None                                    | Remote key column when `mode: relational`.                                                                                                   |
| `remote_ra_column`  | str    | no         | `ra`                                    | Remote right ascension column name.                                                                                                          |
| `remote_dec_column` | str    | no         | `dec`                                   | Remote declination column name.                                                                                                              |
| `chunk_size`        | int    | no         | 50000                                   | Positions uploaded per TAP crossmatch batch.                                                                                                 |
| `use_async`         | bool   | no         | false                                   | Submit queries as asynchronous UWS TAP jobs.                                                                                                 |
| `dialect`           | str    | no         | `top`                                   | SQL row-limit dialect (`top` for ADQL `SELECT TOP n`, `limit` for `LIMIT n`).                                                                |
| `chunk_mib`         | float  | no         | 28.0 (the command line’s `--chunk-mib`) | Byte budget of one inline upload, in MiB, for a service that cannot take the default.                                                        |
| `footprint_table`   | str    | no         | None                                    | A table with one `s_region` polygon per sky patch, from which `mosaics-crossmatch moc` builds the survey’s footprint.                        |

### `adl-healpix` (NSF NOIRLab Astro Data Lab public TAP, matched by `nest4096` ranges)

| Key                    | Type   | Required   | Default                                                                   | Description                                                                                                                                            |
|------------------------|--------|------------|---------------------------------------------------------------------------|--------------------------------------------------------------------------------------------------------------------------------------------------------|
| `table`                | str    | yes        | —                                                                         | Fully qualified table on Astro Data Lab that carries a HEALPix NESTED index column (e.g. `gaia_dr3.gaia_source`, `allwise.source`, `ls_dr11.tractor`). |
| `service`              | str    | no         | `datalab` (the public TAP at `https://datalab.noirlab.edu/tap`, no login) | A full TAP endpoint or a shorthand of the TAP services, for a test archive.                                                                            |
| `remote_ra_column`     | str    | no         | found by UCD (`pos.eq.ra`), then by name                                  | Remote right ascension column name (`mean_fiber_ra` for `desi_dr1.zpix`).                                                                              |
| `remote_dec_column`    | str    | no         | found by UCD (`pos.eq.dec`), then by name                                 | Remote declination column name.                                                                                                                        |
| `index_column`         | str    | no         | `nest4096`                                                                | The table’s HEALPix NESTED index. Its values are checked against the returned positions on the first query of every run.                               |
| `index_order`          | int    | no         | 12 (the order of `nest4096`)                                              | The HEALPix order of `index_column`.                                                                                                                   |
| `max_rows_per_query`   | int    | no         | 50000                                                                     | Most rows one query may return. A query whose result reaches it is split and asked again, never truncated.                                             |
| `max_ranges_per_query` | int    | no         | 50                                                                        | Most index ranges one query may carry. The public TAP times out well above this (probed at 100 and 500).                                               |
| `rows_per_deg2`        | float  | no         | None (counted by a few `COUNT(*)` queries)                                | The table’s mean source density per square degree. When given, the planner sends no counting query.                                                    |

### `local` (Bulk FITS/Parquet file over local DuckDB)

| Key                 | Type                     | Required   | Default   | Description                                                                         |
|---------------------|--------------------------|------------|-----------|-------------------------------------------------------------------------------------|
| `file`              | mapping (`{url, table}`) | yes        | —         | Remote file URL and local DuckDB table name (see `local_file` keys).                |
| `remote_ra_column`  | str                      | yes        | —         | Right ascension column name in the downloaded table (e.g. `plug_ra`, `sup_ra`).     |
| `remote_dec_column` | str                      | yes        | —         | Declination column name in the downloaded table (e.g. `plug_dec`, `sup_dec`).       |
| `dialect`           | str                      | no         | `limit`   | SQL row-limit dialect (`limit` for DuckDB).                                         |
| `use_async`         | bool                     | no         | false     | Accepted for uniform adapter signature; local DuckDB queries always run in-process. |

### `nersc` (Legacy Survey DR11 sweep & tractor files at NERSC)

| Key                 | Type   | Required   | Default   | Description                                                                                 |
|---------------------|--------|------------|-----------|---------------------------------------------------------------------------------------------|
| `table`             | str    | yes        | —         | Virtual Legacy Survey table served from NERSC files (`ls_dr11.sweep` or `ls_dr11.tractor`). |
| `mode`              | str    | no         | `spatial` | Join mode (`spatial` or `relational`).                                                      |
| `key_column`        | str    | no         | None      | Local key column when `mode: relational`.                                                   |
| `remote_key_column` | str    | no         | None      | Remote key column when `mode: relational`.                                                  |
| `remote_ra_column`  | str    | no         | `ra`      | Right ascension column name (`ra`).                                                         |
| `remote_dec_column` | str    | no         | `dec`     | Declination column name (`dec`).                                                            |
| `chunk_size`        | int    | no         | 50000     | Batch size for chunked queries.                                                             |
| `use_async`         | bool   | no         | false     | Accepted for adapter compatibility; queries execute locally against cached Parquet slices.  |
| `dialect`           | str    | no         | `limit`   | SQL row-limit dialect (`limit` for DuckDB).                                                 |

### `lsst` (Rubin Observatory, Rubin Science Platform TAP, token required)

| Key                 | Type   | Required   | Default                                                            | Description                                                                                                              |
|---------------------|--------|------------|--------------------------------------------------------------------|--------------------------------------------------------------------------------------------------------------------------|
| `release`           | str    | no         | None (required unless `service` is given)                          | Data release: `dp2`, `dp1`, `dp02` (`https://data.lsst.cloud/api/tap`) or `dp03` (`https://data.lsst.cloud/api/ssotap`). |
| `table`             | str    | yes        | —                                                                  | Remote table, e.g. `dp2.Object`.                                                                                         |
| `service`           | str    | no         | the release’s endpoint                                             | A full TAP endpoint overriding the release’s, for a test archive or a non-standard service.                              |
| `remote_ra_column`  | str    | no         | found by UCD (`pos.eq.ra`), then by name                           | Remote right ascension column name.                                                                                      |
| `remote_dec_column` | str    | no         | found by UCD (`pos.eq.dec`), then by name                          | Remote declination column name.                                                                                          |
| `footprint_table`   | str    | no         | None                                                               | The release’s `CoaddPatches` table (`dp2.CoaddPatches`), from which `mosaics-crossmatch moc` builds the footprint.       |
| `use_async`         | bool   | no         | false (switches on by itself when the first upload chunk is large) | Submit every upload as a server-side job instead of a synchronous query.                                                 |
| `chunk_mib`         | float  | no         | 28.0 (the command line’s `--chunk-mib`)                            | Byte budget of one inline upload, in MiB. Rubin caps an upload at about 33 MiB.                                          |

### `lsdb` (HATS / HiPSCat partitioned Parquet catalogues via LSDB)

| Key                 | Type    | Required   | Default               | Description                                                                                         |
|---------------------|---------|------------|-----------------------|-----------------------------------------------------------------------------------------------------|
| `path`              | str     | yes        | —                     | HTTP or S3 URI of the HATS catalogue root (may contain `{band}` or `{BAND}` when `per_band: true`). |
| `margin_cache`      | str     | no         | None                  | URI of the HATS pre-computed margin cache table for boundary-safe spatial crossmatching.            |
| `storage_options`   | mapping | no         | {}                    | Filesystem options forwarded to `upath`/`fsspec` (e.g. `{'anon': True}` for public S3 buckets).     |
| `reshape`           | str     | no         | None                  | Optional post-read reshaping strategy for nested or multi-row HATS tables.                          |
| `per_band`          | bool    | no         | false                 | When `true`, opens one HATS catalogue per band by substituting `{band}`/`{BAND}` in `path`.         |
| `remote_ra_column`  | str     | no         | HATS metadata default | Override for the remote right ascension column name.                                                |
| `remote_dec_column` | str     | no         | HATS metadata default | Override for the remote declination column name.                                                    |
<!-- /generated -->

---

## Worked Examples

### 1. Photometric Crossmatch Survey (`skymapper4`-style)

A photometric survey matched locally via HATS (`adapter: lsdb`) in stage 3 of the `targets` pipeline and also declaring single-band HiPS cutouts. Its `crossmatch` section gives the one radius it is matched within, by the targets pipeline and by `mosaics-crossmatch` alike; a survey with an `lsdb` catalog must have one:

```yaml
survey_slug: skymapper4
name: SkyMapper DR4

catalog:
  - name: skymapper4
    type: photometric
    adapter: lsdb
    adapter_params:
      path: https://data.lsdb.io/hats/skymapper_dr4/catalog/sky_mapper_dr4
      margin_cache: https://data.lsdb.io/hats/skymapper_dr4/catalog/sky_mapper_dr4_5arcs

crossmatch:
  radius: 2 arcsec
  radius_note: >-
    Median seeing 2.6 arcsec in r; median offset from Gaia 0.13 arcsec (SkyMapper DR4
    release page).

catalog_filters:
  skymapper4: skymapper4_mag_psf_r <= 20.5
  skymapper4_note: 10 sigma per field, AB (Onken et al. 2024).

cutout:
  - adapter: hips
    adapter_params:
      hips_id: CDS/P/skymapper-R
    bands: [r]
    enabled: false

photometric_system:
  filters:
    g: SkyMapper/SkyMapper.g
    r: SkyMapper/SkyMapper.r
    i: SkyMapper/SkyMapper.i
  measurements:
    - system: ab_mag
      apertures: [psf]
  pixel_unit: unknown
  pixel_unit_note: SkyMapper HiPS tiles do not publish a global flux scale.

column_mapping:
  object_id: 'id'
  raj2000: 'ra'
  dej2000: 'dec'
  g_psf: 'mag_psf_g'
  e_g_psf: 'mag_psf_g_err'
  r_psf: 'mag_psf_r'
  e_r_psf: 'mag_psf_r_err'
  i_psf: 'mag_psf_i'
  e_i_psf: 'mag_psf_i_err'
```

### 2. Spectroscopic Redshift Survey (`sdss17`-style)

A spectroscopic survey contributing to stage-1 target selection (`target_selection`), stage-4 spectroscopic crossmatching (`catalog`), and 1D spectrum extraction (`spectra`):

```yaml
survey_slug: sdss17
name: Sloan Digital Sky Survey DR17

catalog:
  - name: sdss17
    type: spectroscopic
    adapter: local
    adapter_params:
      file:
        url: https://dr19.sdss.org/sas/dr19/spectro/sdss/redux/specObj-dr17.fits
        table: specobj
      remote_ra_column: plug_ra
      remote_dec_column: plug_dec
      dialect: limit

catalog_filters:
  sdss17: sdss17_z BETWEEN 0.0 AND 2.5
  sdss17_note: Restricts the catalogue to the training redshift range.

target_selection:
  catalog_name: sdss17
  query: spec_sdss_dr17.sql
  ra_column: plug_ra
  dec_column: plug_dec

spectra:
  - adapter: template
    adapter_params:
      url_template: sdss-legacy
    format: sdss
    arrays: [flux, error, wavelength, mask]

spectroscopic_system:
  fiber_diameter: per_row
  disperser: grism
  redshift_rank: 2
  flux_scale: 1.0
  flux_calibration: absolute
```

### 3. Cutout-Only Imaging Survey (`unions`-style)

A survey that declares only a `cutout` service and `photometric_system` without participating in catalogue crossmatching:

```yaml
survey_slug: unions
name: UNIONS DR5

cutout:
  - adapter: hips
    adapter_params:
      hips_id: CDS/P/UNIONS/DR5/r
    bands: [color]
    enabled: false

photometric_system:
  pixel_unit: unknown
  pixel_unit_note: Placeholder HiPS definition; disabled until tiles are registered.
```

### 4. Host Survey with `provides` (`ls11`-style)

The anchor survey (`ls11`) that supplies the base photometric sample in stage 2 of the `targets` pipeline and declares additional cross-survey photometry (`unwise`, `gaia3`) carried by the same table via `provides`:

```yaml
survey_slug: ls11
name: DESI Legacy Imaging Surveys DR11

catalog:
  - name: ls11
    type: photometric
    adapter: nersc
    adapter_params:
      table: ls_dr11.sweep
      dialect: limit
      use_async: false

catalog_filters:
  ls11: ls11_flux_z >= 0.631
  ls11_note: 5 sigma point source in z, AB nanomaggies.
  unwise: unwise_flux_w1 >= 8.3
  unwise_note: 5 sigma five-year coadds in W1, AB nanomaggies.

cutout:
  - adapter: lsv
    adapter_params:
      layer: ls-dr11
      bands: grz
    bands: [g, r, z]
  - adapter: nersc
    adapter_params:
      bands: grz
      data_release: dr11
    bands: [g, r, z]

photometric_system:
  filters:
    g: CTIO/DECam.g
    r: CTIO/DECam.r
    i: CTIO/DECam.i
    z: CTIO/DECam.z
  filter_variants:
    column: ls11_hemisphere
    cases:
      N:
        g: BOK/BASS.g
        r: BOK/BASS.r
        z: KPNO/MzLS.z
  measurements:
    - system: linear
      apertures: [default]
      scale: 3.631
      error_kind: ivar
  pixel_unit: nanomaggy
  pixel_unit_note: Legacy Survey coadds are nanomaggies by construction.

target_selection:
  catalog_name: ls11
  query: phot_ls_dr11_full.sql
  ra_column: ra
  dec_column: dec
  name: ls11.parquet
  rescue_where: brick_primary = 1

provides:
  unwise:
    filters:
      w1: WISE/WISE.W1
      w2: WISE/WISE.W2
    measurements:
      - system: linear
        apertures: [default]
        scale: 3.631
        error_kind: ivar
    detection_column: unwise_flux_w1

column_mapping:
  ls_id_dr11: 'id'
  ra: 'ra'
  dec: 'dec'
  flux_g: 'flux_g'
  flux_r: 'flux_r'
  flux_i: 'flux_i'
  flux_z: 'flux_z'
  flux_ivar_g: 'flux_g_ivar'
  flux_ivar_r: 'flux_r_ivar'
  flux_ivar_i: 'flux_i_ivar'
  flux_ivar_z: 'flux_z_ivar'
  unwise:                    # a `provides` key: these columns come back as `unwise_*`
    flux_w1: 'flux_w1'
    flux_w2: 'flux_w2'
    flux_ivar_w1: 'flux_w1_ivar'
    flux_ivar_w2: 'flux_w2_ivar'
```
