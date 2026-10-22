# Cutout Lance Dataset Schema

`mosaics-cutout` writes a directory `<output>.lance` in [Lance](https://lancedb.github.io/lance/) format (built on Apache Arrow), plus an optional `<output>.failures.parquet` sidecar when one or more stamps fail.

## 1. Column layout

Every row in `<output>.lance` contains:

1. **All columns from the input target table** (plus `mosaics_id: int64` if no key column was supplied).
2. **One fixed-shape tensor column per cutout service / band:**
   - **Single-band cutouts (`cutout full`, `lsv`, `nersc`, `sia`, `vsa`, single-band `hips`):** named `cutout_<survey>_<band>` (or `image` for single-service runs such as `mosaics-cutout hips`), stored as an Arrow `FixedShapeTensorType` of element type `float32` (or `uint8` for JPEG/PNG) and logical shape `(C, H, W)` where `C = 1` for a single band, `C = N` for an unsplit multi-band stack, or `C = 3` for `[color]` RGB stamps, and `H = W = --size`.
   - **Failed stamps:** represented as an Arrow `null` at the row level (never an array of `NaN`s or zeros), so `is_null()` distinguishes a failed cutout from a dark sky region.

## 2. Arrow field calibration metadata

Each cutout tensor field in the Lance schema carries its photometric pixel calibration in Arrow field metadata (`field.metadata`, UTF-8 byte keys and values):

| Metadata key          | Example value                                                      | Meaning                                                                                                                                                                |
|-----------------------|--------------------------------------------------------------------|------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `mosaics_unit`        | `b"nanomaggy"`, `b"ab_zeropoint"`, `b"counts_per_s"`, `b"unknown"` | Physical calibration regime of the pixel values.                                                                                                                       |
| `mosaics_calibration` | `b"calibrated"` or `b"uncalibrated"`                               | Whether the pixels can be converted directly to physical flux density ($\mu\text{Jy}$).                                                                                |
| `mosaics_scale`       | `b"3.6307805"`                                                     | Multiplicative factor converting native pixel values to microjanskys ($\mu\text{Jy}$), where $3.6307805\ \mu\text{Jy} = 1\ \text{nanomaggy}$ ($m_{\text{AB}} = 22.5$). |
| `mosaics_offset`      | `b"0.0"`                                                           | Additive offset in microjanskys applied after scaling ($\text{flux}_{\mu\text{Jy}} = \text{pixel} \times \text{scale} + \text{offset}$).                               |

## 3. The `.failures.parquet` sidecar

When any `(target, service)` request fails during a run, MOSAICS writes `<output>.failures.parquet` next to `<output>.lance`:

| Column     | Type                | Meaning                                                                                   |
|------------|---------------------|-------------------------------------------------------------------------------------------|
| `<id_col>` | matches input key   | Target key value from the input table.                                                    |
| `service`  | `string`            | Service label (`<survey>:<adapter>:<band_or_stack>`, e.g. `ls11:lsv:stack`).              |
| `category` | `string`            | Failure classification (`http_status`, `timeout`, `decode`, `empty_coverage`, `network`). |
| `status`   | `int32` (nullable)  | HTTP status code when applicable (`404`, `429`, `503`, …).                                |
| `error`    | `string`            | Human-readable error message from the transport or decoder.                               |
| `url`      | `string` (nullable) | Request URL that failed.                                                                  |
