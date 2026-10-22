# AllWISE (`wise`)

<!-- generated:survey-page -->

The AllWISE Source Catalog is a comprehensive full-sky mid-infrared astronomical catalogue containing 747,634,026 sources detected by NASA’s Wide-field Infrared Survey Explorer (WISE). It combines data from the 4-band cryogenic, 3-band cryo, and NEOWISE post-cryogenic survey phases in the W1 (3.4 µm), W2 (4.6 µm), W3 (12 µm), and W4 (22 µm) mid-infrared passbands.

In `mosaics`, `wise` serves as a standalone mid-infrared catalogue source, crossmatched through Astro Data Lab’s TAP service against table `allwise.source`. The extraction provides profile-fit (PSF) Vega magnitudes (`wise_mag_w1` through `wise_mag_w4`), photometric errors, signal-to-noise ratios, reduced chi-squared fit metrics, blend flags, and per-band contamination/quality flags across all four WISE bands.

| Property                 | Value                                                                                                                                                                                                                                                                                    |
|--------------------------|------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| What MOSAICS extracts    | Catalogue: `wise` (photometric, `adl-healpix`)                                                                                                                                                                                                                                           |
| Archives                 | `datalab.noirlab.edu`                                                                                                                                                                                                                                                                    |
| Bands / wavelength range | `w1` (`WISE/WISE.W1`), `w2` (`WISE/WISE.W2`), `w3` (`WISE/WISE.W3`), `w4` (`WISE/WISE.W4`)                                                                                                                                                                                               |
| Used by                  | `mosaics-targets` (crossmatch), `mosaics-crossmatch`                                                                                                                                                                                                                                     |
| Crossmatch radius        | `3` arcsec. PSF FWHM 6.1, 6.8, 7.4 and 12.0 arcsec in W1-W4 (AllWISE Explanatory Supplement, I.2.a); positions of high signal-to-noise, unconfused sources accurate to about 50 mas (Sect. II.1). Value set by the author (HG2, 2026-10-19); the instrument-based proposal was 5 arcsec. |

## Configuration

### The shipped file

Source: `defs/surveys/wise.yaml`

