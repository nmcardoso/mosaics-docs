# Gaia DR3 (`gaia3`)

<!-- generated:survey-page -->

Gaia Data Release 3 (Gaia DR3) is a comprehensive full-sky survey executed by the European Space Agency’s Gaia space observatory. DR3 delivers ultra-precise five-parameter astrometry (positions, proper motions, and parallaxes), broad-band photometry, radial velocities, and astrophysical characterization for approximately 1.8 billion sources down to $G \approx 21$. Observations are conducted across three optical passbands: $G$ (broad optical, $330\text{--}1050\text{ nm}$), $G_\text{BP}$ (blue photometer, $330\text{--}680\text{ nm}$), and $G_\text{RP}$ (red photometer, $640\text{--}1050\text{ nm}$).

In `mosaics`, Gaia DR3 provides high-precision astrometric anchoring and optical broad-band photometry during crossmatching (`mosaics-targets` and `mosaics-query` via the NOIRLab Astro Data Lab TAP service table `gaia_dr3.gaia_source`). Its HiPS cutout entries are explicitly disabled because the registered CDS tiles are catalogue density maps rather than calibrated imaging pixels.

| Property                 | Value                                                                                                                                                                                                                                                   |
|--------------------------|---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| What MOSAICS extracts    | Catalogue: `gaia3` (photometric, `adl-healpix`); Cutouts: `hips` (color), `hips` (g), `hips` (bp), `hips` (rp)                                                                                                                                          |
| Archives                 | CDS HiPS / hips2fits (`alasky.cds.unistra.fr`), `datalab.noirlab.edu`                                                                                                                                                                                   |
| Bands / wavelength range | `g` (`GAIA/GAIA3.G`), `bp` (`GAIA/GAIA3.Gbp`), `rp` (`GAIA/GAIA3.Grp`)                                                                                                                                                                                  |
| Used by                  | `mosaics-targets` (crossmatch), `mosaics-cutout full`, `mosaics-crossmatch`                                                                                                                                                                             |
| Crossmatch radius        | `1` arcsec. Close pairs are complete only above about 1.5 arcsec (Gaia EDR3, arXiv:2012.01533, Sect. 3); position uncertainty 0.01-0.02 mas at G < 15 and 1.0 mas at G = 21, reference epoch 2016.0. Proper motion between epochs is not accounted for. |

## Configuration

### The shipped file

Source: `defs/surveys/gaia3.yaml`

