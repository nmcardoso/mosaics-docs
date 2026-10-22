# VISTA Kilo-degree Infrared Galaxy Survey (`viking`)

<!-- generated:survey-page -->

The VISTA Kilo-degree Infrared Galaxy Survey (VIKING) is an intermediate-depth, near-infrared survey covering approximately 1,500 deg² of the extragalactic sky in two distinct stripes (one northern, one southern) in the $Z, Y, J, H, K_s$ passbands. Observations were conducted using VIRCAM on the ESO 4.1-metre VISTA telescope at Cerro Paranal, achieving 5-sigma point-source depths of $Z = 22.4$, $Y = 21.4$, $J = 20.9$, $H = 19.9$, and $K_s = 19.3$ mag (Vega). VIKING is roughly two magnitudes deeper than the wide-angle VHS over its selected footprint.

In `mosaics`, `viking` provides near-infrared image cutouts across its five passbands (`z, y, j, h, ks`) via the `vsa` adapter querying the VISTA Science Archive (VSA) Simple Image Access endpoint (`http://wfaudata.roe.ac.uk/vista-siap/`). Because it shares the VSA service URL with `vhs5`, it is shipped in a disabled state (`enabled: false`) to avoid redundant simultaneous requests.

| Property                 | Value                                                  |
|--------------------------|--------------------------------------------------------|
| What MOSAICS extracts    | Cutouts: `vsa` (z,y,j,h,ks)                            |
| Archives                 | VSA (`wfaudata.roe.ac.uk`), `tapvizier.cds.unistra.fr` |
| Bands / wavelength range | borrows passbands from `vhs5`                          |
| Used by                  | `mosaics-cutout full`                                  |

## Configuration

### The shipped file

Source: `defs/surveys/viking.yaml`

```yaml
# VIKING -- cutouts only. Shares the VISTA SIA service with vhs5.

survey_slug: viking
name: VISTA Kilo-degree Infrared Galaxy Survey

# max_mag: 20.9 (max_mag_band: j). 5 sigma point source, VEGA, as vhs5: ZYJHKs
# 22.4/21.4/20.9/19.9/19.3. Roughly two magnitudes deeper than VHS over a far
# smaller footprint, which is the trade the two VISTA surveys make.
cutout:
  - adapter: vsa
    adapter_params:
      service_url: http://wfaudata.roe.ac.uk/vista-siap/
      bands: Z,Y,J,H,Ks     # VIRCAM spelling; the columns are z/y/j/h/ks
    bands: [z, y, j, h, ks]
    enabled: false        # same service as vhs5; enable one of the two

photometric_system:
  pixel_unit: unknown
  pixel_unit_note: As vhs5 -- VSA frames carry a per-image MAGZPT.
  borrow_filters: vhs5        # VIRCAM, the same passbands vhs5 lists
```

### Schema of this file

| Key present          | Type                   | Required   | Default   | Meaning                                                                                            |
|----------------------|------------------------|------------|-----------|----------------------------------------------------------------------------------------------------|
| `survey_slug`        | str                    | yes        | —         | Canonical lowercase identifier for the survey and default column prefix.                           |
| `name`               | str                    | yes        | —         | Human-readable survey and data-release name used in logs, plans, and reports.                      |
| `cutout`             | list of cutout entries | no         | []        | Image cutout backends (`hips`, `lsv`, `nersc`, `sia`, `vsa`) serving this survey’s passbands.      |
| `photometric_system` | mapping                | no         | None      | Passband SVO filter identifiers, flux/magnitude homogenisation rules, and image pixel calibration. |

### Adapter parameters

- **Cutout `None` (`adapter: vsa`, bands `['z', 'y', 'j', 'h', 'ks']`):** configured `adapter_params`: `service_url='http://wfaudata.roe.ac.uk/vista-siap/'`, `bands='Z,Y,J,H,Ks'`.

### Common changes

