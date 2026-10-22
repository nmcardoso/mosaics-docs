# GALEX GR6+7 (`galex`)

<!-- generated:survey-page -->

The Galaxy Evolution Explorer (GALEX) Data Releases 6 and 7 (GR6+7) provide space-based ultraviolet imaging across the extragalactic sky. Launched by NASA, GALEX operated a 50-centimetre Ritchey-Chrétien telescope equipped with simultaneous Far-Ultraviolet (FUV, $\lambda_\text{eff} \approx 1538\text{ \AA}$, $1344\text{--}1786\text{ \AA}$) and Near-Ultraviolet (NUV, $\lambda_\text{eff} \approx 2316\text{ \AA}$, $1771\text{--}2831\text{ \AA}$) microchannel plate detectors. Its flagship All-Sky Imaging Survey (AIS) covered roughly 26,000 deg² down to typical $5\sigma$ AB depths of 20.8 mag in NUV and 19.9 mag in FUV, complemented by deeper Medium (MIS) and Deep (DIS) Imaging Surveys reaching several magnitudes fainter over narrower footprints.

In `mosaics`, GALEX provides ultraviolet photometric crossmatching (`mosaics-targets` and `mosaics-query` via the LineA HATS partition of the 292-million-row merged catalogue) and image cutouts in NUV and FUV (`mosaics-cutout full` via the Legacy Survey Viewer `lsv` adapter, with an alternative MAST SIA endpoint).

| Property                 | Value                                                                                                                                                                                                                                                                                         |
|--------------------------|-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| What MOSAICS extracts    | Catalogue: `galex` (photometric, `lsdb`); Cutouts: `lsv` (nuv,fuv), `sia` (nuv)                                                                                                                                                                                                               |
| Archives                 | HATS (`linea.data.lsdb.io`), Legacy Survey Viewer (`legacysurvey.org`), SIA (`mast.stsci.edu`)                                                                                                                                                                                                |
| Bands / wavelength range | `fuv` (`GALEX/GALEX.FUV`), `nuv` (`GALEX/GALEX.NUV`)                                                                                                                                                                                                                                          |
| Used by                  | `mosaics-targets` (crossmatch), `mosaics-cutout full`, `mosaics-crossmatch`                                                                                                                                                                                                                   |
| Crossmatch radius        | `3` arcsec. PSF FWHM 4.2 arcsec (FUV) and 4.9 arcsec (NUV) on one GALEX page, 4.5 and 5.4 arcsec on another; absolute positional uncertainty 0.5 arcsec rms (Morrissey et al. 2007, arXiv:0706.0755); 99.7 % of matches to SDSS lie within 3.4-3.6 arcsec (GALEX documentation, Chapter 106). |

## Configuration

### The shipped file

Source: `defs/surveys/galex.yaml`

