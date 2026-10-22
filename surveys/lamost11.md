# LAMOST DR11 MRS (`lamost11`)

<!-- generated:survey-page -->

The Large Sky Area Multi-Object Fiber Spectroscopic Telescope (LAMOST, also known as the Guoshoujing Telescope) Data Release 11 (DR11 v2.0) Medium-Resolution Spectroscopic survey (MRS) delivers optical multi-fiber spectra observed from the Xinglong Observatory in Hebei, China. Operating with 4,000 optical fibers across a $5^\circ$ field of view, LAMOST MRS employs Volume Phase Holographic (VPH) gratings to observe targets simultaneously in blue (4900–5400 Å) and red (6300–6800 Å) wavelength channels at a resolving power of $R \approx 7,500$ with $3.3''$ diameter fibres.

In MOSAICS, `lamost11` provides spectroscopic target selection via `mosaics-targets` querying the TAP service table `public.med_catalogue` at `https://www.lamost.org/dr11/v2.0/voservice/tap`, and 1D spectral extraction via `mosaics-spectrum lamost11`, which downloads and splices coadded blue (`COADD_B`) and red (`COADD_R`) FITS extensions into unified calibrated 1D spectra. Because LAMOST MRS reports radial velocities rather than fitted cosmological redshifts, its derived redshift carries `redshift_rank: 4`.

| Property                 | Value                                                                                                                                                                                                                                                                                        |
|--------------------------|----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| What MOSAICS extracts    | Catalogue: `lamost11` (spectroscopic, `tap`); Spectra: `template` (flux,error,wavelength,mask)                                                                                                                                                                                               |
| Archives                 | lamost (`https://www.lamost.org/dr11/v2.0/voservice/tap`)                                                                                                                                                                                                                                    |
| Bands / wavelength range | spectroscopy (disperser `grating`, fibre `3.3` arcsec, calibration `relative`)                                                                                                                                                                                                               |
| Used by                  | `mosaics-targets` (spectroscopic selection), `mosaics-spectrum lamost11` / `full`, `mosaics-crossmatch`, `mosaics-query`                                                                                                                                                                     |
| Crossmatch radius        | `3` arcsec. Fibre diameter 3.3 arcsec on the sky (Zhao et al. 2012, arXiv:1206.3569); fibre positioning accuracy “less than 1.5 arcsec on average” (Luo et al. 2015, arXiv:1505.01570). Both papers describe the instrument and DR1, not DR11 or the medium- resolution survey specifically. |

## Configuration

### The shipped file

Source: `defs/surveys/lamost11.yaml`

```yaml
# LAMOST DR11 medium-resolution survey -- spectroscopy.

survey_slug: lamost11
name: LAMOST DR11 MRS

catalog:
  - name: lamost11
    type: spectroscopic
    adapter: tap
    adapter_params:
      service: lamost
      table: public.med_catalogue
      dialect: top

# Crossmatch: the radius a table of positions is matched within, and what it rests on.
crossmatch:
  radius: 3 arcsec
  radius_note: >-
    Fibre diameter 3.3 arcsec on the sky (Zhao et al. 2012, arXiv:1206.3569); fibre
    positioning accuracy "less than 1.5 arcsec on average" (Luo et al. 2015,
    arXiv:1505.01570). Both papers describe the instrument and DR1, not DR11 or the medium-
    resolution survey specifically.

catalog_filters:
  # See sdss17.yaml for the general rule this and the other five redshift
  # surveys share. `lamost11_z` is derived from `lamost11_rv_lasp0` at
  # extraction time -- see `normalise_redshift` in
  # mosaics/pipelines/targets/spec.py -- rather than published directly, but it
  # is a real column by the time production runs.
  lamost11: lamost11_z BETWEEN 0.0 AND 2.5
  lamost11_note: >-
    Restricts the catalogue to the redshift range the training targets are
    drawn over; a row outside it keeps its candidacy but loses its LAMOST
    spectroscopic columns, as if this survey had not matched it.

target_selection:
  catalog_name: lamost11
  query: spec_lamost_dr11_mrs.sql
  ra_column: ra
  dec_column: dec

spectra:
  - adapter: template
    adapter_params:
      url_template: lamost-dr11
    format: lamost
    arrays: [flux, error, wavelength, mask]

spectroscopic_system:
  # A constant its own SQL could not carry, for the same reason as DESI.
  fiber_diameter: 3.3
  disperser: grating          # VPH gratings (Hou et al. 2018, SPIE 10702)
  # A velocity converted to a redshift rather than a fitted one, so it sits
  # below the three surveys that fit theirs.
  redshift_rank: 4
  # MRS reports three radial velocities per spectrum -- the blue arm, the red
  # arm, and the LASP pipeline's combined value -- so there is nothing to
  # infer and the primary is named. The other two stay in the table.
  velocity_columns: [lamost11_rv_lasp0, lamost11_rv_lasp0_err]
  # LAMOST writes -9999 where it has no measurement. Converting that to a
  # redshift gives z = -1, a real-looking number in a physically impossible
  # place, so it is masked before the conversion rather than after.
  null_sentinel: -9999.0
  # Relative calibration: the fluxes are in the right units and the wrong
  # overall normalisation, and fixing that needs broad-band photometry the
  # decoder has no access to. Leave the numbers alone.
  flux_scale: null
  flux_calibration: relative
```

