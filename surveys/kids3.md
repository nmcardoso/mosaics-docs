# Kilo-Degree Survey DR3 (`kids3`)

<!-- generated:survey-page -->

The Kilo-Degree Survey Data Release 3 (KiDS DR3) is an optical wide-field imaging survey carried out with the OmegaCAM 32-CCD, 268-megapixel wide-field camera on the VLT Survey Telescope (VST) at ESO’s Paranal Observatory. KiDS DR3 covers 440 square degrees across two wide contiguous strips in the Northern and Southern Galactic caps, imaging in four optical broadbands: $u, g, r, i$.

In MOSAICS, `kids3` provides wide-field optical photometric crossmatch via HATS Parquet tables hosted at `https://vizcat.cds.unistra.fr/hats/kids_dr3/` (VizieR `II/347`). It extracts total `MAG_AUTO` and seeing-matched Gaussian Aperture and PSF (`GAaP`) aperture magnitudes, morphology parameters, Galactic foreground extinction corrections from SFD maps, and Bayesian photometric redshifts (`BPZ`). (Image cutouts for KiDS are handled through a separate DR5 configuration, `kids5`).

| Property                 | Value                                                                                                                                                                                                                                                                                                                        |
|--------------------------|------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| What MOSAICS extracts    | Catalogue: `kids3` (photometric, `lsdb`)                                                                                                                                                                                                                                                                                     |
| Archives                 | HATS (`vizcat.cds.unistra.fr`)                                                                                                                                                                                                                                                                                               |
| Bands / wavelength range | `u` (`Paranal/OmegaCAM.u_SDSS`), `g` (`Paranal/OmegaCAM.g_SDSS`), `r` (`Paranal/OmegaCAM.r_SDSS`), `i` (`Paranal/OmegaCAM.i_SDSS`)                                                                                                                                                                                           |
| Used by                  | `mosaics-targets` (crossmatch), `mosaics-crossmatch`                                                                                                                                                                                                                                                                         |
| Crossmatch radius        | `1` arcsec. Mean PSF FWHM 0.68 arcsec (r) to 1.00 arcsec (u) (de Jong et al. 2017, arXiv:1703.02991, Table 3); the paper states no astrometric precision, only the reference catalogs (SDSS DR12 in the north, 2MASS in the south). Value set by the author (HG2, 2026-10-19); the instrument-based proposal was 0.5 arcsec. |

## Configuration

### The shipped file

Source: `defs/surveys/kids3.yaml`

