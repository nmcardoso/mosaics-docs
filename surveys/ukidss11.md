# UKIDSS DR11 (`ukidss11`)

<!-- generated:survey-page -->

The UKIRT Infrared Deep Sky Survey (UKIDSS) Data Release 11plus (DR11plus) Large Area Survey (LAS) is a wide-field near-infrared photometric survey covering approximately 4,000 deg² of the northern and equatorial sky in the $Y, J, H, K$ passbands. Observations were conducted using the Wide Field Camera (WFCAM) on the 3.8-metre United Kingdom Infrared Telescope (UKIRT) on Maunakea, reaching 5-sigma point-source depths of $Y = 20.2$, $J = 19.6$, $H = 18.8$, and $K = 18.2$ mag (Vega).

In `mosaics`, `ukidss11` provides near-infrared photometric catalogue crossmatching via Astro Data Lab TAP (`ukidss_dr11plus.lassource`). Extracted measurements include aperture-corrected 2.0 arcsec point-source magnitudes (`aper3`), Petrosian magnitudes (`petro`), and total Hall magnitudes (`hall`) along with photometric errors, morphological classification statistics, duplicate seam flags, and galactic extinction estimates. The survey definition also declares an IVOA Simple Image Access (SIA) cutout adapter for UKIDSS DR11 imaging from WFAU Edinburgh, which is disabled by default in shipped configurations.

| Property                 | Value                                                                                                                                                                                                                                                                                    |
|--------------------------|------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| What MOSAICS extracts    | Catalogue: `ukidss11` (photometric, `adl-healpix`); Cutouts: `sia` (y,j,h,k)                                                                                                                                                                                                             |
| Archives                 | SIA (`wfaudata.roe.ac.uk`), `datalab.noirlab.edu`                                                                                                                                                                                                                                        |
| Bands / wavelength range | `y` (`UKIRT/UKIDSS.Y`), `j` (`UKIRT/UKIDSS.J`), `h` (`UKIRT/UKIDSS.H`), `k` (`UKIRT/UKIDSS.K`)                                                                                                                                                                                           |
| Used by                  | `mosaics-targets` (crossmatch), `mosaics-cutout full`, `mosaics-crossmatch`                                                                                                                                                                                                              |
| Crossmatch radius        | `1` arcsec. DR1 median seeing 0.82 arcsec; astrometric rms 50-100 mas tied to 2MASS (Lawrence et al. 2007, arXiv:astro-ph/0604426, Table 2). These are DR1 values; no DR11 document was opened. Value set by the author (HG2, 2026-10-19); the instrument-based proposal was 0.5 arcsec. |

## Configuration

### The shipped file

Source: `defs/surveys/ukidss11.yaml`

