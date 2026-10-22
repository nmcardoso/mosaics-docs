# SkyMapper DR4 (`skymapper4`)

<!-- generated:survey-page -->

The SkyMapper Southern Survey Data Release 4 (DR4) provides deep, comprehensive optical photometry across the entire southern celestial hemisphere ($\delta < +2^\circ$, over 20,000 deg²). Acquired with the 1.35m SkyMapper telescope at Siding Spring Observatory, Australia, the survey images the sky in six distinct optical passbands: $u, v, g, r, i, z$.

In `mosaics`, `skymapper4` provides southern optical PSF photometry crossmatched locally from the DR4 master table stored as a HATS partitioned dataset (724,329,344 rows in VizieR II/379/smssdr4), with a 5-arcsec margin cache. Six single-band HiPS tile sets (`CDS/P/skymapper-{U,V,G,R,I,Z}`) are declared for cutouts, but remain disabled pending empirical zero-point calibration against isolated standard stars.

| Property                 | Value                                                                                                                                                                                    |
|--------------------------|------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| What MOSAICS extracts    | Catalogue: `skymapper4` (photometric, `lsdb`); Cutouts: `hips` (u), `hips` (v), `hips` (g), `hips` (r), `hips` (i), `hips` (z)                                                           |
| Archives                 | HATS (`data.lsdb.io`), CDS HiPS / hips2fits (`alasky.cds.unistra.fr`)                                                                                                                    |
| Bands / wavelength range | `u` (`SkyMapper/SkyMapper.u`), `v` (`SkyMapper/SkyMapper.v`), `g` (`SkyMapper/SkyMapper.g`), `r` (`SkyMapper/SkyMapper.r`), `i` (`SkyMapper/SkyMapper.i`), `z` (`SkyMapper/SkyMapper.z`) |
| Used by                  | `mosaics-targets` (crossmatch), `mosaics-cutout full`, `mosaics-crossmatch`                                                                                                              |
| Crossmatch radius        | `2` arcsec. Median seeing 2.5-3.3 arcsec (r band 2.6 arcsec); median offset from Gaia 0.13 arcsec (SkyMapper DR4 release page).                                                          |

## Configuration

### The shipped file

Source: `defs/surveys/skymapper4.yaml`

```yaml
# SkyMapper DR4 -- photometry, and cutouts from HiPS colour tiles.

survey_slug: skymapper4
name: SkyMapper DR4

catalog:
  - name: skymapper4
    type: photometric
    adapter: lsdb
    adapter_params:
      # The DR4 master table as HATS: 724,329,344 rows, matched on this machine
      # rather than uploaded to Data Lab. It keeps the master table's own column
      # names, so every column the map below names is there as spelled (checked
      # 2026-09-14). LineA mirrors the same build at
      # linea.data.lsdb.io/hats/skymapper/catalog/.
      path: https://data.lsdb.io/hats/skymapper_dr4/catalog/sky_mapper_dr4
      # A 5-arcsec halo: wider than the match radius, which is all a margin has
      # to be.
      margin_cache: https://data.lsdb.io/hats/skymapper_dr4/catalog/sky_mapper_dr4_5arcs

# Crossmatch: the radius a table of positions is matched within, and what it rests on.
crossmatch:
  radius: 2 arcsec
  radius_note: >-
    Median seeing 2.5-3.3 arcsec (r band 2.6 arcsec); median offset from Gaia 0.13 arcsec
    (SkyMapper DR4 release page).

catalog_filters:
  skymapper4: skymapper4_mag_psf_r <= 20.5
  skymapper4_note: >-
    10 sigma per field, AB (Onken et al. 2024) -- note the convention is not
    the 5 sigma every other survey here quotes, so this number is
    conservative by roughly 0.75 mag against them. Per filter the main
    survey reaches 18.5 to 20.5; `r` is the deepest. No g/i fallback:
    Onken et al. 2024 gives that 18.5-20.5 range for the survey as a whole,
    not a per-filter table, so there is no single sourced number for g or i
    to fall back to -- only `r`'s own depth is stated plainly enough to use.

cutout:
  # Six keys rather than one adapter, because the survey is published as
  # six single-band HiPS. They used to carry a hand-written `label:` each,
  # since `skymapper4:hips#4` said nothing about which band it was; the
  # default label is `skymapper4:hips:<band>` now, so the labels are gone.
  - adapter: hips
    adapter_params:
      hips_id: CDS/P/skymapper-U
    bands: [u]
    enabled: false
  - adapter: hips
    adapter_params:
      hips_id: CDS/P/skymapper-V
    bands: [v]
    enabled: false
  - adapter: hips
    adapter_params:
      hips_id: CDS/P/skymapper-G
    bands: [g]
    enabled: false
  - adapter: hips
    adapter_params:
      hips_id: CDS/P/skymapper-R
    bands: [r]
    enabled: false
  - adapter: hips
    adapter_params:
      hips_id: CDS/P/skymapper-I
    bands: [i]
    enabled: false
  - adapter: hips
    adapter_params:
      hips_id: CDS/P/skymapper-Z
    bands: [z]
    enabled: false

