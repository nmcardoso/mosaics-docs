# UKIDSS DR10 (`ukidss10`)

<!-- generated:survey-page -->

The UKIRT Infrared Deep Sky Survey Data Release 10 (UKIDSS DR10) provides high-resolution near-infrared imaging across the northern and equatorial sky in four wide passbands: $Y, J, H, K$. Acquired with the Wide Field Camera (WFCAM) on the 3.8m United Kingdom Infrared Telescope (UKIRT) on Maunakea, UKIDSS maps the northern sky across multiple sub-surveys including the Large Area Survey (LAS).

In `mosaics`, `ukidss10` operates exclusively as an image cutout provider (`cutout` adapter `sia`). It retrieves near-infrared image cutouts via the live IVOA Simple Image Access (SIA) service hosted by the Wide Field Astronomy Unit (WFAU) at the Royal Observatory, Edinburgh. Photometric catalogue crossmatching is handled by `ukidss11`, from which `ukidss10` borrows its filter definitions.

| Property                 | Value                                             |
|--------------------------|---------------------------------------------------|
| What MOSAICS extracts    | Cutouts: `sia` (y,j,h,k)                          |
| Archives                 | SIA (`wfaudata.roe.ac.uk`), `datalab.noirlab.edu` |
| Bands / wavelength range | borrows passbands from `ukidss11`                 |
| Used by                  | `mosaics-cutout full`                             |

## Configuration

### The shipped file

Source: `defs/surveys/ukidss10.yaml`

```yaml
# UKIDSS DR10 -- cutouts only. The catalogue's UKIDSS photometry is DR11.

survey_slug: ukidss10
name: UKIDSS DR10

# max_mag: 18.2 (max_mag_band: k). As ukidss11 -- 5 sigma, 2 arcsec aperture,
# VEGA; YJHK 20.2/19.6/18.8/18.2.
cutout:
  - adapter: sia          # VERIFIED LIVE 2026-08-25, IVOA registry
    adapter_params:
      service_url: http://wfaudata.roe.ac.uk/ukidssdr10-siap/
      bands: Y,J,H,K        # WFCAM spelling; the columns are y/j/h/k
    bands: [y, j, h, k]

photometric_system:
  pixel_unit: unknown
  pixel_unit_note: WFCAM frames carry a per-image MAGZPT, so no survey-wide constant exists.
  borrow_filters: ukidss11    # the same WFCAM passbands, one release earlier
```

### Schema of this file

| Key present          | Type                   | Required   | Default   | Meaning                                                                                            |
|----------------------|------------------------|------------|-----------|----------------------------------------------------------------------------------------------------|
| `survey_slug`        | str                    | yes        | —         | Canonical lowercase identifier for the survey and default column prefix.                           |
| `name`               | str                    | yes        | —         | Human-readable survey and data-release name used in logs, plans, and reports.                      |
| `cutout`             | list of cutout entries | no         | []        | Image cutout backends (`hips`, `lsv`, `nersc`, `sia`, `vsa`) serving this survey’s passbands.      |
| `photometric_system` | mapping                | no         | None      | Passband SVO filter identifiers, flux/magnitude homogenisation rules, and image pixel calibration. |

### Adapter parameters

- **Cutout `None` (`adapter: sia`, bands `['y', 'j', 'h', 'k']`):** configured `adapter_params`: `service_url='http://wfaudata.roe.ac.uk/ukidssdr10-siap/'`, `bands='Y,J,H,K'`.

### Common changes

- Disable UKIDSS DR10 cutouts in an entrypoint configuration by setting:
  ```yaml
  overrides:
    surveys/ukidss10:
      cutout: []
  ```
- Select specific passbands on the command line using `--bands ukidss10:j,ukidss10:k` or by overriding `bands: [j, k]` in an entrypoint configuration.
- Adjust SIA query parameters: customize query timeouts or maximum search radii under `adapter_params` when querying dense target fields over high-latency connections.

### Commands

```console
$ uv run -m mosaics.cli.cutout full -t test/objects_sample_1k.csv -o cutouts.lance --config defs/cutout.yaml --survey ukidss10 --plan
```

## Data served by the archive

