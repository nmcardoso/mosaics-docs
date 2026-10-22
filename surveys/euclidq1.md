# Euclid Q1 (`euclidq1`)

<!-- generated:survey-page -->

Euclid Quick Release 1 (Euclid Q1) is the earliest public data release from the European Space Agency’s Euclid space observatory. Euclid surveys the extragalactic sky with two state-of-the-art instruments: the VISible instrument (VIS), providing high-resolution optical imaging in a broad band ($550\text{--}900\text{ nm}$), and the Near-Infrared Spectrometer and Photometer (NISP), providing near-infrared imaging in three passbands ($Y, J, H$) and slitless slit-free grism spectroscopy using red grisms (RGS000/RGS180, covering $12060\text{--}18920\text{ \AA}$ at $R \sim 450$). The Q1 data release spans the Euclid Deep Fields and early calibration patches, cataloguing nearly 30 million sources across its MER photometric final catalogue (29,767,806 rows in 85 HATS partitions) and merged photometric-spectroscopic catalogue (29,953,430 rows in 114 HATS partitions).

In `mosaics`, Euclid Q1 uniquely contributes both photometric and spectroscopic subsystems under distinct column prefixes: `euclidq1p_*` for MER photometry and `euclidq1s_*` for merged SPE spectroscopy. It is used for target selection (`mosaics-targets` using LSDB against the NASA IRSA S3 merged-objects HATS partition), photometric crossmatching (`mosaics-query` via `data.lsdb.io`), and 1D near-infrared slitless spectrum extraction (`mosaics-spectrum euclidq1` or `full`).

| Property                 | Value                                                                                                                                                                                                                                                                                                                                                                                         |
|--------------------------|-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| What MOSAICS extracts    | Catalogue: `euclidq1p` (photometric, `lsdb`), `euclidq1s` (spectroscopic, `lsdb`); Spectra: `url` (flux,error,wavelength,mask)                                                                                                                                                                                                                                                                |
| Archives                 | HATS (`data.lsdb.io`), HATS (`nasa-irsa-euclid-q1`), `irsa.ipac.caltech.edu`                                                                                                                                                                                                                                                                                                                  |
| Bands / wavelength range | `vis` (`Euclid/VIS.vis`), `y` (`Euclid/NISP.Y`), `j` (`Euclid/NISP.J`), `h` (`Euclid/NISP.H`); spectroscopy (disperser `grism`, calibration `absolute`)                                                                                                                                                                                                                                       |
| Used by                  | `mosaics-targets` (spectroscopic selection), `mosaics-spectrum euclidq1` / `full`, `mosaics-crossmatch`                                                                                                                                                                                                                                                                                       |
| Crossmatch radius        | `0.5` arcsec. VIS PSF FWHM 0.16 arcsec; NISP encircled-energy requirement rEE50 < 0.4 arcsec (a requirement, not a measured FWHM); absolute astrometry against Gaia 5 mas (VIS) and 9-15 mas (NISP) (Euclid Q1, arXiv:2503.15303 and arXiv:2503.15304). The grid floor of 0.5 arcsec was chosen because the NISP PSF is larger than the VIS one. One radius serves both catalogs of the file. |

## Configuration

### The shipped file

Source: `defs/surveys/euclidq1.yaml`