```yaml
# GALEX GR6+7 -- ultraviolet photometry and cutouts.

survey_slug: galex
name: GALEX GR6+7

catalog:
  - name: galex
    type: photometric
    adapter: lsdb
    adapter_params:
      # The Linea mirror of GALEX GR6+7: the collection
      # https://linea.data.lsdb.io/hats/galex holds this catalogue and its
      # 10-arcsec margin. It replaced the CDS mirror of GUVcat (VizieR II/335),
      # which shared vizcat.cds.unistra.fr -- and so one --steps-per-source cap
      # -- with sdss16, hst2 and vhs5, and which sends each partition whole.
      # Linea answers range requests.
      #
      # Not the same product. GUVcat is one row per unique source; this is the
      # merged catalogue, 292 million rows, one per detection per visit and
      # tile, with `primary_flag` marking an object's primary detection and
      # -999 / +-99 written for a missing value. The primary_galex reshaper
      # keeps the primary rows and nulls the sentinels before the match; see
      # mosaics/crossmatch/reshapers/galex.py.
      path: https://linea.data.lsdb.io/hats/galex/galex
      margin_cache: https://linea.data.lsdb.io/hats/galex/galex_10arcs
      reshape: primary_galex

# Crossmatch: the radius a table of positions is matched within, and what it rests on.
crossmatch:
  radius: 3 arcsec
  radius_note: >-
    PSF FWHM 4.2 arcsec (FUV) and 4.9 arcsec (NUV) on one GALEX page, 4.5 and 5.4 arcsec on
    another; absolute positional uncertainty 0.5 arcsec rms (Morrissey et al. 2007,
    arXiv:0706.0755); 99.7 % of matches to SDSS lie within 3.4-3.6 arcsec (GALEX
    documentation, Chapter 106).

catalog_filters:
  galex: galex_mag_nuv <= 20.8 OR (galex_mag_nuv IS NULL AND galex_mag_fuv <= 19.9)
  galex_note: >-
    5 sigma, AB, the All-Sky Imaging Survey depth (Bianchi et al. 2017):
    NUV 20.8, FUV 19.9. NUV
    rather than FUV because it is the deeper of the two and the one almost
    every catalogue entry has. The Medium and Deep Imaging Surveys go three
    to five magnitudes fainter over a few per cent of the sky; a survey-wide
    number cannot express that, so this is the AIS figure.

cutout:
  - adapter: lsv          # VERIFIED LIVE 2026-08-25
    adapter_params:
      layer: galex
      bands: n,f            # NUV, FUV; the columns spell them out
    bands: [nuv, fuv]
  # One plane, but still a `stack` entry rather than a `nuv` key: the
  # service is sent the Viewer's abbreviation `n` while the column is
  # `cutout_galex_nuv`, and only a plane order can carry a spelling that
  # differs from the column's. A key has to be its own column token.
  - adapter: sia
    adapter_params:
      service_url: https://mast.stsci.edu/portal_vo/Mashup/VoQuery.asmx/SiaV1?MISSION=GALEX&
      bands: n              # NUV only, so it fills cutout_galex_nuv alone
    bands: [nuv]
    enabled: false        # alternative path; enable to compare

photometric_system:
  filters:
    fuv: GALEX/GALEX.FUV
    nuv: GALEX/GALEX.NUV
  measurements:
    # The calibrated flux for the default family, the AUTO magnitude for the
    # other. The flux is preferred where it exists because it is linear and
    # keeps its sign through a non-detection.
    - system: linear
      apertures: [default]
      scale: 1.0              # fuv_flux and nuv_flux are microjanskys already
    - system: ab_mag
      apertures: [auto]
  pixel_unit: unknown
  pixel_unit_note: >-
    Two separate obstacles, and the second is the harder one.

    First, the Viewer states nothing: a fits-cutout of the galex layer
    carries WCS, SURVEY, BANDS and IMAGETYP and no BUNIT or zero point
    (checked 2026-09-04), and whether it passes GALEX's calibrated count
    rates through unchanged has not been verified against a standard.

    Second, GALEX's zero points are *per band* -- ZP_FUV 18.82 and ZP_NUV
    20.08 (Morrissey et al. 2007), held fixed across every release because
    calibration improvements were absorbed into the flat field -- while
    `zeropoint:` here is one number for the survey. So even a confirmed
    counts_per_s reading could not be written down without a per-band
    zeropoint, which the schema does not have and which no other survey
    needs. counts_per_s remains the one legal pixel unit no survey uses.

# GALEX GR6+7 merged catalogue (MCAT), as the Linea HATS mirror publishes it:
# FUV/NUV astrometry, photometry and morphology, one row per detection.
# Catalog: https://linea.data.lsdb.io/hats/galex/galex, ra/dec = ra/dec.
# Selected via LSDB, not turned into SQL. Upper-case names are the SExtractor
# outputs, lower-case ones the pipeline's own; the aliases are the ones this
# survey had against GUVcat, so nothing downstream changes. GUVcat's `Name`
# and `OName` have no counterpart here and are gone.
column_mapping:
  # objid: 'id'                            # GALEX identifier for the detection
  ra: 'ra'                                 # right ascension, J2000 (ICRS), deg
  dec: 'dec'                               # declination, J2000 (ICRS), deg
  primary_flag: 'flag_primary'             # 1 on the primary detection; the reshaper keeps only those
  E_bv: 'ebv'                              # Galactic reddening E(B-V) from Schlegel+ 1998 maps, mag
  band: 'band'                             # bands with a detection: 1=NUV, 2=FUV, 3=both
  isThereSpectrum: 'has_spectrum'          # 1 if a low-resolution GALEX spectrum exists
  fuv_mag: 'mag_fuv'                       # FUV calibrated magnitude, AB system
  fuv_magerr: 'mag_fuv_err'                # FUV magnitude error
  nuv_mag: 'mag_nuv'                       # NUV calibrated magnitude, AB system
  nuv_magerr: 'mag_nuv_err'                # NUV magnitude error
  FUV_MAG_AUTO: 'mag_auto_fuv'             # FUV Kron-like elliptical aperture (AUTO) magnitude
  FUV_MAGERR_AUTO: 'mag_auto_fuv_err'      # FUV AUTO magnitude error
  NUV_MAG_AUTO: 'mag_auto_nuv'             # NUV Kron-like elliptical aperture (AUTO) magnitude
  NUV_MAGERR_AUTO: 'mag_auto_nuv_err'      # NUV AUTO magnitude error
  fuv_flux: 'flux_fuv'                     # FUV calibrated flux, uJy
  fuv_fluxerr: 'flux_fuv_err'              # FUV flux error, uJy
  nuv_flux: 'flux_nuv'                     # NUV calibrated flux, uJy
  nuv_fluxerr: 'flux_nuv_err'              # NUV flux error, uJy
  NUV_CLASS_STAR: 'class_star_nuv'         # star/galaxy classifier from the NUV image
  FUV_CLASS_STAR: 'class_star_fuv'         # star/galaxy classifier from the FUV image
  NUV_ELLIPTICITY: 'ellipticity_nuv'       # ellipticity (1-b/a) in NUV
  FUV_ELLIPTICITY: 'ellipticity_fuv'       # ellipticity (1-b/a) in FUV
  FUV_FWHM_WORLD: 'fwhm_fuv'               # source FWHM in FUV, deg
  NUV_FWHM_WORLD: 'fwhm_nuv'               # source FWHM in NUV, deg
  fuv_artifact: 'flag_artifact_fuv'        # FUV artifact flag, logical OR of flags near the source
  nuv_artifact: 'flag_artifact_nuv'        # NUV artifact flag, logical OR of flags near the source
  FUV_FLAGS: 'flag_extraction_fuv'         # FUV SExtractor extraction flags
  NUV_FLAGS: 'flag_extraction_nuv'         # NUV SExtractor extraction flags
  # nuv_poserr: 'poserr_nuv'                 # position error of the source in the NUV image, arcsec
  # fuv_poserr: 'poserr_fuv'                 # position error of the source in the FUV image, arcsec
```

