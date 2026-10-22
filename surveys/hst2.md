# Hubble Source Catalog v2 (`hst2`)

<!-- generated:survey-page -->

The Hubble Source Catalog Version 2 (`hst2`) provides multi-instrument point-source and extended-source photometry constructed from deep imaging observations obtained with the Hubble Space Telescope (HST). It integrates detections from WFPC2, the Advanced Camera for Surveys (ACS/WFC), and the Wide Field Camera 3 (WFC3 UVIS and IR channels) across sixteen ultraviolet, optical, and near-infrared filter bands (`f275w`, `f300w`, `f336w`, `f435w`, `f475w`, `f555w`, `f606w`, `f625w`, `f775w`, `f814w`, `f850lp`, `f105w`, `f110w`, `f125w`, `f140w`, `f160w`).

In MOSAICS, `hst2` serves as a high-resolution space-based photometric crossmatch catalogue queried from the detailed HATS table at `https://vizcat.cds.unistra.fr/hats/hsc2_detailed/` (VizieR `II/358` / `II/342`). It also defines an image cutout entry pointing to the Hubble Legacy Archive (HLA) Simple Image Access (SIA) service (`http://hla.stsci.edu/cgi-bin/hlaSIAP.cgi`). Cutouts are disabled by default because HLA pointings represent heterogeneous, targeted observations rather than a uniform survey grid.

| Property                 | Value                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                   |
|--------------------------|-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| What MOSAICS extracts    | Catalogue: `hst2` (photometric, `lsdb`); Cutouts: `sia` (color)                                                                                                                                                                                                                                                                                                                                                                                                                                                         |
| Archives                 | HATS (`vizcat.cds.unistra.fr`), SIA (`hla.stsci.edu`)                                                                                                                                                                                                                                                                                                                                                                                                                                                                   |
| Bands / wavelength range | `f275w` (`HST/WFC3_UVIS2.F275W`), `f300w` (`HST/WFPC2-PC.F300W`), `f336w` (`HST/WFC3_UVIS2.F336W`), `f435w` (`HST/ACS_WFC.F435W`), `f475w` (`HST/ACS_WFC.F475W`), `f555w` (`HST/ACS_WFC.F555W`), `f606w` (`HST/ACS_WFC.F606W`), `f625w` (`HST/ACS_WFC.F625W`), `f775w` (`HST/ACS_WFC.F775W`), `f814w` (`HST/ACS_WFC.F814W`), `f850lp` (`HST/ACS_WFC.F850LP`), `f105w` (`HST/WFC3_IR.F105W`), `f110w` (`HST/WFC3_IR.F110W`), `f125w` (`HST/WFC3_IR.F125W`), `f140w` (`HST/WFC3_IR.F140W`), `f160w` (`HST/WFC3_IR.F160W`) |
| Used by                  | `mosaics-targets` (crossmatch), `mosaics-cutout full`, `mosaics-crossmatch`                                                                                                                                                                                                                                                                                                                                                                                                                                             |
| Crossmatch radius        | `0.5` arcsec. WFC3 PSF FWHM 0.067-0.089 arcsec (UVIS) and 0.124-0.156 arcsec (IR) (WFC3 Instrument Handbook, Tables 6.7 and 7.5); absolute astrometric accuracy of the Version 1 Hubble Source Catalog better than about 0.1 arcsec for most sources (Whitmore et al. 2016, arXiv:1602.04861). No statement specific to Version 2 was found.                                                                                                                                                                            |

## Configuration

### The shipped file

Source: `defs/surveys/hst2.yaml`

