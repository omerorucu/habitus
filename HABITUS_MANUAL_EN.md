# HABITUS — Comprehensive User and Developer Manual

**Version:** v1.0.0  
**Date:** April 2026  
**Platform:** Windows 10/11, standalone PyQt6 desktop application

---

## Table of Contents

1. [Overview](#1-overview)
2. [Architecture and File Structure](#2-architecture-and-file-structure)
3. [Tab ① — Data Loading (Data)](#3-tab--data-loading-data)
4. [Tab ② — Variable Selection (VIF + Correlation)](#4-tab--variable-selection-vif--correlation)
5. [Tab ③ — Model Training + Current Distribution Map](#5-tab--model-training--current-distribution-map)
6. [Tab ④ — Future Scenarios (Projection)](#6-tab--future-scenarios-projection)
7. [Tab ⑤ — Range Change](#7-tab--range-change)
8. [Tab ⑥ — Evaluation](#8-tab--evaluation)
9. [Tab ⑦ — Validation](#9-tab--validation)
10. [Tab ⑧ — Report](#10-tab--report)
11. [Map Viewer (map_widget.py)](#11-map-viewer-map_widgetpy)
12. [SDM Core (sdm_core.py)](#12-sdm-core-sdm_corepy)
13. [Output Files](#13-output-files)
14. [New Features — v1.0.0](#14-new-features--v100)
15. [Build and Distribution](#15-build-and-distribution)
16. [Frequently Asked Questions and Troubleshooting](#16-frequently-asked-questions-and-troubleshooting)

---

## 1. Overview

HABITUS (**H**abitat **A**nalysis and **B**iodiversity **I**ntegrated **T**oolkit for **U**nified **S**DM) is a standalone desktop application developed for modelling the geographic distribution of species. It does not require QGIS; it has its own PyQt6-based interface.

### Supported Algorithms (13)

| Abbreviation | Full Name | Type |
|--------------|-----------|------|
| GLM | Generalized Linear Model | Statistical |
| GAM | Generalized Additive Model | Statistical |
| GBM | Gradient Boosting Machine | Ensemble |
| BRT | Boosted Regression Trees | Ensemble |
| RF | Random Forest | Ensemble |
| SVM | Support Vector Machine | Kernel |
| ANN | Artificial Neural Network | Deep learning |
| XGB | XGBoost | Gradient boosting |
| LGB | LightGBM | Gradient boosting |
| CAT | CatBoost | Gradient boosting |
| MaxEnt | Maximum Entropy | Presence-only |
| ENFA | Ecological Niche Factor Analysis | Mean/variance |
| Mahalanobis | Mahalanobis Distance | Distance-based |

### Workflow (8 steps)

```
① Data → ② Variable Selection → ③ Model → ④ Future → ⑤ Range Change → ⑥ Evaluation → ⑦ Validation → ⑧ Report
```

---

## 2. Architecture and File Structure

```
habitus/
├── main.py                   # Entry point: PROJ fix, QApplication launch
├── main_dialog.py            # Main window, tabs, log, update check
├── sdm_core.py               # All SDM logic (2500+ lines)
├── map_widget.py             # Embedded raster map viewer
├── version.py                # APP_VERSION, GITHUB_REPO constants
├── updater.py                # GitHub release checker (QThread)
├── tabs/
│   ├── tab_data.py           # ① Data loading
│   ├── tab_vif.py            # ② Variable selection — VIF + correlation
│   ├── tab_vif_advanced.py   # ② Sub-tab — advanced analysis
│   ├── tab_models.py         # ③ Model training + current map
│   ├── tab_ensemble.py       # ③ Ensemble combination
│   ├── tab_projection.py     # ④ Future projection
│   ├── tab_range.py          # ⑤ Range change
│   ├── tab_evaluation.py     # ⑥ Evaluation charts
│   ├── tab_validation.py     # ⑦ Validation
│   ├── tab_report.py         # ⑧ Scientific HTML report generator
│   └── tab_help.py           # Help + About (bilingual TR/EN)
├── habitus.spec              # PyInstaller spec file
├── habitus_setup.iss         # Inno Setup installer script
└── requirements.txt          # Python dependencies
```

### Technology Stack

| Component | Library | Purpose |
|-----------|---------|---------|
| UI | PyQt6 | Desktop interface |
| Maps | matplotlib (QtAgg) + rasterio | GeoTIFF display |
| ML | scikit-learn, xgboost, lightgbm, catboost, pygam, elapid | Model training |
| Geospatial | pyproj, geopandas, shapely | Coordinate transformation |
| Build | PyInstaller + Inno Setup | .exe generation |

### Data Flow

```
CSV (occurrence) + Raster (environmental)
    → DataFormatter (normalize, generate PA)
        → SDMModeler.run() (CV training)
            → ensemble (EMwmean / EMca)
                → GeoTIFF output
                    → EvaluationTab (charts)
```

### Color Theme

| Variable | Hex Code | Usage |
|----------|----------|-------|
| Background | `#f0f5f1` | Main window |
| Accent | `#3a8c60` | Headers, borders |
| Dark text | `#1c3328` | Labels |
| Warning | `#f6ad55` | Medium performance |
| Error | `#fc8181` | Low performance |
| Success | `#52b788` | Good performance |

---

## 3. Tab ① — Data Loading (Data)

**File:** `tabs/tab_data.py`

### 3.1 Input Files

#### Occurrence Data (CSV)
- **Format:** Columns: `species`, `longitude`, `latitude` (these names or the first 3 columns)
- **Coordinate system:** WGS 84 (EPSG:4326) decimal degrees
- **Minimum rows:** ~30 occurrences per model is recommended
- **Duplicate coordinates:** Automatically removed (spatial thinning)

#### Environmental Rasters (GeoTIFF)
- **Format:** `.tif`, `.tiff` — single band, float32/float64
- **CRS:** All rasters must share the same coordinate system (WGS 84 recommended)
- **Extent:** Must be large enough to cover all occurrence points
- **Resolution:** All rasters must have the same pixel size (verified with rasterio)
- **NoData:** -9999, NaN, or any NoData value recognized by rasterio

#### Categorical Rasters (optional)
- Discrete variables such as soil type, land cover
- Must be flagged as `Categorical` in the interface
- Encoding: `one-hot` (default), `label`, `target` options (`tab_data.py` / `sdm_core.py::DataFormatter`)

### 3.2 Parameters

| Parameter | Default | Description |
|-----------|---------|-------------|
| Working directory | — | Folder where outputs will be saved |
| Species name | Auto from CSV | Used as file name prefix |
| Categorical encoding | `one-hot` | Categorical raster encoding method |

### 3.3 Output (state)

`dlg.state["formatter"]` → `DataFormatter` object  
`dlg.state["pa_datasets"]` → `[(X_df, y_arr, coords_dict), ...]` list  
`dlg.state["var_names"]` → selected variable names  
`dlg.state["output_dir"]` → working directory

---

## 4. Tab ② — Variable Selection (VIF + Correlation)

**File:** `tabs/tab_vif.py`, `tabs/tab_vif_advanced.py`

### 4.1 VIF + Correlation Sub-Tab

#### Analysis Settings

| Parameter | Default | Range | Description |
|-----------|---------|-------|-------------|
| Correlation threshold (|r| ≥) | 0.80 | 0.30–0.99 | Pairs above this value are considered "highly correlated". Dormann et al. (2013) recommend \|r\| < 0.70 for SDM |
| Correlation method | pearson | pearson / spearman | Pearson: linear; Spearman: rank-based, does not assume normality |
| VIF threshold (informational) | 10 | 2–50 | VIF > 10 indicates severe multicollinearity (O'Brien 2007). Does not force exclusion; used for color coding only |

#### High Correlation Pairs Panel (Left)
- All pairwise combinations above the threshold are listed in descending \|r\| order
- Columns: **Variable A**, **Variable B**, **\|r\|**, **±** (direction)
- Color coding: the variable with the lower AUC is highlighted in red → recommended for removal
- **Click on row:** Both variables are highlighted in the right panel
- **Uncheck Variable A / B:** Deselects one variable from the selected pair

#### No High Correlation Banner (Green box)
- Variables not involved in any high-correlation pair are listed in a green banner
- These variables are automatically selected
- Stored in the `self._safe_vars` set

#### Variable Selection List (Right)
For each variable:
- **Checkbox:** Include/exclude selection
- **VIF badge:** Value + color code (red > threshold, green ≤ threshold)
- **AUC badge:** Single-variable ROC-AUC score
- **Status icon:** ✔ (recommended), ⚠ VIF (high VIF), ⚠ Corr (high correlation), – (other)

**Bulk action buttons:**
- **All:** Select all variables
- **Recommended:** Select only recommended variables (VIF OK + Corr OK)
- **None:** Deselect all

#### Correlation Heatmap
- Pearson/Spearman correlation matrix
- Color: red (negative) → white (0) → green (positive) — RdYlGn color scale
- Orange border: cells above the threshold

### 4.2 Advanced Analysis Sub-Tab (`tab_vif_advanced.py`)

- **PCA Biplot:** Shows relationships among variables and sample distribution
- **Condition Number:** Matrix condition number (κ < 30: good, 30–100: moderate, > 100: severe)
- **Partial Correlation:** Pairwise correlation while controlling for other variables
- **VIF Bar Chart:** VIF value for each variable, with threshold line

### 4.3 Output Files

| File | Description |
|------|-------------|
| `figures/correlation_heatmap.png` | Correlation matrix for all variables |
| `figures/correlation_heatmap_selected.png` | Correlation matrix for selected variables only |
| `figures/vif_barplot.png` | VIF bar chart |
| `figures/high_correlation_pairs.csv` | Table of high-correlation pairs |
| `figures/variable_priority_ranking.csv` | Priority ranking for all variables |

---

## 5. Tab ③ — Model Training + Current Distribution Map

**File:** `tabs/tab_models.py`, `tabs/tab_ensemble.py`

### 5.1 Algorithm Settings

#### General Parameters

| Parameter | Default | Description |
|-----------|---------|-------------|
| CV runs (n_cv_runs) | 2 | Number of cross-validation repetitions. Each repetition uses a different random split. More repetitions = more reliable evaluation, longer runtime |
| Data split % (data_split) | 80 | Training / test split ratio. 80 = 80% of data used for training, 20% for testing |
| PA repetitions (n_pa_rep) | 2 | Number of pseudo-absence repetitions. Each repetition samples different random pseudo-absences. Multiple repetitions reduce variance |
| PA count (n_absences) | 1000 | Number of pseudo-absences to generate per occurrence for ML models. Barbet-Massin et al. (2012): recommend equal to or 10× the number of presences |
| PA strategy | random | `random`: uniform random sampling; `disk`: sampling at a minimum distance (km) from occurrence points; `sre`: sampling outside the bioclimatic envelope |
| Variable importance reps | 3 | Number of repetitions for permutation-based variable importance calculation |
| Threshold method | max_tss | Threshold determination method for binary map (see §11.3) |

#### Algorithm Hyperparameters

**GBM / XGB / LGB / CAT (Gradient Boosting)**

| Parameter | GBM Default | BRT Default | XGB Default | LGB Default | CAT Default | Description |
|-----------|------------|------------|------------|------------|------------|-------------|
| n_estimators | 500 | 1000 | 500 | 500 | 500 | Number of trees. More → slower but better |
| max_depth | 3 | 5 | 6 | -1 (unlimited) | 6 | Maximum depth of each tree. Deeper = more complex model |
| learning_rate | 0.05 | 0.01 | 0.05 | 0.05 | 0.05 | Learning rate. Low + more trees = better |

> **BRT Note:** Following Elith et al. (2008) recommendations: `tc=5` (max_depth=5), `lr=0.01`, `bag.fraction=0.75` (subsample=0.75)

**RF (Random Forest)**

| Parameter | Default | Description |
|-----------|---------|-------------|
| n_estimators | 500 | Number of trees. Oshiro et al. (2012): 64–128 is sufficient, but 500 is recommended for SDM |
| max_depth | None (unlimited) | Leaving unlimited is standard for RF |

**ANN (Artificial Neural Network)**

| Parameter | Default | Description |
|-----------|---------|-------------|
| hidden_layer_sizes | (100, 50) | Number of neurons per hidden layer. (100, 50) = 2 layers, 1st layer 100 neurons, 2nd layer 50 neurons |
| max_iter | 2000 | Maximum number of epochs |
| StandardScaler | Yes | Data is automatically normalized for ANN (within Pipeline) |

**MaxEnt**

| Parameter | Default | Description |
|-----------|---------|-------------|
| Background points (n_background) | 10000 | Number of background points. 10000 is standard for Phillips & Dudík (2008) |
| Background strategy (mx_bg_strategy) | random | `random`: uniform random; `target`: target group approach |
| Regularization multiplier (beta) | 1.0 | Prevents overfitting. Larger β → smoother model |
| Feature types | lqph | linear, quadratic, product, hinge |

**SVM (Support Vector Machine)**

| Parameter | Default | Description |
|-----------|---------|-------------|
| kernel | rbf | Radial basis function kernel |
| C | 1.0 | Regularization parameter. Larger → less regularization |
| probability | True | Required for `predict_proba()` |

**GLM**

- Uses scikit-learn `LogisticRegression`
- `solver='lbfgs'`, `max_iter=1000`, `C=1.0`
- Assumes linear effects on continuous variables

**GAM**

- Uses `pygam.LogisticGAM`
- Spline term (`s(xi)`) for each variable
- Can capture non-linear relationships

**ENFA (Ecological Niche Factor Analysis)**

- PCA-based, uses only occurrence data
- `Marginality`: difference between mean habitat and the environment preferred by the species
- `Specialization`: habitat variance
- Implemented with scikit-learn PCA

**Mahalanobis Distance**

- Uses the covariance matrix of occurrence points in environmental space
- Distance → converted to 1 - suitability

### 5.2 Minimum Sample Size Requirements

The following guidelines are derived from the consensus of the core SDM sample size literature: Wisz et al. (2008) tested algorithm performance at 10, 30, and 100 presence records; Hernandez et al. (2006) compared algorithms under small-sample conditions; van Proosdij et al. (2016) derived algorithm-specific lower thresholds. These are **presence record** counts (not pseudo-absence or background points).

| Algorithm | Minimum | Recommended | Ideal | Data Sensitivity | Notes |
|-----------|---------|-------------|-------|-----------------|-------|
| **MaxEnt** | 10–15 | 30–50 | 100+ | ★ Lowest | Best-performing algorithm under data-poor conditions (Hernandez et al. 2006; Phillips et al. 2006) |
| **GAM** | 50–80 | 100–150 | 200+ | Medium | Requires sufficient variation to fit spline terms; unstable below 50 records |
| **GLM** | 30–50 | 80–100 | 150+ | Low | Parsimonious model, relatively stable with fewer data |
| **ENFA** | 20–30 | 50–100 | 200+ | Low–Medium | Presence-only; niche marginality/specialization require variability |
| **Mahalanobis** | 15–25 | 40–80 | 150+ | Low | Distance-based; robust to small samples but sensitive to multicollinearity |
| **SVM** | 30–50 | 80–150 | 300+ | Medium | Kernel generalisation degrades with very few positives |
| **BRT** | 50 | 100–200 | 300+ | High | Learning rate adjustment partially compensates for small samples (Elith et al. 2008) |
| **RF** | 50 | 100–200 | 500+ | High | Ensemble of trees benefits strongly from larger datasets (Breiman 2001) |
| **GBM** | 50 | 100–200 | 500+ | High | Similar to BRT; requires enough data to avoid overfitting |
| **ANN** | 50–80 | 150–200 | 500+ | High | Hidden-layer weights need sufficient observations; StandardScaler is applied automatically |
| **XGBoost** | 100 | 200–300 | 500+ | Very High | Prone to overfitting on small datasets; regularisation (lambda, alpha) helps |
| **LightGBM** | 100 | 200–300 | 500+ | Very High | Leaf-wise growth → high risk of overfitting below 100 presence records |
| **CatBoost** | 100 | 200–300 | 500+ | Very High | Ordered boosting reduces overfitting but still requires adequate sample size |

**Key references:**
- Wisz M.S. et al. (2008) Effects of sample size on the performance of species distribution models. *Diversity and Distributions* 14:763–773
- Hernandez P.A. et al. (2006) The effect of sample size and species characteristics on performance of different species distribution modeling methods. *Ecography* 29:773–785
- van Proosdij A.S.J. et al. (2016) Minimum required number of specimen records to develop accurate species distribution models. *Ecography* 39:542–552
- Elith J. et al. (2008) A working guide to boosted regression trees. *Journal of Animal Ecology* 77:802–813
- Phillips S.J. et al. (2006) Maximum entropy modeling of species geographic distributions. *Ecological Modelling* 190:231–259
- Breiman L. (2001) Random forests. *Machine Learning* 45:5–32

> **Practical guidance:** For studies with fewer than 30 presence records, restrict algorithm selection to MaxEnt, ENFA, Mahalanobis, and GLM. For 30–100 records, GAM, BRT, SVM, and RF are viable with conservative hyperparameters. Gradient boosting algorithms (XGB, LGB, CAT) should be used only with ≥ 100 well-spatially-thinned records.

### 5.3 Pseudo-Absence Strategies

| Strategy | Code | Description |
|----------|------|-------------|
| Random | `random` | Uniform random sampling within the study area (Barbet-Massin et al. 2012) |
| Disk | `disk` | Sampling at a minimum distance (km) from occurrence points |
| SRE | `sre` | Sampling outside the percentile envelope of all variables. Targets areas outside the bioclimatic envelope |

### 5.4 Ensemble Settings

**File:** `tabs/tab_ensemble.py`

| Parameter | Default | Description |
|-----------|---------|-------------|
| Quality threshold (TSS) | 0.70 | Models below this value are excluded from the ensemble. Zero → all models included |
| Committee Averaging (EMca) | ✓ | Average of binary maps → occurrence probability |
| Weighted Mean (EMwmean) | ✓ | Probability average weighted by TSS |

### 5.5 Map Labels

Raw key → readable label conversion (`_make_current_label` function):

| Raw Key | Displayed Label |
|---------|----------------|
| `Maxent_PA1_prob` | `Current Distribution · Maxent · PA-1 · Probability` |
| `RF_PA2_bin` | `Current Distribution · RF · PA-2 · Binary` |
| `EMwmean_prob` | `Current Distribution · Ensemble: Weighted Mean · Probability` |
| `EMca_bin` | `Current Distribution · Ensemble: Committee Averaging · Binary` |

### 5.6 Train/Test Point Layers

The `_add_traintest_layers(modeler)` function automatically adds point layers to the map:

| Layer | Color | Size | ptype |
|-------|-------|------|-------|
| Train presences | `#27ae60` (green) | 30 | `obs` |
| Test presences | `#f39c12` (orange) | 30 | `obs` |
| Train pseudo-absences | `#e74c3c` (red) | 8 | `bg` |

---

## 6. Tab ④ — Future Scenarios (Projection)

**File:** `tabs/tab_projection.py`

### 6.1 Scenario Settings

| Field | Example | Description |
|-------|---------|-------------|
| Scenario name | `2050_SSP245` | Output folder name and label component. Free text |
| GCM model | `CNRM-ESM2-1` | Global climate model name. Added to map label. Can be left blank |

### 6.2 Map Labels

`_make_future_label(key, scenario, gcm)` function:

| Example Case | Displayed Label |
|-------------|----------------|
| `EMwmean_prob`, scenario=`2050_SSP245`, gcm=`CNRM-ESM2-1` | `Future Distribution · CNRM-ESM2-1 · 2050_SSP245 · Ensemble: Weighted Mean · Probability` |
| `Maxent_PA1_prob`, gcm blank | `Future Distribution · 2050_SSP245 · Maxent · PA-1 · Probability` |

### 6.3 Raster Matching

- Future rasters are matched to training rasters **by positional order** (not by file name)
- The number and order of variables must be the same as in training
- Mismatch: clear error message in the log panel

---

## 7. Tab ⑤ — Range Change

**File:** `tabs/tab_range.py`

### 7.1 Map Styles

| Value | Color | Meaning |
|-------|-------|---------|
| -2 | `#f03b20` (red) | Lost area |
| 0 | `#f0f0f0` (light grey) | Stable absence |
| 1 | `#99d8c9` (light teal) | Stable presence |
| 2 | `#2ca25f` (green) | Gained area |

---

## 8. Tab ⑥ — Evaluation

**File:** `tabs/tab_evaluation.py`

### 8.1 Sub-Tabs

| Tab | Content | Saved File |
|-----|---------|-----------|
| Model Scores | ROC-AUC, TSS, Boyce scatter/box/bar | `evaluation_model_scores.png` |
| ROC Curves | Train + Test ROC curve | `evaluation_roc_curves.png` |
| Omission & PR | Omission Rate + Precision-Recall + Calibration | `evaluation_omission_pr_calibration.png` |
| Variable Importance | Permutation-based variable importance | `evaluation_variable_importance.png` |
| Response Curves | Marginal/PDP/ICE response curves | `evaluation_response_{algo}.png` |
| Boyce Detail | Continuous Boyce Index F-ratio curve | `evaluation_boyce_{algo}.png` |
| Classification Thresholds | Optimal threshold table using 5 methods | — |

### 8.2 Model Scores

**Grouping options:** Algorithm / PA Set / CV Run  
**Chart types:** Scatter + range / Box plot / Bar mean±SD

**Performance criteria:**

| Metric | Random | Good | Excellent |
|--------|--------|------|-----------|
| ROC-AUC | 0.5 | > 0.7 | > 0.9 |
| TSS | 0.0 | > 0.4 | > 0.7 |
| Boyce | ~0 | > 0.5 | > 0.8 |

### 8.3 ROC Curve

Separate subplot per algorithm (3 columns, scrollable):

| Line | Style | Color | Content |
|------|-------|-------|---------|
| Test curve (thick) | Solid | Set2 palette | `Test AUC = X.XXX` |
| Test CV repetitions (thin) | Solid, 25% transparent | Set2 palette | Each CV run |
| Train curve (thick) | Dashed | `#2471A3` blue | `Train AUC = X.XXX` |
| Train CV repetitions (thin) | Dashed, 25% transparent | `#2471A3` blue | Each CV run |
| Reference line | Dotted | Grey | AUC=0.5 (random) |

> **Why does Train AUC = 1.0 appear?** Strong models such as RF, XGB, LGB, CAT, GBM, ANN memorize the training data. This is normal. What matters is the **Test AUC** value.

**Data source:** CV models (RUN1, RUN2...) — not Full models. CV models are evaluated on splits reproduced with the same random seed (`seed = run_idx * 100 + pa_idx`).

### 8.4 Omission Rate, Precision-Recall, and Calibration

3 subplots per algorithm (scrollable):

#### Omission Rate
- X axis: Threshold value (0–1)
- Y axis: Omission rate (proportion of occurrences below that threshold)
- Solid = test set, Dashed = train set
- Low omission = good model
- Vertical grey dotted line: threshold = 0.5

#### Precision-Recall
- X axis: Recall (sensitivity)
- Y axis: Precision
- **AP score** (Average Precision) shown in legend — area under the curve
- Horizontal grey line: baseline (occurrence rate = prevalence)
- Advantage over ROC: more informative for imbalanced datasets (few presences / many background points)

#### Calibration (Reliability Diagram)
- X axis: Mean predicted probability
- Y axis: True occurrence rate
- Dotted diagonal: perfect calibration
- If the curve is close to the diagonal, the model produces reliable probabilities

### 8.5 Response Curves

**Methods:**

| Method | Code | Description |
|--------|------|-------------|
| Marginal | `marginal` | 1 variable is varied, others held at median (biomod2 style). Fast |
| Partial (PDP) | `partial` | Average over actual covariate distribution (Friedman 2001). More robust |
| ICE | `ice` | Separate curve for each sample (Goldstein 2015). Reveals interactions |

### 8.6 Boyce Index

- **CBI (Continuous Boyce Index):** Hirzel et al. (2006)
- Expected / observed ratio curve
- CBI ≈ 0: random; CBI > 0.5: good; CBI > 0.8: excellent
- Particularly recommended for MaxEnt, ENFA, Mahalanobis (does not require pseudo-absences)

### 8.7 Threshold Methods

| Method | Description |
|--------|-------------|
| `max_tss` | Maximizes TSS: Sensitivity + Specificity - 1. SDM standard, independent of prevalence |
| `max_kappa` | Maximizes Cohen's Kappa |
| `sens_spec_eq` | Sensitivity = Specificity equality |
| `min_omission` | Minimum omission rate |
| `prevalence` | Threshold = occurrence rate in the sample |

### 8.8 Saving — Progress Bar

"Save All Charts" now uses `QProgressDialog`:

1. **Rendering Model Scores…**
2. **Rendering ROC Curves…**
3. **Rendering Omission / PR / Calibration…**
4. **Rendering Variable Importance…**
5. **Response curves — {algo}…** (for each algorithm)
6. **Boyce index — {algo}…** (for each algorithm)
7. **Saving evaluation_scores.csv…**

- Can be cancelled with the **Cancel** button
- UI does not freeze thanks to `QApplication.processEvents()`

---

## 9. Tab ⑦ — Validation

**File:** `tabs/tab_validation.py`

- Agreement between two maps (kappa, overall accuracy)
- Area-based validation (field GPS points vs. model output)
- Pseudo-R² values
- Validation with an independent test set

**Modes:**

| Mode | Description |
|------|-------------|
| Binary | Direct 0/1 comparison with a reference map (external distribution maps such as EUFORGEN) |
| Continuous | Reclassification with a user-defined threshold; mismatched dimensions are automatically resampled |

---

## 10. Tab ⑧ — Report

**File:** `tabs/tab_report.py`

### 10.1 General

Tab ⑧ generates a **10-section scientific HTML report** covering all analysis steps (data summary, variable selection, methodology, model parameters, evaluation results, current and future distribution maps, range change, validation, session log). The report is generated with a single click and all figures are embedded as base64-encoded content within the HTML — no external dependencies.

### 10.2 Report Sections

| # | Title | Content |
|---|-------|---------|
| 1 | Study Summary | Species name, working directory, date, number of data points used, selected variables |
| 2 | Variable Selection | Correlation heatmap, PCA, LASSO, Ridge VIF charts; variable priority table; Q1-level interpretations |
| 3 | Methods | SDM methodology, references for algorithms used, data split strategy |
| 4 | Model Parameters | Hyperparameter table and descriptions for each algorithm |
| 5 | Model Evaluation | ROC-AUC, TSS, Boyce, Kappa table; threshold values; performance interpretations |
| 6 | Current Distribution Maps | PNG outputs saved by the user from the map viewer + automatic captions |
| 7 | Future Projections | Future scenario PNG outputs; grouped by period (7.1.1, 7.1.2, …) |
| 8 | Range Change Analysis | Lost / gained / stable habitat area statistics |
| 9 | Validation | External validation metrics |
| 10 | Session Log | Full session log; temporal record of all steps |
| — | Citation | Official citation with DOI link |

### 10.3 Section Selection

The user selects which sections to include in the report via a checkbox list:

| Checkbox | Corresponding Section |
|----------|-----------------------|
| ☑ Variables | Section 2 — Variable selection |
| ☑ Methods | Section 3 — Methodology |
| ☑ Parameters | Section 4 — Model parameters |
| ☑ Evaluation | Section 5 — Evaluation results |
| ☑ Maps (Current) | Section 6 — Current distribution maps |
| ☑ Future Maps | Section 7 — Future projections |
| ☑ Range Change | Section 8 — Range change |
| ☑ Validation | Section 9 — Validation |
| ☑ Session Log | Section 10 — Session log |

### 10.4 Map PNGs

**Section 6 (Current):** PNG files saved by the user in the `output_dir/` root are scanned. Type is inferred from the file name:

| Keyword in file name | Caption type |
|---------------------|-------------|
| `prob` | Suitability probability map |
| `bin` | Binary presence/absence map |
| `range` | Range change map |
| `ensemble` / `EMwmean` / `EMca` | Ensemble model map |

**Section 7 (Future):** Subdirectories under `output_dir/` (excluding current, figures, validation) are treated as time periods; each period is displayed under its own sub-heading (7.1.1, 7.1.2, …).

### 10.5 Geographic Aspect Ratio

All map renders in the report (from EMwmean TIFs) apply a cosine-latitude correction:

```
geo_ratio = (lon_span × cos(lat_mid)) / lat_span
map_h     = map_w / geo_ratio
```

This eliminates the distortion of plate carrée projection and ensures maps are displayed at their true geographic proportions.

### 10.6 Reset Behavior

When "↻ Generate Report" is clicked, the tab is reset and the entire report is regenerated based on the new outputs. This is important for updating the report after performing a "New Analysis".

### 10.7 Output

| File | Description |
|------|-------------|
| `{species}_report.html` | Self-contained HTML; all figures embedded as base64 |

---

## 11. Map Viewer (map_widget.py)

**Class:** `RasterMapWidget(QWidget)`

### 11.1 General API

```python
widget.load_raster(fpath, name, style)
# style: "suitability" | "binary" | "range_change"

widget.add_scatter(lons, lats, color, size, label, ptype)
# ptype: "obs" (train/test occurrence) | "bg" (pseudo-absence)

widget.clear_scatter()   # remove all point layers
widget.clear_all()       # remove all layers + points
widget.set_app_state(state_dict)  # for species name and output directory
```

### 11.2 Control Bar

| Control | Description |
|---------|-------------|
| Layer combo | Switch between loaded layers |
| Colormap combo | Color scale for suitability map (visible only in suitability mode) |
| **Hide BG / Show BG** | Hide/show pseudo-absence points (ptype="bg") |
| **Hide T/T / Show T/T** | Hide/show train/test occurrence points (ptype="obs") |

### 11.3 Color Scales (Suitability)

| Scale | Code | Description |
|-------|------|-------------|
| HABITUS Default | `_habitus_cmap` | Neutral grey → teal → green |
| Viridis | `_viridis` | Color-blind safe |
| YlGnBu | `_ylgnbu` | Ecology standard |
| RdYlGn | `_rdylgn_r` | Traffic light (red=low, green=high) |
| Spectral_r | `_spectral_r` | SDM classic |
| Magma | `_magma` | Dark background |
| Hot | `_hot` | Heat map |
| Greens | `_greens` | Simple green |

### 11.4 Point Visibility Logic

```python
_visible_sets = [
    (lons, lats, color, size, label)
    for lons, lats, color, size, label, ptype in self._scatter_sets
    if (ptype != "bg"  or self._bg_visible)   # BG button
    and (ptype != "obs" or self._obs_visible)  # T/T button
]
```

- The legend shows only visible sets — if a toggle is off, those points are also excluded from the legend

### 11.5 Geographic Aspect Ratio Correction

All map viewers (Distribution, Future, Range Change) apply a cosine-latitude correction when displaying rasters:

```python
lon_span  = bounds.right - bounds.left
lat_span  = bounds.top   - bounds.bottom
lat_mid   = (bounds.top + bounds.bottom) / 2.0
lon_scale = cos(radians(lat_mid))
# imshow aspect parameter:
aspect = 1 / max(lon_scale, 0.01)
```

This corrects the horizontal stretching distortion in plate carrée projection. At high latitudes, maps are displayed with the correct aspect ratio.

### 11.6 Linear Scale Bar

The `_draw_scale_bar(ax, extent)` function adds a scale bar to the **bottom-right corner** of the map:

- A round distance corresponding to ~20% of the map width is selected
- Nice numbers: `[1, 2, 5, 10, 20, 25, 50, 100, 150, 200, 250, 500, 1000, 2000, 5000]` km
- Degree → km conversion using the mid-latitude: `km_per_deg = 111.32 × cos(lat)`
- Color: HABITUS theme green (`#1d5235`)

### 11.7 Map Label Format

Label shown in the layer combo and map title:

**Current maps:**
```
Current Distribution  ·  {Algorithm}  ·  PA-{N}  ·  {Probability|Binary}
Current Distribution  ·  Ensemble: {Weighted Mean|Committee Averaging}  ·  {Probability|Binary}
```

**Future maps:**
```
Future Distribution  ·  {GCM}  ·  {Scenario}  ·  {Algorithm}  ·  PA-{N}  ·  {Probability|Binary}
Future Distribution  ·  {Scenario}  ·  Ensemble: {Name}  ·  {Probability|Binary}
```

### 11.8 Saving

The matplotlib toolbar "Save" button has been replaced with a custom `_save_map_figure()`:

- Default file name: `{species_name}_{layer_name}.png`
- Default directory: output directory
- DPI: 300
- Formats: PNG, JPEG, TIFF, PDF, SVG

---

## 12. SDM Core (sdm_core.py)

**2500+ lines.** Main components:

### 12.1 DataFormatter

```python
DataFormatter(
    occurrence_csv,          # occurrence CSV file
    raster_paths,            # list of environmental rasters
    output_dir,              # output directory
    n_pa_rep     = 2,        # number of PA repetitions
    n_absences   = 1000,     # number of PAs (for ML)
    pa_strategy  = "random", # "random" | "disk" | "sre"
    n_background = 10000,    # MaxEnt background points
    mx_bg_strategy = "random",
    cat_encoding = "one-hot",
    progress_cb  = None,
)
```

Primary outputs:
- `formatter.pa_datasets` → `[(X_df, y_arr, coords), ...]`
- `formatter.var_names` → list of variable names
- `formatter.collinearity_df` → VIF + AUC DataFrame

### 12.2 SDMModeler

```python
SDMModeler(
    algorithms,              # list of algorithms
    pa_datasets,             # DataFormatter output
    var_names,               # selected variables
    output_dir,
    n_cv_runs        = 2,
    data_split       = 80,   # % train/test split
    var_import_n     = 3,    # permutation repetitions
    threshold_method = "max_tss",
    progress_cb      = None,
)
```

Upon executing `modeler.run()`:
1. A model is trained for each algorithm × PA set × CV run
2. Each model is retained (`fitted_models` dict)
3. Train/test coordinates are recorded via `split_coords`
4. Full model + GeoTIFF projection

### 12.3 Threshold Methods (`find_optimal_threshold`)

```python
find_optimal_threshold(y_true, y_prob, method="max_tss")
```

| method | Computation |
|--------|------------|
| `max_tss` | argmax(Sensitivity + Specificity - 1) |
| `max_kappa` | argmax(Cohen's κ) |
| `sens_spec_eq` | argmin(\|Sensitivity - Specificity\|) |
| `min_omission` | argmin(1 - Sensitivity) |
| `prevalence` | thr = mean(y_true) |

### 12.4 Evaluation Metrics (`evaluate_all`)

```python
evaluate_all(y_true, y_prob, threshold_method="max_tss")
```

Returns:
- `ROC`: AUC value (sklearn `roc_auc_score`)
- `TSS`: True Skill Statistic (Sensitivity + Specificity - 1)
- `Boyce`: Continuous Boyce Index (Hirzel et al. 2006)
- `_thr`: Optimal threshold value
- `Sensitivity`, `Specificity`, `Kappa`

### 12.5 Stratified Train/Test Split

From v1.0.0 onward, presence and background points are split **independently**:

```python
pres_idx = np.where(y_arr == 1)[0]
bg_idx   = np.where(y_arr == 0)[0]
n_pres_tr = max(1, int(len(pres_idx) * data_split / 100))
n_bg_tr   = int(len(bg_idx) * data_split / 100)
tr = np.concatenate([pres_idx[:n_pres_tr], bg_idx[:n_bg_tr]])
te = np.concatenate([pres_idx[n_pres_tr:], bg_idx[n_bg_tr:]])
```

This approach prevents the imbalance that can arise with few presence points (e.g., all test points falling into the background class when performing an 80/20 split).

### 12.6 Correlation Matrix

```python
compute_correlation_matrix(X_df, method="pearson")
# method: "pearson" | "spearman"
```

### 12.7 Variable Priority Ranking

```python
variable_priority_ranking(X, y, vif_threshold=10, corr_threshold=0.70, method="pearson")
```

Returns for each variable: `Variable`, `VIF`, `VIF_OK`, `Corr_OK`, `Univariate_AUC`, `Recommended`

### 12.8 Response Curves

```python
compute_response_curves(modeler, pa_datasets, var_names, method="marginal")
# method: "marginal" | "partial" | "ice"
```

Returns: `{algo: {varname: (x, ymean, ysd)}}`

---

## 13. Output Files

All outputs are organized under `output_dir/`:

```
output_dir/
├── {species}/
│   ├── current/
│   │   ├── {algo}_{PA}_prob.tif       # Probability map (0–1)
│   │   ├── {algo}_{PA}_bin.tif        # Binary map (0/1)
│   │   ├── EMwmean_prob.tif           # Weighted mean ensemble
│   │   ├── EMwmean_bin.tif
│   │   ├── EMca_prob.tif              # Committee averaging ensemble
│   │   ├── EMca_bin.tif
│   │   ├── occurrence_train.csv       # Training points
│   │   ├── occurrence_test.csv        # Test points
│   │   └── occurrence_all_splits.csv  # All points (with split column)
│   ├── {scenario}/                    # Future projections
│   │   ├── {algo}_{PA}_prob.tif
│   │   └── EMwmean_prob.tif
│   └── range_change/
│       └── range_change_{scenario}.tif
├── figures/
│   ├── correlation_heatmap.png
│   ├── correlation_heatmap_selected.png
│   ├── vif_barplot.png
│   ├── high_correlation_pairs.csv
│   ├── variable_priority_ranking.csv
│   ├── evaluation_model_scores.png
│   ├── evaluation_roc_curves.png
│   ├── evaluation_omission_pr_calibration.png
│   ├── evaluation_variable_importance.png
│   ├── evaluation_response_{algo}.png
│   ├── evaluation_boyce_{algo}.png
│   ├── evaluation_scores.csv
│   ├── roc_curves/
│   │   └── roc_{algo}.png             # Separate ROC for each algorithm
│   └── response_curves/
│       └── response_{algo}_{var}.png  # Each algorithm × variable
├── threshold_log.json                  # Optimal threshold values
└── {species}_report.html               # Self-contained scientific HTML report
```

### GeoTIFF Properties

- **CRS:** WGS 84 (EPSG:4326)
- **Data type:** float32
- **Suitability values:** 0.0–1.0 (normalized)
- **Binary values:** 0 (absence) or 1 (presence)
- **Range change values:** -2, 0, 1, 2
- **NoData:** -9999 or rasterio default
- **STATISTICS_MINIMUM/MAXIMUM:** Written as raster metadata

---

## 14. New Features — v1.0.0

### 14.1 Map Viewer — Train/Test Toggle Button

**File:** `map_widget.py`

- `self._obs_visible = True` state variable added
- `_btn_obs` button (Hide T/T / Show T/T) added next to the BG button
- `_toggle_obs_points()` and `_refresh_obs_button()` methods
- `clear_scatter()` and `clear_all()` reset `_obs_visible`
- Scatter filter: `ptype != "obs" or self._obs_visible` condition added

### 14.2 Map Viewer — Geographic Aspect Ratio

**File:** `map_widget.py`

- `aspect = 1 / max(cos(lat_mid), 0.01)` applied in all `imshow` calls
- Applies to Distribution, Future, and Range Change map viewers
- Used in conjunction with `tight_layout()`

### 14.3 Map Viewer — Linear Scale Bar

**File:** `map_widget.py`

- `_draw_scale_bar(ax, extent)` method
- Automatically added on every `_draw()` call
- Bottom-right corner, HABITUS color theme, label with semi-transparent white background

### 14.4 Enhanced Map Labels

**Files:** `tabs/tab_models.py`, `tabs/tab_projection.py`

- `_make_current_label(key)`: raw key → readable title
- `_make_future_label(key, scenario, gcm)`: includes GCM + scenario information
- **GCM model** field added to `tab_projection.py`

### 14.5 Window Title

**File:** `main_dialog.py`

- `"HABITUS – Species Distribution Modelling Toolkit"` → `"HABITUS"`

### 14.6 VIF Tab — Safe Variables Banner

**File:** `tabs/tab_vif.py`

- `self._safe_label`: green banner, dark green text, 12px bold
- `_populate_pairs()` → `_rebuild_checklist()` call order corrected (bug fix)
- Banner is shown/hidden dynamically

### 14.7 VIF Tab — Panel Size and Readability

**File:** `tabs/tab_vif.py`

- Dual panel splitter: 400/280 → 520/300
- Header font: 12px → 13px; Table: font-size 12px, row height 30px

### 14.8 ROC Curve — Train + Test Comparison

**File:** `tabs/tab_evaluation.py`

- Each subplot: thick solid test + thick dashed train curve
- Individual CV repetitions shown as thin transparent lines in the background

### 14.9 Omission Rate + Precision-Recall + Calibration Tab

**File:** `tabs/tab_evaluation.py`

- `_omission_tab()` and `_plot_omission()` new methods
- 3-column × N algorithm row layout; separate curves for train and test

### 14.10 Tab ⑧ Report (New Tab)

**File:** `tabs/tab_report.py`

- 10-section Q1-level scientific HTML report generator
- All figures embedded as base64; single self-contained HTML file
- Map PNGs are taken from files saved by the user (see §10)
- Geographic aspect ratio correction applied in all TIF renders
- `TAB_REPORT = 7` in `main_dialog.py`; `reset_all()` resets the tab

### 14.11 Output Directory Reset Bug Fix

**File:** `main_dialog.py`

- `reset_all()` now clears both `tab_data.out_dir` and `tab_models.out_dir`
- New species analyses after "New Analysis" are correctly saved to the new folder
- Previous behavior: `tab_models.out_dir` was not cleared, so all files were written to the old folder

### 14.12 Stratified Train/Test Split

**File:** `sdm_core.py`

- Presence and background points are now split independently
- Imbalance in low-presence situations (e.g., 18 points → only 1 test point error) is eliminated

---

## 15. Build and Distribution

### 15.1 PyInstaller (.exe)

```bash
cd D:\claude_project\habitus
pyinstaller habitus.spec --clean -y
```

**habitus.spec** basic structure:
- `Analysis.datas`: rasterio proj data, pygam, catboost resources
- `EXE.name`: "HABITUS"
- `EXE.icon`: `icon.ico`
- `onedir` mode (folder, not single exe)

### 15.2 Inno Setup (Installer)

```bash
"C:\Program Files (x86)\Inno Setup 6\ISCC.exe" habitus_setup.iss
```

Output: `installer/HABITUS_Setup_v{version}.exe`

### 15.3 Version Update

1. `version.py` → change `APP_VERSION`
2. Update the `About` tab
3. Update version information in `habitus.spec`
4. `git tag v{version}` → `git push origin v{version}`
5. Create a release on the GitHub Releases page, attach the `.exe`

### 15.4 Automatic Update Check

`updater.py`:
- At startup, queries `the GitHub releases API` API
- Runs in a QThread — does not block the UI
- If a new version is available, a notification is shown to the user

---

## 16. Frequently Asked Questions and Troubleshooting

### Why does Train AUC = 1.0 appear?

Models such as RF, XGB, LGB, CAT, GBM, BRT, ANN can memorize the training data. This is completely normal. Look at the **Test AUC** value; that reflects true predictive power.

### I am getting a PROJ_LIB error

The `_proj_valid()` function in `main.py` prioritizes rasterio's own proj.db. Conflicts may occur on systems with PostgreSQL/PostGIS installed. Check the PROJ priority order in `main.py`.

### Rasters are not loading / dimension mismatch

All rasters must share the same CRS, resolution, and extent. Check with `rasterio.open(fpath).crs`.

### Memory usage is too high

- Reduce the number of pseudo-absences (`n_absences`)
- Reduce the MaxEnt background points (`n_background`)
- Reduce the number of trees (`n_estimators`)

### The Evaluation tab opens blank

`showEvent` is triggered when the tab is first opened. Charts are generated automatically when you navigate to the tab after models have finished. The "↻ Refresh All Charts" button forces a refresh.

### Response Curves are very slow

The `partial` (PDP) or `ice` methods are slow. Select the `marginal` method (default).

### The Omission & PR tab is blank

Click "↻ Refresh All Charts", or switch between tabs — `showEvent` will be triggered again.

---

Örücü, Ö.K. & Örücü, S. (2026). HABITUS: Habitat Analysis and Biodiversity Integrated Toolkit for Unified SDM. https://doi.org/10.53463/ecopers.20260435

*This document was produced from HABITUS v1.0.0 source code. Last updated: April 2026.*
