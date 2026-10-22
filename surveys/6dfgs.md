# 6dF Galaxy Survey DR3 (`6dfgs`)

<!-- generated:survey-page -->

The 6dF Galaxy Survey Data Release 3 (6dFGS DR3) is the final release of a wide-area spectroscopic redshift survey of the southern extragalactic sky ($|b| > 10^\circ$, $\delta < 0^\circ$). Carried out with the Six-Degree Field (6dF) multi-object spectrograph on the 1.2-metre UK Schmidt Telescope (UKST) at Siding Spring Observatory, the survey covered approximately 17,000 deg² of the southern hemisphere. Primary targets were selected from the 2MASS Extended Source Catalog to a magnitude limit of $K \le 12.65$, supplemented by secondary samples selected in $b_J$ and $r_F$ from SuperCOSMOS, yielding over 125,000 galaxy spectra and 136,000 redshift measurements. Spectra cover optical wavelengths from 4000 to 7500 Å in the observer frame with a spectral resolution of $R \sim 1000$ using 6.7 arcsec diameter fibres.

In `mosaics`, 6dFGS serves as a bright, low-redshift anchor alongside 2dFGRS, mapping the nearby galaxy population that anchors morphological classification benchmarks. It supports target selection (`mosaics-targets` using `defs/surveys/spec_6dfgs_dr3.sql`), catalogue crossmatching (`mosaics-query` via the CDS VizieR TAP service table `"VII/259/6dfgs"`), and 1D FITS spectrum ingestion (`mosaics-spectrum 6dfgs` or `full`).

| Property                 | Value                                                                                                                                                                                                                                                                                                                 |
|--------------------------|-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| What MOSAICS extracts    | Catalogue: `6dfgs` (spectroscopic, `tap`); Spectra: `url` (flux,wavelength,mask)                                                                                                                                                                                                                                      |
| Archives                 | vizier (`https://tapvizier.cds.unistra.fr/TAPVizieR/tap`), `www-wfau.roe.ac.uk`                                                                                                                                                                                                                                       |
| Bands / wavelength range | spectroscopy (disperser `grating`, fibre `per_row` arcsec, calibration `none`)                                                                                                                                                                                                                                        |
| Used by                  | `mosaics-targets` (spectroscopic selection), `mosaics-spectrum 6dfgs` / `full`, `mosaics-crossmatch`, `mosaics-query`                                                                                                                                                                                                 |
| Crossmatch radius        | `5` arcsec. Fibre aperture on the sky: 6.7 arcsec (Jones et al. 2009, arXiv:0903.5451, Table 1), taken to the nearest value of the grid 0.5, 1, 1.5, 2, 3, 5 arcsec. The 6dFGS paper gives no precision for its 2MASS XSC input positions (the XSC against VLA FIRST has an RMS of 0.85-0.87 arcsec, an upper bound). |

## Configuration

### The shipped file

Source: `defs/surveys/6dfgs.yaml`