```yaml
# AllWISE -- photometry.
#
# The catalogue, as distinct from `unwise`, which is a reprocessing of the same
# imaging into coadds this project takes cutouts from.

survey_slug: wise
name: AllWISE

catalog:
  - name: wise
    type: photometric
    # Astro Data Lab's public TAP, by ranges of the table's `nest4096` index: no login, no
    # upload; the rows that come back are a superset of the matches and the match itself is
    # made here by haversine separation (HG3, 2026-10-20: Data Lab accounts can no longer be
    # authenticated, so the `adl` upload route is unusable). `nest4096` checked in
    # TAP_SCHEMA on 2026-10-20.
    adapter: adl-healpix
    adapter_params:
      table: allwise.source

# Crossmatch: the radius a table of positions is matched within, and what it rests on.
crossmatch:
  radius: 3 arcsec
  radius_note: >-
    PSF FWHM 6.1, 6.8, 7.4 and 12.0 arcsec in W1-W4 (AllWISE Explanatory Supplement, I.2.a);
    positions of high signal-to-noise, unconfused sources accurate to about 50 mas (Sect.
    II.1). Value set by the author (HG2, 2026-10-19); the instrument-based proposal was 5
    arcsec.

photometric_system:
  filters:
    # The same passbands as ls11's W1-W4, but Vega magnitudes rather than AB
    # nanomaggies.
    w1: WISE/WISE.W1
    w2: WISE/WISE.W2
    w3: WISE/WISE.W3
    w4: WISE/WISE.W4
  vega_offsets:
    # The team's own values, recorded in Tractor headers as WISEAB1-4. SVO's
    # zero points imply 2.673/3.313/5.148/6.594 -- uniformly 0.026 mag lower,
    # a different Vega spectrum normalisation rather than a mismatched
    # passband. The team's values are used because they are what the surveys
    # calibrated against, and what ls11's already-AB W1-W4 were converted
    # with.
    w1: 2.699
    w2: 3.339
    w3: 5.174
    w4: 6.620
  measurements:
    - system: vega_mag
      apertures: [default]

# AllWISE source catalogue: mid-infrared W1-W4 profile-fit photometry.
# Table: allwise.source (Astro Data Lab TAP)
# Crossmatched spatially against target_candidates on (ra, dec).
#
# `mpro` is the profile-fit (PSF) magnitude and is the only photometry family
# taken from AllWISE, so the aperture token is omitted: `wise_mag_w1`, not
# `wise_mag_pro_w1`.
#
# Magnitudes are VEGA. This is the standalone AllWISE photometry -- do NOT
# confuse it with ls11's W1-W4, which are unWISE forced photometry already
# converted to AB by the Legacy Survey pipeline. Applying the Vega-to-AB
# offsets to those a second time is the most common error in multi-survey work.
column_mapping:
  designation: 'id'            # sexagesimal source name, the AllWISE identifier
  ra: 'ra'                     # right ascension, J2000, deg
  dec: 'dec'                   # declination, J2000, deg
  # sigra: 'ra_err'            # uncertainty in the right ascension, arcsec
  # sigdec: 'dec_err'          # uncertainty in the declination, arcsec

  w1mpro: 'mag_w1'             # W1 profile-fit magnitude, Vega mag
  w1sigmpro: 'mag_w1_err'      # error on the W1 profile-fit magnitude, mag
  w1snr: 'snr_w1'              # W1 signal-to-noise ratio
  w1rchi2: 'chi2_w1'           # reduced chi-squared of the W1 profile fit
  w2mpro: 'mag_w2'             # W2 profile-fit magnitude, Vega mag
  w2sigmpro: 'mag_w2_err'      # error on the W2 profile-fit magnitude, mag
  w2snr: 'snr_w2'              # W2 signal-to-noise ratio
  w2rchi2: 'chi2_w2'           # reduced chi-squared of the W2 profile fit
  w3mpro: 'mag_w3'             # W3 profile-fit magnitude, Vega mag
  w3sigmpro: 'mag_w3_err'      # error on the W3 profile-fit magnitude, mag
  w3snr: 'snr_w3'              # W3 signal-to-noise ratio
  w3rchi2: 'chi2_w3'           # reduced chi-squared of the W3 profile fit
  w4mpro: 'mag_w4'             # W4 profile-fit magnitude, Vega mag
  w4sigmpro: 'mag_w4_err'      # error on the W4 profile-fit magnitude, mag
  w4snr: 'snr_w4'              # W4 signal-to-noise ratio
  w4rchi2: 'chi2_w4'           # reduced chi-squared of the W4 profile fit

  w1sat: 'sat_w1'              # fraction of W1 pixels that are saturated
  w2sat: 'sat_w2'              # fraction of W2 pixels that are saturated
  w3sat: 'sat_w3'              # fraction of W3 pixels that are saturated
  w4sat: 'sat_w4'              # fraction of W4 pixels that are saturated

  rchi2: 'chi2'                # reduced chi-squared of the joint profile fit
  nb: 'nb'                     # number of blend components fitted
  na: 'na'                     # active deblending flag
  cc_flags: 'cc_flags'         # per-band contamination and confusion flags
  ext_flg: 'ext_flg'           # extended source flag
  var_flg: 'var_flg'           # per-band variability flags
  ph_qual: 'ph_qual'           # per-band photometric quality flags
  moon_lev: 'moon_lev'         # per-band fraction of frames affected by moonlight
  det_bit: 'det_bit'           # bitwise per-band detection flag
```

### Schema of this file

| Key present          | Type                             | Required   | Default   | Meaning                                                                                                                                                                                                                      |
|----------------------|----------------------------------|------------|-----------|------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `survey_slug`        | str                              | yes        | —         | Canonical lowercase identifier for the survey and default column prefix.                                                                                                                                                     |
| `name`               | str                              | yes        | —         | Human-readable survey and data-release name used in logs, plans, and reports.                                                                                                                                                |
| `catalog`            | list of catalog entries          | no         | []        | Remote or local catalogues queried for target extraction or crossmatching.                                                                                                                                                   |
| `crossmatch`         | mapping                          | no         | None      | How a table of positions is matched against this survey. Required for a survey with an `lsdb` catalog; `crossmatch` null removes it, and a survey without it takes no part in `mosaics-crossmatch`.                          |
| `photometric_system` | mapping                          | no         | None      | Passband SVO filter identifiers, flux/magnitude homogenisation rules, and image pixel calibration.                                                                                                                           |
| `column_mapping`     | mapping (remote_column -> alias) | no         | {}        | Projection and column renaming applied when crossmatching against this survey’s photometric catalogue. The loader adds the catalog’s prefix to every alias, or the hosted survey’s for the columns under its `provides` key. |