```yaml
# Euclid Q1 -- photometry and spectroscopy.
#
# The second of the two surveys contributing both halves, so the prefix rule
# applies: `euclidq1p_*` for the photometry, `euclidq1s_*` for the
# spectroscopy.

survey_slug: euclidq1
name: Euclid Q1

catalog:
  - name: euclidq1p
    type: photometric
    adapter: lsdb
    adapter_params:
      # The MER final catalogue as HATS: 29,767,806 rows in 85 partitions,
      # matched on this machine rather than uploaded to Data Lab. Its columns
      # are `euclid_q1.object`'s in upper case (all twenty below checked
      # 2026-09-13). LineA mirrors the same collection at linea.data.lsdb.io.
      path: https://data.lsdb.io/hats/euclid_q1/euclid_q1_merFinalCatalog
      margin_cache: https://data.lsdb.io/hats/euclid_q1/euclid_q1_merFinalCatalog_10arcs

  - name: euclidq1s
    type: spectroscopic
    adapter: lsdb
    adapter_params:
      # S3 rather than the bucket's HTTPS address: each partition is a
      # directory, and S3 lists a directory only through its own API. Opening
      # the catalogue over HTTPS works; reading a partition does not.
      path: s3://nasa-irsa-euclid-q1/contributed/q1/merged_objects/hats/euclid_q1_merged_objects-hats
      storage_options:
        anon: true

# Crossmatch: the radius a table of positions is matched within, and what it rests on.
crossmatch:
  radius: 0.5 arcsec
  radius_note: >-
    VIS PSF FWHM 0.16 arcsec; NISP encircled-energy requirement rEE50 < 0.4 arcsec (a
    requirement, not a measured FWHM); absolute astrometry against Gaia 5 mas (VIS) and 9-15
    mas (NISP) (Euclid Q1, arXiv:2503.15303 and arXiv:2503.15304). The grid floor of 0.5
    arcsec was chosen because the NISP PSF is larger than the VIS one. One radius serves
    both catalogs of the file.

catalog_filters:
  # See sdss17.yaml for the general rule this and the other five redshift
  # surveys share.
  euclidq1s: euclidq1s_z BETWEEN 0.0 AND 2.5
  euclidq1s_note: >-
    Restricts the catalogue to the redshift range the training targets are
    drawn over; a row outside it keeps its candidacy but loses its Euclid Q1
    spectroscopic columns, as if this survey had not matched it.

target_selection:
  # Read from a HATS catalogue with LSDB, cell by cell, on this machine:
  # IRSA's merged-objects build of Q1 on S3, which joins the MER catalogue
  # with the SPE redshift, classification and quality tables into one row per
  # object -- 29,953,430 rows in 114 partitions, the galaxy candidates' ranked
  # solutions flattened into `_rank0` to `_rank4` columns. See
  # mosaics/pipelines/targets/hats_spec.py.
  #
  # Not data.lsdb.io, which publishes Q1's MER catalogue alone -- the one the
  # photometry above reads -- and none of the SPE columns. And no longer Data
  # Lab, whose `euclid_q1` schema this section used to join over TAP
  # (sql/targets/spec_euclid_q1.sql, removed); its columns and cuts carry over
  # one for one.
  catalog_name: euclidq1s
  # Remote column -> alias, in the order the old query selected them.
  columns:
    object_id: euclidq1s_id
    # The coordinates the catalogue is partitioned on: MER's barycentre,
    # which `euclid_q1.object` calls right_ascension and declination.
    ra: euclidq1s_ra
    dec: euclidq1s_dec
    # Redshift and its reliability: the preferred (rank 0) galaxy solution.
    z_galaxy_candidates_spe_z_rank0: euclidq1s_z
    z_galaxy_candidates_spe_z_err_rank0: euclidq1s_z_err
    z_galaxy_candidates_spe_z_prob_rank0: euclidq1s_z_prob
    z_galaxy_candidates_spe_z_rel_rank0: euclidq1s_z_rel
    z_galaxy_candidates_spe_subclass_rank0: euclidq1s_subclass
    z_galaxy_candidates_spe_cont_snr_rank0: euclidq1s_cont_snr
    # Weak labels: the SPE classifier's own probabilities.
    z_spe_class: euclidq1s_class
    z_spe_gal_prob: euclidq1s_prob_gal
    z_spe_qso_prob: euclidq1s_prob_qso
    z_spe_star_prob: euclidq1s_prob_star
    # Spectroscopic quality. Both flags are carried: the warning flag because
    # nothing cuts on it, the error flag so that the cut that does is visible
    # in the output rather than only here.
    z_spe_error_flag: euclidq1s_flag_err
    z_spe_warning_flag: euclidq1s_flag_warn
    z_spe_grism: euclidq1s_grism
    z_spe_npix: euclidq1s_npix
    z_spe_n_dith_max: euclidq1s_n_dith_max
    # Imaging quality, from the MER final catalogue.
    mer_det_quality_flag: euclidq1s_flag_det
    mer_spurious_flag: euclidq1s_flag_spur
    mer_mu_max: euclidq1s_mu_max
    mer_segmentation_area: euclidq1s_seg_area
  # [column, operator, value] triples, pushed down to the Parquet reader.
  # Counted whole-sky against the SPE tables the old query joined: 18,409,905
  # galaxy candidates; rank 0 leaves 3,716,438; an error flag of 0 removes
  # nothing; spurious_flag = 0 leaves 3,687,815; det_quality_flag = 0 leaves
  # 2,597,628, which is what this section should return.
  filters:
    # S1: the preferred redshift solution. SPE reports five per object ranked
    # 0-4, and taking them all would store one spectrum five times. The rank-0
    # column is null for an object with no galaxy solution at all.
    - [z_galaxy_candidates_spe_rank_rank0, '==', 0]
    # S2: the spectroscopic pipeline reported no error. Free in Q1 -- every
    # object with a solution already has flag 0 -- but kept as the statement
    # of intent. The seven bits the DPDD defines: 0 context stage, 1
    # classification stage, 2 unable to read the SIR combined spectrum, 3
    # empty spectrum, 4 NaN values, 5 array-size mismatch after filtering,
    # 6 short spectrum.
    - [z_spe_error_flag, '==', 0]
    # S3: the detection is not flagged spurious, and MER detected it cleanly.
    - [mer_spurious_flag, '==', 0]
    - [mer_det_quality_flag, '==', 0]
  # There is deliberately no cut on the warning flag: every rank-0 object
  # carries warning bit 1 (flag = 2), so `== 0` returns an empty catalogue,
  # which an earlier version of this survey's query did. Nor on redshift
  # reliability, which is carried unfiltered so a threshold can be tuned
  # downstream. 
  # For calibration, of the 3,716,438 rank-0 rows: 
  # z_prob > 0.5 keeps 84.3%, > 0.9 keeps 51.3%, > 0.99 keeps 29.4%; 
  # z_rel > 0.5 keeps 56.5%, > 0.9 keeps 36.9%; 
  # cont_snr > 3 keeps 85.8%.

spectra:
  - adapter: url
    adapter_params:
      url_column: euclid_access_url
    format: generic
    arrays: [flux, error, wavelength, mask]

photometric_system:
  filters:
    vis: Euclid/VIS.vis
    y: Euclid/NISP.Y
    j: Euclid/NISP.J
    h: Euclid/NISP.H
  measurements:
    # Microjanskys AB already, so the conversion is the identity. VIS is a
    # PSF fit and NISP a template fit, which is why they are two families
    # rather than one with four bands.
    - system: linear
      bands: [vis]
      apertures: [psf]
      scale: 1.0
    - system: linear
      bands: [y, j, h]
      apertures: [templfit]
      scale: 1.0

spectroscopic_system:
  # Slitless NISP grism spectroscopy: there is no fibre at all, and a
  # plausible-looking number here would be a fabricated instrument
  # parameter -- worse than a null a model can mask.
  fiber_diameter: null
  # NISP RGS000/RGS180, a transmission grating combined with a prism
  # (Euclid Collaboration, *Euclid III: The NISP Instrument*).
  disperser: grism
  redshift_rank: 3
  flux_scale: 1.0e17          # native erg/s/cm2/Angstrom
  flux_calibration: absolute

# Euclid Q1 MER photometric catalogue: VIS+NISP astrometry and photometry,
# morphology, and detection-quality flags.
# Catalogue: the MER final catalogue, HATS at data.lsdb.io, matched with LSDB.
# Its columns are those of `euclid_q1.object` -- the table the spectroscopic
# query above joins on Data Lab -- spelled in upper case.
# Alias prefix `euclidq1p_` (p for photometric): `euclidq1s_` is already used
# by the separately-extracted Euclid Q1 spectroscopic table.
#
# Fluxes are uJy on the AB scale already, so homogenisation is the identity.
# Do not mix photometry families within one SED: `psf` is PSF-fitting on VIS,
# `templfit` is template-fitting on NISP, and they measure different things.
column_mapping:
  # OBJECT_ID: 'id'                                   # Euclid unique source identifier
  RIGHT_ASCENSION: 'ra'                               # source barycenter RA (SExtractor ALPHA_J2000), deg
  DECLINATION: 'dec'                                  # source barycenter Dec (SExtractor DELTA_J2000), deg

  FLUX_VIS_PSF: 'flux_psf_vis'                         # VIS band PSF-fitting flux (TPHOT), uJy
  FLUXERR_VIS_PSF: 'flux_psf_vis_err'                  # error on flux_vis_psf, uJy
  # FLUX_Y_TEMPLFIT: 'flux_templfit_y'                 # NIR Y band template-fitting flux (TPHOT), uJy
  # FLUXERR_Y_TEMPLFIT: 'flux_templfit_y_err'          # error on flux_y_templfit, uJy
  FLUX_J_TEMPLFIT: 'flux_templfit_j'                   # NIR J band template-fitting flux (TPHOT), uJy
  FLUXERR_J_TEMPLFIT: 'flux_templfit_j_err'            # error on flux_j_templfit, uJy
  # FLUX_H_TEMPLFIT: 'flux_templfit_h'                 # NIR H band template-fitting flux (TPHOT), uJy
  # FLUXERR_H_TEMPLFIT: 'flux_templfit_h_err'          # error on flux_h_templfit, uJy

  MU_MAX: 'mu_max'                                    # peak surface brightness above background in the detection band, mag/arcsec2
  SEGMENTATION_AREA: 'segmentation_area'              # isophotal area above the analysis threshold (SExtractor ISOAREA_IMAGE), pix
  ELLIPTICITY: 'ellipticity'                          # stretch of the object in the detection band, from its minor/major axes
  SEMIMAJOR_AXIS: 'semimajor_axis'                    # semi-major axis of the source (from Asterism), pix

  DET_QUALITY_FLAG: 'flag_quality'                    # detection-step flags; nonzero can indicate corrupted mag_stargal_sep
  SPURIOUS_FLAG: 'flag_spurious'                      # spurious source flag
  MUMAX_MINUS_MAG: 'mumax_minus_mag'                  # mu_max minus mag_stargal_sep, valid even for NIR-only sources, mag/arcsec2
  POINT_LIKE_FLAG: 'flag_point_source'                # set to 1 for VIS detections with det_quality_flag==0 and high point-source probability, else NaN
  EXTENDED_FLAG: 'flag_extended'                      # extended-source flag
```

### Schema of this file

