# Using This Documentation with an LLM Agent

This documentation site is published in both human-readable HTML and machine-readable Markdown so that LLM coding agents, research assistants, and automated scripts can inspect `mosaics` CLI flags, survey configurations, and upstream archive data models without scraping HTML.

## Machine-Readable Endpoints

Every page on this site has a plain-Markdown twin, and the site root publishes two index files following the `llms.txt` standard:

- **[`llms.txt`](https://nmcardoso.github.io/mosaics-docs/llms.txt)** — Concise index of every documentation page with its title and direct `.md` URL. Point an agent here first so it can discover which page answers a question without loading the entire documentation into context.
- **[`llms-full.txt`](https://nmcardoso.github.io/mosaics-docs/llms-full.txt)** — Single-file concatenation of the complete documentation in Markdown. Useful for long-context models or offline indexing.
- **Per-page `.md` twins** — Replace `.html` with `.md` on any page URL (or append `index.md` to directory URLs) to fetch the raw Markdown source of that page, including all generated CLI tables, configuration schemas, and survey data-model tables:
  - [`for-agents.md`]() (this page in Markdown)
  - [`surveys/index.md`](surveys/index.md) (the survey capability matrix)
  - `surveys/<slug>.md` (for example, [`surveys/ls11.md`](surveys/ls11.md), [`surveys/desi1.md`](surveys/desi1.md), [`surveys/lsstdp2.md`](surveys/lsstdp2.md))
  - `cli/<command>.md` (for example, [`cli/cutout.md`](cli/cutout.md), [`cli/targets.md`](cli/targets.md), [`cli/spectrum.md`](cli/spectrum.md))
  - [`configuration/index.md`](configuration/index.md) (complete YAML schema and adapter parameter tables)

## Step-by-Step Workflow for Agents

When asked to design, configure, or debug a `mosaics` data extraction run, an LLM agent should follow this sequence:

### 1. Discover Available Surveys and Capabilities

Fetch [`surveys/index.md`](surveys/index.md) to inspect the **Survey Capability Matrix**. Identify which surveys provide:

- `targets` stage-1 selection (`spectroscopic` or `photometric`),
- stage-3 photometric or stage-4 spectroscopic `catalog` crossmatching,
- `cutout` imaging stamps (and which backend: `hips`, `lsv`, or `nersc`),
- `spectrum` 1D spectra (`url`, `template`, `sparcl`, or `desi-healpix`).

### 2. Inspect Survey Schemas and Shipped YAML/SQL

For each survey `<slug>` involved in the user’s task, fetch `surveys/<slug>.md` (for example, [`surveys/euclidq1.md`](surveys/euclidq1.md) or [`surveys/ls11.md`](surveys/ls11.md)). Each survey page contains:

- The **exact shipped YAML configuration** (`defs/surveys/<slug>.yaml`) and any **SQL query** (`defs/sql/<slug>.sql`).
- The **Source Data Models** table listing every column in the upstream archive table, its data type, physical unit, UCD, whether `mosaics` selects it by default, and what prefixed column name (`{prefix}<col>`) it maps to in the output dataset.

### 3. Construct or Customise a Configuration File

If the user needs additional archive columns, tighter quality constraints, or a subset of surveys:

1. Check [`configuration/index.md`](configuration/index.md) for the exact allowed keys in each section (`catalog`, `catalog_filters`, `column_mapping`, `cutout`, `spectra`, `target_selection`) and `adapter_params`.
2. Remember the strict loader rules:
   - Write the aliases in `column_mapping` **without** a prefix: the loader adds the survey’s (`{slug}_`, or `{slug}p_` / `{slug}s_` when a survey defines both photometric and spectroscopic catalogues), so `ra: ra` gives `{slug}_ra`. An alias that already starts with the prefix raises. Columns of a hosted survey go under a key named after its `provides` prefix.
   - Unknown keys or invalid enum values (`pixel_unit`, `magnitude_system`, `wavelength_frame`, `error_representation`) will fail validation immediately.

### 4. Select the Right CLI Command and Flags

Fetch the relevant CLI reference page before proposing commands:

- **Cutouts:** [`cli/cutout.md`](cli/cutout.md) (`mosaics-cutout hips`, `lsv`, `nersc`, `full`, `peek`).
- **Spectra:** [`cli/spectrum.md`](cli/spectrum.md) (`mosaics-spectrum fetch`, `discover`, `full`, `peek`).
- **Queries:** [`cli/query.md`](cli/query.md) (`mosaics-query cone`, `region`, `download`, `sql`, `sweep`, `ssa`).
- **Targets Pipeline:** [`cli/targets.md`](cli/targets.md) (`mosaics-targets run`, `selection`, `ls11`, `photometric`, `spectroscopic`, `production`, `report`).
- **Crossmatch:** [`cli/crossmatch.md`](cli/crossmatch.md) (`mosaics-crossmatch run`, `moc`, `jobs`, `kill`, `del`): a table of positions against any survey that has a `crossmatch` section, including the Rubin/LSST Data Previews. Concepts: [`concepts/crossmatch.md`](concepts/crossmatch.md); Astro Data Lab without a login (`adl-healpix`, the adapter of `wise`, `gaia3` and `ukidss11`): [`concepts/datalab-healpix.md`](concepts/datalab-healpix.md).
- **Field of view:** [`cli/fov.md`](cli/fov.md) (`mosaics-fov`): add the cutout size that frames each galaxy to any table with the Legacy Survey shape and `r` magnitude; `mosaics-crossmatch run --fov SURVEY` does the same after a match, and `--hle` (on `mosaics-fov`, `mosaics-crossmatch run --fov` and `mosaics-targets`) chooses the half-light-ellipse method alone; `mosaics-cutout --default-fov` sizes the targets whose `fov` is empty. How it is computed: [`concepts/field-of-view.md`](concepts/field-of-view.md).
- **HiPSCat:** [`cli/hipscat.md`](cli/hipscat.md) (`mosaics-hipscat convert`, `import`, `crossmatch`, `info`, `validate`).
- **Distributed Proxy:** [`cli/proxy.md`](cli/proxy.md) (`mosaics-proxy up`, `serve`, `worker`, `status`, `down`).
- **Reports:** [`cli/reports.md`](cli/reports.md) (`mosaics-report targets`, `crossmatch`, `compare-stamps`, `db-constraints`).

Always include `--resume` on long-running extraction commands (`cutout`, `spectrum`, `targets`) so interrupted runs continue cleanly from their unique `--key-column`.

### 5. Verify Outputs and Inspect Reports

After a run finishes:

- Check [`data-model/reading-outputs.md`](data-model/reading-outputs.md) for Python recipes to read Lance cutout/spectrum datasets, distinguish failed rows (`null`) from empty spectra (`[]`), and inspect `<dataset>.failures.parquet`.
- If the pipeline generated a `report/md/` folder, read `report/md/index.md` and `report/md/chapters/*.md` to verify row counts, filter funnels, and scientific assessment checks.