### Schema of this file

| Key present          | Type                                       | Required   | Default   | Meaning                                                                                                                                                                                                                      |
|----------------------|--------------------------------------------|------------|-----------|------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `survey_slug`        | str                                        | yes        | —         | Canonical lowercase identifier for the survey and default column prefix.                                                                                                                                                     |
| `name`               | str                                        | yes        | —         | Human-readable survey and data-release name used in logs, plans, and reports.                                                                                                                                                |
| `catalog`            | list of catalog entries                    | no         | []        | Remote or local catalogues queried for target extraction or crossmatching.                                                                                                                                                   |
| `crossmatch`         | mapping                                    | no         | None      | How a table of positions is matched against this survey. Required for a survey with an `lsdb` catalog; `crossmatch` null removes it, and a survey without it takes no part in `mosaics-crossmatch`.                          |
| `catalog_filters`    | mapping (prefix -> SQL expression or note) | no         | {}        | Post-hoc DuckDB SQL quality/depth/redshift cuts applied during production (nulls failing survey columns without dropping the row).                                                                                           |
| `cutout`             | list of cutout entries                     | no         | []        | Image cutout backends (`hips`, `lsv`, `nersc`, `sia`, `vsa`) serving this survey’s passbands.                                                                                                                                |
| `photometric_system` | mapping                                    | no         | None      | Passband SVO filter identifiers, flux/magnitude homogenisation rules, and image pixel calibration.                                                                                                                           |
| `column_mapping`     | mapping (remote_column -> alias)           | no         | {}        | Projection and column renaming applied when crossmatching against this survey’s photometric catalogue. The loader adds the catalog’s prefix to every alias, or the hosted survey’s for the columns under its `provides` key. |

### Adapter parameters

**Catalogue `galex` (`adapter: lsdb`):**