| Key present            | Type                                       | Required   | Default   | Meaning                                                                                                                                                                                                                      |
|------------------------|--------------------------------------------|------------|-----------|------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `survey_slug`          | str                                        | yes        | —         | Canonical lowercase identifier for the survey and default column prefix.                                                                                                                                                     |
| `name`                 | str                                        | yes        | —         | Human-readable survey and data-release name used in logs, plans, and reports.                                                                                                                                                |
| `catalog`              | list of catalog entries                    | no         | []        | Remote or local catalogues queried for target extraction or crossmatching.                                                                                                                                                   |
| `crossmatch`           | mapping                                    | no         | None      | How a table of positions is matched against this survey. Required for a survey with an `lsdb` catalog; `crossmatch` null removes it, and a survey without it takes no part in `mosaics-crossmatch`.                          |
| `catalog_filters`      | mapping (prefix -> SQL expression or note) | no         | {}        | Post-hoc DuckDB SQL quality/depth/redshift cuts applied during production (nulls failing survey columns without dropping the row).                                                                                           |
| `target_selection`     | mapping                                    | no         | None      | How this survey seeds or joins the candidate table in the targets pipeline.                                                                                                                                                  |
| `spectra`              | list of spectra entries                    | no         | []        | 1D spectrum download backends (`url`, `template`, `sparcl`, `desi-healpix`).                                                                                                                                                 |
| `photometric_system`   | mapping                                    | no         | None      | Passband SVO filter identifiers, flux/magnitude homogenisation rules, and image pixel calibration.                                                                                                                           |
| `spectroscopic_system` | mapping                                    | no         | None      | Spectrograph fibre diameter, disperser, redshift tiebreak rank, and flux calibration facts.                                                                                                                                  |
| `column_mapping`       | mapping (remote_column -> alias)           | no         | {}        | Projection and column renaming applied when crossmatching against this survey’s photometric catalogue. The loader adds the catalog’s prefix to every alias, or the hosted survey’s for the columns under its `provides` key. |

### Adapter parameters

**Catalogue `euclidq1p` (`adapter: lsdb`):**

| Parameter           | Allowed Type   | Default               | Description                                                                                         |
|---------------------|----------------|-----------------------|-----------------------------------------------------------------------------------------------------|
| `path`              | str            | —                     | HTTP or S3 URI of the HATS catalogue root (may contain `{band}` or `{BAND}` when `per_band: true`). |
| `margin_cache`      | str            | None                  | URI of the HATS pre-computed margin cache table for boundary-safe spatial crossmatching.            |
| `storage_options`   | mapping        | {}                    | Filesystem options forwarded to `upath`/`fsspec` (e.g. `{'anon': True}` for public S3 buckets).     |
| `reshape`           | str            | None                  | Optional post-read reshaping strategy for nested or multi-row HATS tables.                          |
| `per_band`          | bool           | false                 | When `true`, opens one HATS catalogue per band by substituting `{band}`/`{BAND}` in `path`.         |
| `remote_ra_column`  | str            | HATS metadata default | Override for the remote right ascension column name.                                                |
| `remote_dec_column` | str            | HATS metadata default | Override for the remote declination column name.                                                    |

**Catalogue `euclidq1s` (`adapter: lsdb`):**

| Parameter           | Allowed Type   | Default               | Description                                                                                         |
|---------------------|----------------|-----------------------|-----------------------------------------------------------------------------------------------------|
| `path`              | str            | —                     | HTTP or S3 URI of the HATS catalogue root (may contain `{band}` or `{BAND}` when `per_band: true`). |
| `margin_cache`      | str            | None                  | URI of the HATS pre-computed margin cache table for boundary-safe spatial crossmatching.            |
| `storage_options`   | mapping        | {}                    | Filesystem options forwarded to `upath`/`fsspec` (e.g. `{'anon': True}` for public S3 buckets).     |
| `reshape`           | str            | None                  | Optional post-read reshaping strategy for nested or multi-row HATS tables.                          |
| `per_band`          | bool           | false                 | When `true`, opens one HATS catalogue per band by substituting `{band}`/`{BAND}` in `path`.         |
| `remote_ra_column`  | str            | HATS metadata default | Override for the remote right ascension column name.                                                |
| `remote_dec_column` | str            | HATS metadata default | Override for the remote declination column name.                                                    |
- **Spectra (`adapter: url`, format `generic`):** configured `adapter_params`: `url_column='euclid_access_url'`.

### Common changes

- **Independent subsystem toggling**: Photometric and spectroscopic catalogues are defined separately. Disable photometric crossmatch with `overrides[surveys/euclidq1].catalog.0.enabled: false`, or disable spectroscopic crossmatch with `overrides[surveys/euclidq1].catalog.1.enabled: false`.
- **Selecting additional photometric bands**: In `column_mapping`, uncomment extra NISP passbands to extract $Y$-band template-fitting flux (`FLUX_Y_TEMPLFIT: 'flux_templfit_y'`) or $H$-band template-fitting flux (`FLUX_H_TEMPLFIT: 'flux_templfit_h'`), or the primary source identifier (`OBJECT_ID: 'id'`); the loader adds the `euclidq1p_` prefix to every alias.
- **Tuning redshift reliability thresholds**: Target selection in `defs/surveys/euclidq1.yaml` retains candidate galaxy redshifts without cutting on reliability (`z_rel`) or posterior probability (`z_prob`). In downstream analysis, tune thresholds to balance sample purity and completeness (e.g. `euclidq1s_z_prob > 0.9` retains 51.3% of candidates; `euclidq1s_z_rel > 0.9` retains 36.9%; `euclidq1s_cont_snr > 3` retains 85.8%).
- **Spectroscopic redshift filter**: Override the redshift boundary in entrypoint configurations via `overrides[surveys/euclidq1].catalog_filters.euclidq1s` (default: `euclidq1s_z BETWEEN 0.0 AND 2.5`).

### Commands

```console
$ uv run -m mosaics.cli.spectrum euclidq1 -t test/objects_sample_1k.csv -o euclidq1_spectra.lance
$ uv run -m mosaics.crossmatch.cli run --survey surveys/euclidq1 -i test/objects_sample_1k.csv -o euclidq1_matched.parquet
$ uv run -m mosaics.cli.targets --config defs/targets.yaml -o ./targets_run --dec-range 0 1 --plan plan.yaml
```

## Data served by the archive

- **Prefix naming convention (`euclidq1p_` vs `euclidq1s_`)**: Because Euclid Q1 supplies both photometric and spectroscopic catalogues, MOSAICS adds the prefix itself, never written in `column_mapping`: `euclidq1p_*` for MER photometry and `euclidq1s_*` for SPE spectroscopy, preventing namespace collisions in merged datasets.
- **Photometric units and aperture families**: MER fluxes are calibrated in microjanskys ($\mu\text{Jy}$) on the AB magnitude scale (`system: linear`, `scale: 1.0`). Note that VIS fluxes are derived from PSF fitting (TPHOT, `apertures: [psf]`), whereas NISP near-infrared fluxes ($Y, J, H$) are derived from template fitting (TPHOT, `apertures: [templfit]`). These two measurement families represent different physical models and should not be mixed across passbands within a single SED without proper calibration.
- **Slitless grism spectroscopy**: NISP spectroscopy is slitless; there are no physical fibres (`fiber_diameter: null`). Spectra are combined 1D FITS tables (`DpdSirCombinedSpectra`) with absolute flux calibration scaled to $10^{-17}\text{ erg s}^{-1}\text{ cm}^{-2}\text{ \AA}^{-1}$ (`flux_scale: 1.0e17`).
- **Target selection filter caveats**:
  - `z_galaxy_candidates_spe_rank_rank0 == 0`: Selects the preferred galaxy redshift solution. The SPE pipeline outputs up to five ranked solutions (ranks 0–4); selecting without rank filtering repeats each object up to five times.
  - `z_spe_error_flag == 0`: Restricts to records without fatal pipeline errors (context stage, reading errors, NaNs, or short spectra).
  - `mer_spurious_flag == 0` and `mer_det_quality_flag == 0`: Excludes spurious detections and corrupted deblending.
  - **Warning flag warning**: Do not filter on `z_spe_warning_flag == 0`. In Q1, every rank-0 galaxy solution carries warning bit 1 (`flag = 2`); filtering on `warning_flag == 0` results in an empty dataset.
