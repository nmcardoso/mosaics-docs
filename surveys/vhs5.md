# VISTA Hemisphere Survey DR5 (`vhs5`)

<!-- generated:survey-page -->

The VISTA Hemisphere Survey Data Release 5 (VHS DR5) is a wide-field near-infrared astronomical survey covering approximately 18,000 deg² of the southern celestial hemisphere in the $Y, J, H, K_s$ passbands. Observations were conducted using the VISTA InfraRed CAMera (VIRCAM) mounted on the ESO 4.1-metre Visible and Infrared Survey Telescope for Astronomy (VISTA) at Cerro Paranal, Chile. The survey reaches a nominal 5-sigma point-source depth of $K_s \sim 18.2$ mag (Vega) across its standard survey footprint.

In `mosaics`, `vhs5` provides near-infrared catalogue crossmatching and FITS image cutouts across the southern sky. Catalogue crossmatching is executed locally via the `lsdb` adapter against CDS HATS partitioned Parquet tables (VizieR catalogue `II/367/vhs_dr5`, comprising 1,374,207,485 rows in 3,132 partitions) using a precomputed 34-arcsec margin cache (`border12_34arcsec`). Image cutouts in $Y, J, H, K_s$ are retrieved using the `vsa` adapter querying the VISTA Science Archive (VSA) Simple Image Access (SIA) service hosted by WFAU Edinburgh.

| Property                 | Value                                                                                                                                                                                                                    |
|--------------------------|--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| What MOSAICS extracts    | Catalogue: `vhs5` (photometric, `lsdb`); Cutouts: `vsa` (y,j,h,ks)                                                                                                                                                       |
| Archives                 | HATS (`vizcat.cds.unistra.fr`), VSA (`wfaudata.roe.ac.uk`)                                                                                                                                                               |
| Bands / wavelength range | `y` (`Paranal/VISTA.Y`), `j` (`Paranal/VISTA.J`), `h` (`Paranal/VISTA.H`), `ks` (`Paranal/VISTA.Ks`)                                                                                                                     |
| Used by                  | `mosaics-targets` (crossmatch), `mosaics-cutout full`, `mosaics-crossmatch`                                                                                                                                              |
| Crossmatch radius        | `1` arcsec. No seeing figure in the DR5 release description (a figure only); astrometric rms about 0.15 arcsec per coordinate against 2MASS (ESO release description, Sect. 4.2). The radius rests on that figure alone. |

## Configuration

### The shipped file

Source: `defs/surveys/vhs5.yaml`