```yaml
# UKIDSS DR11 -- photometry and cutouts.

survey_slug: ukidss11
name: UKIDSS DR11

catalog:
  - name: ukidss11
    type: photometric
    # Astro Data Lab's public TAP, by ranges of the table's `nest4096` index: no login, no
    # upload; the rows that come back are a superset of the matches and the match itself is
    # made here by haversine separation (HG3, 2026-10-20: Data Lab accounts can no longer be
    # authenticated, so the `adl` upload route is unusable). `nest4096` checked in
    # TAP_SCHEMA on 2026-10-20; 300 of 300 sampled rows carry the NESTED order-12 pixel of
    # their position (the column's TAP_SCHEMA description says "Ring scheme", which is wrong).
    adapter: adl-healpix
    adapter_params:
      table: ukidss_dr11plus.lassource

# Crossmatch: the radius a table of positions is matched within, and what it rests on.
crossmatch:
  radius: 1 arcsec
  radius_note: >-
    DR1 median seeing 0.82 arcsec; astrometric rms 50-100 mas tied to 2MASS (Lawrence et al.
    2007, arXiv:astro-ph/0604426, Table 2). These are DR1 values; no DR11 document was
    opened. Value set by the author (HG2, 2026-10-19); the instrument-based proposal was 0.5
    arcsec.

catalog_filters:
  ukidss11: >-
    ukidss11_mag_aper3_k <= 18.2
    OR (ukidss11_mag_aper3_k IS NULL AND ukidss11_mag_aper3_h <= 18.8)
    OR (ukidss11_mag_aper3_k IS NULL AND ukidss11_mag_aper3_h IS NULL AND ukidss11_mag_aper3_j <= 19.6)
  ukidss11_note: >-
    5 sigma point source in a 2 arcsec aperture, VEGA on the WFCAM system,
    to match the photometry above (Lawrence et al. 2007, LAS): YJHK
    20.2/19.6/18.8/18.2. K is the shallowest, and is WFCAM's K rather than
    VIRCAM's Ks -- a different passband with a different Vega offset.

cutout:
  - adapter: sia          # VERIFIED LIVE 2026-08-25, IVOA registry
    adapter_params:
      service_url: http://wfaudata.roe.ac.uk/ukidssdr11-siap/
      bands: Y,J,H,K        # WFCAM spelling; the columns are y/j/h/k
    bands: [y, j, h, k]
    enabled: false

photometric_system:
  filters:
    y: UKIRT/UKIDSS.Y
    j: UKIRT/UKIDSS.J
    h: UKIRT/UKIDSS.H
    k: UKIRT/UKIDSS.K
  vega_offsets:
    # UKIRT/WFCAM (Hewett et al. 2006). The -0.080 mag Y zero-point
    # correction introduced at UKIDSS DR3 is already folded into the released
    # DR11 magnitudes, so it must NOT be applied again here.
    y: 0.634
    j: 0.938
    h: 1.379
    k: 1.900
  measurements:
    - system: vega_mag
      apertures: [aper3, petro, hall]
  pixel_unit: unknown
  pixel_unit_note: As ukidss10 -- WFCAM frames carry a per-image MAGZPT.

# UKIDSS DR11 Large Area Survey source table: near-infrared YJHK photometry
# over the northern UKIDSS footprint.
# Table: ukidss_dr11plus.lassource (Astro Data Lab TAP)
# Crossmatched spatially against target_candidates on (ra, dec).
#
# Magnitudes are VEGA, on the WFCAM system. WFCAM's K is not VIRCAM's Ks
# (vhs5) -- different passband, different Vega-to-AB offset -- so the band is
# spelled `k` here and `ks` there.
#
# Aperture tokens follow the WFAU naming: `aper3` is apermag3, the 2.0 arcsec
# aperture-corrected magnitude recommended for point sources; `petro` is the
# Petrosian magnitude; `hall` is the total (Hall) magnitude. VHS publishes no
# `hall` family, which is the one schema difference between the two surveys.
column_mapping:
  sourceid: 'id'                   # UID of this merged detection, assigned by the merge algorithm
  ra: 'ra'                         # right ascension, J2000, deg
  dec: 'dec'                       # declination, J2000, deg

  yapermag3: 'mag_aper3_y'                # default point-source Y aperture-corrected mag (2.0 arcsec diam), Vega mag
  yapermag3err: 'mag_aper3_y_err'         # error on default point/extended-source Y mag, mag
  japermag3: 'mag_aper3_j'                # default point-source J aperture-corrected mag (2.0 arcsec diam), Vega mag
  japermag3err: 'mag_aper3_j_err'         # error on default point/extended-source J mag, mag
  hapermag3: 'mag_aper3_h'                # default point-source H aperture-corrected mag (2.0 arcsec diam), Vega mag
  hapermag3err: 'mag_aper3_h_err'         # error on default point/extended-source H mag, mag
  kapermag3: 'mag_aper3_k'                # default point-source K aperture-corrected mag (2.0 arcsec diam), Vega mag
  kapermag3err: 'mag_aper3_k_err'         # error on default point/extended-source K mag, mag

  ypetromag: 'mag_petro_y'                # extended-source Y mag (Petrosian), Vega mag
  ypetromagerr: 'mag_petro_y_err'         # error on ypetromag, mag
  jpetromag: 'mag_petro_j'                # extended-source J mag (Petrosian), Vega mag
  jpetromagerr: 'mag_petro_j_err'         # error on jpetromag, mag
  hpetromag: 'mag_petro_h'                # extended-source H mag (Petrosian), Vega mag
  hpetromagerr: 'mag_petro_h_err'         # error on hpetromag, mag
  kpetromag: 'mag_petro_k'                # extended-source K mag (Petrosian), Vega mag
  kpetromagerr: 'mag_petro_k_err'         # error on kpetromag, mag

  yhallmag: 'mag_hall_y'                  # total (Hall) Y mag, Vega mag
  yhallmagerr: 'mag_hall_y_err'           # error on yhallmag, mag
  jhallmag: 'mag_hall_j'                  # total (Hall) J mag, Vega mag
  jhallmagerr: 'mag_hall_j_err'           # error on jhallmag, mag
  hhallmag: 'mag_hall_h'                  # total (Hall) H mag, Vega mag
  hhallmagerr: 'mag_hall_h_err'           # error on hhallmag, mag
  khallmag: 'mag_hall_k'                  # total (Hall) K mag, Vega mag
  khallmagerr: 'mag_hall_k_err'           # error on khallmag, mag

  mergedclass: 'class'                    # class flag: 1|0|-1|-2|-3|-9 = galaxy|noise|star|probableStar|probableGalaxy|saturated
  mergedclassstat: 'class_stat'           # merged N(0,1) stellarness-of-profile statistic
  pstar: 'p_star'                         # probability that the source is a star
  pgalaxy: 'p_galaxy'                     # probability that the source is a galaxy
  pnoise: 'p_noise'                       # probability that the source is noise
  psaturated: 'p_saturated'               # probability that the source is saturated

  priorsec: 'primary'                    # seam code for a unique (=0) or duplicated (!=0) source
  ebv: 'ebv'                              # galactic dust extinction from Schlegel maps, mag

  ypperrbits: 'errbits_y'         # additional WFAU post-processing error bits, Y
  jpperrbits: 'errbits_j'         # additional WFAU post-processing error bits, J
  hpperrbits: 'errbits_h'         # additional WFAU post-processing error bits, H
  kpperrbits: 'errbits_k'         # additional WFAU post-processing error bits, K
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

**Catalogue `ukidss11` (`adapter: adl-healpix`):**

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
- **Cutout `None` (`adapter: sia`, bands `['y', 'j', 'h', 'k']`):** configured `adapter_params`: `service_url='http://wfaudata.roe.ac.uk/ukidssdr11-siap/'`, `bands='Y,J,H,K'`.

### Common changes

- **Disable catalogue crossmatching**: To omit `ukidss11` catalogue matching in a composite pipeline run, set `enabled: false` on the catalogue entry in your entrypoint YAML:
  ```yaml
  overrides:
    surveys/ukidss11:
      catalog:
        - name: ukidss11
          enabled: false
  ```
- **Relax or disable depth filters**: To alter or remove the default 5-sigma point-source selection filter on aperture magnitudes, override `catalog_filters`:
  ```yaml
  overrides:
    surveys/ukidss11:
      catalog_filters:
        ukidss11: null
  ```
- **Enable WFAU SIA image cutouts**: To retrieve FITS image cutouts directly from WFAU Edinburgh, enable the SIA adapter and specify target passbands:
  ```yaml
  overrides:
    surveys/ukidss11:
      cutout:
        - adapter: sia
          adapter_params:
            service_url: http://wfaudata.roe.ac.uk/ukidssdr11-siap/
            bands: Y,J,H,K
          bands: [y, j, h, k]
          enabled: true
  ```

### Commands

```console
$ uv run -m mosaics.cli.cutout full -t test/objects_sample_1k.csv -o cutouts.lance --config defs/cutout.yaml --survey ukidss11 --plan
$ uv run -m mosaics.crossmatch.cli run --survey surveys/ukidss11 -i test/objects_sample_1k.csv -o ukidss11_matched.parquet
$ uv run -m mosaics.cli.targets --config defs/targets.yaml -o ./targets_run --dec-range 0 1 --plan plan.yaml
```

## Data served by the archive

- **Catalogue archive vs cutouts**: In `mosaics`, `ukidss11` queries the DR11plus LAS source catalogue hosted on the NSF NOIRLab Astro Data Lab TAP service (`ukidss_dr11plus.lassource`). In contrast, `ukidss9` accesses DR9 photometry via CDS HATS Parquet partitions using LSDB, and `ukidss10` defines active SIA cutouts from WFAU Edinburgh.
- **Vega system and WFCAM passbands**: Magnitudes are published in the Vega system on the UKIRT WFCAM photometric system. To convert to AB magnitudes, apply the Hewett et al. (2006) offsets: $Y: +0.634$, $J: +0.938$, $H: +1.379$, $K: +1.900$ mag.
- **WFCAM K vs VIRCAM Ks**: WFCAM’s $K$ filter is physically distinct from VIRCAM’s $K_s$ filter (`vhs5`, `viking`) and 2MASS $K_s$. They have different bandpass profiles and Vega-to-AB offsets (WFCAM $K$ offset is $+1.900$ mag; VIRCAM $K_s$ offset is $+1.839$ mag). In `mosaics`, the passband is designated `k` for UKIDSS and `ks` for VISTA surveys.
- **Y-band zero-point calibration**: The $-0.080$ mag $Y$-band zero-point correction introduced at UKIDSS DR3 is already incorporated into released DR11 magnitudes and must not be reapplied.
- **Image pixel units and zero points**: WFCAM frames carry a per-image zero point (`MAGZPT` in FITS headers); no single survey-wide calibration factor exists, so `pixel_unit` is marked as `unknown`.
- **Aperture naming conventions**: `aper3` corresponds to `y/j/h/kAperMag3` (2.0 arcsec diameter aperture-corrected magnitude for point sources), `petro` corresponds to Petrosian magnitudes, and `hall` represents total (Hall) magnitudes. VHS publishes no `hall` family, which is the primary schema difference between UKIDSS and VHS.
- **Point-source depth cut**: The default catalogue filter requires $K \le 18.2$ mag (or $H \le 18.8$ mag, or $J \le 19.6$ mag if $K$ is null), reflecting 5-sigma point-source completeness in a 2 arcsec aperture (Lawrence et al. 2007).

### Catalogue: `ukidss_dr11plus.lassource`

Large Area Survey (LAS) : on completion, 4000 sq.deg. to K=18.4.

**Service URL:** `https://datalab.noirlab.edu/tap`

