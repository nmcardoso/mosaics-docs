# Rubin Observatory Data Preview 2 (`lsstdp2`)

<!-- generated:survey-page -->

Rubin Observatory Data Preview 2 (`lsstdp2`) is a release of the Vera C. Rubin Observatory Legacy Survey of Space and Time (LSST): an optical wide-fast-deep survey of the southern sky made with the 8.4-metre Simonyi Survey Telescope at Cerro Pachón, Chile, through six broad bands ($u, g, r, i, z, y$, covering 3200–10800 Å). Early commissioning and validation products are released as Data Previews through the Rubin Science Platform (RSP). Data Preview 2 carries deep coadd images and all catalogs of its commissioning fields.

In MOSAICS the file is a survey file like any other, matched by `mosaics-crossmatch`: the `dp2.Object` table is queried over the authenticated RSP TAP service with only `(key, ra, dec)` sent, and the Object columns the file maps are joined back onto the table. It is left out of `defs/full.yaml` and `defs/targets.yaml`, because the RSP needs an access token and a targets run must not depend on one; a crossmatch job lists it itself (`surveys: [surveys/lsstdp2]`).

| Property                 | Value                                                                                                                                                                                                         |
|--------------------------|---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| What MOSAICS extracts    | Catalogue: `lsstdp2` (photometric, `lsst`)                                                                                                                                                                    |
| Archives                 | `data.lsst.cloud`                                                                                                                                                                                             |
| Bands / wavelength range | —                                                                                                                                                                                                             |
| Used by                  | `mosaics-crossmatch`                                                                                                                                                                                          |
| Crossmatch radius        | `1` arcsec. No archive source was found for the PSF or the astrometric precision of Data Preview 2. Value set by the author (HG2, 2026-10-19); the research suggested the 1 arcsec of lsstdp1 as a reference. |

## Configuration

### The shipped file

Source: `defs/surveys/lsstdp2.yaml`

```yaml
# Rubin Observatory Data Preview 2 -- the Object table, crossmatch only.
#
# Not part of defs/full.yaml or defs/targets.yaml (full.yaml `exclude:`s it): the
# RSP needs a token (LSST_TOKEN), and a targets run must not depend on one. Use it
# from a crossmatch job:
#
#     surveys: [surveys/lsstdp2]
#
# Columns are what runs/gissel.sh asked the old LSST command for. Aliases carry no prefix:
# the pipeline adds `lsstdp2_`. A pattern key selects every remote column it
# matches; {1} is the text its wildcard matched (the band).
#
# `mosaics-crossmatch moc --survey surveys/lsstdp2 -o lsstdp2-moc.fits` builds the
# footprint a job's `--filter-moc` reads (see `footprint_table`).

survey_slug: lsstdp2
name: Rubin Observatory Data Preview 2

catalog:
  - name: lsstdp2
    type: photometric
    adapter: lsst
    adapter_params:
      release: dp2
      table: dp2.Object
      remote_ra_column: coord_ra
      remote_dec_column: coord_dec
      footprint_table: dp2.CoaddPatches
      # chunk_mib / use_async: the command line's defaults (28 MiB, measured).

column_mapping:
  objectId: id
  coord_ra: ra
  coord_dec: dec
  '[ugrizy]_cModelMag': mag_cmodel_{1}            # -> lsstdp2_mag_cmodel_u ... _y
  '[ugrizy]_cModelMagErr': mag_cmodel_{1}_err
  '[ugrizy]_sersicFlux': flux_sersic_{1}          # nJy
  '[ugrizy]_sersicFluxErr': flux_sersic_{1}_err
  sersic_index: sersic_index
  sersic_indexErr: sersic_index_err
  sersic_ra: sersic_ra
  sersic_dec: sersic_dec
  sersic_raErr: sersic_ra_err
  sersic_decErr: sersic_dec_err
  sersic_ra_dec_Cov: sersic_ra_dec_cov
  sersic_reff_x: sersic_reff_x
  sersic_reff_xErr: sersic_reff_x_err
  sersic_reff_y: sersic_reff_y
  sersic_reff_yErr: sersic_reff_y_err
  sersic_reff_major: sersic_reff_major
  sersic_reff_minor: sersic_reff_minor
  sersic_rho: sersic_rho
  sersic_rhoErr: sersic_rho_err
  sersic_theta: sersic_theta
  sersic_n_eval_jac: sersic_n_eval_jac
  sersic_n_iter: sersic_n_iter
  sersic_chi2_reduced: sersic_chi2_reduced
  sersic_unknown_flag: sersic_unknown_flag
  sersic_no_data_flag: sersic_no_data_flag
  shape_flag: shape_flag
  ebv: ebv
  deblend_blendNChild: deblend_blend_n_child
  detect_fromBlend: detect_from_blend
  detect_isIsolated: detect_is_isolated

crossmatch:
  radius: 1 arcsec
  radius_note: >-
    No archive source was found for the PSF or the astrometric precision of Data Preview 2.
    Value set by the author (HG2, 2026-10-19); the research suggested the 1 arcsec of
    lsstdp1 as a reference.
```