```yaml
# 6dFGS DR3 -- spectroscopy. No per-sample errors, so no ivar array.

survey_slug: 6dfgs
name: 6dF Galaxy Survey DR3

catalog:
  - name: 6dfgs
    type: spectroscopic
    adapter: tap
    adapter_params:
      service: vizier
      table: '"VII/259/6dfgs"'
      dialect: top

# Crossmatch: the radius a table of positions is matched within, and what it rests on.
crossmatch:
  radius: 5 arcsec
  radius_note: >-
    Fibre aperture on the sky: 6.7 arcsec (Jones et al. 2009, arXiv:0903.5451, Table 1),
    taken to the nearest value of the grid 0.5, 1, 1.5, 2, 3, 5 arcsec. The 6dFGS paper
    gives no precision for its 2MASS XSC input positions (the XSC against VLA FIRST has an
    RMS of 0.85-0.87 arcsec, an upper bound).

catalog_filters:
  # See sdss17.yaml for the general rule this and the other five redshift
  # surveys share. Quoted: an identifier starting with a digit needs it.
  # `6dfgs_z` is derived from `6dfgs_cz` at extraction time -- see
  # `normalise_redshift` in mosaics/pipelines/targets/spec.py -- rather than
  # published directly, but it is a real column by the time production runs.
  6dfgs: '"6dfgs_z" BETWEEN 0.0 AND 2.5'
  6dfgs_note: >-
    Restricts the catalogue to the redshift range the training targets are
    drawn over; a row outside it keeps its candidacy but loses its 6dFGS
    spectroscopic columns, as if this survey had not matched it.

target_selection:
  catalog_name: 6dfgs
  query: spec_6dfgs_dr3.sql
  ra_column: RAJ2000
  dec_column: DEJ2000

spectra:
  - adapter: url
    adapter_params:
      # Every url survey needs its OWN url_column: they all default to
      # `access_url`, and two sharing it would fetch the same URL twice and
      # file the result under two different surveys.
      url_column: sixdfgs_access_url
    format: generic
    # No `error`: this survey publishes no per-sample uncertainty at
    # all, so the mask below tests finiteness only -- the
    # signal-to-noise term is skipped rather than evaluated against a
    # column that is not there.
    arrays: [flux, wavelength, mask]

spectroscopic_system:
  fiber_diameter: per_row     # 6.7 arcsec, already selected by the query
  # Ruled 600V/316R before October 2002, VPH 580V/425R after (Jones et al.
  # 2004, MNRAS 355, 747). Both are bare gratings, so the geometry axis gives
  # one honest constant where a technology axis would have forced `mixed` --
  # which matters, because DR3 publishes no epoch column to split them on.
  disperser: grating
  redshift_rank: 5
  flux_scale: null            # uncalibrated counts
  flux_calibration: none
```

Target-selection query (`defs/surveys/spec_6dfgs_dr3.sql`):

