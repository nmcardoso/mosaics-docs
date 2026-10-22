# Pan-STARRS1 (`ps1`)

<!-- generated:survey-page -->

The Panoramic Survey Telescope and Rapid Response System 1 (Pan-STARRS1, PS1) $3\pi$ Steradian Survey observed the three-quarters of the sky north of declination $\delta \approx -30^\circ$ (~30,000 deg²) from the Haleakala Observatory on Maui, Hawaii. The survey imaged the sky across five broad optical and near-infrared passbands: $g_\mathrm{P1}, r_\mathrm{P1}, i_\mathrm{P1}, z_\mathrm{P1}, y_\mathrm{P1}$.

In `mosaics`, `ps1` provides optical and near-infrared image cutouts across the northern and equatorial sky. Stamped cutouts are extracted in $g, r, z$ using the Legacy Survey Viewer (`lsv`) backend via its `ps1` layer.

| Property                 | Value                                                                                                                 |
|--------------------------|-----------------------------------------------------------------------------------------------------------------------|
| What MOSAICS extracts    | Cutouts: `lsv` (g,r,z), `hips` (g), `hips` (r), `hips` (z)                                                            |
| Archives                 | Legacy Survey Viewer (`legacysurvey.org`), CDS HiPS / hips2fits (`alasky.cds.unistra.fr`), `tapvizier.cds.unistra.fr` |
| Bands / wavelength range | `g`, `r`, `z`                                                                                                         |
| Used by                  | `mosaics-cutout full`                                                                                                 |

## Configuration

### The shipped file

Source: `defs/surveys/ps1.yaml`

```yaml
# Pan-STARRS1 -- cutouts only.
#
# Was keyed `ps2` in the old cutout config while declaring `cutout_ps1_*`
# columns; the Viewer layer is PS1 and so is the data, so the slug is now the
# one the columns already used.

survey_slug: ps1
name: Pan-STARRS1

# max_mag: 23.2 (max_mag_band: r). 5 sigma point source in the stacked 3pi
# survey, AB (Chambers et al. 2016): grizy 23.3/23.2/23.1/22.3/21.4. `r` is
# quoted rather than the deepest band because it is the middle of the three
# the stack fills and the three are within 0.2 mag of one another.
cutout:
  - adapter: lsv          # VERIFIED LIVE 2026-08-25
    adapter_params:
      layer: ps1
      bands: grz
    bands: [g, r, z]
  # PanSTARRS DR1 is also published as five single-band HiPS with FITS
  # tiles -- g, r, i, z, y (MOCServer, 2026-09-05). Only the three the
  # stack above already fills are keyed, so the two shapes stay
  # interchangeable rather than one of them widening the schema; add `i:`
  # and `y:` with `CDS/P/PanSTARRS/DR1/{i,y}` if those columns are wanted.
  # Disabled: the note above says the Viewer's scale is unresolved, and
  # the HiPS build's is a separate unknown.
  - adapter: hips
    adapter_params:
      hips_id: CDS/P/PanSTARRS/DR1/g
    bands: [g]
    enabled: false
  - adapter: hips
    adapter_params:
      hips_id: CDS/P/PanSTARRS/DR1/r
    bands: [r]
    enabled: false
  - adapter: hips
    adapter_params:
      hips_id: CDS/P/PanSTARRS/DR1/z
    bands: [z]
    enabled: false

photometric_system:
  pixel_unit: unknown
  pixel_unit_note: >-
    PS1 stacks are natively on a fixed 25.0 AB zero point (Magnier et al.
    2020), but the Viewer rescales them and states no factor: a fits-cutout
    of the ps1 layer carries WCS, SURVEY, BANDS and IMAGETYP and nothing
    else -- no BUNIT, no MAGZPT (checked 2026-09-04). What *is* settled is
    that it is not the native zero point. Background-subtracted total stamp
    flux was compared against the ls-dr10 layer, which is nanomaggies by
    construction, over five fields in g and r: the ratio is order unity
    (0.13-3.0, median 0.67, the scatter being seeing, passband and sky
    rather than scale), where a 25.0 zero point would give 0.10 to within a
    few per cent. So the pixels are nanomaggy-like. That is not enough to
    write a factor down -- pinning it needs aperture photometry against the
    PS1 catalogue on isolated standards -- so this stays unknown, but it is
    now unknown within a factor of two rather than within a factor of ten.
```

