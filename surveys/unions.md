# UNIONS DR5 (`unions`)

<!-- generated:survey-page -->

The Ultraviolet Near-Infrared Optical Northern Survey (UNIONS) is a collaborative deep imaging survey of the northern sky conducted by the Canada-France-Hawaii Telescope (CFHT) and the Panoramic Survey Telescope and Rapid Response System (Pan-STARRS). The CFHT component, known as the Canada-France Imaging Survey (CFIS), supplies deep optical imaging—most notably in $u$ and $r$ bands—using MegaCam on the 3.6-metre CFHT on Maunakea. UNIONS Data Release 5 (DR5) represents a key public milestone for this high-resolution northern dataset.

In `mosaics`, `unions` is configured as a cutout-only survey entry. Currently, it serves as an explicit architectural placeholder for CFIS/UNIONS $r$-band imaging (`CDS/P/UNIONS/DR5/r`) with `enabled: false`.

| Property                 | Value                                                                      |
|--------------------------|----------------------------------------------------------------------------|
| What MOSAICS extracts    | Cutouts: `hips` (color)                                                    |
| Archives                 | CDS HiPS / hips2fits (`alasky.cds.unistra.fr`), `tapvizier.cds.unistra.fr` |
| Bands / wavelength range | `color`                                                                    |
| Used by                  | `mosaics-cutout full`                                                      |

## Configuration

### The shipped file

Source: `defs/surveys/unions.yaml`

```yaml
# UNIONS DR5 -- cutouts only, HiPS tiles, r band.

survey_slug: unions
name: UNIONS DR5

cutout:
  - adapter: hips
    # DOES NOT RESOLVE. Kept so the survey is described rather than
    # forgotten, and so the next person does not re-derive the same
    # guess. `--plan` will not reach it: the adapter is disabled. No
    # single-band sibling exists either -- see the note below -- so
    # `color` is the only key this survey has.
    adapter_params:
      hips_id: CDS/P/UNIONS/DR5/r
    bands: [color]
    enabled: false

photometric_system:
  pixel_unit: unknown
  pixel_unit_note: >-
    Moot, because there are no tiles. no HiPS matching UNIONS or CFIS is registered (MOCServer, 2026-09-04). The
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

- **Cutout `None` (`adapter: hips`, bands `['color']`):** configured `adapter_params`: `hips_id='CDS/P/UNIONS/DR5/r'`.

### Common changes

- **Retain disabled state**: In normal operation, leave the cutout adapter disabled (`enabled: false`) because no public HiPS tiles are currently served for UNIONS on CDS MOCServer.
- **Enable an alternate cutout endpoint**: If a working private or institutional HiPS service or custom cutout adapter becomes available, configure its parameters in an entrypoint YAML:
  ```yaml
  overrides:
    surveys/unions:
      cutout:
        - adapter: hips
          adapter_params:
            hips_id: CDS/P/UNIONS/DR5/r
            service_url: https://your-hips-service.org/
          bands: [color]
          enabled: true
  ```
- **Omit survey from cutout planning**: To prevent `unions` from appearing in cutout planning runs:
  ```yaml
  overrides:
    surveys/unions:
      cutout: []
  ```

### Commands

```console
$ uv run -m mosaics.cli.cutout full -t test/objects_sample_1k.csv -o cutouts.lance --config defs/cutout.yaml --survey unions --plan
```

## Data served by the archive

- **Disabled HiPS status**: As of 2026-09-04, no HiPS dataset matching UNIONS or CFIS is registered on the CDS MOCServer (`CDS/P/UNIONS/DR5/r` does not resolve). The entry in `defs/surveys/unions.yaml` is retained in a disabled state (`enabled: false`) to document the survey configuration and prevent redundant investigations into missing endpoints.
- **Pipeline planning**: When running `mosaics-cutout` with `--plan`, the adapter is bypassed because `enabled: false` is respected by the pipeline generator, avoiding failing network requests.
- **Bands and pixel units**: The survey definition specifies band `[color]` because no single-band sibling HiPS exists. The pixel unit is declared as `unknown` because physical pixel scale and photometric calibration cannot be established without accessible image tiles. If an active endpoint is integrated in the future, the tile scale and zero point must be verified independently.

### Catalogue: `II/317/cfhtls_w`

the CFHTLS Wide Survey (out of 35,651,677 sources)

**Service URL:** `https://tapvizier.cds.unistra.fr/TAPVizieR/tap`

