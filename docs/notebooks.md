# Notebook reference

## Run order for one district

The example below is for a Sentinel-2 district. Replace `<d>` with `ahmednagar`, `solapur`, `belagavi`,
`vijayapura`, `kamareddy` or `west_godavari`.

```
01_data_download/s2_download_<d>.ipynb        # Earth Engine → sentinel2/*.tif   (slow: one export per grid cell × 6 indices × 4 stages)
03_apar/apar_<d>.ipynb                         # Earth Engine → apar/*.tif        (independent of 01, can run in parallel)
02_raster_formation/raster_formation_<d>.ipynb # sentinel2/ + apar/ → output/ mosaics
04_yield_modeling/yield_ml_<d>.ipynb           # SPM + Random Forest → output/<District>_<crop>_yield.tif
05_accuracy_assessment/accuracy_<d>.ipynb      # GP-level metrics
05_accuracy_assessment/accuracy_all_districts.ipynb   # after all districts
```

**Nuh (paddy)** follows the same order with Sentinel-1:
`s1_download_nuh` → `rice_yield_spm_nuh` (APAR/SPM export) → `raster_formation_nuh` → `yield_ml_nuh` → `accuracy_nuh`.

Before running a notebook, edit its path variables (`base`, `BASE_FOLDER`, `shapeFile`, `out_path`).
They are in the first few cells.

---

## 01 — Data download (Earth Engine)

| Notebook | District / crop | Output |
|---|---|---|
| `s2_download_ahmednagar` | Ahmednagar, sorghum | `sentinel2/{NDVI,NDWI,EVI,SAVI,NDTI,ndpi}{1-4}_{n}.tif` |
| `s2_download_solapur` | Solapur, sorghum | ″ |
| `s2_download_belagavi` | Belagavi, sorghum | ″ |
| `s2_download_vijayapura` | Vijayapura, sorghum | ″ |
| `s2_download_kamareddy` | Kamareddy, maize | ″ |
| `s2_download_west_godavari` | West Godavari (Eluru), maize | ″ |
| `s2_download_thoothukudi_siddipet` | Thoothukudi (grid) + Siddipet (CCE), maize | `splits/s2data/…` |
| `s1_download_nuh` | Nuh, paddy | `Processing/s1/BS_{1-4}_{n}.tif` (VV backscatter) |

Main functions:

- `stage_vi(collection, stage, n, geom)`: builds the mean composite, computes the 6 indices, masks and
  reprojects them to the crop mask, and exports each one.
- `image_collection_filter(geom, n)`: splits the collection into the 4 stages and calls `stage_vi` on each.
- `image_download(img, fn, geom)`: scales by 1000, casts to int16 and runs `geemap.ee_export_image`.
  It skips files that already exist, so an interrupted run can simply be re-run.

## 02 — Raster formation

`raster_formation_<d>` mosaics the per-grid-cell tiles into one raster per variable:

- `APAR-<date>.tif` for every APAR date
- `<INDEX><stage>.tif` for every index and stage (for Nuh: `BS_<stage>.tif`)

The main function is `mosaic_image(file_list, name)`, a `rasterio.merge.merge` that writes to `output/`.

## 03 — APAR

`apar_<d>` does the following:

1. Scales MOD15A2H fAPAR.
2. Sums ERA5-Land radiation over each 8-day MODIS period and multiplies it by 0.48 to get PAR.
3. Computes APAR = fAPAR × PAR.
4. Masks APAR to the crop and exports it per grid cell as `apar/APAR-<YYYY_MM_DD>-<n>.tif`.

The water-stress and temperature-stress exports are present but commented out.

| Notebook | District |
|---|---|
| `apar_ahmednagar`, `apar_solapur`, `apar_belagavi`, `apar_vijayapura` | Sorghum districts |
| `apar_kamareddy`, `apar_west_godavari` | Maize districts |

## 04 — Yield modelling

| Notebook | What it does |
|---|---|
| `yield_ml_<d>` (7 districts) | Sums APAR by stage and season, builds the SPM yield proxy, samples features at CCE points, trains a Random Forest, predicts the full raster, cleans NoData, plots observed vs. predicted and computes metrics. |
| `rice_yield_spm_nuh` | Nuh paddy: APAR, temperature stress and water stress exported per GP from Earth Engine, then a stage-binned SPM yield (`Yield_<n>.tif`). Feeds `raster_formation_nuh` and `yield_ml_nuh`. |
| `maize_yield_siddipet_thoothukudi` | Maize APAR/SPM computed in Earth Engine (RUE 4.65, HI 0.26), plus Sentinel-2 band zonal means at the Siddipet CCE points (`Sentinel_cce.csv`). |
| `bengal_gram_yield_osmanabad` | Bengal gram APAR/SPM (RUE 0.93, HI 0.38), a local ΣAPAR → yield step (`yield_gen`), and Sentinel-2 zonal stats at the CCE points. |

Outputs of `yield_ml_<d>`, in `output/`:

- `Sum_apar_{1..5}.tif`
- `Yield.tif`
- `Yield_predicted_new.tif`
- `<District>_<crop>_yield*.tif`
- `Regplot.png`
- `<District>_<crop>_yield.xlsx`

## 05 — Accuracy assessment

| Notebook | Scope |
|---|---|
| `accuracy_<d>` | One district: point-in-GP join, then metrics for each GP |
| `accuracy_thoothukudi` | Thoothukudi maize (reads `Thoothukudi Updated.xlsx`) |
| `accuracy_siddipet_osmanabad` | Siddipet maize and Osmanabad Bengal gram |
| `accuracy_all_districts` | District-level roll-up of every `*.xlsx` in `alldistricts/`: MAE, RMSE, R, MAPE, t-test, index of agreement → `AllDistricts.xlsx` |
| `accuracy_all_districts_v2` | Variant of the above that reads `Observed Yield` / `Predicted Yield` columns (for the Thoothukudi workbook format) |

The main function is `accuracy_parameter(df)`. It returns
`(MAE, MSE, RMSE, R, MAPE, mean_obs, mean_pred)`, and the all-districts version also returns the
t-test and IoA.

## Exploratory

| Notebook | Purpose |
|---|---|
| `eda` | Siddipet: reshapes Earth Engine zonal-stats output (bands × dates) into growth-stage VIs, draws a correlation heatmap, and compares Linear Regression with Random Forest on CCE plot weight. |
| `feasibility_testing_nuh` | Nuh: intersects the CCE 2023 points with the crop layer to keep only valid plots → `CCE_2023_intersect.xlsx`. |
