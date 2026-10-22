# UKIDSS LAS DR9 (`ukidss9`)

<!-- generated:survey-page -->

The UKIDSS Data Release 9 (DR9) Large Area Survey (LAS) is an 82,655,526-row near-infrared photometric catalogue covering the northern and equatorial sky across the $Y, J, H, K$ passbands. Observed with the Wide Field Camera (WFCAM) on the 3.8-metre United Kingdom Infrared Telescope (UKIRT) on Maunakea, LAS provides wide-area near-infrared photometry complementing northern optical surveys.

In `mosaics`, `ukidss9` serves as a local photometric catalogue source using the `lsdb` adapter to crossmatch against CDS-hosted HATS partitioned Parquet files (VizieR catalogue `II/319/las9`) with a precomputed 34-arcsec margin cache (`border12_34arcsec`). This allows client-side spatial matching without uploading candidate coordinates to remote services. Extracted columns include aperture-corrected 2.0 arcsec magnitudes (`aper3`), Petrosian magnitudes (`petro`), and total Hall magnitudes (`hall`), alongside profile classification statistics and duplicate flags.

| Property                 | Value                                                                                                                                                                                                                                                               |
|--------------------------|---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| What MOSAICS extracts    | Catalogue: `ukidss9` (photometric, `lsdb`)                                                                                                                                                                                                                          |
| Archives                 | HATS (`vizcat.cds.unistra.fr`)                                                                                                                                                                                                                                      |
| Bands / wavelength range | `y` (`UKIRT/UKIDSS.Y`), `j` (`UKIRT/UKIDSS.J`), `h` (`UKIRT/UKIDSS.H`), `k` (`UKIRT/UKIDSS.K`)                                                                                                                                                                      |
| Used by                  | `mosaics-targets` (crossmatch), `mosaics-crossmatch`                                                                                                                                                                                                                |
| Crossmatch radius        | `1` arcsec. As ukidss11: DR1 median seeing 0.82 arcsec and astrometric rms 50-100 mas (Lawrence et al. 2007, arXiv:astro-ph/0604426, Table 2); no DR9 document was opened. Value set by the author (HG2, 2026-10-19); the instrument-based proposal was 0.5 arcsec. |

## Configuration

### The shipped file

Source: `defs/surveys/ukidss9.yaml`

