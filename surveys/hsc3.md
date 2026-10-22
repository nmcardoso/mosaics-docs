# Subaru HSC-SSP PDR3 (`hsc3`)

<!-- generated:survey-page -->

The Hyper Suprime-Cam Subaru Strategic Program Public Data Release 3 (HSC-SSP PDR3) delivers deep optical imaging and multi-band photometry obtained with the 870-megapixel Hyper Suprime-Cam on the 8.2-metre Subaru Telescope at Maunakea. PDR3 covers optical broadbands $g, r, i, z, y$ across Wide, Deep, and UltraDeep layers, reaching $5\sigma$ point-source depths of $i \approx 26.2$, $r \approx 26.5$, and $z \approx 25.2$ AB mag in the Wide layer.

In MOSAICS, `hsc3` provides deep photometric crossmatch via HATS datasets (`hsc_pdr3-forced-{BAND}`) hosted at `https://data.lsdb.io/hats/hsc_pdr3/`, querying forced PSF, Gaussian, circular aperture, Kron, and CModel fluxes. It also provides optical cutouts in $g, r, i$ through the Legacy Survey Viewer (`hsc-dr3` layer). (HiPS cutout layers for HSC DR2 Wide are defined in the configuration but disabled because the DR2 zero point is not calibrated for those tiles).

| Property                 | Value                                                                                                                                                                                                                                                                                    |
|--------------------------|------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| What MOSAICS extracts    | Catalogue: `hsc3` (photometric, `lsdb`); Cutouts: `lsv` (g,r,i), `hips` (g), `hips` (r), `hips` (i)                                                                                                                                                                                      |
| Archives                 | HATS (`data.lsdb.io`), Legacy Survey Viewer (`legacysurvey.org`), CDS HiPS / hips2fits (`alasky.cds.unistra.fr`)                                                                                                                                                                         |
| Bands / wavelength range | `g` (`Subaru/HSC.g`), `r` (`Subaru/HSC.r`), `i` (`Subaru/HSC.i`), `z` (`Subaru/HSC.z`), `y` (`Subaru/HSC.Y`)                                                                                                                                                                             |
| Used by                  | `mosaics-targets` (crossmatch), `mosaics-cutout full`, `mosaics-crossmatch`                                                                                                                                                                                                              |
| Crossmatch radius        | `1` arcsec. Median seeing of the Wide layer 0.61-0.79 arcsec (i band 0.61 arcsec); astrometric rms about 13 mas against Gaia DR1 (Aihara et al. 2022, arXiv:2108.13045, Table 1 and Sect. 5.1). Value set by the author (HG2, 2026-10-19); the instrument-based proposal was 0.5 arcsec. |

## Configuration

### The shipped file

Source: `defs/surveys/hsc3.yaml`