- **Enable VIKING cutouts**: To retrieve deeper VIKING cutouts instead of VHS across the VIKING footprint, enable the cutout adapter:
  ```yaml
  overrides:
    surveys/viking:
      cutout:
        - adapter: vsa
          adapter_params:
            service_url: http://wfaudata.roe.ac.uk/vista-siap/
            bands: Z,Y,J,H,Ks
          bands: [z, y, j, h, ks]
          enabled: true
  ```
- **Filter to specific bands**: To request only a subset of passbands (e.g. $Z, J, K_s$):
  ```yaml
  overrides:
    surveys/viking:
      cutout:
        - adapter: vsa
          bands: [z, j, ks]
          adapter_params:
            bands: Z,J,Ks
  ```
- **Disable cutouts**: To ensure VIKING is excluded from cutout planning:
  ```yaml
  overrides:
    surveys/viking:
      cutout: []
  ```

### Commands

```console
$ uv run -m mosaics.cli.cutout full -t test/objects_sample_1k.csv -o cutouts.lance --config defs/cutout.yaml --survey viking --plan
```

## Data served by the archive

- **Relationship to VHS**: Both VIKING and VHS observe with VIRCAM on the ESO 4.1m VISTA telescope and query the same VISTA Science Archive Simple Image Access service at WFAU Edinburgh (`http://wfaudata.roe.ac.uk/vista-siap/`). Whereas VHS prioritizes wide southern coverage (~18,000 deg² to $K_s \sim 18.2$), VIKING reaches roughly two magnitudes deeper ($K_s \sim 19.3$, $J \sim 20.9$, $Z \sim 22.4$) over a targeted ~1,500 deg² footprint.
- **Mutual exclusivity in default configuration**: Because `vhs5` and `viking` query the same underlying VSA cutout service, `viking.yaml` ships with `enabled: false` (“enable one of the two”). Researchers targeting fields in the VIKING footprint can enable `viking` and disable `vhs5`.
- **Filter definitions and Vega offsets**: VIKING borrows its photometric filter definitions from `vhs5` (`borrow_filters: vhs5`), using the VIRCAM passbands and CASU Vega-to-AB offsets (Gonzalez-Fernandez et al. 2018): $Y: +0.618$, $J: +0.937$, $H: +1.384$, $K_s: +1.839$ mag. VIRCAM $K_s$ is not WFCAM $K$.
- **Image pixel units and calibration**: Cutout pixel units are marked as `unknown` because individual VSA FITS images carry per-image zero points (`MAGZPT` in FITS headers) without a survey-wide constant calibration factor.

### Catalogue: `II/343/viking2`

VIKING catalogue DR2 (2016-10-10)

**Service URL:** `https://tapvizier.cds.unistra.fr/TAPVizieR/tap`