```yaml
# Hubble Source Catalog (HSC v2, detailed) -- photometry, plus Hubble Legacy
# Archive imaging -- cutouts.
#
# `hsc` in mosaics/crossmatch/reshapers/hsc.py means *this*, the Hubble Source
# Catalog, not Subaru HSC-SSP, which is `hsc3`. The names sit one letter apart
# and the two are unrelated.
#
# The cutouts section was its own `hla` survey until the two were merged: the
# HLA imaging and the Source Catalog are the same telescope's products and the
# catalogue is built from that imaging, so one file describes both and the
# cutout columns are `cutout_hst2_*`. `hasp` (HST spectral coadds) stays
# separate -- it is a different product, not a different section of this one.

survey_slug: hst2
name: Hubble Source Catalog v2

catalog:
  - name: hst2
    type: photometric
    adapter: lsdb
    adapter_params:
      path: https://vizcat.cds.unistra.fr/hats/hsc2_detailed/
      # The detailed catalogue publishes one row per measurement rather than
      # one per object, so it is pivoted per partition before the crossmatch.
      # The reshaper's output schema and the coordinate columns it leaves
      # behind are taken from the pivot itself rather than declared here.
      reshape: pivot_hsc

# Crossmatch: the radius a table of positions is matched within, and what it rests on.
crossmatch:
  radius: 0.5 arcsec
  radius_note: >-
    WFC3 PSF FWHM 0.067-0.089 arcsec (UVIS) and 0.124-0.156 arcsec (IR) (WFC3 Instrument
    Handbook, Tables 6.7 and 7.5); absolute astrometric accuracy of the Version 1 Hubble
    Source Catalog better than about 0.1 arcsec for most sources (Whitmore et al. 2016,
    arXiv:1602.04861). No statement specific to Version 2 was found.

cutout:
  - adapter: sia
    # No band key: the HLA SIA service serves whatever filter each pointing
    # was taken in, which is not a survey-wide band, so there is no `f606w`
    # key that would mean the same thing everywhere on the sky.
    adapter_params:
      service_url: http://hla.stsci.edu/cgi-bin/hlaSIAP.cgi
    bands: [color]
    enabled: false        # heterogeneous pointings, not a survey grid

photometric_system:
  filters:
    # Three cameras. The Hubble Source Catalog mixes them, so the instrument
    # is part of the answer rather than a survey-wide constant.
    f275w: HST/WFC3_UVIS2.F275W
    f300w: HST/WFPC2-PC.F300W
    f336w: HST/WFC3_UVIS2.F336W
    f435w: HST/ACS_WFC.F435W
    f475w: HST/ACS_WFC.F475W
    f555w: HST/ACS_WFC.F555W
    f606w: HST/ACS_WFC.F606W
    f625w: HST/ACS_WFC.F625W
    f775w: HST/ACS_WFC.F775W
    f814w: HST/ACS_WFC.F814W
    f850lp: HST/ACS_WFC.F850LP
    f105w: HST/WFC3_IR.F105W
    f110w: HST/WFC3_IR.F110W
    f125w: HST/WFC3_IR.F125W
    f140w: HST/WFC3_IR.F140W
    f160w: HST/WFC3_IR.F160W
  measurements:
    # ABMAG, and the one survey with no uncertainty at all -- the detailed
    # catalogue publishes none, so fnu_err_hst2_* is a typed null rather
    # than a guess.
    - system: ab_mag
      apertures: [aper2, auto]
      errors: false
  pixel_unit: unknown
  pixel_unit_note: >-
    HLA products differ by instrument and by PHOTFLAM, which is per image
    rather than per survey. Heterogeneous pointings rather than a survey
    grid, so there is nothing survey-wide to state.

# Hubble Source Catalog v2, detailed (per-filter, per-detection) table
# (VizieR II/358): HST multi-instrument aperture and total photometry.
# Catalog: https://vizcat.cds.unistra.fr/hats/hst2_detailed/
# One row per detection per filter -- downstream groups on MatchID and
# spreads on Filter, so those two plus the pivot-critical measurement
# columns are kept regardless of the usual selection criteria.
# This catalog publishes no per-detection magnitude uncertainty column
# (no e_magAper2 / e_magAuto): only CI and the extraction Flags convey
# per-detection photometric quality.
# ra/dec = RAJ2000/DEJ2000. Selected via LSDB, not turned into SQL.
#
# THIS FILE SELECTS; IT DOES NOT NAME. `pivot_hsc` runs before the rename and
# emits the final column names itself (mosaics/crossmatch/reshapers/hsc.py), so the
# keys below are what is read from the catalog while the aliases are only a
# note on what each one means. What survives the pivot is: RAJ2000, DEJ2000,
# CI and RadKron as hst2_ra / hst2_dec / hst2_concentration_index /
# hst2_kron_radius; magAper2, magAuto and FluxAper2 spread per filter as
# hst2_mag_aper2_f606w, hst2_mag_auto_f606w, hst2_flux_aper2_f606w, plus
# hst2_ndet_<band>, hst2_mag_aper2_std_<band> and hst2_n_filters. Everything
# else here -- including hst2_wavelength, hst2_filter and hst2_instrument --
# is consumed by the collapse and does not reach the catalogue. Filter pivot
# wavelengths come from filters.lance, keyed hst2_<band>, not from
# hst2_wavelength.
column_mapping:
  # CatID: 'cat_id'                        # unique identifier for this catalog row (one detection in one filter)
  # MatchID: 'match_id'                    # identifier shared by all detections of the same physical object
  RAJ2000: 'ra'                            # match-position right ascension, J2000, deg
  DEJ2000: 'dec'                           # match-position declination, J2000, deg
  # Source: 'source_id'                    # source identifier from the detection imagery
  Det: 'flag_detected'                     # whether the source was detected in this observation, Y/N
  # RASdeg: 'source_ra'                    # per-detection right ascension, J2000, deg
  # DESdeg: 'source_dec'                   # per-detection declination, J2000, deg
  # Dist: 'dist'                           # distance from the detection position to the match position, mas
  # e_Dist: 'dist_err'                     # uncertainty on Dist, mas
  AbsCorr: 'flag_abs_astrometry_corrected'      # whether an absolute astrometric correction was applied
  Inst: 'instrument'                       # HST instrument that made the detection
  Mode: 'detector_mode'                    # instrument/detector configuration
  Aperture: 'aperture_radius'              # aperture radius used for FluxAper2/magAper2, arcsec
  # ExpTime: 'exptime'                       # exposure time, s
  # lambda: 'wavelength'                     # pivot wavelength of the filter, Angstrom
  Filter: 'filter'                         # filter name, e.g. F625W
  FluxAper2: 'flux_aper2'                  # count rate within the Aperture-radius aperture
  magAper2: 'mag_aper2'                    # aperture magnitude within the Aperture-radius aperture
  magAuto: 'mag_auto'                      # estimated total magnitude, SExtractor MAG_AUTO algorithm
  CI: 'concentration_index'                # concentration index, normalised by detector and filter
  RadKron: 'kron_radius'                   # mean Kron radius from SExtractor
  Flags: 'flags'                           # bit-encoded source classification (point/extended, saturation, ...)
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

**Catalogue `hst2` (`adapter: lsdb`):**

| Parameter           | Allowed Type   | Default               | Description                                                                                         |
|---------------------|----------------|-----------------------|-----------------------------------------------------------------------------------------------------|
| `path`              | str            | —                     | HTTP or S3 URI of the HATS catalogue root (may contain `{band}` or `{BAND}` when `per_band: true`). |
| `margin_cache`      | str            | None                  | URI of the HATS pre-computed margin cache table for boundary-safe spatial crossmatching.            |
| `storage_options`   | mapping        | {}                    | Filesystem options forwarded to `upath`/`fsspec` (e.g. `{'anon': True}` for public S3 buckets).     |
| `reshape`           | str            | None                  | Optional post-read reshaping strategy for nested or multi-row HATS tables.                          |
| `per_band`          | bool           | false                 | When `true`, opens one HATS catalogue per band by substituting `{band}`/`{BAND}` in `path`.         |
| `remote_ra_column`  | str            | HATS metadata default | Override for the remote right ascension column name.                                                |
| `remote_dec_column` | str            | HATS metadata default | Override for the remote declination column name.                                                    |
- **Cutout `None` (`adapter: sia`, bands `['color']`):** configured `adapter_params`: `service_url='http://hla.stsci.edu/cgi-bin/hlaSIAP.cgi'`.

### Common changes

- Enable HLA cutout extraction in an entrypoint configuration by setting `overrides[surveys/hst2].cutout[0].enabled: true` (bearing in mind that coverage exists only where HST executed targeted pointings).
- Disable `hst2` catalogue crossmatch by setting `overrides[surveys/hst2].catalog: []` or setting `enabled: false`.
- Filter crossmatched sources on concentration index (`hst2_concentration_index`) or SExtractor flags (`hst2_flags`) to select clean stellar point sources or unflagged galaxy profiles.

### Commands

```console
$ uv run -m mosaics.cli.cutout full -t test/objects_sample_1k.csv -o cutouts.lance --config defs/cutout.yaml --survey hst2 --plan
$ uv run -m mosaics.crossmatch.cli run --survey surveys/hst2 -i test/objects_sample_1k.csv -o hst2_matched.parquet
$ uv run -m mosaics.cli.targets --config defs/targets.yaml -o ./targets_run --dec-range 0 1 --plan plan.yaml
```

## Data served by the archive

- **Distinction from Subaru HSC (`hsc3`):** In MOSAICS configurations, `hst2` designates the Hubble Source Catalog, whereas `hsc3` designates the Subaru Hyper Suprime-Cam Strategic Program. The two surveys share similar acronyms but are entirely separate astronomical projects.
- **Detailed measurement table and partition pivoting:** The detailed HSC v2 catalogue publishes one row per measurement per filter rather than one row per physical object. During extraction, MOSAICS pivots each partition on `MatchID`, spreading measurements across per-filter columns (`hst2_mag_aper2_<band>`, `hst2_mag_auto_<band>`, `hst2_flux_aper2_<band>`) and tracking detection counts (`hst2_ndet_<band>`) and standard deviations (`hst2_mag_aper2_std_<band>`).
- **Complete absence of magnitude uncertainties:** The upstream detailed HSC v2 catalogue publishes no magnitude uncertainty columns (no `e_magAper2` or `e_magAuto`). Flux errors (`fnu_err_hst2_*`) are consequently emitted as typed nulls rather than guessed values. Photometric reliability must be assessed via the concentration index (`CI`), SExtractor `Flags`, and repeat detection counts (`ndet`).
- **Heterogeneous cutout calibration:** HLA image cutouts vary across instruments and observation epochs, each with individual `PHOTFLAM` header keywords rather than a survey-wide zero point. Cutout pixel units are therefore marked as unknown.

### Catalogue: `II/342/hsc2`

Hubble Source Catalog V2 ( 383,834,027 sources)

**Service URL:** `https://vizcat.cds.unistra.fr/hats/hsc2_detailed/`

