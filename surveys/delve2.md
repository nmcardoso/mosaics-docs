# DELVE DR2 (`delve2`)

<!-- generated:survey-page -->

The DECam Local Volume Exploration Survey Data Release 2 (DELVE DR2) is a wide-area optical imaging survey of the southern celestial hemisphere conducted with the Dark Energy Camera (DECam) mounted on the Víctor M. Blanco 4-metre Telescope at Cerro Tololo Inter-American Observatory (CTIO). DELVE DR2 combines dedicated survey observations with archival DECam community exposures, covering approximately 17,000 deg² of the southern sky in four optical bands ($g, r, i, z$). The survey catalogues over 2.5 billion astronomical objects (`delve_dr2.objects`, 2,500,247,752 rows) served by NSF’s NOIRLab Astro Data Lab.

In `mosaics`, `delve2` is currently defined as a disabled cutout entry (`enabled: false`) for HiPS color tiles, preserving the surveyed identifier for future reactivation. Additionally, its harvested Astro Data Lab TAP catalogue schema provides the column definitions needed for researchers to perform custom DECam photometric crossmatching.

| Property                 | Value                                                                 |
|--------------------------|-----------------------------------------------------------------------|
| What MOSAICS extracts    | Cutouts: `hips` (color)                                               |
| Archives                 | CDS HiPS / hips2fits (`alasky.cds.unistra.fr`), `datalab.noirlab.edu` |
| Bands / wavelength range | `color`                                                               |
| Used by                  | `mosaics-cutout full`                                                 |

## Configuration

### The shipped file

Source: `defs/surveys/delve2.yaml`

```yaml
# DELVE DR2 -- cutouts only, HiPS colour tiles.

survey_slug: delve2
name: DELVE DR2

cutout:
  - adapter: hips
    # DOES NOT RESOLVE. Kept so the survey is described rather than
    # forgotten, and so the next person does not re-derive the same
    # guess. `--plan` will not reach it: the adapter is disabled. No
    # single-band sibling exists either -- see the note below -- so
    # `color` is the only key this survey has.
    adapter_params:
      hips_id: CDS/P/DELVE/DR2/color
    bands: [color]
    enabled: false

photometric_system:
  pixel_unit: unknown
  pixel_unit_note: >-
    Moot, because there are no tiles. no HiPS matching DELVE is registered (MOCServer, 2026-09-04). The
    pixel unit cannot be resolved for a service that does not exist, and
    this entry is a record of having looked rather than a working adapter.
    Whoever finds a real HiPS for this survey -- or another cutout service
    -- should replace the adapter below and revisit this note; the tiles
    would then need a scale check of their own, as no HiPS build this
    project has seen states one.
```

### Schema of this file

| Key present          | Type                   | Required   | Default   | Meaning                                                                                            |
|----------------------|------------------------|------------|-----------|----------------------------------------------------------------------------------------------------|
| `survey_slug`        | str                    | yes        | —         | Canonical lowercase identifier for the survey and default column prefix.                           |
| `name`               | str                    | yes        | —         | Human-readable survey and data-release name used in logs, plans, and reports.                      |
| `cutout`             | list of cutout entries | no         | []        | Image cutout backends (`hips`, `lsv`, `nersc`, `sia`, `vsa`) serving this survey’s passbands.      |
| `photometric_system` | mapping                | no         | None      | Passband SVO filter identifiers, flux/magnitude homogenisation rules, and image pixel calibration. |

### Adapter parameters

- **Cutout `None` (`adapter: hips`, bands `['color']`):** configured `adapter_params`: `hips_id='CDS/P/DELVE/DR2/color'`.

### Common changes