### Schema of this file

| Key present          | Type                   | Required   | Default   | Meaning                                                                                            |
|----------------------|------------------------|------------|-----------|----------------------------------------------------------------------------------------------------|
| `survey_slug`        | str                    | yes        | —         | Canonical lowercase identifier for the survey and default column prefix.                           |
| `name`               | str                    | yes        | —         | Human-readable survey and data-release name used in logs, plans, and reports.                      |
| `cutout`             | list of cutout entries | no         | []        | Image cutout backends (`hips`, `lsv`, `nersc`, `sia`, `vsa`) serving this survey’s passbands.      |
| `photometric_system` | mapping                | no         | None      | Passband SVO filter identifiers, flux/magnitude homogenisation rules, and image pixel calibration. |

### Adapter parameters

- **Cutout `None` (`adapter: lsv`, bands `['g', 'r', 'z']`):** configured `adapter_params`: `layer='ps1'`, `bands='grz'`.
- **Cutout `None` (`adapter: hips`, bands `['g']`):** configured `adapter_params`: `hips_id='CDS/P/PanSTARRS/DR1/g'`.
- **Cutout `None` (`adapter: hips`, bands `['r']`):** configured `adapter_params`: `hips_id='CDS/P/PanSTARRS/DR1/r'`.
- **Cutout `None` (`adapter: hips`, bands `['z']`):** configured `adapter_params`: `hips_id='CDS/P/PanSTARRS/DR1/z'`.

### Common changes

- Disable Pan-STARRS1 cutouts in an entrypoint configuration by setting:
  ```yaml
  overrides:
    surveys/ps1:
      cutout: []
  ```
- Extract a subset of passbands on the command line using `--bands ps1:g,ps1:r` or by specifying `bands: [g, r]` in an entrypoint override.
- Enable single-band CDS HiPS cutouts by setting `enabled: true` on the corresponding HiPS entries (`CDS/P/PanSTARRS/DR1/g`, `r`, or `z`) under `overrides[surveys/ps1].cutout`.

### Commands

```console
$ uv run -m mosaics.cli.cutout full -t test/objects_sample_1k.csv -o cutouts.lance --config defs/cutout.yaml --survey ps1 --plan
```

## Data served by the archive

- Survey depth: 5-sigma point source depths in the stacked $3\pi$ survey, AB (Chambers et al. 2016): $g = 23.3, r = 23.2, i = 23.1, z = 22.3, y = 21.4$ mag. The $r$ band (23.2 mag) is quoted as the representative depth in MOSAICS because it sits in the middle of the three stacked bands ($g, r, z$), which span a tight range of 0.2 mag.
- Slug and column naming: Earlier configurations referenced this survey as `ps2` while generating `cutout_ps1_*` columns. The survey slug is standardized to `ps1` to match both the Legacy Survey Viewer layer identifier and the extracted dataset column names.
- Cutout backend and pixel scaling: Cutouts are actively served by the Legacy Survey Viewer (`lsv`) layer `ps1` in $g, r, z$. Natively, Pan-STARRS1 coadded stacks use a fixed 25.0 AB zero point (Magnier et al. 2020). However, the Legacy Survey Viewer rescales these stacks without providing `BUNIT` or `MAGZPT` keywords in FITS headers. Total flux comparisons against the Legacy Surveys DR10 layer (`ls-dr10`, which is calibrated in nanomaggies) over test fields indicate that the pixels are nanomaggy-like (median flux ratio 0.67, where a 25.0 zero point would yield ~0.10). Because the exact scaling factor has not been pinned down with isolated standard stars, `pixel_unit` is marked as `unknown`.
- CDS HiPS availability: CDS publishes five single-band HiPS collections with FITS tiles (`CDS/P/PanSTARRS/DR1/{g,r,i,z,y}`, order 11, pixel scale $5.591 \times 10^{-5}$ deg, covering ~76% of the sky). In MOSAICS, only the three bands present in the Viewer stack ($g, r, z$) are declared, and they remain disabled pending independent calibration checks.