```sql
-- 6dF Galaxy Survey DR3: the final release, southern sky at |b| > 10 deg.
--
-- Provider: tap --service vizier (CDS VizieR, ADQL).
--   uv run -m mosaics.cli.query sql --provider tap --service vizier \
--     --path sql/6dfgs_dr3_spectra.sql --ra-col RAJ2000 --dec-col DEJ2000 \
--     -o 6dfgs.parquet
--
-- Retrieval contract for `mosaics.cli.spectrum 6dfgs`: this catalogue has no
-- URL column, so the addresses come from the survey's SSA service and are
-- joined to this table by position:
--
--   uv run -m mosaics.cli.query ssa -u 6dfgs --ra <r> --dec <d> --size <s> \
--     -o 6dfgs_ssa.parquet
--   uv run -m mosaics.cli.spectrum 6dfgs --targets 6dfgs_ssa.parquet \
--     --url-column Reference -o 6dfgs.lance
--
-- 6dFGS spectra are not reliably flux calibrated and must be rescaled to
-- broad-band photometry before being mixed with DESI or SDSS, exactly as for
-- LAMOST. bJmag and rFmag are selected so that rescaling has something to
-- work with.
--
-- Redshift quality. The survey documentation is explicit that only Q = 3 and
-- Q = 4 should be used in galaxy analyses: Q >= 3 redshifts are 98.4%
-- reliable with an rms of 85 km/s. Q = 6 marks a confirmed Galactic source
-- and there is no Q = 5.
--
-- The documented trap is that some legitimate quasar redshifts carry Q = 2,
-- because no quasar templates were used in the first half of the survey.
-- Those rows are kept rather than discarded, which is why the cut below is
-- q_cz >= 2 rather than >= 3. Split them downstream on the q_cz column that
-- is already selected:
--
--   main = frame[frame['q_cz'] >= 3]
--   edge = frame[frame['q_cz'] == 2]     # quasars the templates missed
--
-- rather than in SQL: VizieR speaks ADQL 2.0, which has no CASE expression,
-- so a derived edge_case flag is a parse error here. VizieR table names
-- contain slashes and must be quoted.
--
--  Quality flag (q_cz) values:
-- 0 = 	no redshift quality value available
-- 1 = 	6dFGS or 2dFGRS, unusable measurement.
-- 2 = 	6dFGS or 2dFGRS, possible, but doubtful redshift estimate
-- 3 = 	6dFGS or 2dFGRS, 'probable' redshift (notionally 90% confidence)
-- 4 = 	6dFGS or 2dFGRS, reliable redshift (notionally 99% confidence)
-- 6 = 	6dFGS confirmed Galactic source
-- 9 = 	SDSS and ZCAT sources (not a measure of quality) 
--
-- Cross-survey flag (f_cz) values:
-- 0 = 	sole measurement or agreement within threshold
-- 1 =	1 6dFGS-6dFGS disagreement (median cz listed)
-- 2 =	1 6dFGS-ZCAT disagreement (6dFGS cz listed)
-- 4 =	1 6dFGS-2dFGRS disagreement (6dFGS cz listed)
-- 8 =	1 6dFGS-SDSS disagreement (6dFGS cz listed)
-- 16 =	1 6dFGS-ZCAT agreement (6dFGS cz listed)
-- 32 =	1 6dFGS-2dFGRS agreement (6dFGS cz listed)
-- 64 =	1 6dFGS-SDSS agreement (6dFGS cz listed) 
--
-- Source code for cz value (r_cz) values:
-- 0 = 	no (or unreliable 6dFGS) measure;
-- 1..99 = 	ZCAT;
-- 126 = 	6dFGS;
-- 250 = 	SDSS;
-- 392 = 	2dFGRS 

SELECT
  s.Target      AS 6dfgs_id, 
  s.RAJ2000     AS 6dfgs_ra, 
  s.DEJ2000     AS 6dfgs_dec,
  s.SpecID      AS 6dfgs_spec_id, 
  s.PrID        AS 6dfgs_program,
  s.cz          AS 6dfgs_cz, -- recession velocity
  s.e_cz        AS 6dfgs_cz_err, 
  s.q_cz        AS 6dfgs_cz_quality, -- see above for meaning of quality flag
  s.f_cz        AS 6dfgs_cz_flag, -- see above for meaning of cross-survey flag
  s.r_cz        AS 6dfgs_cz_source, -- source code for cz value, see above
  s."S/G"       AS 6dfgs_class, -- 1=galaxy, 2=star, 3=unclassifiable, 4=noise 
  s.AV          AS 6dfgs_ext, -- extinction in V band
  s.bJmag       AS 6dfgs_mag_b,
  6.7           AS fiber_diameter,
  '6dfgs'       AS "sample"
FROM "VII/259/6dfgs" AS s
WHERE
  s.q_cz >= 3
  AND s.cz IS NOT NULL
  AND (s.f_cz = 0 OR s.f_cz >= 16)
  AND (s."S/G" = 1 OR s."S/G" = 2)
```

### Schema of this file

| Key present            | Type                                       | Required   | Default   | Meaning                                                                                                                                                                                             |
|------------------------|--------------------------------------------|------------|-----------|-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `survey_slug`          | str                                        | yes        | —         | Canonical lowercase identifier for the survey and default column prefix.                                                                                                                            |
| `name`                 | str                                        | yes        | —         | Human-readable survey and data-release name used in logs, plans, and reports.                                                                                                                       |
| `catalog`              | list of catalog entries                    | no         | []        | Remote or local catalogues queried for target extraction or crossmatching.                                                                                                                          |
| `crossmatch`           | mapping                                    | no         | None      | How a table of positions is matched against this survey. Required for a survey with an `lsdb` catalog; `crossmatch` null removes it, and a survey without it takes no part in `mosaics-crossmatch`. |
| `catalog_filters`      | mapping (prefix -> SQL expression or note) | no         | {}        | Post-hoc DuckDB SQL quality/depth/redshift cuts applied during production (nulls failing survey columns without dropping the row).                                                                  |
| `target_selection`     | mapping                                    | no         | None      | How this survey seeds or joins the candidate table in the targets pipeline.                                                                                                                         |
| `spectra`              | list of spectra entries                    | no         | []        | 1D spectrum download backends (`url`, `template`, `sparcl`, `desi-healpix`).                                                                                                                        |
| `spectroscopic_system` | mapping                                    | no         | None      | Spectrograph fibre diameter, disperser, redshift tiebreak rank, and flux calibration facts.                                                                                                         |

