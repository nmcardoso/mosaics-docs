# Rubin Observatory Data Preview 1 (`lsstdp1`)

<!-- generated:survey-page -->

Rubin Observatory Data Preview 1 (`lsstdp1`) is a release of the Vera C. Rubin Observatory Legacy Survey of Space and Time (LSST): an optical wide-fast-deep survey of the southern sky made with the 8.4-metre Simonyi Survey Telescope at Cerro Pachón, Chile, through six broad bands ($u, g, r, i, z, y$, covering 3200–10800 Å). Early commissioning and validation products are released as Data Previews through the Rubin Science Platform (RSP). Data Preview 1 is the first, on-sky commissioning release.

In MOSAICS the file is a survey file like any other, matched by `mosaics-crossmatch`: the `dp1.Object` table is queried over the authenticated RSP TAP service with only `(key, ra, dec)` sent, and the Object columns the file maps are joined back onto the table. It is left out of `defs/full.yaml` and `defs/targets.yaml`, because the RSP needs an access token and a targets run must not depend on one; a crossmatch job lists it itself (`surveys: [surveys/lsstdp1]`).

| Property                 | Value                                                                                                                                                                                                                                               |
|--------------------------|-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| What MOSAICS extracts    | Catalogue: `lsstdp1` (photometric, `lsst`)                                                                                                                                                                                                          |
| Archives                 | `data.lsst.cloud`                                                                                                                                                                                                                                   |
| Bands / wavelength range | —                                                                                                                                                                                                                                                   |
| Used by                  | `mosaics-crossmatch`                                                                                                                                                                                                                                |
| Crossmatch radius        | `1` arcsec. Median PSF FWHM about 1.14 arcsec across all bands, sharpest images about 0.58 arcsec (Rubin Data Preview 1, arXiv:2603.23786, abstract); reference catalog Gaia DR3. The astrometric precision (Sect. V.3 of that paper) was not read. |

## Configuration

### The shipped file

Source: `defs/surveys/lsstdp1.yaml`

```yaml
# Rubin Observatory Data Preview 1 -- the Object table, crossmatch only.
#
# Not part of defs/full.yaml or defs/targets.yaml (full.yaml `exclude:`s it): the
# RSP needs a token (LSST_TOKEN), and a targets run must not depend on one. Use it
# from a crossmatch job:
#
#     surveys: [surveys/lsstdp1]
#
# Columns are what runs/gissel.sh asked the old LSST command for. Aliases carry no prefix:
# the pipeline adds `lsstdp1_`. A pattern key selects every remote column it
# matches; {1} is the text its wildcard matched (the band).
#
# This release has no sersic_ra_dec_Cov, sersic_reff_major, sersic_reff_minor,
# sersic_theta, sersic_chi2_reduced or deblend_blendNChild, which lsstdp2 maps.
#
# `mosaics-crossmatch moc --survey surveys/lsstdp1 -o lsstdp1-moc.fits` builds the
# footprint a job's `--filter-moc` reads (see `footprint_table`).

survey_slug: lsstdp1
name: Rubin Observatory Data Preview 1

catalog:
  - name: lsstdp1
    type: photometric
    adapter: lsst
    adapter_params:
      release: dp1
      table: dp1.Object
      remote_ra_column: coord_ra
      remote_dec_column: coord_dec
      footprint_table: dp1.CoaddPatches
      # chunk_mib / use_async: the command line's defaults (28 MiB, measured).

column_mapping:
  objectId: id
  coord_ra: ra
  coord_dec: dec
  '[ugrizy]_cModelMag': mag_cmodel_{1}            # -> lsstdp1_mag_cmodel_u ... _y
  '[ugrizy]_cModelMagErr': mag_cmodel_{1}_err
  '[ugrizy]_sersicFlux': flux_sersic_{1}          # nJy
  '[ugrizy]_sersicFluxErr': flux_sersic_{1}_err
  sersic_index: sersic_index
  sersic_indexErr: sersic_index_err
  sersic_ra: sersic_ra
  sersic_dec: sersic_dec
  sersic_raErr: sersic_ra_err
  sersic_decErr: sersic_dec_err
  sersic_reff_x: sersic_reff_x
  sersic_reff_xErr: sersic_reff_x_err
  sersic_reff_y: sersic_reff_y
  sersic_reff_yErr: sersic_reff_y_err
  sersic_rho: sersic_rho
  sersic_rhoErr: sersic_rho_err
  sersic_n_eval_jac: sersic_n_eval_jac
  sersic_n_iter: sersic_n_iter
  sersic_unknown_flag: sersic_unknown_flag
  sersic_no_data_flag: sersic_no_data_flag
  shape_flag: shape_flag
  ebv: ebv
  detect_fromBlend: detect_from_blend
  detect_isIsolated: detect_is_isolated

crossmatch:
  radius: 1 arcsec
  radius_note: >-
    Median PSF FWHM about 1.14 arcsec across all bands, sharpest images about 0.58 arcsec
    (Rubin Data Preview 1, arXiv:2603.23786, abstract); reference catalog Gaia DR3. The
    astrometric precision (Sect. V.3 of that paper) was not read.
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

**Catalogue `lsstdp1` (`adapter: lsst`):**

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
  $ uv run -m mosaics.crossmatch.cli run --survey surveys/lsstdp1 -i test/objects_sample_1k.csv -o lsstdp1_matched.parquet
  ```
