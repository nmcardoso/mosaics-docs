# DESI DR1 (`desi1`)

<!-- generated:survey-page -->

The Dark Energy Spectroscopic Instrument Data Release 1 (DESI DR1) is a wide-area spectroscopic survey mapping the expansion history of the Universe and the large-scale structure of the cosmos. Mounted on the Nicholas U. Mayall 4-metre Telescope at Kitt Peak National Observatory (KPNO), DESI utilizes 5,000 robotically positioned optical fibres (1.5 arcsec diameter) feeding 10 three-arm spectrographs with volume phase holographic (VPH) gratings. Spectra cover optical and near-infrared wavelengths from 3600 to 9824 Å in vacuum with a uniform 0.8 Å binning and spectral resolution ranging from $R \sim 2000$ in the blue arm to $R \sim 5500$ in the red/NIR arm. The DR1 “iron” reduction (`specprod: iron`) catalogues over 28 million spectroscopic entries (`desi_dr1.zpix`, 28,425,963 rows before cuts) across more than 14,000 deg² of extragalactic sky.

In `mosaics`, DESI DR1 serves as the premier spectroscopic anchor, holding top priority (`redshift_rank: 1`) during multi-survey redshift tiebreaking thanks to its high resolution, precise radial velocities, and rigorous quality flags. It drives spectroscopic target selection (`mosaics-targets` using `defs/surveys/spec_desi_dr1.sql`), catalogue crossmatching (`mosaics-query` via Astro Data Lab TAP table `desi_dr1.zpix`), and 1D spectrum extraction (`mosaics-spectrum desi1` or `full`).

| Property                 | Value                                                                                                                                                                                                                                                                                                     |
|--------------------------|-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| What MOSAICS extracts    | Catalogue: `desi1` (spectroscopic, `tap`); Spectra: `sparcl` (flux,error,wavelength,mask), `desi-healpix` (flux,error,wavelength,mask)                                                                                                                                                                    |
| Archives                 | datalab (`https://datalab.noirlab.edu/tap`), `data.desi.lbl.gov`                                                                                                                                                                                                                                          |
| Bands / wavelength range | spectroscopy (disperser `grating`, fibre `1.5` arcsec, calibration `absolute`)                                                                                                                                                                                                                            |
| Used by                  | `mosaics-targets` (spectroscopic selection), `mosaics-spectrum desi1` / `full`, `mosaics-crossmatch`, `mosaics-query`                                                                                                                                                                                     |
| Crossmatch radius        | `1.5` arcsec. Fibre diameter 107 um = 1.52 arcsec on the sky at 14.2 arcsec/mm; fibre positioning 11 um rms 2-D (about 0.16 arcsec), astrometric residuals about 30 mas 1-D (Kent et al. 2023, arXiv:2307.06238). That paper does not say whether desi_dr1.zpix carries the target or the fibre position. |

## Configuration

### The shipped file

Source: `defs/surveys/desi1.yaml`

