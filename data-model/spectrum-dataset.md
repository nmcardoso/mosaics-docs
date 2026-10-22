# Spectrum Lance Dataset Schema

`mosaics-spectrum` writes a Lance dataset `<output>.lance` containing all input target columns alongside variable-length Arrow `large_list` columns for each spectroscopic survey.

## 1. Parallel `large_list` columns

For each spectroscopic survey `<slug>` (`sdss17`, `desi1`, `lamost11`, `6dfgs`, `2dfgrs`, `hasp`, `euclidq1`), the dataset contains the parallel arrays declared in `defs/surveys/<slug>.yaml` under `spectra[].arrays`:

| Column              | Arrow Type                  | Units                                                                                                     | Description                                                                                                 |
|---------------------|-----------------------------|-----------------------------------------------------------------------------------------------------------|-------------------------------------------------------------------------------------------------------------|
| `{slug}_flux`       | `large_list<item: float32>` | $10^{-17}\ \text{erg}\ \text{s}^{-1}\ \text{cm}^{-2}\ \text{\AA}^{-1}$ (when calibrated) or native counts | 1D spectral flux density per wavelength pixel.                                                              |
| `{slug}_error`      | `large_list<item: float32>` | Same unit as `{slug}_flux`                                                                                | $1\sigma$ uncertainty per pixel (omitted for `2dfgrs` and `6dfgs`, which publish no per-pixel error).       |
| `{slug}_wavelength` | `large_list<item: float32>` | Angstrom ($\text{\AA}$)                                                                                   | Observer-frame wavelength per pixel.                                                                        |
| `{slug}_mask`       | `large_list<item: bool>`    | Boolean (`true` = usable)                                                                                 | Per-pixel usability flag combining finiteness, archive quality bitmasks, positive error, and `--snr-floor`. |

On every row for a given `<slug>`, all present array columns have **identical list length** ($L \ge 0$), or all are simultaneously Arrow `null` when the observation failed or was not available.

## 2. Field and schema metadata

Each `{slug}_flux` field carries calibration metadata in its Arrow field metadata (`field.metadata`):

| Metadata key          | Example value                                         | Meaning                                                                                                    |
|-----------------------|-------------------------------------------------------|------------------------------------------------------------------------------------------------------------|
| `mosaics_unit`        | `b"1e-17 erg / (Angstrom cm2 s)"` or `b"counts"`      | Flux unit after applying `spectroscopic_system.flux_scale`.                                                |
| `mosaics_calibration` | `b"absolute"`, `b"relative"`, `b"none"`, `b"unknown"` | Spectroscopic flux calibration regime from `spectroscopic_system.flux_calibration`.                        |
| `mosaics_scale`       | `b"1.0"`                                              | Multiplier applied during decoding.                                                                        |
| `mosaics_snr_floor`   | `b"2.0"`                                              | Minimum per-pixel signal-to-noise ratio applied when computing `{slug}_mask` (when `--snr-floor` was set). |