| Parameter           | Allowed Type   | Default               | Description                                                                                         |
|---------------------|----------------|-----------------------|-----------------------------------------------------------------------------------------------------|
| `path`              | str            | —                     | HTTP or S3 URI of the HATS catalogue root (may contain `{band}` or `{BAND}` when `per_band: true`). |
| `margin_cache`      | str            | None                  | URI of the HATS pre-computed margin cache table for boundary-safe spatial crossmatching.            |
| `storage_options`   | mapping        | {}                    | Filesystem options forwarded to `upath`/`fsspec` (e.g. `{'anon': True}` for public S3 buckets).     |
| `reshape`           | str            | None                  | Optional post-read reshaping strategy for nested or multi-row HATS tables.                          |
| `per_band`          | bool           | false                 | When `true`, opens one HATS catalogue per band by substituting `{band}`/`{BAND}` in `path`.         |
| `remote_ra_column`  | str            | HATS metadata default | Override for the remote right ascension column name.                                                |
| `remote_dec_column` | str            | HATS metadata default | Override for the remote declination column name.                                                    |
- **Cutout `None` (`adapter: lsv`, bands `['nuv', 'fuv']`):** configured `adapter_params`: `layer='galex'`, `bands='n,f'`.
- **Cutout `None` (`adapter: sia`, bands `['nuv']`):** configured `adapter_params`: `service_url='https://mast.stsci.edu/portal_vo/Mashup/VoQuery.asmx/SiaV1?MISSION=GALEX&'`, `bands='n'`.

### Common changes

- **Adjusting depth threshold in catalog filters**: The default filter enforces the AIS $5\sigma$ point-source completeness limit (`galex_mag_nuv <= 20.8 OR (galex_mag_nuv IS NULL AND galex_mag_fuv <= 19.9)`). For targets located in deeper Medium (MIS) or Deep (DIS) survey fields, relax or clear the filter by setting `overrides[surveys/galex].catalog_filters.galex: null`.
- **Selecting cutout providers**: Cutouts default to the Legacy Survey Viewer (`adapter: lsv`, `layer: galex`, bands `[nuv, fuv]`). To query MAST directly via IVOA Simple Image Access, disable `lsv` (`cutout.0.enabled: false`) and enable the SIA adapter (`cutout.1.enabled: true`).
- **Astrometric positional errors**: In `column_mapping`, uncomment `nuv_poserr` or `fuv_poserr` to extract positional uncertainties in arcseconds, or `objid` to extract the unique detection ID.
- **Disable catalogue crossmatching**: Disable GALEX catalogue matching in an entrypoint configuration by setting `overrides[surveys/galex].catalog.0.enabled: false`.

### Commands

```console
$ uv run -m mosaics.cli.cutout full -t test/objects_sample_1k.csv -o cutouts.lance --config defs/cutout.yaml --survey galex --plan
$ uv run -m mosaics.crossmatch.cli run --survey surveys/galex -i test/objects_sample_1k.csv -o galex_matched.parquet
$ uv run -m mosaics.cli.targets --config defs/targets.yaml -o ./targets_run --dec-range 0 1 --plan plan.yaml
```

## Data served by the archive

- **Merged catalogue (MCAT) vs GUVcat**: MOSAICS queries the LineA HATS mirror of the GALEX merged catalogue (`galex/galex`, 292 million rows), which contains one record per detection per tile/visit. The built-in `primary_galex` reshaper filters rows by `primary_flag == 1` to isolate unique primary detections and cleans missing-value sentinels (`-999` and $\pm 99$) to nulls before performing spatial crossmatches.
- **Photometric flux scale**: Calibrated fluxes (`fuv_flux`, `nuv_flux`) are published in microjanskys ($\mu\text{Jy}$) on the AB scale (`system: linear`, `scale: 1.0`). Magnitudes (`fuv_mag`, `nuv_mag`, `FUV_MAG_AUTO`, `NUV_MAG_AUTO`) are on the AB system. Physical count-rate zero points are band-dependent: $ZP_\text{FUV} = 18.82$ and $ZP_\text{NUV} = 20.08$ (Morrissey et al. 2007).
- **Cutout pixel units**: Image cutouts from the Legacy Survey Viewer do not provide a standard FITS `BUNIT` or photometric zero-point header. Because zero points differ between the two bands, `pixel_unit` is recorded as `unknown`.
- **Artifact and extraction flags**: The catalogue supplies artifact flags (`fuv_artifact`, `nuv_artifact`) recording proximity to bright stars, dichroic reflections, and field edges, alongside standard SExtractor extraction flags (`FUV_FLAGS`, `NUV_FLAGS`).