```yaml
# KiDS DR3 -- photometry. The KiDS DR5 imaging is a separate product, `kids5`.

survey_slug: kids3
name: Kilo-Degree Survey DR3

catalog:
  - name: kids3
    type: photometric
    adapter: lsdb
    adapter_params:
      path: https://vizcat.cds.unistra.fr/hats/kids_dr3/

# Crossmatch: the radius a table of positions is matched within, and what it rests on.
crossmatch:
  radius: 1 arcsec
  radius_note: >-
    Mean PSF FWHM 0.68 arcsec (r) to 1.00 arcsec (u) (de Jong et al. 2017, arXiv:1703.02991,
    Table 3); the paper states no astrometric precision, only the reference catalogs (SDSS
    DR12 in the north, 2MASS in the south). Value set by the author (HG2, 2026-10-19); the
    instrument-based proposal was 0.5 arcsec.

photometric_system:
  filters:
    # OmegaCAM's SDSS-like set, which SVO suffixes `_SDSS` to distinguish
    # from its Johnson filters of the same letter.
    u: Paranal/OmegaCAM.u_SDSS
    g: Paranal/OmegaCAM.g_SDSS
    r: Paranal/OmegaCAM.r_SDSS
    i: Paranal/OmegaCAM.i_SDSS
  measurements:
    - system: ab_mag
      apertures: [default, gaap]

# KiDS-ESO DR3 multi-band source catalogue (de Jong+ 2017, VizieR II/347):
# ugri MAG_AUTO and GAaP colour-optimised photometry, star/galaxy
# separation, morphology, extinction, and a BPZ photometric redshift.
# Catalog: https://vizcat.cds.unistra.fr/hats/kids_dr3/
# ra/dec = RAJ2000/DEJ2000. Selected via LSDB, not turned into SQL.
column_mapping:
  # KiDS: 'id'                       # source identifier, KIDS JHHMMSS.ss+DDMMSS.ss
  RAJ2000: 'ra'                      # right ascension, J2000, deg
  DEJ2000: 'dec'                     # declination, J2000, deg
  mClass: 'star_class'               # star/galaxy separation flag (SG2DPHOT), bits summed: 1/2/4
  S/G: 'class_star'                  # SExtractor CLASS_STAR classifier (0=galaxy, 1=star)
  Ell: 'ellipticity'                 # SExtractor ellipticity
  Rad: 'kron_radius'                 # Kron radius used for MAG_AUTO, pix
  A: 'semi_major_axis'               # linear semi-major axis, pix
  B: 'semi_minor_axis'               # linear semi-minor axis, pix
  PA: 'pa'                           # position angle, deg
  umag: 'mag_u'                      # u-band MAG_AUTO
  e_umag: 'mag_u_err'                # u-band MAG_AUTO error
  gmag: 'mag_g'                      # g-band MAG_AUTO
  e_gmag: 'mag_g_err'                # g-band MAG_AUTO error
  rmag: 'mag_r'                      # r-band MAG_AUTO
  e_rmag: 'mag_r_err'                # r-band MAG_AUTO error
  imag: 'mag_i'                      # i-band MAG_AUTO
  e_imag: 'mag_i_err'                # i-band MAG_AUTO error
  umagGAAP: 'mag_gaap_u'             # u-band GAaP magnitude (colour-optimised, not a total magnitude)
  e_umagGAAP: 'mag_gaap_u_err'       # u-band GAaP magnitude error
  gmagGAAP: 'mag_gaap_g'             # g-band GAaP magnitude (colour-optimised, not a total magnitude)
  e_gmagGAAP: 'mag_gaap_g_err'       # g-band GAaP magnitude error
  rmagGAAP: 'mag_gaap_r'             # r-band GAaP magnitude (colour-optimised, not a total magnitude)
  e_rmagGAAP: 'mag_gaap_r_err'       # r-band GAaP magnitude error
  imagGAAP: 'mag_gaap_i'             # i-band GAaP magnitude (colour-optimised, not a total magnitude)
  e_imagGAAP: 'mag_gaap_i_err'       # i-band GAaP magnitude error
  Au: 'extinction_u'                 # u-band Galactic foreground extinction (Schlegel+ SFD maps), mag
  Ag: 'extinction_g'                 # g-band Galactic foreground extinction (Schlegel+ SFD maps), mag
  Ar: 'extinction_r'                 # r-band Galactic foreground extinction (Schlegel+ SFD maps), mag
  Ai: 'extinction_i'                 # i-band Galactic foreground extinction (Schlegel+ SFD maps), mag
  zbest: 'photo_z'                   # best-fitting BPZ photometric redshift
  Odds: 'photo_z_odds'               # ODDS: reliability of the BPZ photometric redshift
  uflg: 'flag_u'                     # u-band SExtractor extraction flag (0=most reliable)
  gflg: 'flag_g'                     # g-band SExtractor extraction flag (0=most reliable)
  rflg: 'flag_r'                     # r-band SExtractor extraction flag (0=most reliable)
  iflg: 'flag_i'                     # i-band SExtractor extraction flag (0=most reliable)
  Tflg: 'flag_tile'                  # tile quality warning bitmap (1=u, 2=g, 3=r, 4=i affected)
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

**Catalogue `kids3` (`adapter: lsdb`):**

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

- Disable `kids3` catalogue crossmatch in an entrypoint configuration by setting `overrides[surveys/kids3].catalog: []` or setting `enabled: false`.
- Select clean, reliable detections by filtering on the per-band SExtractor extraction flags (`kids3_flag_u`, `kids3_flag_g`, `kids3_flag_r`, `kids3_flag_i == 0`).
- Differentiate between point sources and galaxies using the star/galaxy classifier `kids3_class_star` or the `SG2DPHOT` separation bitmap `kids3_star_class`.

### Commands

```console
$ uv run -m mosaics.crossmatch.cli run --survey surveys/kids3 -i test/objects_sample_1k.csv -o kids3_matched.parquet
$ uv run -m mosaics.cli.targets --config defs/targets.yaml -o ./targets_run --dec-range 0 1 --plan plan.yaml
```

## Data served by the archive

- **Total vs colour-optimised aperture photometry:** The catalogue provides both SExtractor `MAG_AUTO` (`umag`, `gmag`, `rmag`, `imag`) for total source magnitudes and `GAaP` magnitudes (`umagGAAP`, `gmagGAAP`, `rmagGAAP`, `imagGAAP`). GAaP photometry uses Gaussian Aperture and PSF fitting designed specifically to measure accurate colours across differing seeing conditions; GAaP values are colour-optimised and should not be used as total magnitudes.
- **Passband designation:** OmegaCAM uses an SDSS-like optical filter set, designated in the Spanish Virtual Observatory (SVO) filter profile service as `Paranal/OmegaCAM.u_SDSS`, `g_SDSS`, `r_SDSS`, and `i_SDSS` to distinguish them from standard Johnson filters of the same letter.
- **Extinction and photometric redshifts:** Upstream columns carry Galactic foreground extinction values ($A_u, A_g, A_r, A_i$) computed from Schlegel, Finkbeiner & Davis (SFD98) dust maps, along with best-fitting Bayesian photometric redshifts (`kids3_photo_z` from `zbest`) and their associated reliability parameter (`kids3_photo_z_odds` from `Odds`).
- **Tile quality flags:** The `Tflg` column is a bitmask indicating tile-level imaging defects or issues affecting specific bands (bits 1, 2, 3, and 4 denote warnings in $u, g, r, i$ respectively).

### Catalogue: `II/347/kids_dr3`

\*KiDS-ESO-DR3 multi-band source catalog

**Service URL:** `https://vizcat.cds.unistra.fr/hats/kids_dr3/`

