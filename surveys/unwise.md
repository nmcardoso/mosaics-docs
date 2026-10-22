# unWISE NEO11 (`unwise`)

<!-- generated:survey-page -->

unWISE provides unblurred, full-sky coadded imaging from NASA’s Wide-field Infrared Survey Explorer (WISE) and the NEOWISE reactivation mission. Unlike the standard AllWISE atlas images, unWISE preserves the native angular resolution of the WISE telescope without spatial smoothing or low-pass filtering. The NEO11 coadd data release incorporates eleven years of spaceborne survey observations in the mid-infrared W1 (3.4 µm) and W2 (4.6 µm) channels.

In `mosaics`, `unwise` serves as an image cutout survey delivering mid-infrared W1 and W2 FITS stamps via the Legacy Survey Viewer (`lsv` adapter with `layer: unwise-neo11`). Cutout pixel fluxes are calibrated in nanomaggies ($1\,\mathrm{nanomaggy} = 3.631\,\mu\mathrm{Jy}$, equivalent to an AB zero point of 22.5 mag).

| Property                 | Value                                                                                                            |
|--------------------------|------------------------------------------------------------------------------------------------------------------|
| What MOSAICS extracts    | Cutouts: `lsv` (w1,w2), `hips` (w1), `hips` (w2)                                                                 |
| Archives                 | Legacy Survey Viewer (`legacysurvey.org`), CDS HiPS / hips2fits (`alasky.cds.unistra.fr`), `datalab.noirlab.edu` |
| Bands / wavelength range | `w1`, `w2`                                                                                                       |
| Used by                  | `mosaics-cutout full`                                                                                            |

## Configuration

### The shipped file

Source: `defs/surveys/unwise.yaml`

```yaml
# unWISE NEO11 coadds -- the cutouts.
#
# A reprocessing of WISE imaging, and a different data product from the AllWISE
# catalogue the `wise` definition describes.
#
# The `unwise_*` photometry columns are NOT declared here: they come from the
# Legacy Survey DR11 tractor table, which carries unWISE forced photometry in
# the same rows as its optical fluxes, and defs/surveys/ls11.yaml describes
# them under `provides.unwise`. One query, so one description, in the file
# whose query it is. This file is the imaging. The depth cut on that
# photometry is `catalog_filters.unwise` in ls11.yaml, moved there with it.

survey_slug: unwise
name: unWISE NEO11

cutout:
  - adapter: lsv          # VERIFIED LIVE 2026-08-25
    adapter_params:
      layer: unwise-neo11
      bands: W1,W2          # comma form: band names are two characters
    bands: [w1, w2]
  # The same two bands as single-band HiPS with FITS tiles (MOCServer,
  # 2026-09-05). Note the keys are the *column* tokens, lowercased, while
  # the stack's plane order keeps the archive's `W1`/`W2` spelling.
  # Disabled: these are a different coadd generation from `unwise-neo11`,
  # so `pixel_unit: nanomaggy` -- a statement about the Viewer -- has not
  # been checked against them.
  - adapter: hips
    adapter_params:
      hips_id: CDS/P/unWISE/W1
    bands: [w1]
    enabled: false
  - adapter: hips
    adapter_params:
      hips_id: CDS/P/unWISE/W2
    bands: [w2]
    enabled: false

photometric_system:
  pixel_unit: nanomaggy
  pixel_unit_note: unWISE coadds are served in nanomaggies by the Legacy Survey Viewer.
  # No `borrow_filters` any more: `unwise` has photometry of its own now --
  # hosted by ls11 -- so the registry finds `unwise_w1` directly and a borrow
  # from `wise` would only be a second, quieter way to reach the same
  # WISE/WISE.W1 curve. Two routes to one answer is how they start to
  # disagree.
```

### Schema of this file