```yaml
# VISTA Hemisphere Survey DR5 -- photometry and cutouts.

survey_slug: vhs5
name: VISTA Hemisphere Survey DR5

catalog:
  - name: vhs5
    type: photometric
    adapter: lsdb
    adapter_params:
      # VHS DR5 as VizieR II/367, HATS at CDS: 1,374,207,485 rows in 3,132
      # partitions, matched on this machine rather than uploaded to Data Lab.
      path: https://vizcat.cds.unistra.fr/hats/vhs_dr5/
      # A 34-arcsec halo carrying every column the map below names (checked
      # 2026-09-13).
      margin_cache: https://vizcat.cds.unistra.fr/hats/vhs_dr5/border12_34arcsec

# Crossmatch: the radius a table of positions is matched within, and what it rests on.
crossmatch:
  radius: 1 arcsec
  radius_note: >-
    No seeing figure in the DR5 release description (a figure only); astrometric rms about
    0.15 arcsec per coordinate against 2MASS (ESO release description, Sect. 4.2). The
    radius rests on that figure alone.

catalog_filters:
  vhs5: vhs5_mag_aper3_ks <= 18.2
  vhs5_note: >-
    5 sigma point source over the standard survey area, VEGA on the VIRCAM
    system, to match the photometry above (McMahon et al. 2013). AB would be
    18.2 + 1.84 = 20.0. Ks is the shallowest of the four bands, so it is what
    decides whether an object is in the imaging. No H/J fallback: unlike
    ukidss11's LAS, McMahon et al. 2013 gives VHS's H and J depths only as a
    mean over the combined images (and shallower still in the partial-H
    regions), not the single survey-wide number the other cascades in this
    file rest on, so a value here would be less trustworthy than leaving the
    row unfilled.

cutout:
  - adapter: vsa          # VERIFIED LIVE 2026-08-25, IVOA registry
    adapter_params:
      service_url: http://wfaudata.roe.ac.uk/vista-siap/
      bands: Y,J,H,Ks       # VIRCAM spelling; the columns are y/j/h/ks
    bands: [y, j, h, ks]
    cost:
      latency_s: 2.0
      bandwidth_bps: 15000000
      concurrency: 2
      render_seconds: 0.3

photometric_system:
  filters:
    # VIRCAM. Its Ks is not WFCAM's K, nor 2MASS's Ks.
    y: Paranal/VISTA.Y
    j: Paranal/VISTA.J
    h: Paranal/VISTA.H
    ks: Paranal/VISTA.Ks
  vega_offsets:
    # CASU-recommended (Gonzalez-Fernandez et al. 2018).
    y: 0.618
    j: 0.937
    h: 1.384
    ks: 1.839
  measurements:
    - system: vega_mag
      apertures: [aper3, petro]
  pixel_unit: unknown
  pixel_unit_note: >-
    VSA frames carry a per-image MAGZPT, so one survey-wide number would be
    wrong by however much the frames differ. This stays unknown on purpose;
    a per-image calibration would have to read each frame's own header.

# VISTA Hemisphere Survey (VHS) DR5 catalogue: near-infrared YJHKs
# photometry over the southern-hemisphere VHS footprint.
# Catalogue: VizieR II/367 (McMahon et al. 2020), HATS at CDS, matched with
# LSDB. The names are VizieR's labels from the II/367 ReadMe; each comment
# names the WFAU column it relabels, which is what the aliases were first
# written against.
#
# Magnitudes are VEGA, on the VIRCAM system. Note VIRCAM's Ks is not WFCAM's K
# (ukidss11) and not 2MASS's Ks either -- the three carry different Vega-to-AB
# offsets, which is why the band is spelled `ks` here and `k` in ukidss11.
#
# Aperture tokens follow the WFAU naming: `aper3` is apermag3, the 2.0 arcsec
# aperture-corrected magnitude recommended for point sources; `petro` is the
# Petrosian magnitude recommended for extended sources.
column_mapping:
  SrcID: 'id'                  # unique ID of this merged detection (sourceID)
  RAJ2000: 'ra'                # right ascension, J2000, deg (ra)
  DEJ2000: 'dec'               # declination, J2000, deg (dec)

  Yap3: 'mag_aper3_y'          # default point source Y mag, 2.0 arcsec aperture corrected (yAperMag3), Vega mag
  e_Yap3: 'mag_aper3_y_err'     # error on Yap3 (yAperMag3Err), mag
  Jap3: 'mag_aper3_j'          # default point source J mag, 2.0 arcsec aperture corrected (jAperMag3), Vega mag
  e_Jap3: 'mag_aper3_j_err'     # error on Jap3 (jAperMag3Err), mag
  Hap3: 'mag_aper3_h'          # default point source H mag, 2.0 arcsec aperture corrected (hAperMag3), Vega mag
  e_Hap3: 'mag_aper3_h_err'     # error on Hap3 (hAperMag3Err), mag
  Ksap3: 'mag_aper3_ks'        # default point source Ks mag, 2.0 arcsec aperture corrected (ksAperMag3), Vega mag
  e_Ksap3: 'mag_aper3_ks_err'     # error on Ksap3 (ksAperMag3Err), mag

  Ypmag: 'mag_petro_y'          # extended source Y mag, Petrosian (yPetroMag), Vega mag
  e_Ypmag: 'mag_petro_y_err'     # error on Ypmag (yPetroMagErr), mag
  Jpmag: 'mag_petro_j'          # extended source J mag, Petrosian (jPetroMag), Vega mag
  e_Jpmag: 'mag_petro_j_err'     # error on Jpmag (jPetroMagErr), mag
  Hpmag: 'mag_petro_h'          # extended source H mag, Petrosian (hPetroMag), Vega mag
  e_Hpmag: 'mag_petro_h_err'     # error on Hpmag (hPetroMagErr), mag
  Kspmag: 'mag_petro_ks'        # extended source Ks mag, Petrosian (ksPetroMag), Vega mag
  e_Kspmag: 'mag_petro_ks_err'     # error on Kspmag (ksPetroMagErr), mag

  # mergedClass spans [-3, 1] in DR5: 1|0|-1|-2|-3 = galaxy|noise|star|
  # probableStar|probableGalaxy. II/367 publishes no pSaturated, so the
  # vhs5_p_saturated column the Data Lab table fed is gone.
  Mclass: 'class'               # class flag (mergedClass)
  Mstat: 'class_stat'           # merged N(0,1) stellarness-of-profile statistic (mergedClassStat)
  pStar: 'p_star'               # probability that the source is a star (pStar)
  pGal: 'p_galaxy'              # probability that the source is a galaxy (pGalaxy)
  pNoise: 'p_noise'             # probability that the source is noise (pNoise)

  PriOrSec: 'priorsec'          # seam code for a unique (=0) or duplicated (!=0) source (priOrSec)
  E(B-V): 'ebv'                 # galactic dust extinction from Schlegel maps, mag (eBV)
  prim: 'primary'        # flag marking primary sources (=1) vs. duplicates (=0) (primary_source)

  Yperrbits: 'errbits_y'        # additional WFAU post-processing error bits, Y (yppErrBits)
  Jperrbits: 'errbits_j'        # additional WFAU post-processing error bits, J (jppErrBits)
  Hperrbits: 'errbits_h'        # additional WFAU post-processing error bits, H (hppErrBits)
  Ksperrbits: 'errbits_ks'      # additional WFAU post-processing error bits, Ks (ksppErrBits)
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

**Catalogue `vhs5` (`adapter: lsdb`):**

| Parameter           | Allowed Type   | Default               | Description                                                                                         |
|---------------------|----------------|-----------------------|-----------------------------------------------------------------------------------------------------|
| `path`              | str            | —                     | HTTP or S3 URI of the HATS catalogue root (may contain `{band}` or `{BAND}` when `per_band: true`). |
| `margin_cache`      | str            | None                  | URI of the HATS pre-computed margin cache table for boundary-safe spatial crossmatching.            |
| `storage_options`   | mapping        | {}                    | Filesystem options forwarded to `upath`/`fsspec` (e.g. `{'anon': True}` for public S3 buckets).     |
| `reshape`           | str            | None                  | Optional post-read reshaping strategy for nested or multi-row HATS tables.                          |
| `per_band`          | bool           | false                 | When `true`, opens one HATS catalogue per band by substituting `{band}`/`{BAND}` in `path`.         |
| `remote_ra_column`  | str            | HATS metadata default | Override for the remote right ascension column name.                                                |
| `remote_dec_column` | str            | HATS metadata default | Override for the remote declination column name.                                                    |
- **Cutout `None` (`adapter: vsa`, bands `['y', 'j', 'h', 'ks']`):** configured `adapter_params`: `service_url='http://wfaudata.roe.ac.uk/vista-siap/'`, `bands='Y,J,H,Ks'`.

### Common changes

- **Disable or relax depth filters**: To alter or remove the default 5-sigma point-source selection filter ($K_s \le 18.2$ mag):
  ```yaml
  overrides:
    surveys/vhs5:
      catalog_filters:
        vhs5: null
  ```
- **Disable cutouts**: To perform catalogue crossmatching without fetching image cutouts:
  ```yaml
  overrides:
    surveys/vhs5:
      cutout: []
  ```
- **Select specific cutout passbands**: To download only a subset of bands (e.g. $J$ and $K_s$):
  ```yaml
  overrides:
    surveys/vhs5:
      cutout:
        - adapter: vsa
          bands: [j, ks]
          adapter_params:
            bands: J,Ks
  ```
- **Redirect to a local HATS mirror**: When working with a local mirror of the HATS partition cache:
  ```yaml
  overrides:
    surveys/vhs5:
      catalog:
        - name: vhs5
          adapter: lsdb
          adapter_params:
            path: /path/to/local/hats/vhs_dr5/
            margin_cache: /path/to/local/hats/vhs_dr5/border12_34arcsec
  ```

### Commands

```console
$ uv run -m mosaics.cli.cutout full -t test/objects_sample_1k.csv -o cutouts.lance --config defs/cutout.yaml --survey vhs5 --plan
$ uv run -m mosaics.crossmatch.cli run --survey surveys/vhs5 -i test/objects_sample_1k.csv -o vhs5_matched.parquet
$ uv run -m mosaics.cli.targets --config defs/targets.yaml -o ./targets_run --dec-range 0 1 --plan plan.yaml
```

## Data served by the archive

- **VIRCAM Ks vs WFCAM K and 2MASS Ks**: Magnitudes are in the Vega system on the ESO VISTA VIRCAM photometric system. VIRCAM’s $K_s$ filter is distinct from UKIRT WFCAM’s $K$ filter (`ukidss9`, `ukidss11`) and 2MASS $K_s$. The passband is explicitly designated `ks` in `vhs5` (Vega-to-AB offset $+1.839$ mag, Gonzalez-Fernandez et al. 2018) rather than `k` in UKIDSS ($+1.900$ mag).
- **Vega-to-AB offsets**: Official CASU-recommended Vega-to-AB offsets (Gonzalez-Fernandez et al. 2018) are: $Y: +0.618$, $J: +0.937$, $H: +1.384$, $K_s: +1.839$ mag.
- **Local HATS catalogue matching**: Matching operates locally against CDS’s HATS partitioning of VizieR catalogue II/367 (1,374,207,485 rows, 3,132 partitions) via LSDB. Column aliases map from VizieR II/367 names (`Yap3`, `Jap3`, `Hap3`, `Ksap3`, `Ypmag`, `Mclass`, `prim`) to standardized `vhs5_*` identifiers.
- **Depth filter logic**: The default catalogue filter requires `vhs5_mag_aper3_ks <= 18.2` (5-sigma point source depth, Vega, McMahon et al. 2013). $K_s$ is the shallowest band and determines imaging presence. No $H$ or $J$ fallback is applied because McMahon et al. (2013) quotes $H$ and $J$ depths only as a mean over combined images rather than a uniform survey-wide threshold.
- **Apertures and schema differences with UKIDSS**: VHS publishes 2.0 arcsec aperture-corrected magnitudes (`aper3`) and Petrosian magnitudes (`petro`). Unlike UKIDSS, VHS publishes no total Hall (`hall`) magnitudes, and VizieR II/367 omits `pSaturated`.
- **Image cutout zero points**: VSA FITS images carry a per-image zero point (`MAGZPT` in FITS headers); no survey-wide calibration constant exists, so `pixel_unit` is designated `unknown`.

### Catalogue: `II/367/vhs_dr5`

VISTA Hemisphere Survey band-merged multi-waveband catalogues (VHS) DR5 (version 3); output (on a total of 1,374,207,485 sources)

**Service URL:** `https://vizcat.cds.unistra.fr/hats/vhs_dr5/`

