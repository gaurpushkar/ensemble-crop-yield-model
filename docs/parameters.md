# District parameters

These are the values currently hard-coded in each notebook. If you change a value, change it in every
stage that uses it: the growth-stage dates appear in `01`, `03` and `04`.

## Season, crop mask, model

| District | Crop | Growth-stage set¹ | Crop-mask asset (`projects/ee-gaurpushkar8/assets/…`) | Imagery | RF features |
|---|---|---|---|---|---|
| Ahmednagar | Sorghum | Rabi sorghum | `Ahmednagar_Rabi2022-23_Jowar_gcs` | S2 | S2 stage-4 VIs + `Yield1` |
| Solapur | Sorghum | Rabi sorghum | `Solapur_Rb2022-23_Jowar_gcs` | S2 | S2 stage-4 VIs + `Yield1` |
| Belagavi | Sorghum | Rabi sorghum | `Belagavi_Jowar_Rabi2022-23_gcs` | S2 | S2 stage-4 VIs + `Yield3` |
| Vijayapura | Sorghum | Rabi sorghum | `Vijayapura_Rb2022-23_Jowar_gcs` | S2 | S2 stage-4 VIs + `Yield1` |
| Kamareddy | Maize | Rabi maize | `Kamareddy_Maize_Rabi2022-23_gcs` | S2 | S2 stage-4 VIs + `Yield1` |
| West Godavari | Maize | Rabi maize | `west_godavari_gcs_maize` | S2 | S2 stage-4 VIs + `Yield1` |
| Thoothukudi / Siddipet | Maize | Rabi maize | `Thuthookudi_Maize_Rabi2022-23_gcs` | S2 | SPM only (+ zonal S2 stats at CCE points) |
| Osmanabad | Bengal gram | Rabi Bengal gram | `osmanabad_gcs` | S2 | SPM only (+ zonal S2 stats at CCE points) |
| Nuh | Paddy | Kharif paddy | `nuh_paddy_2023` | S1 (VV) | `BS_1..4` + `Yield` |

¹ The dates for each set are in [methodology.md § 1](methodology.md#1-crop-season-and-growth-stages).
"S2 stage-4 VIs" means `EVI4, NDPI4, NDVI4, SAVI4, NDWI4, NDTI4`.

## SPM coefficients

In the table below, "set" means the value is declared as a constant but not applied in that notebook.

| Where | RUE | HI | Notes |
|---|---|---|---|
| `03_apar/apar_*.ipynb` (all) | 2.8 | 0.6 | Set but not applied; APAR is exported raw |
| `yield_ml_ahmednagar` | 0.26 | 4.25 | Passed as `spm_processing(apar, 4.25, 0.26)`, product = 1.105 |
| `yield_ml_*` (all other districts) | 0.6 | 2.8 | Passed as `spm_processing(apar, 2.8, 0.6)`, product = 1.68 |
| `maize_yield_siddipet_thoothukudi` | 4.65 | 0.26 | Tmax = 34 |
| `bengal_gram_yield_osmanabad` | 0.93 | 0.38 | Also `yield_gen()` = ΣAPAR × 0.93 × 0.38 |
| `rice_yield_spm_nuh` | 2.8 | 0.6 | |

`spm_processing(imageList, hi, rue)` takes HI first, so the district calls pass the RUE-like number in
the HI slot. The yield is a plain product, so the result is unaffected. It only matters if the function
changes.

## Temperature stress

| Parameter | Value |
|---|---|
| Tmin | 5 °C |
| Topt | 27 °C |
| Tmax | 33 °C (34 °C for maize) |

## Random Forest

| Setting | Value |
|---|---|
| `n_estimators` | 1000 |
| `criterion` | `friedman_mse` (`mse` in `exploratory/eda.ipynb`) |
| `random_state` | 123 |
| Train/test split | 80/20 is created but not used: the model is fitted on all points |

## Filtering thresholds

| District | Point filter in `04` (\|pred − obs\| <) | GP filter in `05` (MAPE <) |
|---|---|---|
| Ahmednagar | 500 kg/ha | 50 % |
| Vijayapura | 500 kg/ha | 60 % |
| Belagavi, Kamareddy, West Godavari, Nuh | 2000 kg/ha | Kamareddy 50 %, Nuh 60 %, others none |
| Solapur | none | none |
| Siddipet / Osmanabad (`accuracy_siddipet_osmanabad`) | — | 50 % |