### Adapter parameters

**Catalogue `wise` (`adapter: adl-healpix`):**

| Parameter              | Allowed Type   | Default                                                                   | Description                                                                                                                                            |
|------------------------|----------------|---------------------------------------------------------------------------|--------------------------------------------------------------------------------------------------------------------------------------------------------|
| `table`                | str            | —                                                                         | Fully qualified table on Astro Data Lab that carries a HEALPix NESTED index column (e.g. `gaia_dr3.gaia_source`, `allwise.source`, `ls_dr11.tractor`). |
| `service`              | str            | `datalab` (the public TAP at `https://datalab.noirlab.edu/tap`, no login) | A full TAP endpoint or a shorthand of the TAP services, for a test archive.                                                                            |
| `remote_ra_column`     | str            | found by UCD (`pos.eq.ra`), then by name                                  | Remote right ascension column name (`mean_fiber_ra` for `desi_dr1.zpix`).                                                                              |
| `remote_dec_column`    | str            | found by UCD (`pos.eq.dec`), then by name                                 | Remote declination column name.                                                                                                                        |
| `index_column`         | str            | `nest4096`                                                                | The table’s HEALPix NESTED index. Its values are checked against the returned positions on the first query of every run.                               |
| `index_order`          | int            | 12 (the order of `nest4096`)                                              | The HEALPix order of `index_column`.                                                                                                                   |
| `max_rows_per_query`   | int            | 50000                                                                     | Most rows one query may return. A query whose result reaches it is split and asked again, never truncated.                                             |
| `max_ranges_per_query` | int            | 50                                                                        | Most index ranges one query may carry. The public TAP times out well above this (probed at 100 and 500).                                               |
| `rows_per_deg2`        | float          | None (counted by a few `COUNT(*)` queries)                                | The table’s mean source density per square degree. When given, the planner sends no counting query.                                                    |

### Common changes

- **Disable catalogue crossmatching**: To omit AllWISE catalogue crossmatching in an entrypoint configuration:
  ```yaml
  overrides:
    surveys/wise:
      catalog:
        - name: wise
          enabled: false
  ```
- **Filter on photometric quality**: To retain only unblended, high signal-to-noise sources with clean photometric quality flags in W1 and W2:
  ```yaml
  overrides:
    surveys/wise:
      catalog_filters:
        clean_w1w2: "wise_ph_qual LIKE 'AA%' AND wise_sat_w1 == 0 AND wise_sat_w2 == 0"
  ```
- **Modify crossmatch parameters**: To adjust spatial crossmatch search radius or join constraints, override `catalog[0].adapter_params` in your entrypoint YAML.

### Commands

```console
$ uv run -m mosaics.crossmatch.cli run --survey surveys/wise -i test/objects_sample_1k.csv -o wise_matched.parquet
$ uv run -m mosaics.cli.targets --config defs/targets.yaml -o ./targets_run --dec-range 0 1 --plan plan.yaml
```

## Data served by the archive