| Key present          | Type                   | Required   | Default   | Meaning                                                                                            |
|----------------------|------------------------|------------|-----------|----------------------------------------------------------------------------------------------------|
| `survey_slug`        | str                    | yes        | —         | Canonical lowercase identifier for the survey and default column prefix.                           |
| `name`               | str                    | yes        | —         | Human-readable survey and data-release name used in logs, plans, and reports.                      |
| `cutout`             | list of cutout entries | no         | []        | Image cutout backends (`hips`, `lsv`, `nersc`, `sia`, `vsa`) serving this survey’s passbands.      |
| `photometric_system` | mapping                | no         | None      | Passband SVO filter identifiers, flux/magnitude homogenisation rules, and image pixel calibration. |

### Adapter parameters

- **Cutout `None` (`adapter: lsv`, bands `['w1', 'w2']`):** configured `adapter_params`: `layer='unwise-neo11'`, `bands='W1,W2'`.
- **Cutout `None` (`adapter: hips`, bands `['w1']`):** configured `adapter_params`: `hips_id='CDS/P/unWISE/W1'`.
- **Cutout `None` (`adapter: hips`, bands `['w2']`):** configured `adapter_params`: `hips_id='CDS/P/unWISE/W2'`.

### Common changes

- **Disable unWISE cutouts**: To omit unWISE cutouts from a composite pipeline run:
  ```yaml
  overrides:
    surveys/unwise:
      cutout: []
  ```
- **Filter to a single band**: To request only W1 cutouts from the Legacy Survey Viewer:
  ```yaml
  overrides:
    surveys/unwise:
      cutout:
        - adapter: lsv
          adapter_params:
            layer: unwise-neo11
            bands: W1
          bands: [w1]
  ```
- **Enable CDS HiPS cutouts**: To use CDS HiPS FITS tiles instead of the Viewer:
  ```yaml
  overrides:
    surveys/unwise:
      cutout:
        - adapter: hips
          adapter_params:
            hips_id: CDS/P/unWISE/W1
          bands: [w1]
          enabled: true
  ```

### Commands

```console
$ uv run -m mosaics.cli.cutout full -t test/objects_sample_1k.csv -o cutouts.lance --config defs/cutout.yaml --survey unwise --plan
```

## Data served by the archive

- **Cutouts vs forced photometry**: In `mosaics`, `defs/surveys/unwise.yaml` configures cutouts only. unWISE forced photometry is not declared here; it is extracted as part of the DESI Legacy Imaging Surveys DR11 query (`defs/surveys/ls11.yaml` under `provides.unwise`). This query measures unWISE fluxes at optical target positions, yielding `unwise_*` columns (`unwise_flux_w1`, `unwise_flux_ivar_w1`, `unwise_rchisq_w1`, `unwise_fracflux_w1`, etc.) in AB nanomaggies.
- **unWISE AB fluxes vs AllWISE Vega magnitudes**: Do not confuse `unwise` forced photometry with standalone AllWISE photometry from `defs/surveys/wise.yaml`. AllWISE publishes profile-fit Vega magnitudes (`wise_mag_w1`, etc.). In contrast, `unwise` forced photometry is already on the AB scale ($m_\mathrm{AB} = 22.5 - 2.5 \log_{10}(\mathrm{nanomaggy})$). Applying AllWISE Vega-to-AB offsets to `unwise` fluxes is an error.
- **Cutout backend and pixel calibration**: The active cutout adapter queries the `unwise-neo11` layer from the Legacy Survey Viewer in nanomaggies (`pixel_unit: nanomaggy`). Secondary CDS HiPS adapters (`CDS/P/unWISE/W1`, `CDS/P/unWISE/W2`) represent an earlier coaddition generation and are disabled by default (`enabled: false`) because their pixel scale and calibration differ from the Viewer.
- **Filter definitions**: `borrow_filters` is omitted from `unwise.yaml` because the filter registry resolves `unwise_w1` directly from `ls11.yaml`’s `provides.unwise` definition (`WISE/WISE.W1`), preventing duplicate filter paths.

### Catalogue: `unwise_dr1.object`

Most columns in the unWISE band-merged catalogs are identical to

**Service URL:** `https://datalab.noirlab.edu/tap`

