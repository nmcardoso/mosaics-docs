# HST Advanced Spectral Products (HASP) (`hasp`)

<!-- generated:survey-page -->

The Hubble Advanced Spectral Products (HASP) program, hosted at the Mikulski Archive for Space Telescopes (MAST), provides automated coadditions of one-dimensional ultraviolet and optical spectra obtained with the Cosmic Origins Spectrograph (COS) and the Space Telescope Imaging Spectrograph (STIS) aboard the Hubble Space Telescope (HST). The spectral coadds span 1150 Å to 10270 Å with resolving powers ranging from $R \sim 1,500$ to $45,000$ depending on the grating combination. HASP covers targeted pointings across the entire sky wherever HST performed spectroscopic observations with COS and STIS.

In MOSAICS, HASP provides the corpus’s only ultraviolet spectroscopy, crucial for resolving the Lyman-$\alpha$ forest, interstellar and intergalactic absorption features, and hot-star and accretion-disc continua below 3000 Å. It participates in target selection via `mosaics-targets` using an ADQL query against MAST TAP, and in 1D spectral extraction via `mosaics-spectrum hasp`, which downloads and decodes the coadded `*_cspec.fits` products.

| Property                 | Value                                                                                                                                                                                                                                                                                       |
|--------------------------|---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| What MOSAICS extracts    | Catalogue: `hasp` (spectroscopic, `tap`); Spectra: `url` (flux,error,wavelength,mask)                                                                                                                                                                                                       |
| Archives                 | mast (`https://mast.stsci.edu/vo-tap/api/v0.1/caom/`), `archive.stsci.edu`                                                                                                                                                                                                                  |
| Bands / wavelength range | spectroscopy (calibration `absolute`)                                                                                                                                                                                                                                                       |
| Used by                  | `mosaics-targets` (spectroscopic selection), `mosaics-spectrum hasp` / `full`, `mosaics-crossmatch`, `mosaics-query`                                                                                                                                                                        |
| Crossmatch radius        | `3` arcsec. PROVISIONAL. The COS primary science aperture is 2.5 arcsec in diameter (COS Instrument Handbook, Sect. 3.1); the STIS slit widths were not looked up, and nothing was found on which coordinates the HASP products carry or how accurate they are. The radius covers COS only. |

## Configuration

### The shipped file

Source: `defs/surveys/hasp.yaml`

```yaml
# HST spectral coadds from HASP -- spectroscopy.
#
# A different thing from `hst2`, which carries both the Hubble Source Catalog
# photometry and the Hubble Legacy Archive imaging.

survey_slug: hasp
name: HST Advanced Spectral Products (HASP)

catalog:
  - name: hasp
    type: spectroscopic
    adapter: tap
    adapter_params:
      service: mast
      table: dbo.obspointing
      dialect: top

# No catalog_filters: unlike the other six spectroscopic surveys, HASP has no
# redshift column at all -- it coadds whichever HST spectra fall on a target,
# a different population from the redshift surveys, which is also why it is
# absent from spectroscopic_system.redshift_rank below. There is nothing here
# for a `{prefix}_z BETWEEN ...` cut to test.

# Crossmatch: the radius a table of positions is matched within, and what it rests on.
crossmatch:
  radius: 3 arcsec
  radius_note: >-
    PROVISIONAL. The COS primary science aperture is 2.5 arcsec in diameter (COS Instrument
    Handbook, Sect. 3.1); the STIS slit widths were not looked up, and nothing was found on
    which coordinates the HASP products carry or how accurate they are. The radius covers
    COS only.
  # The selection joins two tables and groups them, which cannot be rewritten as an upload
  # join. The query's header counts about 46,293 rows when it was written (no COUNT(*) was
  # run for this setting), few enough to fetch over the cells the input occupies.
  strategy: extract

target_selection:
  catalog_name: hasp
  query: spec_hst_hasp.sql
  # Both joined tables have an s_ra, so the appended spatial filter has to
  # say which one it means; the query's own header spells this out.
  ra_column: o.s_ra
  dec_column: o.s_dec

spectra:
  - adapter: url
    adapter_params:
      url_column: hst_access_url
    format: hasp
    # HASP publishes an ERROR column but no mask of its own. `mask` is
    # computed from the flux's finiteness and its signal-to-noise, so it
    # is listed here even though the archive has nothing named that.
    arrays: [flux, error, wavelength, mask]

spectroscopic_system:
  # An HST slit, not a fibre. Null means unknown and deliberately not
  # guessed, as for euclidq1.
  fiber_diameter: null
  # Deliberately absent rather than null: a HASP coadd abuts whichever COS
  # and STIS dispersers observed the target, so it is resolved per row --
  # see PER_ROW_DISPERSERS in mosaics/pipelines/targets/spec.py -- and a
  # survey-wide constant would erase that. Note the difference from
  # fiber_diameter, where null is a *value* meaning "no such thing exists".
  disperser: null
  # Absent from the redshift tiebreak: HASP coadds are a different
  # population from the redshift surveys above.
  redshift_rank: null
  flux_scale: 1.0e17          # native erg/s/cm2/Angstrom
  flux_calibration: absolute
```

