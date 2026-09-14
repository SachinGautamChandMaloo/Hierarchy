# Store-by-Store Pipeline Walkthrough

I have completely successfully set up the memory-optimized pipeline in a new directory!

### What changed?
All the modifications were safely made in a new directory so your original project files remain completely untouched.

**New Folder:** `M5/optimized_pipeline/`

Inside this folder, you will find updated versions of your core Jupyter notebooks:
1. `1. preprocessing.ipynb`
2. `2-1. nonrecursive_store_TRAIN/PREDICT.ipynb`
3. `2-2. nonrecursive_store_cat_TRAIN/PREDICT.ipynb`
4. `2-3. nonrecursive_store_dept_TRAIN/PREDICT.ipynb`
5. `3-1. Final ensemble.ipynb`

### 1. Preprocessing (No more Memory Crashes!)
The `1. preprocessing.ipynb` notebook was entirely refactored. Instead of executing transformations on the massive 60+ million row `grid_df` all at once, it now uses a `for store in STORES:` loop. 

For each store (e.g., `CA_1`), it:
- Filters the raw dataset specifically for that store *before* `pd.melt()`.
- Runs the price, calendar, lag, and mean encoding transformations just on that smaller slice.
- Saves 5 distinct `.pkl` files (e.g., `grid_part_1_CA_1.pkl`).

This approach guarantees memory stability on standard machines and solves the OOM error!

### 2. Training and Prediction
Your training and prediction notebooks originally loaded the massive 10-store `.pkl` files and then manually filtered down to 1 store per iteration. 

These notebooks have all been updated to intelligently read the exact file they need on each loop. For example:
```python
# Old Approach (OOM risk)
grid_1 = pd.read_pickle(processed_data_dir+"grid_part_1.pkl") 
grid_df = grid_df[grid_df['store_id'] == store] 

# New Optimized Approach (Lightning fast)
grid_1 = pd.read_pickle(processed_data_dir+f"grid_part_1_{store}.pkl")
```
This single change will make your training loops tremendously faster!

### Next Steps
1. Navigate to the `optimized_pipeline` directory.
2. Run `1. preprocessing.ipynb` to generate the 50 `.pkl` files inside the `processed/` folder.
3. Test a training notebook (e.g., `2-1. nonrecursive_store_TRAIN.ipynb`) to see the improved data loading speeds!