- **Standalone AllWISE vs unWISE forced photometry**: AllWISE provides standalone, profile-fit (PSF) magnitudes in the Vega system derived from WISE coadded atlas images. In contrast, `unwise` forced photometry is delivered via the DESI Legacy Imaging Surveys DR11 pipeline (`provides.unwise` in `defs/surveys/ls11.yaml`) in AB nanomaggies at optical source positions. Researchers must never apply AllWISE Vega-to-AB offsets to `unwise` fluxes, which are already on the AB system ($m_\mathrm{AB} = 22.5 - 2.5 \log_{10}(\mathrm{nanomaggy})$).
- **Photometry family**: Only profile-fit magnitudes (`w*mpro`) are extracted from `allwise.source`; aperture photometry is omitted. Consequently, column aliases omit aperture tokens (e.g. `wise_mag_w1`, `wise_mag_w1_err`).
- **Vega-to-AB zero-point offsets**: Official AllWISE team values (recorded in Tractor headers as WISEAB1–4) are: $W1: +2.699$, $W2: +3.339$, $W3: +5.174$, $W4: +6.620$ mag. These offsets are preferred over theoretical SVO zero points (which differ by a uniform $-0.026$ mag due to Vega spectrum normalisation) because they match the calibration used across legacy survey reductions.
- **Quality and blending indicators**: The catalogue provides essential diagnostics including photometric quality (`wise_ph_qual`), number of blend components (`wise_nb`), active deblending (`wise_na`), contamination/confusion flags (`wise_cc_flags`), and fractional moonlight contamination (`wise_moon_lev`), which should be consulted to filter blended or spurious sources.
- **Cutouts**: `wise` is a catalogue-only survey in `mosaics`. Mid-infrared image cutouts are handled by `unwise` using unblurred NEO11 coadds.

### Catalogue: `allwise.source`

AllWISE Source Table (747,634,026 rows)

**Service URL:** `https://datalab.noirlab.edu/tap`