Target-selection query (`defs/surveys/spec_lamost_dr11_mrs.sql`):

```sql
-- LAMOST DR11 v2.0 medium-resolution spectra (MRS).
--
-- Provider: tap --service lamost (ADQL).
--   uv run -m mosaics.cli.query sql --provider tap --service lamost \
--     --path sql/lamost_dr11_mrs.sql --ra-col ra --dec-col dec -o lamost.parquet
--
-- Retrieval contract for `mosaics.cli.spectrum lamost`: obsid alone addresses a
-- spectrum, via https://www.lamost.org/dr11/v2.0/medspectrum/fits/{obsid}.
-- lmjd, planid, spid and fiberid are the fibre address, kept for provenance.
--
-- One row per spectrum, not per arm. med_catalogue holds a separate row for
-- the blue and red arms of the same observation -- same obsid, mobsid
-- suffixed B or R -- while the downloaded file contains *both*, which
-- read_lamost concatenates. Selecting both rows would therefore fetch every
-- file twice and store each spectrum twice, so this keeps the blue row only.
--
-- The report phrases the S/N cut as an either/or (snrg >= 10 OR snri >= 10)
-- because a red object can be faint in the blue and still carry a usable red
-- spectrum. Expressing that needs the red arm's snr on the same row, and the
-- obvious self-join on obsid does not return within five minutes against this
-- server's 24M rows. The cut is therefore applied to the blue arm alone,
-- which is the conservative direction: it drops some usable red spectra but
-- admits no bad ones. To recover them, run the query again with band = 'R'
-- and concatenate -- obsid still deduplicates the two.
--
-- Two things to know before using this catalogue.
--
-- LAMOST flux calibration is *relative*, not absolute. These spectra must be
-- rescaled to broad-band photometry before being mixed with DESI or SDSS. The
-- Gaia magnitudes are selected here so that rescaling has something to work
-- with without a second crossmatch.
--
-- This server's TAP_SCHEMA.columns.description values are misaligned with
-- their column_name -- descriptions belong to unrelated columns -- so read
-- column meanings from the data, never from the schema browser. It also
-- labels a computed column 'double precision', which is not a VOTable
-- datatype, so pyvo rejects the whole response: no SELECT here may contain a
-- literal. That is why there is no fiber_diameter column, although every
-- LAMOST fibre is 3.3"; add the constant downstream, as for DESI.
--
-- It is also why this query, alone among the spec_*.sql files, selects no
-- `sample` literal either: the server types a string literal as 'text', which
-- is not a VOTable datatype, and pyvo then rejects the entire response with
-- "Unknown datatype 'text' on field '__sample_'". The targets pipeline assigns
-- `sample` in pandas for every survey, so nothing downstream depends on it.
--
-- med_catalogue carries no pipeline class/subclass: MRS is a stellar-focused
-- survey and classification lives in the LASP products. The executive
-- report's "class non-Unknown" cut is therefore expressed as
-- fibertype = 'Obj', which is the same intent -- a real target, not a sky or
-- calibration fibre. Note the value is mixed case.

SELECT
  b.obsid                 AS lamost11_id, 
  b.ra                    AS lamost11_ra, 
  b.dec                   AS lamost11_dec,
  -- fibre address, for provenance
  b.lmjd                  AS lamost11_lmjd, 
  b.lmjm                  AS lamost11_lmjm, 
  b.planid                AS lamost11_planid, 
  b.spid                  AS lamost11_spid, 
  b.fiberid               AS lamost11_fiberid, 
  b.uid                   AS lamost11_uid,
  b.designation           AS lamost11_designation, 
  b.subproject            AS lamost11_program,
  -- b.band                  AS lamost11_band, 
  b.snr                   AS lamost11_snr, 
  b.fibermask             AS lamost11_fibermask,
  b.moon_flg              AS lamost11_moon_flg,
  -- radial velocities from the LASP pipeline, the closest thing MRS has to a
  -- redshift; -9999 is this catalogue's null
  b.rv_b0                 AS lamost11_rv_b0, 
  b.rv_b0_err             AS lamost11_rv_b0_err, 
  b.rv_r0                 AS lamost11_rv_r0, 
  b.rv_r0_err             AS lamost11_rv_r0_err, 
  b.rv_lasp0              AS lamost11_rv_lasp0, 
  b.rv_lasp0_err          AS lamost11_rv_lasp0_err
FROM public.med_catalogue AS b
WHERE
  -- S0: one row per spectrum -- the coadded blue arm
  b.band = 'B'
  AND b.coadd = 1
  -- S1: a real target, not a sky, flat or arc fibre
  AND b.fibertype = 'Obj'
  -- S2: signal to noise floor, on the blue arm; see the note above
  AND b.snr >= 10
  -- S3: bad pixels are nulled from ANDMASK/ORMASK at decode time rather than
  -- rejecting whole spectra, so only a wholly unusable arm is cut here.
  AND b.bad_b = 0
  AND b.bad_r = 0
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

**Catalogue `lamost11` (`adapter: tap`):**

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
- **Spectra (`adapter: template`, format `lamost`):** configured `adapter_params`: `url_template='lamost-dr11'`.

### Common changes

- Relax or remove the default post-query redshift filter in your entrypoint YAML:
  ```yaml
  overrides:
    surveys/lamost11:
      catalog_filters:
        lamost11: null
  ```

  The shipped filter restricts matches to `lamost11_z BETWEEN 0.0 AND 2.5` (derived from the LASP radial velocity `lamost11_rv_lasp0`); setting it to `null` allows high-velocity or peculiar stars to retain their spectroscopic attributes.
- Disable LAMOST MRS target selection or spectrum fetching by setting `overrides[surveys/lamost11].catalog: []` or `overrides[surveys/lamost11].spectra: []`.
- Recover red-arm-only detections by modifying the target selection SQL to filter on `band = 'R'` instead of `band = 'B'`, deduplicating on `obsid`.

### Commands

```console
$ uv run -m mosaics.cli.spectrum lamost11 -t test/objects_sample_1k.csv -o lamost11_spectra.lance
$ uv run -m mosaics.crossmatch.cli run --survey surveys/lamost11 -i test/objects_sample_1k.csv -o lamost11_matched.parquet
$ uv run -m mosaics.cli.targets --config defs/targets.yaml -o ./targets_run --dec-range 0 1 --plan plan.yaml
```

## Data served by the archive

- **TAP schema description misalignment:** On the upstream LAMOST TAP server, column descriptions in `TAP_SCHEMA.columns.description` are misaligned with their corresponding `column_name` (for example, `rv_b0` is described as effective temperature). Column semantics must be interpreted according to official survey release documentation rather than the TAP schema browser.
- **Non-standard VOTable datatypes:** The LAMOST TAP server types computed numerical columns as `'double precision'` and literal strings as `'text'`. Neither is an accepted VOTable datatype, causing standard VO client libraries to reject responses containing literals. Consequently, the target query `spec_lamost_dr11_mrs.sql` excludes literal constants (such as sample identifiers or fibre diameter), and survey constants such as the 3.3 arcsec fibre diameter are injected downstream.
- **Arm deduplication:** The upstream `med_catalogue` stores separate rows for the blue (`band = 'B'`) and red (`band = 'R'`) arms of a single observation under the same `obsid` (distinguished by `mobsid`). Because the downloaded FITS file contains both arms and is concatenated into a single spectrum, selecting both rows would download each file twice. The query selects `band = 'B'` and `coadd = 1` to ensure one row per physical observation.
- **Signal-to-noise floor:** A conservative threshold `snr >= 10` is applied on the blue arm (`band = 'B'`). Red sources that are faint in the blue arm may fail this cut even if their red spectrum is high quality; such sources can be captured with a secondary query on `band = 'R'`.
- **Radial velocity null sentinels:** LAMOST records unmeasured velocities as `-9999.0`. Converting this uncaught into a redshift produces $z \approx -1$; MOSAICS explicitly masks the `-9999.0` sentinel before converting `lamost11_rv_lasp0` to redshift.
- **Relative flux calibration:** LAMOST MRS spectra carry relative flux calibration rather than absolute flux densities. Spectra must be rescaled against broad-band photometry (such as Gaia DR3 $G$, $G_{BP}$, $G_{RP}$) before combining them with absolutely calibrated spectra from DESI or SDSS.

### Catalogue: `public.med_catalogue`

LAMOST MRS General Catalogue

**Service URL:** `https://www.lamost.org/dr11/v2.0/voservice/tap`

