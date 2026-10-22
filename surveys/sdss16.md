# Sloan Digital Sky Survey DR16 (`sdss16`)

<!-- generated:survey-page -->

The Sloan Digital Sky Survey Data Release 16 (SDSS DR16) is a major data release from SDSS-IV, encompassing optical imaging across ~14,555 deg² of the sky observed with the 2.5m Sloan Foundation Telescope at Apache Point Observatory, New Mexico. Imaging is acquired in five broad optical passbands: $u, g, r, i, z$.

In `mosaics`, `sdss16` serves as the primary photometric catalogue for SDSS. It provides point-spread-function (PSF), model, and Petrosian magnitudes alongside object classifications, clean photometry flags, and Petrosian radii. The catalogue is queried locally from VizieR’s HATS build of catalogue V/154 (`V/154/sdss16`), comprising 1,231,051,050 rows partitioned into 3,024 partitions with a 34-arcsec margin cache, enabling high-throughput spatial crossmatching without external network dependencies.

| Property                 | Value                                                                                                                                                                                                                                                               |
|--------------------------|---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| What MOSAICS extracts    | Catalogue: `sdss16` (photometric, `lsdb`)                                                                                                                                                                                                                           |
| Archives                 | HATS (`vizcat.cds.unistra.fr`)                                                                                                                                                                                                                                      |
| Bands / wavelength range | `u` (`SLOAN/SDSS.u`), `g` (`SLOAN/SDSS.g`), `r` (`SLOAN/SDSS.r`), `i` (`SLOAN/SDSS.i`), `z` (`SLOAN/SDSS.z`)                                                                                                                                                        |
| Used by                  | `mosaics-targets` (crossmatch), `mosaics-crossmatch`                                                                                                                                                                                                                |
| Crossmatch radius        | `1` arcsec. Median r-band seeing (psfWidth) 1.32 arcsec; absolute astrometric accuracy 45 mas rms per coordinate against USNO CCD Astrograph Catalog, about 100 mas at r = 22 (Pier et al. 2003, arXiv:astro-ph/0211375, SDSS-I/II calibration, not DR16-specific). |

## Configuration

### The shipped file

Source: `defs/surveys/sdss16.yaml`