### Schema of this file

| Key present      | Type                             | Required   | Default   | Meaning                                                                                                                                                                                                                      |
|------------------|----------------------------------|------------|-----------|------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `survey_slug`    | str                              | yes        | —         | Canonical lowercase identifier for the survey and default column prefix.                                                                                                                                                     |
| `name`           | str                              | yes        | —         | Human-readable survey and data-release name used in logs, plans, and reports.                                                                                                                                                |
| `catalog`        | list of catalog entries          | no         | []        | Remote or local catalogues queried for target extraction or crossmatching.                                                                                                                                                   |
| `column_mapping` | mapping (remote_column -> alias) | no         | {}        | Projection and column renaming applied when crossmatching against this survey’s photometric catalogue. The loader adds the catalog’s prefix to every alias, or the hosted survey’s for the columns under its `provides` key. |
| `crossmatch`     | mapping                          | no         | None      | How a table of positions is matched against this survey. Required for a survey with an `lsdb` catalog; `crossmatch` null removes it, and a survey without it takes no part in `mosaics-crossmatch`.                          |

### Adapter parameters

**Catalogue `lsstdp2` (`adapter: lsst`):**

| Parameter           | Allowed Type   | Default                                                            | Description                                                                                                              |
|---------------------|----------------|--------------------------------------------------------------------|--------------------------------------------------------------------------------------------------------------------------|
| `release`           | str            | None (required unless `service` is given)                          | Data release: `dp2`, `dp1`, `dp02` (`https://data.lsst.cloud/api/tap`) or `dp03` (`https://data.lsst.cloud/api/ssotap`). |
| `table`             | str            | —                                                                  | Remote table, e.g. `dp2.Object`.                                                                                         |
| `service`           | str            | the release’s endpoint                                             | A full TAP endpoint overriding the release’s, for a test archive or a non-standard service.                              |
| `remote_ra_column`  | str            | found by UCD (`pos.eq.ra`), then by name                           | Remote right ascension column name.                                                                                      |
| `remote_dec_column` | str            | found by UCD (`pos.eq.dec`), then by name                          | Remote declination column name.                                                                                          |
| `footprint_table`   | str            | None                                                               | The release’s `CoaddPatches` table (`dp2.CoaddPatches`), from which `mosaics-crossmatch moc` builds the footprint.       |
| `use_async`         | bool           | false (switches on by itself when the first upload chunk is large) | Submit every upload as a server-side job instead of a synchronous query.                                                 |
| `chunk_mib`         | float          | 28.0 (the command line’s `--chunk-mib`)                            | Byte budget of one inline upload, in MiB. Rubin caps an upload at about 33 MiB.                                          |

### Common changes

- Match a table against this release, with its radius and columns from the survey file:
  ```console
  $ uv run -m mosaics.crossmatch.cli run --survey surveys/lsstdp2 -i test/objects_sample_1k.csv -o lsstdp2_matched.parquet
  ```