photometric_system:
  filters:
    # Six filters, uvgriz; `v` is SkyMapper's violet band.
    u: SkyMapper/SkyMapper.u
    v: SkyMapper/SkyMapper.v
    g: SkyMapper/SkyMapper.g
    r: SkyMapper/SkyMapper.r
    i: SkyMapper/SkyMapper.i
    z: SkyMapper/SkyMapper.z
  measurements:
    - system: ab_mag
      apertures: [psf]
  pixel_unit: unknown
  pixel_unit_note: >-
    Still unknown, but for a better reason than before. `CDS/P/skymapper-
    DR4/color` does not exist -- the MOCServer registers six *single-band*
    SkyMapper HiPS instead (checked 2026-09-04), which is a strictly better
    product: one plane per band, matching the six bands the photometry above
    converts, rather than an RGB rendering with no flux in it. What is not
    known is the tiles' scale. SkyMapper images are natively on a per-image
    zero point and the HiPS build states none, so a survey-wide constant
    would be a guess; resolving it needs a check against the SkyMapper
    catalogue on isolated standards.

# SkyMapper Southern Survey DR4 master table: uvgriz photometry over the
# southern sky.
# Catalogue: the DR4 master table, HATS at data.lsdb.io, matched with LSDB.
# Its HATS properties name raj2000/dej2000 as the position, so the crossmatch
# needs no coordinate columns spelled out here.
#
# Magnitudes are AB and need no zero-point shift. DR4 abandoned the APASS/
# Refcat2 anchoring of earlier releases for synthetic photometry derived from
# Gaia DR3 XP spectra through the SkyMapper passbands (Onken et al. 2024).
#
# `v` is SkyMapper's own violet band, not a typo for `u`; the survey has six
# filters, uvgriz.
column_mapping:
  object_id: 'id'                   # unique SkyMapper DR4 object identifier
  raj2000: 'ra'                     # right ascension, J2000, deg
  # e_raj2000: 'ra_err'               # error on the right ascension, arcsec
  dej2000: 'dec'                    # declination, J2000, deg
  # e_dej2000: 'dec_err'              # error on the declination, arcsec

  u_psf: 'mag_psf_u'                    # PSF magnitude, u band, AB mag
  e_u_psf: 'mag_psf_u_err'              # error on the u PSF magnitude, mag
  v_psf: 'mag_psf_v'                    # PSF magnitude, v band, AB mag
  e_v_psf: 'mag_psf_v_err'              # error on the v PSF magnitude, mag
  g_psf: 'mag_psf_g'                    # PSF magnitude, g band, AB mag
  e_g_psf: 'mag_psf_g_err'              # error on the g PSF magnitude, mag
  r_psf: 'mag_psf_r'                    # PSF magnitude, r band, AB mag
  e_r_psf: 'mag_psf_r_err'              # error on the r PSF magnitude, mag
  i_psf: 'mag_psf_i'                    # PSF magnitude, i band, AB mag
  e_i_psf: 'mag_psf_i_err'              # error on the i PSF magnitude, mag
  z_psf: 'mag_psf_z'                    # PSF magnitude, z band, AB mag
  e_z_psf: 'mag_psf_z_err'              # error on the z PSF magnitude, mag

  flags: 'flags'                        # bitwise OR of the source's SExtractor flags
  # u_flags: 'flags_u'                    # bitwise OR of the u-band flags
  # v_flags: 'flags_v'                    # bitwise OR of the v-band flags
  # g_flags: 'flags_g'                    # bitwise OR of the g-band flags
  # r_flags: 'flags_r'                    # bitwise OR of the r-band flags
  # i_flags: 'flags_i'                    # bitwise OR of the i-band flags
  # z_flags: 'flags_z'                    # bitwise OR of the z-band flags

  class_star: 'class_star'              # SExtractor star/galaxy classifier, 1 = point-like
  chi2_psf: 'chi2_psf'                  # reduced chi-squared of the PSF fit
  ngood: 'ngood'                        # number of good detections across all bands
  ebmv_sfd: 'ebv'                       # E(B-V) from the Schlegel/Finkbeiner/Davis maps, mag
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