- **Column descriptions**: The HATS files carry column names and types but no descriptions. MOSAICS takes descriptions from the `euclid_q1` tables of NOIRLab Astro Data Lab, in three steps: the same column name; the same name after removing the merged catalogue’s table prefix (`mer_`, `morph_`, `phz_`, `z_`); or, for a column that differs from a described one by a single name token that appears word for word in that description (`megacam` in place of `decam`, or `2fwhm` in place of `1fwhm`, which the description writes “1 FWHM”), the described column’s text with the token replaced, marked with †. The token that names the quantity itself (the first one after the table prefix) is never replaced. A column with no such counterpart, for example the SED, physical-parameter, emission-line and model families of the merged catalogue, shows `—`. Data Lab keeps at most 64 characters of each description, so a long one ends mid-word.
- **Data access**: Photometry is served from the LSDB HATS collection (`https://data.lsdb.io/hats/euclid_q1/euclid_q1_merFinalCatalog`, mirrored at LineA). Spectroscopic merged objects are read directly from NASA IRSA’s public AWS S3 bucket (`s3://nasa-irsa-euclid-q1/contributed/q1/merged_objects/hats/euclid_q1_merged_objects-hats`) using anonymous S3 credentials.

### Catalogue: `euclid_q1.spe_galaxy_candidates`

**Service URL:** `s3://nasa-irsa-euclid-q1/contributed/q1/merged_objects/hats/euclid_q1_merged_objects-hats` | **Published rows:** 29,953,430

*Showing 100 of 1591 total archive columns (all MOSAICS-mapped columns plus primary table fields; full 1591-column schema is stored in `docs/_data/sources/euclidq1/catalog_euclid_spe.json`).*