```yaml
# SDSS DR16 -- photometry, from VizieR's HATS build of V/154.
#
# This used to be the DR20 photometry in `sdss20.yaml`, crossmatched through
# SkyServer's CrossID form. In a restricted run (`--dec-range -2.5 2.5`) most of
# those batches failed, so the columns now come from a HATS catalogue at CDS,
# matched on this machine with no archive query in the loop. DR16 because V/154
# is the release VizieR publishes, and the prefix says so: these are `sdss16_*`,
# not DR20 numbers under a DR20 name. `sdss20.yaml` keeps its cutouts.

survey_slug: sdss16
name: Sloan Digital Sky Survey DR16

catalog:
  - name: sdss16
    type: photometric
    adapter: lsdb
    adapter_params:
      # 1,231,051,050 rows in 3,024 partitions. CDS serves no HTTP range
      # requests, so a partition is downloaded whole however few of its
      # columns the column map below names.
      path: https://vizcat.cds.unistra.fr/hats/sdss16/
      # A 34-arcsec halo beside the partitions, carrying every column the map
      # below names (checked 2026-09-13). Without it a match near a partition
      # edge would miss the row on the other side.
      margin_cache: https://vizcat.cds.unistra.fr/hats/sdss16/border12_34arcsec

# Crossmatch: the radius a table of positions is matched within, and what it rests on.
crossmatch:
  radius: 1 arcsec
  radius_note: >-
    Median r-band seeing (psfWidth) 1.32 arcsec; absolute astrometric accuracy 45 mas rms
    per coordinate against USNO CCD Astrograph Catalog, about 100 mas at r = 22 (Pier et al.
    2003, arXiv:astro-ph/0211375, SDSS-I/II calibration, not DR16-specific).

photometric_system:
  filters:
    u: SLOAN/SDSS.u
    g: SLOAN/SDSS.g
    r: SLOAN/SDSS.r
    i: SLOAN/SDSS.i
    z: SLOAN/SDSS.z
  measurements:
    # Asinh "luptitudes", as every SDSS magnitude is. V/154 carries no flux
    # columns, so the Lupton inversion is the only route. PSF, model and
    # Petrosian because that is what V/154 publishes: it has no cModel.
    - system: asinh_mag
      apertures: [psf, model, petro]
      softening:              # Lupton, Gunn & Szalay 1999
        u: 1.4e-10
        g: 0.9e-10
        r: 1.2e-10
        i: 1.8e-10
        z: 7.4e-10
      offsets:                # m_AB - m_SDSS, where the system is not AB
        u: -0.04
        z: 0.02

# V/154/sdss16: the SDSS DR16 photometric catalogue as VizieR relabels it.
# Each alias's comment quotes the VizieR description, whose parenthesis is
# the SkyServer column it was taken from -- `upmag` is `psfMag_u`, `umag` is
# `u` (modelMag), `uPmag` is `petroMag_u`. Note the case: `upmag` (PSF) and
# `uPmag` (Petrosian) differ only in one letter.
#
# Like PhotoObjAll, V/154 keeps secondary and family detections; `mode` is
# carried so a duplicate can be filtered downstream rather than silently
# dropped here.
column_mapping:
  # objID: 'id'                        # SDSS unique object identifier
  RA_ICRS: 'ra'                        # right ascension (ICRS) at the epoch of observation, deg
  DE_ICRS: 'dec'                       # declination (ICRS) at the epoch of observation, deg
  mode: 'mode'                         # PhotoMode: 1=primary, 2=secondary, 3=family, 4=outside
  class: 'type'                        # type of object: 3=galaxy, 6=star
  clean: 'flag_clean'                  # clean photometry flag: 1=clean, 0=unclean
  flags: 'flags'                       # photo object attribute flags (PHOTO_FLAGS)

  upmag: 'mag_psf_u'                   # PSF magnitude, u (psfMag_u), mag
  e_upmag: 'mag_psf_u_err'             # mean error on upmag (psfMagErr_u), mag
  gpmag: 'mag_psf_g'                   # PSF magnitude, g (psfMag_g), mag
  e_gpmag: 'mag_psf_g_err'             # mean error on gpmag (psfMagErr_g), mag
  rpmag: 'mag_psf_r'                   # PSF magnitude, r (psfMag_r), mag
  e_rpmag: 'mag_psf_r_err'             # mean error on rpmag (psfMagErr_r), mag
  ipmag: 'mag_psf_i'                   # PSF magnitude, i (psfMag_i), mag
  e_ipmag: 'mag_psf_i_err'             # mean error on ipmag (psfMagErr_i), mag
  zpmag: 'mag_psf_z'                   # PSF magnitude, z (psfMag_z), mag
  e_zpmag: 'mag_psf_z_err'             # mean error on zpmag (psfMagErr_z), mag

  umag: 'mag_model_u'                  # model magnitude, u, AB scale (u), mag
  e_umag: 'mag_model_u_err'            # mean error on umag (err_u), mag
  gmag: 'mag_model_g'                  # model magnitude, g (g), mag
  e_gmag: 'mag_model_g_err'            # mean error on gmag (err_g), mag
  rmag: 'mag_model_r'                  # model magnitude, r (r), mag
  e_rmag: 'mag_model_r_err'            # mean error on rmag (err_r), mag
  imag: 'mag_model_i'                  # model magnitude, i (i), mag
  e_imag: 'mag_model_i_err'            # mean error on imag (err_i), mag
  zmag: 'mag_model_z'                  # model magnitude, z (z), mag
  e_zmag: 'mag_model_z_err'            # mean error on zmag (err_z), mag

  uPmag: 'mag_petro_u'                 # Petrosian magnitude, u (petroMag_u), mag
  e_uPmag: 'mag_petro_u_err'           # mean error on uPmag (petroMagErr_u), mag
  gPmag: 'mag_petro_g'                 # Petrosian magnitude, g (petroMag_g), mag
  e_gPmag: 'mag_petro_g_err'           # mean error on gPmag (petroMagErr_g), mag
  rPmag: 'mag_petro_r'                 # Petrosian magnitude, r (petroMag_r), mag
  e_rPmag: 'mag_petro_r_err'           # mean error on rPmag (petroMagErr_r), mag
  iPmag: 'mag_petro_i'                 # Petrosian magnitude, i (petroMag_i), mag
  e_iPmag: 'mag_petro_i_err'           # mean error on iPmag (petroMagErr_i), mag
  zPmag: 'mag_petro_z'                 # Petrosian magnitude, z (petroMag_z), mag
  e_zPmag: 'mag_petro_z_err'           # mean error on zPmag (petroMagErr_z), mag

  rPrad: 'petro_radius_r'              # Petrosian radius, r (petroRad_r), arcsec
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

**Catalogue `sdss16` (`adapter: lsdb`):**

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

- Disable SDSS DR16 photometric crossmatching in an entrypoint configuration by setting:
  ```yaml
  overrides:
    surveys/sdss16:
      catalog: []
  ```

  or setting `enabled: false` on the catalogue entry.
- Modify column selections by adjusting `column_mapping` in an entrypoint override, for instance omitting Petrosian or PSF bands to minimize dataset width.
- Filter out duplicate detections downstream using the `sdss16_mode` column (`1 = primary`, `2 = secondary`, `3 = family`, `4 = outside`).

### Commands

```console
$ uv run -m mosaics.crossmatch.cli run --survey surveys/sdss16 -i test/objects_sample_1k.csv -o sdss16_matched.parquet
$ uv run -m mosaics.cli.targets --config defs/targets.yaml -o ./targets_run --dec-range 0 1 --plan plan.yaml
```

## Data served by the archive

- Architectural separation of SDSS: SDSS data products are split across three survey definitions in MOSAICS: `sdss16` (photometry), `sdss17` (spectroscopy), and `sdss20` (cutouts). Formerly, SDSS photometry was matched through SkyServer’s CrossID form under `sdss20.yaml`. In restricted-declination test runs (e.g. `--dec-range -2.5 2.5`), remote CrossID batch queries frequently failed. Switching to VizieR’s local HATS build of V/154 provides fast, reliable local crossmatching with no remote archive bottleneck.
- Lupton asinh magnitudes: SDSS magnitudes in V/154 are inverse hyperbolic sine magnitudes (“luptitudes”, Lupton, Gunn & Szalay 1999). Because V/154 carries no linear flux columns, Lupton inversion is the sole route to compute fluxes.
- Softening parameters ($b$):
  - $u$: $1.4 \times 10^{-10}$
  - $g$: $0.9 \times 10^{-10}$
  - $r$: $1.2 \times 10^{-10}$
  - $i$: $1.8 \times 10^{-10}$
  - $z$: $7.4 \times 10^{-10}$
- Offsets to true AB ($m_\mathrm{AB} - m_\mathrm{SDSS}$):
  - $u$: $-0.04$ mag
  - $z$: $+0.02$ mag
  - ($g, r, i$ have zero offsets).
- Available apertures: V/154 provides PSF (`psf`), model (`model`), and Petrosian (`petro`) magnitudes; it does not publish composite model (`cModel`) magnitudes.
- Column casing in V/154: In VizieR V/154, `upmag` represents PSF magnitude ($u$), `umag` represents model magnitude ($u$), and `uPmag` represents Petrosian magnitude ($u$). PSF and Petrosian column names differ only by capitalization (`upmag` vs. `uPmag`).
- Detection modes: Like `PhotoObjAll`, V/154 preserves secondary and family detections. The detection mode is mapped into `sdss16_mode` so downstream pipelines can select primary objects (`mode = 1`) without losing provenance.

### Catalogue: `V/154/sdss16`

* output of the SDSS photometric catalog

**Service URL:** `https://vizcat.cds.unistra.fr/hats/sdss16/`