**Catalogue `skymapper4` (`adapter: lsdb`):**

| Parameter           | Allowed Type   | Default               | Description                                                                                         |
|---------------------|----------------|-----------------------|-----------------------------------------------------------------------------------------------------|
| `path`              | str            | —                     | HTTP or S3 URI of the HATS catalogue root (may contain `{band}` or `{BAND}` when `per_band: true`). |
| `margin_cache`      | str            | None                  | URI of the HATS pre-computed margin cache table for boundary-safe spatial crossmatching.            |
| `storage_options`   | mapping        | {}                    | Filesystem options forwarded to `upath`/`fsspec` (e.g. `{'anon': True}` for public S3 buckets).     |
| `reshape`           | str            | None                  | Optional post-read reshaping strategy for nested or multi-row HATS tables.                          |
| `per_band`          | bool           | false                 | When `true`, opens one HATS catalogue per band by substituting `{band}`/`{BAND}` in `path`.         |
| `remote_ra_column`  | str            | HATS metadata default | Override for the remote right ascension column name.                                                |
| `remote_dec_column` | str            | HATS metadata default | Override for the remote declination column name.                                                    |
- **Cutout `None` (`adapter: hips`, bands `['u']`):** configured `adapter_params`: `hips_id='CDS/P/skymapper-U'`.
- **Cutout `None` (`adapter: hips`, bands `['v']`):** configured `adapter_params`: `hips_id='CDS/P/skymapper-V'`.
- **Cutout `None` (`adapter: hips`, bands `['g']`):** configured `adapter_params`: `hips_id='CDS/P/skymapper-G'`.
- **Cutout `None` (`adapter: hips`, bands `['r']`):** configured `adapter_params`: `hips_id='CDS/P/skymapper-R'`.
- **Cutout `None` (`adapter: hips`, bands `['i']`):** configured `adapter_params`: `hips_id='CDS/P/skymapper-I'`.
- **Cutout `None` (`adapter: hips`, bands `['z']`):** configured `adapter_params`: `hips_id='CDS/P/skymapper-Z'`.

### Common changes

- Modify or disable the production magnitude cut: the default production filter applies `skymapper4_mag_psf_r <= 20.5`. Override or remove this cut in an entrypoint configuration using:
  ```yaml
  overrides:
    surveys/skymapper4:
      catalog_filters:
        skymapper4: null
  ```
- Disable SkyMapper DR4 catalogue crossmatching in an entrypoint by setting:
  ```yaml
  overrides:
    surveys/skymapper4:
      catalog: []
  ```

  or setting `enabled: false` on the catalogue block.
- Enable single-band HiPS cutouts: activate specific passbands by setting `enabled: true` under `overrides[surveys/skymapper4].cutout` (e.g. for `CDS/P/skymapper-R`).

### Commands

```console
$ uv run -m mosaics.cli.cutout full -t test/objects_sample_1k.csv -o cutouts.lance --config defs/cutout.yaml --survey skymapper4 --plan
$ uv run -m mosaics.crossmatch.cli run --survey surveys/skymapper4 -i test/objects_sample_1k.csv -o skymapper4_matched.parquet
$ uv run -m mosaics.cli.targets --config defs/targets.yaml -o ./targets_run --dec-range 0 1 --plan plan.yaml
```