### Catalogue: `II/335/galex_ais`

the revised GALEX catalog of UV sources (GUVcat_AIS) from GR6+7 (82,992,086 sources)

**Service URL:** `https://linea.data.lsdb.io/hats/galex/galex`

*Showing 100 of 368 total archive columns (all MOSAICS-mapped columns plus primary table fields; full 368-column schema is stored in `docs/_data/sources/galex/hats_galex.json`).*

| Column                    | Type     | Unit   | UCD   | Description   |
|---------------------------|----------|--------|-------|---------------|
| `band`                    | `int32`  | `—`    | `—`   | —             |
| `ra`                      | `double` | `—`    | `—`   | —             |
| `dec`                     | `double` | `—`    | `—`   | —             |
| `E_bv`                    | `double` | `—`    | `—`   | —             |
| `primary_flag`            | `int32`  | `—`    | `—`   | —             |
| `nuv_flux`                | `double` | `—`    | `—`   | —             |
| `nuv_fluxerr`             | `double` | `—`    | `—`   | —             |
| `nuv_mag`                 | `double` | `—`    | `—`   | —             |
| `nuv_magerr`              | `double` | `—`    | `—`   | —             |
| `fuv_flux`                | `double` | `—`    | `—`   | —             |
| `fuv_fluxerr`             | `double` | `—`    | `—`   | —             |
| `fuv_mag`                 | `double` | `—`    | `—`   | —             |
| `fuv_magerr`              | `double` | `—`    | `—`   | —             |
| `nuv_artifact`            | `int32`  | `—`    | `—`   | —             |
| `fuv_artifact`            | `int32`  | `—`    | `—`   | —             |
| `NUV_MAG_AUTO`            | `double` | `—`    | `—`   | —             |
| `NUV_MAGERR_AUTO`         | `double` | `—`    | `—`   | —             |
| `NUV_ELLIPTICITY`         | `double` | `—`    | `—`   | —             |
| `NUV_FWHM_WORLD`          | `double` | `—`    | `—`   | —             |
| `NUV_FLAGS`               | `int16`  | `—`    | `—`   | —             |
| `NUV_CLASS_STAR`          | `double` | `—`    | `—`   | —             |
| `FUV_MAG_AUTO`            | `double` | `—`    | `—`   | —             |
| `FUV_MAGERR_AUTO`         | `double` | `—`    | `—`   | —             |
| `FUV_ELLIPTICITY`         | `double` | `—`    | `—`   | —             |
| `FUV_FWHM_WORLD`          | `double` | `—`    | `—`   | —             |
| `FUV_FLAGS`               | `int16`  | `—`    | `—`   | —             |
| `FUV_CLASS_STAR`          | `double` | `—`    | `—`   | —             |
| `isThereSpectrum`         | `bool`   | `—`    | `—`   | —             |
| `objid`                   | `int64`  | `—`    | `—`   | —             |
| `photoExtractID`          | `int64`  | `—`    | `—`   | —             |
| `vsn`                     | `int32`  | `—`    | `—`   | —             |
| `tilenum`                 | `int32`  | `—`    | `—`   | —             |
| `type`                    | `int32`  | `—`    | `—`   | —             |
| `ow`                      | `int32`  | `—`    | `—`   | —             |
| `prod`                    | `int32`  | `—`    | `—`   | —             |
| `img`                     | `int32`  | `—`    | `—`   | —             |
| `try`                     | `int32`  | `—`    | `—`   | —             |
| `id`                      | `int32`  | `—`    | `—`   | —             |
| `subvisit`                | `int16`  | `—`    | `—`   | —             |
| `leg`                     | `int16`  | `—`    | `—`   | —             |
| `alpha_j2000_merged`      | `double` | `—`    | `—`   | —             |
| `delta_j2000_merged`      | `double` | `—`    | `—`   | —             |
| `prob`                    | `double` | `—`    | `—`   | —             |
| `sep`                     | `double` | `—`    | `—`   | —             |
| `sep_img`                 | `double` | `—`    | `—`   | —             |
| `glon`                    | `double` | `—`    | `—`   | —             |
| `glat`                    | `double` | `—`    | `—`   | —             |
| `fov_radius`              | `double` | `—`    | `—`   | —             |
| `nuv_det_x`               | `double` | `—`    | `—`   | —             |
| `nuv_det_y`               | `double` | `—`    | `—`   | —             |
| `fuv_det_x`               | `double` | `—`    | `—`   | —             |
| `fuv_det_y`               | `double` | `—`    | `—`   | —             |
| `chkobj_type`             | `int32`  | `—`    | `—`   | —             |
| `chkobj_id`               | `int32`  | `—`    | `—`   | —             |
| `chkobj_r`                | `double` | `—`    | `—`   | —             |
| `chkobj_pa`               | `double` | `—`    | `—`   | —             |
| `misc_flags`              | `int32`  | `—`    | `—`   | —             |
| `nuv_s2n`                 | `double` | `—`    | `—`   | —             |
| `fuv_s2n`                 | `double` | `—`    | `—`   | —             |
| `nuv_fcat_flux`           | `double` | `—`    | `—`   | —             |
| `nuv_fcat_fluxerr`        | `double` | `—`    | `—`   | —             |
| `fuv_ncat_flux`           | `double` | `—`    | `—`   | —             |
| `fuv_ncat_fluxerr`        | `double` | `—`    | `—`   | —             |
| `fuv_ra_adj`              | `double` | `—`    | `—`   | —             |
| `fuv_dec_adj`             | `double` | `—`    | `—`   | —             |
| `fuv_ncat_flags`          | `int16`  | `—`    | `—`   | —             |
| `fuv_ncat_mag`            | `double` | `—`    | `—`   | —             |
| `fuv_ncat_magerr`         | `double` | `—`    | `—`   | —             |
| `fuv_ncat_s2n`            | `double` | `—`    | `—`   | —             |
| `nuv_fcat_mag`            | `double` | `—`    | `—`   | —             |
| `nuv_fcat_magerr`         | `double` | `—`    | `—`   | —             |
| `nuv_fcat_s2n`            | `double` | `—`    | `—`   | —             |
| `fuv_ncat_fwhm_image`     | `double` | `—`    | `—`   | —             |
| `fuv_ncat_fwhm_world`     | `double` | `—`    | `—`   | —             |
| `fuv_ncat_flux_radius_1`  | `double` | `—`    | `—`   | —             |
| `fuv_ncat_flux_radius_2`  | `double` | `—`    | `—`   | —             |
| `fuv_ncat_flux_radius_3`  | `double` | `—`    | `—`   | —             |
| `fuv_ncat_flux_radius_4`  | `double` | `—`    | `—`   | —             |
| `fuv_ncat_flux_radius_5`  | `double` | `—`    | `—`   | —             |
| `fuv_ncat_flux_aper_1`    | `double` | `—`    | `—`   | —             |
| `fuv_ncat_flux_aper_2`    | `double` | `—`    | `—`   | —             |
| `fuv_ncat_flux_aper_3`    | `double` | `—`    | `—`   | —             |
| `fuv_ncat_flux_aper_4`    | `double` | `—`    | `—`   | —             |
| `fuv_ncat_flux_aper_5`    | `double` | `—`    | `—`   | —             |
| `fuv_ncat_flux_aper_6`    | `double` | `—`    | `—`   | —             |
| `fuv_ncat_flux_aper_7`    | `double` | `—`    | `—`   | —             |
| `fuv_ncat_fluxerr_aper_1` | `double` | `—`    | `—`   | —             |
| `fuv_ncat_fluxerr_aper_2` | `double` | `—`    | `—`   | —             |
| `fuv_ncat_fluxerr_aper_3` | `double` | `—`    | `—`   | —             |
| `fuv_ncat_fluxerr_aper_4` | `double` | `—`    | `—`   | —             |
| `fuv_ncat_fluxerr_aper_5` | `double` | `—`    | `—`   | —             |
| `fuv_ncat_fluxerr_aper_6` | `double` | `—`    | `—`   | —             |
| `fuv_ncat_fluxerr_aper_7` | `double` | `—`    | `—`   | —             |
| `fuv_ncat_mag_aper_1`     | `double` | `—`    | `—`   | —             |
| `fuv_ncat_mag_aper_2`     | `double` | `—`    | `—`   | —             |
| `fuv_ncat_mag_aper_3`     | `double` | `—`    | `—`   | —             |
| `fuv_ncat_mag_aper_4`     | `double` | `—`    | `—`   | —             |
| `fuv_ncat_mag_aper_5`     | `double` | `—`    | `—`   | —             |
| `fuv_ncat_mag_aper_6`     | `double` | `—`    | `—`   | —             |
| `fuv_ncat_mag_aper_7`     | `double` | `—`    | `—`   | —             |