```yaml
# Gaia DR3 -- photometry, and cutouts from a HiPS flux-colour rendering.

survey_slug: gaia3
name: Gaia DR3

catalog:
  - name: gaia3
    type: photometric
    # Astro Data Lab's public TAP, by ranges of the table's `nest4096` index: no login, no
    # upload; the rows that come back are a superset of the matches and the match itself is
    # made here by haversine separation (HG3, 2026-10-20: Data Lab accounts can no longer be
    # authenticated, so the `adl` upload route is unusable). `nest4096` checked in
    # TAP_SCHEMA on 2026-10-20.
    adapter: adl-healpix
    adapter_params:
      table: gaia_dr3.gaia_source

# Crossmatch: the radius a table of positions is matched within, and what it rests on.
crossmatch:
  radius: 1 arcsec
  radius_note: >-
    Close pairs are complete only above about 1.5 arcsec (Gaia EDR3, arXiv:2012.01533, Sect.
    3); position uncertainty 0.01-0.02 mas at G < 15 and 1.0 mas at G = 21, reference epoch
    2016.0. Proper motion between epochs is not accounted for.

cutout:
  - adapter: hips
    # Verified against the MOCServer 2026-09-05: this id is registered.
    adapter_params:
      hips_id: CDS/P/DM/flux-color-Rp-G-Bp/I/355/gaiadr3
    bands: [color]
    enabled: false
  # The per-band siblings, registered and verified alongside it on the same
  # date, one plane each rather than the colour one's four. Disabled for
  # the same reason: a `CDS/P/DM/...` tile is binned catalogue flux, not
  # survey pixels, so a single plane is no more calibratable than three.
  # They are named because "does gaia3 serve g?" now has a place to be
  # asked, and the honest answer is "only as a density map".
  - adapter: hips
    adapter_params:
      hips_id: CDS/P/DM/flux-G/I/355/gaiadr3
    bands: [g]
    enabled: false
  - adapter: hips
    adapter_params:
      hips_id: CDS/P/DM/flux-Bp/I/355/gaiadr3
    bands: [bp]
    enabled: false
  - adapter: hips
    adapter_params:
      hips_id: CDS/P/DM/flux-Rp/I/355/gaiadr3
    bands: [rp]
    enabled: false

photometric_system:
  filters:
    # GAIA3 rather than GAIA2, so the curves match the DR3 zero points.
    g: GAIA/GAIA3.G
    bp: GAIA/GAIA3.Gbp
    rp: GAIA/GAIA3.Grp
  measurements:
    # e-/s scaled straight to uJy by Riello's per-band constant. Published as
    # c_nu in W m^-2 Hz^-1 (1.736011, 2.620707, 3.298815) x 1e-33, and
    # 1 W m^-2 Hz^-1 is 1e32 uJy. Equivalent to going through the AB zero
    # points 25.8010 / 25.3540 / 25.1040, and preferred over them because it
    # is one multiplication on a linear flux rather than a round trip through
    # a magnitude. (Riello et al. 2021, A&A 649, A3.)
    - system: linear
      apertures: [default]
      scale:
        g: 0.1736011
        bp: 0.2620707
        rp: 0.3298815
  pixel_unit: unknown
  pixel_unit_note: >-
    Resolved to a final `unknown`. `CDS/P/DM/...` is a CDS *density map*: a
    rendering built by binning the Gaia catalogue onto HEALPix, not imaging,
    so there are no survey pixels to calibrate even in principle. hips2fits
    returns a 4-plane uint8 RGBA image in [0, 255] with no BUNIT (checked
    2026-09-04), confirming it. The adapter stays disabled; it would also
    fail on shape, one column against four planes.

# Gaia DR3 main source catalogue: astrometry, three-band photometry,
# radial velocities and DSC classification probabilities.
# Table: gaia_dr3.gaia_source (Astro Data Lab TAP)
# Crossmatched spatially against target_candidates on (ra, dec).
#
# Photometry is native e-/s on the VEGAMAG scale. The published fluxes are the
# quantity to homogenise from -- Riello et al. 2021 gives a direct per-band
# f_nu constant for them -- so both the flux and its error are kept alongside
# the magnitude.
column_mapping:
  # source_id: 'id'             # unique Gaia DR3 source identifier
  ra: 'ra'                      # barycentric right ascension, ICRS, deg
  dec: 'dec'                    # barycentric declination, ICRS, deg
  # ra_error: 'ra_err'          # standard error of ra, mas
  # dec_error: 'dec_err'        # standard error of dec, mas

  parallax: 'parallax'                     # absolute stellar parallax, mas
  parallax_error: 'parallax_err'           # standard error of the parallax, mas
  parallax_over_error: 'parallax_snr'      # parallax divided by its error
  pmra: 'pmra'                             # proper motion in ra * cos(dec), mas/yr
  pmra_error: 'pmra_err'                   # standard error of pmra, mas/yr
  pmdec: 'pmdec'                           # proper motion in dec, mas/yr
  pmdec_error: 'pmdec_err'                 # standard error of pmdec, mas/yr
  pm: 'pm'                                 # total proper motion, mas/yr
  ruwe: 'ruwe'                             # renormalised unit weight error of the astrometric fit

  astrometric_excess_noise: 'excess_noise'               # excess noise of the astrometric source, mas
  astrometric_excess_noise_sig: 'excess_noise_sig'       # significance of the excess noise

  phot_g_mean_mag: 'mag_g'                     # G-band mean magnitude, VEGAMAG
  phot_g_mean_flux: 'flux_g'                   # G-band mean flux, e-/s
  phot_g_mean_flux_error: 'flux_g_err'         # error on the G-band mean flux, e-/s
  phot_bp_mean_mag: 'mag_bp'                   # BP-band mean magnitude, VEGAMAG
  phot_bp_mean_flux: 'flux_bp'                 # BP-band mean flux, e-/s
  phot_bp_mean_flux_error: 'flux_bp_err'       # error on the BP-band mean flux, e-/s
  phot_rp_mean_mag: 'mag_rp'                   # RP-band mean magnitude, VEGAMAG
  phot_rp_mean_flux: 'flux_rp'                 # RP-band mean flux, e-/s
  phot_rp_mean_flux_error: 'flux_rp_err'       # error on the RP-band mean flux, e-/s

  phot_bp_rp_excess_factor: 'bp_rp_excess'      # BP/RP flux excess factor, a blending diagnostic

  radial_velocity: 'rv'                  # spectroscopic radial velocity, km/s
  radial_velocity_error: 'rv_err'        # error on the radial velocity, km/s

  classprob_dsc_combmod_star: 'prob_star'           # DSC combmod probability the source is a star
  classprob_dsc_combmod_galaxy: 'prob_galaxy'       # DSC combmod probability the source is a galaxy
  classprob_dsc_combmod_quasar: 'prob_quasar'       # DSC combmod probability the source is a quasar

  duplicated_source: 'flag_dup'                    # flag: source had duplicate observations removed
  astrometric_params_solved: 'astrom_params'       # which astrometric parameters were solved for
  phot_variable_flag: 'flag_variable'              # photometric variability flag
  non_single_star: 'non_single_star'               # flag: astrometric/spectroscopic/eclipsing binary
  visibility_periods_used: 'vis_periods'           # number of visibility periods used in the astrometry
```

