# Sloan Digital Sky Survey DR17 (`sdss17`)

<!-- generated:survey-page -->

The Sloan Digital Sky Survey Data Release 17 (SDSS DR17) is the final data release of SDSS-IV, encompassing all observations completed through the end of the fourth phase of the survey. It provides wide-area optical and near-infrared spectroscopy across both celestial hemispheres observed from Apache Point Observatory and Las Campanas Observatory.

In `mosaics`, `sdss17` is the primary spectroscopic survey. It drives stage-1 spectroscopic target selection (`mosaics-targets`), spectroscopic crossmatching, and 1D spectrum extraction (`mosaics-spectrum sdss17`). Target selection and catalogue crossmatching execute locally over the complete SDSS Science Archive Server (SAS) summary table `specObj-dr17.fits` (6.7 GB, 5,801,200 spectra, 133 columns), downloaded once, converted to a declination-sorted Parquet file, and queried cell by cell using DuckDB via `spec_sdss_dr17.sql`.

| Property                 | Value                                                                                                                                                                                                                                                                                                                                             |
|--------------------------|---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| What MOSAICS extracts    | Catalogue: `sdss17` (spectroscopic, `local`); Spectra: `template` (flux,error,wavelength,mask)                                                                                                                                                                                                                                                    |
| Archives                 | `dr19.sdss.org`, `data.sdss.org`                                                                                                                                                                                                                                                                                                                  |
| Bands / wavelength range | spectroscopy (disperser `grism`, fibre `per_row` arcsec, calibration `absolute`)                                                                                                                                                                                                                                                                  |
| Used by                  | `mosaics-targets` (spectroscopic selection), `mosaics-spectrum sdss17` / `full`, `mosaics-crossmatch`, `mosaics-query`                                                                                                                                                                                                                            |
| Crossmatch radius        | `2` arcsec. Fibre diameter 2 arcsec for the BOSS spectrograph and 3 arcsec for the SDSS one (DR17 spectroscopic basics); plug positions come from the imaging astrometry, 45-100 mas rms. The radius is the BOSS fibre; use 3 arcsec when the SDSS-I/II fibres matter. Whether plug_ra/plug_dec are the plate-hole coordinates was not confirmed. |

## Configuration

### The shipped file

Source: `defs/surveys/sdss17.yaml`

```yaml
# SDSS DR17 -- spectroscopy.
#
# Split out of `sdss20.yaml`, which now carries only SDSS cutouts -- the
# photometry is `sdss16.yaml`. A file declaring a single section takes the bare
# slug as its prefix, so these columns are `sdss17_*`.
#
# `sdss20` and `sdss17` describe the same telescope and overlap heavily on the
# sky. They are kept apart the same way `hasp` and `hst2` are: a data product
# is a survey here, not an observatory.

survey_slug: sdss17
name: Sloan Digital Sky Survey DR17

catalog:
  - name: sdss17
    type: spectroscopic
    adapter: local
    adapter_params:
      # A downloaded file, not an archive. The whole DR17 spectroscopic
      # catalogue is one FITS table on the SDSS Science Archive Server --
      # 6.7 GB, 5,801,200 spectra -- fetched once by the download/specobj step,
      # converted to Parquet sorted by PLUG_DEC and queried cell by cell with
      # DuckDB. It replaced Astro Data Lab's `sdss_dr17` schema, which answered
      # the same cuts with 3,944,198 rows; see the query's own header.
      file:
        url: https://dr19.sdss.org/sas/dr19/spectro/sdss/redux/specObj-dr17.fits
        table: specobj
      # The plugged fibre position: what the CAS publishes as specobjall.ra/dec.
      remote_ra_column: plug_ra
      remote_dec_column: plug_dec
      dialect: limit              # DuckDB spells a row cap LIMIT

# Crossmatch: the radius a table of positions is matched within, and what it rests on.
crossmatch:
  radius: 2 arcsec
  radius_note: >-
    Fibre diameter 2 arcsec for the BOSS spectrograph and 3 arcsec for the SDSS one (DR17
    spectroscopic basics); plug positions come from the imaging astrometry, 45-100 mas rms.
    The radius is the BOSS fibre; use 3 arcsec when the SDSS-I/II fibres matter. Whether
    plug_ra/plug_dec are the plate-hole coordinates was not confirmed.

catalog_filters:
  # Applied in production only -- see catalog_filters in the module docstring
  # of mosaics/pipelines/targets/production.py. A null or NaN redshift fails the
  # same as one outside the range: BETWEEN is false for either under DuckDB's
  # three-valued logic, and the caller treats a false or null filter alike.
  sdss17: sdss17_z BETWEEN 0.0 AND 2.5
  sdss17_note: >-
    Restricts the catalogue to the redshift range the training targets are
    drawn over; a row outside it keeps its candidacy but loses its SDSS
    spectroscopic columns, as if this survey had not matched it.

target_selection:
  catalog_name: sdss17
  query: spec_sdss_dr17.sql
  ra_column: plug_ra
  dec_column: plug_dec

spectra:
  # The query selects plate, mjd, fiberid, run2d and instrument, which is
  # everything these templates address a spectrum on the SAS by.
  - adapter: template
    adapter_params:
      url_template: sdss-legacy
    format: sdss
    arrays: [flux, error, wavelength, mask]

spectroscopic_system:
  # DR17 genuinely spans two spectrographs (3.0 and 2.0 arcsec).
  # PER_ROW_FIBERS in mosaics/pipelines/targets/spec.py derives the diameter
  # from `sdss17_instrument`, which is what `per_row` points at.
  fiber_diameter: per_row
  # Both eras are prism-grating-prism: SDSS-I/II a surface-relief grism, BOSS
  # a VPH grism (Smee et al. 2013, AJ 146, 32). Unlike the fibre, one
  # constant is exact.
  disperser: grism
  redshift_rank: 2
  flux_scale: 1.0             # native 1e-17 cgs already
  flux_calibration: absolute
```