### Images

- **Pixel unit:** `unknown` | **AB zero point:** `—` | **Calibration note:** Two separate obstacles, and the second is the harder one.
  First, the Viewer states nothing: a fits-cutout of the galex layer carries WCS, SURVEY, BANDS and IMAGETYP and no BUNIT or zero point (checked 2026-09-04), and whether it passes GALEX’s calibrated count rates through unchanged has not been verified against a standard.
  Second, GALEX’s zero points are *per band* – ZP_FUV 18.82 and ZP_NUV 20.08 (Morrissey et al. 2007), held fixed across every release because calibration improvements were absorbed into the flat field – while `zeropoint:` here is one number for the survey. So even a confirmed counts_per_s reading could not be written down without a per-band zeropoint, which the schema does not have and which no other survey needs. counts_per_s remains the one legal pixel unit no survey uses.
- **Cutout service `None`:** adapter `lsv`, bands `['nuv', 'fuv']`, params `{'layer': 'galex', 'bands': 'n,f'}`.
- **Cutout service `None`:** adapter `sia`, bands `['nuv']`, params `{'service_url': 'https://mast.stsci.edu/portal_vo/Mashup/VoQuery.asmx/SiaV1?MISSION=GALEX&', 'bands': 'n'}`.

### Spectra

No 1D spectroscopic backends are configured for this survey.