### Catalogue: `II/349/ps1`

Pan-STARRS DR1 catalogue, output (on a total of 1,919,106,885 sources)

**Service URL:** `https://tapvizier.cds.unistra.fr/TAPVizieR/tap`

| Column      | Type     | Unit     | UCD                           | Description                                                                                             |
|-------------|----------|----------|-------------------------------|---------------------------------------------------------------------------------------------------------|
| `RAJ2000`   | `double` | `deg`    | `pos.eq.ra;meta.main`         | Right ascension (J2000) (weighted mean) at epoch “Epoch” (raMean) (1)                                   |
| `DEJ2000`   | `double` | `deg`    | `pos.eq.dec;meta.main`        | Declination (J2000) (weighted mean) at epoch “Epoch” (decMean) (1)                                      |
| `objID`     | `long`   | `—`      | `meta.id;meta.main`           | Unique object identifier (objID)                                                                        |
| `f_objID`   | `long`   | `—`      | `meta.code`                   | Information flag bitmask indicating details of the photometry (objInfoFlag) (2)                         |
| `Qual`      | `short`  | `—`      | `meta.code.qual`              | Subset of objInfoFlag denoting whether this object is real or a likely false positive (qualityFlag) (3) |
| `e_RAJ2000` | `double` | `arcsec` | `stat.error;pos.eq.ra`        | ? Right ascension standard deviation from single epoch detections (raMeanErr)                           |
| `e_DEJ2000` | `double` | `arcsec` | `stat.error;pos.eq.dec`       | ? Declination standard deviation from single epoch detections (decMeanErr)                              |
| `Epoch`     | `double` | `d`      | `time.epoch`                  | ? Modified Julian Date (MJD) of the mean epoch (epochMean)                                              |
| `Ns`        | `short`  | `—`      | `meta.number`                 | [0/50] Number of stack detections (nStackDetections)                                                    |
| `Nd`        | `short`  | `—`      | `meta.number`                 | [3/369] Number of single epoch detections in all filters (nDetections)                                  |
| `gmag`      | `double` | `mag`    | `phot.mag;em.opt.B`           | [-33/33]? Mean PSF AB magnitude from g filter (4866{AA}) detections (gMeanPSFMag)                       |
| `e_gmag`    | `float`  | `mag`    | `stat.error;phot.mag`         | [0/0.5]? Error in gmag (gMeanPSFMagErr)                                                                 |
| `gKmag`     | `double` | `mag`    | `phot.mag;em.opt.B`           | ? Mean Kron (1980) AB magnitude from g filter detections (gMeanKronMag)                                 |
| `e_gKmag`   | `float`  | `mag`    | `stat.error;phot.mag`         | ? Error in gKmag (gMeanKronMagErr)                                                                      |
| `gFlags`    | `int`    | `—`      | `meta.code`                   | Information flag bitmask for mean object from g filter detections (gFlags) (4)                          |
| `rmag`      | `double` | `mag`    | `phot.mag;em.opt.R`           | [-6/30]? Mean PSF AB magnitude from r filter (6215{AA}) detections (rMeanPSFMag)                        |
| `e_rmag`    | `float`  | `mag`    | `stat.error;phot.mag`         | [0/0.5]? Error in rmag (rMeanPSFMagErr)                                                                 |
| `rKmag`     | `double` | `mag`    | `phot.mag;em.opt.R`           | ? Mean Kron (1980) AB magnitude from r filter detections (rMeanKronMag)                                 |
| `e_rKmag`   | `float`  | `mag`    | `stat.error;phot.mag`         | ? Error in rKmag (rMeanKronMagErr)                                                                      |
| `rFlags`    | `int`    | `—`      | `meta.code`                   | Information flag bitmask for mean object from r filter detections (rFlags) (4)                          |
| `imag`      | `double` | `mag`    | `phot.mag;em.opt.I`           | [-5/30]? Mean PSF AB magnitude from i filter (7545{AA}) detections (iMeanPSFMag)                        |
| `e_imag`    | `float`  | `mag`    | `stat.error;phot.mag`         | [0/0.5]? Error in imag (iMeanPSFMagErr)                                                                 |
| `iKmag`     | `double` | `mag`    | `phot.mag;em.opt.I`           | ? Mean Kron (1980) AB magnitude from i filter detections (iMeanKronMag)                                 |
| `e_iKmag`   | `float`  | `mag`    | `stat.error;phot.mag`         | ? Error in iKmag (iMeanKronMagErr)                                                                      |
| `iFlags`    | `int`    | `—`      | `meta.code`                   | Information flag bitmask for mean object from i filter detections (iFlags) (4)                          |
| `zmag`      | `double` | `mag`    | `phot.mag;em.opt.I`           | [-5/29]? Mean PSF AB magnitude from z filter (8679{AA}) detections (zMeanPSFMag)                        |
| `e_zmag`    | `float`  | `mag`    | `stat.error;phot.mag`         | [0/0.5]? Error in zmag (zMeanPSFMagErr)                                                                 |
| `zKmag`     | `double` | `mag`    | `phot.mag;em.opt.I`           | ? Mean Kron (1980) AB magnitude from z filter detections (zMeanKronMag)                                 |
| `e_zKmag`   | `float`  | `mag`    | `stat.error;phot.mag`         | ? Error in zKmag (zMeanKronMagErr)                                                                      |
| `zFlags`    | `int`    | `—`      | `meta.code`                   | Information flag bitmask for mean object from z filter detections (zFlags) (4)                          |
| `ymag`      | `double` | `mag`    | `phot.mag;em.IR.J`            | [-5/29]? Mean PSF AB magnitude from y filter (9633{AA}) detections (yMeanPSFMag)                        |
| `e_ymag`    | `float`  | `mag`    | `stat.error;phot.mag;em.IR.J` | [0/0.5]? Error in ymag (yMeanPSFMagErr)                                                                 |
| `yKmag`     | `double` | `mag`    | `phot.mag;em.IR.J`            | ? Mean Kron (1980) AB magnitude from y filter detections (yMeanKronMag)                                 |
| `e_yKmag`   | `float`  | `mag`    | `stat.error;phot.mag`         | ? Error in yKmag (yMeanKronMagErr)                                                                      |
| `yFlags`    | `int`    | `—`      | `meta.code`                   | Information flag bitmask for mean object from y filter detections (yFlags) (4)                          |