### Adapter parameters

**Catalogue `6dfgs` (`adapter: tap`):**

| Parameter           | Allowed Type   | Default                                 | Description                                                                                                                                  |
|---------------------|----------------|-----------------------------------------|----------------------------------------------------------------------------------------------------------------------------------------------|
| `service`           | str            | —                                       | Base URL of the IVOA TAP endpoint (e.g. `http://tapvizier.cds.unistra.fr/TAPVizieR/tap`, `https://mast.stsci.edu/vo-tap/api/v0.1/hasp/tap`). |
| `table`             | str            | None (read from SQL query)              | Remote TAP table name (quoted when required by VizieR, e.g. `"VII/250/2dfgrs"`).                                                             |
| `mode`              | str            | `spatial`                               | Crossmatch join mode (`spatial` or `relational`).                                                                                            |
| `key_column`        | str            | None                                    | Local key column when `mode: relational`.                                                                                                    |
| `remote_key_column` | str            | None                                    | Remote key column when `mode: relational`.                                                                                                   |
| `remote_ra_column`  | str            | `ra`                                    | Remote right ascension column name.                                                                                                          |
| `remote_dec_column` | str            | `dec`                                   | Remote declination column name.                                                                                                              |
| `chunk_size`        | int            | 50000                                   | Positions uploaded per TAP crossmatch batch.                                                                                                 |
| `use_async`         | bool           | false                                   | Submit queries as asynchronous UWS TAP jobs.                                                                                                 |
| `dialect`           | str            | `top`                                   | SQL row-limit dialect (`top` for ADQL `SELECT TOP n`, `limit` for `LIMIT n`).                                                                |
| `chunk_mib`         | float          | 28.0 (the command line’s `--chunk-mib`) | Byte budget of one inline upload, in MiB, for a service that cannot take the default.                                                        |
| `footprint_table`   | str            | None                                    | A table with one `s_region` polygon per sky patch, from which `mosaics-crossmatch moc` builds the survey’s footprint.                        |
- **Spectra (`adapter: url`, format `generic`):** configured `adapter_params`: `url_column='sixdfgs_access_url'`.

### Common changes

- **Redshift range filter**: Override the catalogue redshift constraint in your entrypoint YAML via `overrides[surveys/6dfgs].catalog_filters.6dfgs` (default: `'"6dfgs_z" BETWEEN 0.0 AND 2.5'`). Note that `6dfgs_z` is derived from the heliocentric recession velocity `6dfgs_cz` ($z = cz/c$). To disable the filter, set `overrides[surveys/6dfgs].catalog_filters.6dfgs: null`.
- **Target selection quality cuts**: Edit `defs/surveys/spec_6dfgs_dr3.sql` to adjust quality filtering (`s.q_cz >= 3`). If studying quasars that were observed before quasar cross-correlation templates were introduced into the pipeline, consider relaxing the cut to `s.q_cz >= 2` and filtering downstream on `q_cz`.
- **Disable catalogue crossmatching**: Turn off 6dFGS catalogue queries in an entrypoint configuration by setting `overrides[surveys/6dfgs].catalog.0.enabled: false`.
- **Spectrum access column**: The spectrum retrieval adapter relies on the SSA-resolved URL column (`url_column: sixdfgs_access_url`). To skip downloading 1D spectra, set `overrides[surveys/6dfgs].spectra: []`.

### Commands