- Cutout-only role and relation to `ukidss11`: `ukidss10` is used solely for image cutouts because the WFAU DR10 SIA service (`http://wfaudata.roe.ac.uk/ukidssdr10-siap/`) is live, responsive, and IVOA compliant. Photometric catalogue crossmatching is handled by `ukidss11` (querying the DR11 Large Area Survey table `ukidss_dr11plus.lassource` at Astro Data Lab).
- Passband notation: Cutouts are extracted in $y, j, h, k$. The WFCAM camera employs the $K$ passband ($2.20\ \mu\text{m}$), which is distinct from the VIRCAM $K_s$ passband ($2.15\ \mu\text{m}$) used in VISTA surveys like `vhs5`.
- Nominal survey depths: 5-sigma point source depths in a 2-arcsec aperture on the Vega system (Lawrence et al. 2007): $Y = 20.2, J = 19.6, H = 18.8, K = 18.2$ mag. The $K$ band (18.2 mag) is the shallowest.
- Photometric zero points and pixel units: WFCAM frames carry individual per-image zero points (`MAGZPT` in FITS headers), meaning no single survey-wide zero point exists. As a result, `pixel_unit` is designated as `unknown`.

### Catalogue: `ukidss_dr11plus.lassource`

Large Area Survey (LAS) : on completion, 4000 sq.deg. to K=18.4.

**Service URL:** `https://datalab.noirlab.edu/tap`

*Showing 100 of 218 total archive columns (all MOSAICS-mapped columns plus primary table fields; full 218-column schema is stored in `docs/_data/sources/ukidss10/catalog_ukidss_las.json`).*