## Data served by the archive

- Passband complement and the violet $v$ filter: SkyMapper observes in six passbands ($u, v, g, r, i, z$). The $v$ filter is SkyMapper’s narrow violet passband centered on the stellar Ca II K $\lambda 3933$ Å line, designed for stellar metallicity measurements; it is an independent filter and not a typo for $u$.
- Photometric calibration: Magnitudes are in the AB system. DR4 abandoned the APASS and Refcat2 calibrations used in earlier releases, adopting synthetic photometry tied directly to Gaia DR3 BP/RP (XP) spectrophotometry integrated through the SkyMapper filter transmission profiles (Onken et al. 2024).
- 10-sigma depth convention: The production filter `skymapper4_mag_psf_r <= 20.5` reflects a 10-sigma point-source detection limit per field (Onken et al. 2024), contrasting with the 5-sigma convention adopted by other surveys in MOSAICS. Consequently, this depth cut is conservative by roughly 0.75 mag relative to 5-sigma limits. While the main survey spans 18.5 to 20.5 mag across bands, only the $r$-band depth is explicitly cited without a per-filter table breakdown in Onken et al. (2024), precluding an explicit $g$ or $i$ fallback threshold.
- Cutouts and pixel scaling: CDS publishes six single-band HiPS collections (`CDS/P/skymapper-{U,V,G,R,I,Z}`, order 9, pixel scale $2.236 \times 10^{-4}$ deg, sky fraction ~0.45–0.50). Because SkyMapper frames natively have per-image zero points and the HiPS build specifies no survey-wide scale factor, `pixel_unit` is marked as `unknown` and the cutout adapters remain disabled.
- Local HATS partitioning: The master catalogue is queried at `https://data.lsdb.io/hats/skymapper_dr4/catalog/sky_mapper_dr4` with a 5-arcsec margin cache (`sky_mapper_dr4_5arcs`) and mirrored at LIneA (`linea.data.lsdb.io/hats/skymapper/catalog/`).

### Catalogue: `II/379/smssdr4`

SkyMapper DR4

**Service URL:** `https://data.lsdb.io/hats/skymapper_dr4/catalog/sky_mapper_dr4`

*Showing 100 of 121 total archive columns (all MOSAICS-mapped columns plus primary table fields; full 121-column schema is stored in `docs/_data/sources/skymapper4/hats_skymapper_dr4.json`).*