Target-selection query (`defs/surveys/spec_sdss_dr17.sql`):

```sql
-- SDSS DR17 spectroscopy: scientifically reliable spectra.
--
-- Provider: local (DuckDB, over the SAS specObj file).
--   `specobj` is specObj-dr17.fits from the SDSS Science Archive Server,
--   https://dr19.sdss.org/sas/dr19/spectro/sdss/redux/specObj-dr17.fits --
--   6.7 GB, 5,801,200 spectra, 133 columns. The targets pipeline's
--   download/specobj step fetches it once, keeps its scalar columns as a
--   Parquet file sorted by PLUG_DEC, and deletes the FITS; this query then
--   runs once per sky cell against that file, with the cell's declination
--   band appended to the WHERE clause below.
--
-- It replaced Astro Data Lab's `sdss_dr17` schema (specobjall joined to
-- platex), where these cuts returned 3,944,198 rows -- 1,113,689 `SDSS` and
-- 2,830,509 `BOSS`. The file is those two tables folded into one, under the
-- spectro pipeline's own column names rather than the CAS's:
--
--   specobjall.ra, .dec       -> PLUG_RA, PLUG_DEC
--   specobjall.zErr           -> Z_ERR
--   specobjall.sciencePrimary -> SPECPRIMARY  (best version of the spectrum
--                                              at this location)
--   specobjall.snMedian       -> SN_MEDIAN_ALL
--   platex.plateQuality       -> PLATEQUALITY, carried on every spectrum row
--
-- DuckDB matches identifiers case-insensitively, so they are written in
-- lowercase here.
--
-- Retrieval contract for `mosaics.cli.spectrum sdss17`: plate, mjd, fiberid and
-- run2d address a spectrum on the SAS, and the specobjid is the resume key.
-- The file spans two eras that the SAS keeps in two different trees, and
-- `instrument` is what tells them apart -- which is why it is selected. Split
-- the catalogue and run twice into the same dataset:
--
--   spectrum sdss17 -t sdss.parquet -o sdss.lance --id-col sdss17_id \
--     --url-template sdss-legacy      # instrument = 'SDSS', the 3" fibres
--   spectrum sdss17 -t sdss.parquet -o sdss.lance --id-col sdss17_id --resume \
--     --url-template sdss-eboss       # instrument = 'BOSS', the 2" fibres
--
-- sdss-legacy is DR20's own spectro/sdss tree. sdss-eboss reads from DR17,
-- because DR20's boss tree holds only SDSS-V products.
--
-- The 3" SDSS and 2" BOSS spectrographs are carried as an explicit
-- fiber_diameter rather than silently mixed: PER_ROW_FIBERS in
-- mosaics/pipelines/targets/spec.py derives it from `sdss17_instrument`.
-- `survey` and `programname` are the coarser provenance tags and are kept as
-- weak labels alongside class/subclass.
--
-- Three things about this table:
--
-- * SPECOBJID is a decimal string of an unsigned 64-bit number. It is wrapped
--   onto the signed range with the same bits -- the convention the CAS route
--   returned -- so a handful of ids are negative and every id is still unique,
--   which is all the resume key needs.
-- * Its integers and floats are 32-bit. They are cast to BIGINT and DOUBLE, the
--   types the CAS route stored.
-- * Its strings are fixed-width in the FITS file; the conversion strips the
--   padding, so `class != 'SKY'` compares what it appears to.

SELECT
  CAST(
    CASE
      WHEN CAST(specobjid AS HUGEINT) >= 9223372036854775808
        THEN CAST(specobjid AS HUGEINT) - 18446744073709551616
      ELSE CAST(specobjid AS HUGEINT)
    END AS BIGINT
  )                                   AS sdss17_id,
  plug_ra                             AS sdss17_ra,
  plug_dec                            AS sdss17_dec,
  -- retrieval keys, aliased to the exact names the URL templates interpolate
  CAST(plate AS BIGINT)               AS sdss17_plate,
  CAST(mjd AS BIGINT)                 AS sdss17_mjd,
  CAST(fiberid AS BIGINT)             AS sdss17_fiberid,
  run2d                               AS sdss17_run2d,
  -- provenance and weak labels. `instrument` is doubly load-bearing: it picks
  -- the URL template above, and PER_ROW_FIBERS reads it for fiber_diameter.
  instrument                          AS sdss17_instrument,
  survey                              AS sdss17_survey,
  programname                         AS sdss17_program,
  class                               AS sdss17_class,
  subclass                            AS sdss17_subclass,
  class_noqso                         AS sdss17_class_noqso,
  subclass_noqso                      AS sdss17_subclass_noqso,
  -- redshift and its reliability
  CAST(z AS DOUBLE)                   AS sdss17_z,
  CAST(z_err AS DOUBLE)               AS sdss17_z_err,
  CAST(zwarning AS BIGINT)            AS sdss17_z_flag,
  CAST(specprimary AS BIGINT)         AS sdss17_science_primary,
  -- signal to noise over the whole spectrum
  CAST(sn_median_all AS DOUBLE)       AS sdss17_snr,
  platequality                        AS sdss17_quality,
  'sdss17'                            AS "sample"
FROM specobj
WHERE
  -- S1: the plate as a whole was judged good
  platequality = 'good'
  -- S2: the redshift fit raised no warning of any kind
  AND zwarning = 0
  -- S3: one spectrum per object, the one the pipeline considers definitive
  AND specprimary = 1
  -- S4: sky fibres and unassigned targets carry no object
  AND class IS NOT NULL
  AND class != 'SKY'
```