### Images

- **Pixel unit:** `unknown` | **AB zero point:** `—` | **Calibration note:** PS1 stacks are natively on a fixed 25.0 AB zero point (Magnier et al. 2020), but the Viewer rescales them and states no factor: a fits-cutout of the ps1 layer carries WCS, SURVEY, BANDS and IMAGETYP and nothing else – no BUNIT, no MAGZPT (checked 2026-09-04). What *is* settled is that it is not the native zero point. Background-subtracted total stamp flux was compared against the ls-dr10 layer, which is nanomaggies by construction, over five fields in g and r: the ratio is order unity (0.13-3.0, median 0.67, the scatter being seeing, passband and sky rather than scale), where a 25.0 zero point would give 0.10 to within a few per cent. So the pixels are nanomaggy-like. That is not enough to write a factor down – pinning it needs aperture photometry against the PS1 catalogue on isolated standards – so this stays unknown, but it is now unknown within a factor of two rather than within a factor of ten.
- **Cutout service `None`:** adapter `lsv`, bands `['g', 'r', 'z']`, params `{'layer': 'ps1', 'bands': 'grz'}`.
- **Cutout service `None`:** adapter `hips`, bands `['g']`, params `{'hips_id': 'CDS/P/PanSTARRS/DR1/g'}`.
- **Cutout service `None`:** adapter `hips`, bands `['r']`, params `{'hips_id': 'CDS/P/PanSTARRS/DR1/r'}`.
- **Cutout service `None`:** adapter `hips`, bands `['z']`, params `{'hips_id': 'CDS/P/PanSTARRS/DR1/z'}`.

