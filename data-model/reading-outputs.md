# Reading Output Datasets in Python

MOSAICS writes standard Apache Parquet (`.parquet`) and Lance (`.lance`) datasets that open directly in `lance` (`pylance`), `pyarrow`, `pandas`, and PyTorch without requiring MOSAICS itself to be imported.

## 1. Reading a cutout or spectrum Lance dataset with `lance` and `pyarrow`

```python
import lance
import numpy as np

# Open the dataset on disk (zero-copy metadata read)
ds = lance.dataset('cutouts.lance')
print(ds.schema)

# Inspect pixel calibration metadata on a cutout column
field = ds.schema.field('cutout_ls11_g')
print(field.metadata)
# {b'mosaics_unit': b'nanomaggy', b'mosaics_calibration': b'calibrated', b'mosaics_scale': b'3.6307805', b'mosaics_offset': b'0.0'}

# Read a slice of rows or specific columns into PyArrow
table = ds.to_table(columns=['id', 'ra', 'dec', 'cutout_ls11_g'], limit=32)

# Convert a FixedShapeTensor column to a (N, C, H, W) NumPy float32 array
tensor_col = table['cutout_ls11_g'].combine_chunks()
stamps = tensor_col.to_numpy_ndarray()  # shape: (32, 1, 96, 96)
```

## 2. Reading variable-length spectra and applying the boolean mask

```python
import lance
import numpy as np

ds = lance.dataset('sdss17_spectra.lance')
table = ds.to_table(
  columns=['id', 'sdss17_wavelength', 'sdss17_flux', 'sdss17_error', 'sdss17_mask'],
  limit=10,
)

for row_idx in range(table.num_rows):
  if table['sdss17_flux'][row_idx].is_valid:
    wl = np.asarray(table['sdss17_wavelength'][row_idx].as_py(), dtype=np.float32)
    flux = np.asarray(table['sdss17_flux'][row_idx].as_py(), dtype=np.float32)
    err = np.asarray(table['sdss17_error'][row_idx].as_py(), dtype=np.float32)
    usable = np.asarray(table['sdss17_mask'][row_idx].as_py(), dtype=bool)
    # Keep only unmasked samples (usable == True)
    wl_clean, flux_clean, err_clean = wl[usable], flux[usable], err[usable]
```

## 3. Reading the targets catalogue with `pyarrow` or `pandas`

```python
import pandas as pd
import pyarrow.parquet as pq

# Read only the anchor coordinates, redshift, and Legacy Survey + VHS magnitudes
cols = [
  'id', 'ra', 'dec', 'z', 'z_survey', 'split',
  'ls11', 'ls11_mag_g', 'ls11_mag_r', 'ls11_mag_z',
  'vhs5', 'vhs5_mag_aper3_j', 'vhs5_mag_aper3_ks',
]
df = pd.read_parquet('targets_run/targets.parquet', columns=cols)
train_df = df[df['split'] == 'train']
```

## 4. Inspecting the `.failures.parquet` sidecar

```python
import pandas as pd

failures = pd.read_parquet('cutouts.failures.parquet')
print(failures.groupby(['service', 'category']).size())
```
