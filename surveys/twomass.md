# 2MASS Extended Source Catalog (`twomass`)

<!-- generated:survey-page -->

The Two Micron All Sky Survey (2MASS) Extended Source Catalog (XSC) provides all-sky near-infrared imaging and photometry in three passbands: $J$ (1.25 $\mu$m), $H$ (1.65 $\mu$m), and $K_s$ (2.17 $\mu$m). Observed from Mount Hopkins, Arizona and Cerro Tololo, Chile, 2MASS mapped 99.998% of the celestial sphere.

In `mosaics`, `twomass` provides near-infrared photometry and morphological parameters for resolved extended sources (predominantly galaxies). The slug is spelled `twomass` rather than `2mass` because column aliases become SQL identifiers, which cannot begin with a leading numeral. Rather than making remote archive queries, MOSAICS downloads the complete catalogue (1,647,599 sources, VizieR VII/233/xsc) once as a local FITS table, querying it cell by cell using DuckDB.

| Property                 | Value                                                                                                                                                                                                                                                                                         |
|--------------------------|-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| What MOSAICS extracts    | Catalogue: `twomass` (photometric, `local`)                                                                                                                                                                                                                                                   |
| Archives                 | `cdsarc.cds.unistra.fr`, `tapvizier.cds.unistra.fr`                                                                                                                                                                                                                                           |
| Bands / wavelength range | `j` (`2MASS/2MASS.J`), `h` (`2MASS/2MASS.H`), `ks` (`2MASS/2MASS.Ks`)                                                                                                                                                                                                                         |
| Used by                  | `mosaics-targets` (crossmatch), `mosaics-crossmatch`, `mosaics-query`                                                                                                                                                                                                                         |
| Crossmatch radius        | `3` arcsec. Effective resolution of the 2MASS system about 5 arcsec (Explanatory Supplement, II.2); the Extended Source Catalog against VLA FIRST has an RMS of 0.85-0.87 arcsec, an upper bound because it includes radio-optical scatter. The grid value nearest 0.75 x 5 arcsec was taken. |

## Configuration

### The shipped file

Source: `defs/surveys/twomass.yaml`

