# Data requirements

No data is stored in this repository. This page lists what each stage needs and what it writes.

## External sources (via Google Earth Engine)

| Dataset | Earth Engine ID | Used for | Resolution |
|---|---|---|---|
| Sentinel-2 L2A (harmonized) | `COPERNICUS/S2_SR_HARMONIZED` | Vegetation indices | 10–20 m |
| Sentinel-1 GRD | `COPERNICUS/S1_GRD` | VV backscatter (Nuh) | 10 m |
| MODIS LAI/FPAR | `MODIS/061/MOD15A2H` (`Fpar_500m`) | fAPAR | 500 m, 8-day |
| MODIS surface reflectance | `MODIS/061/MOD09A1` | NDWI for water stress | 500 m, 8-day |
| ERA5-Land daily | `ECMWF/ERA5_LAND/DAILY_AGGR` | Solar radiation, 2 m temperature | ~11 km, daily |

You need an Earth Engine account with access to the crop-mask assets under
`projects/ee-gaurpushkar8/assets/`. Anyone else needs those assets shared with them, or has to upload
their own crop mask and change the asset ID.

## Local inputs (per district)

| Input | Format | Required fields | Used in |
|---|---|---|---|
| Sampling grid / GP split | Shapefile (EPSG:4326) | `Id` (some districts: `FID_1`) | 01, 03 |
| CCE yield records | `.xlsx` or `.csv` | `Longitude`, `Latitude`, `Yield (kg_p_ha)`² | 04, 05 |
| GP boundaries | Shapefile | `GP_Name` | 05 |

² Nuh uses `Yield(kg_p_ha)` with no space, and Siddipet uses lowercase `longitude`/`latitude`.

The per-GP download loops iterate over grid features by their `Id`. For large districts the notebooks
use a finer grid for Sentinel-2 (`*_pt1*.shp`) and a coarser one for APAR (`*_pt25*.shp`). This keeps
each `ee_export_image` request under Earth Engine's direct-download size limit.

## Directory layout

The notebooks expect this layout under one base folder per district:

```
<district>/
├── shape/                 # grids: <district>_pt1*.shp (S2), <district>_pt25*.shp (APAR)
├── cce/                   # CCE yield table (xlsx/csv)
├── sentinel2/             # 01 output: <INDEX><stage>_<gridId>.tif      e.g. NDVI4_17.tif
├── apar/                  # 03 output: APAR-<YYYY_MM_DD>-<gridId>.tif
└── output/                # 02 mosaics + 04 model outputs
    ├── NDVI1.tif … EVI4.tif          # mosaicked index per stage
    ├── APAR-<YYYY_MM_DD>.tif         # mosaicked APAR per date
    ├── Sum_apar_1..5.tif             # stage and season ΣAPAR
    ├── Yield.tif                     # SPM yield proxy (rename to Yield1.tif, see known-issues)
    ├── Yield_predicted_new.tif       # raw RF prediction
    ├── <District>_<crop>_yield.tif   # final yield map (kg/ha)
    ├── Regplot.png                   # observed vs. predicted
    └── <District>_<crop>_yield.xlsx  # points with predictions
```

For Nuh, the data comes from `Processing/s1` (backscatter) and `Processing/raw` (APAR), and the
mosaics are written to `Processing/Rasters/output`.

## Raster conventions

- **Scaling:** Earth Engine exports are multiplied by 1000 and cast to `int16`. Divide by 1000 to get
  the physical value back (`apar_processing` does this).
- **NoData:** masked pixels arrive as the minimum value of the raster. The code treats `x.min()` as
  NoData rather than using the file's `nodata` tag.
- **CRS:** everything is on the crop-mask projection (geographic, "gcs").

## Outputs

| File | Stage | Description |
|---|---|---|
| `<District>_<crop>_yield.tif` | 04 | Final 10 m yield map, kg/ha, float32, LZW compressed |
| `<District>_<crop>_yield.xlsx` | 04 | CCE points with observed and predicted yield |
| `<District>_GP*.xlsx` | 05 | GP-level MAE / RMSE / R / MAPE / observed and predicted means |
| `AllDistricts.xlsx` | 05 | District-level summary including t-test and index of agreement |