| Column             | Type     | Unit   | UCD               | Description                                                                                                                   |
|--------------------|----------|--------|-------------------|-------------------------------------------------------------------------------------------------------------------------------|
| `gaia_g_mean_mag`  | `double` | `—`    | `—`               | The “phot_g_mean_mag” field of Gaia DR3 catalog                                                                               |
| `fibertype`        | `char`   | `—`    | `—`               | Fiber type of target [Obj, Sky, F-std, Unused, PosErr, Dead]                                                                  |
| `gaia_bp_mean_mag` | `double` | `—`    | `—`               | The “phot_bp_mean_mag” field of Gaia DR3 catalog                                                                              |
| `dec`              | `double` | `—`    | `POS_EQ_DEC_MAIN` | Declination from input catalog (degree)                                                                                       |
| `rv_b1`            | `double` | `—`    | `—`               | Surface gravity given by the LASP (dex)                                                                                       |
| `rv_b1_err`        | `double` | `—`    | `—`               | Surface gravity uncertainty given by the LASP (dex)                                                                           |
| `rv_b0`            | `double` | `—`    | `—`               | Effective temperature given by the LASP (K)                                                                                   |
| `rv_b0_err`        | `double` | `—`    | `—`               | Effective temperature uncertainty given by the LASP (K)                                                                       |
| `rv_r0`            | `double` | `—`    | `—`               | Metallicity uncertainty given by the LASP (dex)                                                                               |
| `rv_r1`            | `double` | `—`    | `—`               | Uncertainty of vsini_el given by the LASP                                                                                     |
| `rv_r1_err`        | `double` | `—`    | `—`               | Radial velocity of B band spectra measured with 483 selected KURUCZ synthetic templates (km/s)                                |
| `rv_r_flag`        | `int`    | `—`    | `—`               | Uncertainty of rv_b0 measured by the method LASP used (km/s)                                                                  |
| `rv_br0_err`       | `double` | `—`    | `—`               | Uncertainty of rv_b1 measured by the error propagation (km/s)                                                                 |
| `rv_br1_err`       | `double` | `—`    | `—`               | Radial velocity of R band spectra measured with 483 selected KURUCZ synthetic templates (km/s)                                |
| `rv_br_flag`       | `int`    | `—`    | `—`               | Uncertainty of rv_r0 measured by the method LASP used (km/s)                                                                  |
| `rv_lasp0`         | `double` | `—`    | `—`               | Radial velocity after zero-point correction of rv_r0 (km/s)                                                                   |
| `rv_lasp1`         | `double` | `—`    | `—`               | Radial velocity flag for R band spectra                                                                                       |
| `rv_lasp1_err`     | `double` | `—`    | `—`               | Radial velocity of B + R spectra measured with 483 selected KURUCZ synthetic templates (km/s)                                 |
| `fibermask`        | `int`    | `—`    | `—`               | Radial velocity after zero-point correction of rv_br0 (km/s)                                                                  |
| `bad_b`            | `int`    | `—`    | `—`               | Uncertainty of rv_br1 measured by the error propagation (km/s)                                                                |
| `lunardate`        | `int`    | `—`    | `—`               | Uncertainty of rv_lasp0 also given by the LASP (km/s)                                                                         |
| `lmjm`             | `long`   | `—`    | `—`               | Local modified Julian minute                                                                                                  |
| `dec_obs`          | `double` | `—`    | `pos.eq.dec`      | Fiber pointing declination (degree)                                                                                           |
| `subproject`       | `char`   | `—`    | `—`               | Uncertainty of rv_lasp1 measured by the error propagation (km/s)                                                              |
| `obsid`            | `long`   | `—`    | `meta.id`         | Target observation date                                                                                                       |
| `designation`      | `char`   | `—`    | `—`               | Declination from input catalog (degree)                                                                                       |
| `obsdate`          | `char`   | `—`    | `TIME_DATE`       | Target observation date                                                                                                       |
| `snr`              | `double` | `—`    | `—`               | The median value of all pixel S/Ns in B band spectrum or R band                                                               |
| `offsets`          | `int`    | `—`    | `—`               | Whether there is a fiber offset during observation                                                                            |
| `gaia_rp_mean_mag` | `double` | `—`    | `—`               | The “phot_rp_mean_mag” field of Gaia DR3 catalog                                                                              |
| `offsets_v`        | `double` | `—`    | `—`               | If the “offsets” field is true, “offsets_v” gives the offset distance from the target”s coordinator in input catalog (arcsec) |
| `ra`               | `double` | `—`    | `POS_EQ_RA_MAIN`  | Right ascension from input catalog (degree)                                                                                   |
| `lmjd`             | `int`    | `—`    | `—`               | Local modified Julian day                                                                                                     |
| `mjd`              | `int`    | `—`    | `—`               | Modified Julian day                                                                                                           |
| `gaia_source_id`   | `char`   | `—`    | `—`               | The “source_id” field of Gaia DR3 catalog                                                                                     |
| `spid`             | `int`    | `—`    | `—`               | Spectrograph ID                                                                                                               |
| `fiberid`          | `int`    | `—`    | `—`               | Fiber ID                                                                                                                      |
| `band`             | `char`   | `—`    | `—`               | Having two values of B and R, which represent B and R band spectra                                                            |
| `ra_obs`           | `double` | `—`    | `pos.eq.ra`       | Fiber pointing right ascension (degree)                                                                                       |
| `planid`           | `char`   | `—`    | `—`               | Local modified Julian minute                                                                                                  |
| `rv_b_flag`        | `int`    | `—`    | `—`               | Metallicity given by the LASP (dex)                                                                                           |
| `rv_lasp0_err`     | `double` | `—`    | `—`               | Uncertainty of rv_r measured by the error propagation (km/s)                                                                  |
| `bad_r`            | `int`    | `—`    | `—`               | Radial velocity flag for B + R spectra (km/s)                                                                                 |
| `coadd`            | `int`    | `—`    | `—`               | Uncertainty of rv_br0 measured by the method LASP used (km/s)                                                                 |
| `moon_flg`         | `char`   | `—`    | `—`               | Radial velocity after zero-point correction of rv_lasp0 (km/s)                                                                |
| `uid`              | `char`   | `—`    | `—`               | Plan name                                                                                                                     |
| `mobsid`           | `char`   | `—`    | `ID_MAIN`         | Unique ID for each medium-resolution spectrum in each exposure                                                                |
| `moon_angle`       | `double` | `—`    | `—`               | Radial velocity measured by the LASP (km/s)                                                                                   |
| `rv_br1`           | `double` | `—`    | `—`               | Radial velocity flag for B band spectra                                                                                       |
| `rv_r0_err`        | `double` | `—`    | `—`               | Projected rotation velocity given by the LASP                                                                                 |
| `rv_br0`           | `double` | `—`    | `—`               | Radial velocity after zero-point correction of rv_b0 (km/s)                                                                   |
| `gp_id`            | `long`   | `—`    | `—`               | Right ascension from input catalog (degree)                                                                                   |

