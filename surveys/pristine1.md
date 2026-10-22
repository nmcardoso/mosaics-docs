# Pristine DR1 (`pristine1`)

<!-- generated:survey-page -->

The Pristine survey is a narrowband optical survey conducted with the MegaPrime/MegaCam wide-field imager on the 3.6m Canada-France-Hawaii Telescope (CFHT) atop Maunakea. The survey utilizes a specialized narrowband filter centered on the stellar Ca II H & K lines ($\lambda \sim 3933$ Å and $3968$ Å) to photometrically identify metal-poor stars across the Milky Way halo and disc. Pristine Data Release 1 (DR1) covers over 6,500 deg² of the northern sky.

In `mosaics`, `pristine1` represents a cutout-only survey configuration. However, because no public image HiPS or image cutout service is currently published by CDS or upstream archives for Pristine imaging, the cutout adapter is maintained in a disabled state as an explicit architectural placeholder.

| Property                 | Value                                                                      |
|--------------------------|----------------------------------------------------------------------------|
| What MOSAICS extracts    | Cutouts: `hips` (color)                                                    |
| Archives                 | CDS HiPS / hips2fits (`alasky.cds.unistra.fr`), `tapvizier.cds.unistra.fr` |
| Bands / wavelength range | `color`                                                                    |
| Used by                  | `mosaics-cutout full`                                                      |

## Configuration

### The shipped file

Source: `defs/surveys/pristine1.yaml`

```yaml
# Pristine DR1 -- cutouts only, HiPS tiles.

survey_slug: pristine1
name: Pristine DR1

cutout:
  - adapter: hips
    # DOES NOT RESOLVE. Kept so the survey is described rather than
    # forgotten, and so the next person does not re-derive the same
    # guess. `--plan` will not reach it: the adapter is disabled. No
    # single-band sibling exists either -- see the note below -- so
    # `color` is the only key this survey has.
    adapter_params:
      hips_id: CDS/P/Pristine/DR1
    bands: [color]
    enabled: false

photometric_system:
  pixel_unit: unknown
  pixel_unit_note: >-
    Moot, because there are no tiles. CDS publishes no image HiPS for Pristine at all -- only the VizieR catalogue tables J/A+A/692/A115/pristine_dr1 and its siblings (MOCServer, 2026-09-04). The
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

- **Cutout `None` (`adapter: hips`, bands `['color']`):** configured `adapter_params`: `hips_id='CDS/P/Pristine/DR1'`.

### Common changes

- Enable an alternate cutout backend: if an active image cutout service or custom HiPS repository becomes available, configure the provider under `overrides[surveys/pristine1].cutout` and set `enabled: true`.
- Disable Pristine in entrypoint configurations: to omit `pristine1` completely from batch cutout runs, override the cutout list in your entrypoint YAML with:
  ```yaml
  overrides:
    surveys/pristine1:
      cutout: []
  ```

### Commands

```console
$ uv run -m mosaics.cli.cutout full -t test/objects_sample_1k.csv -o cutouts.lance --config defs/cutout.yaml --survey pristine1 --plan
```

## Data served by the archive

- Disabled cutout adapter placeholder: CDS publishes no image HiPS tiles for Pristine DR1—only the VizieR catalogue tables `J/A+A/692/A115/pristine_dr1` and associated tables (checked via CDS MOCServer, 2026-09-04). The survey definition preserves `hips_id: CDS/P/Pristine/DR1` with `enabled: false` so that the survey is documented rather than forgotten, preventing repeated searches for non-existent endpoints.
- Band definition: Because no single-band image HiPS exists, `color` is the sole key defined for this cutout adapter.
- Pixel scale and unit: The pixel unit is declared as `unknown` because no image tiles are currently accessible to resolve a zero point or physical unit. Any future cutout adapter integration will require a dedicated pixel scale and calibration validation.
- Pipeline execution: Running `mosaics-cutout` with `--plan` will not attempt to fetch Pristine tiles because `enabled: false` is honored by the plan generator.

### Catalogue: `J/A+A/692/A115/pristine_dr1`

Pristine DR1 metallicity catalogue (table 3) (6380652 sources)

**Service URL:** `https://tapvizier.cds.unistra.fr/TAPVizieR/tap`