*Showing 100 of 130 total archive columns (all MOSAICS-mapped columns plus primary table fields; full 130-column schema is stored in `docs/_data/sources/sdss16/hats_sdss16.json`).*

| Column      | Type     | Unit     | UCD                     | Description                                                               |
|-------------|----------|----------|-------------------------|---------------------------------------------------------------------------|
| `RA_ICRS`   | `double` | `deg`    | `pos.eq.ra;meta.main`   | Right Ascension of the object (ICRS) at Epoch=”MJD” (ra)                  |
| `DE_ICRS`   | `double` | `deg`    | `pos.eq.dec;meta.main`  | Declination of the object (ICRS) at Epoch=”MJD” (dec)                     |
| `mode`      | `int8`   | `—`      | `meta.code.class`       | [1/4] PhotoMode (1=primary, 2=secondary, 3=family, 4=outside)             |
| `class`     | `int8`   | `—`      | `src.class`             | [0/8]? Type of object (3=galaxy, 6=star) (type) (2)                       |
| `clean`     | `bool`   | `—`      | `meta.code`             | [0/1] Clean photometry flag (1=clean; 0=unclean)                          |
| `flags`     | `int64`  | `—`      | `—`                     | —                                                                         |
| `upmag`     | `float`  | `—`      | `—`                     | —                                                                         |
| `gpmag`     | `float`  | `—`      | `—`                     | —                                                                         |
| `rpmag`     | `float`  | `—`      | `—`                     | —                                                                         |
| `ipmag`     | `float`  | `—`      | `—`                     | —                                                                         |
| `zpmag`     | `float`  | `—`      | `—`                     | —                                                                         |
| `e_upmag`   | `float`  | `—`      | `—`                     | —                                                                         |
| `e_gpmag`   | `float`  | `—`      | `—`                     | —                                                                         |
| `e_rpmag`   | `float`  | `—`      | `—`                     | —                                                                         |
| `e_ipmag`   | `float`  | `—`      | `—`                     | —                                                                         |
| `e_zpmag`   | `float`  | `—`      | `—`                     | —                                                                         |
| `uPmag`     | `float`  | `—`      | `—`                     | —                                                                         |
| `gPmag`     | `float`  | `—`      | `—`                     | —                                                                         |
| `rPmag`     | `float`  | `—`      | `—`                     | —                                                                         |
| `iPmag`     | `float`  | `—`      | `—`                     | —                                                                         |
| `zPmag`     | `float`  | `—`      | `—`                     | —                                                                         |
| `e_uPmag`   | `float`  | `—`      | `—`                     | —                                                                         |
| `e_gPmag`   | `float`  | `—`      | `—`                     | —                                                                         |
| `e_rPmag`   | `float`  | `—`      | `—`                     | —                                                                         |
| `e_iPmag`   | `float`  | `—`      | `—`                     | —                                                                         |
| `e_zPmag`   | `float`  | `—`      | `—`                     | —                                                                         |
| `rPrad`     | `float`  | `—`      | `—`                     | —                                                                         |
| `umag`      | `float`  | `mag`    | `phot.mag;em.opt.U`     | [4/37.3]? Model magnitude in u filter AB scale (u) (7)                    |
| `gmag`      | `float`  | `mag`    | `phot.mag;em.opt.B`     | [3.7/39.6]? Model magnitude in g filter AB scale (g) (7)                  |
| `rmag`      | `float`  | `mag`    | `phot.mag;em.opt.R`     | [3.8/40.4]? Model magnitude in r filter AB scale (r) (7)                  |
| `imag`      | `float`  | `mag`    | `phot.mag;em.opt.I`     | [2.6/39.5]? Model magnitude in i filter AB scale (i) (7)                  |
| `zmag`      | `float`  | `mag`    | `phot.mag;em.opt.I`     | [3.5/37.2]? Model magnitude in z filter AB scale (z) (7)                  |
| `e_umag`    | `float`  | `mag`    | `stat.error;phot.mag`   | [8.9e-8/3444]? Mean error on umag (err_u)                                 |
| `e_gmag`    | `float`  | `mag`    | `stat.error;phot.mag`   | [3.8e-8/5955]? Mean error on gmag (err_g)                                 |
| `e_rmag`    | `float`  | `mag`    | `stat.error;phot.mag`   | [5.2e-8/915]? Mean error on rmag (err_r)                                  |
| `e_imag`    | `float`  | `mag`    | `stat.error;phot.mag`   | [0/634]? Mean error on imag (err_i)                                       |
| `e_zmag`    | `float`  | `mag`    | `stat.error;phot.mag`   | [9.4e-8/3529]? Mean error on zmag (err_z)                                 |
| `objID`     | `int64`  | `—`      | `meta.id;meta.main`     | SDSS unique object identifier (1)                                         |
| `fieldID`   | `int64`  | `—`      | `—`                     | —                                                                         |
| `uPrad`     | `float`  | `—`      | `—`                     | —                                                                         |
| `gPrad`     | `float`  | `—`      | `—`                     | —                                                                         |
| `iPrad`     | `float`  | `—`      | `—`                     | —                                                                         |
| `zPrad`     | `float`  | `—`      | `—`                     | —                                                                         |
| `e_uPrad`   | `float`  | `—`      | `—`                     | —                                                                         |
| `e_gPrad`   | `float`  | `—`      | `—`                     | —                                                                         |
| `e_rPrad`   | `float`  | `—`      | `—`                     | —                                                                         |
| `e_iPrad`   | `float`  | `—`      | `—`                     | —                                                                         |
| `e_zPrad`   | `float`  | `—`      | `—`                     | —                                                                         |
| `udVrad`    | `float`  | `—`      | `—`                     | —                                                                         |
| `gdVrad`    | `float`  | `—`      | `—`                     | —                                                                         |
| `rdVrad`    | `float`  | `—`      | `—`                     | —                                                                         |
| `idVrad`    | `float`  | `—`      | `—`                     | —                                                                         |
| `zdVrad`    | `float`  | `—`      | `—`                     | —                                                                         |
| `udVell`    | `float`  | `—`      | `—`                     | —                                                                         |
| `gdVell`    | `float`  | `—`      | `—`                     | —                                                                         |
| `rdVell`    | `float`  | `—`      | `—`                     | —                                                                         |
| `idVell`    | `float`  | `—`      | `—`                     | —                                                                         |
| `zdVell`    | `float`  | `—`      | `—`                     | —                                                                         |
| `uPA`       | `float`  | `—`      | `—`                     | —                                                                         |
| `gPA`       | `float`  | `—`      | `—`                     | —                                                                         |
| `rPA`       | `float`  | `—`      | `—`                     | —                                                                         |
| `iPA`       | `float`  | `—`      | `—`                     | —                                                                         |
| `zPA`       | `float`  | `—`      | `—`                     | —                                                                         |
| `uFlags`    | `int64`  | `—`      | `—`                     | —                                                                         |
| `gFlags`    | `int64`  | `—`      | `—`                     | —                                                                         |
| `rFlags`    | `int64`  | `—`      | `—`                     | —                                                                         |
| `iFlags`    | `int64`  | `—`      | `—`                     | —                                                                         |
| `zFlags`    | `int64`  | `—`      | `—`                     | —                                                                         |
| `uc`        | `int8`   | `—`      | `—`                     | —                                                                         |
| `gc`        | `int8`   | `—`      | `—`                     | —                                                                         |
| `rc`        | `int8`   | `—`      | `—`                     | —                                                                         |
| `ic`        | `int8`   | `—`      | `—`                     | —                                                                         |
| `zc`        | `int8`   | `—`      | `—`                     | —                                                                         |
| `us`        | `float`  | `—`      | `—`                     | —                                                                         |
| `gs`        | `float`  | `—`      | `—`                     | —                                                                         |
| `rs`        | `float`  | `—`      | `—`                     | —                                                                         |
| `is`        | `float`  | `—`      | `—`                     | —                                                                         |
| `zs`        | `float`  | `—`      | `—`                     | —                                                                         |
| `e_RA_ICRS` | `double` | `arcsec` | `stat.error;pos.eq.ra`  | [0/51703] Mean error on RAdeg (raErr)                                     |
| `e_DE_ICRS` | `double` | `arcsec` | `stat.error;pos.eq.dec` | [0/22795] Mean error on DEdeg (decErr)                                    |
| `uoRA`      | `float`  | `—`      | `—`                     | —                                                                         |
| `goRA`      | `float`  | `—`      | `—`                     | —                                                                         |
| `roRA`      | `float`  | `—`      | `—`                     | —                                                                         |
| `ioRA`      | `float`  | `—`      | `—`                     | —                                                                         |
| `zoRA`      | `float`  | `—`      | `—`                     | —                                                                         |
| `uoDE`      | `float`  | `—`      | `—`                     | —                                                                         |
| `goDE`      | `float`  | `—`      | `—`                     | —                                                                         |
| `roDE`      | `float`  | `—`      | `—`                     | —                                                                         |
| `ioDE`      | `float`  | `—`      | `—`                     | —                                                                         |
| `zoDE`      | `float`  | `—`      | `—`                     | —                                                                         |
| `parentID`  | `int64`  | `—`      | `—`                     | —                                                                         |
| `SpObjID`   | `string` | `—`      | `—`                     | —                                                                         |
| `Status`    | `int16`  | `—`      | `—`                     | —                                                                         |
| `spInst`    | `string` | `—`      | `—`                     | —                                                                         |
| `spType`    | `string` | `—`      | `—`                     | —                                                                         |
| `zsp`       | `float`  | `—`      | `src.redshift`          | [-0.012/7.06]? Spectroscopic final redshift (when SpObjID>0) (spec_z) (6) |
| `e_zsp`     | `float`  | `—`      | `stat.error`            | [-6/30689]? Mean error on zsp (negative for bad fit) (spec_zErr) (6)      |
| `f_zsp`     | `int16`  | `—`      | `meta.code`             | [0/902]? Zwarning flag (spec_zWarning) (6)(9)                             |
| `spCl`      | `string` | `—`      | `—`                     | —                                                                         |
| `subCl`     | `string` | `—`      | `—`                     | —                                                                         |

