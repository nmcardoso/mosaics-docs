# DENIS DR3 (`denis3`)

<!-- generated:survey-page -->

The Deep Near-Infrared Survey of the Southern Sky Data Release 3 (DENIS DR3) is an astronomical survey that imaged the southern sky ($\delta < +2^\circ$) simultaneously in two near-infrared bands and one optical band. Carried out with a dedicated 1-metre telescope at the European Southern Observatory (ESO) at La Silla, Chile, DENIS observed in Gunn-$i$ ($0.82\,\mu\text{m}$), $J$ ($1.25\,\mu\text{m}$), and $K_s$ ($2.15\,\mu\text{m}$). The DR3 point-source catalogue (DENIS Consortium 2005, VizieR `B/denis/denis`) provides point-spread-function (PSF) fitting photometry for millions of southern objects, supplemented by cross-matched optical $B$ and $R$ magnitudes from USNO-A2.0.

In `mosaics`, DENIS DR3 is used for photometric crossmatching (`mosaics-targets` and `mosaics-query`). The catalogue is served in HATS (Hierarchical Adaptive Tilings Scheme) Parquet format from CDS (`https://vizcat.cds.unistra.fr/hats/denis3/`), enabling high-throughput spatial cone queries via the LSDB adapter without TAP server overhead.

| Property                 | Value                                                                                                                                                                                                                                                                                                                              |
|--------------------------|------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| What MOSAICS extracts    | Catalogue: `denis3` (photometric, `lsdb`)                                                                                                                                                                                                                                                                                          |
| Archives                 | HATS (`vizcat.cds.unistra.fr`)                                                                                                                                                                                                                                                                                                     |
| Bands / wavelength range | `i` (`DENIS/DENIS.I`), `j` (`DENIS/DENIS.J`), `ks` (`DENIS/DENIS.Ks`)                                                                                                                                                                                                                                                              |
| Used by                  | `mosaics-targets` (crossmatch), `mosaics-crossmatch`                                                                                                                                                                                                                                                                               |
| Crossmatch radius        | `3` arcsec. PROVISIONAL. No PSF or seeing figure was found; the only statement is that a general extracted point source has a position “accuracy better than 1arcsec” (CDS ReadMe of B/denis), a bound and not a 1-sigma value. The radius is three times that bound, to be replaced when a seeing or pixel scale source is found. |

## Configuration

### The shipped file

Source: `defs/surveys/denis3.yaml`