### Schema of this file

| Key present          | Type                             | Required   | Default   | Meaning                                                                                                                                                                                                                      |
|----------------------|----------------------------------|------------|-----------|------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `survey_slug`        | str                              | yes        | —         | Canonical lowercase identifier for the survey and default column prefix.                                                                                                                                                     |
| `name`               | str                              | yes        | —         | Human-readable survey and data-release name used in logs, plans, and reports.                                                                                                                                                |
| `catalog`            | list of catalog entries          | no         | []        | Remote or local catalogues queried for target extraction or crossmatching.                                                                                                                                                   |
| `crossmatch`         | mapping                          | no         | None      | How a table of positions is matched against this survey. Required for a survey with an `lsdb` catalog; `crossmatch` null removes it, and a survey without it takes no part in `mosaics-crossmatch`.                          |
| `cutout`             | list of cutout entries           | no         | []        | Image cutout backends (`hips`, `lsv`, `nersc`, `sia`, `vsa`) serving this survey’s passbands.                                                                                                                                |
| `photometric_system` | mapping                          | no         | None      | Passband SVO filter identifiers, flux/magnitude homogenisation rules, and image pixel calibration.                                                                                                                           |
| `column_mapping`     | mapping (remote_column -> alias) | no         | {}        | Projection and column renaming applied when crossmatching against this survey’s photometric catalogue. The loader adds the catalog’s prefix to every alias, or the hosted survey’s for the columns under its `provides` key. |

### Adapter parameters

**Catalogue `gaia3` (`adapter: adl-healpix`):**

| Parameter              | Allowed Type   | Default                                                                   | Description                                                                                                                                            |
|------------------------|----------------|---------------------------------------------------------------------------|--------------------------------------------------------------------------------------------------------------------------------------------------------|
| `table`                | str            | —                                                                         | Fully qualified table on Astro Data Lab that carries a HEALPix NESTED index column (e.g. `gaia_dr3.gaia_source`, `allwise.source`, `ls_dr11.tractor`). |
| `service`              | str            | `datalab` (the public TAP at `https://datalab.noirlab.edu/tap`, no login) | A full TAP endpoint or a shorthand of the TAP services, for a test archive.                                                                            |
| `remote_ra_column`     | str            | found by UCD (`pos.eq.ra`), then by name                                  | Remote right ascension column name (`mean_fiber_ra` for `desi_dr1.zpix`).                                                                              |
| `remote_dec_column`    | str            | found by UCD (`pos.eq.dec`), then by name                                 | Remote declination column name.                                                                                                                        |
| `index_column`         | str            | `nest4096`                                                                | The table’s HEALPix NESTED index. Its values are checked against the returned positions on the first query of every run.                               |
| `index_order`          | int            | 12 (the order of `nest4096`)                                              | The HEALPix order of `index_column`.                                                                                                                   |
| `max_rows_per_query`   | int            | 50000                                                                     | Most rows one query may return. A query whose result reaches it is split and asked again, never truncated.                                             |
| `max_ranges_per_query` | int            | 50                                                                        | Most index ranges one query may carry. The public TAP times out well above this (probed at 100 and 500).                                               |
| `rows_per_deg2`        | float          | None (counted by a few `COUNT(*)` queries)                                | The table’s mean source density per square degree. When given, the planner sends no counting query.                                                    |
- **Cutout `None` (`adapter: hips`, bands `['color']`):** configured `adapter_params`: `hips_id='CDS/P/DM/flux-color-Rp-G-Bp/I/355/gaiadr3'`.
- **Cutout `None` (`adapter: hips`, bands `['g']`):** configured `adapter_params`: `hips_id='CDS/P/DM/flux-G/I/355/gaiadr3'`.
- **Cutout `None` (`adapter: hips`, bands `['bp']`):** configured `adapter_params`: `hips_id='CDS/P/DM/flux-Bp/I/355/gaiadr3'`.
- **Cutout `None` (`adapter: hips`, bands `['rp']`):** configured `adapter_params`: `hips_id='CDS/P/DM/flux-Rp/I/355/gaiadr3'`.

### Common changes