```yaml
# UKIDSS DR9 Large Area Survey -- photometry.
#
# The targets pipeline's UKIDSS, and a different data release from `ukidss11`.
# DR11PLUS is served by Astro Data Lab and by WFAU and by no HATS server, while
# LAS DR9 is VizieR II/319, which CDS publishes as HATS -- so this survey is
# matched on this machine instead of being uploaded to Data Lab. A different
# release is a different prefix, so nothing downstream can mistake these
# columns for DR11's.
#
# Photometry only: `ukidss11.yaml` keeps the cutouts, which are DR11 imaging.

survey_slug: ukidss9
name: UKIDSS LAS DR9

catalog:
  - name: ukidss9
    type: photometric
    adapter: lsdb
    adapter_params:
      # VizieR II/319 (las9), HATS at CDS: 82,655,526 rows.
      path: https://vizcat.cds.unistra.fr/hats/ukidss_dr9_las/main
      # The collection's default margin, a 34-arcsec halo.
      margin_cache: https://vizcat.cds.unistra.fr/hats/ukidss_dr9_las/border12_34arcsec

# Crossmatch: the radius a table of positions is matched within, and what it rests on.
crossmatch:
  radius: 1 arcsec
  radius_note: >-
    As ukidss11: DR1 median seeing 0.82 arcsec and astrometric rms 50-100 mas (Lawrence et
    al. 2007, arXiv:astro-ph/0604426, Table 2); no DR9 document was opened. Value set by the
    author (HG2, 2026-10-19); the instrument-based proposal was 0.5 arcsec.

photometric_system:
  filters:
    y: UKIRT/UKIDSS.Y
    j: UKIRT/UKIDSS.J
    h: UKIRT/UKIDSS.H
    k: UKIRT/UKIDSS.K
  vega_offsets:
    # UKIRT/WFCAM (Hewett et al. 2006), as for ukidss11. The -0.080 mag Y
    # zero-point correction introduced at UKIDSS DR3 is already folded into
    # the DR9 magnitudes, so it must NOT be applied again here.
    y: 0.634
    j: 0.938
    h: 1.379
    k: 1.900
  measurements:
    - system: vega_mag
      apertures: [aper3, petro, hall]

# UKIDSS DR9 Large Area Survey source table: near-infrared YJHK photometry
# over the northern UKIDSS footprint.
# Catalogue: VizieR II/319 (las9), HATS at CDS, matched with LSDB. The names
# are VizieR's labels from the II/319 ReadMe; each comment names the WFAU
# column it relabels.
#
# Magnitudes are VEGA, on the WFCAM system. WFCAM's K is not VIRCAM's Ks
# (vhs5) -- different passband, different Vega-to-AB offset -- so the band is
# spelled `k` here and `ks` there.
#
# The LAS observes J twice. II/319 publishes both epochs; the first is the one
# carried, since it is the one every J family below has.
#
# Aperture tokens follow the WFAU naming, as in ukidss11: `aper3` is apermag3,
# the 2.0 arcsec aperture-corrected magnitude recommended for point sources;
# `petro` is the Petrosian magnitude; `hall` is the total (Hall) magnitude.
# II/319 publishes no E(B-V), no pSaturated and no priOrSec; `m` is its
# duplicate flag instead.
column_mapping:
  # ID: 'id'                         # UID of this merged detection (sourceID)
  RAJ2000: 'ra'                    # right ascension, J2000, deg, at Epoch (ra)
  DEJ2000: 'dec'                   # declination, J2000, deg, at Epoch (dec)

  Ymag: 'mag_aper3_y'                # default Y magnitude, 2.0 arcsec aperture corrected (yAperMag3), Vega mag
  e_Ymag: 'mag_aper3_y_err'          # error on Ymag (yAperMag3Err), mag
  Jmag1: 'mag_aper3_j'               # J magnitude, first epoch (j_1AperMag3), Vega mag
  e_Jmag1: 'mag_aper3_j_err'         # error on Jmag1 (j_1AperMag3Err), mag
  Hmag: 'mag_aper3_h'                # default H magnitude (hAperMag3), Vega mag
  e_Hmag: 'mag_aper3_h_err'          # error on Hmag (hAperMag3Err), mag
  Kmag: 'mag_aper3_k'                # default K magnitude (kAperMag3), Vega mag
  e_Kmag: 'mag_aper3_k_err'          # error on Kmag (kAperMag3Err), mag

  pYmag: 'mag_petro_y'                # Petrosian Y magnitude (yPetroMag), Vega mag
  e_pYmag: 'mag_petro_y_err'          # error on pYmag (yPetroMagErr), mag
  pJmag1: 'mag_petro_j'               # Petrosian J magnitude, first epoch (j_1PetroMag), Vega mag
  e_pJmag1: 'mag_petro_j_err'         # error on pJmag1 (j_1PetroMagErr), mag
  pHmag: 'mag_petro_h'                # Petrosian H magnitude (hPetroMag), Vega mag
  e_pHmag: 'mag_petro_h_err'          # error on pHmag (hPetroMagErr), mag
  pKmag: 'mag_petro_k'                # Petrosian K magnitude (kPetroMag), Vega mag
  e_pKmag: 'mag_petro_k_err'          # error on pKmag (kPetroMagErr), mag

  Ytot: 'mag_hall_y'                 # total point source Y magnitude (yHallMag), Vega mag
  e_Ytot: 'mag_hall_y_err'           # error on Ytot (yHallMagErr), mag
  Jtot1: 'mag_hall_j'                # total point source J magnitude, first epoch (j_1HallMag), Vega mag
  e_Jtot1: 'mag_hall_j_err'          # error on Jtot1 (j_1HallMagErr), mag
  Htot: 'mag_hall_h'                 # total point source H magnitude (hHallMag), Vega mag
  e_Htot: 'mag_hall_h_err'           # error on Htot (hHallMagErr), mag
  Ktot: 'mag_hall_k'                 # total point source K magnitude (kHallMag), Vega mag
  e_Ktot: 'mag_hall_k_err'           # error on Ktot (kHallMagErr), mag

  cl: 'class'                        # merged class, 1|0|-1|-2|-3 = galaxy|noise|star|probableStar|probableGalaxy (mergedClass)
  mStat: 'class_stat'                # merged profile class statistic (mergedClassStat)
  'p\*': 'prob_star'                  # probability that the source is a star (pStar)
  pG: 'prob_galaxy'                  # probability that the source is a galaxy (pGalaxy)
  pN: 'prob_noise'                   # probability that the source is noise (pNoise)
  m: 'duplicate'                     # 2 marks a duplicated source, 1 a unique one

  Yflags: 'errbits_y'                # WFAU post-processing error bits, Y (yppErrBits)
  Jflags1: 'errbits_j'               # WFAU post-processing error bits, J first epoch (j_1ppErrBits)
  Hflags: 'errbits_h'                # WFAU post-processing error bits, H (hppErrBits)
  Kflags: 'errbits_k'                # WFAU post-processing error bits, K (kppErrBits)
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

**Catalogue `ukidss9` (`adapter: lsdb`):**

| Parameter           | Allowed Type   | Default               | Description                                                                                         |
|---------------------|----------------|-----------------------|-----------------------------------------------------------------------------------------------------|
| `path`              | str            | —                     | HTTP or S3 URI of the HATS catalogue root (may contain `{band}` or `{BAND}` when `per_band: true`). |
| `margin_cache`      | str            | None                  | URI of the HATS pre-computed margin cache table for boundary-safe spatial crossmatching.            |
| `storage_options`   | mapping        | {}                    | Filesystem options forwarded to `upath`/`fsspec` (e.g. `{'anon': True}` for public S3 buckets).     |
| `reshape`           | str            | None                  | Optional post-read reshaping strategy for nested or multi-row HATS tables.                          |
| `per_band`          | bool           | false                 | When `true`, opens one HATS catalogue per band by substituting `{band}`/`{BAND}` in `path`.         |
| `remote_ra_column`  | str            | HATS metadata default | Override for the remote right ascension column name.                                                |
| `remote_dec_column` | str            | HATS metadata default | Override for the remote declination column name.                                                    |

### Common changes

- **Disable catalogue crossmatching**: To disable `ukidss9` catalogue matching in an entrypoint configuration, set `enabled: false`:
  ```yaml
  overrides:
    surveys/ukidss9:
      catalog:
        - name: ukidss9
          enabled: false
  ```
- **Redirect to a local HATS mirror**: If the HATS dataset has been mirrored locally, update the directory path and margin cache:
  ```yaml
  overrides:
    surveys/ukidss9:
      catalog:
        - name: ukidss9
          adapter: lsdb
          adapter_params:
            path: /path/to/local/hats/ukidss_dr9_las/main
            margin_cache: /path/to/local/hats/ukidss_dr9_las/border12_34arcsec
  ```
- **Filter duplicate detections**: To retain only unique sources and exclude duplicate detections in overlapping frames, add a post-hoc filter:
  ```yaml
  overrides:
    surveys/ukidss9:
      catalog_filters:
        unique_only: "ukidss9_duplicate == 1"
  ```

### Commands

```console
$ uv run -m mosaics.crossmatch.cli run --survey surveys/ukidss9 -i test/objects_sample_1k.csv -o ukidss9_matched.parquet
$ uv run -m mosaics.cli.targets --config defs/targets.yaml -o ./targets_run --dec-range 0 1 --plan plan.yaml
```

## Data served by the archive

- **Local HATS vs remote TAP**: `ukidss9` matches against CDS’s HATS partitioning of VizieR catalogue II/319 (82,655,526 rows) via LSDB on the local machine. This is distinct from `ukidss11`, which performs remote TAP queries against Astro Data Lab. The columns are mapped with the prefix `ukidss9_*` so downstream analyses cannot confuse them with DR11 data.
- **Column mapping from VizieR II/319**: Column names in `ukidss9.yaml` map directly from VizieR II/319 ReadMe labels (`Ymag`, `Jmag1`, `Hmag`, `Kmag`, `pYmag`, `Ytot`, `cl`, `m`) to `ukidss9_*` fields.
- **Multi-epoch J band**: The UKIDSS LAS observed the $J$ band twice. VizieR II/319 publishes both epochs (`Jmag1` and `Jmag2`). In `ukidss9.yaml`, the first epoch (`Jmag1`, `pJmag1`, `Jtot1`, `e_Jmag1`, `Jflags1`) is selected because it is uniformly available across all aperture measurement families (`aper3`, `petro`, `hall`).
- **Missing columns relative to WFAU DR11**: VizieR II/319 does not publish galactic extinction $E(B-V)$, `pSaturated`, or seam codes (`priOrSec`). Duplicate status is tracked via the flag `m`, where 1 denotes a unique source and 2 denotes a duplicate.
- **Vega system and WFCAM passbands**: Magnitudes are in the Vega system on the UKIRT WFCAM system. Apply Hewett et al. (2006) offsets to convert to AB: $Y: +0.634$, $J: +0.938$, $H: +1.379$, $K: +1.900$ mag. The $-0.080$ mag $Y$-band zero-point correction from DR3 is already incorporated into DR9. WFCAM’s $K$ band is not VIRCAM’s $K_s$ band.
- **Cutouts**: `ukidss9` is a catalogue-only survey in `mosaics`. Near-infrared image cutouts are handled by `ukidss10` or `ukidss11`.

### Catalogue: `II/319/las9`

the UKIDSS-DR9 Large Area Survey (on a total of 82,655,526 sources)

**Service URL:** `https://vizcat.cds.unistra.fr/hats/ukidss_dr9_las/main`