### Images

No image cutout backends are configured for this survey.

### Spectra

- **Fibre diameter:** `3.3` | **Disperser:** `grating` | **Redshift rank:** `4` | **Flux calibration:** `relative` | **Flux scale:** `None`
- **Adapter `template`:** decoder format `lamost`, output parallel arrays `['flux', 'error', 'wavelength', 'mask']`, params `{'url_template': 'lamost-dr11'}`.
  - **hdus:** Coadded `COADD_B` and `COADD_R` extensions (`FLUX`, `IVAR`, `WAVELENGTH`, `ORMASK`) spliced in wavelength order.
  - **wavelength_range_angstrom:** 4900 - 5400 A (blue arm) and 6300 - 6800 A (red arm), vacuum
  - **resolution:** R ~ 7500 (3.3 arcsec fibres)
  - **flux_calibration:** relative
  - **arrays:** [‘flux’, ‘error’, ‘wavelength’, ‘mask’]

## References and acknowledgement

- Hou, Y. et al. (2018), *LAMOST medium-resolution spectrographs*, Proc. SPIE 10702, Ground-based and Airborne Instrumentation for Astronomy VII, 107021M (`doi:10.1117/12.2311893`).
- LAMOST DR11 MRS Data Production Description: [https://www.lamost.org/dr11/v2.0/doc/mr-data-production-description](https://www.lamost.org/dr11/v2.0/doc/mr-data-production-description)
- LAMOST DR11 TAP Service: [https://www.lamost.org/dr11/v2.0/voservice/tap](https://www.lamost.org/dr11/v2.0/voservice/tap)

Guoshoujing Telescope (the Large Sky Area Multi-Object Fiber Spectroscopic Telescope, LAMOST) is a National Major Scientific Project built by the Chinese Academy of Sciences. Funding for the project has been provided by the National Development and Reform Commission. LAMOST is operated and managed by the National Astronomical Observatories, Chinese Academy of Sciences.

## Provenance of this page

| Source ID            | Kind             | Service / Endpoint                                                    | Query or URL                                                                                                             | Retrieved (UTC)        |
|----------------------|------------------|-----------------------------------------------------------------------|--------------------------------------------------------------------------------------------------------------------------|------------------------|
| `catalog_lamost_med` | `tap_table`      | `https://www.lamost.org/dr11/v2.0/voservice/tap`                      | `SELECT column_name, datatype, unit, ucd, description FROM TAP_SCHEMA.columns WHERE table_name = 'public.med_catalogue'` | `2026-10-18T16:33:57Z` |
| `spectra_lamost_mrs` | `spectrum_model` | `https://www.lamost.org/dr11/v2.0/doc/mr-data-production-description` | `https://www.lamost.org/dr11/v2.0/doc/mr-data-production-description`                                                    | `2026-10-18T16:33:57Z` |
<!-- /generated -->