```yaml
# Subaru HSC-SSP PDR3 -- forced photometry and cutouts.
#
# Not to be confused with `hst2`, the Hubble Source Catalog, whose module is
# also called hsc. This is Subaru.

survey_slug: hsc3
name: Subaru HSC-SSP PDR3

catalog:
  - name: hsc3
    type: photometric
    adapter: lsdb
    adapter_params:
      # Five catalogues with an identical schema and no band in their column
      # names, which is how the survey is published. Each carries its own
      # margin cache. The production phase joins them back on the object id.
      path: https://data.lsdb.io/hats/hsc_pdr3/hsc_pdr3-forced-{BAND}
      margin_cache: https://data.lsdb.io/hats/hsc_pdr3/hsc_pdr3-forced-{BAND}_10arcs
      per_band: [g, r, i, z, y]
    unflagged: >-
      the HATS build carries no pixel flags at all -- none of its 241 columns
      is a flag, and no boolean column survived the conversion. LSST forced
      photometry is normally filtered on those to reject saturated,
      interpolated, edge and cosmic-ray-affected measurements, and that is
      not possible from this catalogue. Extendedness, deblend_nChild and the
      flux apCorr terms are the only quality signal it offers

# Crossmatch: the radius a table of positions is matched within, and what it rests on.
crossmatch:
  radius: 1 arcsec
  radius_note: >-
    Median seeing of the Wide layer 0.61-0.79 arcsec (i band 0.61 arcsec); astrometric rms
    about 13 mas against Gaia DR1 (Aihara et al. 2022, arXiv:2108.13045, Table 1 and Sect.
    5.1). Value set by the author (HG2, 2026-10-19); the instrument-based proposal was 0.5
    arcsec.

catalog_filters:
  hsc3: >-
    hsc3_flux_psf_i >= 2.089
    OR (hsc3_flux_psf_i IS NULL AND hsc3_flux_psf_r >= 1.585)
    OR (hsc3_flux_psf_i IS NULL AND hsc3_flux_psf_r IS NULL AND hsc3_flux_psf_z >= 5.248)
  hsc3_note: >-
    5 sigma point source, AB, PDR3 Wide (Aihara et al. 2022, Table 2): i
    26.2, r 26.5, z 25.2. i is the deepest of the three and the one almost
    every row has, which is why an object invisible everywhere else can
    still be worth a cutout here; r and z are the fallback where i itself is
    missing. There is no ivar here to test for "missing" by -- unlike ls11,
    this catalogue has no inverse-variance column -- so the fallback tests
    the flux itself for null.

cutout:
  - adapter: lsv          # VERIFIED LIVE 2026-08-25
    adapter_params:
      layer: hsc-dr3
      bands: gri
    bands: [g, r, i]
  # HSC-SSP DR2 wide, also published per band with FITS tiles (MOCServer,
  # 2026-09-05); `CDS/P/HSC/DR2/deep/*` and `CDS/P/HSCLA/2016/*` are the
  # other two builds. DR2, not the PDR3 the photometry and the Viewer
  # layer above describe, so the 27.0 zero point stated for this survey is
  # not established for these tiles. Disabled until it is.
  - adapter: hips
    adapter_params:
      hips_id: CDS/P/HSC/DR2/wide/g
    bands: [g]
    enabled: false
  - adapter: hips
    adapter_params:
      hips_id: CDS/P/HSC/DR2/wide/r
    bands: [r]
    enabled: false
  - adapter: hips
    adapter_params:
      hips_id: CDS/P/HSC/DR2/wide/i
    bands: [i]
    enabled: false

photometric_system:
  filters:
    g: Subaru/HSC.g
    r: Subaru/HSC.r
    i: Subaru/HSC.i
    z: Subaru/HSC.z
    y: Subaru/HSC.Y
  measurements:
    # Instrumental ADU on a fixed 27.0 mag coadd zero point.
    - system: linear
      apertures: [psf, gaussian, aper12, kron, cmodel]
      scale: 0.057548         # 3631e6 * 10 ** (-0.4 * 27.0) uJy per ADU
  pixel_unit: ab_zeropoint
  zeropoint: 27.0
  pixel_unit_note: >-
    HSC-SSP coadds carry a fixed 27.0 mag zero point, the same one the
    photometry's ADU-to-uJy scale is derived from. Good to a few per cent:
    aperture corrections are not applied at the image level.

# HSC PDR3 forced-photometry catalogue (Aihara+ 2022): LSST-pipeline
# forced PSF/CModel/Kron/aperture photometry and extendedness.
# Catalog: https://data.lsdb.io/hats/hsc_pdr3/hsc_pdr3-forced-G
# (+ the R/I/Z/Y siblings -- same schema, one file covers all five).
# ra/dec = coord_ra/coord_dec. Selected via LSDB, not turned into SQL.
#
# UNITS -- every flux here is instrumental (ADU), not nJy: all 242 columns of
# the HATS build were checked and none is calibrated. HSC-SSP coadds carry a
# fixed 27.0 mag zero point, so f_nu[uJy] = 0.057548 * instFlux; the HSC team
# notes this is good to a few per cent only, because aperture corrections are
# not applied at the image level. The *_apcorr columns are kept for a future
# refinement that wants to apply them.
# The band is not in any column name -- each of the five per-band
# catalogs carries this identical schema -- so aliases stay band-agnostic
# (hsc3_flux_psf, not hsc3_g_psf_flux); the phase module appends
# the band itself.
# WARNING -- this HATS conversion carries NO pixel flags. All 241 columns
# were checked: none contains 'flag', there is no base_PixelFlags_flag_*
# family, and no boolean column of any kind survived the conversion. LSST
# forced photometry is normally filtered on those before use, to reject
# saturated, interpolated, edge and cosmic-ray-affected measurements, and
# that filtering is simply not possible from this catalogue. The fluxes
# below are unfiltered; treat them accordingly. Extendedness,
# deblend_nChild and the flux apCorr terms are the only quality signal
# available here.
column_mapping:
  # id: 'id'                                                       # forced-photometry source identifier
  coord_ra: 'ra'                                                   # right ascension, deg
  coord_dec: 'dec'                                                 # declination, deg
  deblend_nChild: 'deblend_nchild_{band}'                                 # number of child sources this record was deblended into
  base_ClassificationExtendedness_value: 'extendedness_{band}'            # star/galaxy extendedness classifier (0=point source, 1=extended)
  base_Variance_value: 'variance_{band}'                                  # pixel variance at the source position
  base_PsfFlux_instFlux: 'flux_psf_{band}'                                # PSF-fit instrumental flux
  base_PsfFlux_instFluxErr: 'flux_psf_{band}_err'                         # PSF-fit instrumental flux error
  # base_PsfFlux_apCorr: 'flux_psf_apcorr_{band}'                           # aperture correction applied to the PSF flux
  # base_PsfFlux_apCorrErr: 'flux_psf_apcorr_{band}_err'                    # error on the PSF flux aperture correction
  base_PsfFlux_area: 'flux_psf_area_{band}'                               # effective area of the PSF flux measurement, pix^2
  base_GaussianFlux_instFlux: 'flux_gaussian_{band}'                      # Gaussian-weighted instrumental flux
  base_GaussianFlux_instFluxErr: 'flux_gaussian_{band}_err'               # Gaussian-weighted instrumental flux error
  base_CircularApertureFlux_12_0_instFlux: 'flux_aper12_{band}'      # circular-aperture instrumental flux, 1.2 arcsec radius
  base_CircularApertureFlux_12_0_instFluxErr: 'flux_aper12_{band}_err'     # error on the 1.2 arcsec circular-aperture flux
  ext_photometryKron_KronFlux_instFlux: 'flux_kron_{band}'                # Kron-aperture instrumental flux
  ext_photometryKron_KronFlux_instFluxErr: 'flux_kron_{band}_err'         # Kron-aperture instrumental flux error
  ext_photometryKron_KronFlux_radius: 'kron_radius_{band}'                # Kron aperture radius, pix
  # ext_photometryKron_KronFlux_apCorr: 'flux_kron_apcorr_{band}'           # aperture correction applied to the Kron flux
  # ext_photometryKron_KronFlux_apCorrErr: 'flux_kron_apcorr_{band}_err'     # error on the Kron flux aperture correction
  modelfit_CModel_instFlux: 'flux_cmodel_{band}'                          # CModel (composite exponential+de Vaucouleurs) total instrumental flux
  modelfit_CModel_instFluxErr: 'flux_cmodel_{band}_err'                   # CModel total instrumental flux error
  # modelfit_CModel_exp_instFlux: 'flux_cmodel_exp_{band}'                  # CModel exponential-component instrumental flux
  # modelfit_CModel_exp_instFluxErr: 'flux_cmodel_exp_{band}_err'           # CModel exponential-component instrumental flux error
  # modelfit_CModel_dev_instFlux: 'flux_cmodel_dev_{band}'                  # CModel de Vaucouleurs-component instrumental flux
  # modelfit_CModel_dev_instFluxErr: 'flux_cmodel_dev_{band}_err'           # CModel de Vaucouleurs-component instrumental flux error
  # modelfit_CModel_fracDev: 'cmodel_frac_dev_{band}'                       # de Vaucouleurs-component flux fraction in the CModel fit
  # modelfit_CModel_objective: 'cmodel_objective_{band}'                    # CModel fit objective (chi^2-like goodness of fit)
  # modelfit_CModel_apCorr: 'flux_cmodel_apcorr_{band}'                     # aperture correction applied to the CModel total flux
  # modelfit_CModel_apCorrErr: 'flux_cmodel_apcorr_{band}_err'              # error on the CModel total flux aperture correction
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

**Catalogue `hsc3` (`adapter: lsdb`):**

| Parameter           | Allowed Type   | Default               | Description                                                                                         |
|---------------------|----------------|-----------------------|-----------------------------------------------------------------------------------------------------|
| `path`              | str            | —                     | HTTP or S3 URI of the HATS catalogue root (may contain `{band}` or `{BAND}` when `per_band: true`). |
| `margin_cache`      | str            | None                  | URI of the HATS pre-computed margin cache table for boundary-safe spatial crossmatching.            |
| `storage_options`   | mapping        | {}                    | Filesystem options forwarded to `upath`/`fsspec` (e.g. `{'anon': True}` for public S3 buckets).     |
| `reshape`           | str            | None                  | Optional post-read reshaping strategy for nested or multi-row HATS tables.                          |
| `per_band`          | bool           | false                 | When `true`, opens one HATS catalogue per band by substituting `{band}`/`{BAND}` in `path`.         |
| `remote_ra_column`  | str            | HATS metadata default | Override for the remote right ascension column name.                                                |
| `remote_dec_column` | str            | HATS metadata default | Override for the remote declination column name.                                                    |
- **Cutout `None` (`adapter: lsv`, bands `['g', 'r', 'i']`):** configured `adapter_params`: `layer='hsc-dr3'`, `bands='gri'`.
- **Cutout `None` (`adapter: hips`, bands `['g']`):** configured `adapter_params`: `hips_id='CDS/P/HSC/DR2/wide/g'`.
- **Cutout `None` (`adapter: hips`, bands `['r']`):** configured `adapter_params`: `hips_id='CDS/P/HSC/DR2/wide/r'`.
- **Cutout `None` (`adapter: hips`, bands `['i']`):** configured `adapter_params`: `hips_id='CDS/P/HSC/DR2/wide/i'`.

### Common changes

- Relax or disable the default post-fetch flux filter in your entrypoint YAML:
  ```yaml
  overrides:
    surveys/hsc3:
      catalog_filters:
        hsc3: null
  ```

  The shipped filter enforces $5\sigma$ point-source detection thresholds across $i$, $r$, or $z$ (`hsc3_flux_psf_i >= 2.089 OR (hsc3_flux_psf_i IS NULL AND hsc3_flux_psf_r >= 1.585) OR ...`). Setting it to `null` retains faint sources down to the catalogue extraction limit.
- Disable `hsc3` catalogue extraction or cutouts entirely in a custom run by setting `overrides[surveys/hsc3].catalog: []` or `overrides[surveys/hsc3].cutout: []`.
- Select specific aperture fluxes (such as Kron `hsc3_flux_kron_{band}` or CModel `hsc3_flux_cmodel_{band}`) depending on whether your science requires point-source or extended-source flux measurements.

### Commands

```console
$ uv run -m mosaics.cli.cutout full -t test/objects_sample_1k.csv -o cutouts.lance --config defs/cutout.yaml --survey hsc3 --plan
$ uv run -m mosaics.crossmatch.cli run --survey surveys/hsc3 -i test/objects_sample_1k.csv -o hsc3_matched.parquet
$ uv run -m mosaics.cli.targets --config defs/targets.yaml -o ./targets_run --dec-range 0 1 --plan plan.yaml
```

## Data served by the archive

- **Five per-band tables and column naming:** The upstream HATS collection partitions forced photometry across five separate tables (`hsc_pdr3-forced-{BAND}`) for $g, r, i, z, y$, each sharing an identical 241-column schema without band prefixes in column names. MOSAICS joins them on object ID and dynamically appends the band suffix (e.g. `hsc3_flux_psf_i`).
- **Complete absence of pixel flags:** The HATS conversion carries no pixel flags whatsoever. None of the 241 columns in the remote Parquet tables contains `flag`, and no boolean quality flags survived conversion. Standard LSST/HSC pipeline filtering on saturation, edge proximity, bad interpolation, or cosmic rays cannot be performed on flags. Measurements are therefore unflagged; source extendedness (`hsc3_extendedness_{band}`), deblend child count (`hsc3_deblend_nchild_{band}`), and aperture correction terms are the only quality indicators available.
- **Instrumental fluxes and 27.0 zero point:** All published fluxes (`base_PsfFlux_instFlux`, `modelfit_CModel_instFlux`, etc.) are instrumental ADU counts, not calibrated nanojanskys. HSC-SSP coadds use a fixed 27.0 AB magnitude zero point, converting to microjanskys via $f_\nu [\mu\text{Jy}] = 0.057548 \times \text{instFlux}$ ($3631\times 10^6 \times 10^{-0.4 \times 27.0}$). Because aperture corrections are not applied at the coadd image level, this scale is accurate to within a few percent.
- **Null handling and filter fallbacks:** The HATS catalogue does not provide an inverse-variance column (`ivar`). Undetected or unobserved bands appear as `NULL` fluxes rather than zero-weight entries. The default `catalog_filters` tests flux values directly for `NULL` to fall back from $i$ to $r$ and $z$.

### Catalogue: `hsc_pdr3_forced`

**Service URL:** `https://data.lsdb.io/hats/hsc_pdr3/hsc_pdr3-forced-G`

*Showing 100 of 241 total archive columns (all MOSAICS-mapped columns plus primary table fields; full 241-column schema is stored in `docs/_data/sources/hsc3/hats_hsc_pdr3.json`).*

| Column                                           | Type     | Unit   | UCD   | Description   |
|--------------------------------------------------|----------|--------|-------|---------------|
| `coord_ra`                                       | `double` | `—`    | `—`   | —             |
| `coord_dec`                                      | `double` | `—`    | `—`   | —             |
| `deblend_nChild`                                 | `int32`  | `—`    | `—`   | —             |
| `base_CircularApertureFlux_12_0_instFlux`        | `double` | `—`    | `—`   | —             |
| `base_CircularApertureFlux_12_0_instFluxErr`     | `double` | `—`    | `—`   | —             |
| `base_GaussianFlux_instFlux`                     | `double` | `—`    | `—`   | —             |
| `base_GaussianFlux_instFluxErr`                  | `double` | `—`    | `—`   | —             |
| `base_PsfFlux_instFlux`                          | `double` | `—`    | `—`   | —             |
| `base_PsfFlux_instFluxErr`                       | `double` | `—`    | `—`   | —             |
| `base_PsfFlux_area`                              | `float`  | `—`    | `—`   | —             |
| `base_Variance_value`                            | `double` | `—`    | `—`   | —             |
| `ext_photometryKron_KronFlux_instFlux`           | `double` | `—`    | `—`   | —             |
| `ext_photometryKron_KronFlux_instFluxErr`        | `double` | `—`    | `—`   | —             |
| `ext_photometryKron_KronFlux_radius`             | `float`  | `—`    | `—`   | —             |
| `modelfit_CModel_instFlux`                       | `double` | `—`    | `—`   | —             |
| `modelfit_CModel_instFluxErr`                    | `double` | `—`    | `—`   | —             |
| `base_ClassificationExtendedness_value`          | `double` | `—`    | `—`   | —             |
| `id`                                             | `int64`  | `—`    | `—`   | —             |
| `parent`                                         | `int64`  | `—`    | `—`   | —             |
| `base_SdssCentroid_x`                            | `double` | `—`    | `—`   | —             |
| `base_SdssCentroid_y`                            | `double` | `—`    | `—`   | —             |
| `base_SdssCentroid_xErr`                         | `float`  | `—`    | `—`   | —             |
| `base_SdssCentroid_yErr`                         | `float`  | `—`    | `—`   | —             |
| `base_TransformedCentroid_x`                     | `double` | `—`    | `—`   | —             |
| `base_TransformedCentroid_y`                     | `double` | `—`    | `—`   | —             |
| `base_InputCount_value`                          | `int32`  | `—`    | `—`   | —             |
| `base_SdssShape_xx`                              | `double` | `—`    | `—`   | —             |
| `base_SdssShape_yy`                              | `double` | `—`    | `—`   | —             |
| `base_SdssShape_xy`                              | `double` | `—`    | `—`   | —             |
| `base_SdssShape_xxErr`                           | `float`  | `—`    | `—`   | —             |
| `base_SdssShape_yyErr`                           | `float`  | `—`    | `—`   | —             |
| `base_SdssShape_xyErr`                           | `float`  | `—`    | `—`   | —             |
| `base_SdssShape_x`                               | `double` | `—`    | `—`   | —             |
| `base_SdssShape_y`                               | `double` | `—`    | `—`   | —             |
| `base_SdssShape_instFlux`                        | `double` | `—`    | `—`   | —             |
| `base_SdssShape_instFluxErr`                     | `double` | `—`    | `—`   | —             |
| `base_SdssShape_psf_xx`                          | `double` | `—`    | `—`   | —             |
| `base_SdssShape_psf_yy`                          | `double` | `—`    | `—`   | —             |
| `base_SdssShape_psf_xy`                          | `double` | `—`    | `—`   | —             |
| `base_SdssShape_instFlux_xx_Cov`                 | `float`  | `—`    | `—`   | —             |
| `base_SdssShape_instFlux_yy_Cov`                 | `float`  | `—`    | `—`   | —             |
| `base_SdssShape_instFlux_xy_Cov`                 | `float`  | `—`    | `—`   | —             |
| `base_TransformedShape_xx`                       | `double` | `—`    | `—`   | —             |
| `base_TransformedShape_yy`                       | `double` | `—`    | `—`   | —             |
| `base_TransformedShape_xy`                       | `double` | `—`    | `—`   | —             |
| `modelfit_DoubleShapeletPsfApprox_0_xx`          | `double` | `—`    | `—`   | —             |
| `modelfit_DoubleShapeletPsfApprox_0_yy`          | `double` | `—`    | `—`   | —             |
| `modelfit_DoubleShapeletPsfApprox_0_xy`          | `double` | `—`    | `—`   | —             |
| `modelfit_DoubleShapeletPsfApprox_0_x`           | `double` | `—`    | `—`   | —             |
| `modelfit_DoubleShapeletPsfApprox_0_y`           | `double` | `—`    | `—`   | —             |
| `modelfit_DoubleShapeletPsfApprox_0_0`           | `double` | `—`    | `—`   | —             |
| `modelfit_DoubleShapeletPsfApprox_0_1`           | `double` | `—`    | `—`   | —             |
| `modelfit_DoubleShapeletPsfApprox_0_2`           | `double` | `—`    | `—`   | —             |
| `modelfit_DoubleShapeletPsfApprox_0_3`           | `double` | `—`    | `—`   | —             |
| `modelfit_DoubleShapeletPsfApprox_0_4`           | `double` | `—`    | `—`   | —             |
| `modelfit_DoubleShapeletPsfApprox_0_5`           | `double` | `—`    | `—`   | —             |
| `modelfit_DoubleShapeletPsfApprox_1_xx`          | `double` | `—`    | `—`   | —             |
| `modelfit_DoubleShapeletPsfApprox_1_yy`          | `double` | `—`    | `—`   | —             |
| `modelfit_DoubleShapeletPsfApprox_1_xy`          | `double` | `—`    | `—`   | —             |
| `modelfit_DoubleShapeletPsfApprox_1_x`           | `double` | `—`    | `—`   | —             |
| `modelfit_DoubleShapeletPsfApprox_1_y`           | `double` | `—`    | `—`   | —             |
| `modelfit_DoubleShapeletPsfApprox_1_0`           | `double` | `—`    | `—`   | —             |
| `modelfit_DoubleShapeletPsfApprox_1_1`           | `double` | `—`    | `—`   | —             |
| `modelfit_DoubleShapeletPsfApprox_1_2`           | `double` | `—`    | `—`   | —             |
| `base_CircularApertureFlux_3_0_instFlux`         | `double` | `—`    | `—`   | —             |
| `base_CircularApertureFlux_3_0_instFluxErr`      | `double` | `—`    | `—`   | —             |
| `base_CircularApertureFlux_4_5_instFlux`         | `double` | `—`    | `—`   | —             |
| `base_CircularApertureFlux_4_5_instFluxErr`      | `double` | `—`    | `—`   | —             |
| `base_CircularApertureFlux_6_0_instFlux`         | `double` | `—`    | `—`   | —             |
| `base_CircularApertureFlux_6_0_instFluxErr`      | `double` | `—`    | `—`   | —             |
| `base_CircularApertureFlux_9_0_instFlux`         | `double` | `—`    | `—`   | —             |
| `base_CircularApertureFlux_9_0_instFluxErr`      | `double` | `—`    | `—`   | —             |
| `base_CircularApertureFlux_17_0_instFlux`        | `double` | `—`    | `—`   | —             |
| `base_CircularApertureFlux_17_0_instFluxErr`     | `double` | `—`    | `—`   | —             |
| `base_CircularApertureFlux_25_0_instFlux`        | `double` | `—`    | `—`   | —             |
| `base_CircularApertureFlux_25_0_instFluxErr`     | `double` | `—`    | `—`   | —             |
| `base_CircularApertureFlux_35_0_instFlux`        | `double` | `—`    | `—`   | —             |
| `base_CircularApertureFlux_35_0_instFluxErr`     | `double` | `—`    | `—`   | —             |
| `base_CircularApertureFlux_50_0_instFlux`        | `double` | `—`    | `—`   | —             |
| `base_CircularApertureFlux_50_0_instFluxErr`     | `double` | `—`    | `—`   | —             |
| `base_CircularApertureFlux_70_0_instFlux`        | `double` | `—`    | `—`   | —             |
| `base_CircularApertureFlux_70_0_instFluxErr`     | `double` | `—`    | `—`   | —             |
| `base_LocalBackground_instFlux`                  | `double` | `—`    | `—`   | —             |
| `base_LocalBackground_instFluxErr`               | `double` | `—`    | `—`   | —             |
| `ext_photometryKron_KronFlux_radius_for_radius`  | `float`  | `—`    | `—`   | —             |
| `ext_photometryKron_KronFlux_psf_radius`         | `float`  | `—`    | `—`   | —             |
| `ext_convolved_ConvolvedFlux_seeing`             | `float`  | `—`    | `—`   | —             |
| `ext_convolved_ConvolvedFlux_0_3_3_instFlux`     | `double` | `—`    | `—`   | —             |
| `ext_convolved_ConvolvedFlux_0_3_3_instFluxErr`  | `double` | `—`    | `—`   | —             |
| `ext_convolved_ConvolvedFlux_0_4_5_instFlux`     | `double` | `—`    | `—`   | —             |
| `ext_convolved_ConvolvedFlux_0_4_5_instFluxErr`  | `double` | `—`    | `—`   | —             |
| `ext_convolved_ConvolvedFlux_0_6_0_instFlux`     | `double` | `—`    | `—`   | —             |
| `ext_convolved_ConvolvedFlux_0_6_0_instFluxErr`  | `double` | `—`    | `—`   | —             |
| `ext_convolved_ConvolvedFlux_0_kron_instFlux`    | `double` | `—`    | `—`   | —             |
| `ext_convolved_ConvolvedFlux_0_kron_instFluxErr` | `double` | `—`    | `—`   | —             |
| `ext_convolved_ConvolvedFlux_1_3_3_instFlux`     | `double` | `—`    | `—`   | —             |
| `ext_convolved_ConvolvedFlux_1_3_3_instFluxErr`  | `double` | `—`    | `—`   | —             |
| `ext_convolved_ConvolvedFlux_1_4_5_instFlux`     | `double` | `—`    | `—`   | —             |
| `ext_convolved_ConvolvedFlux_1_4_5_instFluxErr`  | `double` | `—`    | `—`   | —             |
| `ext_convolved_ConvolvedFlux_1_6_0_instFlux`     | `double` | `—`    | `—`   | —             |

### Images

- **Pixel unit:** `ab_zeropoint` | **AB zero point:** `27.0` | **Calibration note:** HSC-SSP coadds carry a fixed 27.0 mag zero point, the same one the photometry’s ADU-to-uJy scale is derived from. Good to a few per cent: aperture corrections are not applied at the image level.
- **Cutout service `None`:** adapter `lsv`, bands `['g', 'r', 'i']`, params `{'layer': 'hsc-dr3', 'bands': 'gri'}`.
- **Cutout service `None`:** adapter `hips`, bands `['g']`, params `{'hips_id': 'CDS/P/HSC/DR2/wide/g'}`.
- **Cutout service `None`:** adapter `hips`, bands `['r']`, params `{'hips_id': 'CDS/P/HSC/DR2/wide/r'}`.
- **Cutout service `None`:** adapter `hips`, bands `['i']`, params `{'hips_id': 'CDS/P/HSC/DR2/wide/i'}`.

| HiPS ID                | Regime   | Frame        | Order   | Pixel scale (deg)   | Tile format   | Mirrors                                             |
|------------------------|----------|--------------|---------|---------------------|---------------|-----------------------------------------------------|
| `CDS/P/HSC/DR2/wide/g` | Optical  | `equatorial` | `12`    | `2.795E-5`          | `png fits`    | `alasky.cds.unistra.fr`, `alaskybis.cds.unistra.fr` |
| `CDS/P/HSC/DR2/wide/r` | Optical  | `equatorial` | `12`    | `2.795E-5`          | `png fits`    | `alasky.cds.unistra.fr`, `alaskybis.cds.unistra.fr` |
| `CDS/P/HSC/DR2/wide/i` | Optical  | `equatorial` | `12`    | `2.795E-5`          | `png fits`    | `alasky.cds.unistra.fr`, `alaskybis.cds.unistra.fr` |

### Spectra

No 1D spectroscopic backends are configured for this survey.

## References and acknowledgement

- Aihara, H. et al. (2022), *Third data release of the Hyper Suprime-Cam Subaru Strategic Program*, PASJ, 74, 247 (`doi:10.1093/pasj/psab122`).
- Aihara, H. et al. (2018), *The Hyper Suprime-Cam SSP Survey: Overview and survey design*, PASJ, 70, S4 (`doi:10.1093/pasj/psx081`).
- Bosch, J. et al. (2018), *The Hyper Suprime-Cam software processing pipeline*, PASJ, 70, S5 (`doi:10.1093/pasj/psx166`).
- Subaru HSC-SSP PDR3 Data Release: [https://hsc-release.mtk.nao.ac.jp/](https://hsc-release.mtk.nao.ac.jp/)
- HATS partitioned tables: [https://data.lsdb.io/hats/hsc_pdr3/](https://data.lsdb.io/hats/hsc_pdr3/)

The Hyper Suprime-Cam (HSC) collaboration includes the astronomical communities of Japan and Taiwan, and Princeton University. The HSC instrumentation and software were developed by the National Astronomical Observatory of Japan (NAOJ), the Kavli Institute for the Physics and Mathematics of the Universe (Kavli IPMU), the University of Tokyo, the High Energy Accelerator Research Organization (KEK), the Academia Sinica Institute for Astronomy and Astrophysics in Taiwan (ASIAA), and Princeton University. Funding was contributed by the FIRST program from the Japanese Cabinet Office, the Ministry of Education, Culture, Sports, Science and Technology (MEXT), the Japan Society for the Promotion of Science (JSPS), Japan Science and Technology Agency (JST), the Toray Science Foundation, NAOJ, Kavli IPMU, KEK, ASIAA, and Princeton University. Based on data collected at the Subaru Telescope and retrieved from the HSC data archive system, which is operated by the Subaru Telescope and Astronomy Data Center (ADC) at NAOJ.

## Provenance of this page

| Source ID       | Kind           | Service / Endpoint                                     | Query or URL                                                                                | Retrieved (UTC)        |
|-----------------|----------------|--------------------------------------------------------|---------------------------------------------------------------------------------------------|------------------------|
| `hats_hsc_pdr3` | `hats_catalog` | `https://data.lsdb.io/hats/hsc_pdr3/hsc_pdr3-forced-G` | `https://data.lsdb.io/hats/hsc_pdr3/hsc_pdr3-forced-G/properties`                           | `2026-10-18T16:58:30Z` |
| `hips_hsc`      | `hips`         | `https://alasky.cds.unistra.fr/MocServer/query`        | `https://alasky.cds.unistra.fr/MocServer/query?ID=CDS/P/HSC/DR2/wide/g&fmt=json&get=record` | `2026-10-18T16:32:16Z` |
<!-- /generated -->