| Column       | Type     | Unit   | UCD                    | Description                                                              |
|--------------|----------|--------|------------------------|--------------------------------------------------------------------------|
| `KiDS`       | `string` | `—`    | `meta.id;meta.main`    | Source identifier (KIDS JHHMMSS.ss+DDMMSS.ss) (ID)                       |
| `RAJ2000`    | `double` | `deg`  | `pos.eq.ra;meta.main`  | Right ascension (J2000) (RAJ2000)                                        |
| `DEJ2000`    | `double` | `deg`  | `pos.eq.dec;meta.main` | Declination (J2000) (DECJ2000)                                           |
| `CollID`     | `int32`  | `—`    | `—`                    | —                                                                        |
| `ListID`     | `int32`  | `—`    | `—`                    | —                                                                        |
| `SrcID`      | `int32`  | `—`    | `—`                    | —                                                                        |
| `mClass`     | `int8`   | `—`    | `src.class.starGalaxy` | [0/6] Star/galaxy separation (SG2DPHOT) (1)                              |
| `A`          | `double` | `—`    | `—`                    | —                                                                        |
| `B`          | `double` | `—`    | `—`                    | —                                                                        |
| `S/G`        | `float`  | `—`    | `src.class.starGalaxy` | SExtractor star/galaxy classifier (0=galaxy) (CLASS_STAR)                |
| `Ell`        | `float`  | `—`    | `src.ellipticity`      | Ellipticity (ELLIPTICITY)                                                |
| `Rad`        | `double` | `pix`  | `phys.angSize;src`     | [3/33] Kron-radius used for MAG_AUTO (KRON_RADIUS)                       |
| `PA`         | `double` | `deg`  | `pos.posAng`           | [-90/90] Position angle (POSANG)                                         |
| `urad`       | `float`  | `—`    | `—`                    | —                                                                        |
| `grad`       | `float`  | `—`    | `—`                    | —                                                                        |
| `rrad`       | `float`  | `—`    | `—`                    | —                                                                        |
| `irad`       | `float`  | `—`    | `—`                    | —                                                                        |
| `uFWHM`      | `float`  | `—`    | `—`                    | —                                                                        |
| `gFWHM`      | `float`  | `—`    | `—`                    | —                                                                        |
| `rFWHM`      | `float`  | `—`    | `—`                    | —                                                                        |
| `iFWHM`      | `float`  | `—`    | `—`                    | —                                                                        |
| `uflg`       | `int8`   | `—`    | `meta.code`            | [0/120] u-band SExtractor extraction flag (0=most reliable) (FLAG_U) (2) |
| `gflg`       | `int8`   | `—`    | `meta.code`            | g-band SExtractor extraction flag (FLAG_G) (2)                           |
| `rflg`       | `int8`   | `—`    | `meta.code`            | r-band SExtractor extraction flag (FLAG_R) (2)                           |
| `iflg`       | `int8`   | `—`    | `meta.code`            | i-band SExtractor extraction flag (FLAG_I) (2)                           |
| `uflgISO`    | `int16`  | `—`    | `—`                    | —                                                                        |
| `gflgISO`    | `int16`  | `—`    | `—`                    | —                                                                        |
| `rflgISO`    | `int16`  | `—`    | `—`                    | —                                                                        |
| `iflgISO`    | `int16`  | `—`    | `—`                    | —                                                                        |
| `e_umag`     | `float`  | `—`    | `—`                    | —                                                                        |
| `e_gmag`     | `float`  | `—`    | `—`                    | —                                                                        |
| `e_rmag`     | `float`  | `—`    | `—`                    | —                                                                        |
| `e_imag`     | `float`  | `—`    | `—`                    | —                                                                        |
| `e_umagISO`  | `float`  | `—`    | `—`                    | —                                                                        |
| `e_gmagISO`  | `float`  | `—`    | `—`                    | —                                                                        |
| `e_rmagISO`  | `float`  | `—`    | `—`                    | —                                                                        |
| `e_imagISO`  | `float`  | `—`    | `—`                    | —                                                                        |
| `umag`       | `float`  | `mag`  | `phot.mag;em.opt.U`    | [8/41]? u-band MAG_AUTO (MAG_AUTO_U)                                     |
| `gmag`       | `float`  | `mag`  | `phot.mag;em.opt.B`    | [8/44]? g-band MAG_AUTO (MAG_AUTO_G)                                     |
| `rmag`       | `float`  | `mag`  | `phot.mag;em.opt.R`    | [9/42]? r-band MAG_AUTO (MAG_AUTO_R)                                     |
| `imag`       | `float`  | `mag`  | `phot.mag;em.opt.I`    | [8/42]? i-band MAG_AUTO (MAG_AUTO_I)                                     |
| `umagISO`    | `float`  | `—`    | `—`                    | —                                                                        |
| `gmagISO`    | `float`  | `—`    | `—`                    | —                                                                        |
| `rmagISO`    | `float`  | `—`    | `—`                    | —                                                                        |
| `imagISO`    | `float`  | `—`    | `—`                    | —                                                                        |
| `uNmp`       | `int32`  | `—`    | `—`                    | —                                                                        |
| `gNmp`       | `int32`  | `—`    | `—`                    | —                                                                        |
| `rNmp`       | `int32`  | `—`    | `—`                    | —                                                                        |
| `iNmp`       | `int32`  | `—`    | `—`                    | —                                                                        |
| `uisoA`      | `int32`  | `—`    | `—`                    | —                                                                        |
| `gisoA`      | `int32`  | `—`    | `—`                    | —                                                                        |
| `risoA`      | `int32`  | `—`    | `—`                    | —                                                                        |
| `iisoA`      | `int32`  | `—`    | `—`                    | —                                                                        |
| `uXpos`      | `float`  | `—`    | `—`                    | —                                                                        |
| `gXpos`      | `float`  | `—`    | `—`                    | —                                                                        |
| `rXpos`      | `float`  | `—`    | `—`                    | —                                                                        |
| `iXpos`      | `float`  | `—`    | `—`                    | —                                                                        |
| `uYpos`      | `float`  | `—`    | `—`                    | —                                                                        |
| `gYpos`      | `float`  | `—`    | `—`                    | —                                                                        |
| `rYpos`      | `float`  | `—`    | `—`                    | —                                                                        |
| `iYPpos`     | `float`  | `—`    | `—`                    | —                                                                        |
| `amaj`       | `float`  | `—`    | `—`                    | —                                                                        |
| `bmin`       | `float`  | `—`    | `—`                    | —                                                                        |
| `PAGAAP`     | `float`  | `—`    | `—`                    | —                                                                        |
| `umagGAAP`   | `float`  | `—`    | `—`                    | —                                                                        |
| `gmagGAAP`   | `float`  | `—`    | `—`                    | —                                                                        |
| `rmagGAAP`   | `float`  | `—`    | `—`                    | —                                                                        |
| `imagGAAP`   | `float`  | `—`    | `—`                    | —                                                                        |
| `e_umagGAAP` | `float`  | `—`    | `—`                    | —                                                                        |
| `e_gmagGAAP` | `float`  | `—`    | `—`                    | —                                                                        |
| `e_rmagGAAP` | `float`  | `—`    | `—`                    | —                                                                        |
| `e_imagGAAP` | `float`  | `—`    | `—`                    | —                                                                        |
| `dumag`      | `float`  | `—`    | `—`                    | —                                                                        |
| `dgmag`      | `float`  | `—`    | `—`                    | —                                                                        |
| `drmag`      | `float`  | `—`    | `—`                    | —                                                                        |
| `dimag`      | `float`  | `—`    | `—`                    | —                                                                        |
| `Au`         | `float`  | `—`    | `—`                    | —                                                                        |
| `Ag`         | `float`  | `—`    | `—`                    | —                                                                        |
| `Ar`         | `float`  | `—`    | `—`                    | —                                                                        |
| `Ai`         | `float`  | `—`    | `—`                    | —                                                                        |
| `zbest`      | `float`  | `—`    | `src.redshift.phot`    | [0/3.5] Best-fitting BPZ photometric redshift (Z_B_BPZ)                  |
| `Odds`       | `float`  | `—`    | `—`                    | —                                                                        |
| `TB`         | `float`  | `—`    | `—`                    | —                                                                        |
| `Tflg`       | `int8`   | `—`    | `meta.code`            | [0/12] Tile quality warning flag (TILE_FLAG) (9)                         |
| `u-g`        | `float`  | `—`    | `—`                    | —                                                                        |
| `u-r`        | `float`  | `—`    | `—`                    | —                                                                        |
| `u-i`        | `float`  | `—`    | `—`                    | —                                                                        |
| `g-r`        | `float`  | `—`    | `—`                    | —                                                                        |
| `g-i`        | `float`  | `—`    | `—`                    | —                                                                        |
| `r-i`        | `float`  | `—`    | `—`                    | —                                                                        |