*Showing 100 of 130 total archive columns (all MOSAICS-mapped columns plus primary table fields; full 130-column schema is stored in `docs/_data/sources/vhs5/hats_vhs_dr5.json`).*

| Column       | Type     | Unit   | UCD                      | Description                                                                                      |
|--------------|----------|--------|--------------------------|--------------------------------------------------------------------------------------------------|
| `SrcID`      | `int64`  | `—`    | `meta.id;meta.main`      | Unique ID of this merged detection as assigned by merge algorithm unless PrioOrSec!=0 (SOURCEID) |
| `RAJ2000`    | `double` | `deg`  | `pos.eq.ra;meta.main`    | Celestial Right Ascension (J2000) (RA2000)                                                       |
| `DEJ2000`    | `double` | `deg`  | `pos.eq.dec;meta.main`   | [-90/1.5] Celestial Declination (J2000) (DEC2000)                                                |
| `PriOrSec`   | `int64`  | `—`    | `meta.code`              | Seam code for a unique (=0) or duplicated (!=0) source (5)                                       |
| `Mstat`      | `float`  | `—`    | `—`                      | —                                                                                                |
| `Mclass`     | `int8`   | `—`    | `meta.code.class`        | [-3/1] Class flag (MERGEDCLASS) (2)                                                              |
| `pStar`      | `float`  | `—`    | `—`                      | —                                                                                                |
| `pGal`       | `float`  | `—`    | `—`                      | —                                                                                                |
| `pNoise`     | `float`  | `—`    | `—`                      | —                                                                                                |
| `E(B-V)`     | `float`  | `—`    | `—`                      | —                                                                                                |
| `Ypmag`      | `float`  | `mag`  | `phot.mag;em.IR.NIR`     | [3.6/37]? Extended source Y (Vega) mag (Petrosian) (YPETROMAG) (3)                               |
| `e_Ypmag`    | `float`  | `—`    | `—`                      | —                                                                                                |
| `Yap3`       | `float`  | `mag`  | `phot.mag;em.IR.NIR`     | [4.2/34]? Default point source Y aperture corrected Vega mag (2.0” diam) (YAPERMAG3) (3)         |
| `e_Yap3`     | `float`  | `—`    | `—`                      | —                                                                                                |
| `Yperrbits`  | `int32`  | `—`    | `meta.code`              | [0/12587216] Additional WFAU post-processing error bits in Y (4)                                 |
| `Jpmag`      | `float`  | `mag`  | `phot.mag;em.IR.J`       | [-24/38]? Extended source J (Vega) mag (Petrosian) (JPETROMAG) (3)                               |
| `e_Jpmag`    | `float`  | `—`    | `—`                      | —                                                                                                |
| `Jap3`       | `float`  | `mag`  | `phot.mag;em.IR.J`       | [-21.1/34.1]? Default point source J aperture corrected Vega mag (2.0” diam) (JAPERMAG3) (3)     |
| `e_Jap3`     | `float`  | `—`    | `—`                      | —                                                                                                |
| `Jperrbits`  | `int32`  | `—`    | `meta.code`              | [0/12587216]? Additional WFAU post-processing error bits in J (4)                                |
| `Hpmag`      | `float`  | `mag`  | `phot.mag;em.IR.H`       | [-24/36]? Extended source H (Vega) mag (Petrosian) (HPETROMAG) (3)                               |
| `e_Hpmag`    | `float`  | `—`    | `—`                      | —                                                                                                |
| `Hap3`       | `float`  | `mag`  | `phot.mag;em.IR.H`       | [-21.5/29]? Default point source H aperture corrected Vega mag (2.0” diam) (HAPERMAG3) (3)       |
| `e_Hap3`     | `float`  | `—`    | `—`                      | —                                                                                                |
| `Hperrbits`  | `int32`  | `—`    | `meta.code`              | [0/12587216]? Additional WFAU post-processing error bits in H (4)                                |
| `Kspmag`     | `float`  | `mag`  | `phot.mag;em.IR.K`       | [-20.3/34]? Extended source Ks (Vega) mag (Petrosian) (KSPETROMAG) (3)                           |
| `e_Kspmag`   | `float`  | `—`    | `—`                      | —                                                                                                |
| `Ksap3`      | `float`  | `mag`  | `phot.mag;em.IR.K`       | [-33.5/31]? Default point source Ks aperture corrected Vega mag (2.0” diam) (KSAPERMAG3) (3)     |
| `e_Ksap3`    | `float`  | `—`    | `—`                      | —                                                                                                |
| `Ksperrbits` | `int32`  | `—`    | `meta.code`              | [0/12587216]? Additional WFAU post-processing error bits in Ks (4)                               |
| `prim`       | `float`  | `—`    | `—`                      | —                                                                                                |
| `Name`       | `string` | `—`    | `meta.id`                | Source name in IAU convention (VHS JHHMMSS.ss+DDMMSS.s) (SOURCENAME)                             |
| `cuEventID`  | `int32`  | `—`    | `—`                      | —                                                                                                |
| `FrameID`    | `int64`  | `—`    | `—`                      | —                                                                                                |
| `GLON`       | `double` | `—`    | `—`                      | —                                                                                                |
| `GLAT`       | `double` | `—`    | `—`                      | —                                                                                                |
| `Y-Jpnt`     | `float`  | `mag`  | `phot.color;em.IR;em.IR` | [-12.1/10.3]? Point source color Y-J (using aperMag3) (YMJPNT)                                   |
| `e_Y-Jpnt`   | `float`  | `—`    | `—`                      | —                                                                                                |
| `J-Hpnt`     | `float`  | `mag`  | `phot.color;em.IR;em.IR` | [-11.3/9.7]? Point source color J-H (using aperMag3) (JMHPNT)                                    |
| `e_J-Hpnt`   | `float`  | `—`    | `—`                      | —                                                                                                |
| `H-Kspnt`    | `float`  | `mag`  | `phot.color;em.IR;em.IR` | [-10.7/11.3]? Point source color H-Ks (using aperMag3) (HMKSPNT)                                 |
| `e_H-Kspnt`  | `float`  | `—`    | `—`                      | —                                                                                                |
| `J-Kspnt`    | `float`  | `mag`  | `phot.color;em.IR;em.IR` | [-12/21.4]? Point source color J-Ks (using aperMag3) (JMKSPNT)                                   |
| `e_J-Kspnt`  | `float`  | `—`    | `—`                      | —                                                                                                |
| `Y-Jext`     | `float`  | `mag`  | `phot.color;em.IR;em.IR` | [-12/10.3]? Extended source color Y-J (using aperMagNoAperCorr3) (YMJEXT)                        |
| `e_Y-Jext`   | `float`  | `—`    | `—`                      | —                                                                                                |
| `J-Hext`     | `float`  | `mag`  | `phot.color;em.IR;em.IR` | [-11.2/9.8]? Extended source color J-H (using aperMagNoAperCorr3) (JMHEXT)                       |
| `e_J-Hext`   | `float`  | `—`    | `—`                      | —                                                                                                |
| `H-Ksext`    | `float`  | `mag`  | `phot.color;em.IR;em.IR` | [-10.6/11.2]? Extended source color H-Ks (using aperMagNoAperCorr3) (HMKSEXT)                    |
| `e_H-Ksext`  | `float`  | `—`    | `—`                      | —                                                                                                |
| `J-Ksext`    | `float`  | `mag`  | `phot.color;em.IR;em.IR` | [-11.8/22.1]? Extended source color J-Ks (using aperMagNoAperCorr3) (JMKSEXT)                    |
| `e_J-Ksext`  | `float`  | `—`    | `—`                      | —                                                                                                |
| `AY`         | `float`  | `mag`  | `phys.absorption.gal`    | [5E-4/53] The galactic extinction correction in the Y band                                       |
| `AJ`         | `float`  | `mag`  | `phys.absorption.gal`    | [4E-4/38] The galactic extinction correction in the J band                                       |
| `AH`         | `float`  | `mag`  | `phys.absorption.gal`    | [2E-4/24] The galactic extinction correction in the H band                                       |
| `AKs`        | `float`  | `mag`  | `phys.absorption.gal`    | [1E-4/16] The galactic extinction correction in the Ks band                                      |
| `YMJD`       | `double` | `—`    | `—`                      | —                                                                                                |
| `Yap4`       | `float`  | `—`    | `—`                      | —                                                                                                |
| `e_Yap4`     | `float`  | `—`    | `—`                      | —                                                                                                |
| `Yap6`       | `float`  | `—`    | `—`                      | —                                                                                                |
| `e_Yap6`     | `float`  | `—`    | `—`                      | —                                                                                                |
| `Yapc3`      | `float`  | `—`    | `phot.mag;em.IR.NIR`     | [4.6/35]? Default extended source Y aperture Vega mag (2.0 arcsec diam) (YAPERMAGNOAPERCORR3)    |
| `Yapc4`      | `float`  | `—`    | `—`                      | —                                                                                                |
| `Yapc6`      | `float`  | `—`    | `—`                      | —                                                                                                |
| `Yamaj`      | `float`  | `—`    | `—`                      | —                                                                                                |
| `Yrms`       | `float`  | `—`    | `—`                      | —                                                                                                |
| `Yell`       | `float`  | `—`    | `—`                      | —                                                                                                |
| `YPA`        | `float`  | `—`    | `—`                      | —                                                                                                |
| `Yerrbits`   | `int8`   | `—`    | `—`                      | —                                                                                                |
| `Yavgap3`    | `float`  | `—`    | `stat.likelihood`        | [0/112]? Average confidence in 2 arcsec diam default aperture 3 Y (YAVERAGECONF)                 |
| `Yclass`     | `int8`   | `—`    | `—`                      | —                                                                                                |
| `YclasSt`    | `float`  | `—`    | `—`                      | —                                                                                                |
| `Yseq`       | `int32`  | `—`    | `—`                      | —                                                                                                |
| `JMJD`       | `double` | `—`    | `—`                      | —                                                                                                |
| `Jap4`       | `float`  | `—`    | `—`                      | —                                                                                                |
| `e_Jap4`     | `float`  | `—`    | `—`                      | —                                                                                                |
| `Jap6`       | `float`  | `—`    | `—`                      | —                                                                                                |
| `e_Jap6`     | `float`  | `—`    | `—`                      | —                                                                                                |
| `Japc3`      | `float`  | `mag`  | `phot.mag;em.IR.J`       | [-20.5/35]? Default extended source J aperture Vega mag (2.0 arcsec diam) (JAPERMAGNOAPERCORR3)  |
| `Japc4`      | `float`  | `—`    | `—`                      | —                                                                                                |
| `Japc6`      | `float`  | `—`    | `—`                      | —                                                                                                |
| `Jamaj`      | `float`  | `—`    | `—`                      | —                                                                                                |
| `Jrms`       | `float`  | `—`    | `—`                      | —                                                                                                |
| `Jell`       | `float`  | `—`    | `—`                      | —                                                                                                |
| `JPA`        | `float`  | `—`    | `—`                      | —                                                                                                |
| `Jerrbits`   | `int8`   | `—`    | `—`                      | —                                                                                                |
| `Javgap3`    | `float`  | `—`    | `stat.likelihood`        | [0/112]? Average confidence in 2 arcsec diam default aperture 3 J (JAVERAGECONF)                 |
| `Jclass`     | `int8`   | `—`    | `—`                      | —                                                                                                |
| `JclasStat`  | `float`  | `—`    | `—`                      | —                                                                                                |
| `Jseq`       | `int32`  | `—`    | `—`                      | —                                                                                                |
| `Jxi`        | `float`  | `—`    | `—`                      | —                                                                                                |
| `Jeta`       | `float`  | `—`    | `—`                      | —                                                                                                |
| `HMJD`       | `double` | `—`    | `—`                      | —                                                                                                |
| `Hap4`       | `float`  | `—`    | `—`                      | —                                                                                                |
| `e_Hap4`     | `float`  | `—`    | `—`                      | —                                                                                                |
| `Hap6`       | `float`  | `—`    | `—`                      | —                                                                                                |
| `e_Hap6`     | `float`  | `—`    | `—`                      | —                                                                                                |
| `Hapc3`      | `float`  | `mag`  | `phot.mag;em.IR.H`       | [-21/29.1]? Default extended source H aperture Vega mag (2.0 arcsec diam) (HAPERMAGNOAPERCORR3)  |
| `Hapc4`      | `float`  | `—`    | `—`                      | —                                                                                                |
| `Hapc6`      | `float`  | `—`    | `—`                      | —                                                                                                |

