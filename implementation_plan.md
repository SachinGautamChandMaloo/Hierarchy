# Optimized Store-by-Store Pipeline

This plan outlines the steps to restructure the M5 forecasting pipeline to prevent memory crashes by processing data store-by-store, resulting in 50 smaller `.pkl` files (5 per store) instead of 5 massive ones. 

## User Review Required

> [!WARNING]
> **Mean Encodings (Global vs. Local)**
> In the original notebook, mean encodings like `enc_item_id_state_id_mean` were computed globally across all stores in a state. By splitting the preprocessing per store, this feature will be computed using *only that specific store's data*. For a store-specific model, this is completely fine (and often preferred to prevent data leakage between stores), but technically it is a slight mathematical change to this specific feature. The pipeline architecture remains identical.

## Proposed Changes

### Optimized Pipeline Directory
We will create a new directory `optimized_pipeline` to keep your original files untouched.

#### [NEW] `optimized_pipeline/`
A copy of all `.ipynb` files will be placed here.

### Preprocessing Updates

#### [MODIFY] `1. preprocessing.ipynb`
- Wrap the entire logic (melting, prices, calendar, lags, mean encoding) in a `for store in STORES:` loop.
- Filter the raw `train_df` for the specific store *before* melting to drastically reduce memory usage.
- Update the save paths to include the store name (e.g., `grid_part_1_{store}.pkl`, `lags_df_28_{store}.pkl`).
- This will successfully generate all 50 `.pkl` files without crashing.

### Training & Prediction Updates

The data loading function `prepare_data(store)` in all model notebooks currently loads the massive 60M row `.pkl` files and then filters by store. We will update them to load the store-specific files directly.

#### [MODIFY] `2-1. nonrecursive_store_TRAIN.ipynb`
#### [MODIFY] `2-1. nonrecursive_store_PREDICT.ipynb`
- Update `prepare_data()` to load `grid_part_1_{store}.pkl` etc.

#### [MODIFY] `2-2. nonrecursive_store_cat_TRAIN.ipynb`
#### [MODIFY] `2-2. nonrecursive_store_cat_PREDICT.ipynb`
- Update `prepare_data()` to load store-specific pkls.

#### [MODIFY] `2-3. nonrecursive_store_dept_TRAIN.ipynb`
#### [MODIFY] `2-3. nonrecursive_store_dept_PREDICT.ipynb`
- Update `prepare_data()` to load store-specific pkls.

#### [MODIFY] `3-1. Final ensemble.ipynb`
- Update paths to read from the new pipeline's submission directories.

## Verification Plan

### Automated Tests
- N/A

### Manual Verification
- After updating the notebooks, you will be able to run `1. preprocessing.ipynb` locally to generate the 50 files.
- You can then run a training notebook for a single store (e.g., `CA_1`) to verify that the training starts successfully and memory usage remains low.