| Column      | Type     | Unit   | UCD                      | Description                                                                                                         |
|-------------|----------|--------|--------------------------|---------------------------------------------------------------------------------------------------------------------|
| `CatID`     | `int32`  | `—`    | `—`                      | —                                                                                                                   |
| `RAJ2000`   | `double` | `deg`  | `pos.eq.ra;meta.main`    | Right Ascension for the match (J2000) (MatchRA)                                                                     |
| `DEJ2000`   | `double` | `deg`  | `pos.eq.dec;meta.main`   | Declination for the match (J2000) (MatchDec)                                                                        |
| `MatchID`   | `int32`  | `—`    | `meta.id`                | Identifier for the match (MatchID)                                                                                  |
| `MemID`     | `int16`  | `—`    | `—`                      | —                                                                                                                   |
| `Source`    | `int64`  | `—`    | `meta.id`                | Source identifier from white light (detection) image (SourceID)                                                     |
| `ImageID`   | `int32`  | `—`    | `—`                      | —                                                                                                                   |
| `Det`       | `string` | `—`    | `meta.code`              | [Y/N] Is source detected in this observation ? (Det)                                                                |
| `RASdeg`    | `double` | `—`    | `—`                      | —                                                                                                                   |
| `DESdeg`    | `double` | `—`    | `—`                      | —                                                                                                                   |
| `Dist`      | `double` | `mas`  | `pos.angDistance`        | Distance from source position to the match position (D)                                                             |
| `e_Dist`    | `double` | `—`    | `—`                      | —                                                                                                                   |
| `AbsCorr`   | `string` | `—`    | `—`                      | —                                                                                                                   |
| `XImage`    | `double` | `—`    | `—`                      | —                                                                                                                   |
| `YImage`    | `double` | `—`    | `—`                      | —                                                                                                                   |
| `ImageName` | `string` | `—`    | `—`                      | —                                                                                                                   |
| `Inst`      | `string` | `—`    | `—`                      | —                                                                                                                   |
| `Mode`      | `string` | `—`    | `—`                      | —                                                                                                                   |
| `DetID`     | `string` | `—`    | `—`                      | —                                                                                                                   |
| `Aperture`  | `string` | `—`    | `—`                      | —                                                                                                                   |
| `ExpTime`   | `double` | `—`    | `—`                      | —                                                                                                                   |
| `TimeS`     | `string` | `—`    | `—`                      | —                                                                                                                   |
| `TimeE`     | `string` | `—`    | `—`                      | —                                                                                                                   |
| `MJDS`      | `double` | `d`    | `time.start;obs`         | Observation start time in MJD (StartMJD)                                                                            |
| `MJDE`      | `double` | `d`    | `time.end;obs`           | Observation stop time in MJD (StopMJD)                                                                              |
| `lambda`    | `double` | `—`    | `—`                      | —                                                                                                                   |
| `Filter`    | `string` | `—`    | `meta.id;instr.filter`   | Filter (Filter) (1)                                                                                                 |
| `TargetID`  | `string` | `—`    | `—`                      | —                                                                                                                   |
| `FluxAper2` | `float`  | `ct/s` | `phot.flux`              | ? Count rate in Filter with an aperture radius 0.3” (WFPC2), 0.15” (ACS, WFC3/UVIS) and 0.45” (WFC3/IR) (FluxAper2) |
| `magAper2`  | `float`  | `mag`  | `phot.mag`               | ? Magnitude (AB) in Filter an aperture radius 0.3” (WFPC2), 0.15” (ACS, WFC3/UVIS) and 0.45” (WFC3/IR) (MagAper2)   |
| `magAuto`   | `float`  | `mag`  | `phot.mag`               | ? Estimated total magnitude in Filter using Source Extractor algorithm (MagAuto)                                    |
| `PropID`    | `int16`  | `—`    | `—`                      | —                                                                                                                   |
| `CI`        | `float`  | `—`    | `—`                      | —                                                                                                                   |
| `RadKron`   | `float`  | `pix`  | `phys.angSize;stat.mean` | ? Mean value of the Kron Radius from Source Extractor (KronRadius)                                                  |
| `Flags`     | `int8`   | `—`    | `—`                      | —                                                                                                                   |