```yaml
# DENIS DR3 -- photometry.

survey_slug: denis3
name: DENIS DR3

catalog:
  - name: denis3
    type: photometric
    adapter: lsdb
    adapter_params:
      path: https://vizcat.cds.unistra.fr/hats/denis3/

# Crossmatch: the radius a table of positions is matched within, and what it rests on.
crossmatch:
  radius: 3 arcsec
  radius_note: >-
    PROVISIONAL. No PSF or seeing figure was found; the only statement is that a general
    extracted point source has a position "accuracy better than 1arcsec" (CDS ReadMe of
    B/denis), a bound and not a 1-sigma value. The radius is three times that bound, to be
    replaced when a seeing or pixel scale source is found.

photometric_system:
  filters:
    # The third band is Ks, though the catalogue column is `Kmag`.
    i: DENIS/DENIS.I
    j: DENIS/DENIS.J
    ks: DENIS/DENIS.Ks
  vega_offsets:
    # The data-team report does not cover this survey, so these are derived
    # from SVO's own AB and Vega zero points for the shipped curves,
    # 2.5 log10(ZP_AB / ZP_Vega) -- self-consistent with the profiles the
    # pipeline stores, which is the best available anchor without a DENIS
    # calibration paper.
    i: 0.432
    j: 0.899
    ks: 1.851
  measurements:
    - system: vega_mag
      apertures: [default]

# DENIS DR3 point-source catalogue (DENIS Consortium 2005, VizieR B/denis):
# I/J/Ks PSF-fit photometry over the Southern sky, plus a USNO-A2.0
# cross-match for optical B/R magnitudes.
# Catalog: https://vizcat.cds.unistra.fr/hats/denis3/
# ra/dec = RAJ2000/DEJ2000. Selected via LSDB, not turned into SQL.
column_mapping:
  # DENIS: 'id'                      # DENIS identifier, JHHMMSS.s-DDMMSS
  RAJ2000: 'ra'                      # right ascension, J2000, deg
  DEJ2000: 'dec'                     # declination, J2000, deg
  Imag: 'mag_i'                      # I-band (Gunn-i, 0.82um) magnitude from PSF fit
  e_Imag: 'mag_i_err'                # I-band magnitude error from PSF fit
  Jmag: 'mag_j'                      # J-band (1.25um) magnitude from PSF fit
  e_Jmag: 'mag_j_err'                # J-band magnitude error from PSF fit
  Kmag: 'mag_ks'                     # Ks-band (2.15um) magnitude from PSF fit
  e_Kmag: 'mag_ks_err'               # Ks-band magnitude error from PSF fit
  q_Imag: 'flag_quality_i'           # I-band quality flag, 0-100 (higher is better)
  q_Jmag: 'flag_quality_j'           # J-band quality flag, 0-100 (higher is better)
  q_Kmag: 'flag_quality_ks'          # Ks-band quality flag, 0-100 (higher is better)
  Ipsf: 'psf_corr_i'                 # I-band correlation to the PSF, 0-100
  Jpsf: 'psf_corr_j'                 # J-band correlation to the PSF, 0-100
  Kpsf: 'psf_corr_ks'                # Ks-band correlation to the PSF, 0-100
  # Rmag: 'mag_r'                    # R magnitude of the nearest USNO-A2.0 match
  # Bmag: 'mag_b'                    # B magnitude of the nearest USNO-A2.0 match
  Iflg: 'flag_i'                     # I-band image+source flag, hex-encoded bitmask
  Jflg: 'flag_j'                     # J-band image+source flag, hex-encoded bitmask
  Kflg: 'flag_ks'                    # Ks-band image+source flag, hex-encoded bitmask
  mult: 'flag_mult'                  # true if merged from two overlapping-image detections
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

**Catalogue `denis3` (`adapter: lsdb`):**

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

- **Add optical cross-match or identifier columns**: In your entrypoint YAML, customize `column_mapping` to extract the DENIS object identifier (`DENIS: 'id'`) or the cross-matched USNO-A2.0 photographic optical magnitudes (`Bmag: 'mag_b'`, `Rmag: 'mag_r'`); the loader adds the `denis3_` prefix to every alias.
- **Quality and correlation filtering**: Add post-hoc filters in your pipeline to discard low-significance detections, such as filtering on PSF correlation (`denis3_psf_corr_* >= 50`) or demanding high photometric quality scores (`denis3_flag_quality_* >= 80`).
- **Disable catalogue crossmatching**: Disable DENIS DR3 matching in an entrypoint configuration by setting `overrides[surveys/denis3].catalog.0.enabled: false`.
- **Handling blended sources**: Filter out objects flagged as overlapping-image merges by checking the boolean column `denis3_flag_mult`.

### Commands

```console
$ uv run -m mosaics.crossmatch.cli run --survey surveys/denis3 -i test/objects_sample_1k.csv -o denis3_matched.parquet
$ uv run -m mosaics.cli.targets --config defs/targets.yaml -o ./targets_run --dec-range 0 1 --plan plan.yaml
```

## Data served by the archive

- **Photometric system and Vega offsets**: Published DENIS magnitudes are on the Vega system (`measurements: system: vega_mag`). To align with AB-calibrated surveys, `mosaics` applies Vega-to-AB offsets derived from SVO zero points ($2.5 \log_{10}(\text{ZP}_\text{AB} / \text{ZP}_\text{Vega})$): $0.432$ mag for Gunn-$i$, $0.899$ mag for $J$, and $1.851$ mag for $K_s$.
- **Band terminology**: The third infrared passband is $K_s$ ($2.15\,\mu\text{m}$), although the VizieR table column is labelled `Kmag`.
- **Quality metrics**: The catalogue provides three distinct quality metrics per band:
  - `q_Imag`, `q_Jmag`, `q_Kmag`: Numerical quality flags ranging from 0 to 100, where higher numbers indicate superior data quality.
  - `Ipsf`, `Jpsf`, `Kpsf`: Source correlation to the stellar PSF (0 to 100).
  - `Iflg`, `Jflg`, `Kflg`: Hex-encoded bitmasks recording image-level and source-extraction conditions.
- **HATS Parquet delivery**: The catalogue is stored and partitioned on CDS servers as HATS Parquet. The LSDB adapter resolves and retrieves partitions directly over HTTPS, eliminating ADQL query timeouts on large spatial footprints.

### Catalogue: `B/denis/denis`

DENIS output

**Service URL:** `https://vizcat.cds.unistra.fr/hats/denis3/`