| Column       | Type     | Unit   | UCD                    | Description                                                                       |
|--------------|----------|--------|------------------------|-----------------------------------------------------------------------------------|
| `recno`      | `int`    | `—`    | `meta.record`          | Record number assigned by the VizieR team. Should Not be used for identification. |
| `RAJ2000`    | `double` | `deg`  | `pos.eq.ra;meta.main`  | Celestial Right Ascension (J2000) (RA2000)                                        |
| `DEJ2000`    | `double` | `deg`  | `pos.eq.dec;meta.main` | Celestial Declination (J2000) (DEC2000)                                           |
| `Name`       | `char`   | `—`    | `meta.id`              | IAU Name (not unique), VIKING JHHMMSS.SS+DDMMSS.ss (IAUNAME)                      |
| `Mclass`     | `short`  | `—`    | `src.class`            | [-9/1] Class flag (MERGEDCLASS) (1)                                               |
| `E(B-V)`     | `float`  | `—`    | `phys.absorption.gal`  | The galactic dust extinction value measured from the Schlegel maps (EBV)          |
| `Zpmag`      | `double` | `mag`  | `phot.mag;em.opt.I`    | ? Extended source Z mag (Petrosian) (ZPETROMAG)                                   |
| `Zperrbits`  | `int`    | `—`    | `meta.code`            | Additional WFAU post-processing error bits in Z (ZPPERRBITS) (2)                  |
| `Ypmag`      | `double` | `mag`  | `phot.mag;em.IR.J`     | ? Extended source Y mag (Petrosian) (YPETROMAG)                                   |
| `Yperrbits`  | `int`    | `—`    | `meta.code`            | Additional WFAU post-processing error bits in Y (YPPERRBITS) (2)                  |
| `Jpmag`      | `double` | `mag`  | `phot.mag;em.IR.J`     | ? Extended source J mag (Petrosian) (JPETROMAG)                                   |
| `Jerrbits`   | `short`  | `—`    | `meta.code`            | [-1/28] processing warning/error bitwise flags in J (JERRBITS) (2)                |
| `Jperrbits`  | `int`    | `—`    | `meta.code`            | Additional WFAU post-processing error bits in J (JPPERRBITS) (2)                  |
| `Hpmag`      | `double` | `mag`  | `phot.mag;em.IR.H`     | ? Extended source H mag (Petrosian) (HPETROMAG)                                   |
| `Hperrbits`  | `int`    | `—`    | `meta.code`            | Additional WFAU post-processing error bits in H (HPPERRBITS) (2)                  |
| `Kspmag`     | `double` | `mag`  | `phot.mag;em.IR.K`     | ? Extended source Ksmag (Petrosian) (KSPETROMAG)                                  |
| `Ksperrbits` | `int`    | `—`    | `meta.code`            | Additional WFAU post-processing error bits in Ks (KSPPERRBITS) (2)                |

### Images

- **Pixel unit:** `unknown` | **AB zero point:** `—` | **Calibration note:** As vhs5 – VSA frames carry a per-image MAGZPT.
- **Cutout service `None`:** adapter `vsa`, bands `['z', 'y', 'j', 'h', 'ks']`, params `{'service_url': 'http://wfaudata.roe.ac.uk/vista-siap/', 'bands': 'Z,Y,J,H,Ks'}`.

### Spectra

No 1D spectroscopic backends are configured for this survey.

## References and acknowledgement

- Edge, A. et al. (2013), *The VISTA Kilo-degree Infrared Galaxy Survey (VIKING)*, *The Messenger*, 154, 32.
- Gonzalez-Fernandez, C. et al. (2018), *The VISTA Science Archive: calibration and Vega-to-AB offsets*, *MNRAS*, 474, 5459 (`doi:10.1093/mnras/stx3075`).
- Cross, N. J. G. et al. (2012), *The VISTA Science Archive (VSA)*, *A&A*, 548, A119 (`doi:10.1051/0004-6361/201219607`).
- VSA SIAP service: [http://wfaudata.roe.ac.uk/vista-siap/](http://wfaudata.roe.ac.uk/vista-siap/)
- VizieR catalogue II/343 (VIKING DR2): [https://vizier.cds.unistra.fr/viz-bin/VizieR?-source=II/343](https://vizier.cds.unistra.fr/viz-bin/VizieR?-source=II/343)

Based on observations made with ESO Telescopes at the La Silla Paranal Observatory under programme ID 179.A-2004 (the VISTA Kilo-degree Infrared Galaxy Survey). The VISTA Data Flow System pipeline processing and science archive are described in Irwin et al. (2004), Hambly et al. (2008), and Cross et al. (2012).

## Provenance of this page

| Source ID            | Kind        | Service / Endpoint                               | Query or URL                                                                               | Retrieved (UTC)        |
|----------------------|-------------|--------------------------------------------------|--------------------------------------------------------------------------------------------|------------------------|
| `catalog_viking_dr2` | `tap_table` | `https://tapvizier.cds.unistra.fr/TAPVizieR/tap` | `https://vizier.cds.unistra.fr/viz-bin/votable?-source=II/343/viking2&-out.max=0&-out.all` | `2026-10-18T17:00:06Z` |
| `vsa_viking`         | `vsa`       | `http://wfaudata.roe.ac.uk/vista-siap/`          | `http://wfaudata.roe.ac.uk/vista-siap/`                                                    | `2026-10-18T16:41:59Z` |
<!-- /generated -->