```yaml
# DESI DR1 -- spectroscopy.

survey_slug: desi1
name: DESI DR1

catalog:
  - name: desi1
    type: spectroscopic
    adapter: tap
    adapter_params:
      service: datalab
      table: desi_dr1.zpix
      dialect: top

# Crossmatch: the radius a table of positions is matched within, and what it rests on.
crossmatch:
  radius: 1.5 arcsec
  radius_note: >-
    Fibre diameter 107 um = 1.52 arcsec on the sky at 14.2 arcsec/mm; fibre positioning 11
    um rms 2-D (about 0.16 arcsec), astrometric residuals about 30 mas 1-D (Kent et al.
    2023, arXiv:2307.06238). That paper does not say whether desi_dr1.zpix carries the
    target or the fibre position.

catalog_filters:
  # See sdss17.yaml for the general rule this and the other five redshift
  # surveys share.
  desi1: desi1_z BETWEEN 0.0 AND 2.5
  desi1_note: >-
    Restricts the catalogue to the redshift range the training targets are
    drawn over; a row outside it keeps its candidacy but loses its DESI
    spectroscopic columns, as if this survey had not matched it.

target_selection:
  catalog_name: desi1
  query: spec_desi_dr1.sql
  ra_column: mean_fiber_ra
  dec_column: mean_fiber_dec

spectra:
  # DESI publishes through both SPARCL and bulk HEALPix coadds. They are
  # alternatives, not a choice the router makes, so exactly one is enabled
  # and the other stays as documentation.
  - adapter: sparcl         # one request per target
    adapter_params:
      dataset: DESI-DR1
      id_column: targetid
    arrays: [flux, error, wavelength, mask]
  - adapter: desi-healpix   # bulk coadds, ~1.1 GB each
    adapter_params:
      release: dr1
      specprod: iron
    arrays: [flux, error, wavelength, mask]
    enabled: false

spectroscopic_system:
  # A constant its own SQL could not carry: the service mangles a numeric
  # literal in a SELECT, which the query's header documents.
  fiber_diameter: 1.5
  # Bare VPH gratings, no prism, one per b/r/z arm (DESI Collaboration 2022,
  # AJ 164, 207). VPH is a fabrication technique, not a different component.
  disperser: grating
  # Leads the tiebreak on resolution and on the rigour of its redshift
  # warnings.
  redshift_rank: 1
  flux_scale: 1.0             # native 1e-17 cgs already
  flux_calibration: absolute
```

Target-selection query (`defs/surveys/spec_desi_dr1.sql`):