```yaml
# 2MASS All-Sky Extended Source Catalog -- photometry.
#
# The slug is `twomass`, not `2mass`: the aliases become SQL identifiers and
# one cannot start with a digit.

survey_slug: twomass
name: 2MASS Extended Source Catalog

catalog:
  - name: twomass
    type: photometric
    adapter: local
    adapter_params:
      # A downloaded file, not an archive. The whole catalogue -- 1,647,599
      # sources -- is one FITS table from CDS, fetched once by the download/xsc
      # step and matched on this machine with DuckDB. Against Astro Data Lab
      # every Legacy Survey row of a cell was uploaded to find the handful of
      # galaxies 2MASS resolved in it.
      file:
        url: https://cdsarc.cds.unistra.fr/viz-bin/nph-Cat/fits.gz?VII/233/xsc.dat
        table: xsc
      # VizieR's labels for the peak-pixel position, which IRSA and Data Lab
      # call ra/dec. The file's columns are the catalogue's ReadMe labels, so
      # nothing here can be inferred.
      remote_ra_column: RAdeg
      remote_dec_column: DEdeg

# Crossmatch: the radius a table of positions is matched within, and what it rests on.
crossmatch:
  radius: 3 arcsec
  radius_note: >-
    Effective resolution of the 2MASS system about 5 arcsec (Explanatory Supplement, II.2);
    the Extended Source Catalog against VLA FIRST has an RMS of 0.85-0.87 arcsec, an upper
    bound because it includes radio-optical scatter. The grid value nearest 0.75 x 5 arcsec
    was taken.

photometric_system:
  filters:
    # The same passbands as the point-source catalogue: both are measured on
    # the same survey images.
    j: 2MASS/2MASS.J
    h: 2MASS/2MASS.H
    ks: 2MASS/2MASS.Ks
  vega_offsets:
    # Blanton & Roweis 2007. SVO's zero points reproduce these to three
    # decimals, which is the cleanest confirmation available that the filter
    # registry points at the right passbands.
    j: 0.894
    h: 1.374
    ks: 1.840
  measurements:
    # The XSC has no default magnitude; see below for why k20fe.
    - system: vega_mag
      apertures: [k20fe]

# 2MASS All-Sky Extended Source Catalog: near-infrared JHKs photometry of the
# ~1.6 million sources 2MASS resolved, nearly all of them galaxies.
# File: VizieR VII/233/xsc, as CDS's nph-Cat serves it -- an ASCII FITS table
# whose column names are the ReadMe labels. Each alias's comment names the
# original 2MASS column in parentheses, which is what Data Lab and IRSA call
# it, so the aliases are the ones this survey had before it moved here.
#
# The extended catalogue rather than the point-source one because the targets
# are galaxies. The PSC measures every source as though it were unresolved,
# which underestimates the light of anything that is not; the XSC measures a
# source through apertures fitted to its own light. The price is coverage:
# ~1.6M sources against the PSC's ~471M, so most candidates -- and every star
# among them -- match nothing.
#
# There is no default magnitude: every XSC magnitude names the aperture it was
# measured through. This one is `k20fe`, the fiducial elliptical aperture at
# the Ks-band 20 mag/arcsec^2 isophote, applied unchanged to J and H. One
# aperture for all three bands is what makes a colour from these measure the
# same light twice rather than two different areas of the galaxy.
# `twomass_r_k20fe` is its semi-major axis.
#
# Magnitudes are Vega. The catalogue spells the band `K`; it is aliased to
# `ks` because the passband genuinely is Ks, and a consumer asking for
# `twomass_mag_k20fe_ks` should not have to know that 2MASS abbreviates it.
#
# Nullable columns (`?` in the ReadMe) are blank where the value is missing,
# and the file declares no TNULL for them. The conversion reads a blank field
# as null whatever its type -- astropy would read a blank integer as 0, which
# for a confusion flag means "clean". `cc`, `fgFlg` and `edgeFlg` are text:
# `cc` is `0` for all but ~1,300 sources and a letter for those.
column_mapping:
  extKey: 'id'                           # entry counter (ext_key)
  RAdeg: 'ra'                            # right ascension, J2000, peak pixel, deg (ra)
  DEdeg: 'dec'                           # declination, J2000, peak pixel, deg (dec)

  J.K20e: 'mag_k20fe_j'                  # J magnitude in the K20e ellipse, Vega mag (j_m_k20fe)
  e_J.K20e: 'mag_k20fe_j_err'            # 1-sigma uncertainty, mag (j_msig_k20fe)
  f_J.K20e: 'flag_k20fe_j'               # J confusion flag (j_flg_k20fe)

  H.K20e: 'mag_k20fe_h'                  # H magnitude in the K20e ellipse, Vega mag (h_m_k20fe)
  e_H.K20e: 'mag_k20fe_h_err'            # 1-sigma uncertainty, mag (h_msig_k20fe)
  f_H.K20e: 'flag_k20fe_h'               # H confusion flag (h_flg_k20fe)

  K.K20e: 'mag_k20fe_ks'                 # Ks magnitude in the K20e ellipse, Vega mag (k_m_k20fe)
  e_K.K20e: 'mag_k20fe_ks_err'           # 1-sigma uncertainty, mag (k_msig_k20fe)
  f_K.K20e: 'flag_k20fe_ks'              # Ks confusion flag (k_flg_k20fe)

  r.K20e: 'r_k20fe'                      # semi-major axis of the K20e ellipse, arcsec (r_k20fe)
  r3sig: 'r_3sig'                        # semi-major axis of the 3-sigma Ks isophote, arcsec (r_3sig)
  Sb/a: 'axis_ratio'                     # minor/major axis ratio of the 3-sigma super-coadd isophote (sup_ba)
  Spa: 'pa'                              # position angle of that ellipse, deg E of N (sup_phi)
  Jconc: 'concentration_index_j'         # J concentration index (j_con_indx)
  Hconc: 'concentration_index_h'         # H concentration index (h_con_indx)
  Kconc: 'concentration_index_ks'        # Ks concentration index (k_con_indx)
  n_J.ext: 'chi2_fit_j'                  # chi^2 of the J radial-profile fit (j_pchi)
  n_H.ext: 'chi2_fit_h'                  # chi^2 of the H radial-profile fit (h_pchi)
  n_K.ext: 'chi2_fit_ks'                 # chi^2 of the Ks radial-profile fit (k_pchi)

  xScore: 'class_extended'               # extended score, 1 (extended) to 2 (point-like) (e_score)
  gScore: 'class_galaxy'                 # galaxy score, 1 (extended) to 2 (point-like) (g_score)
  # vc: 'vc'                             # visual verification of the source (vc)

  cc: 'flag_cc'                          # artifact contamination and/or confusion flag (cc_flg)
  ctmFlg: 'flag_contam'                  # 1 where stellar flux is included in the magnitudes (contam_flg)
  fgFlg: 'flag_flux'                     # flux-growth convergence flag (fg_flg)
  sysFlg: 'flag_sys'                     # 1 where a nearby galaxy's flux is included (sys_flg)
  useSrc: 'flag_use_src'                 # the preferred detection of a tile-overlap duplicate (use_src)
  dupSrc: 'flag_dup_src'                 # detected more than once in a tile overlap (dup_src)
  edgeFlg: 'flag_dist_edge'              # nearest scan edges, [n|s][e|w] (dist_edge_flg)

  # xid: 'flag_id'                       # 0 in no other catalogue, 1 in a galaxy catalogue, 2 LCSB (id_flg)
  # idGal: 'flag_id_cat'                 # the galaxy catalogue matched, where xid is 1 (id_cat)
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

**Catalogue `twomass` (`adapter: local`):**

| Parameter           | Allowed Type             | Default   | Description                                                                         |
|---------------------|--------------------------|-----------|-------------------------------------------------------------------------------------|
| `file`              | mapping (`{url, table}`) | —         | Remote file URL and local DuckDB table name (see `local_file` keys).                |
| `remote_ra_column`  | str                      | —         | Right ascension column name in the downloaded table (e.g. `plug_ra`, `sup_ra`).     |
| `remote_dec_column` | str                      | —         | Declination column name in the downloaded table (e.g. `plug_dec`, `sup_dec`).       |
| `dialect`           | str                      | `limit`   | SQL row-limit dialect (`limit` for DuckDB).                                         |
| `use_async`         | bool                     | false     | Accepted for uniform adapter signature; local DuckDB queries always run in-process. |

### Common changes

- Disable 2MASS XSC crossmatching in an entrypoint configuration by setting:
  ```yaml
  overrides:
    surveys/twomass:
      catalog: []
  ```

  or setting `enabled: false` on the catalogue entry.
- Extract additional morphological flags by extending `column_mapping` in an entrypoint override (for instance, uncommenting visual verification flags `vc` or galaxy catalogue identifications `xid`, `idGal`).
- Unmatched targets: Because the XSC covers only resolved extended sources (~1.65 million sources all-sky), point sources and stars will not match and will naturally carry `null` in all `twomass_*` columns.

### Commands

```console
$ uv run -m mosaics.crossmatch.cli run --survey surveys/twomass -i test/objects_sample_1k.csv -o twomass_matched.parquet
$ uv run -m mosaics.cli.targets --config defs/targets.yaml -o ./targets_run --dec-range 0 1 --plan plan.yaml
```

## Data served by the archive

- Extended Source Catalog versus Point Source Catalog: The pipeline deliberately matches against the XSC rather than the Point Source Catalog (PSC). Although the PSC contains ~471 million detections, it models every source as unresolved, underestimating the light of extended galaxies. The XSC measures light through apertures fitted to extended galaxy profiles. The trade-off is coverage: with ~1.65 million sources all-sky, stellar targets will not match the XSC.
- Matched elliptical aperture (`k20fe`): The XSC provides multiple aperture definitions without a single default magnitude. MOSAICS standardizes on the `k20fe` fiducial elliptical aperture, measured at the $K_s$-band 20 mag/arcsec² isophote and applied identically to the $J$ and $H$ images. Applying the same aperture across all three bands ensures that colors measure the same physical region of the galaxy. The semi-major axis is recorded in `twomass_r_k20fe`.
- Passband naming and Vega offsets: Magnitudes are in the Vega system. The catalogue names the third band `K`, but MOSAICS aliases it to `ks` because the passband is genuinely $K_s$ ($2.17\ \mu\text{m}$). Vega-to-AB offsets (Blanton & Roweis 2007) are:
  - $J$: 0.894 mag
  - $H$: 1.374 mag
  - $K_s$: 1.840 mag
- Null handling in ASCII FITS: Missing values in VizieR VII/233 are blank strings without explicit `TNULL` headers. The loader converts blank values to SQL `null` regardless of column data type, preventing blank integers from being erroneously parsed as `0` (which would signify “clean” for confusion flags like `cc`).

### Catalogue: `VII/233/xsc`

\*The 2MASS Extended Catalog (2MASX)

**Service URL:** `https://tapvizier.cds.unistra.fr/TAPVizieR/tap`