### Schema of this file

| Key present            | Type                                       | Required   | Default   | Meaning                                                                                                                                                                                             |
|------------------------|--------------------------------------------|------------|-----------|-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `survey_slug`          | str                                        | yes        | —         | Canonical lowercase identifier for the survey and default column prefix.                                                                                                                            |
| `name`                 | str                                        | yes        | —         | Human-readable survey and data-release name used in logs, plans, and reports.                                                                                                                       |
| `catalog`              | list of catalog entries                    | no         | []        | Remote or local catalogues queried for target extraction or crossmatching.                                                                                                                          |
| `crossmatch`           | mapping                                    | no         | None      | How a table of positions is matched against this survey. Required for a survey with an `lsdb` catalog; `crossmatch` null removes it, and a survey without it takes no part in `mosaics-crossmatch`. |
| `catalog_filters`      | mapping (prefix -> SQL expression or note) | no         | {}        | Post-hoc DuckDB SQL quality/depth/redshift cuts applied during production (nulls failing survey columns without dropping the row).                                                                  |
| `target_selection`     | mapping                                    | no         | None      | How this survey seeds or joins the candidate table in the targets pipeline.                                                                                                                         |
| `spectra`              | list of spectra entries                    | no         | []        | 1D spectrum download backends (`url`, `template`, `sparcl`, `desi-healpix`).                                                                                                                        |
| `spectroscopic_system` | mapping                                    | no         | None      | Spectrograph fibre diameter, disperser, redshift tiebreak rank, and flux calibration facts.                                                                                                         |