| Column          | Type     | Unit   | UCD                     | Description                                                                                                                                               |
|-----------------|----------|--------|-------------------------|-----------------------------------------------------------------------------------------------------------------------------------------------------------|
| `GaiaDR3`       | `long`   | `—`    | `meta.id;meta.main`     | Gaia DR3 source_id (source_id)                                                                                                                            |
| `RAICRS`        | `double` | `deg`  | `pos.eq.ra;meta.main`   | Gaia DR3 right ascension ICRS (J2016) (RA)                                                                                                                |
| `DEICRS`        | `double` | `deg`  | `pos.eq.dec;meta.main`  | Gaia DR3 declination ICRS (J2016) (Dec)                                                                                                                   |
| `E(B-V)`        | `float`  | `mag`  | `phot.color.excess`     | Schlegel et al. (1998ApJ…500..525S) extinction value (E(B-V))                                                                                             |
| `Flag`          | `short`  | `—`    | `meta.code.error`       | Morphology and quality flag10 (merged_CASU_flag) (G1)                                                                                                     |
| `Gmag0`         | `float`  | `mag`  | `phot.mag;em.opt`       | De-reddened Gaia G magnitude (G_0)                                                                                                                        |
| `Bpmag0`        | `float`  | `mag`  | `phot.mag;em.opt.B`     | De-reddened Gaia GBP magnitude (BP_0)                                                                                                                     |
| `Rpmag0`        | `float`  | `mag`  | `phot.mag;em.opt.R`     | De-reddened Gaia GRP magnitude (RP_0)                                                                                                                     |
| `CaHKmag0`      | `float`  | `mag`  | `phot.mag;em.opt`       | De-reddened Pristine CaHK magnitude (CaHK_0)                                                                                                              |
| `[Fe/H]phPgm`   | `float`  | `—`    | `phys.abund.Fe`         | ? Photometric metallicity [Fe/H]phot based on the Pristine CaHK magnitudes and using the (favored) giant model (FeH_Pristine)                             |
| `[Fe/H]phP`     | `float`  | `—`    | `phys.abund.Fe`         | ? Median value of FeH_Pristine based on the Monte Carlo sampling of the magnitude uncertainties (FeH_Pristine_50th)                                       |
| `MCfCaHKP`      | `float`  | `—`    | `meta.note`             | Fraction of the Monte Carlo samples of the magnitude uncertainties that fall outside the model color-color space (mcfrac_Pristine)                        |
| `Gmag0dw`       | `float`  | `mag`  | `phot.mag;em.opt`       | De-reddened Gaia G magnitude for the dwarf model (G_0_dw)                                                                                                 |
| `Bpmag0dw`      | `float`  | `mag`  | `phot.mag;em.opt.B`     | De-reddened Gaia GBP magnitude for the dwarf model (BP_0_dw)                                                                                              |
| `Rpmag0dw`      | `float`  | `mag`  | `phot.mag;em.opt.R`     | De-reddened Gaia GRP magnitude for the dwarf model (RP_0_dw)                                                                                              |
| `CaHKmag0dw`    | `float`  | `mag`  | `phot.mag;em.opt`       | De-reddened Pristine-like CaHKsyn magnitude (CaHK_0_dw)                                                                                                   |
| `[Fe/H]phPgmdw` | `float`  | `—`    | `phys.abund.Fe`         | ? Photometric metallicity [Fe/H]phot based on the Pristine CaHK magnitudes and using the (favored) giant model for the dwarf model (FeH_Pristine_dw)      |
| `[Fe/H]phPdw`   | `float`  | `—`    | `phys.abund.Fe`         | ? Median value of FeH_Pristine based on the Monte Carlo sampling of the magnitude uncertainties for the dwarf model (FeH_Pristine_dw_50th)                |
| `MCfCaHKPdw`    | `float`  | `—`    | `meta.note`             | Fraction of the Monte Carlo samples of the magnitude uncertainties that fall outside the model color-color space for the dwarf model (mcfrac_Pristine_dw) |
| `RUWE`          | `float`  | `—`    | `stat.error`            | ? Gaia DR3 Renormalised Unit Weight Error (RUWE)                                                                                                          |
| `Pvar`          | `float`  | `—`    | `stat.fit.goodness`     | Probability for the source to be variable, as defined in Section 5 (Pvar)                                                                                 |
| `Cstar`         | `double` | `mag`  | `phot.flux;arith.ratio` | Gaia DR3 corrected flux excess, C\*, as defined in equation 6 of Riello et al. (2021A&A…649A…3R) (Cstar)                                                  |
| `s_Cstar`       | `double` | `mag`  | `stat.stdev;phot.mag`   | Normalized standard deviation of C\* for the G magnitude of this source, as defined in equation 18 of Riello et al. (2021A&A…649A…3R) (Cstar_1sigma)      |