Target-selection query (`defs/surveys/spec_hst_hasp.sql`):

```sql
-- HST COS and STIS spectra from the Hubble Advanced Spectral Products (HASP).
--
-- Provider: tap --service mast (MAST CAOM, ADQL).
--   uv run -m mosaics.cli.query sql --provider tap --service mast \
--     --path sql/hst_hasp_spectra.sql --ra-col o.s_ra --dec-col o.s_dec \
--     -o hasp.parquet
--
-- Note the *qualified* --ra-col: both joined tables have an s_ra, so the
-- spatial filter the splitter appends is ambiguous unless it says which one.
--
-- Retrieval contract for `mosaics.cli.spectrum hst`: access_url is the direct
-- download and is also the resume key, since it is what uniquely identifies a
-- product.
--   uv run -m mosaics.cli.spectrum hst --targets hasp.parquet \
--     --url-column access_url --id-col access_url -o hasp.lance
--
-- This is the corpus's only ultraviolet spectroscopy. GALEX contributes two UV
-- broad bands but nothing else resolves the Lyman-alpha forest, the
-- interstellar and intergalactic absorption lines, or the hot-star and
-- accretion-disc continua below 3000 A.
--
-- The join is the whole point of this query. dbo.obspointing carries
-- provenance_name, which is the only way to identify a HASP product, but has
-- no access_url; ivoa.obscore carries access_url but not provenance_name.
-- Going the other way -- scanning obscore for access_url LIKE '%cspec%' --
-- is a full-table scan that returns 504. Filtering access_url *after* the
-- join is cheap, because the join has already narrowed the rows to HASP.
--
-- The GROUP BY is not cosmetic. CAOM sometimes registers one product under
-- two observation ids (a target and its '-corr' reprocessing, say), so a
-- plain join returns the same access_url twice -- 531 duplicates in 46,293
-- rows when this was written. That makes access_url unusable as a resume key,
-- which mosaics rejects outright rather than working around. One row per
-- product is the honest shape of this catalogue.
--
-- It does not remove every duplicate, and cannot. A spatially split run
-- groups within each declination slice, and the two registrations of one
-- product can carry slightly different coordinates and so land in different
-- slices -- 45 of 45,807 rows survived this way. Drop them before retrieval:
--
--   frame.drop_duplicates('access_url')
--
-- A HASP spectrum can abut several gratings with discontinuous resolution, so
-- instrument_name and filters are carried as tokens; filters holds the
-- semicolon-separated grating list, e.g. 'G130M;G750L'.
--
-- HASP fluxes are absolutely calibrated in erg/s/cm2/A, so unlike LAMOST and
-- 6dFGS nothing needs rescaling downstream.

SELECT
  c.access_url                  AS hasp_url,
  MIN(o.obs_id)                 AS hasp_id,
  MIN(o.s_ra)                   AS hasp_ra,
  MIN(o.s_dec)                  AS hasp_dec,
  MIN(o.target_name)            AS hasp_name,
  MIN(o.target_classification)  AS hasp_class,
  MIN(o.instrument_name)        AS hasp_instrument,
  MIN(o.filters)                AS hasp_filters,
  MIN(o.proposal_id)            AS hasp_proposal_id,
  -- MAX(o.t_exptime)              AS hasp_exptime,
  -- MIN(o.em_min)                 AS hasp_em_min,
  -- MAX(o.em_max)                 AS hasp_em_max,
  MAX(o.calib_level)            AS hasp_calib_level,
  'hasp'                        AS "sample"
FROM dbo.obspointing AS o
JOIN ivoa.obscore AS c ON o.obs_id = c.obs_id
WHERE
  o.obs_collection = 'HST'
  AND o.provenance_name = 'HASP'
  AND o.dataproduct_type = 'spectrum'
  AND o.intentType = 'science'
  AND o.t_exptime > 0
  AND c.access_url LIKE '%_cspec.fits'
GROUP BY c.access_url
```