| Column      | Type     | Unit     | UCD                                    | Description                                                 |
|-------------|----------|----------|----------------------------------------|-------------------------------------------------------------|
| `ULAS`      | `string` | `—`      | `meta.id;meta.main`                    | Name of source in survey (G5)                               |
| `RAJ2000`   | `double` | `deg`    | `pos.eq.ra;meta.main`                  | Right ascension (J2000), at Epoch                           |
| `DEJ2000`   | `double` | `deg`    | `pos.eq.dec;meta.main`                 | Declination (J2000), at Epoch                               |
| `Ymag`      | `float`  | `mag`    | `phot.mag;em.IR.J`                     | ? UKIDSS Y default magnitude (yAperMag3) (G1)               |
| `Jmag1`     | `float`  | `mag`    | `phot.mag;em.IR.J`                     | ? UKIDSS J magnitude in the first epoch (j_1AperMag3) (G1)  |
| `Jmag2`     | `float`  | `mag`    | `phot.mag;em.IR.J`                     | ? UKIDSS J magnitude in the second epoch (j_2AperMag3) (G1) |
| `Hmag`      | `float`  | `mag`    | `phot.mag;em.IR.H`                     | ? UKIDSS H default magnitude (hAperMag3) (G1)               |
| `Kmag`      | `float`  | `mag`    | `phot.mag;em.IR.K`                     | ? UKIDSS K default magnitude (kAperMag3) (G1)               |
| `e_Ymag`    | `float`  | `mag`    | `stat.error;phot.mag`                  | ? Mean error on Ymag (yAperMag3Err)                         |
| `e_Jmag1`   | `float`  | `mag`    | `stat.error;phot.mag`                  | ? Mean error on Jmag1 (j_1AperMag3Err)                      |
| `e_Jmag2`   | `float`  | `mag`    | `stat.error;phot.mag`                  | ? Mean error on Jmag2 (j_2AperMag3Err)                      |
| `e_Hmag`    | `float`  | `mag`    | `stat.error;phot.mag`                  | ? Mean error on Hmag (hAperMag3Err)                         |
| `e_Kmag`    | `float`  | `mag`    | `stat.error;phot.mag`                  | ? Mean error on Kmag (kAperMag3Err)                         |
| `ID`        | `int64`  | `—`      | `—`                                    | —                                                           |
| `m`         | `int8`   | `—`      | `meta.code.multip`                     | [1/2] “2” indicates a duplicated source (G6)                |
| `Epoch`     | `float`  | `yr`     | `time.epoch`                           | Epoch of position measurement (epoch)                       |
| `e_RAJ2000` | `int16`  | `—`      | `—`                                    | —                                                           |
| `e_DEJ2000` | `int16`  | `—`      | `—`                                    | —                                                           |
| `pmRA`      | `float`  | `mas/yr` | `pos.pm;pos.eq.ra`                     | ? Proper motion in RA direction (muRa)                      |
| `pmDE`      | `float`  | `mas/yr` | `pos.pm;pos.eq.dec`                    | ? Proper motion in Dec direction (muDec)                    |
| `e_pmRA`    | `float`  | `—`      | `—`                                    | —                                                           |
| `e_pmDE`    | `float`  | `—`      | `—`                                    | —                                                           |
| `chi2`      | `float`  | `—`      | `—`                                    | —                                                           |
| `nf`        | `int8`   | `—`      | `—`                                    | —                                                           |
| `mStat`     | `float`  | `—`      | `—`                                    | —                                                           |
| `cl`        | `int8`   | `—`      | `meta.code.class;src.class.starGalaxy` | [-3,1] Merged source class (mergedClass) (G2)               |
| `p*`        | `float`  | `—`      | `—`                                    | —                                                           |
| `pG`        | `float`  | `—`      | `—`                                    | —                                                           |
| `pN`        | `float`  | `—`      | `—`                                    | —                                                           |
| `Ytot`      | `float`  | `—`      | `—`                                    | —                                                           |
| `e_Ytot`    | `float`  | `—`      | `—`                                    | —                                                           |
| `pYmag`     | `float`  | `—`      | `—`                                    | —                                                           |
| `e_pYmag`   | `float`  | `—`      | `—`                                    | —                                                           |
| `Ydiam`     | `float`  | `—`      | `—`                                    | —                                                           |
| `Yell`      | `float`  | `—`      | `—`                                    | —                                                           |
| `Ypa`       | `float`  | `—`      | `—`                                    | —                                                           |
| `Ybad`      | `int8`   | `—`      | `—`                                    | —                                                           |
| `Ycl`       | `int8`   | `—`      | `—`                                    | —                                                           |
| `Yflags`    | `int32`  | `—`      | `—`                                    | —                                                           |
| `Jtot1`     | `float`  | `—`      | `—`                                    | —                                                           |
| `e_Jtot1`   | `float`  | `—`      | `—`                                    | —                                                           |
| `pJmag1`    | `float`  | `—`      | `—`                                    | —                                                           |
| `e_pJmag1`  | `float`  | `—`      | `—`                                    | —                                                           |
| `Jdiam1`    | `float`  | `—`      | `—`                                    | —                                                           |
| `Jell1`     | `float`  | `—`      | `—`                                    | —                                                           |
| `Jpa1`      | `float`  | `—`      | `—`                                    | —                                                           |
| `Jbad1`     | `int8`   | `—`      | `—`                                    | —                                                           |
| `Jcl1`      | `int8`   | `—`      | `—`                                    | —                                                           |
| `Jflags1`   | `int32`  | `—`      | `—`                                    | —                                                           |
| `Jdx1`      | `float`  | `—`      | `—`                                    | —                                                           |
| `Jdy1`      | `float`  | `—`      | `—`                                    | —                                                           |
| `Jtot2`     | `float`  | `—`      | `—`                                    | —                                                           |
| `e_Jtot2`   | `float`  | `—`      | `—`                                    | —                                                           |
| `pJmag2`    | `float`  | `—`      | `—`                                    | —                                                           |
| `e_pJmag2`  | `float`  | `—`      | `—`                                    | —                                                           |
| `Jdiam2`    | `float`  | `—`      | `—`                                    | —                                                           |
| `Jell2`     | `float`  | `—`      | `—`                                    | —                                                           |
| `Jpa2`      | `float`  | `—`      | `—`                                    | —                                                           |
| `Jbad2`     | `int8`   | `—`      | `—`                                    | —                                                           |
| `Jcl2`      | `int8`   | `—`      | `—`                                    | —                                                           |
| `Jflags2`   | `int32`  | `—`      | `—`                                    | —                                                           |
| `Jdx2`      | `float`  | `—`      | `—`                                    | —                                                           |
| `Jdy2`      | `float`  | `—`      | `—`                                    | —                                                           |
| `Htot`      | `float`  | `—`      | `—`                                    | —                                                           |
| `e_Htot`    | `float`  | `—`      | `—`                                    | —                                                           |
| `pHmag`     | `float`  | `—`      | `—`                                    | —                                                           |
| `e_pHmag`   | `float`  | `—`      | `—`                                    | —                                                           |
| `Hdiam`     | `float`  | `—`      | `—`                                    | —                                                           |
| `Hell`      | `float`  | `—`      | `—`                                    | —                                                           |
| `Hpa`       | `float`  | `—`      | `—`                                    | —                                                           |
| `Hbad`      | `int8`   | `—`      | `—`                                    | —                                                           |
| `Hcl`       | `int8`   | `—`      | `—`                                    | —                                                           |
| `Hflags`    | `int32`  | `—`      | `—`                                    | —                                                           |
| `Hdx`       | `float`  | `—`      | `—`                                    | —                                                           |
| `Hdy`       | `float`  | `—`      | `—`                                    | —                                                           |
| `Ktot`      | `float`  | `—`      | `—`                                    | —                                                           |
| `e_Ktot`    | `float`  | `—`      | `—`                                    | —                                                           |
| `pKmag`     | `float`  | `—`      | `—`                                    | —                                                           |
| `e_pKmag`   | `float`  | `—`      | `—`                                    | —                                                           |
| `Kdiam`     | `float`  | `—`      | `—`                                    | —                                                           |
| `Kell`      | `float`  | `—`      | `—`                                    | —                                                           |
| `Kpa`       | `float`  | `—`      | `—`                                    | —                                                           |
| `Kbad`      | `int8`   | `—`      | `—`                                    | —                                                           |
| `Kcl`       | `int8`   | `—`      | `—`                                    | —                                                           |
| `Kflags`    | `int32`  | `—`      | `—`                                    | —                                                           |
| `Kdx`       | `float`  | `—`      | `—`                                    | —                                                           |
| `Kdy`       | `float`  | `—`      | `—`                                    | —                                                           |