| Column    | Type           | Unit     | UCD                     | Description                                                                       |
|-----------|----------------|----------|-------------------------|-----------------------------------------------------------------------------------|
| `recno`   | `int`          | `—`      | `meta.record`           | Record number assigned by the VizieR team. Should Not be used for identification. |
| `2MASX`   | `char`         | `—`      | `meta.id;meta.main`     | Source name (HHMMSSss+DDMMSSs, J2000)                                             |
| `All`     | `char`         | `—`      | `meta.ref.url`          | Display all original 369 columns for this 2MASX source                            |
| `RAJ2000` | `double`       | `deg`    | `pos.eq.ra;meta.main`   | Right ascension (J2000), based on peak pixel                                      |
| `DEJ2000` | `double`       | `deg`    | `pos.eq.dec;meta.main`  | Declination (J2000), based on peak pixel                                          |
| `r3sig`   | `float`        | `arcsec` | `phys.angSize.smajAxis` | ? Semi-major axis of 3{sigma} K isophotal ellipse (r_3sig)                        |
| `Sb/a`    | `float`        | `—`      | `phys.size.axisRatio`   | ? Minor/major axis ratio to rS3sig (sup_ba)                                       |
| `Spa`     | `short`        | `deg`    | `pos.posAng`            | ? Pos.angle of rS3sig (sup_phi)                                                   |
| `J.ext`   | `float`        | `mag`    | `phot.mag;em.IR.J`      | ? J magnitude in r.ext (j_m_ext)                                                  |
| `e_J.ext` | `float`        | `mag`    | `stat.error`            | ? {sigma}(J.ext) (j_msig_ext)                                                     |
| `H.ext`   | `float`        | `mag`    | `phot.mag;em.IR.H`      | ? J magnitude in r.ext (h_m_ext)                                                  |
| `e_H.ext` | `float`        | `mag`    | `stat.error`            | ? {sigma}(H.ext) (h_msig_ext)                                                     |
| `K.ext`   | `float`        | `mag`    | `phot.mag;em.IR.K`      | ? J magnitude in r.ext (k_m_ext)                                                  |
| `e_K.ext` | `float`        | `mag`    | `stat.error`            | ? {sigma}(K.ext) (k_msig_ext)                                                     |
| `xid`     | `unsignedByte` | `—`      | `meta.code`             | [0,1] Type/galaxy source (id_flg) (7)                                             |