### Schema of this file

| Key present            | Type                    | Required   | Default   | Meaning                                                                                                                                                                                             |
|------------------------|-------------------------|------------|-----------|-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `survey_slug`          | str                     | yes        | —         | Canonical lowercase identifier for the survey and default column prefix.                                                                                                                            |
| `name`                 | str                     | yes        | —         | Human-readable survey and data-release name used in logs, plans, and reports.                                                                                                                       |
| `catalog`              | list of catalog entries | no         | []        | Remote or local catalogues queried for target extraction or crossmatching.                                                                                                                          |
| `crossmatch`           | mapping                 | no         | None      | How a table of positions is matched against this survey. Required for a survey with an `lsdb` catalog; `crossmatch` null removes it, and a survey without it takes no part in `mosaics-crossmatch`. |
| `target_selection`     | mapping                 | no         | None      | How this survey seeds or joins the candidate table in the targets pipeline.                                                                                                                         |
| `spectra`              | list of spectra entries | no         | []        | 1D spectrum download backends (`url`, `template`, `sparcl`, `desi-healpix`).                                                                                                                        |
| `spectroscopic_system` | mapping                 | no         | None      | Spectrograph fibre diameter, disperser, redshift tiebreak rank, and flux calibration facts.                                                                                                         |

### Adapter parameters

**Catalogue `hasp` (`adapter: tap`):**

| Parameter           | Allowed Type   | Default                                 | Description                                                                                                                                  |
|---------------------|----------------|-----------------------------------------|----------------------------------------------------------------------------------------------------------------------------------------------|
| `service`           | str            | —                                       | Base URL of the IVOA TAP endpoint (e.g. `http://tapvizier.cds.unistra.fr/TAPVizieR/tap`, `https://mast.stsci.edu/vo-tap/api/v0.1/hasp/tap`). |
| `table`             | str            | None (read from SQL query)              | Remote TAP table name (quoted when required by VizieR, e.g. `"VII/250/2dfgrs"`).                                                             |
| `mode`              | str            | `spatial`                               | Crossmatch join mode (`spatial` or `relational`).                                                                                            |
| `key_column`        | str            | None                                    | Local key column when `mode: relational`.                                                                                                    |
| `remote_key_column` | str            | None                                    | Remote key column when `mode: relational`.                                                                                                   |
| `remote_ra_column`  | str            | `ra`                                    | Remote right ascension column name.                                                                                                          |
| `remote_dec_column` | str            | `dec`                                   | Remote declination column name.                                                                                                              |
| `chunk_size`        | int            | 50000                                   | Positions uploaded per TAP crossmatch batch.                                                                                                 |
| `use_async`         | bool           | false                                   | Submit queries as asynchronous UWS TAP jobs.                                                                                                 |
| `dialect`           | str            | `top`                                   | SQL row-limit dialect (`top` for ADQL `SELECT TOP n`, `limit` for `LIMIT n`).                                                                |
| `chunk_mib`         | float          | 28.0 (the command line’s `--chunk-mib`) | Byte budget of one inline upload, in MiB, for a service that cannot take the default.                                                        |
| `footprint_table`   | str            | None                                    | A table with one `s_region` polygon per sky patch, from which `mosaics-crossmatch moc` builds the survey’s footprint.                        |
- **Spectra (`adapter: url`, format `hasp`):** configured `adapter_params`: `url_column='hst_access_url'`.

### Common changes

- Restrict or shift the spatial footprint in `mosaics-targets` by passing declination bounds with `--dec-range <min> <max>`.
- Disable HASP spectral extraction in an entrypoint configuration by setting `overrides[surveys/hasp].spectra: []`.
- Filter downloaded spectra downstream by instrument grating combinations using the tokenised `hasp_filters` column (e.g., isolating specific COS far-UV gratings such as `G130M` or STIS optical gratings).

### Commands