*Showing 100 of 218 total archive columns (all MOSAICS-mapped columns plus primary table fields; full 218-column schema is stored in `docs/_data/sources/ukidss11/catalog_ukidss11_las.json`).*

| Column            | Type            | Unit      | UCD                    | Description                                                      |
|-------------------|-----------------|-----------|------------------------|------------------------------------------------------------------|
| `ra`              | `adql:DOUBLE`   | `degrees` | `pos.eq.ra;meta.main`  | Celestial Right Ascension                                        |
| `dec`             | `adql:DOUBLE`   | `degrees` | `pos.eq.dec;meta.main` | Celestial Declination                                            |
| `sourceid`        | `adql:BIGINT`   | `—`       | `meta.id;meta.main`    | UID (unique over entire WSA via programme ID prefix) of this mer |
| `priorsec`        | `adql:BIGINT`   | `—`       | `—`                    | Seam code for a unique (=0) or duplicated (!=0) source (eg. flag |
| `mergedclassstat` | `adql:REAL`     | `—`       | `stat_prop`            | Merged N(0,1) stellarness-of-profile statistic                   |
| `pstar`           | `adql:REAL`     | `—`       | `stat_prop`            | Probability that the source is a star                            |
| `pgalaxy`         | `adql:REAL`     | `—`       | `stat_prop`            | Probability that the source is a galaxy                          |
| `pnoise`          | `adql:REAL`     | `—`       | `stat_prop`            | Probability that the source is noise                             |
| `psaturated`      | `adql:REAL`     | `—`       | `stat_prop`            | Probability that the source is saturated                         |
| `ebv`             | `adql:REAL`     | `—`       | `—`                    | The galactic dust extinction value measured from the Schlegel, F |
| `yhallmag`        | `adql:REAL`     | `mag`     | `phot_mag`             | Total point source Y mag                                         |
| `yhallmagerr`     | `adql:REAL`     | `mag`     | `error`                | Error in total point source Y mag                                |
| `ypetromag`       | `adql:REAL`     | `mag`     | `phot_mag`             | Extended source Y mag (Petrosian)                                |
| `ypetromagerr`    | `adql:REAL`     | `mag`     | `error`                | Error in extended source Y mag (Petrosian)                       |
| `yapermag3`       | `adql:REAL`     | `mag`     | `phot_mag`             | Default point source Y aperture corrected mag (2.0 arcsec apertu |
| `yapermag3err`    | `adql:REAL`     | `mag`     | `error`                | Error in default point source Y mag (2.0 arcsec aperture diamete |
| `jhallmag`        | `adql:REAL`     | `mag`     | `phot_mag`             | Total point source J mag                                         |
| `jhallmagerr`     | `adql:REAL`     | `mag`     | `error`                | Error in total point source J mag                                |
| `jpetromag`       | `adql:REAL`     | `mag`     | `phot_mag`             | Extended source J mag (Petrosian)                                |
| `jpetromagerr`    | `adql:REAL`     | `mag`     | `error`                | Error in extended source J mag (Petrosian)                       |
| `japermag3`       | `adql:REAL`     | `mag`     | `phot_mag`             | Default point source J aperture corrected mag (2.0 arcsec apertu |
| `japermag3err`    | `adql:REAL`     | `mag`     | `error`                | Error in default point source J mag (2.0 arcsec aperture diamete |
| `hhallmag`        | `adql:REAL`     | `mag`     | `phot_mag`             | Total point source H mag                                         |
| `hhallmagerr`     | `adql:REAL`     | `mag`     | `error`                | Error in total point source H mag                                |
| `hpetromag`       | `adql:REAL`     | `mag`     | `phot_mag`             | Extended source H mag (Petrosian)                                |
| `hpetromagerr`    | `adql:REAL`     | `mag`     | `error`                | Error in extended source H mag (Petrosian)                       |
| `hapermag3`       | `adql:REAL`     | `mag`     | `phot_mag`             | Default point source H aperture corrected mag (2.0 arcsec apertu |
| `hapermag3err`    | `adql:REAL`     | `mag`     | `error`                | Error in default point source H mag (2.0 arcsec aperture diamete |
| `khallmag`        | `adql:REAL`     | `mag`     | `phot_mag`             | Total point source K mag                                         |
| `khallmagerr`     | `adql:REAL`     | `mag`     | `error`                | Error in total point source K mag                                |
| `kpetromag`       | `adql:REAL`     | `mag`     | `phot_mag`             | Extended source K mag (Petrosian)                                |
| `kpetromagerr`    | `adql:REAL`     | `mag`     | `error`                | Error in extended source K mag (Petrosian)                       |
| `kapermag3`       | `adql:REAL`     | `mag`     | `phot_mag`             | Default point source K aperture corrected mag (2.0 arcsec apertu |
| `kapermag3err`    | `adql:REAL`     | `mag`     | `error`                | Error in default point source K mag (2.0 arcsec aperture diamete |
| `ypperrbits`      | `adql:INTEGER`  | `—`       | `—`                    | additional WFAU post-processing error bits in Y                  |
| `jpperrbits`      | `adql:INTEGER`  | `—`       | `—`                    | additional WFAU post-processing error bits in J                  |
| `hpperrbits`      | `adql:INTEGER`  | `—`       | `—`                    | additional WFAU post-processing error bits in H                  |
| `kpperrbits`      | `adql:INTEGER`  | `—`       | `—`                    | additional WFAU post-processing error bits in K                  |
| `mergedclass`     | `adql:SMALLINT` | `—`       | `—`                    | Class flag from available measurements (1|0|-1|-2|-3|-9=galaxy|n |
| `epoch`           | `adql:DOUBLE`   | `years`   | `time_epoch`           | Epoch of position measurement                                    |
| `cx`              | `adql:DOUBLE`   | `—`       | `pos_eq_x`             | unit vector of spherical co-ordinates                            |
| `cy`              | `adql:DOUBLE`   | `—`       | `pos_eq_y`             | unit vector of spherical co-ordinates                            |
| `cz`              | `adql:DOUBLE`   | `—`       | `pos_eq_z`             | unit vector of spherical co-ordinates                            |
| `l`               | `adql:DOUBLE`   | `degrees` | `pos_gal_lon`          | Galactic longitude                                               |
| `b`               | `adql:DOUBLE`   | `degrees` | `pos_gal_lat`          | Galactic latitude                                                |
| `lambda`          | `adql:DOUBLE`   | `degrees` | `pos`                  | SDSS system spherical co-ordinate 1                              |
| `eta`             | `adql:DOUBLE`   | `degrees` | `pos`                  | SDSS system spherical co-ordinate 2                              |
| `elon`            | `adql:DOUBLE`   | `—`       | `pos.ecliptic.lon`     | Ecliptic longitude                                               |
| `elat`            | `adql:DOUBLE`   | `—`       | `pos.ecliptic.lat`     | Ecliptic latitude                                                |
| `framesetid`      | `adql:BIGINT`   | `—`       | `refer_code`           | UID of the set of frames that this merged source comes from      |
| `htmid`           | `adql:BIGINT`   | `—`       | `—`                    | Hierarchical Triangular Mesh (HTM) index, 20 deep, for equatoria |
| `sigra`           | `adql:REAL`     | `degrees` | `error`                | Uncertainty in RA                                                |
| `sigdec`          | `adql:REAL`     | `degrees` | `error`                | Uncertainty in Dec                                               |
| `mura`            | `adql:REAL`     | `mas/yr`  | `pos_eq_pmra`          | Proper motion in RA direction                                    |
| `mudec`           | `adql:REAL`     | `mas/yr`  | `pos_eq_pmdec`         | Proper motion in Dec direction                                   |
| `sigmura`         | `adql:REAL`     | `mas/yr`  | `error`                | Error on proper motion in RA direction                           |
| `sigmudec`        | `adql:REAL`     | `mas/yr`  | `error`                | Error on proper motion in Dec direction                          |
| `chi2`            | `adql:REAL`     | `—`       | `fit_chi2`             | Chi-squared value of proper motion solution                      |
| `ymjpnt`          | `adql:REAL`     | `mag`     | `phot_color`           | Point source colour Y-J (using aperMag3)                         |
| `ymjpnterr`       | `adql:REAL`     | `mag`     | `error`                | Error on point source colour Y-J                                 |
| `jmhpnt`          | `adql:REAL`     | `mag`     | `phot_color`           | Point source colour J-H (using aperMag3)                         |
| `jmhpnterr`       | `adql:REAL`     | `mag`     | `error`                | Error on point source colour J-H                                 |
| `hmkpnt`          | `adql:REAL`     | `mag`     | `phot_color`           | Point source colour H-K (using aperMag3)                         |
| `hmkpnterr`       | `adql:REAL`     | `mag`     | `error`                | Error on point source colour H-K                                 |
| `j_1mhpnt`        | `adql:REAL`     | `mag`     | `phot_color`           | Point source colour J_1-H (using aperMag3)                       |
| `j_1mhpnterr`     | `adql:REAL`     | `mag`     | `error`                | Error on point source colour J_1-H                               |
| `j_2mhpnt`        | `adql:REAL`     | `mag`     | `phot_color`           | Point source colour J_2-H (using aperMag3)                       |
| `j_2mhpnterr`     | `adql:REAL`     | `mag`     | `error`                | Error on point source colour J_2-H                               |
| `ymj_1pnt`        | `adql:REAL`     | `mag`     | `phot_color`           | Point source colour Y-J_1 (using aperMag3)                       |
| `ymj_1pnterr`     | `adql:REAL`     | `mag`     | `error`                | Error on point source colour Y-J_1                               |
| `ymj_2pnt`        | `adql:REAL`     | `mag`     | `phot_color`           | Point source colour Y-J_2 (using aperMag3)                       |
| `ymj_2pnterr`     | `adql:REAL`     | `mag`     | `error`                | Error on point source colour Y-J_2                               |
| `ymjext`          | `adql:REAL`     | `mag`     | `phot_color`           | Extended source colour Y-J (using aperMag3)                      |
| `ymjexterr`       | `adql:REAL`     | `mag`     | `error`                | Error on extended source colour Y-J                              |
| `jmhext`          | `adql:REAL`     | `mag`     | `phot_color`           | Extended source colour J-H (using aperMag3)                      |
| `jmhexterr`       | `adql:REAL`     | `mag`     | `error`                | Error on extended source colour J-H                              |
| `hmkext`          | `adql:REAL`     | `mag`     | `phot_color`           | Extended source colour H-K (using aperMag3)                      |
| `hmkexterr`       | `adql:REAL`     | `mag`     | `error`                | Error on extended source colour H-K                              |
| `j_1mhext`        | `adql:REAL`     | `mag`     | `phot_color`           | Extended source colour J_1-H (using aperMag3)                    |
| `j_1mhexterr`     | `adql:REAL`     | `mag`     | `error`                | Error on extended source colour J_1-H                            |
| `j_2mhext`        | `adql:REAL`     | `mag`     | `phot_color`           | Extended source colour J_2-H (using aperMag3)                    |
| `j_2mhexterr`     | `adql:REAL`     | `mag`     | `error`                | Error on extended source colour J_2-H                            |
| `ymj_1ext`        | `adql:REAL`     | `mag`     | `phot_color`           | Extended source colour Y-J_1 (using aperMag3)                    |
| `ymj_1exterr`     | `adql:REAL`     | `mag`     | `error`                | Error on extended source colour Y-J_1                            |
| `ymj_2ext`        | `adql:REAL`     | `mag`     | `phot_color`           | Extended source colour Y-J_2 (using aperMag3)                    |
| `ymj_2exterr`     | `adql:REAL`     | `mag`     | `error`                | Error on extended source colour Y-J_2                            |
| `ay`              | `adql:REAL`     | `mag`     | `—`                    | The galactic extinction correction in the Y band for extragalact |
| `aj`              | `adql:REAL`     | `mag`     | `—`                    | The galactic extinction correction in the J band for extragalact |
| `ah`              | `adql:REAL`     | `mag`     | `—`                    | The galactic extinction correction in the H band for extragalact |
| `ak`              | `adql:REAL`     | `mag`     | `—`                    | The galactic extinction correction in the K band for extragalact |
| `ypsfmag`         | `adql:REAL`     | `mag`     | `phot_mag`             | Point source profile-fitted Y mag                                |
| `ypsfmagerr`      | `adql:REAL`     | `mag`     | `error`                | Error in point source profile-fitted Y mag                       |
| `ysermag2d`       | `adql:REAL`     | `mag`     | `phot_mag`             | Extended source Y mag (profile-fitted)                           |
| `ysermag2derr`    | `adql:REAL`     | `mag`     | `error`                | Error in extended source Y mag (profile-fitted)                  |
| `yapermag4`       | `adql:REAL`     | `mag`     | `phot_mag`             | Point source Y aperture corrected mag (2.8 arcsec aperture diame |
| `yapermag4err`    | `adql:REAL`     | `mag`     | `error`                | Error in point source Y mag (2.8 arcsec aperture diameter)       |
| `yapermag6`       | `adql:REAL`     | `mag`     | `phot_mag`             | Point source Y aperture corrected mag (5.7 arcsec aperture diame |
| `yapermag6err`    | `adql:REAL`     | `mag`     | `error`                | Error in point source Y mag (5.7 arcsec aperture diameter)       |
| `ygausig`         | `adql:REAL`     | `pixels`  | `morph_param`          | RMS of axes of ellipse fit in Y                                  |
| `yell`            | `adql:REAL`     | `—`       | `phys_ellipticity`     | 1-b/a, where a/b=semi-major/minor axes in Y                      |

### Images

- **Pixel unit:** `unknown` | **AB zero point:** `—` | **Calibration note:** As ukidss10 – WFCAM frames carry a per-image MAGZPT.
- **Cutout service `None`:** adapter `sia`, bands `['y', 'j', 'h', 'k']`, params `{'service_url': 'http://wfaudata.roe.ac.uk/ukidssdr11-siap/', 'bands': 'Y,J,H,K'}`.

### Spectra

No 1D spectroscopic backends are configured for this survey.

## References and acknowledgement

- Lawrence, A. et al. (2007), *The UKIRT Infrared Deep Sky Survey (UKIDSS)*, *MNRAS*, 379, 1599 (`doi:10.1111/j.1365-2966.2007.12040.x`).
- Hewett, P. C. et al. (2006), *The UKIDSS photometric system for WFCAM*, *MNRAS*, 367, 454 (`doi:10.1111/j.1365-2966.2006.10027.x`).
- Casali, M. et al. (2007), *The UKIRT wide-field camera WFCAM*, *A&A*, 467, 777 (`doi:10.1051/0004-6361:20066514`).
- Hambly, N. C. et al. (2008), *The WFCAM Science Archive*, *MNRAS*, 384, 637 (`doi:10.1111/j.1365-2966.2007.12700.x`).
- WFAU UKIDSS Archive: [http://wfaudata.roe.ac.uk/ukidssdr11-siap/](http://wfaudata.roe.ac.uk/ukidssdr11-siap/)
- Astro Data Lab UKIDSS DR11plus documentation: [https://datalab.noirlab.edu/](https://datalab.noirlab.edu/)

This work is based in part on data obtained as part of the UKIRT Infrared Deep Sky Survey (UKIDSS). The UKIRT is operated by the Joint Astronomy Centre on behalf of the Science and Technology Facilities Council of the U.K. (STFC). This research uses services or data provided by the Astro Data Lab at NSF NOIRLab. NOIRLab is operated by the Association of Universities for Research in Astronomy (AURA), Inc. under a cooperative agreement with the National Science Foundation.

## Provenance of this page

| Source ID              | Kind        | Service / Endpoint                           | Query or URL                                                                                                                  | Retrieved (UTC)        |
|------------------------|-------------|----------------------------------------------|-------------------------------------------------------------------------------------------------------------------------------|------------------------|
| `catalog_ukidss11_las` | `tap_table` | `https://datalab.noirlab.edu/tap`            | `SELECT column_name, datatype, unit, ucd, description FROM TAP_SCHEMA.columns WHERE table_name = 'ukidss_dr11plus.lassource'` | `2026-10-18T16:39:37Z` |
| `sia_ukidss11`         | `sia`       | `http://wfaudata.roe.ac.uk/ukidssdr11-siap/` | `http://wfaudata.roe.ac.uk/ukidssdr11-siap/`                                                                                  | `2026-10-18T16:39:37Z` |
<!-- /generated -->