| Column            | Type     | Unit   | UCD         | Description                                                          |
|-------------------|----------|--------|-------------|----------------------------------------------------------------------|
| `object_id`       | `int64`  | `—`    | `—`         | —                                                                    |
| `raj2000`         | `double` | `—`    | `—`         | —                                                                    |
| `dej2000`         | `double` | `—`    | `—`         | —                                                                    |
| `flags`           | `int32`  | `—`    | `meta.code` | Bitwise OR of Source Extractor flags across all observations (flags) |
| `ngood`           | `int32`  | `—`    | `—`         | —                                                                    |
| `class_star`      | `double` | `—`    | `—`         | —                                                                    |
| `chi2_psf`        | `double` | `—`    | `—`         | —                                                                    |
| `u_psf`           | `double` | `—`    | `—`         | —                                                                    |
| `e_u_psf`         | `double` | `—`    | `—`         | —                                                                    |
| `v_psf`           | `double` | `—`    | `—`         | —                                                                    |
| `e_v_psf`         | `double` | `—`    | `—`         | —                                                                    |
| `g_psf`           | `double` | `—`    | `—`         | —                                                                    |
| `e_g_psf`         | `double` | `—`    | `—`         | —                                                                    |
| `r_psf`           | `double` | `—`    | `—`         | —                                                                    |
| `e_r_psf`         | `double` | `—`    | `—`         | —                                                                    |
| `i_psf`           | `double` | `—`    | `—`         | —                                                                    |
| `e_i_psf`         | `double` | `—`    | `—`         | —                                                                    |
| `z_psf`           | `double` | `—`    | `—`         | —                                                                    |
| `e_z_psf`         | `double` | `—`    | `—`         | —                                                                    |
| `ebmv_sfd`        | `double` | `—`    | `—`         | —                                                                    |
| `e_raj2000`       | `int64`  | `—`    | `—`         | —                                                                    |
| `e_dej2000`       | `int64`  | `—`    | `—`         | —                                                                    |
| `smss_j`          | `string` | `—`    | `—`         | —                                                                    |
| `mean_epoch`      | `double` | `—`    | `—`         | —                                                                    |
| `rms_epoch`       | `double` | `—`    | `—`         | —                                                                    |
| `glon`            | `double` | `—`    | `—`         | —                                                                    |
| `glat`            | `double` | `—`    | `—`         | —                                                                    |
| `nimaflags`       | `int64`  | `—`    | `—`         | —                                                                    |
| `u_flags`         | `int32`  | `—`    | `—`         | —                                                                    |
| `u_nimaflags`     | `int64`  | `—`    | `—`         | —                                                                    |
| `u_ngood`         | `int32`  | `—`    | `—`         | —                                                                    |
| `u_nclip`         | `int32`  | `—`    | `—`         | —                                                                    |
| `v_flags`         | `int32`  | `—`    | `—`         | —                                                                    |
| `v_nimaflags`     | `int64`  | `—`    | `—`         | —                                                                    |
| `v_ngood`         | `int32`  | `—`    | `—`         | —                                                                    |
| `v_nclip`         | `int32`  | `—`    | `—`         | —                                                                    |
| `g_flags`         | `int32`  | `—`    | `—`         | —                                                                    |
| `g_nimaflags`     | `int64`  | `—`    | `—`         | —                                                                    |
| `g_ngood`         | `int32`  | `—`    | `—`         | —                                                                    |
| `g_nclip`         | `int32`  | `—`    | `—`         | —                                                                    |
| `r_flags`         | `int32`  | `—`    | `—`         | —                                                                    |
| `r_nimaflags`     | `int64`  | `—`    | `—`         | —                                                                    |
| `r_ngood`         | `int32`  | `—`    | `—`         | —                                                                    |
| `r_nclip`         | `int32`  | `—`    | `—`         | —                                                                    |
| `i_flags`         | `int32`  | `—`    | `—`         | —                                                                    |
| `i_nimaflags`     | `int64`  | `—`    | `—`         | —                                                                    |
| `i_ngood`         | `int32`  | `—`    | `—`         | —                                                                    |
| `i_nclip`         | `int32`  | `—`    | `—`         | —                                                                    |
| `z_flags`         | `int32`  | `—`    | `—`         | —                                                                    |
| `z_nimaflags`     | `int64`  | `—`    | `—`         | —                                                                    |
| `z_ngood`         | `int32`  | `—`    | `—`         | —                                                                    |
| `z_nclip`         | `int32`  | `—`    | `—`         | —                                                                    |
| `flags_psf`       | `int64`  | `—`    | `—`         | —                                                                    |
| `radius_petro`    | `double` | `—`    | `—`         | —                                                                    |
| `mean_fwhm`       | `double` | `—`    | `—`         | —                                                                    |
| `u_petro`         | `double` | `—`    | `—`         | —                                                                    |
| `e_u_petro`       | `double` | `—`    | `—`         | —                                                                    |
| `u_apc05`         | `double` | `—`    | `—`         | —                                                                    |
| `e_u_apc05`       | `double` | `—`    | `—`         | —                                                                    |
| `u_mmvar`         | `double` | `—`    | `—`         | —                                                                    |
| `v_petro`         | `double` | `—`    | `—`         | —                                                                    |
| `e_v_petro`       | `double` | `—`    | `—`         | —                                                                    |
| `v_apc05`         | `double` | `—`    | `—`         | —                                                                    |
| `e_v_apc05`       | `double` | `—`    | `—`         | —                                                                    |
| `v_mmvar`         | `double` | `—`    | `—`         | —                                                                    |
| `g_petro`         | `double` | `—`    | `—`         | —                                                                    |
| `e_g_petro`       | `double` | `—`    | `—`         | —                                                                    |
| `g_apc05`         | `double` | `—`    | `—`         | —                                                                    |
| `e_g_apc05`       | `double` | `—`    | `—`         | —                                                                    |
| `g_mmvar`         | `double` | `—`    | `—`         | —                                                                    |
| `r_petro`         | `double` | `—`    | `—`         | —                                                                    |
| `e_r_petro`       | `double` | `—`    | `—`         | —                                                                    |
| `r_apc05`         | `double` | `—`    | `—`         | —                                                                    |
| `e_r_apc05`       | `double` | `—`    | `—`         | —                                                                    |
| `r_mmvar`         | `double` | `—`    | `—`         | —                                                                    |
| `i_petro`         | `double` | `—`    | `—`         | —                                                                    |
| `e_i_petro`       | `double` | `—`    | `—`         | —                                                                    |
| `i_apc05`         | `double` | `—`    | `—`         | —                                                                    |
| `e_i_apc05`       | `double` | `—`    | `—`         | —                                                                    |
| `i_mmvar`         | `double` | `—`    | `—`         | —                                                                    |
| `z_petro`         | `double` | `—`    | `—`         | —                                                                    |
| `e_z_petro`       | `double` | `—`    | `—`         | —                                                                    |
| `z_apc05`         | `double` | `—`    | `—`         | —                                                                    |
| `e_z_apc05`       | `double` | `—`    | `—`         | —                                                                    |
| `z_mmvar`         | `double` | `—`    | `—`         | —                                                                    |
| `self_id1`        | `int64`  | `—`    | `—`         | —                                                                    |
| `self_dist1`      | `double` | `—`    | `—`         | —                                                                    |
| `self_id2`        | `int64`  | `—`    | `—`         | —                                                                    |
| `self_dist2`      | `double` | `—`    | `—`         | —                                                                    |
| `self_id3`        | `int64`  | `—`    | `—`         | —                                                                    |
| `self_dist3`      | `double` | `—`    | `—`         | —                                                                    |
| `cnt_self_15`     | `double` | `—`    | `—`         | —                                                                    |
| `ebmv_gnilc`      | `double` | `—`    | `—`         | —                                                                    |
| `ebmv_g_err`      | `double` | `—`    | `—`         | —                                                                    |
| `gaia_dr3_id1`    | `int64`  | `—`    | `—`         | —                                                                    |
| `gaia_dr3_dist1`  | `double` | `—`    | `—`         | —                                                                    |
| `gaia_dr3_id2`    | `int64`  | `—`    | `—`         | —                                                                    |
| `gaia_dr3_dist2`  | `double` | `—`    | `—`         | —                                                                    |
| `cnt_gaia_dr3_15` | `int64`  | `—`    | `—`         | —                                                                    |
| `twomass_key`     | `int64`  | `—`    | `—`         | —                                                                    |

