# 2dF Galaxy Redshift Survey (`2dfgrs`)

<!-- generated:survey-page -->

The 2dF Galaxy Redshift Survey (2dFGRS) is a major spectroscopic survey that mapped the three-dimensional distribution of more than 220,000 galaxies in the nearby universe. Carried out at the 3.9-metre Anglo-Australian Telescope (AAT) using the Two-degree Field (2dF) multi-fibre spectrograph, the survey targeted a magnitude-limited sample ($b_J \le 19.45$) covering approximately 2,000 deg² across two broad declination strips (in the Northern and Southern Galactic caps) and 99 random two-degree fields. Spectra cover optical wavelengths from 3600 to 8000 Å in the observer frame at a spectral resolution of $R \sim 600\text{--}1000$ (9 Å FWHM) using 2.1 arcsec diameter fibres and 300B ruled gratings.

In `mosaics`, 2dFGRS acts as a bright, low-redshift anchor alongside 6dFGS, populating the nearby, well-resolved galaxy population that forms the basis of morphology benchmarks. It provides target selection for spectroscopic training samples (`mosaics-targets` using `defs/surveys/spec_2dfgrs.sql`), catalogue crossmatching (`mosaics-query` via the CDS VizieR TAP service table `"VII/250/2dfgrs"`), and 1D FITS spectrum retrieval (`mosaics-spectrum 2dfgrs` or `full`).

| Property                 | Value                                                                                                                                                                                                                                                                                                                                                              |
|--------------------------|--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| What MOSAICS extracts    | Catalogue: `2dfgrs` (spectroscopic, `tap`); Spectra: `template` (flux,wavelength,mask)                                                                                                                                                                                                                                                                             |
| Archives                 | vizier (`https://tapvizier.cds.unistra.fr/TAPVizieR/tap`), `www.roe.ac.uk`                                                                                                                                                                                                                                                                                         |
| Bands / wavelength range | spectroscopy (disperser `grating`, fibre `per_row` arcsec, calibration `none`)                                                                                                                                                                                                                                                                                     |
| Used by                  | `mosaics-targets` (spectroscopic selection), `mosaics-spectrum 2dfgrs` / `full`, `mosaics-crossmatch`, `mosaics-query`                                                                                                                                                                                                                                             |
| Crossmatch radius        | `2` arcsec. Fibre aperture on the sky: 140 um cores, 2.16 arcsec at the field centre to 1.99 arcsec at its edge; fibre positioning precision 0.16 arcsec on average (11 um), none outside 0.3 arcsec (Colless et al. 2001, arXiv:astro-ph/0106498, Sect. 2). The astrometric precision of the APM input positions was not found. The radius is the fibre diameter. |

## Configuration

### The shipped file

Source: `defs/surveys/2dfgrs.yaml`

```yaml
# 2dFGRS -- spectroscopy. No per-sample errors, so no ivar array.

survey_slug: 2dfgrs
name: 2dF Galaxy Redshift Survey

catalog:
  - name: 2dfgrs
    type: spectroscopic
    adapter: tap
    adapter_params:
      service: vizier
      table: '"VII/250/2dfgrs"'
      dialect: top

# Crossmatch: the radius a table of positions is matched within, and what it rests on.
crossmatch:
  radius: 2 arcsec
  radius_note: >-
    Fibre aperture on the sky: 140 um cores, 2.16 arcsec at the field centre to 1.99 arcsec
    at its edge; fibre positioning precision 0.16 arcsec on average (11 um), none outside
    0.3 arcsec (Colless et al. 2001, arXiv:astro-ph/0106498, Sect. 2). The astrometric
    precision of the APM input positions was not found. The radius is the fibre diameter.
  # 245,591 observations (the query's header), 221,414 of them with reliable redshifts: small
  # enough to fetch over the cells the input occupies, so no catalogue is uploaded to VizieR.
  # No COUNT(*) was run for this setting.
  strategy: extract

catalog_filters:
  # See sdss17.yaml for the general rule this and the other five redshift
  # surveys share. Quoted: an identifier starting with a digit needs it.
  2dfgrs: '"2dfgrs_z" BETWEEN 0.0 AND 2.5'
  2dfgrs_note: >-
    Restricts the catalogue to the redshift range the training targets are
    drawn over; a row outside it keeps its candidacy but loses its 2dFGRS
    spectroscopic columns, as if this survey had not matched it.

target_selection:
  catalog_name: 2dfgrs
  query: spec_2dfgrs.sql
  ra_column: RAJ2000
  dec_column: DEJ2000

spectra:
  - adapter: template
    adapter_params:
      url_template: 2dfgrs
    format: generic
    # No `error`: this survey publishes no per-sample uncertainty at
    # all, so the mask below tests finiteness only -- the
    # signal-to-noise term is skipped rather than evaluated against a
    # column that is not there.
    arrays: [flux, wavelength, mask]

spectroscopic_system:
  fiber_diameter: per_row     # 2.1 arcsec, already selected by the query
  # Conventional ruled gratings, 300B for the survey (Lewis et al. 2002,
  # MNRAS 333, 279).
  disperser: grating
  redshift_rank: 6            # the older, shallower of the two
  # Uncalibrated counts: there is no factor at all, and inventing one would
  # produce a column that looks comparable to SDSS and is not.
  flux_scale: null
  flux_calibration: none
```