| Column       | Type     | Unit   | UCD                    | Description                             |
|--------------|----------|--------|------------------------|-----------------------------------------|
| `DENIS`      | `string` | `—`    | `meta.id;meta.main`    | DENIS name, identifier                  |
| `RAJ2000`    | `double` | `deg`  | `pos.eq.ra;meta.main`  | Right ascension (J2000)                 |
| `DEJ2000`    | `double` | `deg`  | `pos.eq.dec;meta.main` | Declination (J2000)                     |
| `Image`      | `int32`  | `—`    | `—`                    | —                                       |
| `Strip`      | `int16`  | `—`    | `obs.field`            | DENIS strip number                      |
| `Imag`       | `float`  | `mag`  | `phot.mag;em.opt.I`    | ?Magnitude in I band from PSF fit       |
| `e_Imag`     | `float`  | `mag`  | `stat.error;phot.mag`  | ?Magnitude error in I band from PSF fit |
| `Jmag`       | `float`  | `mag`  | `phot.mag;em.IR.J`     | ?Magnitude in J band from PSF fit       |
| `e_Jmag`     | `float`  | `mag`  | `stat.error;phot.mag`  | ?Magnitude error in J band from PSF fit |
| `Kmag`       | `float`  | `mag`  | `phot.mag;em.IR.K`     | ?Magnitude in K band from PSF fit       |
| `e_Kmag`     | `float`  | `mag`  | `stat.error;phot.mag`  | ?Magnitude error in K band from PSF fit |
| `Imag5.5`    | `float`  | `—`    | `—`                    | —                                       |
| `e_Imag5.5`  | `float`  | `—`    | `—`                    | —                                       |
| `Jmag5.5`    | `float`  | `—`    | `—`                    | —                                       |
| `e_Jmag5.5`  | `float`  | `—`    | `—`                    | —                                       |
| `Kmag5.5`    | `float`  | `—`    | `—`                    | —                                       |
| `e_Kmag5.5`  | `float`  | `—`    | `—`                    | —                                       |
| `Imag4.5`    | `float`  | `—`    | `—`                    | —                                       |
| `e_Imag4.5`  | `float`  | `—`    | `—`                    | —                                       |
| `Jmag4.5`    | `float`  | `—`    | `—`                    | —                                       |
| `e_Jmag4.5`  | `float`  | `—`    | `—`                    | —                                       |
| `Kmag4.5`    | `float`  | `—`    | `—`                    | —                                       |
| `e_Kmag4.5`  | `float`  | `—`    | `—`                    | —                                       |
| `Imag3.5`    | `float`  | `—`    | `—`                    | —                                       |
| `e_Imag3.5`  | `float`  | `—`    | `—`                    | —                                       |
| `Jmag3.5`    | `float`  | `—`    | `—`                    | —                                       |
| `e_Jmag3.5`  | `float`  | `—`    | `—`                    | —                                       |
| `Kmag3.5`    | `float`  | `—`    | `—`                    | —                                       |
| `e_Kmag3.5`  | `float`  | `—`    | `—`                    | —                                       |
| `Imag3.5c`   | `float`  | `—`    | `—`                    | —                                       |
| `e_Imag3.5c` | `float`  | `—`    | `—`                    | —                                       |
| `Jmag3.5c`   | `float`  | `—`    | `—`                    | —                                       |
| `e_Jmag3.5c` | `float`  | `—`    | `—`                    | —                                       |
| `Kmag3.5c`   | `float`  | `—`    | `—`                    | —                                       |
| `e_Kmag3.5c` | `float`  | `—`    | `—`                    | —                                       |
| `Imag2.5c`   | `float`  | `—`    | `—`                    | —                                       |
| `e_Imag2.5c` | `float`  | `—`    | `—`                    | —                                       |
| `Jmag2.5c`   | `float`  | `—`    | `—`                    | —                                       |
| `e_Jmag2.5c` | `float`  | `—`    | `—`                    | —                                       |
| `Kmag2.5c`   | `float`  | `—`    | `—`                    | —                                       |
| `e_Kmag2.5c` | `float`  | `—`    | `—`                    | —                                       |
| `Imag1.5c`   | `float`  | `—`    | `—`                    | —                                       |
| `e_Imag1.5c` | `float`  | `—`    | `—`                    | —                                       |
| `Jmag1.5c`   | `float`  | `—`    | `—`                    | —                                       |
| `e_Jmag1.5c` | `float`  | `—`    | `—`                    | —                                       |
| `Kmag1.5c`   | `float`  | `—`    | `—`                    | —                                       |
| `e_Kmag1.5c` | `float`  | `—`    | `—`                    | —                                       |
| `Rmag`       | `float`  | `mag`  | `phot.mag;em.opt.R`    | ?R magnitude of USNOA2.0 nearest match  |
| `Bmag`       | `float`  | `mag`  | `phot.mag;em.opt.B`    | ?B magnitude of USNOA2.0 nearest match  |
| `q_Imag`     | `int8`   | `—`    | `—`                    | —                                       |
| `q_Jmag`     | `int8`   | `—`    | `—`                    | —                                       |
| `q_Kmag`     | `int8`   | `—`    | `—`                    | —                                       |
| `Ipsf`       | `int8`   | `—`    | `src.class.starGalaxy` | [0,100]?Correlation to PSF for I band   |
| `Jpsf`       | `int8`   | `—`    | `src.class.starGalaxy` | [0,100]?Correlation to PSF for J band   |
| `Kpsf`       | `int8`   | `—`    | `src.class.starGalaxy` | [0,100]?Correlation to PSF for K band   |
| `IMpsf`      | `float`  | `—`    | `—`                    | —                                       |
| `JMpsf`      | `float`  | `—`    | `—`                    | —                                       |
| `KMpsf`      | `float`  | `—`    | `—`                    | —                                       |
| `Ix`         | `float`  | `—`    | `—`                    | —                                       |
| `Iy`         | `float`  | `—`    | `—`                    | —                                       |
| `Jx`         | `float`  | `—`    | `—`                    | —                                       |
| `Jy`         | `float`  | `—`    | `—`                    | —                                       |
| `Kx`         | `float`  | `—`    | `—`                    | —                                       |
| `Ky`         | `float`  | `—`    | `—`                    | —                                       |
| `A2RAdeg`    | `double` | `—`    | `—`                    | —                                       |
| `A2DEdeg`    | `double` | `—`    | `—`                    | —                                       |
| `A2Ep`       | `float`  | `—`    | `—`                    | —                                       |
| `Dist`       | `int16`  | `—`    | `—`                    | —                                       |
| `ObsJD`      | `double` | `—`    | `—`                    | —                                       |
| `Iflg`       | `string` | `—`    | `—`                    | —                                       |
| `Jflg`       | `string` | `—`    | `—`                    | —                                       |
| `Kflg`       | `string` | `—`    | `—`                    | —                                       |
| `mult`       | `bool`   | `—`    | `—`                    | —                                       |

