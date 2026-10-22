# Sloan Digital Sky Survey DR20 (`sdss20`)

<!-- generated:survey-page -->

The Sloan Digital Sky Survey DR20 (`sdss20`) survey entry in MOSAICS is dedicated exclusively to image cutout extraction across the northern and equatorial sky. The survey covers optical imaging in the $g, r, i$ passbands observed with the 2.5m telescope at Apache Point Observatory.

In `mosaics`, `sdss20` provides image cutouts through the Legacy Survey Viewer (`lsv`) backend via its `sdss` coadd layer, outputting calibrated image stamps in nanomaggies. Additional cutout adapters for Astro Data Lab Simple Image Access (SIA) and CDS HiPS collections are declared in the survey configuration as alternate backends.

| Property                 | Value                                                                                                                                              |
|--------------------------|----------------------------------------------------------------------------------------------------------------------------------------------------|
| What MOSAICS extracts    | Cutouts: `lsv` (g,r,i), `sia` (g), `hips` (g), `hips` (r), `hips` (i)                                                                              |
| Archives                 | Legacy Survey Viewer (`legacysurvey.org`), SIA (`datalab.noirlab.edu`), CDS HiPS / hips2fits (`alasky.cds.unistra.fr`), `tapvizier.cds.unistra.fr` |
| Bands / wavelength range | borrows passbands from `sdss16`                                                                                                                    |
| Used by                  | `mosaics-cutout full`                                                                                                                              |

## Configuration

### The shipped file

Source: `defs/surveys/sdss20.yaml`

```yaml
# SDSS DR20 -- cutouts.
#
# Both catalogue halves that used to sit here are surveys of their own now. The
# spectroscopy is `sdss17.yaml`, answered by Astro Data Lab. The photometry is
# `sdss16.yaml`: it was crossmatched through SkyServer's CrossID form, which
# failed most batches of a restricted run, and now comes from VizieR's HATS
# build of V/154 instead. What remains is the imaging the Legacy Survey Viewer
# serves, which borrows that survey's passbands.

survey_slug: sdss20
name: Sloan Digital Sky Survey DR20

# max_mag: 22.2 (max_mag_band: r). 5 sigma point source, single-epoch
# imaging, AB (SDSS ugriz 22.0/22.2/22.2/21.3/20.5). The photometry in
# `sdss16.yaml` is asinh (Lupton) magnitudes, which are AB, so no conversion
# is needed.
cutout:
  - adapter: lsv          # VERIFIED LIVE 2026-08-25
    adapter_params:
      layer: sdss
      bands: gri
    bands: [g, r, i]
  # Three keys for the three bands the stack fills, each with more than one
  # backend: that is the point of the mapping -- interchangeable backends
  # for one column, and the router picks between them. The HiPS ids are
  # SDSS9, verified registered with FITS tiles (MOCServer, 2026-09-05);
  # they are an earlier release than the DR20 this file is named for, and
  # `pixel_unit: nanomaggy` describes the Viewer rather than those tiles,
  # so they stay disabled until one is checked against the other.
  - adapter: sia
    adapter_params:
      service_url: https://datalab.noirlab.edu/sia/sdss_dr9
    bands: [g]
    enabled: false
  - adapter: hips
    adapter_params:
      hips_id: CDS/P/SDSS9/g
    bands: [g]
    enabled: false
  - adapter: hips
    adapter_params:
      hips_id: CDS/P/SDSS9/r
    bands: [r]
    enabled: false
  - adapter: hips
    adapter_params:
      hips_id: CDS/P/SDSS9/i
    bands: [i]
    enabled: false

photometric_system:
  pixel_unit: nanomaggy
  pixel_unit_note: The Legacy Survey Viewer serves SDSS in nanomaggies.
  borrow_filters: sdss16      # the same ugriz passbands, one catalogue release apart
```

### Schema of this file

| Key present          | Type                   | Required   | Default   | Meaning                                                                                            |
|----------------------|------------------------|------------|-----------|----------------------------------------------------------------------------------------------------|
| `survey_slug`        | str                    | yes        | —         | Canonical lowercase identifier for the survey and default column prefix.                           |
| `name`               | str                    | yes        | —         | Human-readable survey and data-release name used in logs, plans, and reports.                      |
| `cutout`             | list of cutout entries | no         | []        | Image cutout backends (`hips`, `lsv`, `nersc`, `sia`, `vsa`) serving this survey’s passbands.      |
| `photometric_system` | mapping                | no         | None      | Passband SVO filter identifiers, flux/magnitude homogenisation rules, and image pixel calibration. |