```console
$ uv run -m mosaics.cli.spectrum hasp -t test/objects_sample_1k.csv -o hasp_spectra.lance
$ uv run -m mosaics.crossmatch.cli run --survey surveys/hasp -i test/objects_sample_1k.csv -o hasp_matched.parquet
$ uv run -m mosaics.cli.targets --config defs/targets.yaml -o ./targets_run --dec-range 0 1 --plan plan.yaml
```

## Data served by the archive

- **Ambiguous coordinates in ADQL:** Both `dbo.obspointing` (aliased as `o`) and `ivoa.obscore` (aliased as `c`) carry `s_ra` and `s_dec` columns. Spatial slice filters appended by `mosaics-targets` must explicitly qualify the coordinates (`o.s_ra`, `o.s_dec`) to avoid ambiguous-column SQL errors.
- **Table join rationale and timeouts:** `dbo.obspointing` carries the `provenance_name = 'HASP'` flag needed to isolate HASP products but lacks direct access URLs, whereas `ivoa.obscore` contains `access_url` but lacks provenance. Attempting to scan `ivoa.obscore` directly for `access_url LIKE '%cspec%'` results in full-table scans that timeout with HTTP 504. Joining `dbo.obspointing` with `ivoa.obscore` on `obs_id` narrows rows cheaply before filtering.
- **Product deduplication and resume keys:** MAST CAOM sometimes registers a single product under multiple observation IDs (e.g. initial target and reprocessed observations). The target query groups by `c.access_url` so each unique FITS product forms a single row and serves as an unambiguous resume key for `mosaics-spectrum hasp`. When runs are split into declination slices, observations with slightly differing coordinates at slice boundaries can occasionally produce residual duplicates across slices that should be deduplicated on `access_url` prior to retrieval.
- **No redshift attribute:** Unlike cosmological redshift surveys, HASP coadds do not derive or report redshift values. HASP consequently carries no `catalog_filters` on redshift, and `redshift_rank` is set to null.
- **Flux calibration and quality masks:** Fluxes in `*_cspec.fits` are absolutely calibrated in physical units of $\text{erg}\,\text{s}^{-1}\,\text{cm}^{-2}\,\text{Å}^{-1}$ (scaled in MOSAICS by $1.0\times 10^{17}$). While HASP publishes an `ERROR` column, it does not publish an intrinsic data quality mask; quality mask arrays are synthesized during decoding based on flux finiteness and signal-to-noise.

### Catalogue: `dbo.ObsPointing`

MAST portal view of observations marked as science

**Service URL:** `https://mast.stsci.edu/vo-tap/api/v0.1/caom/`

| Column                  | Type      | Unit   | UCD                          | Description                                |
|-------------------------|-----------|--------|------------------------------|--------------------------------------------|
| `dataproduct_type`      | `char`    | `n.n`  | `meta.code.class`            | type of product                            |
| `calib_level`           | `int`     | `n.n`  | `meta.code;obs.calib`        | calibration level (-1 through 5)           |
| `obs_collection`        | `char`    | `n.n`  | `meta.id`                    | name of data collection                    |
| `obs_id`                | `char`    | `n.n`  | `meta.id;meta.main`          | internal dataset identifier                |
| `target_name`           | `char`    | `n.n`  | `meta.id;src`                | name of target                             |
| `s_ra`                  | `double`  | `deg`  | `pos.eq.ra;meta.main`        | ra of central coordinates                  |
| `s_dec`                 | `double`  | `deg`  | `pos.eq.dec;meta.main`       | dec of central coordinates                 |
| `t_min`                 | `double`  | `d`    | `time.start;obs.exposure`    | observation start mjd                      |
| `t_max`                 | `double`  | `d`    | `time.end;obs.exposure`      | observation end mjd                        |
| `t_exptime`             | `double`  | `s`    | `time.duration;obs.exposure` | total exposure time                        |
| `wavelength_region`     | `char`    | `n.n`  | `em.energy`                  | em region                                  |
| `filters`               | `char`    | `n.n`  | `instr.bandpass`             | em region                                  |
| `em_min`                | `double`  | `m`    | `em.wl;stat.min`             | minimum wavelength                         |
| `em_max`                | `double`  | `m`    | `em.wl;stat.max`             | maximum wavelength                         |
| `target_classification` | `char`    | `n.n`  | `src.class`                  | target classification from proposal        |
| `obs_title`             | `char`    | `n.n`  | `meta.title`                 | proposal title                             |
| `t_obs_release`         | `double`  | `n.n`  | `time.release;meta.dataset`  | release date mjd                           |
| `instrument_name`       | `char`    | `n.n`  | `instr`                      | instrument name                            |
| `proposal_pi`           | `char`    | `n.n`  | `meta.id.PI`                 | proposal principal investigator name       |
| `proposal_id`           | `char`    | `n.n`  | `obs.param`                  | collection specific proposal identifier    |
| `proposal_type`         | `char`    | `n.n`  | `obs.param`                  | project that created observation           |
| `provenance_name`       | `char`    | `n.n`  | `obs.param`                  | process that created plane                 |
| `project`               | `char`    | `n.n`  | `obs.param`                  | project that created plane                 |
| `sequence_number`       | `int`     | `n.n`  | `meta.number`                | sequence number assigned by observatory    |
| `s_region`              | `char`    | `n.n`  | `pos.outline;obs.field`      | region bounded by observation              |
| `jpegurl`               | `char`    | `n.n`  | `meta.ref.url`               | url to download the data preview           |
| `dataurl`               | `char`    | `n.n`  | `meta.ref.url`               | url to download the data                   |
| `datarights`            | `char`    | `n.n`  | `meta.code.status`           | current data access rights                 |
| `mtflag`                | `boolean` | `n.n`  | `meta.code.status`           | flag for moving target                     |
| `srcden`                | `double`  | `n.n`  | `meta.number`                | estimated source density per square degree |
| `intenttype`            | `char`    | `n.n`  | `meta.code`                  | science/calibration purpose                |
| `obsid`                 | `long`    | `n.n`  | `meta.record`                | internal observation id                    |
| `objid`                 | `long`    | `n.n`  | `meta.record`                | internal object id                         |