Target-selection query (`defs/surveys/spec_2dfgrs.sql`):

```sql
-- 2dF Galaxy Redshift Survey, final release.
--
-- Provider: tap --service vizier (CDS VizieR, ADQL).
--   uv run -m mosaics.cli.query sql --provider tap --service vizier \
--     --path sql/2dfgrs_spectra.sql --ra-col RAJ2000 --dec-col DEJ2000 \
--     -o 2dfgrs.parquet
--
-- Retrieval contract for `mosaics.cli.spectrum 2dfgrs`: SeqNum addresses a
-- spectrum, through the url-template of the same name. 2dFGRS is the one
-- survey in the corpus with no VO service at all -- no SSA, no datalink --
-- so the only route is the survey's own archive at 2dfgrs.net, and that
-- template is the least verified thing in this directory.
--
-- 245,591 spectroscopic observations, of which 221,414 have reliable
-- (Q >= 3) redshifts. Together with 6dFGS this is the corpus's bright,
-- low-redshift anchor: both are magnitude limited several magnitudes
-- brighter than DESI, and they populate exactly the nearby, well-resolved
-- galaxies that dominate the morphology benchmarks.
--
-- Where a source has several spectra the survey's own rule is to take the
-- highest quality code, then the latest observation. That is a per-object
-- choice this query cannot make in ADQL without a window function VizieR
-- does not support, so both the object name and the sequence number are
-- selected and the deduplication is left to the caller:
--
--   frame.sort_values(['q_z', 'z.obs'], ascending=False).drop_duplicates('Name')
--
-- 2dFGRS spectra are not reliably flux calibrated and are rescaled to
-- broad-band photometry, as for LAMOST and 6dFGS; Bjmag is selected for it.
-- VizieR table names contain slashes and must be quoted.

SELECT
  s.SeqNum      AS 2dfgrs_id, 
  s.Name        AS 2dfgrs_name, 
  s.RAJ2000     AS 2dfgrs_ra, 
  s.DEJ2000     AS 2dfgrs_dec,
  s.z           AS 2dfgrs_z, -- best redshift (heliocentric)
  s."z.obs"     AS 2dfgrs_z_obs, -- best observed redshift
  s.q_z         AS 2dfgrs_z_flag, -- quality flag (1 - 5), reliable redshifts have quality>=3
  s.n_z         AS 2dfgrs_z_type, -- abs = 1, emi = 2, man = 3 
  s.SNR         AS 2dfgrs_snr, 
  s.ObsRun      AS 2dfgrs_run, 
  s.UKST        AS 2dfgrs_plate, 
  s.Rx          AS 2dfgrs_r, -- cross-correlation R value, higher is better
  s.eta         AS 2dfgrs_class, -- derived from PCA, negative for early-type galaxies, positive for later-type galaxies
  s.Bjmag       AS 2dfgrs_mag_b, -- without extinction correction
  s.Bjsel       AS 2dfgrs_mag_b_dered, -- extinction corrected
  s.Gext        AS 2dfgrs_ext,
  2.1           AS fiber_diameter,
  '2dfgrs'      AS "sample"
  -- s."z.em"      AS 2dfgrs_z_em, 
  -- s."z.abs"     AS 2dfgrs_z_abs, 
FROM "VII/250/2dfgrs" AS s
WHERE
  s.q_z >= 3
  AND s.z IS NOT NULL
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

**Catalogue `2dfgrs` (`adapter: tap`):**

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
- **Spectra (`adapter: template`, format `generic`):** configured `adapter_params`: `url_template='2dfgrs'`.

### Common changes

- **Redshift range filter**: Adjust or remove the catalogue redshift filter in your entrypoint YAML by overriding `overrides[surveys/2dfgrs].catalog_filters.2dfgrs` (default: `'"2dfgrs_z" BETWEEN 0.0 AND 2.5'`). To remove the filter entirely, set `overrides[surveys/2dfgrs].catalog_filters.2dfgrs: null`.
- **Target selection cuts**: Modify `defs/surveys/spec_2dfgrs.sql` to tune the quality threshold (`s.q_z >= 3`), or add cuts on the spectral type parameter `s.eta` (derived from principal component analysis, where negative values indicate early-type galaxies and positive values indicate late-type galaxies) or cross-correlation peak `s.Rx`.
- **Disable catalogue crossmatching**: Disable 2dFGRS catalogue crossmatching in a composite pipeline run by setting `overrides[surveys/2dfgrs].catalog.0.enabled: false`.
- **Disable spectrum retrieval**: Skip downloading 1D spectra for 2dFGRS by setting `overrides[surveys/2dfgrs].spectra: []` in your entrypoint YAML.

### Commands

```console
$ uv run -m mosaics.cli.spectrum 2dfgrs -t test/objects_sample_1k.csv -o 2dfgrs_spectra.lance
$ uv run -m mosaics.crossmatch.cli run --survey surveys/2dfgrs -i test/objects_sample_1k.csv -o 2dfgrs_matched.parquet
$ uv run -m mosaics.cli.targets --config defs/targets.yaml -o ./targets_run --dec-range 0 1 --plan plan.yaml
```

## Data served by the archive

- **Spectroscopic uncertainties and ivar**: 2dFGRS publishes no per-sample measurement uncertainties or error arrays. Consequently, no inverse variance (`ivar`) array is constructed; the spectrum extraction pipeline checks for finiteness only (`arrays: [flux, wavelength, mask]`), omitting the signal-to-noise weighting step used for surveys with published error columns.
- **Flux calibration**: Spectra are recorded in uncalibrated analogue-to-digital counts (`flux_calibration: none`, `flux_scale: null`). No physical flux conversion factor exists in the archive; to combine 2dFGRS spectra with flux-calibrated surveys like SDSS or DESI, spectra must be rescaled to broad-band photometry (such as the unextincted $b_J$ magnitude `Bjmag` or dereddened `Bjsel` included in the target query).
- **Redshift quality codes**: Redshift quality `q_z` ranges from 1 to 5, where $q_z \ge 3$ indicates reliable redshifts (notionally 90% or greater confidence). The default target selection query requires `s.q_z >= 3 AND s.z IS NOT NULL`.
- **Duplicate observations**: When multiple spectroscopic observations exist for an object, the survey protocol selects the highest quality code (`q_z`), breaking ties with the latest observation epoch (`z.obs`). Because VizieR ADQL 2.0 does not support window functions, deduplication is executed downstream after table retrieval.
- **Archive access and naming**: The VizieR catalogue identifier contains slashes and digits and must be quoted in ADQL queries (`"VII/250/2dfgrs"`). Because 2dFGRS does not provide standard IVOA SSA or Datalink endpoints, FITS spectra are retrieved via a URL template addressing `SeqNum` against the survey’s web archive.

### Catalogue: `VII/250/2dfgrs`

Best observations of 2dFGRS

**Service URL:** `https://tapvizier.cds.unistra.fr/TAPVizieR/tap`