### Adapter parameters

- **Cutout `None` (`adapter: lsv`, bands `['g', 'r', 'i']`):** configured `adapter_params`: `layer='sdss'`, `bands='gri'`.
- **Cutout `None` (`adapter: sia`, bands `['g']`):** configured `adapter_params`: `service_url='https://datalab.noirlab.edu/sia/sdss_dr9'`.
- **Cutout `None` (`adapter: hips`, bands `['g']`):** configured `adapter_params`: `hips_id='CDS/P/SDSS9/g'`.
- **Cutout `None` (`adapter: hips`, bands `['r']`):** configured `adapter_params`: `hips_id='CDS/P/SDSS9/r'`.
- **Cutout `None` (`adapter: hips`, bands `['i']`):** configured `adapter_params`: `hips_id='CDS/P/SDSS9/i'`.

### Common changes

- Disable SDSS cutouts in an entrypoint configuration by setting:
  ```yaml
  overrides:
    surveys/sdss20:
      cutout: []
  ```
- Select specific passbands on the command line using `--bands sdss20:g,sdss20:r` or by overriding `bands: [g, r]` in an entrypoint YAML.
- Enable alternative cutout backends: activate the Astro Data Lab SIA endpoint or CDS HiPS layers by setting `enabled: true` on the respective block under `overrides[surveys/sdss20].cutout`.

### Commands

```console
$ uv run -m mosaics.cli.cutout full -t test/objects_sample_1k.csv -o cutouts.lance --config defs/cutout.yaml --survey sdss20 --plan
```

## Data served by the archive

- Architectural separation of SDSS: SDSS data products are split across three survey configurations in MOSAICS: `sdss16` (photometry), `sdss17` (spectroscopy), and `sdss20` (cutouts). Formerly, all three were declared in `sdss20.yaml`. The catalogue crossmatch via SkyServer CrossID proved unreliable in restricted runs, so photometry moved to VizieR HATS (`sdss16`) and spectroscopy moved to local SAS FITS querying (`sdss17`). `sdss20` retains the image cutout functionality.
- Pixel units and calibration: The Legacy Survey Viewer serves SDSS cutouts in nanomaggies ($1\ \text{nanomaggy} = 3.631 \times 10^{-6}\ \text{Jy}$, corresponding to an AB zero point of 22.5 mag). Passband definitions and filter curves are borrowed directly from `sdss16`.
- Survey depth: 5-sigma point source depths in single-epoch SDSS imaging, AB: $u = 22.0, g = 22.2, r = 22.2, i = 21.3, z = 20.5$ mag. The $r$-band depth of 22.2 mag is quoted as the representative depth in MOSAICS.
- Inactive HiPS and SIA backends: CDS publishes SDSS DR9 HiPS tiles (`CDS/P/SDSS9/g`, `r`, `i` at order 10, pixel scale $1.118 \times 10^{-4}$ deg, sky fraction ~0.36–0.39), and Astro Data Lab hosts an IVOA SIA service for SDSS DR9. Because `pixel_unit: nanomaggy` describes the Legacy Survey Viewer mosaics rather than these DR9 FITS tiles, they remain disabled pending empirical calibration validation.

### Catalogue: `V/154/sdss16`

* output of the SDSS photometric catalog

**Service URL:** `https://tapvizier.cds.unistra.fr/TAPVizieR/tap`

