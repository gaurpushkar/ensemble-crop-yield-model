# Satellite-Based Crop Yield Estimation (GP level)

Jupyter notebooks for estimating crop yield at the Gram Panchayat (GP) level across Indian districts,
using Sentinel-1/2 imagery, MODIS, and ERA5-Land weather from Google Earth Engine, a light-use-efficiency
(APAR) approach, and Random Forest regression calibrated against Crop Cutting Experiment (CCE) data.

## Coverage

| District | State | Crop | Season |
|---|---|---|---|
| Ahmednagar | Maharashtra | Sorghum (Jowar) | Rabi 2022-23 |
| Solapur | Maharashtra | Sorghum (Jowar) | Rabi 2022-23 |
| Belagavi | Karnataka | Sorghum (Jowar) | Rabi 2022-23 |
| Vijayapura | Karnataka | Sorghum (Jowar) | Rabi 2022-23 |
| Kamareddy | Telangana | Maize | Rabi 2022-23 |
| West Godavari (Eluru) | Andhra Pradesh | Maize | Rabi 2022-23 |
| Siddipet | Telangana | Maize | Rabi 2022-23 |
| Thoothukudi | Tamil Nadu | Maize | Rabi 2022-23 |
| Osmanabad | Maharashtra | Bengal gram | Rabi 2022-23 |
| Nuh | Haryana | Paddy (Rice) | 2023 |

## Documentation

| Doc | Contents |
|---|---|
| [docs/methodology.md](docs/methodology.md) | Growth stages, vegetation indices, APAR, the SPM model, Random Forest, accuracy metrics, limitations |
| [docs/notebooks.md](docs/notebooks.md) | Run order and what each notebook reads and writes |
| [docs/parameters.md](docs/parameters.md) | Settings per district: dates, crop-mask assets, RUE/HI, features, filter thresholds |
| [docs/data.md](docs/data.md) | Earth Engine datasets, required local inputs, directory layout, raster conventions |
| [docs/known-issues.md](docs/known-issues.md) | Bugs and caveats found during review, with suggested fixes |

Each notebook also starts with a header cell giving its purpose, inputs, outputs and position in the pipeline.

## Workflow

```
01_data_download ──► 02_raster_formation ──► 03_apar ──► 04_yield_modeling ──► 05_accuracy_assessment
   (GEE export)        (mosaic/stack)       (fAPAR×PAR)   (Random Forest)       (vs. CCE yields)
```

| Stage | Folder | What it does |
|---|---|---|
| 1 | `notebooks/01_data_download` | Exports Sentinel-2 SR (`COPERNICUS/S2_SR_HARMONIZED`) and Sentinel-1 GRD tiles from Earth Engine over district grids; computes NDVI, EVI, SAVI, NDWI. |
| 2 | `notebooks/02_raster_formation` | Unzips downloaded tiles, mosaics and stacks them into analysis-ready rasters with `rasterio`. |
| 3 | `notebooks/03_apar` | Builds APAR images from MODIS fAPAR (`MOD15A2H`), MODIS reflectance (`MOD09A1`) and ERA5-Land solar radiation (`ECMWF/ERA5_LAND/DAILY_AGGR`). |
| 4 | `notebooks/04_yield_modeling` | Extracts features at CCE points, trains `RandomForestRegressor`, and predicts a district yield raster. Also includes crop-specific notebooks for rice (SPM, Nuh), maize (Siddipet/Thoothukudi) and Bengal gram (Osmanabad). |
| 5 | `notebooks/05_accuracy_assessment` | Aggregates predicted yield to GP boundaries and compares with observed CCE yield (MAE, RMSE, R², MAPE). |
| – | `notebooks/exploratory` | EDA and feasibility testing. |

## Setup

```bash
conda create -n crop-yield -c conda-forge python=3.11 geopandas rasterio
conda activate crop-yield
pip install -r requirements.txt

# Authenticate Earth Engine once
earthengine authenticate
```

Recent versions of `earthengine-api` require a Cloud project:
`ee.Initialize(project="your-gcp-project-id")`.

To keep notebook outputs out of commits:

```bash
pip install pre-commit && pre-commit install
```

## Data

Input data (CCE yield records, GP boundary shapefiles, downloaded imagery) and outputs (yield rasters)
are **not** included in this repository. The notebooks currently reference absolute local Windows paths
(e.g. `C:\Local\...\YieldData\21Jan\<district>\`); update the path variables at the top of each notebook
to point to your own data directory before running.

Expected layout per district (as used in the notebooks):

```
<district>/
├── shape/        # sampling grid / point shapefiles
├── sentinel2/    # downloaded S2 tiles
├── apar/         # APAR rasters
└── output/       # predicted yield rasters (<District>_<crop>_yield.tif)
```

## Repository structure

```
.
├── docs/                    # methodology, notebook reference, parameters, data, known issues
├── notebooks/
│   ├── 01_data_download/
│   ├── 02_raster_formation/
│   ├── 03_apar/
│   ├── 04_yield_modeling/
│   ├── 05_accuracy_assessment/
│   └── exploratory/
├── requirements.txt
├── .pre-commit-config.yaml
└── .gitignore
```

## Status and caveats

This is research code. It is committed as it was used to produce the Rabi 2022-23 and Kharif 2023
yield maps. Accuracy figures produced by the notebooks are **in-sample** and computed **after outlier
filtering**, so treat them as optimistic. See [docs/known-issues.md](docs/known-issues.md) before
reusing the results or the code.