| Column    | Type           | Unit   | UCD                           | Description                                                                       |
|-----------|----------------|--------|-------------------------------|-----------------------------------------------------------------------------------|
| `recno`   | `int`          | `—`    | `meta.record`                 | Record number assigned by the VizieR team. Should Not be used for identification. |
| `SeqNum`  | `char`         | `—`    | `meta.id`                     | 2dFGRS serial number (SEQNUM)                                                     |
| `Name`    | `char`         | `—`    | `meta.id;meta.main`           | 2dFGRS name (NAME)                                                                |
| `UKST`    | `char`         | `—`    | `obs.field`                   | UKST plate number (IFIELD)                                                        |
| `RAJ2000` | `char`         | `—`    | `pos.eq.ra;meta.main`         | Right ascension (J2000, hours)                                                    |
| `DEJ2000` | `char`         | `—`    | `pos.eq.dec;meta.main`        | Declination (J2000, degrees)                                                      |
| `Bjmag`   | `float`        | `mag`  | `phot.mag;em.opt.B`           | Final bj magnitude (BJG) (1)                                                      |
| `Bjsel`   | `float`        | `mag`  | `phot.mag;em.opt.B`           | Final bj magnitude corrected from extinction (BJSEL) (G2)                         |
| `z`       | `float`        | `—`    | `src.redshift`                | ? Best redshift (heliocentric) (Z_HELIO) (2)                                      |
| `q_z`     | `unsignedByte` | `—`    | `meta.code.qual;src.redshift` | [1,5] Redshift quality for best spectrum (reliable redshifts have quality>=3) (2) |
| `n_z`     | `unsignedByte` | `—`    | `meta.note`                   | Redshift type (abs=1, emi=2, man=3) (ABEMMA)                                      |
| `SNR`     | `float`        | `—`    | `em.wl.central;stat.median`   | Median signal to noise ratio per pixel (2)                                        |
| `Sp`      | `char`         | `—`    | `meta.ref.url`                | Show the Image and annotated Spectrum of this 2dF Object                          |