- Build the release footprint once and pre-filter crossmatches to skip targets outside Rubin coverage:
  ```console
  $ uv run -m mosaics.crossmatch.cli moc --survey surveys/lsstdp2 -o lsstdp2-moc.fits
  ```

  then name the file under `overrides:` (`crossmatch: {moc: lsstdp2-moc.fits}`) and run with `--filter-moc`.
- Ask for other Object columns (patterns are allowed) from the command line, without editing the file:
  ```console
  $ uv run -m mosaics.crossmatch.cli run --survey surveys/lsstdp2 -i targets.parquet -o out.parquet --columns objectId coord_ra coord_dec '*_cModelMag'
  ```
- Tune upload chunk size and concurrency for the network you are on:
  ```console
  --chunk-mib 1 -j 6 --limit-share 0.85
  ```

### Commands

```console
$ uv run -m mosaics.crossmatch.cli run --survey surveys/lsstdp2 -i test/objects_sample_1k.csv -o lsstdp2_matched.parquet
```

## Data served by the archive

- **RSP token authentication:** Unlike public TAP services that permit anonymous access, Rubin TAP requires an authenticated Rubin Science Platform (RSP) access token, read from `--lsst-token` or the environment (`LSST_TOKEN`, then `LSST_PASSWORD`, `LSST_PASS`). Authentication is transmitted via HTTP Basic with the literal username `x-oauth-basic` and the token as the password. A token is never read from a YAML file.
- **Rate limiting and concurrency budgets:** Rubin enforces per-account rate limits of 1,000 requests per minute and a ceiling of 12 concurrent Qserv queries. The pipeline enforces these bounds locally through a windowed rate limiter, claiming a configurable fraction (default `--limit-share 0.85`, or 850 requests/min and 10 concurrent slots) so that concurrent tools on a shared account are not starved.
- **Upload payload size limits:** Rubin TAP enforces strict inline upload payload limits near 33 MiB. The crossmatch uploads positions in narrow batches containing only `(key, ra, dec)`, budgeted by default to 28 MiB (`--chunk-mib 28`), to maximize the number of target positions per chunk without exceeding the upload ceiling.
- **Footprint filtering with MOC:** Because the Data Previews cover targeted commissioning fields rather than the full sky, crossmatching an all-sky target catalog directly sends hundreds of futile queries. `mosaics-crossmatch moc` builds a Multi-Order Coverage FITS map from the `dp2.CoaddPatches` table (`s_region` polygons at HEALPix order 10, $\sim 3.4'$ resolution); `--filter-moc` excludes out-of-footprint targets locally before upload.
- **Synchronous vs asynchronous execution:** Queries submit synchronously by default for fast turnaround, but switch automatically to asynchronous TAP jobs once payload sizes exceed upload thresholds or when `--use-async` is specified, preventing wall-clock query timeouts.

### Catalogue: `dp2.CoaddPatches`

Static information about the subset of tracts and patches from the standard LSST skymap that apply to coadds in these catalogs.

**Service URL:** `https://data.lsst.cloud/api/tap`

| Column       | Type     | Unit   | UCD                      | Description                                                             |
|--------------|----------|--------|--------------------------|-------------------------------------------------------------------------|
| `lsst_tract` | `long`   | `—`    | `meta.id;obs.field`      | ID number of the top level, ‘tract’, within the standard LSST skymap    |
| `lsst_patch` | `long`   | `—`    | `meta.id.part;obs.field` | ID number of the second level, ‘patch’, within the standard LSST skymap |
| `s_ra`       | `double` | `deg`  | `pos.eq.ra`              | Central Spatial Position in ICRS; Right ascension                       |
| `s_dec`      | `double` | `deg`  | `pos.eq.dec`             | Central Spatial Position in ICRS; Declination                           |
| `s_region`   | `char`   | `—`    | `pos.outline;obs.field`  | Sky region covered by the coadd (expressed in ICRS frame)               |

### Catalogue: `dp2.Object`

Descriptions of static astronomical objects (or the static aspects of variable and slowly-moving objects) detected and measured on coadds.

**Service URL:** `https://data.lsst.cloud/api/tap`

*Showing 100 of 1248 total archive columns (all MOSAICS-mapped columns plus primary table fields; full 1248-column schema is stored in `docs/_data/sources/lsstdp2/lsst_dp2_object.json`).*

| Column                    | Type      | Unit       | UCD                                              | Description                                                                                                                                                                         |
|---------------------------|-----------|------------|--------------------------------------------------|-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `objectId`                | `long`    | `—`        | `meta.id;src`                                    | Unique id. Unique ObjectID                                                                                                                                                          |
| `coord_ra`                | `double`  | `deg`      | `pos.eq.ra;meta.main`                            | Fiducial ICRS Right Ascension of centroid used for database indexing                                                                                                                |
| `coord_dec`               | `double`  | `deg`      | `pos.eq.dec;meta.main`                           | Fiducial ICRS Declination of centroid used for database indexing                                                                                                                    |
| `u_cModelMag`             | `float`   | `mag`      | `—`                                              | AB magnitude of cModelFlux. Forced on u-band.                                                                                                                                       |
| `u_cModelMagErr`          | `float`   | `mag`      | `—`                                              | Uncertainty in magnitudes on cModelFlux. Forced on u-band.                                                                                                                          |
| `g_cModelMag`             | `float`   | `mag`      | `—`                                              | AB magnitude of cModelFlux. Forced on g-band.                                                                                                                                       |
| `g_cModelMagErr`          | `float`   | `mag`      | `—`                                              | Uncertainty in magnitudes on cModelFlux. Forced on g-band.                                                                                                                          |
| `r_cModelMag`             | `float`   | `mag`      | `—`                                              | AB magnitude of cModelFlux. Forced on r-band.                                                                                                                                       |
| `r_cModelMagErr`          | `float`   | `mag`      | `—`                                              | Uncertainty in magnitudes on cModelFlux. Forced on r-band.                                                                                                                          |
| `i_cModelMag`             | `float`   | `mag`      | `—`                                              | AB magnitude of cModelFlux. Forced on i-band.                                                                                                                                       |
| `i_cModelMagErr`          | `float`   | `mag`      | `—`                                              | Uncertainty in magnitudes on cModelFlux. Forced on i-band.                                                                                                                          |
| `z_cModelMag`             | `float`   | `mag`      | `—`                                              | AB magnitude of cModelFlux. Forced on z-band.                                                                                                                                       |
| `z_cModelMagErr`          | `float`   | `mag`      | `—`                                              | Uncertainty in magnitudes on cModelFlux. Forced on z-band.                                                                                                                          |
| `y_cModelMag`             | `float`   | `mag`      | `—`                                              | AB magnitude of cModelFlux. Forced on y-band.                                                                                                                                       |
| `y_cModelMagErr`          | `float`   | `mag`      | `—`                                              | Uncertainty in magnitudes on cModelFlux. Forced on y-band.                                                                                                                          |
| `ebv`                     | `float`   | `mag`      | `—`                                              | E(B-V) at coord_ra/coord_dec per Schlegel, Finkbeiner & Davis (1998)                                                                                                                |
| `detect_isIsolated`       | `boolean` | `—`        | `—`                                              | This source is not a part of a blend.                                                                                                                                               |
| `detect_fromBlend`        | `boolean` | `—`        | `—`                                              | This source is deblended from a parent with more than one child.                                                                                                                    |
| `deblend_blendNChild`     | `int`     | `—`        | `—`                                              | Number of children in the deconvolved parent source that this source was deblended from.                                                                                            |
| `shape_flag`              | `boolean` | `—`        | `—`                                              | General failure flag, set if anything went wrong. Reference band.                                                                                                                   |
| `sersic_ra`               | `double`  | `deg`      | `pos.eq.ra`                                      | Centroid (right ascension) from the multiband Sersic model fit.                                                                                                                     |
| `sersic_raErr`            | `float`   | `deg`      | `stat.error;pos.eq.ra`                           | Error on the centroid (right ascension) from the multiband Sersic model fit.                                                                                                        |
| `sersic_dec`              | `double`  | `deg`      | `pos.eq.dec`                                     | Centroid (declination) from the multiband Sersic model fit.                                                                                                                         |
| `sersic_decErr`           | `float`   | `deg`      | `stat.error;pos.eq.dec`                          | Error on the centroid (declination) from the multiband Sersic model fit.                                                                                                            |
| `sersic_ra_dec_Cov`       | `float`   | `deg**2`   | `stat.covariance;pos.eq.ra;pos.eq.dec`           | Covariance between right ascension and declination from the multiband Sersic model fit.                                                                                             |
| `sersic_reff_x`           | `float`   | `pixel`    | `—`                                              | Effective radius (x-axis) from the multiband Sersic model fit, prior to convolution with the PSF model (which is shrunk by 0.1 pixels in quadrature by default).                    |
| `sersic_reff_xErr`        | `float`   | `pixel`    | `—`                                              | Error on the effective radius (x-axis) from the multiband Sersic model fit.                                                                                                         |
| `sersic_reff_y`           | `float`   | `pixel`    | `—`                                              | Effective radius (y-axis) from the multiband Sersic model fit, prior to convolution with the PSF model (which is shrunk by 0.1 pixels in quadrature by default).                    |
| `sersic_reff_yErr`        | `float`   | `pixel`    | `—`                                              | Error on the effective radius (y-axis) from the multiband Sersic model fit.                                                                                                         |
| `sersic_rho`              | `float`   | `—`        | `—`                                              | Ellipse rho (correlation coefficient) from the multiband Sersic model fit.                                                                                                          |
| `sersic_rhoErr`           | `float`   | `—`        | `—`                                              | Error on ellipse rho (correlation coefficient) from the multiband Sersic model fit.                                                                                                 |
| `sersic_reff_major`       | `float`   | `arcsec`   | `—`                                              | Effective radius (major axis) from the multiband Sersic model fit, prior to convolution with the PSF model (which is shrunk by 0.1 pixels in quadrature by default).                |
| `sersic_reff_minor`       | `float`   | `arcsec`   | `—`                                              | Effective radius (minor axis) from the multiband Sersic model fit, prior to convolution with the PSF model (which is shrunk by 0.1 pixels in quadrature by default).                |
| `sersic_theta`            | `float`   | `—`        | `—`                                              | Position angle of the multiband Sersic fit ellipse.                                                                                                                                 |
| `u_sersicFlux`            | `float`   | `nJy`      | `—`                                              | u-band flux from the multiband Sersic model fit.                                                                                                                                    |
| `u_sersicFluxErr`         | `float`   | `nJy`      | `—`                                              | Error on the u-band flux from the multiband Sersic model fit.                                                                                                                       |
| `g_sersicFlux`            | `float`   | `nJy`      | `—`                                              | g-band flux from the multiband Sersic model fit.                                                                                                                                    |
| `g_sersicFluxErr`         | `float`   | `nJy`      | `—`                                              | Error on the g-band flux from the multiband Sersic model fit.                                                                                                                       |
| `r_sersicFlux`            | `float`   | `nJy`      | `—`                                              | r-band flux from the multiband Sersic model fit.                                                                                                                                    |
| `r_sersicFluxErr`         | `float`   | `nJy`      | `—`                                              | Error on the r-band flux from the multiband Sersic model fit.                                                                                                                       |
| `i_sersicFlux`            | `float`   | `nJy`      | `—`                                              | i-band flux from the multiband Sersic model fit.                                                                                                                                    |
| `i_sersicFluxErr`         | `float`   | `nJy`      | `—`                                              | Error on the i-band flux from the multiband Sersic model fit.                                                                                                                       |
| `z_sersicFlux`            | `float`   | `nJy`      | `—`                                              | z-band flux from the multiband Sersic model fit.                                                                                                                                    |
| `z_sersicFluxErr`         | `float`   | `nJy`      | `—`                                              | Error on the z-band flux from the multiband Sersic model fit.                                                                                                                       |
| `y_sersicFlux`            | `float`   | `nJy`      | `—`                                              | y-band flux from the multiband Sersic model fit.                                                                                                                                    |
| `y_sersicFluxErr`         | `float`   | `nJy`      | `—`                                              | Error on the y-band flux from the multiband Sersic model fit.                                                                                                                       |
| `sersic_index`            | `float`   | `—`        | `—`                                              | Sersic profile index parameter from the multiband Sersic model fit.                                                                                                                 |
| `sersic_indexErr`         | `float`   | `—`        | `—`                                              | Error on the Sersic profile index parameter from the multiband Sersic model fit.                                                                                                    |
| `sersic_n_eval_jac`       | `int`     | `—`        | `—`                                              | Number of Jacobian evaluations for the multiband Sersic model fit.                                                                                                                  |
| `sersic_n_iter`           | `int`     | `—`        | `—`                                              | Number of iterations in the non-linear fit for the multiband Sersic model.                                                                                                          |
| `sersic_chi2_reduced`     | `float`   | `—`        | `—`                                              | Reduced chi-squared of the best-fit model parameters (chi divided the number of data points) for the multiband Sersic model.                                                        |
| `sersic_unknown_flag`     | `boolean` | `—`        | `—`                                              | Flag set for failures with an unexpected or unknown cause for the multiband Sersic model.                                                                                           |
| `sersic_no_data_flag`     | `boolean` | `—`        | `—`                                              | Flag set when there is insufficient data to fit for the multiband Sersic model.                                                                                                     |
| `tract`                   | `long`    | `—`        | `—`                                              | Skymap tract ID                                                                                                                                                                     |
| `patch`                   | `long`    | `—`        | `—`                                              | Skymap patch ID                                                                                                                                                                     |
| `coord_raErr`             | `float`   | `deg`      | `stat.error;pos.eq.ra;meta.main`                 | Angular error in fiducial ICRS Right Ascension (∂RA\*cos(Dec)) of centroid                                                                                                          |
| `coord_decErr`            | `float`   | `deg`      | `stat.error;pos.eq.dec;meta.main`                | Error in fiducial ICRS Declination of centroid                                                                                                                                      |
| `coord_ra_dec_Cov`        | `float`   | `deg**2`   | `stat.covariance;pos.eq.ra;pos.eq.dec;meta.main` | Covariance between fiducial ICRS Right Ascension (RA\*cos(Dec)) and Declination of centroid                                                                                         |
| `coord_flag`              | `boolean` | `—`        | `—`                                              | General Failure Flag. Reference band.                                                                                                                                               |
| `refBand`                 | `char`    | `—`        | `meta.id;instr.bandpass`                         | Reference band - parameters measured on coadds of this band were used for multi-band forced photometry                                                                              |
| `refExtendedness`         | `float`   | `—`        | `—`                                              | Flux-ratio measure of whether an object is point-like (0) or extended (1). Reference band.                                                                                          |
| `refSizeExtendedness`     | `float`   | `—`        | `—`                                              | Moments-based comparison of source size to the local PSF to estimate whether an object is point-like (0) or extended (1). Measured in the reference band; values range from 0 to 1. |
| `griz_model_extendedness` | `float`   | `—`        | `—`                                              | Sersic model flux- and size-based estimate of whether the object is extended (resolved) or not, from 0 (compact) to 1 (extended). Uses griz-band fluxes; values range from 0 to 1.  |
| `u_ra`                    | `double`  | `deg`      | `pos.eq.ra`                                      | Position in right ascension, measured on u-band.                                                                                                                                    |
| `u_dec`                   | `double`  | `deg`      | `pos.eq.dec`                                     | Position in declination, measured on u-band.                                                                                                                                        |
| `u_raErr`                 | `float`   | `deg`      | `stat.error;pos.eq.ra`                           | Error in right ascension, measured on u-band.                                                                                                                                       |
| `u_decErr`                | `float`   | `deg`      | `stat.error;pos.eq.dec`                          | Error in declination, measured on u-band.                                                                                                                                           |
| `u_ra_dec_Cov`            | `float`   | `deg**2`   | `stat.covariance;pos.eq.ra;pos.eq.dec`           | Covariance between right ascension and declination, measured on u-band.                                                                                                             |
| `u_psfFlux`               | `float`   | `nJy`      | `—`                                              | Flux derived from using the PSF model as a weight function. Forced on u-band.                                                                                                       |
| `u_psfFluxErr`            | `float`   | `nJy`      | `—`                                              | Flux uncertainty derived from using the PSF model as a weight function. Forced on u-band.                                                                                           |
| `u_psfMag`                | `float`   | `mag`      | `—`                                              | AB magnitude of the u-band psfFlux.                                                                                                                                                 |
| `u_psfMagErr`             | `float`   | `mag`      | `—`                                              | Uncertainty in magnitudes of the u-band psfFlux.                                                                                                                                    |
| `u_free_psfFlux`          | `float`   | `nJy`      | `—`                                              | Flux derived from using the PSF model as a weight function. Measured on u-band.                                                                                                     |
| `u_free_psfFluxErr`       | `float`   | `nJy`      | `—`                                              | Flux uncertainty derived from using the PSF model as a weight function. Measured on u-band.                                                                                         |
| `u_free_psfFlux_flag`     | `boolean` | `—`        | `—`                                              | Flag set if the unforced PSF flux failed in this band for any reason. Measured on u-band.                                                                                           |
| `u_cModel_dev_reff_major` | `float`   | `arcsec`   | `—`                                              | Half-light ellipse of the de Vaucouleurs fit (major axis). Measured on u-band.                                                                                                      |
| `u_cModel_dev_reff_minor` | `float`   | `arcsec`   | `—`                                              | Half-light ellipse of the de Vaucouleurs fit (minor axis). Measured on u-band.                                                                                                      |
| `u_cModel_dev_theta`      | `float`   | `—`        | `—`                                              | Position angle of the de Vaucouleurs fit ellipse. Measured on u-band.                                                                                                               |
| `u_cModel_exp_reff_major` | `float`   | `arcsec`   | `—`                                              | Half-light ellipse of the exponential fit (major axis). Measured on u-band.                                                                                                         |
| `u_cModel_exp_reff_minor` | `float`   | `arcsec`   | `—`                                              | Half-light ellipse of the exponential fit (minor axis). Measured on u-band.                                                                                                         |
| `u_cModel_exp_theta`      | `float`   | `—`        | `—`                                              | Position angle of the exponential fit ellipse. Measured on u-band.                                                                                                                  |
| `u_cModel_chi2`           | `float`   | `—`        | `—`                                              | -ln(likelihood) (chi^2) in CModel fit. Measured on u-band.                                                                                                                          |
| `u_cModel_devFlux`        | `float`   | `nJy`      | `—`                                              | Flux from the de Vaucouleurs fit. Measured on u-band.                                                                                                                               |
| `u_cModel_devFluxErr`     | `float`   | `nJy`      | `—`                                              | Flux uncertainty from the de Vaucouleurs fit. Measured on u-band.                                                                                                                   |
| `u_cModel_expFlux`        | `float`   | `nJy`      | `—`                                              | Flux from the exponential fit. Measured on u-band.                                                                                                                                  |
| `u_cModel_expFluxErr`     | `float`   | `nJy`      | `—`                                              | Flux uncertainty from the exponential fit. Measured on u-band.                                                                                                                      |
| `u_gaapPsfFlux`           | `float`   | `nJy`      | `—`                                              | GAaP Flux with PSF aperture after reconvolving the image to a Gaussian PSF larger than the original by a factor of 1.15. Forced on u-band.                                          |
| `u_gaapPsfFluxErr`        | `float`   | `nJy`      | `—`                                              | GAaP Flux uncertainty with PSF aperture after multiplying the seeing by 1.15. Forced on u-band.                                                                                     |
| `u_gaap0p7Flux`           | `float`   | `nJy`      | `—`                                              | GAaP Flux with 0.7 arcsec aperture after reconvolving the image to a Gaussian PSF larger than the original by a factor of 1.15. Forced on u-band.                                   |
| `u_gaap0p7FluxErr`        | `float`   | `nJy`      | `—`                                              | GAaP Flux uncertainty with 0.7 arcsec aperture after multiplying the seeing by 1.15. Forced on u-band.                                                                              |
| `u_gaap1p0Flux`           | `float`   | `nJy`      | `—`                                              | GAaP Flux with 1.0 arcsec aperture after reconvolving the image to a Gaussian PSF larger than the original by a factor of 1.15. Forced on u-band.                                   |
| `u_gaap1p0FluxErr`        | `float`   | `nJy`      | `—`                                              | GAaP Flux uncertainty with 1.0 arcsec aperture after multiplying the seeing by 1.15. Forced on u-band.                                                                              |
| `u_ixx`                   | `float`   | `pixel**2` | `—`                                              | Gaussian-weighted adaptive moments. Measured on u-band.                                                                                                                             |
| `u_iyy`                   | `float`   | `pixel**2` | `—`                                              | Gaussian-weighted adaptive moments. Measured on u-band.                                                                                                                             |
| `u_ixy`                   | `float`   | `pixel**2` | `—`                                              | Gaussian-weighted adaptive moments. Measured on u-band.                                                                                                                             |
| `u_i_flag`                | `boolean` | `—`        | `—`                                              | General failure flag, set if anything went wrong. Measured on u-band.                                                                                                               |
| `u_ixxPSF`                | `float`   | `pixel**2` | `—`                                              | Gaussian-weighted adaptive moments of the PSF model at the position of this object. Measured on u-band.                                                                             |
| `u_iyyPSF`                | `float`   | `pixel**2` | `—`                                              | Gaussian-weighted adaptive moments of the PSF model at the position of this object. Measured on u-band.                                                                             |
| `u_ixyPSF`                | `float`   | `pixel**2` | `—`                                              | Gaussian-weighted adaptive moments. Measured on u-band.                                                                                                                             |
| `u_iPSF_flag`             | `boolean` | `—`        | `—`                                              | Flag set for any failure in the PSF model moments. Measured on u-band.                                                                                                              |

### Images

No image cutout backends are configured for this survey.

### Spectra

No 1D spectroscopic backends are configured for this survey.

## References and acknowledgement

- Ivezić, Ž. et al. (2019), *LSST: From Science Drivers to Reference Design and Anticipated Data Products*, ApJ, 873, 111 (`doi:10.3847/1538-4357/ab042c`).
- Rubin Observatory Data Preview documentation: [https://dp2.lsst.io/](https://dp2.lsst.io/).
- Rubin Science Platform TAP endpoint: `https://data.lsst.cloud/api/tap`.

This material is based upon work supported in part by the National Science Foundation through Cooperative Agreements AST-1258333 and AST-2241526 and Cooperative Support Agreements AST-1202910 and 2211468 managed by the Association of Universities for Research in Astronomy (AURA), and the Department of Energy under Contract No. DE-AC02-76SF00515 with the SLAC National Accelerator Laboratory managed by Stanford University. Additional Rubin Observatory funding comes from private donations, grants to universities, and in-kind support from LSST-DA Institutional Members. This research uses services or data provided by the Rubin Science Platform at NSF-DOE Vera C. Rubin Observatory, which is jointly funded by the U.S. National Science Foundation and the U.S. Department of Energy, Office of Science.

## Provenance of this page

| Source ID               | Kind       | Service / Endpoint                | Query or URL                                              | Retrieved (UTC)        |
|-------------------------|------------|-----------------------------------|-----------------------------------------------------------|------------------------|
| `lsst_dp2_coaddpatches` | `lsst_tap` | `https://data.lsst.cloud/api/tap` | `https://data.lsst.cloud/api/tap/tables/dp2.CoaddPatches` | `2026-10-18T17:00:09Z` |
| `lsst_dp2_object`       | `lsst_tap` | `https://data.lsst.cloud/api/tap` | `https://data.lsst.cloud/api/tap/tables/dp2.Object`       | `2026-10-18T17:00:08Z` |
<!-- /generated -->