## References and acknowledgement

- Martin, D. C., et al. (2005), *ApJ*, 619, L1 (`doi:10.1086/430219`) — The Galaxy Evolution Explorer: A Space Ultraviolet Survey.
- Morrissey, P., et al. (2007), *ApJS*, 173, 682 (`doi:10.1086/520512`) — The Calibration and Data Products of GALEX.
- Bianchi, L., Shiao, B., & Thilker, D. (2017), *ApJS*, 230, 24 (`doi:10.3847/1538-4365/aa7053`) — A Revised Catalog of GALEX Ultraviolet Sources for the Entire Sky (GUVcat_AIS).
- LineA HATS GALEX Collection: [https://linea.data.lsdb.io/hats/galex/galex](https://linea.data.lsdb.io/hats/galex/galex).
- MAST GALEX Archive: [https://archive.stsci.edu/galex/](https://archive.stsci.edu/galex/).
- MAST SIA Endpoint: `https://mast.stsci.edu/portal_vo/Mashup/VoQuery.asmx/SiaV1?MISSION=GALEX&`.

Based on observations made with the NASA Galaxy Evolution Explorer. GALEX was operated for NASA by the California Institute of Technology under NASA contract NAS5-98034. Authors should acknowledge NASA, the Mikulski Archive for Space Telescopes (MAST) at the Space Telescope Science Institute (STScI), and the LineA Science Server for HATS partitioning and hosting. Cite Martin et al. (2005) and Morrissey et al. (2007).

## Provenance of this page

| Source ID      | Kind           | Service / Endpoint                                                          | Query or URL                                                                | Retrieved (UTC)        |
|----------------|----------------|-----------------------------------------------------------------------------|-----------------------------------------------------------------------------|------------------------|
| `cutout_galex` | `sia`          | `https://mast.stsci.edu/portal_vo/Mashup/VoQuery.asmx/SiaV1?MISSION=GALEX&` | `https://mast.stsci.edu/portal_vo/Mashup/VoQuery.asmx/SiaV1?MISSION=GALEX&` | `2026-10-18T16:31:22Z` |
| `hats_galex`   | `hats_catalog` | `https://linea.data.lsdb.io/hats/galex/galex`                               | `https://linea.data.lsdb.io/hats/galex/galex/properties`                    | `2026-10-18T16:57:42Z` |
<!-- /generated -->