### Images

No image cutout backends are configured for this survey.

### Spectra

- **Fibre diameter:** `per_row` | **Disperser:** `grating` | **Redshift rank:** `6` | **Flux calibration:** `none` | **Flux scale:** `None`
- **Adapter `template`:** decoder format `generic`, output parallel arrays `['flux', 'wavelength', 'mask']`, params `{'url_template': '2dfgrs'}`.
  - **hdus:** Primary HDU 0 carries the 1D flux array and linear wavelength WCS (`CRVAL1`, `CDELT1`, `CRPIX1`).
  - **wavelength_range_angstrom:** 3600 - 8000 A (observer frame, air)
  - **resolution:** R ~ 600 - 1000 (9 A FWHM, 2.0 arcsec fibres)
  - **flux_calibration:** none (uncalibrated ADU counts)
  - **arrays:** [‘flux’, ‘wavelength’, ‘mask’]

## References and acknowledgement

- Colless, M., et al. (2001), *MNRAS*, 328, 1039 (`doi:10.1046/j.1365-8711.2001.04902.x`) — The 2dF Galaxy Redshift Survey: spectra and redshifts.
- Colless, M., et al. (2003), arXiv:astro-ph/0306581 — The 2dF Galaxy Redshift Survey: Final Data Release.
- Lewis, I. J., et al. (2002), *MNRAS*, 333, 279 (`doi:10.1046/j.1365-8711.2002.05333.x`) — The 2dF facility at the Anglo-Australian Telescope.
- VizieR Catalogue: `VII/250/2dfgrs` ([https://cdsarc.cds.unistra.fr/viz-bin/cat/VII/250](https://cdsarc.cds.unistra.fr/viz-bin/cat/VII/250)).
- Survey Archive: [https://www.roe.ac.uk/~jap/2df/](https://www.roe.ac.uk/~jap/2df/) and [http://www.2dfgrs.net/](http://www.2dfgrs.net/).

Publications using 2dFGRS data should cite Colless et al. (2001, 2003) and Lewis et al. (2002). Authors should include the standard acknowledgement: “The 2dF Galaxy Redshift Survey was made possible through the dedicated efforts of the staff of the Anglo-Australian Observatory, both in creating the Two-degree Field facility and in supporting the survey observations.” When data are retrieved via CDS VizieR, acknowledge the VizieR service operated at CDS, Strasbourg, France.

## Provenance of this page

| Source ID        | Kind             | Service / Endpoint                               | Query or URL                                                                               | Retrieved (UTC)        |
|------------------|------------------|--------------------------------------------------|--------------------------------------------------------------------------------------------|------------------------|
| `catalog_2dfgrs` | `tap_table`      | `https://tapvizier.cds.unistra.fr/TAPVizieR/tap` | `https://vizier.cds.unistra.fr/viz-bin/votable?-source=VII/250/2dfgrs&-out.max=0&-out.all` | `2026-10-18T16:52:24Z` |
| `spectra_2dfgrs` | `spectrum_model` | `https://www.roe.ac.uk/~jap/2df/`                | `https://www.roe.ac.uk/~jap/2df/`                                                          | `2026-10-18T16:28:12Z` |
<!-- /generated -->