```sql
-- DESI DR1 spectroscopy: scientifically reliable spectra from the iron reduction.
--
-- Provider: tap --service datalab (Astro Data Lab, ADQL).
--   uv run -m mosaics.cli.query sql --provider tap --service datalab \
--     --path sql/desi_dr1_spectra.sql \
--     --ra-col mean_fiber_ra --dec-col mean_fiber_dec -o desi.parquet
--
-- Retrieval contract: targetid alone is enough for `spectrum desi` (SPARCL
-- resolves it), while healpix, survey and program are what `--via healpix`
-- needs to find the coadd file. Both sets are selected so either route works
-- from one catalogue.
--
-- Two deviations from the executive report, both confirmed against the live
-- schema rather than assumed:
--   * OBJTYPE=TGT is not applied: desi_dr1.zpix has no objtype column at all.
--     coadd_fiberstatus = 0 already excludes sky, broken and unassigned fibres,
--     which is what that cut was for.
--   * zcat_primary is a Postgres boolean, so `= 1` raises a type error. ADQL
--     will not take a bare boolean as a predicate either, so it is compared
--     against the string literal 'true', which the Data Lab backend casts.
--
-- No fiber_diameter column, unlike the SDSS query. Every DESI fibre is 1.5"
-- so the value is a constant, but this TAP service truncates every numeric
-- literal to an integer -- 1.5, 3.0/2 and z*0+1.5 all come back as 1 -- and a
-- silently wrong aperture is worse than an absent one. Add the constant
-- downstream when concatenating with SDSS.
--
-- zpix is the per-HEALPix coadd catalogue, the ingested equivalent of
-- zall-pix-iron.fits: 28,425,963 rows before cuts.

SELECT
  z.targetid          AS desi1_id, 
  z.mean_fiber_ra     AS desi1_ra, 
  z.mean_fiber_dec    AS desi1_dec,
  -- retrieval keys: targetid for SPARCL, the triple for the coadd files
  z.healpix           AS desi1_healpix, 
  z.survey            AS desi1_survey, 
  z.program           AS desi1_program,
  -- redshift and its reliability. deltachi2 is kept as a feature rather than
  -- a cut: it is the margin between the best and second-best redshift fit.
  z.z                 AS desi1_z, 
  z.zerr              AS desi1_z_err, 
  z.zwarn             AS desi1_z_flag, 
  z.deltachi2         AS desi1_z_deltachi2, 
  z.chi2              AS desi1_z_chi2,
  -- weak labels
  z.spectype          AS desi1_class, 
  z.subtype           AS desi1_subclass,
  -- fibre and exposure bookkeeping
  z.coadd_fiberstatus AS desi1_fiber_status, 
  z.coadd_numexp      AS desi1_numexp, 
  z.coadd_exptime     AS desi1_exptime, 
  z.coadd_numtile     AS desi1_numtile,
  'desi1'             AS "sample"
FROM desi_dr1.zpix    AS z
WHERE
  -- S1: the redshift fit raised no warning
  z.zwarn = 0
  -- S2: every fibre that contributed to the coadd was healthy
  AND z.coadd_fiberstatus = 0
  -- S3: one row per target, the one the catalogue considers definitive
  AND z.zcat_primary = 'true'
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

**Catalogue `desi1` (`adapter: tap`):**

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
- **Spectra (`adapter: sparcl`, format `None`):** configured `adapter_params`: `dataset='DESI-DR1'`, `id_column='targetid'`.
- **Spectra (`adapter: desi-healpix`, format `None`):** configured `adapter_params`: `release='dr1'`, `specprod='iron'`.

### Common changes

- **Switching spectrum extraction backend**: By default, `mosaics` retrieves individual spectra via the NOIRLab SPARCL API (`adapter: sparcl`, `dataset: DESI-DR1`). For large batch processing, researchers can switch to the local/bulk HEALPix coadd adapter by setting `adapter: desi-healpix` (`release: dr1`, `specprod: iron`, `enabled: true`) and disabling SPARCL (`enabled: false`).
- **Redshift range filter**: Override the catalogue redshift constraint in your entrypoint YAML via `overrides[surveys/desi1].catalog_filters.desi1` (default: `desi1_z BETWEEN 0.0 AND 2.5`). To remove the boundary, set `overrides[surveys/desi1].catalog_filters.desi1: null`.
- **Target selection cuts**: Modify `defs/surveys/spec_desi_dr1.sql` to refine spectroscopic filtering. For example, add a threshold on the fit confidence margin `z.deltachi2` (the $\Delta\chi^2$ between the best and second-best redshift template fits) or restrict by spectral classification `z.spectype` (`GALAXY`, `QSO`, `STAR`).
- **Disable catalogue crossmatching**: Disable DESI DR1 catalogue matching in an entrypoint configuration by setting `overrides[surveys/desi1].catalog.0.enabled: false`.

### Commands

```console
$ uv run -m mosaics.cli.spectrum desi1 -t test/objects_sample_1k.csv -o desi1_spectra.lance
$ uv run -m mosaics.crossmatch.cli run --survey surveys/desi1 -i test/objects_sample_1k.csv -o desi1_matched.parquet
$ uv run -m mosaics.cli.targets --config defs/targets.yaml -o ./targets_run --dec-range 0 1 --plan plan.yaml
```

## Data served by the archive

- **Flux scale and calibration**: DESI spectra have absolute spectrophotometric flux calibration expressed in physical units of $10^{-17}\text{ erg s}^{-1}\text{ cm}^{-2}\text{ \AA}^{-1}$. In `mosaics`, `flux_scale: 1.0` and `flux_calibration: absolute`, and inverse variance arrays (`ivar`) are extracted directly from the pipeline variance products (`arrays: [flux, error, wavelength, mask]`).
- **Fibre diameter**: The physical fibre aperture is 1.5 arcsec on the sky. The Astro Data Lab TAP backend truncates floating-point literals in `SELECT` statements (e.g. `1.5` returns as integer `1`); therefore, `fiber_diameter: 1.5` is configured directly in the YAML specification rather than the SQL query.
- **Boolean syntax in ADQL**: In the Astro Data Lab PostgreSQL backend, `zcat_primary` is stored as a native boolean. Integer equality (`= 1`) produces a type error, so the target selection query compares against the string literal `'true'` (`z.zcat_primary = 'true'`).
- **Spectroscopic quality criteria**:
  - `zwarn = 0`: Ensures that Redrock raised no warning flags during spectral template fitting.
  - `coadd_fiberstatus = 0`: Verifies that all fibres contributing to the coadded spectrum were healthy and free from hardware anomalies.
  - `zcat_primary = 'true'`: Restricts rows to the single primary coadd per target, eliminating duplicate visits and tiles.
- **Redshift tiebreak ranking**: DESI DR1 is assigned `redshift_rank: 1` in MOSAICS, taking precedence over older or lower-resolution surveys when reconciling candidate coordinates across multiple spectroscopic catalogues.

### Catalogue: `desi_dr1.zpix`

Redshift and other quantities for spectra that have been grouped

**Service URL:** `https://datalab.noirlab.edu/tap`