```console
$ uv run -m mosaics.cli.spectrum 6dfgs -t test/objects_sample_1k.csv -o 6dfgs_spectra.lance
$ uv run -m mosaics.crossmatch.cli run --survey surveys/6dfgs -i test/objects_sample_1k.csv -o 6dfgs_matched.parquet
$ uv run -m mosaics.cli.targets --config defs/targets.yaml -o ./targets_run --dec-range 0 1 --plan plan.yaml
```

## Data served by the archive

- **Spectroscopic uncertainties and ivar**: 6dFGS does not publish per-sample uncertainty or error arrays. The spectrum decoding pipeline checks for finiteness only (`arrays: [flux, wavelength, mask]`), without producing an inverse variance (`ivar`) array.
- **Flux calibration**: Spectra are uncalibrated counts (`flux_calibration: none`, `flux_scale: null`). To combine 6dFGS spectra with absolute-calibrated surveys like SDSS or DESI, the counts must be rescaled against broad-band photometry (such as the $b_J$ magnitude `bJmag` included in the target query, or 2MASS near-infrared magnitudes).
- **Redshift quality codes (`q_cz`)**: Values range from 0 to 6. Values $Q = 3$ (‘probable’, ~90% confidence) and $Q = 4$ (‘reliable’, ~99% confidence, rms error $\sim 85\text{ km/s}$) indicate reliable galaxy redshifts; $Q = 6$ marks confirmed Galactic stars; $Q = 1$ is unusable; $Q = 2$ indicates possible but doubtful redshifts, though it includes bona fide quasars that lacked cross-correlation templates in early survey phases. The default target query selects `s.q_cz >= 3`.
- **Cross-survey consistency flag (`f_cz`)**: Flag $f_{cz} = 0$ indicates a sole measurement or agreement within threshold; values $\ge 16$ denote agreement with external catalogues (ZCAT, 2dFGRS, or SDSS). The default query requires `(s.f_cz = 0 OR s.f_cz >= 16)`.
- **Disperser configuration**: Bare ruled gratings (600V/316R) were employed before October 2002, and volume phase holographic (VPH 580V/425R) gratings were used thereafter. Because DR3 does not publish an observation epoch column to separate the setups, the instrument disperser is classified uniformly as `grating`.
- **FITS structure**: In the multi-extension FITS file, the combined, spliced $V+R$ spectrum is stored in HDU 7 (or in the primary HDU when separate extensions are omitted).

### Catalogue: `VII/259/6dfgs`

The 6dF Galaxy Survey Redshift Catalogue Data Release 3 (final release)

**Service URL:** `https://tapvizier.cds.unistra.fr/TAPVizieR/tap`

| Column    | Type           | Unit   | UCD                        | Description                                                                           |
|-----------|----------------|--------|----------------------------|---------------------------------------------------------------------------------------|
| `6dFGS`   | `char`         | `—`    | `meta.id;meta.main`        | Galaxy survey target ID (gHHMMSSs+DDMMSS)                                             |
| `Nm`      | `unsignedByte` | `—`    | `meta.number`              | Number of 6dFGS measurements (of any quality)                                         |
| `bJmag`   | `float`        | `mag`  | `phot.mag;em.opt.B`        | Recalibrated b_J_ magnitude (BMAG)                                                    |
| `rFmag`   | `float`        | `mag`  | `phot.mag;em.opt.R`        | Recalibrated r_F_ magnitude (RMAG)                                                    |
| `S/G`     | `unsignedByte` | `—`    | `meta.id`                  | [0/4] SuperCOSMOS star/galaxy classifier: 1=galaxy, 2=star, 3=unclassifiable, 4=noise |
| `f_cz`    | `short`        | `—`    | `meta.code`                | Comparison flags between z measurements (2)                                           |
| `cz`      | `int`          | `km/s` | `phys.veloc;meta.modelled` | Best available combined recession velocity cz                                         |
| `e_cz`    | `short`        | `km/s` | `stat.error`               | Combined uncertainty in cz (3)                                                        |
| `q_cz`    | `unsignedByte` | `—`    | `meta.code.qual`           | Best redshift quality value available (5)                                             |
| `AV`      | `float`        | `mag`  | `phys.absorption.gal`      | Galactic extinction in V magnitudes (from Schlegel et al., 1998ApJ…500..525S)         |
| `Target`  | `int`          | `—`    | `meta.id`                  | [2/179260] TARGETID number in 6dFGS database                                          |
| `SpecID`  | `char`         | `—`    | `meta.id;meta.file`        | SPECID values of redshifts contributing to the final combined redshift                |
| `Sp`      | `char`         | `—`    | `meta.ref.url`             | show the spectrum (if existing) and thumbnails of associated target                   |
| `Obs`     | `char`         | `—`    | `meta.ref.url`             | show the parameters of the spectra available for this target                          |
| `Simbad`  | `char`         | `—`    | `—`                        | Simbad column added by the CDS                                                        |
| `RAJ2000` | `char`         | `—`    | `pos.eq.ra;meta.main`      | Right ascension (J2000)                                                               |
| `DEJ2000` | `char`         | `—`    | `pos.eq.dec;meta.main`     | Declination (J2000)                                                                   |