### Images

No image cutout backends are configured for this survey.

### Spectra

- **Fibre diameter:** `None` | **Disperser:** `None` | **Redshift rank:** `None` | **Flux calibration:** `absolute` | **Flux scale:** `1e+17`
- **Adapter `url`:** decoder format `hasp`, output parallel arrays `['flux', 'error', 'wavelength', 'mask']`, params `{'url_column': 'hst_access_url'}`.
  - **hdus:** Binary table HDU 1 (`WAVELENGTH`, `FLUX`, `ERROR`, `DQ`).
  - **wavelength_range_angstrom:** 1150 - 10270 A (depends on COS/STIS grating combination)
  - **resolution:** R ~ 1500 - 45000 (mixed gratings)
  - **flux_calibration:** absolute (scaled by `1e17` to `10^-17 erg s^-1 cm^-2 A^-1`)
  - **arrays:** [‘flux’, ‘error’, ‘wavelength’, ‘mask’]

## References and acknowledgement

- Debes, J. et al. (2024), *The Hubble Advanced Spectral Products (HASP) Program*, STScI Instrument Science Report COS 2024-01 / STIS 2024-01.
- HASP High-Level Science Product portal: [https://archive.stsci.edu/hlsp/hasp](https://archive.stsci.edu/hlsp/hasp)
- MAST CAOM TAP service: [https://mast.stsci.edu/vo-tap/api/v0.1/caom/](https://mast.stsci.edu/vo-tap/api/v0.1/caom/)

Based on observations obtained with the NASA/ESA Hubble Space Telescope, retrieved from the Mikulski Archive for Space Telescopes (MAST) at the Space Telescope Science Institute (STScI). STScI is operated by the Association of Universities for Research in Astronomy, Inc. under NASA contract NAS 5-26555.

## Provenance of this page

| Source ID                  | Kind             | Service / Endpoint                             | Query or URL                                                                                                        | Retrieved (UTC)        |
|----------------------------|------------------|------------------------------------------------|---------------------------------------------------------------------------------------------------------------------|------------------------|
| `catalog_hasp_obspointing` | `tap_table`      | `https://mast.stsci.edu/vo-tap/api/v0.1/caom/` | `SELECT column_name, datatype, unit, ucd, description FROM TAP_SCHEMA.columns WHERE table_name = 'dbo.ObsPointing'` | `2026-10-18T16:31:25Z` |
| `spectra_hasp`             | `spectrum_model` | `https://archive.stsci.edu/hlsp/hasp`          | `https://archive.stsci.edu/hlsp/hasp`                                                                               | `2026-10-18T16:31:25Z` |
<!-- /generated -->