### Images

No image cutout backends are configured for this survey.

### Spectra

No 1D spectroscopic backends are configured for this survey.

## References and acknowledgement

- Jarrett, T. H. et al. (2000), *2MASS Extended Source Catalog: Overview and Cut Analysis*, *AJ*, 119, 2498 (`doi:10.1086/301330`).
- Skrutskie, M. F. et al. (2006), *The Two Micron All Sky Survey (2MASS)*, *AJ*, 131, 1163 (`doi:10.1086/498708`).
- Blanton, M. R. & Roweis, S. (2007), *K-Corrections and Filter Transformations in the Ultraviolet, Optical, and Near-Infrared*, *AJ*, 133, 734 (`doi:10.1086/510127`).
- VizieR VII/233/xsc: `https://cdsarc.cds.unistra.fr/viz-bin/nph-Cat/fits.gz?VII/233/xsc.dat`.

This publication makes use of data products from the Two Micron All Sky Survey, which is a joint project of the University of Massachusetts and the Infrared Processing and Analysis Center/California Institute of Technology, funded by the National Aeronautics and Space Administration and the National Science Foundation.

## Provenance of this page

| Source ID             | Kind        | Service / Endpoint                               | Query or URL                                                                            | Retrieved (UTC)        |
|-----------------------|-------------|--------------------------------------------------|-----------------------------------------------------------------------------------------|------------------------|
| `catalog_twomass_xsc` | `tap_table` | `https://tapvizier.cds.unistra.fr/TAPVizieR/tap` | `https://vizier.cds.unistra.fr/viz-bin/votable?-source=VII/233/xsc&-out.max=0&-out.all` | `2026-10-18T16:59:35Z` |
<!-- /generated -->