### Images

No image cutout backends are configured for this survey.

### Spectra

No 1D spectroscopic backends are configured for this survey.

## References and acknowledgement

- Epchtein, N., et al. (1997), *The Messenger*, 87, 27 — The Deep Near-Infrared Southern Sky Survey (DENIS).
- Epchtein, N., et al. (1999), *A&A*, 349, 236 — The DENIS southern sky survey: 1. Survey description and first results.
- DENIS Consortium (2005), VizieR Catalogue `B/denis` ([https://cdsarc.cds.unistra.fr/viz-bin/cat/B/denis](https://cdsarc.cds.unistra.fr/viz-bin/cat/B/denis)).
- CDS HATS Partition: [https://vizcat.cds.unistra.fr/hats/denis3/](https://vizcat.cds.unistra.fr/hats/denis3/).

Publications utilizing DENIS data should cite Epchtein et al. (1997, 1999) and acknowledge the DENIS team and the European Southern Observatory: “The DENIS project was conducted by a consortium of European institutes led by the Institut d’Astrophysique de Paris, using the 1-m telescope at the European Southern Observatory (La Silla, Chile).” When accessing the catalogue via CDS, acknowledge the Centre de Données astronomiques de Strasbourg (CDS).

## Provenance of this page

| Source ID     | Kind           | Service / Endpoint                           | Query or URL                                                | Retrieved (UTC)        |
|---------------|----------------|----------------------------------------------|-------------------------------------------------------------|------------------------|
| `hats_denis3` | `hats_catalog` | `https://vizcat.cds.unistra.fr/hats/denis3/` | `https://vizcat.cds.unistra.fr/hats/denis3/main/properties` | `2026-10-18T16:52:52Z` |
<!-- /generated -->