| Column      | Type           | Unit     | UCD                            | Description                                                                                                                                               |
|-------------|----------------|----------|--------------------------------|-----------------------------------------------------------------------------------------------------------------------------------------------------------|
| `objID`     | `long`         | `—`      | `meta.id;meta.main`            | SDSS unique object identifier (1)                                                                                                                         |
| `RA_ICRS`   | `double`       | `deg`    | `pos.eq.ra;meta.main`          | Right Ascension of the object (ICRS) at Epoch=”MJD” (ra)                                                                                                  |
| `DE_ICRS`   | `double`       | `deg`    | `pos.eq.dec;meta.main`         | Declination of the object (ICRS) at Epoch=”MJD” (dec)                                                                                                     |
| `mode`      | `unsignedByte` | `—`      | `meta.code.class`              | [1/4] PhotoMode (1=primary, 2=secondary, 3=family, 4=outside)                                                                                             |
| `class`     | `unsignedByte` | `—`      | `src.class`                    | [0/8]? Type of object (3=galaxy, 6=star) (type) (2)                                                                                                       |
| `clean`     | `unsignedByte` | `—`      | `meta.code`                    | [0/1] Clean photometry flag (1=clean; 0=unclean)                                                                                                          |
| `e_RA_ICRS` | `double`       | `arcsec` | `stat.error;pos.eq.ra`         | [0/51703] Mean error on RAdeg (raErr)                                                                                                                     |
| `e_DE_ICRS` | `double`       | `arcsec` | `stat.error;pos.eq.dec`        | [0/22795] Mean error on DEdeg (decErr)                                                                                                                    |
| `umag`      | `float`        | `mag`    | `phot.mag;em.opt.U`            | [4/37.3]? Model magnitude in u filter AB scale (u) (7)                                                                                                    |
| `gmag`      | `float`        | `mag`    | `phot.mag;em.opt.B`            | [3.7/39.6]? Model magnitude in g filter AB scale (g) (7)                                                                                                  |
| `rmag`      | `float`        | `mag`    | `phot.mag;em.opt.R`            | [3.8/40.4]? Model magnitude in r filter AB scale (r) (7)                                                                                                  |
| `imag`      | `float`        | `mag`    | `phot.mag;em.opt.I`            | [2.6/39.5]? Model magnitude in i filter AB scale (i) (7)                                                                                                  |
| `zmag`      | `float`        | `mag`    | `phot.mag;em.opt.I`            | [3.5/37.2]? Model magnitude in z filter AB scale (z) (7)                                                                                                  |
| `e_umag`    | `double`       | `mag`    | `stat.error;phot.mag`          | [8.9e-8/3444]? Mean error on umag (err_u)                                                                                                                 |
| `e_gmag`    | `double`       | `mag`    | `stat.error;phot.mag`          | [3.8e-8/5955]? Mean error on gmag (err_g)                                                                                                                 |
| `e_rmag`    | `double`       | `mag`    | `stat.error;phot.mag`          | [5.2e-8/915]? Mean error on rmag (err_r)                                                                                                                  |
| `e_imag`    | `double`       | `mag`    | `stat.error;phot.mag`          | [0/634]? Mean error on imag (err_i)                                                                                                                       |
| `e_zmag`    | `double`       | `mag`    | `stat.error;phot.mag`          | [9.4e-8/3529]? Mean error on zmag (err_z)                                                                                                                 |
| `zsp`       | `double`       | `—`      | `src.redshift`                 | [-0.012/7.06]? Spectroscopic final redshift (when SpObjID>0) (spec_z) (6)                                                                                 |
| `e_zsp`     | `double`       | `—`      | `stat.error`                   | [-6/30689]? Mean error on zsp (negative for bad fit) (spec_zErr) (6)                                                                                      |
| `f_zsp`     | `short`        | `—`      | `meta.code`                    | [0/902]? Zwarning flag (spec_zWarning) (6)(9)                                                                                                             |
| `zph`       | `float`        | `—`      | `src.redshift`                 | [5e-4/1]? Photometric redshift estimated by robust fit to nearest neighbors in a reference set (photoz_z) (12)                                            |
| `e_zph`     | `float`        | `—`      | `stat.error`                   | [0.003/0.3]? Estimated error of the photometric redshift (photoz_zErr) (12)                                                                               |
| `<zph>`     | `float`        | `—`      | `src.redshift.phot`            | [0.009/0.9]? Average redshift of the nearest neighbors; if significantly different from zph this might be a better estimate than zph (photoz_nnAvgZ) (12) |
| `Q`         | `unsignedByte` | `—`      | `meta.code.qual;obs.param;obs` | [1/3] Quality of the observation: 1=bad; 2=barely acceptable; 3=good (field_quality) (13)                                                                 |
| `SDSS16`    | `char`         | `—`      | `meta.id`                      | SDSS-DR16 name based on J2000 position; provided by CDS                                                                                                   |
| `Sp-ID`     | `char`         | `—`      | `meta.id`                      | ? Spectroscopic Plate-MJD-Fiber identifier (plate-mjd-fiberID)                                                                                            |
| `MJD`       | `double`       | `d`      | `time.epoch`                   | Mean observation date (average calculated by CDS from the uMJD, gMJD, rMJD, iMJD, zMJD)                                                                   |

### Images