### Adapter parameters

**Catalogue `sdss17` (`adapter: local`):**

| Parameter           | Allowed Type             | Default   | Description                                                                         |
|---------------------|--------------------------|-----------|-------------------------------------------------------------------------------------|
| `file`              | mapping (`{url, table}`) | —         | Remote file URL and local DuckDB table name (see `local_file` keys).                |
| `remote_ra_column`  | str                      | —         | Right ascension column name in the downloaded table (e.g. `plug_ra`, `sup_ra`).     |
| `remote_dec_column` | str                      | —         | Declination column name in the downloaded table (e.g. `plug_dec`, `sup_dec`).       |
| `dialect`           | str                      | `limit`   | SQL row-limit dialect (`limit` for DuckDB).                                         |
| `use_async`         | bool                     | false     | Accepted for uniform adapter signature; local DuckDB queries always run in-process. |
- **Spectra (`adapter: template`, format `sdss`):** configured `adapter_params`: `url_template='sdss-legacy'`.

### Common changes

- Adjust or relax the production redshift filter: the default production filter restricts matches to `sdss17_z BETWEEN 0.0 AND 2.5`. In an entrypoint configuration, remove or modify this cut with:
  ```yaml
  overrides:
    surveys/sdss17:
      catalog_filters:
        sdss17: null
  ```
- Extract 1D spectra for both spectrograph eras: because DR17 spans both legacy SDSS and BOSS spectrographs residing in distinct SAS directory trees, run `mosaics-spectrum` in two sequential passes writing to the same Lance dataset:
  ```console
  $ mosaics-spectrum sdss17 -t sdss.parquet -o sdss.lance --id-col sdss17_id --url-template sdss-legacy
  $ mosaics-spectrum sdss17 -t sdss.parquet -o sdss.lance --id-col sdss17_id --resume --url-template sdss-eboss
  ```
- Change target selection criteria: replace `target_selection.query: spec_sdss_dr17.sql` with a custom SQL file in an entrypoint configuration to modify plate quality or target classification constraints.

### Commands

```console
$ uv run -m mosaics.cli.spectrum sdss17 -t test/objects_sample_1k.csv -o sdss17_spectra.lance
$ uv run -m mosaics.crossmatch.cli run --survey surveys/sdss17 -i test/objects_sample_1k.csv -o sdss17_matched.parquet
$ uv run -m mosaics.cli.targets --config defs/targets.yaml -o ./targets_run --dec-range 0 1 --plan plan.yaml
```

## Data served by the archive

- Architectural role: SDSS spectroscopy is maintained independently under `sdss17`, separating it from `sdss16` (photometry) and `sdss20` (cutouts) so that each data product retains distinct provenance, prefixes, and archive adapters.
- Local execution with DuckDB: The pipeline queries the local `specObj-dr17.fits` table converted to Parquet rather than querying remote TAP endpoints (such as Astro Data Lab’s `sdss_dr17` schema), eliminating network query timeouts and delivering 3,944,198 clean science targets.
- Target selection cuts in `spec_sdss_dr17.sql`:
  - `platequality = 'good'`: The spectroscopic plate as a whole was judged good.
  - `zwarning = 0`: Redshift measurement completed without warnings or pipeline flags.
  - `specprimary = 1`: The definitive, highest signal-to-noise spectrum at this sky position.
  - `class IS NOT NULL AND class != 'SKY'`: Removes sky calibration fibres and unassigned targets.