### Images

No image cutout backends are configured for this survey.

### Spectra

No 1D spectroscopic backends are configured for this survey.

## References and acknowledgement

- Ahumada, R. et al. (2020), *The 16th Data Release of the Sloan Digital Sky Surveys: First Release from the APOGEE-2 Southern Survey and Full Release of eBOSS Spectra*, *ApJS*, 249, 3 (`doi:10.3847/1538-4365/ab929e`).
- Lupton, R. H., Gunn, J. E., & Szalay, A. S. (1999), *A Modified Magnitude System that Produces Well-Behaved Magnitudes, Colors, and Uncertainties Even at Zero or Negative Flux*, *AJ*, 118, 1406 (`doi:10.1086/301004`).
- Gunn, J. E. et al. (2006), *The 2.5 m Telescope of the Sloan Digital Sky Survey*, *AJ*, 131, 2332 (`doi:10.1086/500975`).
- VizieR HATS dataset: `https://vizcat.cds.unistra.fr/hats/sdss16/`.

Funding for the Sloan Digital Sky Survey IV has been provided by the Alfred P. Sloan Foundation, the U.S. Department of Energy Office of Science, and the Participating Institutions. SDSS-IV is managed by the Astrophysical Research Consortium for the Participating Institutions of the SDSS Collaboration including the Brazilian Participation Group, the Carnegie Institution for Science, Carnegie Mellon University, the Chilean Participation Group, the French Participation Group, Harvard-Smithsonian Center for Astrophysics, Instituto de Astrofísica de Canarias, The Johns Hopkins University, Kavli Institute for the Physics and Mathematics of the Universe (IPMU) / University of Tokyo, the Lawrence Berkeley National Laboratory, Leibniz Institut für Astrophysik Potsdam (AIP), Max-Planck-Institut für Astronomie (MPIA Heidelberg), Max-Planck-Institut für Astrophysik (MPA Garching), Max-Planck-Institut für Extraterrestrische Physik (MPE), National Astronomical Observatories of China, New Mexico State University, New York University, University of Notre Dame, Observatório Nacional / MCTI, The Ohio State University, Pennsylvania State University, Shanghai Astronomical Observatory, United Kingdom Participation Group, Universidad Nacional Autónoma de México, University of Arizona, University of Colorado Boulder, University of Oxford, University of Portsmouth, University of Utah, University of Virginia, University of Washington, University of Wisconsin, Vanderbilt University, and Yale University.

## Provenance of this page

| Source ID     | Kind           | Service / Endpoint                           | Query or URL                                                | Retrieved (UTC)        |
|---------------|----------------|----------------------------------------------|-------------------------------------------------------------|------------------------|
| `hats_sdss16` | `hats_catalog` | `https://vizcat.cds.unistra.fr/hats/sdss16/` | `https://vizcat.cds.unistra.fr/hats/sdss16/main/properties` | `2026-10-18T16:59:19Z` |
<!-- /generated -->