| Column    | Type           | Unit   | UCD                            | Description                                                                       |
|-----------|----------------|--------|--------------------------------|-----------------------------------------------------------------------------------|
| `recno`   | `int`          | `—`    | `meta.record`                  | Record number assigned by the VizieR team. Should Not be used for identification. |
| `CFHTLS`  | `char`         | `—`    | `meta.id;meta.main`            | Unique object identifier (CFHTLS_INDEX) (G1)                                      |
| `RAJ2000` | `double`       | `deg`  | `pos.eq.ra;meta.main`          | Right ascension (J2000) (ALPHA_J2000)                                             |
| `DEJ2000` | `double`       | `deg`  | `pos.eq.dec;meta.main`         | Declination (J2000) (DELTA_J2000)                                                 |
| `umag`    | `float`        | `mag`  | `phot.mag;em.opt.U`            | ? u\*-band 3”-aperture AB mag. (MAG_APER_u)                                       |
| `e_umag`  | `float`        | `mag`  | `stat.error;phot.mag;em.opt.U` | ? RMS error in umag (MAGERR_APER_u)                                               |
| `gmag`    | `float`        | `mag`  | `phot.mag;em.opt.B`            | ? g-band 3”-aperture AB mag. (MAG_APER_g)                                         |
| `e_gmag`  | `float`        | `mag`  | `stat.error;phot.mag;em.opt.B` | ? RMS error in gmag (MAGERR_APER_g)                                               |
| `rmag`    | `float`        | `mag`  | `phot.mag;em.opt.R`            | ? r-band 3”-aperture AB mag. (MAG_APER_r)                                         |
| `e_rmag`  | `float`        | `mag`  | `stat.error;phot.mag;em.opt.R` | ? RMS error in rmag (MAGERR_APER_r)                                               |
| `imag`    | `float`        | `mag`  | `phot.mag;em.opt.I`            | ? i/y-band 3”-aperture AB mag. (MAG_APER_i) (2)                                   |
| `e_imag`  | `float`        | `mag`  | `stat.error;phot.mag;em.opt.I` | ? RMS error in imag (MAGERR_APER_i)                                               |
| `zmag`    | `float`        | `mag`  | `phot.mag;em.opt.I`            | ? z-band 3”-aperture AB mag. (MAG_APER_z)                                         |
| `e_zmag`  | `float`        | `mag`  | `stat.error;phot.mag`          | ? RMS error in zmag (MAGERR_APER_z)                                               |
| `Sfl`     | `short`        | `—`    | `meta.code`                    | [0/63] Sextractor extraction flags (FLAGS) (G3)                                   |
| `Tfl`     | `short`        | `—`    | `meta.code`                    | [0/255] TERAPIX flag mask (FLAG_MASK) (G4)                                        |
| `ifl`     | `unsignedByte` | `—`    | `meta.code`                    | [0/1] i-filter used, 1=y (FLAG_IFILTER) (2)                                       |

### Images

- **Pixel unit:** `unknown` | **AB zero point:** `—` | **Calibration note:** Moot, because there are no tiles. no HiPS matching UNIONS or CFIS is registered (MOCServer, 2026-09-04). The pixel unit cannot be resolved for a service that does not exist, and this entry is a record of having looked rather than a working adapter. Whoever finds a real HiPS for this survey – or another cutout service – should replace the adapter below and revisit this note; the tiles would then need a scale check of their own, as no HiPS build this project has seen states one.
- **Cutout service `None`:** adapter `hips`, bands `['color']`, params `{'hips_id': 'CDS/P/UNIONS/DR5/r'}`.

### Spectra

No 1D spectroscopic backends are configured for this survey.

## References and acknowledgement

- UNIONS Collaboration: [https://www.skysurveys.org/unions/](https://www.skysurveys.org/unions/)
- Ibata, R. et al. (2017), *The Canada-France Imaging Survey: First Results from the u-Band Component*, *ApJ*, 848, 129 (`doi:10.3847/1538-4357/aa8cd0`).
- CDS MOCServer query service: [https://alasky.cds.unistra.fr/MocServer/query](https://alasky.cds.unistra.fr/MocServer/query)

Based on observations obtained with MegaPrime/MegaCam, a joint project of CFHT and CEA/DAPNIA, at the Canada-France-Hawaii Telescope (CFHT) which is operated by the National Research Council (NRC) of Canada, the Institut National des Sciences de l’Univers of the Centre National de la Recherche Scientifique (CNRS) of France, and the University of Hawaii. The authors acknowledge the sacred nature of Maunakea and its cultural significance to the Indigenous Hawaiian people.

## Provenance of this page

| Source ID             | Kind        | Service / Endpoint                               | Query or URL                                                                                | Retrieved (UTC)        |
|-----------------------|-------------|--------------------------------------------------|---------------------------------------------------------------------------------------------|------------------------|
| `catalog_cfhtls_wide` | `tap_table` | `https://tapvizier.cds.unistra.fr/TAPVizieR/tap` | `https://vizier.cds.unistra.fr/viz-bin/votable?-source=II/317/cfhtls_w&-out.max=0&-out.all` | `2026-10-18T16:59:47Z` |
| `hips_unions_dr5`     | `hips`      | `https://alasky.cds.unistra.fr/MocServer/query`  | `https://alasky.cds.unistra.fr/MocServer/query?ID=CDS/P/UNIONS/DR5/r&fmt=json&get=record`   | `2026-10-18T16:59:48Z` |
<!-- /generated -->