| Column             | Type            | Unit     | UCD                    | Description                                                      |
|--------------------|-----------------|----------|------------------------|------------------------------------------------------------------|
| `x_w1`             | `adql:DOUBLE`   | `pix`    | `—`                    | W1 channel x coordinate (pix)                                    |
| `x_w2`             | `adql:DOUBLE`   | `pix`    | `—`                    | W2 channel x coordinate (pix)                                    |
| `y_w1`             | `adql:DOUBLE`   | `pix`    | `—`                    | W1 channel y coordinate (pix)                                    |
| `y_w2`             | `adql:DOUBLE`   | `pix`    | `—`                    | W2 channel y coordinate (pix)                                    |
| `ra12_w1`          | `adql:DOUBLE`   | `—`      | `—`                    | Positions from individual-image catalogs                         |
| `ra12_w2`          | `adql:DOUBLE`   | `—`      | `—`                    | Positions from individual-image catalogs                         |
| `dec12_w1`         | `adql:DOUBLE`   | `—`      | `—`                    | Positions from individual-image catalogs                         |
| `dec12_w2`         | `adql:DOUBLE`   | `—`      | `—`                    | Positions from individual-image catalogs                         |
| `ra`               | `adql:DOUBLE`   | `—`      | `pos.eq.ra;meta.main`  | W1 position, if available; otherwise W2 position                 |
| `dec`              | `adql:DOUBLE`   | `—`      | `pos.eq.dec;meta.main` | W1 position, if available; otherwise W2 position                 |
| `glon`             | `adql:DOUBLE`   | `deg`    | `—`                    | Galactic Longitude                                               |
| `glat`             | `adql:DOUBLE`   | `deg`    | `—`                    | Galactic Latitude                                                |
| `elon`             | `adql:DOUBLE`   | `deg`    | `—`                    | Ecliptic Longitude                                               |
| `elat`             | `adql:DOUBLE`   | `deg`    | `—`                    | Ecliptic Latitude                                                |
| `flux_w1`          | `adql:REAL`     | `nMgy`   | `—`                    | W1 flux (Vega nMgy)                                              |
| `flux_w2`          | `adql:REAL`     | `nMgy`   | `—`                    | W2 flux (Vega nMgy)                                              |
| `dx_w1`            | `adql:REAL`     | `—`      | `—`                    | uncertainty in W1 x position                                     |
| `dx_w2`            | `adql:REAL`     | `—`      | `—`                    | uncertainty in W2 x position                                     |
| `dy_w1`            | `adql:REAL`     | `—`      | `—`                    | uncertainty in W1 y position                                     |
| `dy_w2`            | `adql:REAL`     | `—`      | `—`                    | uncertainty in W2 y position                                     |
| `dflux_w1`         | `adql:REAL`     | `—`      | `—`                    | uncertainty in W1 flux (statistical only)                        |
| `dflux_w2`         | `adql:REAL`     | `—`      | `—`                    | uncertainty in W2 flux (statistical only)                        |
| `qf_w1`            | `adql:REAL`     | `—`      | `—`                    | quality factor for W1                                            |
| `qf_w2`            | `adql:REAL`     | `—`      | `—`                    | quality factor for W2                                            |
| `rchi2_w1`         | `adql:REAL`     | `—`      | `—`                    | average X^2 per pixel, weighted by PSF, for W1                   |
| `rchi2_w2`         | `adql:REAL`     | `—`      | `—`                    | average X^2 per pixel, weighted by PSF, for W2                   |
| `fracflux_w1`      | `adql:REAL`     | `—`      | `—`                    | fraction of W1 flux in this objects PSF that comes from this obj |
| `fracflux_w2`      | `adql:REAL`     | `—`      | `—`                    | fraction of W2 flux in this objects PSF that comes from this obj |
| `fluxlbs_w1`       | `adql:REAL`     | `nMgy`   | `—`                    | local-background-subtracted flux (Vega nMgy) in W1               |
| `fluxlbs_w2`       | `adql:REAL`     | `nMgy`   | `—`                    | local-background-subtracted flux (Vega nMgy) in W2               |
| `dfluxlbs_w1`      | `adql:REAL`     | `—`      | `—`                    | uncertainty in local-background-subtracted flux in W1            |
| `dfluxlbs_w2`      | `adql:REAL`     | `—`      | `—`                    | uncertainty in local-background-subtracted flux in W2            |
| `fwhm_w1`          | `adql:REAL`     | `pixels` | `—`                    | full-width at half-maximum of PSF (pixels) in W1                 |
| `fwhm_w2`          | `adql:REAL`     | `pixels` | `—`                    | full-width at half-maximum of PSF (pixels) in W2                 |
| `spread_model_w1`  | `adql:REAL`     | `—`      | `—`                    | SExtractor spread_model parameter for W1                         |
| `spread_model_w2`  | `adql:REAL`     | `—`      | `—`                    | SExtractor spread_model parameter for W2                         |
| `dspread_model_w1` | `adql:REAL`     | `—`      | `—`                    | uncertainty in spread_model for W1                               |
| `dspread_model_w2` | `adql:REAL`     | `—`      | `—`                    | uncertainty in spread_model for W2                               |
| `sky_w1`           | `adql:REAL`     | `nMgy`   | `—`                    | sky (Vega nMgy) in W1                                            |
| `sky_w2`           | `adql:REAL`     | `nMgy`   | `—`                    | sky (Vega nMgy) in W2                                            |
| `htm9`             | `adql:INTEGER`  | `—`      | `—`                    | HTM index (order 9 => ~10 arcmin size)                           |
| `ring256`          | `adql:INTEGER`  | `—`      | `—`                    | HEALPIX index (Nsides 256 => ~14 arcmin size)                    |
| `nest4096`         | `adql:INTEGER`  | `—`      | `—`                    | HEALPIX index (Nsides 4096, Nest scheme => ~52 arcsec size       |
| `mag_w1_vg`        | `adql:REAL`     | `—`      | `—`                    | W1 magnitude [Vega]                                              |
| `mag_w2_vg`        | `adql:REAL`     | `—`      | `—`                    | W2 magnitude [Vega]                                              |
| `w1_w2_vg`         | `adql:REAL`     | `—`      | `—`                    | W1-W2 color [Vega]                                               |
| `random_id`        | `adql:REAL`     | `—`      | `—`                    | Random ID in the range 0.0 => 100.0                              |
| `nm_w1`            | `adql:SMALLINT` | `—`      | `—`                    | number of single-exposure W1 images of this part of sky in coadd |
| `nm_w2`            | `adql:SMALLINT` | `—`      | `—`                    | number of single-exposure W2 images of this part of sky in coadd |
| `primary12_w1`     | `adql:SMALLINT` | `—`      | `—`                    | the center of this source is in the primary region of its coadd  |
| `primary12_w2`     | `adql:SMALLINT` | `—`      | `—`                    | the center of this source is in the primary region of its coadd  |
| `flags_unwise_w1`  | `adql:SMALLINT` | `—`      | `—`                    | unWISE Coadd Flags flags at central pixel for W1                 |
| `flags_unwise_w2`  | `adql:SMALLINT` | `—`      | `—`                    | unWISE Coadd Flags flags at central pixel for W2                 |
| `flags_info_w1`    | `adql:SMALLINT` | `—`      | `—`                    | additional informational flags at central pixel for W1           |
| `flags_info_w2`    | `adql:SMALLINT` | `—`      | `—`                    | additional informational flags at central pixel for W2           |
| `primary`          | `adql:SMALLINT` | `—`      | `—`                    | the center of this source is in the primary region of its coadd  |
| `coadd_id`         | `adql:VARCHAR`  | `—`      | `—`                    | Unique astrometric footprint string encoding the tile central (R |
| `unwise_detid_w1`  | `adql:VARCHAR`  | `—`      | `—`                    | unique ID                                                        |
| `unwise_detid_w2`  | `adql:VARCHAR`  | `—`      | `—`                    | unique ID                                                        |
| `unwise_objid`     | `adql:VARCHAR`  | `—`      | `meta.id;meta.main`    | unique object id                                                 |

### Images

- **Pixel unit:** `nanomaggy` | **AB zero point:** `—` | **Calibration note:** unWISE coadds are served in nanomaggies by the Legacy Survey Viewer.
- **Cutout service `None`:** adapter `lsv`, bands `['w1', 'w2']`, params `{'layer': 'unwise-neo11', 'bands': 'W1,W2'}`.
- **Cutout service `None`:** adapter `hips`, bands `['w1']`, params `{'hips_id': 'CDS/P/unWISE/W1'}`.
- **Cutout service `None`:** adapter `hips`, bands `['w2']`, params `{'hips_id': 'CDS/P/unWISE/W2'}`.

| HiPS ID           | Regime   | Frame        | Order   | Pixel scale (deg)   | Tile format   | Mirrors                                             |
|-------------------|----------|--------------|---------|---------------------|---------------|-----------------------------------------------------|
| `CDS/P/unWISE/W1` | Infrared | `equatorial` | `8`     | `0.229`             | `jpeg fits`   | `alasky.cds.unistra.fr`, `alaskybis.cds.unistra.fr` |
| `CDS/P/unWISE/W2` | Infrared | `equatorial` | `8`     | `0.229`             | `jpeg fits`   | `alasky.cds.unistra.fr`, `alaskybis.cds.unistra.fr` |

### Spectra

No 1D spectroscopic backends are configured for this survey.

## References and acknowledgement

- Lang, D. (2014), *unWISE: Unblurred Coadds of the WISE Imaging*, *AJ*, 147, 108 (`doi:10.1088/0004-6256/147/5/108`).
- Meisner, A. M., Lang, D., & Schlegel, D. J. (2017), *Deep Full-sky Coadds from 3 Years of WISE and NEOWISE Data*, *AJ*, 154, 161 (`doi:10.3847/1538-3881/aa894e`).
- Meisner, A. M. et al. (2021), *The unWISE Catalog: Two Billion Infrared Sources from Five Years of WISE Imaging*, *ApJS*, 257, 64 (`doi:10.3847/1538-4365/ac2a38`).
- Schlafly, E. F. et al. (2019), *The unWISE Catalog of Two Billion Objects*, *ApJS*, 240, 30 (`doi:10.3847/1538-4365/aafbea`).
- Legacy Survey Viewer API: [https://www.legacysurvey.org/viewer](https://www.legacysurvey.org/viewer)
- CDS HiPS records: `CDS/P/unWISE/W1`, `CDS/P/unWISE/W2`.

This publication makes use of data products from the Wide-field Infrared Survey Explorer, which is a joint project of the University of California, Los Angeles, and the Jet Propulsion Laboratory/California Institute of Technology, funded by the National Aeronautics and Space Administration. This work also makes use of the unWISE coadds and catalogues generated by Lang (2014) and Meisner et al. When using Legacy Survey Viewer cutout services, acknowledge the DESI Legacy Imaging Surveys.

## Provenance of this page

| Source ID        | Kind        | Service / Endpoint                              | Query or URL                                                                                                          | Retrieved (UTC)        |
|------------------|-------------|-------------------------------------------------|-----------------------------------------------------------------------------------------------------------------------|------------------------|
| `catalog_unwise` | `tap_table` | `https://datalab.noirlab.edu/tap`               | `SELECT column_name, datatype, unit, ucd, description FROM TAP_SCHEMA.columns WHERE table_name = 'unwise_dr1.object'` | `2026-10-18T16:40:29Z` |
| `hips_unwise`    | `hips`      | `https://alasky.cds.unistra.fr/MocServer/query` | `https://alasky.cds.unistra.fr/MocServer/query?ID=CDS/P/unWISE/W1&fmt=json&get=record`                                | `2026-10-18T16:40:33Z` |
<!-- /generated -->