- **Cutout adapter status**: The HiPS cutout adapter (`CDS/P/DELVE/DR2/color`) is disabled by default (`enabled: false`) because the tile repository is not currently registered on the CDS MOCServer. Do not toggle `enabled: true` unless a functioning tile endpoint or proxy has been verified.
- **Enabling catalogue crossmatching**: To crossmatch target coordinates against the DELVE DR2 2.5-billion-object catalogue, define a `catalog` entry in your custom entrypoint YAML using `adapter: adl` (or `adapter: tap` with `service: datalab`) targeting table `delve_dr2.objects`.
- **Selecting photometric apertures**: When querying `delve_dr2.objects`, configure `column_mapping` to select between SExtractor Kron auto magnitudes (`mag_auto_*`), aperture magnitudes (`mag_aper_*`), or PSF magnitudes (`mag_psf_*`) in the $g, r, i, z$ filters.
- **Star-galaxy classification**: Filter morphological candidates in custom catalogue queries using the neural-network classifier `class_star_*` (0 for extended galaxies, 1 for point-source stars) or the multiclass integer flag `extended_class_*`.

### Commands

```console
$ uv run -m mosaics.cli.cutout full -t test/objects_sample_1k.csv -o cutouts.lance --config defs/cutout.yaml --survey delve2 --plan
```

## Data served by the archive

- **HiPS cutout availability**: The configured HiPS identifier `CDS/P/DELVE/DR2/color` does not currently resolve on the CDS HiPS / hips2fits network (verified via MOCServer search). The adapter is set to `enabled: false` so that CLI commands such as `mosaics-cutout --plan` gracefully skip it.
- **Pixel units**: Because image tiles are currently unreachable, `pixel_unit` is designated as `unknown`. When an active tile service is connected, pixel scale and flux conversion factors must be verified before quantitative cutout measurements are performed.
- **Astro Data Lab catalogue schema**: The DELVE DR2 catalogue table at Astro Data Lab (`delve_dr2.objects`, 2,500,247,752 rows) provides 126 columns, including median astrometric coordinates (`ra`, `dec`), multi-band optical photometry, exposure metadata (`expnum_*`, `exptime_*`), and Galactic dust extinction $E(B-V)$ from Schlegel, Finkbeiner, & Davis (1998).
- **Aperture geometry**: Image shape and orientation parameters (`a_image_*`, `b_image_*`, `theta_image_*`) are supplied in image pixel and degree units for adaptive apertures in each band.

### Catalogue: `delve_dr2.objects`

Main DELVE DR2 object table (2,500,247,752 rows)

**Service URL:** `https://datalab.noirlab.edu/tap`

*Showing 100 of 126 total archive columns (all MOSAICS-mapped columns plus primary table fields; full 126-column schema is stored in `docs/_data/sources/delve2/catalog_delve_dr2.json`).*