| Column            | Type          | Unit      | UCD                    | Description                                                      |
|-------------------|---------------|-----------|------------------------|------------------------------------------------------------------|
| `ra`              | `adql:DOUBLE` | `degrees` | `pos.eq.ra;meta.main`  | Celestial Right Ascension                                        |
| `dec`             | `adql:DOUBLE` | `degrees` | `pos.eq.dec;meta.main` | Celestial Declination                                            |
| `epoch`           | `adql:DOUBLE` | `years`   | `time_epoch`           | Epoch of position measurement                                    |
| `cx`              | `adql:DOUBLE` | `—`       | `pos_eq_x`             | unit vector of spherical co-ordinates                            |
| `cy`              | `adql:DOUBLE` | `—`       | `pos_eq_y`             | unit vector of spherical co-ordinates                            |
| `cz`              | `adql:DOUBLE` | `—`       | `pos_eq_z`             | unit vector of spherical co-ordinates                            |
| `l`               | `adql:DOUBLE` | `degrees` | `pos_gal_lon`          | Galactic longitude                                               |
| `b`               | `adql:DOUBLE` | `degrees` | `pos_gal_lat`          | Galactic latitude                                                |
| `lambda`          | `adql:DOUBLE` | `degrees` | `pos`                  | SDSS system spherical co-ordinate 1                              |
| `eta`             | `adql:DOUBLE` | `degrees` | `pos`                  | SDSS system spherical co-ordinate 2                              |
| `elon`            | `adql:DOUBLE` | `—`       | `pos.ecliptic.lon`     | Ecliptic longitude                                               |
| `elat`            | `adql:DOUBLE` | `—`       | `pos.ecliptic.lat`     | Ecliptic latitude                                                |
| `sourceid`        | `adql:BIGINT` | `—`       | `meta.id;meta.main`    | UID (unique over entire WSA via programme ID prefix) of this mer |
| `framesetid`      | `adql:BIGINT` | `—`       | `refer_code`           | UID of the set of frames that this merged source comes from      |
| `htmid`           | `adql:BIGINT` | `—`       | `—`                    | Hierarchical Triangular Mesh (HTM) index, 20 deep, for equatoria |
| `priorsec`        | `adql:BIGINT` | `—`       | `—`                    | Seam code for a unique (=0) or duplicated (!=0) source (eg. flag |
| `sigra`           | `adql:REAL`   | `degrees` | `error`                | Uncertainty in RA                                                |
| `sigdec`          | `adql:REAL`   | `degrees` | `error`                | Uncertainty in Dec                                               |
| `mura`            | `adql:REAL`   | `mas/yr`  | `pos_eq_pmra`          | Proper motion in RA direction                                    |
| `mudec`           | `adql:REAL`   | `mas/yr`  | `pos_eq_pmdec`         | Proper motion in Dec direction                                   |
| `sigmura`         | `adql:REAL`   | `mas/yr`  | `error`                | Error on proper motion in RA direction                           |
| `sigmudec`        | `adql:REAL`   | `mas/yr`  | `error`                | Error on proper motion in Dec direction                          |
| `chi2`            | `adql:REAL`   | `—`       | `fit_chi2`             | Chi-squared value of proper motion solution                      |
| `ymjpnt`          | `adql:REAL`   | `mag`     | `phot_color`           | Point source colour Y-J (using aperMag3)                         |
| `ymjpnterr`       | `adql:REAL`   | `mag`     | `error`                | Error on point source colour Y-J                                 |
| `jmhpnt`          | `adql:REAL`   | `mag`     | `phot_color`           | Point source colour J-H (using aperMag3)                         |
| `jmhpnterr`       | `adql:REAL`   | `mag`     | `error`                | Error on point source colour J-H                                 |
| `hmkpnt`          | `adql:REAL`   | `mag`     | `phot_color`           | Point source colour H-K (using aperMag3)                         |
| `hmkpnterr`       | `adql:REAL`   | `mag`     | `error`                | Error on point source colour H-K                                 |
| `j_1mhpnt`        | `adql:REAL`   | `mag`     | `phot_color`           | Point source colour J_1-H (using aperMag3)                       |
| `j_1mhpnterr`     | `adql:REAL`   | `mag`     | `error`                | Error on point source colour J_1-H                               |
| `j_2mhpnt`        | `adql:REAL`   | `mag`     | `phot_color`           | Point source colour J_2-H (using aperMag3)                       |
| `j_2mhpnterr`     | `adql:REAL`   | `mag`     | `error`                | Error on point source colour J_2-H                               |
| `ymj_1pnt`        | `adql:REAL`   | `mag`     | `phot_color`           | Point source colour Y-J_1 (using aperMag3)                       |
| `ymj_1pnterr`     | `adql:REAL`   | `mag`     | `error`                | Error on point source colour Y-J_1                               |
| `ymj_2pnt`        | `adql:REAL`   | `mag`     | `phot_color`           | Point source colour Y-J_2 (using aperMag3)                       |
| `ymj_2pnterr`     | `adql:REAL`   | `mag`     | `error`                | Error on point source colour Y-J_2                               |
| `ymjext`          | `adql:REAL`   | `mag`     | `phot_color`           | Extended source colour Y-J (using aperMag3)                      |
| `ymjexterr`       | `adql:REAL`   | `mag`     | `error`                | Error on extended source colour Y-J                              |
| `jmhext`          | `adql:REAL`   | `mag`     | `phot_color`           | Extended source colour J-H (using aperMag3)                      |
| `jmhexterr`       | `adql:REAL`   | `mag`     | `error`                | Error on extended source colour J-H                              |
| `hmkext`          | `adql:REAL`   | `mag`     | `phot_color`           | Extended source colour H-K (using aperMag3)                      |
| `hmkexterr`       | `adql:REAL`   | `mag`     | `error`                | Error on extended source colour H-K                              |
| `j_1mhext`        | `adql:REAL`   | `mag`     | `phot_color`           | Extended source colour J_1-H (using aperMag3)                    |
| `j_1mhexterr`     | `adql:REAL`   | `mag`     | `error`                | Error on extended source colour J_1-H                            |
| `j_2mhext`        | `adql:REAL`   | `mag`     | `phot_color`           | Extended source colour J_2-H (using aperMag3)                    |
| `j_2mhexterr`     | `adql:REAL`   | `mag`     | `error`                | Error on extended source colour J_2-H                            |
| `ymj_1ext`        | `adql:REAL`   | `mag`     | `phot_color`           | Extended source colour Y-J_1 (using aperMag3)                    |
| `ymj_1exterr`     | `adql:REAL`   | `mag`     | `error`                | Error on extended source colour Y-J_1                            |
| `ymj_2ext`        | `adql:REAL`   | `mag`     | `phot_color`           | Extended source colour Y-J_2 (using aperMag3)                    |
| `ymj_2exterr`     | `adql:REAL`   | `mag`     | `error`                | Error on extended source colour Y-J_2                            |
| `mergedclassstat` | `adql:REAL`   | `—`       | `stat_prop`            | Merged N(0,1) stellarness-of-profile statistic                   |
| `pstar`           | `adql:REAL`   | `—`       | `stat_prop`            | Probability that the source is a star                            |
| `pgalaxy`         | `adql:REAL`   | `—`       | `stat_prop`            | Probability that the source is a galaxy                          |
| `pnoise`          | `adql:REAL`   | `—`       | `stat_prop`            | Probability that the source is noise                             |
| `psaturated`      | `adql:REAL`   | `—`       | `stat_prop`            | Probability that the source is saturated                         |
| `ebv`             | `adql:REAL`   | `—`       | `—`                    | The galactic dust extinction value measured from the Schlegel, F |
| `ay`              | `adql:REAL`   | `mag`     | `—`                    | The galactic extinction correction in the Y band for extragalact |
| `aj`              | `adql:REAL`   | `mag`     | `—`                    | The galactic extinction correction in the J band for extragalact |
| `ah`              | `adql:REAL`   | `mag`     | `—`                    | The galactic extinction correction in the H band for extragalact |
| `ak`              | `adql:REAL`   | `mag`     | `—`                    | The galactic extinction correction in the K band for extragalact |
| `yhallmag`        | `adql:REAL`   | `mag`     | `phot_mag`             | Total point source Y mag                                         |
| `yhallmagerr`     | `adql:REAL`   | `mag`     | `error`                | Error in total point source Y mag                                |
| `ypetromag`       | `adql:REAL`   | `mag`     | `phot_mag`             | Extended source Y mag (Petrosian)                                |
| `ypetromagerr`    | `adql:REAL`   | `mag`     | `error`                | Error in extended source Y mag (Petrosian)                       |
| `ypsfmag`         | `adql:REAL`   | `mag`     | `phot_mag`             | Point source profile-fitted Y mag                                |
| `ypsfmagerr`      | `adql:REAL`   | `mag`     | `error`                | Error in point source profile-fitted Y mag                       |
| `ysermag2d`       | `adql:REAL`   | `mag`     | `phot_mag`             | Extended source Y mag (profile-fitted)                           |
| `ysermag2derr`    | `adql:REAL`   | `mag`     | `error`                | Error in extended source Y mag (profile-fitted)                  |
| `yapermag3`       | `adql:REAL`   | `mag`     | `phot_mag`             | Default point source Y aperture corrected mag (2.0 arcsec apertu |
| `yapermag3err`    | `adql:REAL`   | `mag`     | `error`                | Error in default point source Y mag (2.0 arcsec aperture diamete |
| `yapermag4`       | `adql:REAL`   | `mag`     | `phot_mag`             | Point source Y aperture corrected mag (2.8 arcsec aperture diame |
| `yapermag4err`    | `adql:REAL`   | `mag`     | `error`                | Error in point source Y mag (2.8 arcsec aperture diameter)       |
| `yapermag6`       | `adql:REAL`   | `mag`     | `phot_mag`             | Point source Y aperture corrected mag (5.7 arcsec aperture diame |
| `yapermag6err`    | `adql:REAL`   | `mag`     | `error`                | Error in point source Y mag (5.7 arcsec aperture diameter)       |
| `ygausig`         | `adql:REAL`   | `pixels`  | `morph_param`          | RMS of axes of ellipse fit in Y                                  |
| `yell`            | `adql:REAL`   | `—`       | `phys_ellipticity`     | 1-b/a, where a/b=semi-major/minor axes in Y                      |
| `ypa`             | `adql:REAL`   | `degrees` | `pos_pos-ang`          | ellipse fit celestial orientation in Y                           |
| `yclassstat`      | `adql:REAL`   | `—`       | `stat_prop`            | N(0,1) stellarness-of-profile statistic in Y                     |
| `yxi`             | `adql:REAL`   | `arcsec`  | `pos_eq_ra_off`        | Offset of Y detection from master position (+east/-west)         |
| `yeta`            | `adql:REAL`   | `arcsec`  | `pos_eq_dec_off`       | Offset of Y detection from master position (+north/-south)       |
| `jhallmag`        | `adql:REAL`   | `mag`     | `phot_mag`             | Total point source J mag                                         |
| `jhallmagerr`     | `adql:REAL`   | `mag`     | `error`                | Error in total point source J mag                                |
| `jpetromag`       | `adql:REAL`   | `mag`     | `phot_mag`             | Extended source J mag (Petrosian)                                |
| `jpetromagerr`    | `adql:REAL`   | `mag`     | `error`                | Error in extended source J mag (Petrosian)                       |
| `jpsfmag`         | `adql:REAL`   | `mag`     | `phot_mag`             | Point source profile-fitted J mag                                |
| `jpsfmagerr`      | `adql:REAL`   | `mag`     | `error`                | Error in point source profile-fitted J mag                       |
| `jsermag2d`       | `adql:REAL`   | `mag`     | `phot_mag`             | Extended source J mag (profile-fitted)                           |
| `jsermag2derr`    | `adql:REAL`   | `mag`     | `error`                | Error in extended source J mag (profile-fitted)                  |
| `japermag3`       | `adql:REAL`   | `mag`     | `phot_mag`             | Default point source J aperture corrected mag (2.0 arcsec apertu |
| `japermag3err`    | `adql:REAL`   | `mag`     | `error`                | Error in default point source J mag (2.0 arcsec aperture diamete |
| `japermag4`       | `adql:REAL`   | `mag`     | `phot_mag`             | Point source J aperture corrected mag (2.8 arcsec aperture diame |
| `japermag4err`    | `adql:REAL`   | `mag`     | `error`                | Error in point source J mag (2.8 arcsec aperture diameter)       |
| `japermag6`       | `adql:REAL`   | `mag`     | `phot_mag`             | Point source J aperture corrected mag (5.7 arcsec aperture diame |
| `japermag6err`    | `adql:REAL`   | `mag`     | `error`                | Error in point source J mag (5.7 arcsec aperture diameter)       |
| `jgausig`         | `adql:REAL`   | `pixels`  | `morph_param`          | RMS of axes of ellipse fit in J                                  |
| `jell`            | `adql:REAL`   | `—`       | `phys_ellipticity`     | 1-b/a, where a/b=semi-major/minor axes in J                      |
| `jpa`             | `adql:REAL`   | `degrees` | `pos_pos-ang`          | ellipse fit celestial orientation in J                           |
| `jclassstat`      | `adql:REAL`   | `—`       | `stat_prop`            | N(0,1) stellarness-of-profile statistic in J                     |
| `jxi`             | `adql:REAL`   | `arcsec`  | `pos_eq_ra_off`        | Offset of J detection from master position (+east/-west)         |

### Images

- **Pixel unit:** `unknown` | **AB zero point:** `—` | **Calibration note:** WFCAM frames carry a per-image MAGZPT, so no survey-wide constant exists.
- **Cutout service `None`:** adapter `sia`, bands `['y', 'j', 'h', 'k']`, params `{'service_url': 'http://wfaudata.roe.ac.uk/ukidssdr10-siap/', 'bands': 'Y,J,H,K'}`.

### Spectra

No 1D spectroscopic backends are configured for this survey.

## References and acknowledgement

- Lawrence, A. et al. (2007), *The UKIRT Infrared Deep Sky Survey (UKIDSS)*, *MNRAS*, 379, 1599 (`doi:10.1111/j.1365-2966.2007.12040.x`).
- Casali, M. et al. (2007), *The UKIRT Wide Field Camera (WFCAM)*, *A&A*, 467, 777 (`doi:10.1051/0004-6361:20066514`).
- Hambly, N. C. et al. (2008), *The WFCAM Science Archive*, *MNRAS*, 384, 637 (`doi:10.1111/j.1365-2966.2007.12700.x`).
- Hewett, P. C. et al. (2006), *The UKIRT Infrared Deep Sky Survey ZY JHK Photometric System: Passbands and Synthetic Colours*, *MNRAS*, 367, 454 (`doi:10.1111/j.1365-2966.2005.09996.x`).
- WFAU UKIDSS DR10 SIA endpoint: `http://wfaudata.roe.ac.uk/ukidssdr10-siap/`.

The UKIDSS project is defined in Lawrence et al. (2007). UKIDSS uses the UKIRT Wide Field Camera (WFCAM; Casali et al. 2007) and a photometric system described in Hewett et al. (2006). The pipeline processing and science archive are described in Irwin et al. (2004) and Hambly et al. (2008). This work is based on data obtained from the UKIRT Science Archive, funded by the UK Science and Technology Facilities Council.

## Provenance of this page

| Source ID            | Kind        | Service / Endpoint                           | Query or URL                                                                                                                  | Retrieved (UTC)        |
|----------------------|-------------|----------------------------------------------|-------------------------------------------------------------------------------------------------------------------------------|------------------------|
| `catalog_ukidss_las` | `tap_table` | `https://datalab.noirlab.edu/tap`            | `SELECT column_name, datatype, unit, ucd, description FROM TAP_SCHEMA.columns WHERE table_name = 'ukidss_dr11plus.lassource'` | `2026-10-18T16:39:29Z` |
| `sia_ukidss10`       | `sia`       | `http://wfaudata.roe.ac.uk/ukidssdr10-siap/` | `http://wfaudata.roe.ac.uk/ukidssdr10-siap/`                                                                                  | `2026-10-18T16:39:29Z` |
<!-- /generated -->