### Images

- **Pixel unit:** `unknown` | **AB zero point:** `—` | **Calibration note:** Still unknown, but for a better reason than before. `CDS/P/skymapper- DR4/color` does not exist – the MOCServer registers six *single-band* SkyMapper HiPS instead (checked 2026-09-04), which is a strictly better product: one plane per band, matching the six bands the photometry above converts, rather than an RGB rendering with no flux in it. What is not known is the tiles’ scale. SkyMapper images are natively on a per-image zero point and the HiPS build states none, so a survey-wide constant would be a guess; resolving it needs a check against the SkyMapper catalogue on isolated standards.
- **Cutout service `None`:** adapter `hips`, bands `['u']`, params `{'hips_id': 'CDS/P/skymapper-U'}`.
- **Cutout service `None`:** adapter `hips`, bands `['v']`, params `{'hips_id': 'CDS/P/skymapper-V'}`.
- **Cutout service `None`:** adapter `hips`, bands `['g']`, params `{'hips_id': 'CDS/P/skymapper-G'}`.
- **Cutout service `None`:** adapter `hips`, bands `['r']`, params `{'hips_id': 'CDS/P/skymapper-R'}`.
- **Cutout service `None`:** adapter `hips`, bands `['i']`, params `{'hips_id': 'CDS/P/skymapper-I'}`.
- **Cutout service `None`:** adapter `hips`, bands `['z']`, params `{'hips_id': 'CDS/P/skymapper-Z'}`.