### Images

- **Pixel unit:** `unknown` | **AB zero point:** `—` | **Calibration note:** VSA frames carry a per-image MAGZPT, so one survey-wide number would be wrong by however much the frames differ. This stays unknown on purpose; a per-image calibration would have to read each frame’s own header.
- **Cutout service `None`:** adapter `vsa`, bands `['y', 'j', 'h', 'ks']`, params `{'service_url': 'http://wfaudata.roe.ac.uk/vista-siap/', 'bands': 'Y,J,H,Ks'}`.

### Spectra

No 1D spectroscopic backends are configured for this survey.

## References and acknowledgement

- McMahon, R. G. et al. (2013), *First Data Release of the VISTA Hemisphere Survey (VHS DR1)*, *The Messenger*, 154, 35.
- McMahon, R. G. et al. (2020), VizieR Online Data Catalog: II/367 (VISTA Hemisphere Survey DR5).
- Gonzalez-Fernandez, C. et al. (2018), *The VISTA Science Archive: calibration and Vega-to-AB offsets*, *MNRAS*, 474, 5459 (`doi:10.1093/mnras/stx3075`).
- Cross, N. J. G. et al. (2012), *The VISTA Science Archive (VSA)*, *A&A*, 548, A119 (`doi:10.1051/0004-6361/201219607`).
- VSA SIAP service: [http://wfaudata.roe.ac.uk/vista-siap/](http://wfaudata.roe.ac.uk/vista-siap/)
- CDS HATS repository: [https://vizcat.cds.unistra.fr/hats/vhs_dr5/](https://vizcat.cds.unistra.fr/hats/vhs_dr5/)

Based on observations made with ESO Telescopes at the La Silla Paranal Observatory under programme ID 179.A-2010 (the VISTA Hemisphere Survey). The VISTA Data Flow System pipeline processing and science archive are described in Irwin et al. (2004), Hambly et al. (2008), and Cross et al. (2012). When referencing catalogue data accessed via CDS VizieR, acknowledge the VizieR service operated at CDS, Strasbourg, France.

## Provenance of this page

| Source ID      | Kind           | Service / Endpoint                            | Query or URL                                                 | Retrieved (UTC)        |
|----------------|----------------|-----------------------------------------------|--------------------------------------------------------------|------------------------|
| `hats_vhs_dr5` | `hats_catalog` | `https://vizcat.cds.unistra.fr/hats/vhs_dr5/` | `https://vizcat.cds.unistra.fr/hats/vhs_dr5/main/properties` | `2026-10-18T17:00:01Z` |
| `vsa_vista`    | `vsa`          | `http://wfaudata.roe.ac.uk/vista-siap/`       | `http://wfaudata.roe.ac.uk/vista-siap/`                      | `2026-10-18T16:41:18Z` |
<!-- /generated -->