### Images

No image cutout backends are configured for this survey.

### Spectra

- **Fibre diameter:** `per_row` | **Disperser:** `grating` | **Redshift rank:** `5` | **Flux calibration:** `none` | **Flux scale:** `None`
- **Adapter `url`:** decoder format `generic`, output parallel arrays `['flux', 'wavelength', 'mask']`, params `{'url_column': 'sixdfgs_access_url'}`.
  - **hdus:** HDU 7 (`V+R` combined spectrum) or primary 1D spectrum HDU with linear WCS.
  - **wavelength_range_angstrom:** 4000 - 7500 A (observer frame)
  - **resolution:** R ~ 1000 (6.7 arcsec fibres)
  - **flux_calibration:** none (uncalibrated counts)
  - **arrays:** [‘flux’, ‘wavelength’, ‘mask’]

## References and acknowledgement

- Jones, D. H., et al. (2004), *MNRAS*, 355, 747 (`doi:10.1111/j.1365-2966.2004.08353.x`) — The 6dF Galaxy Survey: samples, observational techniques and the first data release.
- Jones, D. H., et al. (2009), *MNRAS*, 399, 683 (`doi:10.1111/j.1360-9017.2009.15577.x`) — The 6dF Galaxy Survey: final redshift release (DR3) and velocity library.
- VizieR Catalogue: `VII/259/6dfgs` ([https://cdsarc.cds.unistra.fr/viz-bin/cat/VII/259](https://cdsarc.cds.unistra.fr/viz-bin/cat/VII/259)).
- Survey Archive: [http://www-wfau.roe.ac.uk/6dFGS/](http://www-wfau.roe.ac.uk/6dFGS/).

Users of 6dFGS data should cite Jones et al. (2004, 2009) and include the following acknowledgement: “The 6dF Galaxy Survey was conducted using the Six-Degree Field facility on the 1.2-m UK Schmidt Telescope at Siding Spring Observatory, operated by the Anglo-Australian Observatory (AAO).” When querying data via CDS VizieR, acknowledge the VizieR database operated at CDS, Strasbourg, France.

## Provenance of this page

| Source ID       | Kind             | Service / Endpoint                               | Query or URL                                                                              | Retrieved (UTC)        |
|-----------------|------------------|--------------------------------------------------|-------------------------------------------------------------------------------------------|------------------------|
| `catalog_6dfgs` | `tap_table`      | `https://tapvizier.cds.unistra.fr/TAPVizieR/tap` | `https://vizier.cds.unistra.fr/viz-bin/votable?-source=VII/259/6dfgs&-out.max=0&-out.all` | `2026-10-18T16:52:28Z` |
| `spectra_6dfgs` | `spectrum_model` | `http://www-wfau.roe.ac.uk/6dFGS/`               | `http://www-wfau.roe.ac.uk/6dFGS/`                                                        | `2026-10-18T16:28:54Z` |
<!-- /generated -->