- **Pixel unit:** `nanomaggy` | **AB zero point:** `—` | **Calibration note:** The Legacy Survey Viewer serves SDSS in nanomaggies.
- **Cutout service `None`:** adapter `lsv`, bands `['g', 'r', 'i']`, params `{'layer': 'sdss', 'bands': 'gri'}`.
- **Cutout service `None`:** adapter `sia`, bands `['g']`, params `{'service_url': 'https://datalab.noirlab.edu/sia/sdss_dr9'}`.
- **Cutout service `None`:** adapter `hips`, bands `['g']`, params `{'hips_id': 'CDS/P/SDSS9/g'}`.
- **Cutout service `None`:** adapter `hips`, bands `['r']`, params `{'hips_id': 'CDS/P/SDSS9/r'}`.
- **Cutout service `None`:** adapter `hips`, bands `['i']`, params `{'hips_id': 'CDS/P/SDSS9/i'}`.

| HiPS ID         | Regime   | Frame        | Order   | Pixel scale (deg)   | Tile format     | Mirrors                                             |
|-----------------|----------|--------------|---------|---------------------|-----------------|-----------------------------------------------------|
| `CDS/P/SDSS9/g` | Optical  | `equatorial` | `10`    | `1.118E-4`          | `png jpeg fits` | `alasky.cds.unistra.fr`, `alaskybis.cds.unistra.fr` |
| `CDS/P/SDSS9/r` | Optical  | `equatorial` | `10`    | `1.118E-4`          | `png jpeg fits` | `alasky.cds.unistra.fr`, `alaskybis.cds.unistra.fr` |
| `CDS/P/SDSS9/i` | Optical  | `equatorial` | `10`    | `1.118E-4`          | `png jpeg fits` | `alasky.cds.unistra.fr`, `alaskybis.cds.unistra.fr` |

### Spectra

No 1D spectroscopic backends are configured for this survey.

## References and acknowledgement

- York, D. G. et al. (2000), *The Sloan Digital Sky Survey: Technical Summary*, *AJ*, 120, 1579 (`doi:10.1086/301513`).
- Gunn, J. E. et al. (1998), *The Sloan Digital Sky Survey Photometric Camera*, *AJ*, 116, 3040 (`doi:10.1086/300645`).
- Legacy Survey Viewer: [https://www.legacysurvey.org/viewer](https://www.legacysurvey.org/viewer).
- CDS HiPS records: `CDS/P/SDSS9/g`, `CDS/P/SDSS9/r`, `CDS/P/SDSS9/i`.
- Astro Data Lab SDSS DR9 SIA: `https://datalab.noirlab.edu/sia/sdss_dr9`.

For the SDSS imaging: Funding for the SDSS and SDSS-II has been provided by the Alfred P. Sloan Foundation, the Participating Institutions, the National Science Foundation, the U.S. Department of Energy, the National Aeronautics and Space Administration, the Japanese Monbukagakusho, the Max Planck Society, and the Higher Education Funding Council for England. The SDSS is managed by the Astrophysical Research Consortium for the Participating Institutions.

For the Legacy Survey Viewer: The Legacy Surveys consist of three individual and complementary projects: the Dark Energy Camera Legacy Survey (DECaLS), the Mayall z-band Legacy Survey (MzLS), and the Beijing-Arizona Sky Survey (BASS). The Legacy Survey Viewer is hosted by the National Optical Astronomy Observatory (NOAO) and Lawrence Berkeley National Laboratory (LBNL).

## Provenance of this page

| Source ID               | Kind        | Service / Endpoint                               | Query or URL                                                                             | Retrieved (UTC)        |
|-------------------------|-------------|--------------------------------------------------|------------------------------------------------------------------------------------------|------------------------|
| `catalog_sdss_photoobj` | `tap_table` | `https://tapvizier.cds.unistra.fr/TAPVizieR/tap` | `https://vizier.cds.unistra.fr/viz-bin/votable?-source=V/154/sdss16&-out.max=0&-out.all` | `2026-10-18T16:59:24Z` |
| `hips_sdss9`            | `hips`      | `https://alasky.cds.unistra.fr/MocServer/query`  | `https://alasky.cds.unistra.fr/MocServer/query?ID=CDS/P/SDSS9/g&fmt=json&get=record`     | `2026-10-18T16:37:47Z` |
| `sia_sdss_dr9`          | `sia`       | `https://datalab.noirlab.edu/sia/sdss_dr9`       | `https://datalab.noirlab.edu/sia/sdss_dr9`                                               | `2026-10-18T16:37:47Z` |
<!-- /generated -->
