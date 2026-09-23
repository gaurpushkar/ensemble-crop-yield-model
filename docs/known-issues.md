# Known issues

These issues were found while documenting the code. None of them has been fixed yet: the notebooks
are committed exactly as they were used to produce the results. Fixing any of these will change the
numbers.

## Affects results

| # | Issue | Where | Effect | Suggested fix |
|---|---|---|---|---|
| 1 | **The model is evaluated on its own training data.** `train_test_split` runs, but `regressor.fit(X, y)` uses every point, and the metrics are computed on those same points. | `04_yield_modeling/yield_ml_*` | Accuracy (R², RMSE, MAPE) is optimistic | Fit on `X_train, y_train` and report metrics on `X_test`, or use k-fold CV / leave-one-GP-out |
| 2 | **Points are filtered on prediction error before metrics.** Points where \|pred − obs\| ≥ 500/2000 kg/ha are dropped, and GPs with MAPE ≥ 50–60 % are dropped. | `04` and `05` | Also inflates accuracy | Report metrics before and after filtering |
| 3 | **SAVI uses L = 7.5 instead of 0.5.** The code has `(B8 + B4 + 7.5)` as the denominator. | `01_data_download/s2_download_*` (all S2) | SAVI is scaled down and close to linear in (B8 − B4). It still works as an RF feature but is not true SAVI. | `b8.add(b4).add(0.5)` |
| 4 | **Sentinel-2 has no cloud mask.** Stage composites are `.mean()` over all scenes. | `01_data_download/s2_download_*` | Clouds and shadow bias the indices low | Filter on `CLOUDY_PIXEL_PERCENTAGE` and mask with the SCL band or `s2cloudless` |
| 5 | **The index of agreement formula is wrong.** The code uses `(|P − Ō| + |O|)²`, but Willmott's d uses `(|P − Ō| + |O − Ō|)²`. | `accuracy_all_districts*.ipynb` | IoA is overstated | `abs(y_pred[d]-muo) + abs(y_true[d]-muo)` |
| 6 | **The SPM output name doesn't match the RF input.** `spm_processing()` writes `Yield.tif`, but the RF reads `Yield1.tif` (`Yield3.tif` in Belagavi). | `yield_ml_*` | The pipeline fails unless the file is renamed by hand | Write `Yield1.tif` directly, or read `Yield.tif` |
| 7 | **NoData is guessed from the raster minimum.** It is `x.min()` or a hard-coded value (`683.493286` in Ahmednagar), not the file's `nodata` tag. The prediction step also fills NaN with 0 and predicts on background pixels. | `yield_ml_*` | Fragile: a real minimum pixel gets masked, and each rerun needs a new constant | Use `src.nodata` / masked arrays and predict only on valid pixels |

## Breaks on current library versions

| # | Issue | Where | Fix |
|---|---|---|---|
| 8 | `mean_squared_error(..., squared=False)` was removed in scikit-learn 1.6 | All of `04` and `05`, and `eda` | Use `root_mean_squared_error()`, or keep `scikit-learn<1.6` (pinned in `requirements.txt`) |
| 9 | `RandomForestRegressor(criterion='mse')` was removed in scikit-learn 1.2 | `exploratory/eda.ipynb` | `criterion='squared_error'` |
| 10 | `ee.Initialize()` with no project fails on newer `earthengine-api` | All GEE notebooks | `ee.Initialize(project='<gcp-project>')` |
| 11 | Hard-coded Windows paths, split by backslash (`filename.split('\\')`) | All | Move to one `DATA_DIR` with `pathlib` |

## Housekeeping

- **Invalid unused date.** `g3 = '2023-30-01'` is not a valid date, in `03_apar/apar_{ahmednagar,belagavi,solapur,vijayapura}`. It is only referenced in commented-out code, so it has no effect today, but it will fail if that code is re-enabled. It should be `'2023-01-30'`.
- **Leftover Osmanabad code in the Nuh notebook.** `04_yield_modeling/rice_yield_spm_nuh.ipynb` contains cells copied from the Osmanabad Bengal gram notebook (cells 22 onwards: Osmanabad paths, the `0.93 × 0.38` Bengal gram coefficients, and Osmanabad CCE zonal stats).
- **Broken, unused function.** `problem_gp_download()` in `01_data_download/*` refers to an undefined `problem_gp`. It is never called.
- **Swapped argument order.** `spm_processing(imageList, hi, rue)` is called with the RUE and HI values swapped. The output is unaffected (it's a product), but the naming is misleading.
- **Duplicated code.** Every district is a copy of the same notebooks with different constants. A shared `yieldlib.py` plus one config per district would remove about 90 % of the duplication.