- Dual spectrographs and fibre sizes: DR17 combines spectra from two distinct instruments:
  - SDSS legacy spectrograph: $3.0^{\prime\prime}$ fibre diameter, covering 3800–9200 Å (vacuum), using a surface-relief grism.
  - BOSS spectrograph: $2.0^{\prime\prime}$ fibre diameter, covering 3600–10400 Å (vacuum), using a volume phase holographic (VPH) grism (Smee et al. 2013).
  - The fibre diameter is resolved per row from `sdss17_instrument`. Both eras provide resolution $R \sim 1500 - 2500$ with absolute flux calibration ($10^{-17}\ \mathrm{erg\ s^{-1}\ cm^{-2}\ \text{Å}^{-1}}$).
- Redshift precedence: SDSS DR17 holds redshift tiebreak rank `2` in MOSAICS candidate merging.
- Identifier mapping: `SPECOBJID` is an unsigned 64-bit integer wrapped to signed 64-bit `BIGINT` matching CAS convention, ensuring unique keys across all rows.

### Catalogue: `specobj`

SDSS DR17 optical spectroscopic summary catalogue (specObj-dr17.fits).

**Service URL:** `https://dr19.sdss.org/sas/dr19/spectro/sdss/redux/specObj-dr17.fits` | **Published rows:** 5,801,200

*Showing 100 of 133 total archive columns (all MOSAICS-mapped columns plus primary table fields; full 133-column schema is stored in `docs/_data/sources/sdss17/fits_sdss17_specobj.json`).*