| HiPS ID                 | Regime   | Frame        | Order   | Pixel scale (deg)   | Tile format     | Mirrors                                             |
|-------------------------|----------|--------------|---------|---------------------|-----------------|-----------------------------------------------------|
| `CDS/P/PanSTARRS/DR1/g` | Optical  | `equatorial` | `11`    | `5.591E-5`          | `fits jpeg png` | `alasky.cds.unistra.fr`, `alaskybis.cds.unistra.fr` |
| `CDS/P/PanSTARRS/DR1/r` | Optical  | `equatorial` | `11`    | `5.591E-5`          | `fits jpeg png` | `alasky.cds.unistra.fr`, `alaskybis.cds.unistra.fr` |
| `CDS/P/PanSTARRS/DR1/z` | Optical  | `equatorial` | `11`    | `5.591E-5`          | `fits jpeg png` | `alasky.cds.unistra.fr`, `alaskybis.cds.unistra.fr` |

### Spectra

No 1D spectroscopic backends are configured for this survey.

## References and acknowledgement

- Chambers, K. C. et al. (2016), *The Pan-STARRS1 Surveys*, arXiv:1612.05560.
- Magnier, E. A. et al. (2020), *Pan-STARRS Photometric and Astrometric Calibration*, *ApJS*, 251, 6 (`doi:10.3847/1538-4365/abb82a`).
- Waters, C. Z. et al. (2020), *Pan-STARRS Data Release 2 Processing*, *ApJS*, 251, 4 (`doi:10.3847/1538-4365/abb82b`).
- Flewelling, H. A. et al. (2020), *The Pan-STARRS1 Database and Data Products*, *ApJS*, 251, 7 (`doi:10.3847/1538-4365/abb82d`).
- CDS HiPS records: `CDS/P/PanSTARRS/DR1/g`, `CDS/P/PanSTARRS/DR1/r`, `CDS/P/PanSTARRS/DR1/z`.

The Pan-STARRS1 Surveys (PS1) and the PS1 public science archive have been made possible through contributions by the Institute for Astronomy, the University of Hawaii, the Pan-STARRS Project Office, the Max-Planck Society and its participating institutes, the Johns Hopkins University, Durham University, the University of Edinburgh, the Queen’s University Belfast, the Harvard-Smithsonian Center for Astrophysics, the Las Cumbres Observatory Global Telescope Network Incorporated, the National Central University of Taiwan, the Space Telescope Science Institute, the National Aeronautics and Space Administration under Grant No. NNX08AR22G issued through the Planetary Science Division of the NASA Science Mission Directorate, the National Science Foundation Grant No. AST-1238877, the University of Maryland, Eotvos Lorand University (ELTE), the Los Alamos National Laboratory, and the Gordon and Betty Moore Foundation.

## Provenance of this page

| Source ID         | Kind        | Service / Endpoint                               | Query or URL                                                                                 | Retrieved (UTC)        |
|-------------------|-------------|--------------------------------------------------|----------------------------------------------------------------------------------------------|------------------------|
| `catalog_ps1_dr1` | `tap_table` | `https://tapvizier.cds.unistra.fr/TAPVizieR/tap` | `https://vizier.cds.unistra.fr/viz-bin/votable?-source=II/349/ps1&-out.max=0&-out.all`       | `2026-10-18T16:59:07Z` |
| `hips_ps1_dr1`    | `hips`      | `https://alasky.cds.unistra.fr/MocServer/query`  | `https://alasky.cds.unistra.fr/MocServer/query?ID=CDS/P/PanSTARRS/DR1/g&fmt=json&get=record` | `2026-10-18T16:36:33Z` |
<!-- /generated -->