| Column          | Type            | Unit     | UCD                               | Description                                                      |
|-----------------|-----------------|----------|-----------------------------------|------------------------------------------------------------------|
| `designation`   | `adql:CHAR`     | `—`      | `meta.id`                         | Unique source ID                                                 |
| `ra`            | `adql:DOUBLE`   | `deg`    | `pos.eq.ra;meta.main`             | Right Ascension (ICRS)                                           |
| `dec`           | `adql:DOUBLE`   | `deg`    | `pos.eq.dec;meta.main`            | Declination (ICRS)                                               |
| `htm9`          | `adql:INTEGER`  | `—`      | `—`                               | HTM index (order 9 => ~10 arcmin size)                           |
| `w4cc_map_str`  | `adql:CHAR`     | `—`      | `—`                               | Contamination and confusion string                               |
| `ring256`       | `adql:INTEGER`  | `—`      | `—`                               | HEALPIX index (Nsides 256, Ring scheme => ~14 arcmin size)       |
| `nest4096`      | `adql:INTEGER`  | `—`      | `—`                               | HEALPIX index (Nsides 4096, Nest scheme => ~52 arcsec size       |
| `random_id`     | `adql:REAL`     | `—`      | `—`                               | Random ID in the range 0.0 => 100.0                              |
| `sigra`         | `adql:REAL`     | `arcsec` | `—`                               | One-sigma uncertaintly in RA from non-moving source extraction   |
| `sigdec`        | `adql:REAL`     | `arcsec` | `—`                               | One-sigma uncertaintly in Dec from non-moving source extraction  |
| `sigradec`      | `adql:REAL`     | `arcsec` | `—`                               | The co-sigma of equatorial position uncertainties sigra/sigdec   |
| `glon`          | `adql:DOUBLE`   | `deg`    | `pos.gal.lon`                     | Galactic Longitude                                               |
| `glat`          | `adql:DOUBLE`   | `deg`    | `pos.gal.lat`                     | Galactic Latitude                                                |
| `elon`          | `adql:DOUBLE`   | `deg`    | `pos.ecl.lon`                     | Ecliptic Longitude                                               |
| `elat`          | `adql:DOUBLE`   | `deg`    | `pos.ecl.lat`                     | Ecliptic Latitude                                                |
| `wx`            | `adql:REAL`     | `pix`    | `—`                               | X-pixel coord of source in Atlas image                           |
| `wy`            | `adql:REAL`     | `pix`    | `—`                               | Y-pixel coord of source in Atlas image                           |
| `cntr`          | `adql:BIGINT`   | `—`      | `—`                               | Unique ID number for this object in the AllWISE Catalog/Reject t |
| `source_id`     | `adql:CHAR`     | `—`      | `—`                               | Unique source ID                                                 |
| `coadd_id`      | `adql:CHAR`     | `—`      | `—`                               | Atlas Tile identifier from which source was extracted            |
| `src`           | `adql:INTEGER`  | `—`      | `—`                               | Sequential number of the source extraction in the Atlas Tile     |
| `w1mpro`        | `adql:REAL`     | `mag`    | `phot.mag;em.IR.3-4um`            | W1 magnitude measured with profile-fitting photometry            |
| `w1sigmpro`     | `adql:REAL`     | `mag`    | `—`                               | W1 measurement uncertainty in mag units                          |
| `w1snr`         | `adql:REAL`     | `—`      | `—`                               | W1 S/N ratio                                                     |
| `w1rchi2`       | `adql:REAL`     | `—`      | `—`                               | Reduced chi^2 of the W1 measurement                              |
| `w2mpro`        | `adql:REAL`     | `mag`    | `phot.mag;em.IR.4-8um`            | W2 magnitude measured with profile-fitting photometry            |
| `w2sigmpro`     | `adql:REAL`     | `mag`    | `—`                               | W2 measurement uncertainty in mag units                          |
| `w2snr`         | `adql:REAL`     | `—`      | `—`                               | W2 S/N ratio                                                     |
| `w2rchi2`       | `adql:REAL`     | `—`      | `—`                               | Reduced chi^2 of the W2 measurement                              |
| `w3mpro`        | `adql:REAL`     | `mag`    | `phot.flux.density;em.IR.8-15um`  | W3 magnitude measured with profile-fitting photometry            |
| `w3sigmpro`     | `adql:REAL`     | `mag`    | `—`                               | W3 measurement uncertainty in mag units                          |
| `w3snr`         | `adql:REAL`     | `—`      | `—`                               | W3 S/N ratio                                                     |
| `w3rchi2`       | `adql:REAL`     | `—`      | `—`                               | Reduced chi^2 of the W3 measurement                              |
| `w4mpro`        | `adql:REAL`     | `mag`    | `phot.flux.density;em.IR.15-30um` | W4 magnitude measured with profile-fitting photometry            |
| `w4sigmpro`     | `adql:REAL`     | `mag`    | `—`                               | W4 measurement uncertainty in mag units                          |
| `w4snr`         | `adql:REAL`     | `—`      | `—`                               | W4 S/N ratio                                                     |
| `w4rchi2`       | `adql:REAL`     | `—`      | `—`                               | Reduced chi^2 of the W4 measurement                              |
| `rchi2`         | `adql:REAL`     | `—`      | `—`                               | Combined reduced chi^w in all bands                              |
| `nb`            | `adql:INTEGER`  | `—`      | `meta.number`                     | Number of PSF components used simultaneously in profile-fitting  |
| `na`            | `adql:INTEGER`  | `—`      | `meta.number`                     | Active deblending flag (0 = source not actively deblended, 1 = s |
| `w1sat`         | `adql:REAL`     | `—`      | `instr.saturation`                | Saturated pixel fraction, W1                                     |
| `w2sat`         | `adql:REAL`     | `—`      | `instr.saturation`                | Saturated pixel fraction, W2                                     |
| `w3sat`         | `adql:REAL`     | `—`      | `instr.saturation`                | Saturated pixel fraction, W3                                     |
| `w4sat`         | `adql:REAL`     | `—`      | `instr.saturation`                | Saturated pixel fraction, W4                                     |
| `satnum`        | `adql:CHAR`     | `—`      | `instr.saturation`                | Minimum sample at which saturation occurs in each band           |
| `ra_pm`         | `adql:DOUBLE`   | `deg`    | `—`                               | RA at epoch MJD=55400.0 (2010.5589) from the profile-fitting mea |
| `dec_pm`        | `adql:DOUBLE`   | `deg`    | `—`                               | Declination at epoch MJD=55400.0 (2010.5589) from the profile-fi |
| `sigra_pm`      | `adql:REAL`     | `arcsec` | `—`                               | One-sigma uncertaintly in ra_pm                                  |
| `sigdec_pm`     | `adql:REAL`     | `arcsec` | `—`                               | One-sigma uncertaintly in dec_pm                                 |
| `w4cc_map`      | `adql:INTEGER`  | `—`      | `—`                               | Contamination and confusion map for this source in W4            |
| `sigradec_pm`   | `adql:REAL`     | `arcsec` | `—`                               | The co-sigma of the equatorial position uncertainties from the p |
| `pmra`          | `adql:INTEGER`  | `mas/yr` | `—`                               | Apparent motion in RA for this source                            |
| `sigpmra`       | `adql:INTEGER`  | `mas/yr` | `—`                               | Uncertainty in RA motion estimation                              |
| `pmdec`         | `adql:INTEGER`  | `mas/yr` | `—`                               | Apparent motion in Dec for this source                           |
| `sigpmdec`      | `adql:INTEGER`  | `mas/yr` | `—`                               | Uncertainty in Dec motion estimation                             |
| `w1rchi2_pm`    | `adql:REAL`     | `—`      | `—`                               | Reduced chi^2 of W1 including motion estimation                  |
| `w2rchi2_pm`    | `adql:REAL`     | `—`      | `—`                               | Reduced chi^2 of W2 including motion estimation                  |
| `w3rchi2_pm`    | `adql:REAL`     | `—`      | `—`                               | Reduced chi^2 of W3 including motion estimation                  |
| `w4rchi2_pm`    | `adql:REAL`     | `—`      | `—`                               | Reduced chi^2 of W4 including motion estimation                  |
| `rchi2_pm`      | `adql:REAL`     | `—`      | `—`                               | Combined reduced chi^2 in all bands including motion estimation  |
| `pmcode`        | `adql:CHAR`     | `—`      | `—`                               | This is a five character string that encodes information about f |
| `cc_flags`      | `adql:CHAR`     | `—`      | `—`                               | Contamination and confusion flag                                 |
| `rel`           | `adql:CHAR`     | `—`      | `—`                               | Small separation, same-Tile detection flag                       |
| `ext_flg`       | `adql:INTEGER`  | `—`      | `—`                               | Extended source flag                                             |
| `var_flg`       | `adql:CHAR`     | `—`      | `—`                               | Variability flag                                                 |
| `ph_qual`       | `adql:CHAR`     | `—`      | `—`                               | Photometric quality flag                                         |
| `det_bit`       | `adql:INTEGER`  | `—`      | `—`                               | Bit-encoded integer indicating bands in which a source has a w?s |
| `moon_lev`      | `adql:CHAR`     | `—`      | `—`                               | Scattered moonlight contamination flag                           |
| `w1nm`          | `adql:INTEGER`  | `—`      | `—`                               | Integer frame detection count – no. of individual 7.7s exposure  |
| `w1m`           | `adql:INTEGER`  | `—`      | `—`                               | Integer frame coverage – no. of individual 7.2s W1 exposures on  |
| `w2nm`          | `adql:INTEGER`  | `—`      | `—`                               | Integer frame detection count – no. of individual 7.7s exposure  |
| `w2m`           | `adql:INTEGER`  | `—`      | `—`                               | Integer frame coverage – no. of individual 7.2s W2 exposures on  |
| `w3nm`          | `adql:INTEGER`  | `—`      | `—`                               | Integer frame detection count – no. of individual 7.7s exposure  |
| `w3m`           | `adql:INTEGER`  | `—`      | `—`                               | Integer frame coverage – no. of individual 7.2s W3 exposures on  |
| `w4nm`          | `adql:INTEGER`  | `—`      | `—`                               | Integer frame detection count – no. of individual 7.7s exposure  |
| `w4m`           | `adql:INTEGER`  | `—`      | `—`                               | Integer frame coverage – no. of individual 7.2s W4 exposures on  |
| `w1cov`         | `adql:REAL`     | `—`      | `—`                               | Mean pixel coverage in W1                                        |
| `w2cov`         | `adql:REAL`     | `—`      | `—`                               | Mean pixel coverage in W2                                        |
| `w3cov`         | `adql:REAL`     | `—`      | `—`                               | Mean pixel coverage in W3                                        |
| `w4cov`         | `adql:REAL`     | `—`      | `—`                               | Mean pixel coverage in W4                                        |
| `w1cc_map`      | `adql:INTEGER`  | `—`      | `—`                               | Contamination and confusion map for this source in W1            |
| `w1cc_map_str`  | `adql:CHAR`     | `—`      | `—`                               | Contamination and confusion string                               |
| `w2cc_map`      | `adql:INTEGER`  | `—`      | `—`                               | Contamination and confusion map for this source in W2            |
| `w2cc_map_str`  | `adql:CHAR`     | `—`      | `—`                               | Contamination and confusion string                               |
| `w3cc_map`      | `adql:INTEGER`  | `—`      | `—`                               | Contamination and confusion map for this source in W3            |
| `w3cc_map_str`  | `adql:CHAR`     | `—`      | `—`                               | Contamination and confusion string                               |
| `best_use_cntr` | `adql:BIGINT`   | `—`      | `—`                               | Cntr identification value of the source extraction that was sele |
| `ngrp`          | `adql:SMALLINT` | `—`      | `—`                               | Excess number of positionally associated duplicate resolution gr |
| `tmass_key`     | `adql:INTEGER`  | `—`      | `—`                               | 2MASS PSC association. Unique identifier of the closest source i |
| `r_2mass`       | `adql:REAL`     | `—`      | `—`                               | Distance separating the positions of the WISE source and associa |
| `pa_2mass`      | `adql:REAL`     | `—`      | `—`                               | Position angle (degrees E of N) of the vector from the WISE sour |
| `n_2mass`       | `adql:INTEGER`  | `—`      | `—`                               | The number of 2MASS PSC entries found within a 3arcsec radius of |
| `j_m_2mass`     | `adql:REAL`     | `—`      | `—`                               | 2MASS J-band magnitude or magnitude upper limit of the associate |
| `j_msig_2mass`  | `adql:REAL`     | `—`      | `—`                               | 2MASS J-band corrected photometric uncertainty of the associated |
| `h_m_2mass`     | `adql:REAL`     | `—`      | `—`                               | 2MASS H-band magnitude or magnitude upper limit of the associate |
| `h_msig_2mass`  | `adql:REAL`     | `—`      | `—`                               | 2MASS H-band corrected photometric uncertainty of the associated |
| `k_m_2mass`     | `adql:REAL`     | `—`      | `—`                               | 2MASS K-band magnitude or magnitude upper limit of the associate |
| `k_msig_2mass`  | `adql:REAL`     | `—`      | `—`                               | 2MASS K-band corrected photometric uncertainty of the associated |
| `spt_ind`       | `adql:INTEGER`  | `—`      | `—`                               | Level 7 HTM spatial index key                                    |
| `htm20`         | `adql:BIGINT`   | `—`      | `—`                               | Level 20 HTM spatial index key                                   |

### Images

No image cutout backends are configured for this survey.

### Spectra

No 1D spectroscopic backends are configured for this survey.

## References and acknowledgement

- Wright, E. L. et al. (2010), *The Wide-field Infrared Survey Explorer (WISE): Mission Description and Initial On-orbit Performance*, *AJ*, 140, 1868 (`doi:10.1088/0004-6256/140/6/1868`).
- Cutri, R. M. et al. (2013), *Explanatory Supplement to the AllWISE Data Release Products*, Tech. Rep., IPAC/Caltech.
- Mainzer, A. et al. (2011), *Preliminary Results from NEOWISE: An Enhancement to the Wide-field Infrared Survey Explorer*, *ApJ*, 731, 53 (`doi:10.3847/1538-4357/731/1/53`).
- Astro Data Lab AllWISE documentation: [https://datalab.noirlab.edu/](https://datalab.noirlab.edu/)

This publication makes use of data products from the Wide-field Infrared Survey Explorer, which is a joint project of the University of California, Los Angeles, and the Jet Propulsion Laboratory/California Institute of Technology, funded by the National Aeronautics and Space Administration. This research uses services or data provided by the Astro Data Lab at NSF NOIRLab. NOIRLab is operated by the Association of Universities for Research in Astronomy (AURA), Inc. under a cooperative agreement with the National Science Foundation.

## Provenance of this page

| Source ID         | Kind        | Service / Endpoint                | Query or URL                                                                                                       | Retrieved (UTC)        |
|-------------------|-------------|-----------------------------------|--------------------------------------------------------------------------------------------------------------------|------------------------|
| `catalog_allwise` | `tap_table` | `https://datalab.noirlab.edu/tap` | `SELECT column_name, datatype, unit, ucd, description FROM TAP_SCHEMA.columns WHERE table_name = 'allwise.source'` | `2026-10-18T16:42:05Z` |
<!-- /generated -->