- **Coordinate error and identifier columns**: In `column_mapping`, uncomment astrometric coordinate uncertainties (`ra_error: 'ra_err'`, `dec_error: 'dec_err'`) or the unique 64-bit integer identifier (`source_id: 'id'`); the loader adds the `gaia3_` prefix to every alias.
- **Astrometric quality filtering**: Filter out perturbed or unrelaxed astrometric solutions by applying a cut on the renormalised unit weight error in your analysis pipeline (e.g. `gaia3_ruwe < 1.4` selects reliable single-star astrometric solutions).
- **Photometric blending and variability diagnostics**: Use the BP/RP excess factor `gaia3_bp_rp_excess` to identify extended objects, contaminated apertures, or severe background blending, and `gaia3_flag_variable` to select or exclude variable stars.
- **Disable catalogue crossmatching**: Disable Gaia DR3 crossmatching in an entrypoint configuration by setting `overrides[surveys/gaia3].catalog.0.enabled: false`.
- **Do not enable HiPS cutouts for pixel science**: All four HiPS cutout entries (`color`, `g`, `bp`, `rp`) are disabled (`enabled: false`). Do not enable them for photometric science, as they yield 8-bit density maps rather than physical pixel values.

### Commands

```console
$ uv run -m mosaics.cli.cutout full -t test/objects_sample_1k.csv -o cutouts.lance --config defs/cutout.yaml --survey gaia3 --plan
$ uv run -m mosaics.crossmatch.cli run --survey surveys/gaia3 -i test/objects_sample_1k.csv -o gaia3_matched.parquet
$ uv run -m mosaics.cli.targets --config defs/targets.yaml -o ./targets_run --dec-range 0 1 --plan plan.yaml
```

## Data served by the archive

- **Photometric flux conversion and linear scaling**: Native Gaia photometry is measured in photo-electrons per second ($\text{e}^-/\text{s}$) on the Vega system (`VEGAMAG`). Rather than converting through magnitude transformations, `mosaics` scales fluxes directly to physical microjanskys ($\mu\text{Jy}$) using the absolute flux constants $c_\nu$ published by Riello et al. (2021):
  - $G$ band scale factor: $0.1736011$
  - $G_\text{BP}$ band scale factor: $0.2620707$
  - $G_\text{RP}$ band scale factor: $0.3298815$
    These constants correspond to $1.736011 \times 10^{-33}$, $2.620707 \times 10^{-33}$, and $3.298815 \times 10^{-33}\text{ W m}^{-2}\text{ Hz}^{-1}\text{ (e}^-/\text{s})^{-1}$ (equivalent to AB zero points 25.8010, 25.3540, and 25.1040).
- **HiPS tiles are CDS density maps**: The registered CDS HiPS identifiers (`CDS/P/DM/flux-color-Rp-G-Bp/I/355/gaiadr3`, `CDS/P/DM/flux-G/...`, `CDS/P/DM/flux-Bp/...`, `CDS/P/DM/flux-Rp/...`) are binned HEALPix density renderings of the catalogue, not optical telescope exposures. Queries to hips2fits return 4-plane uint8 RGBA images in $[0, 255]$ with no standard FITS `BUNIT` or flux zero points. As a result, `pixel_unit` is designated as `unknown` and all cutout adapters remain disabled (`enabled: false`).
- **Astrometric excess noise**: Sources with significant unmodelled orbital motion or morphology carry non-zero `astrometric_excess_noise` and `astrometric_excess_noise_sig`, which should be checked when selecting point sources.

### Catalogue: `gaia_dr3.gaia_source`

This table has an entry for every Gaia observed source as publis

**Service URL:** `https://datalab.noirlab.edu/tap`

*Showing 100 of 156 total archive columns (all MOSAICS-mapped columns plus primary table fields; full 156-column schema is stored in `docs/_data/sources/gaia3/catalog_gaia_dr3.json`).*