| Column                       | Type            | Unit      | UCD                    | Description                                                      |
|------------------------------|-----------------|-----------|------------------------|------------------------------------------------------------------|
| `tsnr2_lrg_b`                | `adql:REAL`     | `—`       | `—`                    | LRG B template (S/N)^2                                           |
| `tsnr2_gpbdark_r`            | `adql:REAL`     | `—`       | `—`                    | GPBDARK R template (S/N)^2                                       |
| `tsnr2_elg_r`                | `adql:REAL`     | `—`       | `—`                    | ELG R template (S/N)^2                                           |
| `tsnr2_gpbbright_r`          | `adql:REAL`     | `—`       | `—`                    | GPBBRIGHT R template (S/N)^2                                     |
| `tsnr2_lya_r`                | `adql:REAL`     | `—`       | `—`                    | LYA R template (S/N)^2                                           |
| `tsnr2_bgs_r`                | `adql:REAL`     | `—`       | `—`                    | BGS R template (S/N)^2                                           |
| `tsnr2_gpbbackup_r`          | `adql:REAL`     | `—`       | `—`                    | GPBBACKUP R template (S/N)^2                                     |
| `z`                          | `adql:DOUBLE`   | `—`       | `—`                    | Redshift measured by Redrock                                     |
| `zerr`                       | `adql:DOUBLE`   | `—`       | `—`                    | Redshift error from Redrock                                      |
| `chi2`                       | `adql:DOUBLE`   | `—`       | `—`                    | Best fit chi squared                                             |
| `coeff_0`                    | `adql:DOUBLE`   | `—`       | `—`                    | Redrock template coefficients                                    |
| `coeff_1`                    | `adql:DOUBLE`   | `—`       | `—`                    | Redrock template coefficients                                    |
| `coeff_2`                    | `adql:DOUBLE`   | `—`       | `—`                    | Redrock template coefficients                                    |
| `coeff_3`                    | `adql:DOUBLE`   | `—`       | `—`                    | Redrock template coefficients                                    |
| `coeff_4`                    | `adql:DOUBLE`   | `—`       | `—`                    | Redrock template coefficients                                    |
| `coeff_5`                    | `adql:DOUBLE`   | `—`       | `—`                    | Redrock template coefficients                                    |
| `coeff_6`                    | `adql:DOUBLE`   | `—`       | `—`                    | Redrock template coefficients                                    |
| `coeff_7`                    | `adql:DOUBLE`   | `—`       | `—`                    | Redrock template coefficients                                    |
| `coeff_8`                    | `adql:DOUBLE`   | `—`       | `—`                    | Redrock template coefficients                                    |
| `coeff_9`                    | `adql:DOUBLE`   | `—`       | `—`                    | Redrock template coefficients                                    |
| `deltachi2`                  | `adql:DOUBLE`   | `—`       | `—`                    | Delta-chi-squared for template fit from Redrock                  |
| `mean_fiber_ra`              | `adql:DOUBLE`   | `deg`     | `pos.eq.ra;meta.main`  | Mean (over exposures) RA of actual fiber position                |
| `mean_fiber_dec`             | `adql:DOUBLE`   | `deg`     | `pos.eq.dec;meta.main` | Mean (over exposures) DEC of actual fiber position               |
| `elon`                       | `adql:DOUBLE`   | `deg`     | `—`                    | Ecliptic longitude                                               |
| `elat`                       | `adql:DOUBLE`   | `deg`     | `—`                    | Ecliptic latitude                                                |
| `glon`                       | `adql:DOUBLE`   | `deg`     | `—`                    | Galactic longitude                                               |
| `glat`                       | `adql:DOUBLE`   | `deg`     | `—`                    | Galactic latitude                                                |
| `id`                         | `adql:VARCHAR`  | `—`       | `meta.id;meta.main`    | Arbitrary primary key composed from targetid, survey, program. N |
| `targetid`                   | `adql:BIGINT`   | `—`       | `—`                    | Unique DESI target ID                                            |
| `coadd_exptime`              | `adql:REAL`     | `seconds` | `—`                    | Sum of the exposure times contributing to the coadd for this tar |
| `mean_delta_x`               | `adql:REAL`     | `—`       | `—`                    | Mean (over exposures) fiber difference between measured and requ |
| `rms_delta_x`                | `adql:REAL`     | `—`       | `—`                    | RMS (over exposures) of the fiber difference between measured an |
| `mean_delta_y`               | `adql:REAL`     | `—`       | `—`                    | Mean (over exposures) fiber difference between measured and requ |
| `rms_delta_y`                | `adql:REAL`     | `—`       | `—`                    | RMS (over exposures) of the fiber difference between measured an |
| `std_fiber_ra`               | `adql:REAL`     | `deg`     | `—`                    | Standard deviation (over exposures) of RA of actual fiber positi |
| `std_fiber_dec`              | `adql:REAL`     | `deg`     | `—`                    | Standard deviation (over exposures) of DEC of actual fiber posit |
| `mean_psf_to_fiber_specflux` | `adql:REAL`     | `—`       | `—`                    | Mean of the PSF_TO_FIBER_SPECFLUX values of the spectra used in  |
| `tsnr2_gpbdark_b`            | `adql:REAL`     | `—`       | `—`                    | GPBDARK B template (S/N)^2                                       |
| `tsnr2_elg_b`                | `adql:REAL`     | `—`       | `—`                    | ELG B template (S/N)^2                                           |
| `tsnr2_gpbbright_b`          | `adql:REAL`     | `—`       | `—`                    | GPBBRIGHT B template (S/N)^2                                     |
| `tsnr2_lya_b`                | `adql:REAL`     | `—`       | `—`                    | LYA B template (S/N)^2                                           |
| `tsnr2_bgs_b`                | `adql:REAL`     | `—`       | `—`                    | BGS B template (S/N)^2                                           |
| `tsnr2_gpbbackup_b`          | `adql:REAL`     | `—`       | `—`                    | GPBBACKUP B template (S/N)^2                                     |
| `tsnr2_qso_b`                | `adql:REAL`     | `—`       | `—`                    | QSO B template (S/N)^2                                           |
| `tsnr2_qso_r`                | `adql:REAL`     | `—`       | `—`                    | QSO R template (S/N)^2                                           |
| `tsnr2_lrg_r`                | `adql:REAL`     | `—`       | `—`                    | LRG R template (S/N)^2                                           |
| `tsnr2_gpbdark_z`            | `adql:REAL`     | `—`       | `—`                    | GPBDARK Z template (S/N)^2                                       |
| `tsnr2_elg_z`                | `adql:REAL`     | `—`       | `—`                    | ELG Z template (S/N)^2                                           |
| `tsnr2_gpbbright_z`          | `adql:REAL`     | `—`       | `—`                    | GPBBRIGHT Z template (S/N)^2                                     |
| `tsnr2_lya_z`                | `adql:REAL`     | `—`       | `—`                    | LYA Z template (S/N)^2                                           |
| `tsnr2_bgs_z`                | `adql:REAL`     | `—`       | `—`                    | BGS Z template (S/N)^2                                           |
| `tsnr2_gpbbackup_z`          | `adql:REAL`     | `—`       | `—`                    | GPBBACKUP Z template (S/N)^2                                     |
| `tsnr2_qso_z`                | `adql:REAL`     | `—`       | `—`                    | QSO Z template (S/N)^2                                           |
| `tsnr2_lrg_z`                | `adql:REAL`     | `—`       | `—`                    | LRG Z template (S/N)^2                                           |
| `tsnr2_gpbdark`              | `adql:REAL`     | `—`       | `—`                    | GPBDARK template (S/N)^2 summed over B, R, Z                     |
| `tsnr2_elg`                  | `adql:REAL`     | `—`       | `—`                    | ELG template (S/N)^2 summed over B, R, Z                         |
| `tsnr2_gpbbright`            | `adql:REAL`     | `—`       | `—`                    | GPBBRIGHT template (S/N)^2 summed over B, R, Z                   |
| `tsnr2_lya`                  | `adql:REAL`     | `—`       | `—`                    | LYA template (S/N)^2 summed over B, R, Z                         |
| `tsnr2_bgs`                  | `adql:REAL`     | `—`       | `—`                    | BGS template (S/N)^2 summed over B, R, Z                         |
| `tsnr2_gpbbackup`            | `adql:REAL`     | `—`       | `—`                    | GPBBACKUP template (S/N)^2 summed over B, R, Z                   |
| `tsnr2_qso`                  | `adql:REAL`     | `—`       | `—`                    | QSO template (S/N)^2 summed over B, R, Z                         |
| `tsnr2_lrg`                  | `adql:REAL`     | `—`       | `—`                    | LRG template (S/N)^2 summed over B, R, Z                         |
| `random_id`                  | `adql:REAL`     | `—`       | `—`                    | Random ID in the range 0.0 => 100.0 (not unique)                 |
| `spgrpval`                   | `adql:INTEGER`  | `—`       | `—`                    | Value corresponding to the grouping type indicated in the spgrp  |
| `healpix`                    | `adql:INTEGER`  | `—`       | `—`                    | HEALPixel containing this location at NSIDE=64 in the NESTED sch |
| `coadd_fiberstatus`          | `adql:INTEGER`  | `—`       | `—`                    | Bitwise AND of the FIBERSTATUS values of the spectra used in the |
| `htm9`                       | `adql:INTEGER`  | `—`       | `—`                    | Ninth level (~10 arcmin resolution) Hierarchical Triangular Mesh |
| `ring256`                    | `adql:INTEGER`  | `—`       | `—`                    | Ring-scheme HEALPix (Gorski et al. 2005) index with Nsides=256 ( |
| `nest4096`                   | `adql:INTEGER`  | `—`       | `—`                    | Nested-scheme HEALPix (Gorski et al. 2005) index with Nsides=409 |
| `zwarn`                      | `adql:BIGINT`   | `—`       | `—`                    | Redshift warning bitmask measured by Redrock                     |
| `npixels`                    | `adql:BIGINT`   | `—`       | `—`                    | Number of unmasked wavelengths that contributed to the redshift  |
| `ncoeff`                     | `adql:BIGINT`   | `—`       | `—`                    | Number of Redrock template coefficients                          |
| `coadd_numexp`               | `adql:SMALLINT` | `—`       | `—`                    | Number of exposures included in the coadd for this target        |
| `coadd_numnight`             | `adql:SMALLINT` | `—`       | `—`                    | Number of different nights included in the coadd for this target |
| `coadd_numtile`              | `adql:SMALLINT` | `—`       | `—`                    | Number of different tiles included in the coadd for this target  |
| `sv_nspec`                   | `adql:SMALLINT` | `—`       | `—`                    | Number of coadded spectra for this TARGETID in SV (SV1+2+3)      |
| `main_nspec`                 | `adql:SMALLINT` | `—`       | `—`                    | Number of coadded spectra for this TARGETID in Main survey       |
| `zcat_nspec`                 | `adql:SMALLINT` | `—`       | `—`                    | Number of coadded spectra for this TARGETID in this zcatalog     |
| `sv_primary`                 | `adql:BOOLEAN`  | `—`       | `—`                    | True/False flag for the primary coadded spectrum in SV (SV1+2+3) |
| `main_primary`               | `adql:BOOLEAN`  | `—`       | `—`                    | True/False flag for the primary coadded spectrum in Main survey  |
| `zcat_primary`               | `adql:BOOLEAN`  | `—`       | `—`                    | True/False flag for the primary coadded spectrum in this zcatalo |
| `spgrp`                      | `adql:VARCHAR`  | `—`       | `—`                    | Type of spectral grouping. For zpix table this is always = healp |
| `subtype`                    | `adql:VARCHAR`  | `—`       | `—`                    | Spectral subtype                                                 |
| `program`                    | `adql:VARCHAR`  | `—`       | `—`                    | DESI program type - BRIGHT, DARK, or BACKUP                      |
| `spectype`                   | `adql:VARCHAR`  | `—`       | `—`                    | Spectype from Redrock file                                       |
| `survey`                     | `adql:VARCHAR`  | `—`       | `—`                    | Survey name                                                      |
| `mean_mjd`                   | `adql:DOUBLE`   | `day`     | `—`                    | Mean value of the Modified Julian Date (when the shutter was ope |
| `min_mjd`                    | `adql:DOUBLE`   | `day`     | `—`                    | Minimum value of the Modified Julian Date (when the shutter was  |
| `max_mjd`                    | `adql:DOUBLE`   | `day`     | `—`                    | Maximum value of the Modified Julian Date (when the shutter was  |
| `mws_target`                 | `adql:BIGINT`   | `—`       | `—`                    | Milky Way Survey targeting bits                                  |
| `sv1_mws_target`             | `adql:BIGINT`   | `—`       | `—`                    | MWS (bright time program) target selection bitmask for SV1       |
| `sv2_mws_target`             | `adql:BIGINT`   | `—`       | `—`                    | MWS (bright time program) target selection bitmask for SV2       |
| `sv3_mws_target`             | `adql:BIGINT`   | `—`       | `—`                    | MWS (bright time program) target selection bitmask for SV3       |
| `scnd_target`                | `adql:BIGINT`   | `—`       | `—`                    | Target selection bitmask for secondary programs                  |
| `sv1_scnd_target`            | `adql:BIGINT`   | `—`       | `—`                    | Secondary target selection bitmask for SV1                       |
| `sv2_scnd_target`            | `adql:BIGINT`   | `—`       | `—`                    | Secondary target selection bitmask for SV2                       |
| `sv3_scnd_target`            | `adql:BIGINT`   | `—`       | `—`                    | Secondary target selection bitmask for SV3                       |
| `bgs_target`                 | `adql:BIGINT`   | `—`       | `—`                    | BGS (bright time program) target selection bitmask               |
| `sv1_bgs_target`             | `adql:BIGINT`   | `—`       | `—`                    | BGS (bright time program) target selection bitmask for SV1       |
| `sv2_bgs_target`             | `adql:BIGINT`   | `—`       | `—`                    | BGS (bright time program) target selection bitmask for SV2       |
| `sv3_bgs_target`             | `adql:BIGINT`   | `—`       | `—`                    | BGS (bright time program) target selection bitmask for SV3       |
| `desi_target`                | `adql:BIGINT`   | `—`       | `—`                    | Dark survey + calibration targeting bits                         |
| `sv1_desi_target`            | `adql:BIGINT`   | `—`       | `—`                    | DESI (dark time program) target selection bitmask for SV1        |
| `sv2_desi_target`            | `adql:BIGINT`   | `—`       | `—`                    | DESI (dark time program) target selection bitmask for SV2        |
| `sv3_desi_target`            | `adql:BIGINT`   | `—`       | `—`                    | DESI (dark time program) target selection bitmask for SV3        |
| `cmx_target`                 | `adql:BIGINT`   | `—`       | `—`                    | Target selection bitmask for commissioning                       |
| `desiname`                   | `adql:VARCHAR`  | `—`       | `—`                    | Human-readable identifier of a sky location DESI JXXX.XXXX[+/-]Y |

### Images

No image cutout backends are configured for this survey.

### Spectra

- **Fibre diameter:** `1.5` | **Disperser:** `grating` | **Redshift rank:** `1` | **Flux calibration:** `absolute` | **Flux scale:** `1.0`
- **Adapter `sparcl`:** decoder format `None`, output parallel arrays `['flux', 'error', 'wavelength', 'mask']`, params `{'dataset': 'DESI-DR1', 'id_column': 'targetid'}`.
- **Adapter `desi-healpix`:** decoder format `None`, output parallel arrays `['flux', 'error', 'wavelength', 'mask']`, params `{'release': 'dr1', 'specprod': 'iron'}`.
  - **hdus:** B/R/Z camera extensions (`B_WAVELENGTH`, `B_FLUX`, `B_IVAR`, `B_MASK`, and R/Z equivalents) spliced onto a single grid, or SPARCL pre-coadded arrays.
  - **wavelength_range_angstrom:** 3600 - 9824 A (vacuum, 0.8 A linear step)
  - **resolution:** R ~ 2000 (blue) to 5500 (NIR), 1.5 arcsec fibres
  - **flux_calibration:** absolute (`10^-17 erg s^-1 cm^-2 A^-1`)
  - **arrays:** [‘flux’, ‘error’, ‘wavelength’, ‘mask’]

## References and acknowledgement

- DESI Collaboration, et al. (2022), *AJ*, 164, 207 (`doi:10.3847/1538-3881/ac8210`) — Overview of the Instrumentation for the Dark Energy Spectroscopic Instrument.
- DESI Collaboration, et al. (2024), *AJ*, 168, 58 (`doi:10.3847/1538-3881/ad5836`) — Data Release 1 of the Dark Energy Spectroscopic Instrument.
- Guy, J., et al. (2023), *AJ*, 165, 144 (`doi:10.3847/1538-3881/acb212`) — The Spectroscopic Data Processing Pipeline for the Dark Energy Spectroscopic Instrument.
- NOIRLab Astro Data Lab TAP service: [https://datalab.noirlab.edu/tap](https://datalab.noirlab.edu/tap) (table `desi_dr1.zpix`).
- DESI Data Release 1 Portal: [https://data.desi.lbl.gov/public/dr1/](https://data.desi.lbl.gov/public/dr1/).
- NOIRLab SPARCL: [https://astrosparcl.datalab.noirlab.edu/](https://astrosparcl.datalab.noirlab.edu/).

Publications using DESI DR1 data must cite DESI Collaboration et al. (2022, 2024) and Guy et al. (2023), and include the standard DESI acknowledgement: “This research used data obtained with the Dark Energy Spectroscopic Instrument (DESI). DESI construction and operations is managed by Lawrence Berkeley National Laboratory for the U.S. Department of Energy Office of Science.” When querying via Astro Data Lab or SPARCL, include the official NOIRLab acknowledgement ([https://datalab.noirlab.edu/acknowledgements.php](https://datalab.noirlab.edu/acknowledgements.php)).

## Provenance of this page

| Source ID           | Kind             | Service / Endpoint                                         | Query or URL                                                                                                      | Retrieved (UTC)        |
|---------------------|------------------|------------------------------------------------------------|-------------------------------------------------------------------------------------------------------------------|------------------------|
| `catalog_desi_zpix` | `tap_table`      | `https://datalab.noirlab.edu/tap`                          | `SELECT column_name, datatype, unit, ucd, description FROM TAP_SCHEMA.columns WHERE table_name = 'desi_dr1.zpix'` | `2026-10-18T16:30:07Z` |
| `spectra_desi_dr1`  | `spectrum_model` | `https://data.desi.lbl.gov/public/dr1/spectro/redux/iron/` | `https://data.desi.lbl.gov/public/dr1/spectro/redux/iron/`                                                        | `2026-10-18T16:30:07Z` |
<!-- /generated -->