| HiPS ID             | Regime   | Frame        | Order   | Pixel scale (deg)   | Tile format   | Mirrors                                             |
|---------------------|----------|--------------|---------|---------------------|---------------|-----------------------------------------------------|
| `CDS/P/skymapper-G` | Optical  | `equatorial` | `9`     | `2.236E-4`          | `png fits`    | `alasky.cds.unistra.fr`, `alaskybis.cds.unistra.fr` |
| `CDS/P/skymapper-U` | Optical  | `equatorial` | `9`     | `2.236E-4`          | `png fits`    | `alasky.cds.unistra.fr`, `alaskybis.cds.unistra.fr` |
| `CDS/P/skymapper-V` | Optical  | `equatorial` | `9`     | `2.236E-4`          | `png fits`    | `alasky.cds.unistra.fr`, `alaskybis.cds.unistra.fr` |
| `CDS/P/skymapper-R` | Optical  | `equatorial` | `9`     | `2.236E-4`          | `png fits`    | `alasky.cds.unistra.fr`, `alaskybis.cds.unistra.fr` |
| `CDS/P/skymapper-I` | Optical  | `equatorial` | `9`     | `2.236E-4`          | `png fits`    | `alasky.cds.unistra.fr`, `alaskybis.cds.unistra.fr` |
| `CDS/P/skymapper-Z` | Optical  | `equatorial` | `9`     | `2.236E-4`          | `png fits`    | `alasky.cds.unistra.fr`, `alaskybis.cds.unistra.fr` |

### Spectra

No 1D spectroscopic backends are configured for this survey.

## References and acknowledgement

- Onken, C. A. et al. (2024), *SkyMapper Southern Survey: Data Release 4*, *PASA*, 41, e040 (`doi:10.1017/pasa.2024.34`).
- Wolf, C. et al. (2018), *SkyMapper Southern Survey: First Data Release (DR1)*, *PASA*, 35, e010 (`doi:10.1017/pasa.2018.10`).
- HATS DR4 master catalog at LSDB: `https://data.lsdb.io/hats/skymapper_dr4/catalog/sky_mapper_dr4`.
- CDS HiPS records: `CDS/P/skymapper-U`, `CDS/P/skymapper-V`, `CDS/P/skymapper-G`, `CDS/P/skymapper-R`, `CDS/P/skymapper-I`, `CDS/P/skymapper-Z`.

The national facility capability for SkyMapper has been funded through ARC LIEF grant LE130100104 from the Australian Research Council, awarded to the University of Sydney, the Australian National University, Swinburne University of Technology, the University of Queensland, the University of Western Australia, the University of Melbourne, Macquarie University, and Monash University, and through the Optical Astronomy System Infrastructure Support (OASIS) scheme by the Australian Government Department of Industry, Innovation, Science, Research and Tertiary Education. SkyMapper operations were supported through the Research School of Astronomy and Astrophysics of the ANU.

## Provenance of this page

| Source ID            | Kind           | Service / Endpoint                                               | Query or URL                                                                             | Retrieved (UTC)        |
|----------------------|----------------|------------------------------------------------------------------|------------------------------------------------------------------------------------------|------------------------|
| `hats_skymapper_dr4` | `hats_catalog` | `https://data.lsdb.io/hats/skymapper_dr4/catalog/sky_mapper_dr4` | `https://data.lsdb.io/hats/skymapper_dr4/catalog/sky_mapper_dr4/properties`              | `2026-10-18T16:59:30Z` |
| `hips_skymapper`     | `hips`         | `https://alasky.cds.unistra.fr/MocServer/query`                  | `https://alasky.cds.unistra.fr/MocServer/query?ID=CDS/P/skymapper-G&fmt=json&get=record` | `2026-10-18T16:38:43Z` |
<!-- /generated -->