| Column                         | Type            | Unit       | UCD                    | Description                                                      |
|--------------------------------|-----------------|------------|------------------------|------------------------------------------------------------------|
| `astrometric_excess_noise`     | `adql:REAL`     | `mas`      | `—`                    | Excess noise of the source Angle[mas]                            |
| `astrometric_excess_noise_sig` | `adql:REAL`     | `—`        | `—`                    | Significance of excess noise                                     |
| `dec`                          | `adql:DOUBLE`   | `deg`      | `pos.eq.dec;meta.main` | Declination Angle[deg]                                           |
| `parallax`                     | `adql:DOUBLE`   | `mas`      | `—`                    | Parallax Angle[mas]                                              |
| `parallax_error`               | `adql:REAL`     | `mas`      | `—`                    | Standard error of Parallax Angle[mas]                            |
| `parallax_over_error`          | `adql:REAL`     | `mas`      | `—`                    | Standard error of parallax Angle[mas]                            |
| `phot_bp_mean_flux`            | `adql:DOUBLE`   | `e-/s`     | `—`                    | Integrated BP mean flux Flux[e-/s]                               |
| `phot_bp_mean_flux_error`      | `adql:REAL`     | `e-/s`     | `—`                    | Error on the integrated BP mean flux Flux[e-/s]                  |
| `phot_bp_mean_mag`             | `adql:REAL`     | `mag`      | `—`                    | Integrated BP mean magnitude Magnitude[mag]                      |
| `phot_bp_rp_excess_factor`     | `adql:REAL`     | `—`        | `—`                    | BP/RP excess factor                                              |
| `phot_g_mean_flux`             | `adql:DOUBLE`   | `e-/s`     | `—`                    | G-band mean flux Flux[e-/s]                                      |
| `phot_g_mean_flux_error`       | `adql:REAL`     | `e-/s`     | `—`                    | Error on G-band mean flux Flux[e-/s]                             |
| `phot_g_mean_mag`              | `adql:REAL`     | `mag`      | `—`                    | G-band mean magnitude Magnitude[mag]                             |
| `phot_rp_mean_flux`            | `adql:DOUBLE`   | `e-/s`     | `—`                    | Integrated RP mean flux Flux[e-/s]                               |
| `phot_rp_mean_flux_error`      | `adql:REAL`     | `e-/s`     | `—`                    | Error on the integrated RP mean flux Flux[e-/s]                  |
| `phot_rp_mean_mag`             | `adql:REAL`     | `mag`      | `—`                    | Integrated RP mean magnitude Magnitude[mag]                      |
| `pm`                           | `adql:REAL`     | `mas/year` | `—`                    | Total proper motion Angular Velocity[mas/year]                   |
| `pmdec`                        | `adql:DOUBLE`   | `mas/year` | `—`                    | Proper motion in declination direction Angular Velocity[mas/year |
| `pmdec_error`                  | `adql:REAL`     | `mas/year` | `—`                    | Standard error of proper motion in declination direction Angular |
| `pmra`                         | `adql:DOUBLE`   | `mas/year` | `—`                    | Proper motion in right ascension direction Angular Velocity[mas/ |
| `pmra_error`                   | `adql:REAL`     | `mas/year` | `—`                    | Standard error of proper motion in right ascension direction Ang |
| `ra`                           | `adql:DOUBLE`   | `deg`      | `pos.eq.ra;meta.main`  | Barycentric right ascension alpha of the source in ICRS at the r |
| `radial_velocity`              | `adql:REAL`     | `km/s`     | `—`                    | Radial velocity [km/s]                                           |
| `radial_velocity_error`        | `adql:REAL`     | `km/s`     | `—`                    | Radial velocity error [km/s]                                     |
| `ruwe`                         | `adql:REAL`     | `—`        | `—`                    | Renormalised unit weight error                                   |
| `classprob_dsc_combmod_galaxy` | `adql:REAL`     | `—`        | `—`                    | Probability from DSC-Combmod of being a galaxy (data used: BP/RP |
| `classprob_dsc_combmod_quasar` | `adql:REAL`     | `—`        | `—`                    | Probability from DSC-Combmod of being a quasar (data used: BP/RP |
| `classprob_dsc_combmod_star`   | `adql:REAL`     | `—`        | `—`                    | Probability from DSC-Combmod of being a single star (but not a w |
| `astrometric_params_solved`    | `adql:INTEGER`  | `byte`     | `—`                    | Which parameters have been solved for [byte]                     |
| `duplicated_source`            | `adql:SMALLINT` | `—`        | `—`                    | Source with multiple source identifiers                          |
| `non_single_star`              | `adql:SMALLINT` | `—`        | `—`                    | Flag indicating the availability of additional information in th |
| `visibility_periods_used`      | `adql:SMALLINT` | `—`        | `—`                    | Number of visibility periods used in Astrometric solution        |
| `phot_variable_flag`           | `adql:CHAR`     | `—`        | `—`                    | Photometric variability flag                                     |
| `ag_gspphot`                   | `adql:REAL`     | `mag`      | `—`                    | Extinction in G band from GSP-Phot Aeneas best library using BP/ |
| `ag_gspphot_lower`             | `adql:REAL`     | `mag`      | `—`                    | Lower confidence level (16%) of extinction in G band from GSP-Ph |
| `ag_gspphot_upper`             | `adql:REAL`     | `mag`      | `—`                    | Upper confidence level (84%) of extinction in G band from GSP-Ph |
| `astrometric_chi2_al`          | `adql:REAL`     | `—`        | `—`                    | AL chi-square value Astrometric goodness-of-fit (chi^2)          |
| `astrometric_gof_al`           | `adql:REAL`     | `—`        | `—`                    | Goodness of fit statistic of model wrt along-scan observations   |
| `astrometric_sigma5d_max`      | `adql:REAL`     | `—`        | `—`                    | Matched FOV transits used in the AGIS solution                   |
| `azero_gspphot`                | `adql:REAL`     | `mag`      | `—`                    | Monochromatic extinction A0 at 547.7nm from GSP-Phot Aeneas best |
| `azero_gspphot_lower`          | `adql:REAL`     | `mag`      | `—`                    | Lower confidence level (16%) of monochromatic extinction A0 at 5 |
| `azero_gspphot_upper`          | `adql:REAL`     | `mag`      | `—`                    | Upper confidence level (84%) of monochromatic extinction A0 at 5 |
| `b`                            | `adql:DOUBLE`   | `deg`      | `—`                    | Galactic latitude Angle[deg]                                     |
| `bp_g`                         | `adql:REAL`     | `mag`      | `—`                    | BP - G colour Magnitude[mag]                                     |
| `bp_rp`                        | `adql:REAL`     | `mag`      | `—`                    | BP - RP colour Magnitude[mag]                                    |
| `dec_error`                    | `adql:REAL`     | `mas`      | `—`                    | Standard error of declination Angle[mas]                         |
| `dec_parallax_corr`            | `adql:REAL`     | `—`        | `—`                    | Correlation between declination and parallax [-1,+1]             |
| `dec_pmdec_corr`               | `adql:REAL`     | `—`        | `—`                    | Correlation between declination and proper motion in declination |
| `dec_pmra_corr`                | `adql:REAL`     | `—`        | `—`                    | Correlation between declination and proper motion in right ascen |
| `dec_pseudocolour_corr`        | `adql:REAL`     | `—`        | `—`                    | Correlation between declination and pseudocolour [-1,+1]         |
| `distance_gspphot`             | `adql:REAL`     | `pc`       | `—`                    | Distance from GSP-Phot Aeneas best library using BP/RP spectra ( |
| `distance_gspphot_lower`       | `adql:REAL`     | `pc`       | `—`                    | Lower confidence level (16%) of distance from GSP-Phot Aeneas be |
| `distance_gspphot_upper`       | `adql:REAL`     | `pc`       | `—`                    | Upper confidence level (84%) of distance from GSP-Phot Aeneas be |
| `ebpminrp_gspphot`             | `adql:REAL`     | `mag`      | `—`                    | Reddening E(G_BP - G_RP) from GSP-Phot Aeneas best library using |
| `ebpminrp_gspphot_lower`       | `adql:REAL`     | `mag`      | `—`                    | Lower confidence level (16%) of reddening E(G_BP - G_RP) from GS |
| `ebpminrp_gspphot_upper`       | `adql:REAL`     | `mag`      | `—`                    | Upper confidence level (84%) of reddening E(G_BP - G_RP) from GS |
| `ecl_lat`                      | `adql:DOUBLE`   | `deg`      | `—`                    | Ecliptic latitude Angle[deg]                                     |
| `ecl_lon`                      | `adql:DOUBLE`   | `deg`      | `—`                    | Ecliptic longitude Angle[deg]                                    |
| `g_rp`                         | `adql:REAL`     | `mag`      | `—`                    | G - RP colour Magnitude[mag]                                     |
| `grvs_mag`                     | `adql:REAL`     | `mag`      | `—`                    | Integrated Grvs magnitude (Magnitude[mag])                       |
| `grvs_mag_error`               | `adql:REAL`     | `mag`      | `—`                    | Grvs magnitude uncertainty (Magnitude[mag])                      |
| `ipd_gof_harmonic_amplitude`   | `adql:REAL`     | `—`        | `—`                    | Amplitude of the IPD GoF versus position angle of scan           |
| `ipd_gof_harmonic_phase`       | `adql:REAL`     | `deg`      | `—`                    | Phase of the IPD GoF versus position angle of scan Angle[deg]    |
| `l`                            | `adql:DOUBLE`   | `deg`      | `—`                    | Galactic longitude Angle[deg]                                    |
| `logg_gspphot`                 | `adql:REAL`     | `—`        | `—`                    | Surface gravity from GSP-Phot Aeneas best library using BP/RP sp |
| `logg_gspphot_lower`           | `adql:REAL`     | `—`        | `—`                    | Lower confidence level (16%) of surface gravity from GSP-Phot Ae |
| `logg_gspphot_upper`           | `adql:REAL`     | `—`        | `—`                    | Upper confidence level (84%) of surface gravity from GSP-Phot Ae |
| `mh_gspphot`                   | `adql:REAL`     | `dex`      | `—`                    | Iron abundance from GSP-Phot Aeneas best library using BP/RP spe |
| `mh_gspphot_lower`             | `adql:REAL`     | `dex`      | `—`                    | Lower confidence level (16%) of iron abundance from GSP-Phot Aen |
| `mh_gspphot_upper`             | `adql:REAL`     | `dex`      | `—`                    | Upper confidence level (84%) of iron abundance from GSP-Phot Aen |
| `nu_eff_used_in_astrometry`    | `adql:REAL`     | `um^-1`    | `—`                    | Effective wavenumber of the source used in the astrometric solut |
| `parallax_pmdec_corr`          | `adql:REAL`     | `—`        | `—`                    | Correlation between parallax and proper motion in declination [- |
| `parallax_pmra_corr`           | `adql:REAL`     | `—`        | `—`                    | Correlation between parallax and proper motion in right ascensio |
| `parallax_pseudocolour_corr`   | `adql:REAL`     | `—`        | `—`                    | Correlation between parallax and pseudocolour [-1,+1]            |
| `phot_bp_mean_flux_over_error` | `adql:REAL`     | `—`        | `—`                    | Integrated BP mean flux divided by its error                     |
| `phot_g_mean_flux_over_error`  | `adql:REAL`     | `—`        | `—`                    | G-band mean flux divided by its error                            |
| `phot_rp_mean_flux_over_error` | `adql:REAL`     | `—`        | `—`                    | Integrated RP mean flux divided by its error                     |
| `pmdec_pseudocolour_corr`      | `adql:REAL`     | `—`        | `—`                    | Correlation between proper motion in declination and pseudocolou |
| `pmra_pmdec_corr`              | `adql:REAL`     | `—`        | `—`                    | Correlation between proper motion in right ascension and proper  |
| `pmra_pseudocolour_corr`       | `adql:REAL`     | `—`        | `—`                    | Correlation between proper motion in right asension and pseudoco |
| `pseudocolour`                 | `adql:REAL`     | `um^-1`    | `—`                    | Astrometrically estimated pseudocolour of the source Misc[um^-1] |
| `pseudocolour_error`           | `adql:REAL`     | `um^-1`    | `—`                    | Standard error of the pseudocolour of the source Misc[um^-1]     |
| `ra_dec_corr`                  | `adql:REAL`     | `—`        | `—`                    | Correlation between right ascension and declination [-1,+1]      |
| `ra_error`                     | `adql:REAL`     | `mas`      | `—`                    | Standard error of the right ascension of the source in ICRS at t |
| `ra_parallax_corr`             | `adql:REAL`     | `—`        | `—`                    | Correlation between right ascension and parallax [-1,+1]         |
| `ra_pmdec_corr`                | `adql:REAL`     | `—`        | `—`                    | Correlation between right ascension and proper motion in declina |
| `ra_pmra_corr`                 | `adql:REAL`     | `—`        | `—`                    | Correlation between right ascension and proper motion in right a |
| `ra_pseudocolour_corr`         | `adql:REAL`     | `—`        | `—`                    | Correlation between right ascension and pseudocolour [-1,+1]     |
| `rv_chisq_pvalue`              | `adql:REAL`     | `—`        | `—`                    | P-value for constancy based on a chi-squared criterion           |
| `rv_expected_sig_to_noise`     | `adql:REAL`     | `—`        | `—`                    | Expected signal to noise ratio in the combination of the spectra |
| `rv_renormalised_gof`          | `adql:REAL`     | `—`        | `—`                    | Radial velocity renormalised goodness of fit                     |
| `rvs_spec_sig_to_noise`        | `adql:REAL`     | `—`        | `—`                    | Signal to noise ratio in the mean RVS spectrum                   |
| `scan_direction_mean_k1`       | `adql:REAL`     | `—`        | `—`                    | Mean position angle of scan directions across the source         |
| `scan_direction_mean_k2`       | `adql:REAL`     | `—`        | `—`                    | Mean position angle of scan directions across the source         |
| `scan_direction_mean_k3`       | `adql:REAL`     | `—`        | `—`                    | Mean position angle of scan directions across the source         |
| `scan_direction_mean_k4`       | `adql:REAL`     | `—`        | `—`                    | Mean position angle of scan directions across the source         |
| `scan_direction_strength_k1`   | `adql:REAL`     | `—`        | `—`                    | Degree of concentration of scan directions across the source     |
| `scan_direction_strength_k2`   | `adql:REAL`     | `—`        | `—`                    | Degree of concentration of scan directions across the source     |
| `scan_direction_strength_k3`   | `adql:REAL`     | `—`        | `—`                    | Degree of concentration of scan directions across the source     |
| `scan_direction_strength_k4`   | `adql:REAL`     | `—`        | `—`                    | Degree of concentration of scan directions across the source     |

### Images

- **Pixel unit:** `unknown` | **AB zero point:** `—` | **Calibration note:** Resolved to a final `unknown`. `CDS/P/DM/...` is a CDS *density map*: a rendering built by binning the Gaia catalogue onto HEALPix, not imaging, so there are no survey pixels to calibrate even in principle. hips2fits returns a 4-plane uint8 RGBA image in [0, 255] with no BUNIT (checked 2026-09-04), confirming it. The adapter stays disabled; it would also fail on shape, one column against four planes.
- **Cutout service `None`:** adapter `hips`, bands `['color']`, params `{'hips_id': 'CDS/P/DM/flux-color-Rp-G-Bp/I/355/gaiadr3'}`.
- **Cutout service `None`:** adapter `hips`, bands `['g']`, params `{'hips_id': 'CDS/P/DM/flux-G/I/355/gaiadr3'}`.
- **Cutout service `None`:** adapter `hips`, bands `['bp']`, params `{'hips_id': 'CDS/P/DM/flux-Bp/I/355/gaiadr3'}`.
- **Cutout service `None`:** adapter `hips`, bands `['rp']`, params `{'hips_id': 'CDS/P/DM/flux-Rp/I/355/gaiadr3'}`.

| HiPS ID                                     | Regime   | Frame        | Order   | Pixel scale (deg)   | Tile format   | Mirrors                                             |
|---------------------------------------------|----------|--------------|---------|---------------------|---------------|-----------------------------------------------------|
| `CDS/P/DM/flux-G/I/355/gaiadr3`             | Optical  | `equatorial` | `7`     | `8.946E-4`          | `jpeg fits`   | `alasky.cds.unistra.fr`, `alaskybis.cds.unistra.fr` |
| `CDS/P/DM/flux-color-Rp-G-Bp/I/355/gaiadr3` | Optical  | `equatorial` | `7`     | `—`                 | `jpeg`        | `alasky.cds.unistra.fr`, `alaskybis.cds.unistra.fr` |
| `CDS/P/DM/flux-Bp/I/355/gaiadr3`            | Optical  | `equatorial` | `7`     | `8.946E-4`          | `jpeg fits`   | `alasky.cds.unistra.fr`, `alaskybis.cds.unistra.fr` |
| `CDS/P/DM/flux-Rp/I/355/gaiadr3`            | Optical  | `equatorial` | `7`     | `8.946E-4`          | `jpeg fits`   | `alasky.cds.unistra.fr`, `alaskybis.cds.unistra.fr` |

### Spectra

No 1D spectroscopic backends are configured for this survey.

## References and acknowledgement

- Gaia Collaboration: Prusti, T., et al. (2016), *A&A*, 595, A1 (`doi:10.1051/0004-6361/201629272`) — The Gaia mission.
- Gaia Collaboration: Vallenari, A., et al. (2023), *A&A*, 674, A1 (`doi:10.1051/0004-6361/202243940`) — Gaia Data Release 3: Summary of the content and survey properties.
- Riello, M., et al. (2021), *A&A*, 649, A3 (`doi:10.1051/0004-6361/202039587`) — Gaia Early Data Release 3: Photometric content and validation.
- NOIRLab Astro Data Lab TAP service: [https://datalab.noirlab.edu/tap](https://datalab.noirlab.edu/tap) (table `gaia_dr3.gaia_source`).
- CDS HiPS Density Maps: [https://alasky.cds.unistra.fr/ancillary/GaiaDR3/](https://alasky.cds.unistra.fr/ancillary/GaiaDR3/).

This work has made use of data from the European Space Agency (ESA) mission Gaia ([https://www.cosmos.esa.int/gaia](https://www.cosmos.esa.int/gaia)), processed by the Gaia Data Processing and Analysis Consortium (DPAC, [https://www.cosmos.esa.int/web/gaia/dpac/consortium](https://www.cosmos.esa.int/web/gaia/dpac/consortium)). Funding for the DPAC has been provided by national institutions, in particular the institutions participating in the Gaia Multilateral Agreement. When querying via NOIRLab Astro Data Lab, cite the Astro Data Lab acknowledgement policy ([https://datalab.noirlab.edu/acknowledgements.php](https://datalab.noirlab.edu/acknowledgements.php)).

## Provenance of this page

| Source ID          | Kind        | Service / Endpoint                              | Query or URL                                                                                                             | Retrieved (UTC)        |
|--------------------|-------------|-------------------------------------------------|--------------------------------------------------------------------------------------------------------------------------|------------------------|
| `catalog_gaia_dr3` | `tap_table` | `https://datalab.noirlab.edu/tap`               | `SELECT column_name, datatype, unit, ucd, description FROM TAP_SCHEMA.columns WHERE table_name = 'gaia_dr3.gaia_source'` | `2026-10-18T16:30:31Z` |
| `hips_gaia_dr3`    | `hips`      | `https://alasky.cds.unistra.fr/MocServer/query` | `https://alasky.cds.unistra.fr/MocServer/query?ID=CDS/P/DM/flux-G/I/355/gaiadr3&fmt=json&get=record`                     | `2026-10-18T16:30:40Z` |
<!-- /generated -->