| Column                                   | Type     | Unit          | UCD                 | Description                                                        |
|------------------------------------------|----------|---------------|---------------------|--------------------------------------------------------------------|
| `object_id`                              | `int64`  | `—`           | `meta.id;meta.main` | Euclid unique source identifier                                    |
| `ra`                                     | `double` | `—`           | `—`                 | —                                                                  |
| `dec`                                    | `double` | `—`           | `—`                 | —                                                                  |
| `mer_spurious_flag`                      | `int16`  | `—`           | `—`                 | Spurious source flag                                               |
| `mer_det_quality_flag`                   | `int16`  | `—`           | `—`                 | Detection step flags that could indicate the possible corruption   |
| `mer_mu_max`                             | `float`  | `mag/arcsec2` | `—`                 | Peak surface brightness above the background in the detection ba   |
| `mer_segmentation_area`                  | `int32`  | `pix`         | `—`                 | Isophotal area of the source above the analysis threshold (SExtr   |
| `z_spe_error_flag`                       | `int16`  | `—`           | `—`                 | Processing error flag                                              |
| `z_spe_warning_flag`                     | `int16`  | `—`           | `—`                 | Processing warning flag                                            |
| `z_spe_grism`                            | `binary` | `—`           | `—`                 | The type of GRISM [BGS/RGS/BGS|RGS]                                |
| `z_spe_npix`                             | `int16`  | `—`           | `—`                 | The number of pixels                                               |
| `z_spe_n_dith_max`                       | `int16`  | `—`           | `—`                 | Maximum number of dithers                                          |
| `z_spe_star_prob`                        | `float`  | `—`           | `—`                 | The probability of the object to be a star                         |
| `z_spe_gal_prob`                         | `float`  | `—`           | `—`                 | The probability of the object to be a galaxy                       |
| `z_spe_qso_prob`                         | `float`  | `—`           | `—`                 | The probability of the object to be a quasar                       |
| `z_spe_class`                            | `binary` | `—`           | `—`                 | The classification of the object related to highest probability    |
| `z_galaxy_candidates_spe_z_rank0`        | `float`  | `—`           | `—`                 | —                                                                  |
| `z_galaxy_candidates_spe_z_err_rank0`    | `float`  | `—`           | `—`                 | —                                                                  |
| `z_galaxy_candidates_spe_z_prob_rank0`   | `float`  | `—`           | `—`                 | —                                                                  |
| `z_galaxy_candidates_spe_z_rel_rank0`    | `float`  | `—`           | `—`                 | —                                                                  |
| `z_galaxy_candidates_spe_subclass_rank0` | `binary` | `—`           | `—`                 | —                                                                  |
| `z_galaxy_candidates_spe_cont_snr_rank0` | `float`  | `—`           | `—`                 | —                                                                  |
| `tileid`                                 | `int64`  | `—`           | `—`                 | —                                                                  |
| `mer_right_ascension_psf_fitting`        | `double` | `—`           | `—`                 | —                                                                  |
| `mer_declination_psf_fitting`            | `double` | `—`           | `—`                 | —                                                                  |
| `mer_segmentation_map_id`                | `int64`  | `—`           | `—`                 | Source ID in the associated segmentation map                       |
| `mer_vis_det`                            | `int16`  | `—`           | `—`                 | Flag to indicate if the source is detected in the VIS mosaic (1)   |
| `mer_flux_vis_1fwhm_aper`                | `float`  | `uJy`         | `—`                 | VIS band source aperture photometry flux (1 FWHM diameter) on PS   |
| `mer_flux_vis_2fwhm_aper`                | `float`  | `—`           | `—`                 | VIS band source aperture photometry flux (2 FWHM diameter) on PS † |
| `mer_flux_vis_3fwhm_aper`                | `float`  | `—`           | `—`                 | VIS band source aperture photometry flux (3 FWHM diameter) on PS † |
| `mer_flux_vis_4fwhm_aper`                | `float`  | `—`           | `—`                 | VIS band source aperture photometry flux (4 FWHM diameter) on PS † |
| `mer_flux_y_1fwhm_aper`                  | `float`  | `uJy`         | `—`                 | NIR Y band source aperture photometry flux (1 FWHM diameter) on    |
| `mer_flux_y_2fwhm_aper`                  | `float`  | `—`           | `—`                 | NIR Y band source aperture photometry flux (2 FWHM diameter) on †  |
| `mer_flux_y_3fwhm_aper`                  | `float`  | `—`           | `—`                 | NIR Y band source aperture photometry flux (3 FWHM diameter) on †  |
| `mer_flux_y_4fwhm_aper`                  | `float`  | `—`           | `—`                 | NIR Y band source aperture photometry flux (4 FWHM diameter) on †  |
| `mer_flux_j_1fwhm_aper`                  | `float`  | `uJy`         | `—`                 | NIR J band source aperture photometry flux (1 FWHM diameter) on    |
| `mer_flux_j_2fwhm_aper`                  | `float`  | `—`           | `—`                 | NIR J band source aperture photometry flux (2 FWHM diameter) on †  |
| `mer_flux_j_3fwhm_aper`                  | `float`  | `—`           | `—`                 | NIR J band source aperture photometry flux (3 FWHM diameter) on †  |
| `mer_flux_j_4fwhm_aper`                  | `float`  | `—`           | `—`                 | NIR J band source aperture photometry flux (4 FWHM diameter) on †  |
| `mer_flux_h_1fwhm_aper`                  | `float`  | `uJy`         | `—`                 | NIR H band source aperture photometry flux (1 FWHM diameter) on    |
| `mer_flux_h_2fwhm_aper`                  | `float`  | `—`           | `—`                 | NIR H band source aperture photometry flux (2 FWHM diameter) on †  |
| `mer_flux_h_3fwhm_aper`                  | `float`  | `—`           | `—`                 | NIR H band source aperture photometry flux (3 FWHM diameter) on †  |
| `mer_flux_h_4fwhm_aper`                  | `float`  | `—`           | `—`                 | NIR H band source aperture photometry flux (4 FWHM diameter) on †  |
| `mer_flux_nir_stack_1fwhm_aper`          | `float`  | `uJy`         | `—`                 | NIR stack band source aperture photometry flux (1 FWHM diameter)   |
| `mer_flux_nir_stack_2fwhm_aper`          | `float`  | `—`           | `—`                 | NIR stack band source aperture photometry flux (2 FWHM diameter) † |
| `mer_flux_nir_stack_3fwhm_aper`          | `float`  | `—`           | `—`                 | NIR stack band source aperture photometry flux (3 FWHM diameter) † |
| `mer_flux_nir_stack_4fwhm_aper`          | `float`  | `—`           | `—`                 | NIR stack band source aperture photometry flux (4 FWHM diameter) † |
| `mer_flux_u_ext_decam_1fwhm_aper`        | `float`  | `—`           | `—`                 | Uext DECAM band source aperture photometry flux (1 FWHM diamet †   |
| `mer_flux_u_ext_decam_2fwhm_aper`        | `float`  | `—`           | `—`                 | —                                                                  |
| `mer_flux_u_ext_decam_3fwhm_aper`        | `float`  | `—`           | `—`                 | —                                                                  |
| `mer_flux_u_ext_decam_4fwhm_aper`        | `float`  | `—`           | `—`                 | —                                                                  |
| `mer_flux_g_ext_decam_1fwhm_aper`        | `float`  | `uJy`         | `—`                 | Gext DECam band source aperture photometry flux (1 FWHM diameter   |
| `mer_flux_g_ext_decam_2fwhm_aper`        | `float`  | `—`           | `—`                 | Gext DECam band source aperture photometry flux (2 FWHM diameter † |
| `mer_flux_g_ext_decam_3fwhm_aper`        | `float`  | `—`           | `—`                 | Gext DECam band source aperture photometry flux (3 FWHM diameter † |
| `mer_flux_g_ext_decam_4fwhm_aper`        | `float`  | `—`           | `—`                 | Gext DECam band source aperture photometry flux (4 FWHM diameter † |
| `mer_flux_r_ext_decam_1fwhm_aper`        | `float`  | `uJy`         | `—`                 | Rext DECam band source aperture photometry flux (1 FWHM diameter   |
| `mer_flux_r_ext_decam_2fwhm_aper`        | `float`  | `—`           | `—`                 | Rext DECam band source aperture photometry flux (2 FWHM diameter † |
| `mer_flux_r_ext_decam_3fwhm_aper`        | `float`  | `—`           | `—`                 | Rext DECam band source aperture photometry flux (3 FWHM diameter † |
| `mer_flux_r_ext_decam_4fwhm_aper`        | `float`  | `—`           | `—`                 | Rext DECam band source aperture photometry flux (4 FWHM diameter † |
| `mer_flux_i_ext_decam_1fwhm_aper`        | `float`  | `uJy`         | `—`                 | Iext DECam band source aperture photometry flux (1 FWHM diameter   |
| `mer_flux_i_ext_decam_2fwhm_aper`        | `float`  | `—`           | `—`                 | Iext DECam band source aperture photometry flux (2 FWHM diameter † |
| `mer_flux_i_ext_decam_3fwhm_aper`        | `float`  | `—`           | `—`                 | Iext DECam band source aperture photometry flux (3 FWHM diameter † |
| `mer_flux_i_ext_decam_4fwhm_aper`        | `float`  | `—`           | `—`                 | Iext DECam band source aperture photometry flux (4 FWHM diameter † |
| `mer_flux_z_ext_decam_1fwhm_aper`        | `float`  | `uJy`         | `—`                 | Zext DECam band source aperture photometry flux (1 FWHM diameter   |
| `mer_flux_z_ext_decam_2fwhm_aper`        | `float`  | `—`           | `—`                 | Zext DECam band source aperture photometry flux (2 FWHM diameter † |
| `mer_flux_z_ext_decam_3fwhm_aper`        | `float`  | `—`           | `—`                 | Zext DECam band source aperture photometry flux (3 FWHM diameter † |
| `mer_flux_z_ext_decam_4fwhm_aper`        | `float`  | `—`           | `—`                 | Zext DECam band source aperture photometry flux (4 FWHM diameter † |
| `mer_flux_u_ext_lsst_1fwhm_aper`         | `float`  | `—`           | `—`                 | Uext LSST band source aperture photometry flux (1 FWHM diamet †    |
| `mer_flux_u_ext_lsst_2fwhm_aper`         | `float`  | `—`           | `—`                 | —                                                                  |
| `mer_flux_u_ext_lsst_3fwhm_aper`         | `float`  | `—`           | `—`                 | —                                                                  |
| `mer_flux_u_ext_lsst_4fwhm_aper`         | `float`  | `—`           | `—`                 | —                                                                  |
| `mer_flux_g_ext_lsst_1fwhm_aper`         | `float`  | `—`           | `—`                 | —                                                                  |
| `mer_flux_g_ext_lsst_2fwhm_aper`         | `float`  | `—`           | `—`                 | —                                                                  |
| `mer_flux_g_ext_lsst_3fwhm_aper`         | `float`  | `—`           | `—`                 | —                                                                  |
| `mer_flux_g_ext_lsst_4fwhm_aper`         | `float`  | `—`           | `—`                 | —                                                                  |
| `mer_flux_r_ext_lsst_1fwhm_aper`         | `float`  | `—`           | `—`                 | —                                                                  |
| `mer_flux_r_ext_lsst_2fwhm_aper`         | `float`  | `—`           | `—`                 | —                                                                  |
| `mer_flux_r_ext_lsst_3fwhm_aper`         | `float`  | `—`           | `—`                 | —                                                                  |
| `mer_flux_r_ext_lsst_4fwhm_aper`         | `float`  | `—`           | `—`                 | —                                                                  |
| `mer_flux_i_ext_lsst_1fwhm_aper`         | `float`  | `—`           | `—`                 | Iext LSST band source aperture photometry flux (1 FWHM diameter †  |
| `mer_flux_i_ext_lsst_2fwhm_aper`         | `float`  | `—`           | `—`                 | —                                                                  |
| `mer_flux_i_ext_lsst_3fwhm_aper`         | `float`  | `—`           | `—`                 | —                                                                  |
| `mer_flux_i_ext_lsst_4fwhm_aper`         | `float`  | `—`           | `—`                 | —                                                                  |
| `mer_flux_z_ext_lsst_1fwhm_aper`         | `float`  | `—`           | `—`                 | —                                                                  |
| `mer_flux_z_ext_lsst_2fwhm_aper`         | `float`  | `—`           | `—`                 | —                                                                  |
| `mer_flux_z_ext_lsst_3fwhm_aper`         | `float`  | `—`           | `—`                 | —                                                                  |
| `mer_flux_z_ext_lsst_4fwhm_aper`         | `float`  | `—`           | `—`                 | —                                                                  |
| `mer_flux_u_ext_megacam_1fwhm_aper`      | `float`  | `uJy`         | `—`                 | Uext Megacam band source aperture photometry flux (1 FWHM diamet   |
| `mer_flux_u_ext_megacam_2fwhm_aper`      | `float`  | `—`           | `—`                 | Uext Megacam band source aperture photometry flux (2 FWHM diamet † |
| `mer_flux_u_ext_megacam_3fwhm_aper`      | `float`  | `—`           | `—`                 | Uext Megacam band source aperture photometry flux (3 FWHM diamet † |
| `mer_flux_u_ext_megacam_4fwhm_aper`      | `float`  | `—`           | `—`                 | Uext Megacam band source aperture photometry flux (4 FWHM diamet † |
| `mer_flux_r_ext_megacam_1fwhm_aper`      | `float`  | `uJy`         | `—`                 | Rext Megacam band source aperture photometry flux (1 FWHM diamet   |
| `mer_flux_r_ext_megacam_2fwhm_aper`      | `float`  | `—`           | `—`                 | Rext Megacam band source aperture photometry flux (2 FWHM diamet † |
| `mer_flux_r_ext_megacam_3fwhm_aper`      | `float`  | `—`           | `—`                 | Rext Megacam band source aperture photometry flux (3 FWHM diamet † |
| `mer_flux_r_ext_megacam_4fwhm_aper`      | `float`  | `—`           | `—`                 | Rext Megacam band source aperture photometry flux (4 FWHM diamet † |
| `mer_flux_g_ext_jpcam_1fwhm_aper`        | `float`  | `—`           | `—`                 | —                                                                  |
| `mer_flux_g_ext_jpcam_2fwhm_aper`        | `float`  | `—`           | `—`                 | —                                                                  |
| `mer_flux_g_ext_jpcam_3fwhm_aper`        | `float`  | `—`           | `—`                 | —                                                                  |
| `mer_flux_g_ext_jpcam_4fwhm_aper`        | `float`  | `—`           | `—`                 | —                                                                  |
| `mer_flux_i_ext_panstarrs_1fwhm_aper`    | `float`  | `uJy`         | `—`                 | Iext PS band source aperture photometry flux (1 FWHM diameter) o   |

† Derived description: this column differs from a described sibling column by one name token, and that token appears in the sibling’s description (an aperture `1fwhm` as “1 FWHM”), so the description is the sibling’s with that token replaced (for example `megacam` by `decam`). The archive itself describes only the sibling. The rule and the sibling of each column are in the harvested JSON (`description_rule`, `description_sibling`).

### Catalogue: `euclid_q1.object`

**Service URL:** `https://data.lsdb.io/hats/euclid_q1/euclid_q1_merFinalCatalog`

*Showing 100 of 472 total archive columns (all MOSAICS-mapped columns plus primary table fields; full 472-column schema is stored in `docs/_data/sources/euclidq1/hats_euclid_mer.json`).*

| Column                            | Type     | Unit          | UCD                    | Description                                                        |
|-----------------------------------|----------|---------------|------------------------|--------------------------------------------------------------------|
| `RIGHT_ASCENSION`                 | `double` | `deg`         | `pos.eq.ra;meta.main`  | Source barycenter RA coordinate (SExtractor ALPHA_J2000) decimal   |
| `DECLINATION`                     | `double` | `deg`         | `pos.eq.dec;meta.main` | Source barycenter DEC coordinate (SExtractor DELTA_J2000) decima   |
| `FLUX_J_TEMPLFIT`                 | `float`  | `uJy`         | `—`                    | NIR J band source template fitting flux (TPHOT measurement)        |
| `FLUXERR_J_TEMPLFIT`              | `float`  | `uJy`         | `—`                    | NIR J band source template fitting flux error (TPHOT measurement   |
| `FLUX_VIS_PSF`                    | `float`  | `uJy`         | `—`                    | VIS band source PSF-fitting photometry flux (TPHOT measurement)    |
| `FLUXERR_VIS_PSF`                 | `float`  | `uJy`         | `—`                    | VIS band source PSF-fitting photometry flux error (TPHOT measure   |
| `POINT_LIKE_FLAG`                 | `int16`  | `—`           | `—`                    | Point-like flag : flag set to 1 for VIS detections with (DET_QUA   |
| `EXTENDED_FLAG`                   | `int16`  | `—`           | `—`                    | Extended source flag                                               |
| `SPURIOUS_FLAG`                   | `int16`  | `—`           | `—`                    | Spurious source flag                                               |
| `DET_QUALITY_FLAG`                | `int16`  | `—`           | `—`                    | Detection step flags that could indicate the possible corruption   |
| `MU_MAX`                          | `float`  | `mag/arcsec2` | `—`                    | Peak surface brightness above the background in the detection ba   |
| `MUMAX_MINUS_MAG`                 | `float`  | `mag/arcsec2` | `—`                    | The difference between MU_MAX and MAG_STARGAL_SEP, valid even fo   |
| `SEGMENTATION_AREA`               | `int32`  | `pix`         | `—`                    | Isophotal area of the source above the analysis threshold (SExtr   |
| `SEMIMAJOR_AXIS`                  | `float`  | `pix`         | `—`                    | Semi-major axis of the source (from Asterism)                      |
| `ELLIPTICITY`                     | `float`  | `—`           | `—`                    | A parametrization of how stretched an object is in the detection   |
| `OBJECT_ID`                       | `int64`  | `—`           | `meta.id;meta.main`    | Euclid unique source identifier                                    |
| `RIGHT_ASCENSION_PSF_FITTING`     | `double` | `—`           | `—`                    | —                                                                  |
| `DECLINATION_PSF_FITTING`         | `double` | `—`           | `—`                    | —                                                                  |
| `SEGMENTATION_MAP_ID`             | `int64`  | `—`           | `—`                    | Source ID in the associated segmentation map                       |
| `VIS_DET`                         | `int16`  | `—`           | `—`                    | Flag to indicate if the source is detected in the VIS mosaic (1)   |
| `FLUX_VIS_1FWHM_APER`             | `float`  | `uJy`         | `—`                    | VIS band source aperture photometry flux (1 FWHM diameter) on PS   |
| `FLUX_VIS_2FWHM_APER`             | `float`  | `—`           | `—`                    | VIS band source aperture photometry flux (2 FWHM diameter) on PS † |
| `FLUX_VIS_3FWHM_APER`             | `float`  | `—`           | `—`                    | VIS band source aperture photometry flux (3 FWHM diameter) on PS † |
| `FLUX_VIS_4FWHM_APER`             | `float`  | `—`           | `—`                    | VIS band source aperture photometry flux (4 FWHM diameter) on PS † |
| `FLUX_Y_1FWHM_APER`               | `float`  | `uJy`         | `—`                    | NIR Y band source aperture photometry flux (1 FWHM diameter) on    |
| `FLUX_Y_2FWHM_APER`               | `float`  | `—`           | `—`                    | NIR Y band source aperture photometry flux (2 FWHM diameter) on †  |
| `FLUX_Y_3FWHM_APER`               | `float`  | `—`           | `—`                    | NIR Y band source aperture photometry flux (3 FWHM diameter) on †  |
| `FLUX_Y_4FWHM_APER`               | `float`  | `—`           | `—`                    | NIR Y band source aperture photometry flux (4 FWHM diameter) on †  |
| `FLUX_J_1FWHM_APER`               | `float`  | `uJy`         | `—`                    | NIR J band source aperture photometry flux (1 FWHM diameter) on    |
| `FLUX_J_2FWHM_APER`               | `float`  | `—`           | `—`                    | NIR J band source aperture photometry flux (2 FWHM diameter) on †  |
| `FLUX_J_3FWHM_APER`               | `float`  | `—`           | `—`                    | NIR J band source aperture photometry flux (3 FWHM diameter) on †  |
| `FLUX_J_4FWHM_APER`               | `float`  | `—`           | `—`                    | NIR J band source aperture photometry flux (4 FWHM diameter) on †  |
| `FLUX_H_1FWHM_APER`               | `float`  | `uJy`         | `—`                    | NIR H band source aperture photometry flux (1 FWHM diameter) on    |
| `FLUX_H_2FWHM_APER`               | `float`  | `—`           | `—`                    | NIR H band source aperture photometry flux (2 FWHM diameter) on †  |
| `FLUX_H_3FWHM_APER`               | `float`  | `—`           | `—`                    | NIR H band source aperture photometry flux (3 FWHM diameter) on †  |
| `FLUX_H_4FWHM_APER`               | `float`  | `—`           | `—`                    | NIR H band source aperture photometry flux (4 FWHM diameter) on †  |
| `FLUX_NIR_STACK_1FWHM_APER`       | `float`  | `uJy`         | `—`                    | NIR stack band source aperture photometry flux (1 FWHM diameter)   |
| `FLUX_NIR_STACK_2FWHM_APER`       | `float`  | `—`           | `—`                    | NIR stack band source aperture photometry flux (2 FWHM diameter) † |
| `FLUX_NIR_STACK_3FWHM_APER`       | `float`  | `—`           | `—`                    | NIR stack band source aperture photometry flux (3 FWHM diameter) † |
| `FLUX_NIR_STACK_4FWHM_APER`       | `float`  | `—`           | `—`                    | NIR stack band source aperture photometry flux (4 FWHM diameter) † |
| `FLUX_U_EXT_DECAM_1FWHM_APER`     | `float`  | `—`           | `—`                    | Uext DECAM band source aperture photometry flux (1 FWHM diamet †   |
| `FLUX_U_EXT_DECAM_2FWHM_APER`     | `float`  | `—`           | `—`                    | —                                                                  |
| `FLUX_U_EXT_DECAM_3FWHM_APER`     | `float`  | `—`           | `—`                    | —                                                                  |
| `FLUX_U_EXT_DECAM_4FWHM_APER`     | `float`  | `—`           | `—`                    | —                                                                  |
| `FLUX_G_EXT_DECAM_1FWHM_APER`     | `float`  | `uJy`         | `—`                    | Gext DECam band source aperture photometry flux (1 FWHM diameter   |
| `FLUX_G_EXT_DECAM_2FWHM_APER`     | `float`  | `—`           | `—`                    | Gext DECam band source aperture photometry flux (2 FWHM diameter † |
| `FLUX_G_EXT_DECAM_3FWHM_APER`     | `float`  | `—`           | `—`                    | Gext DECam band source aperture photometry flux (3 FWHM diameter † |
| `FLUX_G_EXT_DECAM_4FWHM_APER`     | `float`  | `—`           | `—`                    | Gext DECam band source aperture photometry flux (4 FWHM diameter † |
| `FLUX_R_EXT_DECAM_1FWHM_APER`     | `float`  | `uJy`         | `—`                    | Rext DECam band source aperture photometry flux (1 FWHM diameter   |
| `FLUX_R_EXT_DECAM_2FWHM_APER`     | `float`  | `—`           | `—`                    | Rext DECam band source aperture photometry flux (2 FWHM diameter † |
| `FLUX_R_EXT_DECAM_3FWHM_APER`     | `float`  | `—`           | `—`                    | Rext DECam band source aperture photometry flux (3 FWHM diameter † |
| `FLUX_R_EXT_DECAM_4FWHM_APER`     | `float`  | `—`           | `—`                    | Rext DECam band source aperture photometry flux (4 FWHM diameter † |
| `FLUX_I_EXT_DECAM_1FWHM_APER`     | `float`  | `uJy`         | `—`                    | Iext DECam band source aperture photometry flux (1 FWHM diameter   |
| `FLUX_I_EXT_DECAM_2FWHM_APER`     | `float`  | `—`           | `—`                    | Iext DECam band source aperture photometry flux (2 FWHM diameter † |
| `FLUX_I_EXT_DECAM_3FWHM_APER`     | `float`  | `—`           | `—`                    | Iext DECam band source aperture photometry flux (3 FWHM diameter † |
| `FLUX_I_EXT_DECAM_4FWHM_APER`     | `float`  | `—`           | `—`                    | Iext DECam band source aperture photometry flux (4 FWHM diameter † |
| `FLUX_Z_EXT_DECAM_1FWHM_APER`     | `float`  | `uJy`         | `—`                    | Zext DECam band source aperture photometry flux (1 FWHM diameter   |
| `FLUX_Z_EXT_DECAM_2FWHM_APER`     | `float`  | `—`           | `—`                    | Zext DECam band source aperture photometry flux (2 FWHM diameter † |
| `FLUX_Z_EXT_DECAM_3FWHM_APER`     | `float`  | `—`           | `—`                    | Zext DECam band source aperture photometry flux (3 FWHM diameter † |
| `FLUX_Z_EXT_DECAM_4FWHM_APER`     | `float`  | `—`           | `—`                    | Zext DECam band source aperture photometry flux (4 FWHM diameter † |
| `FLUX_U_EXT_LSST_1FWHM_APER`      | `float`  | `—`           | `—`                    | Uext LSST band source aperture photometry flux (1 FWHM diamet †    |
| `FLUX_U_EXT_LSST_2FWHM_APER`      | `float`  | `—`           | `—`                    | —                                                                  |
| `FLUX_U_EXT_LSST_3FWHM_APER`      | `float`  | `—`           | `—`                    | —                                                                  |
| `FLUX_U_EXT_LSST_4FWHM_APER`      | `float`  | `—`           | `—`                    | —                                                                  |
| `FLUX_G_EXT_LSST_1FWHM_APER`      | `float`  | `—`           | `—`                    | —                                                                  |
| `FLUX_G_EXT_LSST_2FWHM_APER`      | `float`  | `—`           | `—`                    | —                                                                  |
| `FLUX_G_EXT_LSST_3FWHM_APER`      | `float`  | `—`           | `—`                    | —                                                                  |
| `FLUX_G_EXT_LSST_4FWHM_APER`      | `float`  | `—`           | `—`                    | —                                                                  |
| `FLUX_R_EXT_LSST_1FWHM_APER`      | `float`  | `—`           | `—`                    | —                                                                  |
| `FLUX_R_EXT_LSST_2FWHM_APER`      | `float`  | `—`           | `—`                    | —                                                                  |
| `FLUX_R_EXT_LSST_3FWHM_APER`      | `float`  | `—`           | `—`                    | —                                                                  |
| `FLUX_R_EXT_LSST_4FWHM_APER`      | `float`  | `—`           | `—`                    | —                                                                  |
| `FLUX_I_EXT_LSST_1FWHM_APER`      | `float`  | `—`           | `—`                    | Iext LSST band source aperture photometry flux (1 FWHM diameter †  |
| `FLUX_I_EXT_LSST_2FWHM_APER`      | `float`  | `—`           | `—`                    | —                                                                  |
| `FLUX_I_EXT_LSST_3FWHM_APER`      | `float`  | `—`           | `—`                    | —                                                                  |
| `FLUX_I_EXT_LSST_4FWHM_APER`      | `float`  | `—`           | `—`                    | —                                                                  |
| `FLUX_Z_EXT_LSST_1FWHM_APER`      | `float`  | `—`           | `—`                    | —                                                                  |
| `FLUX_Z_EXT_LSST_2FWHM_APER`      | `float`  | `—`           | `—`                    | —                                                                  |
| `FLUX_Z_EXT_LSST_3FWHM_APER`      | `float`  | `—`           | `—`                    | —                                                                  |
| `FLUX_Z_EXT_LSST_4FWHM_APER`      | `float`  | `—`           | `—`                    | —                                                                  |
| `FLUX_U_EXT_MEGACAM_1FWHM_APER`   | `float`  | `uJy`         | `—`                    | Uext Megacam band source aperture photometry flux (1 FWHM diamet   |
| `FLUX_U_EXT_MEGACAM_2FWHM_APER`   | `float`  | `—`           | `—`                    | Uext Megacam band source aperture photometry flux (2 FWHM diamet † |
| `FLUX_U_EXT_MEGACAM_3FWHM_APER`   | `float`  | `—`           | `—`                    | Uext Megacam band source aperture photometry flux (3 FWHM diamet † |
| `FLUX_U_EXT_MEGACAM_4FWHM_APER`   | `float`  | `—`           | `—`                    | Uext Megacam band source aperture photometry flux (4 FWHM diamet † |
| `FLUX_R_EXT_MEGACAM_1FWHM_APER`   | `float`  | `uJy`         | `—`                    | Rext Megacam band source aperture photometry flux (1 FWHM diamet   |
| `FLUX_R_EXT_MEGACAM_2FWHM_APER`   | `float`  | `—`           | `—`                    | Rext Megacam band source aperture photometry flux (2 FWHM diamet † |
| `FLUX_R_EXT_MEGACAM_3FWHM_APER`   | `float`  | `—`           | `—`                    | Rext Megacam band source aperture photometry flux (3 FWHM diamet † |
| `FLUX_R_EXT_MEGACAM_4FWHM_APER`   | `float`  | `—`           | `—`                    | Rext Megacam band source aperture photometry flux (4 FWHM diamet † |
| `FLUX_G_EXT_JPCAM_1FWHM_APER`     | `float`  | `—`           | `—`                    | —                                                                  |
| `FLUX_G_EXT_JPCAM_2FWHM_APER`     | `float`  | `—`           | `—`                    | —                                                                  |
| `FLUX_G_EXT_JPCAM_3FWHM_APER`     | `float`  | `—`           | `—`                    | —                                                                  |
| `FLUX_G_EXT_JPCAM_4FWHM_APER`     | `float`  | `—`           | `—`                    | —                                                                  |
| `FLUX_I_EXT_PANSTARRS_1FWHM_APER` | `float`  | `uJy`         | `—`                    | Iext PS band source aperture photometry flux (1 FWHM diameter) o   |
| `FLUX_I_EXT_PANSTARRS_2FWHM_APER` | `float`  | `—`           | `—`                    | Iext PS band source aperture photometry flux (2 FWHM diameter) o † |
| `FLUX_I_EXT_PANSTARRS_3FWHM_APER` | `float`  | `—`           | `—`                    | Iext PS band source aperture photometry flux (3 FWHM diameter) o † |
| `FLUX_I_EXT_PANSTARRS_4FWHM_APER` | `float`  | `—`           | `—`                    | Iext PS band source aperture photometry flux (4 FWHM diameter) o † |
| `FLUX_Z_EXT_PANSTARRS_1FWHM_APER` | `float`  | `—`           | `—`                    | —                                                                  |
| `FLUX_Z_EXT_PANSTARRS_2FWHM_APER` | `float`  | `—`           | `—`                    | —                                                                  |
| `FLUX_Z_EXT_PANSTARRS_3FWHM_APER` | `float`  | `—`           | `—`                    | —                                                                  |
| `FLUX_Z_EXT_PANSTARRS_4FWHM_APER` | `float`  | `—`           | `—`                    | —                                                                  |

† Derived description: this column differs from a described sibling column by one name token, and that token appears in the sibling’s description (an aperture `1fwhm` as “1 FWHM”), so the description is the sibling’s with that token replaced (for example `megacam` by `decam`). The archive itself describes only the sibling. The rule and the sibling of each column are in the harvested JSON (`description_rule`, `description_sibling`).

### Images

No image cutout backends are configured for this survey.

### Spectra

- **Fibre diameter:** `None` | **Disperser:** `grism` | **Redshift rank:** `3` | **Flux calibration:** `absolute` | **Flux scale:** `1e+17`
- **Adapter `url`:** decoder format `generic`, output parallel arrays `['flux', 'error', 'wavelength', 'mask']`, params `{'url_column': 'euclid_access_url'}`.
  - **hdus:** Combined 1D spectrum table extension (`WAVELENGTH`, `SIGNAL`, `VAR`, `MASK`).
  - **wavelength_range_angstrom:** 12060 - 18920 A (NISP RG_0 + RG_294 + RG_m294 red grisms)
  - **resolution:** R ~ 450 (slitless grism)
  - **flux_calibration:** absolute (scaled to `10^-17 erg s^-1 cm^-2 A^-1`)
  - **arrays:** [‘flux’, ‘error’, ‘wavelength’, ‘mask’]

## References and acknowledgement

- Euclid Collaboration: Mellier, Y., et al. (2024), *A&A* — Euclid Quick Release 1.
- Euclid Collaboration: Scaramella, R., et al. (2022), *A&A*, 662, A112 (`doi:10.1051/0004-6361/202141938`) — Euclid preparation. I. The Euclid Wide Survey.
- Euclid Collaboration: Cropper, M., et al. (2024), *A&A* — Euclid: The VIS instrument.
- Euclid Collaboration: Jahnke, K., et al. (2024), *A&A* — Euclid: The NISP instrument.
- NASA IRSA Euclid Archive: [https://irsa.ipac.caltech.edu/data/Euclid/](https://irsa.ipac.caltech.edu/data/Euclid/).
- LSDB HATS Collection: [https://data.lsdb.io/hats/euclid_q1/](https://data.lsdb.io/hats/euclid_q1/).

Euclid is an ESA space mission with contributions from its Member States, NASA, and the Euclid Consortium. Publications using Euclid Q1 data must acknowledge the European Space Agency (ESA), the Euclid Consortium, the NASA/IPAC Infrared Science Archive (IRSA), and LineA/LSDB for HATS hosting. Authors should follow the official Euclid Consortium publication and citation policies.

## Provenance of this page

| Source ID            | Kind             | Service / Endpoint                                                                          | Query or URL                                                                                           | Retrieved (UTC)        |
|----------------------|------------------|---------------------------------------------------------------------------------------------|--------------------------------------------------------------------------------------------------------|------------------------|
| `catalog_euclid_spe` | `hats_catalog`   | `s3://nasa-irsa-euclid-q1/contributed/q1/merged_objects/hats/euclid_q1_merged_objects-hats` | `s3://nasa-irsa-euclid-q1/contributed/q1/merged_objects/hats/euclid_q1_merged_objects-hats/properties` | `2026-10-22T05:09:58Z` |
| `hats_euclid_mer`    | `hats_catalog`   | `https://data.lsdb.io/hats/euclid_q1/euclid_q1_merFinalCatalog`                             | `https://data.lsdb.io/hats/euclid_q1/euclid_q1_merFinalCatalog/properties`                             | `2026-10-22T05:09:15Z` |
| `spectra_euclid_q1`  | `spectrum_model` | `https://irsa.ipac.caltech.edu/SIA?COLLECTION=euclid_DpdSirCombinedSpectra`                 | `https://irsa.ipac.caltech.edu/SIA?COLLECTION=euclid_DpdSirCombinedSpectra`                            | `2026-10-18T16:54:39Z` |
<!-- /generated -->