### Images

- **Pixel unit:** `unknown` | **AB zero point:** `—` | **Calibration note:** HLA products differ by instrument and by PHOTFLAM, which is per image rather than per survey. Heterogeneous pointings rather than a survey grid, so there is nothing survey-wide to state.
- **Cutout service `None`:** adapter `sia`, bands `['color']`, params `{'service_url': 'http://hla.stsci.edu/cgi-bin/hlaSIAP.cgi'}`.

### Spectra

No 1D spectroscopic backends are configured for this survey.

## References and acknowledgement

- Whitmore, B. C. et al. (2016), *Version 1 of the Hubble Source Catalog*, AJ, 151, 134 (`doi:10.3847/0004-6256/151/6/134`).
- Hubble Source Catalog v2 VizieR catalogue `II/358` / `II/342`: [https://cdsarc.cds.unistra.fr/viz-bin/cat/II/358](https://cdsarc.cds.unistra.fr/viz-bin/cat/II/358)
- Hubble Legacy Archive (HLA) SIA endpoint: [http://hla.stsci.edu/cgi-bin/hlaSIAP.cgi](http://hla.stsci.edu/cgi-bin/hlaSIAP.cgi)
- HATS detailed table endpoint: [https://vizcat.cds.unistra.fr/hats/hsc2_detailed/](https://vizcat.cds.unistra.fr/hats/hsc2_detailed/)

This research has made use of the Hubble Source Catalog, constructed and maintained by the Space Telescope Science Institute (STScI) in collaboration with the Johns Hopkins University, based on data from the Hubble Legacy Archive (HLA). The Hubble Legacy Archive is a collaboration between the Space Telescope Science Institute (STScI/NASA), the Space Telescope European Coordinating Facility (ST-ECF/ESA) and the Canadian Astronomy Data Centre (CADC/NRC/CSA).

## Provenance of this page

| Source ID        | Kind           | Service / Endpoint                                  | Query or URL                                                       | Retrieved (UTC)        |
|------------------|----------------|-----------------------------------------------------|--------------------------------------------------------------------|------------------------|
| `cutout_hla_sia` | `sia`          | `http://hla.stsci.edu/cgi-bin/hlaSIAP.cgi`          | `http://hla.stsci.edu/cgi-bin/hlaSIAP.cgi`                         | `2026-10-18T16:33:01Z` |
| `hats_hsc2`      | `hats_catalog` | `https://vizcat.cds.unistra.fr/hats/hsc2_detailed/` | `https://vizcat.cds.unistra.fr/hats/hsc2_detailed/main/properties` | `2026-10-18T16:58:42Z` |
<!-- /generated -->