| Column               | Type            | Unit    | UCD                    | Description                                                      |
|----------------------|-----------------|---------|------------------------|------------------------------------------------------------------|
| `theta_image_z`      | `adql:REAL`     | `deg`   | `—`                    | Position angle of z-band adaptive aperture in image coordinates  |
| `a_image_g`          | `adql:REAL`     | `pixel` | `—`                    | Semi-major axis of g-band adaptive aperture in image coordinates |
| `a_image_i`          | `adql:REAL`     | `pixel` | `—`                    | Semi-major axis of i-band adaptive aperture in image coordinates |
| `a_image_r`          | `adql:REAL`     | `pixel` | `—`                    | Semi-major axis of r-band adaptive aperture in image coordinates |
| `a_image_z`          | `adql:REAL`     | `pixel` | `—`                    | Semi-major axis of z-band adaptive aperture in image coordinates |
| `b_image_g`          | `adql:REAL`     | `pixel` | `—`                    | Semi-minor axis of g-band adaptive aperture in image coordinates |
| `b_image_i`          | `adql:REAL`     | `pixel` | `—`                    | Semi-minor axis of i-band adaptive aperture in image coordinates |
| `b_image_r`          | `adql:REAL`     | `pixel` | `—`                    | Semi-minor axis of r-band adaptive aperture in image coordinates |
| `b_image_z`          | `adql:REAL`     | `pixel` | `—`                    | Semi-minor axis of z-band adaptive aperture in image coordinates |
| `ccdnum_g`           | `adql:INTEGER`  | `—`     | `—`                    | CCD number for best g-band exposure                              |
| `ccdnum_i`           | `adql:INTEGER`  | `—`     | `—`                    | CCD number for best i-band exposure                              |
| `ccdnum_r`           | `adql:INTEGER`  | `—`     | `—`                    | CCD number for best r-band exposure                              |
| `ccdnum_z`           | `adql:INTEGER`  | `—`     | `—`                    | CCD number for best z-band exposure                              |
| `class_star_g`       | `adql:REAL`     | `—`     | `—`                    | Neural-net g-band star-galaxy classifier (see SExtractor manual  |
| `class_star_i`       | `adql:REAL`     | `—`     | `—`                    | Neural-net i-band star-galaxy classifier (see SExtractor manual  |
| `class_star_r`       | `adql:REAL`     | `—`     | `—`                    | Neural-net r-band star-galaxy classifier (see SExtractor manual  |
| `class_star_z`       | `adql:REAL`     | `—`     | `—`                    | Neural-net z-band star-galaxy classifier (see SExtractor manual  |
| `dec`                | `adql:DOUBLE`   | `deg`   | `pos.eq.dec;meta.main` | Declination derived from the median position of each detection   |
| `ebv`                | `adql:REAL`     | `mag`   | `—`                    | Schlegel, Finkbeiner, and Davis (1998) E(B-V) value              |
| `expnum_g`           | `adql:INTEGER`  | `—`     | `—`                    | Exposure number for best g-band exposure                         |
| `expnum_i`           | `adql:INTEGER`  | `—`     | `—`                    | Exposure number for best i-band exposure                         |
| `expnum_r`           | `adql:INTEGER`  | `—`     | `—`                    | Exposure number for best r-band exposure                         |
| `expnum_z`           | `adql:INTEGER`  | `—`     | `—`                    | Exposure number for best z-band exposure                         |
| `exptime_g`          | `adql:REAL`     | `sec`   | `—`                    | Shutter-open g-band exposure time                                |
| `exptime_i`          | `adql:REAL`     | `sec`   | `—`                    | Shutter-open i-band exposure time                                |
| `exptime_r`          | `adql:REAL`     | `sec`   | `—`                    | Shutter-open r-band exposure time                                |
| `exptime_z`          | `adql:REAL`     | `sec`   | `—`                    | Shutter-open z-band exposure time                                |
| `extended_class_g`   | `adql:SMALLINT` | `—`     | `—`                    | Spread_model-based morphology class (-9 unknown, 0 high-confiden |
| `extended_class_i`   | `adql:SMALLINT` | `—`     | `—`                    | Spread_model-based morphology class (-9 unknown, 0 high-confiden |
| `extended_class_r`   | `adql:SMALLINT` | `—`     | `—`                    | Spread_model-based morphology class (-9 unknown, 0 high-confiden |
| `extended_class_z`   | `adql:SMALLINT` | `—`     | `—`                    | Spread_model-based morphology class (-9 unknown, 0 high-confiden |
| `extinction_g`       | `adql:REAL`     | `mag`   | `—`                    | Interstellar extinction value derived from SFD E(B-V) and DES A( |
| `extinction_i`       | `adql:REAL`     | `mag`   | `—`                    | Interstellar extinction value derived from SFD E(B-V) and DES A( |
| `extinction_r`       | `adql:REAL`     | `mag`   | `—`                    | Interstellar extinction value derived from SFD E(B-V) and DES A( |
| `extinction_z`       | `adql:REAL`     | `mag`   | `—`                    | Interstellar extinction value derived from SFD E(B-V) and DES A( |
| `flags_g`            | `adql:SMALLINT` | `—`     | `—`                    | SExtractor flags for the best detection in g-band                |
| `flags_i`            | `adql:SMALLINT` | `—`     | `—`                    | SExtractor flags for the best detection in i-band                |
| `flags_r`            | `adql:SMALLINT` | `—`     | `—`                    | SExtractor flags for the best detection in r-band                |
| `flags_z`            | `adql:SMALLINT` | `—`     | `—`                    | SExtractor flags for the best detection in z-band                |
| `hpx2048`            | `adql:BIGINT`   | `—`     | `—`                    | NSIDE 2048 HEALPix index, in RING scheme, for each object        |
| `htm9`               | `adql:INTEGER`  | `—`     | `—`                    | HTM Level-9 index                                                |
| `mag_auto_g`         | `adql:REAL`     | `mag`   | `—`                    | Automatic aperture magnitude in g-band (see SExtractor manual fo |
| `mag_auto_i`         | `adql:REAL`     | `mag`   | `—`                    | Automatic aperture magnitude in i-band (see SExtractor manual fo |
| `mag_auto_r`         | `adql:REAL`     | `mag`   | `—`                    | Automatic aperture magnitude in r-band (see SExtractor manual fo |
| `mag_auto_z`         | `adql:REAL`     | `mag`   | `—`                    | Automatic aperture magnitude in z-band (see SExtractor manual fo |
| `mag_psf_g`          | `adql:REAL`     | `mag`   | `—`                    | PSF magnitude in g-band (see SExtractor manual for details)      |
| `mag_psf_i`          | `adql:REAL`     | `mag`   | `—`                    | PSF magnitude in i-band (see SExtractor manual for details)      |
| `mag_psf_r`          | `adql:REAL`     | `mag`   | `—`                    | PSF magnitude in r-band (see SExtractor manual for details)      |
| `mag_psf_z`          | `adql:REAL`     | `mag`   | `—`                    | PSF magnitude in z-band (see SExtractor manual for details)      |
| `magerr_auto_g`      | `adql:REAL`     | `mag`   | `—`                    | Automatic aperture magnitude error in g-band (see SExtractor man |
| `magerr_auto_i`      | `adql:REAL`     | `mag`   | `—`                    | Automatic aperture magnitude error in i-band (see SExtractor man |
| `magerr_auto_r`      | `adql:REAL`     | `mag`   | `—`                    | Automatic aperture magnitude error in r-band (see SExtractor man |
| `magerr_auto_z`      | `adql:REAL`     | `mag`   | `—`                    | Automatic aperture magnitude error in z-band (see SExtractor man |
| `magerr_psf_g`       | `adql:REAL`     | `mag`   | `—`                    | PSF magnitude error in g-band (see SExtractor manual for details |
| `magerr_psf_i`       | `adql:REAL`     | `mag`   | `—`                    | PSF magnitude error in i-band (see SExtractor manual for details |
| `magerr_psf_r`       | `adql:REAL`     | `mag`   | `—`                    | PSF magnitude error in r-band (see SExtractor manual for details |
| `magerr_psf_z`       | `adql:REAL`     | `mag`   | `—`                    | PSF magnitude error in z-band (see SExtractor manual for details |
| `nepochs_g`          | `adql:SMALLINT` | `—`     | `—`                    | Number of single epoch g-band detections that went into this obj |
| `nepochs_i`          | `adql:SMALLINT` | `—`     | `—`                    | Number of single epoch i-band detections that went into this obj |
| `nepochs_r`          | `adql:SMALLINT` | `—`     | `—`                    | Number of single epoch r-band detections that went into this obj |
| `nepochs_z`          | `adql:SMALLINT` | `—`     | `—`                    | Number of single epoch z-band detections that went into this obj |
| `nest4096`           | `adql:INTEGER`  | `—`     | `—`                    | Healpix ID with nside 4096, nested schema                        |
| `quick_object_id`    | `adql:BIGINT`   | `—`     | `meta.id;meta.main`    | Unique identifier for each object                                |
| `ra`                 | `adql:DOUBLE`   | `deg`   | `pos.eq.ra;meta.main`  | Right ascension derived from the median position of each detecti |
| `random_id`          | `adql:REAL`     | `—`     | `—`                    | Random ID in the range 0 to 100 (decimal)                        |
| `ring256`            | `adql:INTEGER`  | `—`     | `—`                    | Healpix ID with nside 256, ring schema                           |
| `spread_model_g`     | `adql:REAL`     | `—`     | `—`                    | Likelihood-based g-band star-galaxy classifier (see Desai et al  |
| `spread_model_i`     | `adql:REAL`     | `—`     | `—`                    | Likelihood-based i-band star-galaxy classifier (see Desai et al  |
| `spread_model_r`     | `adql:REAL`     | `—`     | `—`                    | Likelihood-based r-band star-galaxy classifier (see Desai et al  |
| `spread_model_z`     | `adql:REAL`     | `—`     | `—`                    | Likelihood-based z-band star-galaxy classifier (see Desai et al  |
| `spreaderr_model_g`  | `adql:REAL`     | `—`     | `—`                    | Likelihood-based g-band star-galaxy classifier error (see Desai  |
| `spreaderr_model_i`  | `adql:REAL`     | `—`     | `—`                    | Likelihood-based i-band star-galaxy classifier error (see Desai  |
| `spreaderr_model_r`  | `adql:REAL`     | `—`     | `—`                    | Likelihood-based r-band star-galaxy classifier error (see Desai  |
| `spreaderr_model_z`  | `adql:REAL`     | `—`     | `—`                    | Likelihood-based z-band star-galaxy classifier error (see Desai  |
| `t_eff_g`            | `adql:REAL`     | `—`     | `—`                    | Effective exposure depth scale factor for best g-band exposure ( |
| `t_eff_i`            | `adql:REAL`     | `—`     | `—`                    | Effective exposure depth scale factor for best i-band exposure ( |
| `t_eff_r`            | `adql:REAL`     | `—`     | `—`                    | Effective exposure depth scale factor for best r-band exposure ( |
| `t_eff_z`            | `adql:REAL`     | `—`     | `—`                    | Effective exposure depth scale factor for best z-band exposure ( |
| `theta_image_g`      | `adql:REAL`     | `deg`   | `—`                    | Position angle of g-band adaptive aperture in image coordinates  |
| `theta_image_i`      | `adql:REAL`     | `deg`   | `—`                    | Position angle of i-band adaptive aperture in image coordinates  |
| `theta_image_r`      | `adql:REAL`     | `deg`   | `—`                    | Position angle of r-band adaptive aperture in image coordinates  |
| `wavg_flags_g`       | `adql:SMALLINT` | `mag`   | `—`                    | OR of SExtractor flags from all g-band detections                |
| `wavg_flags_i`       | `adql:SMALLINT` | `mag`   | `—`                    | OR of SExtractor flags from all i-band detections                |
| `wavg_flags_r`       | `adql:SMALLINT` | `mag`   | `—`                    | OR of SExtractor flags from all r-band detections                |
| `wavg_flags_z`       | `adql:SMALLINT` | `mag`   | `—`                    | OR of SExtractor flags from all z-band detections                |
| `wavg_mag_auto_g`    | `adql:REAL`     | `mag`   | `—`                    | Weighted average automatic magnitude in g-band                   |
| `wavg_mag_auto_i`    | `adql:REAL`     | `mag`   | `—`                    | Weighted average automatic magnitude in i-band                   |
| `wavg_mag_auto_r`    | `adql:REAL`     | `mag`   | `—`                    | Weighted average automatic magnitude in r-band                   |
| `wavg_mag_auto_z`    | `adql:REAL`     | `mag`   | `—`                    | Weighted average automatic magnitude in z-band                   |
| `wavg_mag_psf_g`     | `adql:REAL`     | `mag`   | `—`                    | Weighted average PSF magnitude in g-band                         |
| `wavg_mag_psf_i`     | `adql:REAL`     | `mag`   | `—`                    | Weighted average PSF magnitude in i-band                         |
| `wavg_mag_psf_r`     | `adql:REAL`     | `mag`   | `—`                    | Weighted average PSF magnitude in r-band                         |
| `wavg_mag_psf_z`     | `adql:REAL`     | `mag`   | `—`                    | Weighted average PSF magnitude in z-band                         |
| `wavg_magerr_auto_g` | `adql:REAL`     | `mag`   | `—`                    | Sum in quadrature of the automatic magnitude errors from each g- |
| `wavg_magerr_auto_i` | `adql:REAL`     | `mag`   | `—`                    | Sum in quadrature of the automatic magnitude errors from each i- |
| `wavg_magerr_auto_r` | `adql:REAL`     | `mag`   | `—`                    | Sum in quadrature of the automatic magnitude errors from each r- |
| `wavg_magerr_auto_z` | `adql:REAL`     | `mag`   | `—`                    | Sum in quadrature of the automatic magnitude errors from each z- |
| `wavg_magerr_psf_g`  | `adql:REAL`     | `mag`   | `—`                    | Sum in quadrature of the PSF magnitude errors from each g-band d |
| `wavg_magerr_psf_i`  | `adql:REAL`     | `mag`   | `—`                    | Sum in quadrature of the PSF magnitude errors from each i-band d |
| `wavg_magerr_psf_r`  | `adql:REAL`     | `mag`   | `—`                    | Sum in quadrature of the PSF magnitude errors from each r-band d |

### Images

- **Pixel unit:** `unknown` | **AB zero point:** `—` | **Calibration note:** Moot, because there are no tiles. no HiPS matching DELVE is registered (MOCServer, 2026-09-04). The pixel unit cannot be resolved for a service that does not exist, and this entry is a record of having looked rather than a working adapter. Whoever finds a real HiPS for this survey – or another cutout service – should replace the adapter below and revisit this note; the tiles would then need a scale check of their own, as no HiPS build this project has seen states one.
- **Cutout service `None`:** adapter `hips`, bands `['color']`, params `{'hips_id': 'CDS/P/DELVE/DR2/color'}`.

### Spectra

No 1D spectroscopic backends are configured for this survey.

## References and acknowledgement

- Drlica-Wagner, A., et al. (2021), *ApJS*, 256, 2 (`doi:10.3847/1538-4365/ac004f`) — The DECam Local Volume Exploration Survey: First Data Release.
- Drlica-Wagner, A., et al. (2022), *ApJS*, 261, 38 (`doi:10.3847/1538-4365/ac7798`) — The Second Data Release of the DECam Local Volume Exploration Survey.
- NOIRLab Astro Data Lab TAP service: [https://datalab.noirlab.edu/tap](https://datalab.noirlab.edu/tap) (table `delve_dr2.objects`).
- DELVE Survey Website: [https://delve-survey.github.io/](https://delve-survey.github.io/).

Researchers using DELVE DR2 data should cite Drlica-Wagner et al. (2021, 2022) and acknowledge the DELVE collaboration, the Dark Energy Camera (DECam), Cerro Tololo Inter-American Observatory (CTIO), and NSF’s NOIRLab Astro Data Lab. Please include the official Astro Data Lab acknowledgement as detailed at [https://datalab.noirlab.edu/acknowledgements.php](https://datalab.noirlab.edu/acknowledgements.php).

## Provenance of this page

| Source ID           | Kind        | Service / Endpoint                              | Query or URL                                                                                                          | Retrieved (UTC)        |
|---------------------|-------------|-------------------------------------------------|-----------------------------------------------------------------------------------------------------------------------|------------------------|
| `catalog_delve_dr2` | `tap_table` | `https://datalab.noirlab.edu/tap`               | `SELECT column_name, datatype, unit, ucd, description FROM TAP_SCHEMA.columns WHERE table_name = 'delve_dr2.objects'` | `2026-10-18T16:29:09Z` |
| `hips_color`        | `hips`      | `https://alasky.cds.unistra.fr/MocServer/query` | `https://alasky.cds.unistra.fr/MocServer/query?ID=CDS/P/DELVE/DR2/color&fmt=json&get=record`                          | `2026-10-18T16:52:29Z` |
<!-- /generated -->