### Images

No image cutout backends are configured for this survey.

### Spectra

No 1D spectroscopic backends are configured for this survey.

## References and acknowledgement

- Lawrence, A. et al. (2007), *The UKIRT Infrared Deep Sky Survey (UKIDSS)*, *MNRAS*, 379, 1599 (`doi:10.1111/j.1365-2966.2007.12040.x`).
- Hewett, P. C. et al. (2006), *The UKIDSS photometric system for WFCAM*, *MNRAS*, 367, 454 (`doi:10.1111/j.1365-2966.2006.10027.x`).
- Casali, M. et al. (2007), *The UKIRT wide-field camera WFCAM*, *A&A*, 467, 777 (`doi:10.1051/0004-6361:20066514`).
- Hambly, N. C. et al. (2008), *The WFCAM Science Archive*, *MNRAS*, 384, 637 (`doi:10.1111/j.1365-2966.2007.12700.x`).
- VizieR catalogue II/319: [https://vizier.cds.unistra.fr/viz-bin/VizieR?-source=II/319](https://vizier.cds.unistra.fr/viz-bin/VizieR?-source=II/319)
- CDS HATS repository: [https://vizcat.cds.unistra.fr/hats/ukidss_dr9_las/main](https://vizcat.cds.unistra.fr/hats/ukidss_dr9_las/main)

This work is based in part on data obtained as part of the UKIRT Infrared Deep Sky Survey (UKIDSS). The UKIRT is operated by the Joint Astronomy Centre on behalf of the Science and Technology Facilities Council of the U.K. (STFC). This research has made use of the VizieR catalogue access tool, CDS, Strasbourg, France (Ochsenbein et al. 2000, A&AS 143, 23).

## Provenance of this page

| Source ID          | Kind           | Service / Endpoint                                       | Query or URL                                                        | Retrieved (UTC)        |
|--------------------|----------------|----------------------------------------------------------|---------------------------------------------------------------------|------------------------|
| `hats_ukidss9_las` | `hats_catalog` | `https://vizcat.cds.unistra.fr/hats/ukidss_dr9_las/main` | `https://vizcat.cds.unistra.fr/hats/ukidss_dr9_las/main/properties` | `2026-10-18T16:59:43Z` |
<!-- /generated -->