### Images

No image cutout backends are configured for this survey.

### Spectra

No 1D spectroscopic backends are configured for this survey.

## References and acknowledgement

- de Jong, J. T. A. et al. (2017), *The third data release of the Kilo-Degree Survey and associated data products*, A&A, 604, A134 (`doi:10.1051/0004-6361/201628742`).
- KiDS DR3 Overview and Data Products: [http://kids.strw.leidenuniv.nl/DR3/](http://kids.strw.leidenuniv.nl/DR3/)
- VizieR catalogue `II/347`: [https://cdsarc.cds.unistra.fr/viz-bin/cat/II/347](https://cdsarc.cds.unistra.fr/viz-bin/cat/II/347)
- HATS catalogue endpoint: [https://vizcat.cds.unistra.fr/hats/kids_dr3/](https://vizcat.cds.unistra.fr/hats/kids_dr3/)

Based on observations made with ESO Telescopes at the La Silla Paranal Observatory under programme IDs 177.A-3016, 179.A-2004, and 298.A-5015. Based on data products produced by the KiDS consortium. The KiDS core team acknowledges support from the European Research Council, the Netherlands Organisation for Scientific Research (NWO), and the German Science Foundation (DFG).

## Provenance of this page

| Source ID       | Kind           | Service / Endpoint                             | Query or URL                                                  | Retrieved (UTC)        |
|-----------------|----------------|------------------------------------------------|---------------------------------------------------------------|------------------------|
| `hats_kids_dr3` | `hats_catalog` | `https://vizcat.cds.unistra.fr/hats/kids_dr3/` | `https://vizcat.cds.unistra.fr/hats/kids_dr3/main/properties` | `2026-10-18T16:58:58Z` |
<!-- /generated -->