| Column                | Type   | Unit        | UCD               | Description                                                      |
|-----------------------|--------|-------------|-------------------|------------------------------------------------------------------|
| `SURVEY`              | `6A`   | `—`         | `—`               | Survey name                                                      |
| `INSTRUMENT`          | `4A`   | `—`         | `—`               | Instrument used (SDSS or BOSS spectrograph)                      |
| `CHUNK`               | `16A`  | `—`         | `—`               | Chunk name                                                       |
| `PROGRAMNAME`         | `27A`  | `—`         | `—`               | Program name                                                     |
| `PLATERUN`            | `16A`  | `—`         | `—`               | Plate drill run name                                             |
| `PLATEQUALITY`        | `8A`   | `—`         | `—`               | —                                                                |
| `PLATESN2`            | `E`    | `—`         | `—`               | Overall signal-to-noise-squared measure for plate (only set for  |
| `DEREDSN2`            | `E`    | `—`         | `—`               | Dereddened signal-to-noise-squared measure for plate (only set f |
| `LAMBDA_EFF`          | `E`    | `—`         | `—`               | —                                                                |
| `BLUEFIBER`           | `J`    | `—`         | `—`               | Set to 1 if this hole was designated a “blue fiber”, 0 if design |
| `ZOFFSET`             | `E`    | `microns`   | `—`               | Washer thickness used (for backstopping BOSS quasar targets, so  |
| `SNTURNOFF`           | `E`    | `—`         | `—`               | Signal to noise measure for MS turnoff stars on plate (-9999 if  |
| `NTURNOFF`            | `J`    | `—`         | `—`               | —                                                                |
| `SPECPRIMARY`         | `B`    | `—`         | `—`               | —                                                                |
| `SPECSDSS`            | `B`    | `—`         | `—`               | —                                                                |
| `SPECLEGACY`          | `B`    | `—`         | `—`               | —                                                                |
| `SPECSEGUE`           | `B`    | `—`         | `—`               | —                                                                |
| `SPECSEGUE1`          | `B`    | `—`         | `—`               | —                                                                |
| `SPECSEGUE2`          | `B`    | `—`         | `—`               | —                                                                |
| `SPECBOSS`            | `B`    | `—`         | `—`               | —                                                                |
| `BOSS_SPECOBJ_ID`     | `J`    | `—`         | `—`               | —                                                                |
| `SPECOBJID`           | `22A`  | `—`         | `meta.id;src`     | Unique database ID based on PLATE, MJD, FIBERID, RUN2D           |
| `FLUXOBJID`           | `19A`  | `—`         | `meta.id;src`     | Object ID of photoObj match (flux-based)                         |
| `BESTOBJID`           | `19A`  | `—`         | `meta.id;src`     | Object ID of photoObj match (position-based)                     |
| `TARGETOBJID`         | `22A`  | `—`         | `meta.id;src`     | Object ID of original target                                     |
| `PLATEID`             | `20A`  | `—`         | `meta.id;instr`   | Database ID of Plate                                             |
| `NSPECOBS`            | `I`    | `—`         | `—`               | —                                                                |
| `FIRSTRELEASE`        | `4A`   | `—`         | `—`               | Name of first release this object was associated with            |
| `RUN2D`               | `7A`   | `—`         | `—`               | 2D Reduction version of spectrum                                 |
| `RUN1D`               | `7A`   | `—`         | `—`               | 1D Reduction version of spectrum                                 |
| `DESIGNID`            | `J`    | `—`         | `—`               | Design ID number                                                 |
| `CX`                  | `D`    | `—`         | `pos.cartesian.x` | x of Normal unit vector in J2000                                 |
| `CY`                  | `D`    | `—`         | `pos.cartesian.y` | y of Normal unit vector in J2000                                 |
| `CZ`                  | `D`    | `—`         | `pos.cartesian.z` | z of Normal unit vector in J2000                                 |
| `XFOCAL`              | `E`    | `mm`        | `—`               | X focal plane position (+RA direction)                           |
| `YFOCAL`              | `E`    | `mm`        | `—`               | Y focal plane position (+Dec direction)                          |
| `SOURCETYPE`          | `25A`  | `—`         | `—`               | For Legacy, SEGUE-2 and BOSS science targets, type of object tar |
| `TARGETTYPE`          | `8A`   | `—`         | `—`               | Nature of target: SCIENCE, STANDARD, or SKY                      |
| `THING_ID_TARGETING`  | `K`    | `—`         | `—`               | thing_id value from the version of resolve from which the target |
| `THING_ID`            | `J`    | `—`         | `—`               | Unique identifier from global resolve                            |
| `PRIMTARGET`          | `J`    | `—`         | `—`               | target selection information at plate design, primary science se |
| `SECTARGET`           | `J`    | `—`         | `—`               | target selection information at plate design, secondary/qa/calib |
| `LEGACY_TARGET1`      | `J`    | `—`         | `—`               | for Legacy program, target selection information at plate design |
| `LEGACY_TARGET2`      | `J`    | `—`         | `—`               | for Legacy program target selection information at plate design, |
| `SPECIAL_TARGET1`     | `K`    | `—`         | `—`               | for Special program target selection information at plate design |
| `SPECIAL_TARGET2`     | `K`    | `—`         | `—`               | for Special program target selection information at plate design |
| `SEGUE1_TARGET1`      | `J`    | `—`         | `—`               | SEGUE-1 target selection information at plate design, primary sc |
| `SEGUE1_TARGET2`      | `J`    | `—`         | `—`               | SEGUE-1 target selection information at plate design, secondary/ |
| `SEGUE2_TARGET1`      | `J`    | `—`         | `—`               | SEGUE-2 target selection information at plate design, primary sc |
| `SEGUE2_TARGET2`      | `J`    | `—`         | `—`               | SEGUE-2 target selection information at plate design, secondary/ |
| `MARVELS_TARGET1`     | `J`    | `—`         | `—`               | —                                                                |
| `MARVELS_TARGET2`     | `J`    | `—`         | `—`               | —                                                                |
| `BOSS_TARGET1`        | `K`    | `—`         | `—`               | BOSS target selection information at plate                       |
| `BOSS_TARGET2`        | `K`    | `—`         | `—`               | —                                                                |
| `EBOSS_TARGET0`       | `K`    | `—`         | `—`               | EBOSS target selection information, for SEQUELS plates           |
| `EBOSS_TARGET1`       | `K`    | `—`         | `—`               | EBOSS target selection information, for eBOSS plates             |
| `EBOSS_TARGET2`       | `K`    | `—`         | `—`               | EBOSS target selection information, for TDSS, SPIDERS, ELG, etc. |
| `EBOSS_TARGET_ID`     | `K`    | `—`         | `—`               | EBOSS unique target identifier for every spectroscopic target,   |
| `ANCILLARY_TARGET1`   | `K`    | `—`         | `—`               | BOSS ancillary science target selection information at plate des |
| `ANCILLARY_TARGET2`   | `K`    | `—`         | `—`               | BOSS ancillary target selection information at plate design      |
| `SPECTROGRAPHID`      | `I`    | `—`         | `—`               | which spectrograph (1,2)                                         |
| `PLATE`               | `J`    | `—`         | `—`               | Plate number                                                     |
| `TILE`                | `J`    | `—`         | `—`               | Tile number                                                      |
| `MJD`                 | `J`    | `days`      | `—`               | MJD of observation                                               |
| `FIBERID`             | `J`    | `—`         | `—`               | Fiber ID                                                         |
| `OBJID`               | `5J`   | `—`         | `—`               | —                                                                |
| `PLUG_RA`             | `D`    | `—`         | `—`               | —                                                                |
| `PLUG_DEC`            | `D`    | `—`         | `—`               | —                                                                |
| `CLASS`               | `6A`   | `—`         | `—`               | Spectroscopic class (GALAXY, QSO, or STAR)                       |
| `SUBCLASS`            | `21A`  | `—`         | `—`               | Spectroscopic subclass                                           |
| `Z`                   | `E`    | `—`         | `—`               | Final Redshift                                                   |
| `Z_ERR`               | `E`    | `—`         | `—`               | —                                                                |
| `RCHI2`               | `E`    | `—`         | `—`               | Reduced chi-squared of best fit                                  |
| `DOF`                 | `J`    | `—`         | `—`               | Degrees of freedom in best fit                                   |
| `RCHI2DIFF`           | `E`    | `—`         | `—`               | Difference in reduced chi-squared between best and second best f |
| `TFILE`               | `24A`  | `—`         | `—`               | File name of best fit template source                            |
| `TCOLUMN`             | `10J`  | `—`         | `—`               | —                                                                |
| `NPOLY`               | `J`    | `—`         | `—`               | Number of polynomial terms used in the fit                       |
| `THETA`               | `10E`  | `—`         | `—`               | —                                                                |
| `VDISP`               | `E`    | `—`         | `—`               | —                                                                |
| `VDISP_ERR`           | `E`    | `—`         | `—`               | —                                                                |
| `VDISPZ`              | `E`    | `—`         | `—`               | —                                                                |
| `VDISPZ_ERR`          | `E`    | `—`         | `—`               | —                                                                |
| `VDISPCHI2`           | `E`    | `—`         | `—`               | —                                                                |
| `VDISPNPIX`           | `E`    | `—`         | `—`               | —                                                                |
| `VDISPDOF`            | `J`    | `—`         | `—`               | —                                                                |
| `WAVEMIN`             | `E`    | `Angstroms` | `—`               | Minimum observed (vacuum) wavelength                             |
| `WAVEMAX`             | `E`    | `Angstroms` | `—`               | Maximum observed (vacuum) wavelength                             |
| `WCOVERAGE`           | `E`    | `—`         | `—`               | Coverage in wavelength, in units of log10 wavelength             |
| `ZWARNING`            | `J`    | `—`         | `—`               | Bitmask of warning values; 0 means all is well                   |
| `SN_MEDIAN_ALL`       | `E`    | `—`         | `—`               | —                                                                |
| `SN_MEDIAN`           | `5E`   | `—`         | `—`               | —                                                                |
| `CHI68P`              | `E`    | `—`         | `—`               | 68-th percentile value of abs(chi) of the best-fit synthetic spe |
| `FRACNSIGMA`          | `10E`  | `—`         | `—`               | —                                                                |
| `FRACNSIGHI`          | `10E`  | `—`         | `—`               | —                                                                |
| `FRACNSIGLO`          | `10E`  | `—`         | `—`               | —                                                                |
| `SPECTROFLUX`         | `5E`   | `—`         | `—`               | —                                                                |
| `SPECTROFLUX_IVAR`    | `5E`   | `—`         | `—`               | —                                                                |
| `SPECTROSYNFLUX`      | `5E`   | `—`         | `—`               | —                                                                |
| `SPECTROSYNFLUX_IVAR` | `5E`   | `—`         | `—`               | —                                                                |

### Images

No image cutout backends are configured for this survey.

### Spectra

- **Fibre diameter:** `per_row` | **Disperser:** `grism` | **Redshift rank:** `2` | **Flux calibration:** `absolute` | **Flux scale:** `1.0`
- **Adapter `template`:** decoder format `sdss`, output parallel arrays `['flux', 'error', 'wavelength', 'mask']`, params `{'url_template': 'sdss-legacy'}`.
  - **hdus:** HDU 1 (`COADD` binary table: `flux`, `loglam`, `ivar`, `and_mask`). Wavelength is `10**loglam`.
  - **wavelength_range_angstrom:** 3800 - 9200 A (SDSS legacy, 3.0 arcsec) or 3600 - 10400 A (BOSS, 2.0 arcsec), vacuum
  - **resolution:** R ~ 1500 - 2500
  - **flux_calibration:** absolute (`10^-17 erg s^-1 cm^-2 A^-1`)
  - **arrays:** [‘flux’, ‘error’, ‘wavelength’, ‘mask’]

## References and acknowledgement

- Abdurro’uf et al. (2022), *The Seventeenth Data Release of the Sloan Digital Sky Surveys: Complete Release of MaNGA, MaStar, and APOGEE-2 Data*, *ApJS*, 259, 35 (`doi:10.3847/1538-4365/ac4414`).
- Smee, S. A. et al. (2013), *The Multi-object, Fiber-fed Spectrographs for the Sloan Digital Sky Survey and the Baryon Oscillation Spectroscopic Survey*, *AJ*, 146, 32 (`doi:10.1088/0004-6256/146/2/32`).
- SDSS Science Archive Server: `https://dr19.sdss.org/sas/dr19/spectro/sdss/redux/specObj-dr17.fits`.
- SDSS 1D spectrum data model: `https://data.sdss.org/datamodel/files/BOSS_SPECTRO_REDUX/RUN2D/spectra/lite/PLATE4/spec.html`.

Funding for the Sloan Digital Sky Survey IV has been provided by the Alfred P. Sloan Foundation, the U.S. Department of Energy Office of Science, and the Participating Institutions. SDSS-IV is managed by the Astrophysical Research Consortium for the Participating Institutions of the SDSS Collaboration including the Brazilian Participation Group, the Carnegie Institution for Science, Carnegie Mellon University, the Chilean Participation Group, the French Participation Group, Harvard-Smithsonian Center for Astrophysics, Instituto de Astrofísica de Canarias, The Johns Hopkins University, Kavli Institute for the Physics and Mathematics of the Universe (IPMU) / University of Tokyo, the Lawrence Berkeley National Laboratory, Leibniz Institut für Astrophysik Potsdam (AIP), Max-Planck-Institut für Astronomie (MPIA Heidelberg), Max-Planck-Institut für Astrophysik (MPA Garching), Max-Planck-Institut für Extraterrestrische Physik (MPE), National Astronomical Observatories of China, New Mexico State University, New York University, University of Notre Dame, Observatório Nacional / MCTI, The Ohio State University, Pennsylvania State University, Shanghai Astronomical Observatory, United Kingdom Participation Group, Universidad Nacional Autónoma de México, University of Arizona, University of Colorado Boulder, University of Oxford, University of Portsmouth, University of Utah, University of Virginia, University of Washington, University of Wisconsin, Vanderbilt University, and Yale University.

## Provenance of this page

| Source ID             | Kind             | Service / Endpoint                                                                             | Query or URL                                                                                      | Retrieved (UTC)        |
|-----------------------|------------------|------------------------------------------------------------------------------------------------|---------------------------------------------------------------------------------------------------|------------------------|
| `fits_sdss17_specobj` | `fits_header`    | `https://dr19.sdss.org/sas/dr19/spectro/sdss/redux/specObj-dr17.fits`                          | `GET https://dr19.sdss.org/sas/dr19/spectro/sdss/redux/specObj-dr17.fits (Range: bytes=0-262143)` | `2026-10-18T16:37:26Z` |
| `spectra_sdss17`      | `spectrum_model` | `https://data.sdss.org/datamodel/files/BOSS_SPECTRO_REDUX/RUN2D/spectra/lite/PLATE4/spec.html` | `https://data.sdss.org/datamodel/files/BOSS_SPECTRO_REDUX/RUN2D/spectra/lite/PLATE4/spec.html`    | `2026-10-18T16:37:26Z` |
<!-- /generated -->