- Build the release footprint once and pre-filter crossmatches to skip targets outside Rubin coverage:
  ```console
  $ uv run -m mosaics.crossmatch.cli moc --survey surveys/lsstdp1 -o lsstdp1-moc.fits
  ```

  then name the file under `overrides:` (`crossmatch: {moc: lsstdp1-moc.fits}`) and run with `--filter-moc`.
- Ask for other Object columns (patterns are allowed) from the command line, without editing the file:
  ```console
  $ uv run -m mosaics.crossmatch.cli run --survey surveys/lsstdp1 -i targets.parquet -o out.parquet --columns objectId coord_ra coord_dec '*_cModelMag'
  ```
- Tune upload chunk size and concurrency for the network you are on:
  ```console
  --chunk-mib 1 -j 6 --limit-share 0.85
  ```

### Commands

```console
$ uv run -m mosaics.crossmatch.cli run --survey surveys/lsstdp1 -i test/objects_sample_1k.csv -o lsstdp1_matched.parquet
```

## Data served by the archive

- **RSP token authentication:** Unlike public TAP services that permit anonymous access, Rubin TAP requires an authenticated Rubin Science Platform (RSP) access token, read from `--lsst-token` or the environment (`LSST_TOKEN`, then `LSST_PASSWORD`, `LSST_PASS`). Authentication is transmitted via HTTP Basic with the literal username `x-oauth-basic` and the token as the password. A token is never read from a YAML file.
- **Rate limiting and concurrency budgets:** Rubin enforces per-account rate limits of 1,000 requests per minute and a ceiling of 12 concurrent Qserv queries. The pipeline enforces these bounds locally through a windowed rate limiter, claiming a configurable fraction (default `--limit-share 0.85`, or 850 requests/min and 10 concurrent slots) so that concurrent tools on a shared account are not starved.
- **Upload payload size limits:** Rubin TAP enforces strict inline upload payload limits near 33 MiB. The crossmatch uploads positions in narrow batches containing only `(key, ra, dec)`, budgeted by default to 28 MiB (`--chunk-mib 28`), to maximize the number of target positions per chunk without exceeding the upload ceiling.
- **Footprint filtering with MOC:** Because the Data Previews cover targeted commissioning fields rather than the full sky, crossmatching an all-sky target catalog directly sends hundreds of futile queries. `mosaics-crossmatch moc` builds a Multi-Order Coverage FITS map from the `dp1.CoaddPatches` table (`s_region` polygons at HEALPix order 10, $\sim 3.4'$ resolution); `--filter-moc` excludes out-of-footprint targets locally before upload.
- **Synchronous vs asynchronous execution:** Queries submit synchronously by default for fast turnaround, but switch automatically to asynchronous TAP jobs once payload sizes exceed upload thresholds or when `--use-async` is specified, preventing wall-clock query timeouts.
- **Columns this release lacks:** `sersic_ra_dec_Cov`, `sersic_reff_major`, `sersic_reff_minor`, `sersic_theta`, `sersic_chi2_reduced` and `deblend_blendNChild` are not in `dp1.Object`, so this file does not map them; `lsstdp2` does.

### Catalogue: `dp1.CoaddPatches`

Static information about the subset of tracts and patches from the standard LSST skymap that apply to coadds in these catalogs

**Service URL:** `https://data.lsst.cloud/api/tap`

| Column       | Type     | Unit   | UCD                      | Description                                                             |
|--------------|----------|--------|--------------------------|-------------------------------------------------------------------------|
| `lsst_tract` | `long`   | `—`    | `meta.id;obs.field`      | ID number of the top level, ‘tract’, within the standard LSST skymap    |
| `lsst_patch` | `long`   | `—`    | `meta.id.part;obs.field` | ID number of the second level, ‘patch’, within the standard LSST skymap |
| `s_ra`       | `double` | `deg`  | `pos.eq.ra`              | Central Spatial Position in ICRS; Right ascension                       |
| `s_dec`      | `double` | `deg`  | `pos.eq.dec`             | Central Spatial Position in ICRS; Declination                           |
| `s_region`   | `char`   | `—`    | `pos.outline;obs.field`  | Sky region covered by the coadd (expressed in ICRS frame)               |

### Catalogue: `dp1.Object`

Descriptions of static astronomical objects (or the static aspects of variable and slowly-moving objects) detected and measured on coadds.

**Service URL:** `https://data.lsst.cloud/api/tap`

*Showing 100 of 1296 total archive columns (all MOSAICS-mapped columns plus primary table fields; full 1296-column schema is stored in `docs/_data/sources/lsstdp1/lsst_dp1_object.json`).*

| Column                              | Type      | Unit       | UCD                                              | Description                                                                                                                                                                                                          |
|-------------------------------------|-----------|------------|--------------------------------------------------|----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `coord_dec`                         | `double`  | `deg`      | `pos.eq.dec;meta.main`                           | Fiducial ICRS Declination of centroid used for database indexing                                                                                                                                                     |
| `coord_ra`                          | `double`  | `deg`      | `pos.eq.ra;meta.main`                            | Fiducial ICRS Right Ascension of centroid used for database indexing                                                                                                                                                 |
| `detect_fromBlend`                  | `boolean` | `—`        | `—`                                              | This source is deblended from a parent with more than one child.                                                                                                                                                     |
| `detect_isIsolated`                 | `boolean` | `—`        | `—`                                              | This source is not a part of a blend.                                                                                                                                                                                |
| `ebv`                               | `float`   | `mag`      | `—`                                              | E(B-V) at coord_ra/coord_dec per Schlegel, Finkbeiner & Davis (1998)                                                                                                                                                 |
| `g_cModelMag`                       | `float`   | `mag`      | `—`                                              | AB magnitude of cModelFlux. Forced on g-band.                                                                                                                                                                        |
| `g_cModelMagErr`                    | `float`   | `mag`      | `—`                                              | Uncertainty in magnitudes on cModelFlux. Forced on g-band.                                                                                                                                                           |
| `g_sersicFlux`                      | `float`   | `nJy`      | `—`                                              | g-band flux from the multiband Sersic model fit.                                                                                                                                                                     |
| `g_sersicFluxErr`                   | `float`   | `nJy`      | `—`                                              | Error on the g-band flux from the multiband Sersic model fit.                                                                                                                                                        |
| `i_cModelMag`                       | `float`   | `mag`      | `—`                                              | AB magnitude of cModelFlux. Forced on i-band.                                                                                                                                                                        |
| `i_cModelMagErr`                    | `float`   | `mag`      | `—`                                              | Uncertainty in magnitudes on cModelFlux. Forced on i-band.                                                                                                                                                           |
| `i_sersicFlux`                      | `float`   | `nJy`      | `—`                                              | i-band flux from the multiband Sersic model fit.                                                                                                                                                                     |
| `i_sersicFluxErr`                   | `float`   | `nJy`      | `—`                                              | Error on the i-band flux from the multiband Sersic model fit.                                                                                                                                                        |
| `objectId`                          | `long`    | `—`        | `meta.id;src`                                    | Unique id. Unique ObjectID                                                                                                                                                                                           |
| `r_cModelMag`                       | `float`   | `mag`      | `—`                                              | AB magnitude of cModelFlux. Forced on r-band.                                                                                                                                                                        |
| `r_cModelMagErr`                    | `float`   | `mag`      | `—`                                              | Uncertainty in magnitudes on cModelFlux. Forced on r-band.                                                                                                                                                           |
| `r_sersicFlux`                      | `float`   | `nJy`      | `—`                                              | r-band flux from the multiband Sersic model fit.                                                                                                                                                                     |
| `r_sersicFluxErr`                   | `float`   | `nJy`      | `—`                                              | Error on the r-band flux from the multiband Sersic model fit.                                                                                                                                                        |
| `sersic_dec`                        | `double`  | `deg`      | `—`                                              | Centroid (declination) from the multiband Sersic model fit.                                                                                                                                                          |
| `sersic_decErr`                     | `float`   | `deg`      | `—`                                              | Error on the centroid (declination) from the multiband Sersic model fit.                                                                                                                                             |
| `sersic_index`                      | `float`   | `—`        | `—`                                              | Sersic profile index parameter from the multiband Sersic model fit.                                                                                                                                                  |
| `sersic_indexErr`                   | `float`   | `—`        | `—`                                              | Error on the Sersic profile index parameter from the multiband Sersic model fit.                                                                                                                                     |
| `sersic_n_eval_jac`                 | `int`     | `—`        | `—`                                              | Number of Jacobian evaluations for the multiband Sersic model fit.                                                                                                                                                   |
| `sersic_n_iter`                     | `int`     | `—`        | `—`                                              | Number of iterations in the non-linear fit for the multiband Sersic model.                                                                                                                                           |
| `sersic_no_data_flag`               | `boolean` | `—`        | `—`                                              | Flag set when there is insufficient data to fit for the multiband Sersic model.                                                                                                                                      |
| `sersic_ra`                         | `double`  | `deg`      | `—`                                              | Centroid (right ascension) from the multiband Sersic model fit.                                                                                                                                                      |
| `sersic_raErr`                      | `float`   | `deg`      | `—`                                              | Error on the centroid (right ascension) from the multiband Sersic model fit.                                                                                                                                         |
| `sersic_reff_x`                     | `float`   | `pixel`    | `—`                                              | Effective radius (x-axis) from the multiband Sersic model fit, prior to convolution with the adjusted (see config) PSF value.                                                                                        |
| `sersic_reff_xErr`                  | `float`   | `pixel`    | `—`                                              | Error on the effective radius (x-axis) from the multiband Sersic model fit.                                                                                                                                          |
| `sersic_reff_y`                     | `float`   | `pixel`    | `—`                                              | Effective radius (y-axis) from the multiband Sersic model fit, prior to convolution with the adjusted (see config) PSF value.                                                                                        |
| `sersic_reff_yErr`                  | `float`   | `pixel`    | `—`                                              | Error on the effective radius (y-axis) from the multiband Sersic model fit.                                                                                                                                          |
| `sersic_rho`                        | `float`   | `—`        | `—`                                              | Ellipse rho (correlation coefficient) from the multiband Sersic model fit.                                                                                                                                           |
| `sersic_rhoErr`                     | `float`   | `—`        | `—`                                              | Error on ellipse rho (correlation coefficient) from the multiband Sersic model fit.                                                                                                                                  |
| `sersic_unknown_flag`               | `boolean` | `—`        | `—`                                              | Flag set for failures with an unexpected or unknown cause for the multiband Sersic model.                                                                                                                            |
| `shape_flag`                        | `boolean` | `—`        | `—`                                              | General failure flag, set if anything went wrong. Reference band.                                                                                                                                                    |
| `u_cModelMag`                       | `float`   | `mag`      | `—`                                              | AB magnitude of cModelFlux. Forced on u-band.                                                                                                                                                                        |
| `u_cModelMagErr`                    | `float`   | `mag`      | `—`                                              | Uncertainty in magnitudes on cModelFlux. Forced on u-band.                                                                                                                                                           |
| `u_sersicFlux`                      | `float`   | `nJy`      | `—`                                              | u-band flux from the multiband Sersic model fit.                                                                                                                                                                     |
| `u_sersicFluxErr`                   | `float`   | `nJy`      | `—`                                              | Error on the u-band flux from the multiband Sersic model fit.                                                                                                                                                        |
| `y_cModelMag`                       | `float`   | `mag`      | `—`                                              | AB magnitude of cModelFlux. Forced on y-band.                                                                                                                                                                        |
| `y_cModelMagErr`                    | `float`   | `mag`      | `—`                                              | Uncertainty in magnitudes on cModelFlux. Forced on y-band.                                                                                                                                                           |
| `y_sersicFlux`                      | `float`   | `nJy`      | `—`                                              | y-band flux from the multiband Sersic model fit.                                                                                                                                                                     |
| `y_sersicFluxErr`                   | `float`   | `nJy`      | `—`                                              | Error on the y-band flux from the multiband Sersic model fit.                                                                                                                                                        |
| `z_cModelMag`                       | `float`   | `mag`      | `—`                                              | AB magnitude of cModelFlux. Forced on z-band.                                                                                                                                                                        |
| `z_cModelMagErr`                    | `float`   | `mag`      | `—`                                              | Uncertainty in magnitudes on cModelFlux. Forced on z-band.                                                                                                                                                           |
| `z_sersicFlux`                      | `float`   | `nJy`      | `—`                                              | z-band flux from the multiband Sersic model fit.                                                                                                                                                                     |
| `z_sersicFluxErr`                   | `float`   | `nJy`      | `—`                                              | Error on the z-band flux from the multiband Sersic model fit.                                                                                                                                                        |
| `coord_decErr`                      | `float`   | `deg`      | `stat.error;pos.eq.dec;meta.main`                | Error in fiducial ICRS Declination of centroid                                                                                                                                                                       |
| `coord_ra_dec_Cov`                  | `float`   | `deg**2`   | `stat.covariance;pos.eq.ra;pos.eq.dec;meta.main` | Covariance between fiducial ICRS Right Ascension and Declination of centroid                                                                                                                                         |
| `coord_raErr`                       | `float`   | `deg`      | `stat.error;pos.eq.ra;meta.main`                 | Error in fiducial ICRS Right Ascension of centroid                                                                                                                                                                   |
| `deblend_failed`                    | `boolean` | `—`        | `—`                                              | Deblender failed to deblend this source                                                                                                                                                                              |
| `deblend_incompleteData`            | `boolean` | `—`        | `—`                                              | One or more bands were not deblended due to an inability to model the PSF.                                                                                                                                           |
| `deblend_isolatedParent`            | `boolean` | `—`        | `—`                                              | Deblender skipped this footprint because there was only a single peak                                                                                                                                                |
| `deblend_iterations`                | `int`     | `—`        | `—`                                              | Number of iterations during deblending                                                                                                                                                                               |
| `deblend_logL`                      | `float`   | `—`        | `—`                                              | Log likelihood of the entire blend in scarlet_lite.                                                                                                                                                                  |
| `deblend_masked`                    | `boolean` | `—`        | `—`                                              | Deblender skipped this source because there were too many masked pixels.                                                                                                                                             |
| `deblend_nChild`                    | `int`     | `—`        | `—`                                              | Number of children this object has (defaults to 0)                                                                                                                                                                   |
| `deblend_nPeaks`                    | `int`     | `—`        | `—`                                              | Number of peaks this parent footprint has (even if the deblender failed or skipped this blend)                                                                                                                       |
| `deblend_parentTooBig`              | `boolean` | `—`        | `—`                                              | Deblender skipped this source because the parent footprint was too large.                                                                                                                                            |
| `deblend_peak_center_x`             | `int`     | `pixel`    | `—`                                              | x-coordinate of the peak after source detection                                                                                                                                                                      |
| `deblend_peak_center_y`             | `int`     | `pixel`    | `—`                                              | y-coordinate of the peak after source detection                                                                                                                                                                      |
| `deblend_skipped`                   | `boolean` | `—`        | `—`                                              | Deblender skipped this source                                                                                                                                                                                        |
| `deblend_tooManyPeaks`              | `boolean` | `—`        | `—`                                              | Deblender skipped this source because there were too many peaks in the Footprint.                                                                                                                                    |
| `detect_isDeblendedModelSource`     | `boolean` | `—`        | `—`                                              | True if source has no children and is in the inner region of a coadd patch and is in the inner region of a coadd tract and is not detected in a pseudo-filter (see config.pseudoFilterList) and is a deblended child |
| `footprintArea`                     | `int`     | `pixel`    | `—`                                              | Number of pixels in the sources detection footprint. Reference band.                                                                                                                                                 |
| `g_ap03Flux`                        | `float`   | `nJy`      | `—`                                              | Flux within 3.0-pixel aperture. Forced on g-band.                                                                                                                                                                    |
| `g_ap03FluxErr`                     | `float`   | `nJy`      | `—`                                              | Flux uncertainty within 3.0-pixel aperture. Forced on g-band.                                                                                                                                                        |
| `g_ap03Flux_flag`                   | `boolean` | `—`        | `—`                                              | Flag set for any failure with the 3.0-pixel aperture flux. Forced on g-band.                                                                                                                                         |
| `g_ap06Flux`                        | `float`   | `nJy`      | `—`                                              | Flux within 6.0-pixel aperture. Forced on g-band.                                                                                                                                                                    |
| `g_ap06FluxErr`                     | `float`   | `nJy`      | `—`                                              | Flux uncertainty within 6.0-pixel aperture. Forced on g-band.                                                                                                                                                        |
| `g_ap06Flux_flag`                   | `boolean` | `—`        | `—`                                              | Flag set for any failure with the 6.0-pixel aperture flux. Forced on g-band.                                                                                                                                         |
| `g_ap09Flux`                        | `float`   | `nJy`      | `—`                                              | Flux within 9.0-pixel aperture. Forced on g-band.                                                                                                                                                                    |
| `g_ap09FluxErr`                     | `float`   | `nJy`      | `—`                                              | Flux uncertainty within 9.0-pixel aperture. Forced on g-band.                                                                                                                                                        |
| `g_ap09Flux_flag`                   | `boolean` | `—`        | `—`                                              | Flag set for any failure with the 9.0-pixel aperture flux. Forced on g-band.                                                                                                                                         |
| `g_ap12Flux`                        | `float`   | `nJy`      | `—`                                              | Flux within 12.0-pixel aperture. Forced on g-band.                                                                                                                                                                   |
| `g_ap12FluxErr`                     | `float`   | `nJy`      | `—`                                              | Flux uncertainty within 12.0-pixel aperture. Forced on g-band.                                                                                                                                                       |
| `g_ap12Flux_flag`                   | `boolean` | `—`        | `—`                                              | Flag set for any failure with the 12.0-pixel aperture flux. Forced on g-band.                                                                                                                                        |
| `g_ap17Flux`                        | `float`   | `nJy`      | `—`                                              | Flux within 17.0-pixel aperture. Forced on g-band.                                                                                                                                                                   |
| `g_ap17FluxErr`                     | `float`   | `nJy`      | `—`                                              | Flux uncertainty within 17.0-pixel aperture. Forced on g-band.                                                                                                                                                       |
| `g_ap17Flux_flag`                   | `boolean` | `—`        | `—`                                              | Flag set for any failure with the 17.0-pixel aperture flux. Forced on g-band.                                                                                                                                        |
| `g_ap25Flux`                        | `float`   | `nJy`      | `—`                                              | Flux within 25.0-pixel aperture. Forced on g-band.                                                                                                                                                                   |
| `g_ap25FluxErr`                     | `float`   | `nJy`      | `—`                                              | Flux uncertainty within 25.0-pixel aperture. Forced on g-band.                                                                                                                                                       |
| `g_ap25Flux_flag`                   | `boolean` | `—`        | `—`                                              | Flag set for any failure with the 25.0-pixel aperture flux. Forced on g-band.                                                                                                                                        |
| `g_ap35Flux`                        | `float`   | `nJy`      | `—`                                              | Flux within 35.0-pixel aperture. Forced on g-band.                                                                                                                                                                   |
| `g_ap35FluxErr`                     | `float`   | `nJy`      | `—`                                              | Flux uncertainty within 35.0-pixel aperture. Forced on g-band.                                                                                                                                                       |
| `g_ap35Flux_flag`                   | `boolean` | `—`        | `—`                                              | Flag set for any failure with the 35.0-pixel aperture flux. Forced on g-band.                                                                                                                                        |
| `g_ap50Flux`                        | `float`   | `nJy`      | `—`                                              | Flux within 50.0-pixel aperture. Forced on g-band.                                                                                                                                                                   |
| `g_ap50FluxErr`                     | `float`   | `nJy`      | `—`                                              | Flux uncertainty within 50.0-pixel aperture. Forced on g-band.                                                                                                                                                       |
| `g_ap50Flux_flag`                   | `boolean` | `—`        | `—`                                              | Flag set for any failure with the 50.0-pixel aperture flux. Forced on g-band.                                                                                                                                        |
| `g_ap70Flux`                        | `float`   | `nJy`      | `—`                                              | Flux within 70.0-pixel aperture. Forced on g-band.                                                                                                                                                                   |
| `g_ap70FluxErr`                     | `float`   | `nJy`      | `—`                                              | Flux uncertainty within 70.0-pixel aperture. Forced on g-band.                                                                                                                                                       |
| `g_ap70Flux_flag`                   | `boolean` | `—`        | `—`                                              | Flag set for any failure with the 70.0-pixel aperture flux. Forced on g-band.                                                                                                                                        |
| `g_apFlux_flag`                     | `boolean` | `—`        | `—`                                              | Duplicate of g_ap12Flux_flag.                                                                                                                                                                                        |
| `g_apFlux_flag_apertureTruncated`   | `boolean` | `—`        | `—`                                              | Duplicate of g_ap12Flux_flag_apertureTruncated.                                                                                                                                                                      |
| `g_apFlux_flag_sincCoeffsTruncated` | `boolean` | `—`        | `—`                                              | Duplicate of g_ap12Flux_flag_sincCoeffsTruncated. Measured on g-band.                                                                                                                                                |
| `g_bdChi2`                          | `float`   | `—`        | `—`                                              | -ln(likelihood) (chi^2) in CModel fit. Measured on g-band.                                                                                                                                                           |
| `g_bdE1`                            | `float`   | `pixel**2` | `—`                                              | Flux-weighted average of exponential and de Vaucouleurs ellipticities. Measured on g-band.                                                                                                                           |
| `g_bdE2`                            | `float`   | `pixel**2` | `—`                                              | Flux-weighted average of exponential and de Vaucouleurs ellipticities. Measured on g-band.                                                                                                                           |
| `g_bdFluxB`                         | `float`   | `nJy`      | `—`                                              | Flux from the de Vaucouleurs fit. Measured on g-band.                                                                                                                                                                |
| `g_bdFluxBErr`                      | `float`   | `nJy`      | `—`                                              | Flux uncertainty from the de Vaucouleurs fit. Measured on g-band.                                                                                                                                                    |

### Images

No image cutout backends are configured for this survey.

### Spectra

No 1D spectroscopic backends are configured for this survey.

## References and acknowledgement

- Ivezić, Ž. et al. (2019), *LSST: From Science Drivers to Reference Design and Anticipated Data Products*, ApJ, 873, 111 (`doi:10.3847/1538-4357/ab042c`).
- Rubin Observatory Data Preview documentation: [https://dp1.lsst.io/](https://dp1.lsst.io/).
- Rubin Science Platform TAP endpoint: `https://data.lsst.cloud/api/tap`.

This material is based upon work supported in part by the National Science Foundation through Cooperative Agreements AST-1258333 and AST-2241526 and Cooperative Support Agreements AST-1202910 and 2211468 managed by the Association of Universities for Research in Astronomy (AURA), and the Department of Energy under Contract No. DE-AC02-76SF00515 with the SLAC National Accelerator Laboratory managed by Stanford University. Additional Rubin Observatory funding comes from private donations, grants to universities, and in-kind support from LSST-DA Institutional Members. This research uses services or data provided by the Rubin Science Platform at NSF-DOE Vera C. Rubin Observatory, which is jointly funded by the U.S. National Science Foundation and the U.S. Department of Energy, Office of Science.

## Provenance of this page

| Source ID               | Kind       | Service / Endpoint                | Query or URL                                              | Retrieved (UTC)        |
|-------------------------|------------|-----------------------------------|-----------------------------------------------------------|------------------------|
| `lsst_dp1_coaddpatches` | `lsst_tap` | `https://data.lsst.cloud/api/tap` | `https://data.lsst.cloud/api/tap/tables/dp1.CoaddPatches` | `2026-10-18T17:00:13Z` |
| `lsst_dp1_object`       | `lsst_tap` | `https://data.lsst.cloud/api/tap` | `https://data.lsst.cloud/api/tap/tables/dp1.Object`       | `2026-10-18T17:00:11Z` |
<!-- /generated -->