### Images

- **Pixel unit:** `unknown` | **AB zero point:** `—` | **Calibration note:** Moot, because there are no tiles. CDS publishes no image HiPS for Pristine at all – only the VizieR catalogue tables J/A+A/692/A115/pristine_dr1 and its siblings (MOCServer, 2026-09-04). The pixel unit cannot be resolved for a service that does not exist, and this entry is a record of having looked rather than a working adapter. Whoever finds a real HiPS for this survey – or another cutout service – should replace the adapter below and revisit this note; the tiles would then need a scale check of their own, as no HiPS build this project has seen states one.
- **Cutout service `None`:** adapter `hips`, bands `['color']`, params `{'hips_id': 'CDS/P/Pristine/DR1'}`.

### Spectra

No 1D spectroscopic backends are configured for this survey.

## References and acknowledgement

- Martin, N. F. et al. (2024), *The Pristine survey — XXIII. Data Release 1 and an all-sky metallicity catalogue based on Gaia DR3 BP/RP spectro-photometry*, *A&A*, 692, A115 (`doi:10.1051/0004-6361/202450849`).
- Starkenburg, E. et al. (2017), *The Pristine survey — I. Mining the Milky Way halo for most metal-poor stars*, *MNRAS*, 471, 2587 (`doi:10.1093/mnras/stx1068`).
- VizieR catalogue: `J/A+A/692/A115` (`https://vizier.cds.unistra.fr/viz-bin/VizieR?-source=J/A+A/692/A115`).

Based on observations obtained with MegaPrime/MegaCam, a joint project of CFHT and CEA/IRFU, at the Canada-France-Hawaii Telescope (CFHT) which is operated by the National Research Council (NRC) of Canada, the Institut National des Sciences de l’Univers of the Centre National de la Recherche Scientifique (CNRS) of France, and the University of Hawaii. The authors acknowledge the sacred nature of Maunakea and its cultural significance to the Indigenous Hawaiian people.

## Provenance of this page

| Source ID              | Kind        | Service / Endpoint                               | Query or URL                                                                                            | Retrieved (UTC)        |
|------------------------|-------------|--------------------------------------------------|---------------------------------------------------------------------------------------------------------|------------------------|
| `catalog_pristine_dr1` | `tap_table` | `https://tapvizier.cds.unistra.fr/TAPVizieR/tap` | `https://vizier.cds.unistra.fr/viz-bin/votable?-source=J/A+A/692/A115/pristine_dr1&-out.max=0&-out.all` | `2026-10-18T16:59:02Z` |
| `hips_pristine_dr1`    | `hips`      | `https://alasky.cds.unistra.fr/MocServer/query`  | `https://alasky.cds.unistra.fr/MocServer/query?ID=CDS/P/Pristine/DR1&fmt=json&get=record`               | `2026-10-18T16:59:03Z` |
<!-- /generated -->
