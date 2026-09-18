# HABITUS v1.0.1

**Habitat Analysis and Biodiversity Integrated Toolkit for Unified Species Distribution Modelling (SDM)**

[![License](https://img.shields.io/badge/license-MIT-green)](LICENSE.txt)
[![Platform](https://img.shields.io/badge/platform-Windows%20%7C%20macOS%20%7C%20Linux-blue)](https://github.com/omerorucu/habitus/releases/latest)

---

## What's new in v1.0.2

> **This release ships for Windows and Linux only.** The macOS build is held
> back while a packaging fault is fixed: `elapid` cannot import there, because
> `rasterio` and `pyproj` each bundle a `libproj.25.9.8.1.dylib` of the same
> version but a different build, and PyInstaller resolves pyproj's extension
> against the wrong one. MaxEnt therefore does not load and the macOS build
> carries 12 of the 13 algorithms. The same build definition produced v1.0.0
> and v1.0.1, so this has almost certainly been true since the first release;
> nothing reported it because nothing checked. **macOS users: v1.0.1 remains
> available and is not affected differently.**

Two contributions shaped this release: a detailed assessment from **Citlalli
Esparza Estrada** (UNAM) and a bug report with three requests from **Maxwell
C. Obiakara** (University of Lagos). Both supplied references, and the work
follows them rather than an opinion about them.

- **Where the model is extrapolating, on every projection.** Four rasters are
  written beside the suitability maps using ExDet (Mesgaran et al. 2014,
  [doi:10.1111/ddi.12209](https://doi.org/10.1111/ddi.12209)). `NT1` marks
  cells outside the calibration range of at least one predictor, which is what
  a MESS surface detects. `NT2` marks cells where every predictor is
  individually in range but the *combination* never occurred during
  calibration — a univariate check cannot see those by construction, and they
  are not rare: in the paper's worked example, 6,617 of the 10,785 points that
  passed the univariate test had a distorted correlation structure. Two `MIC`
  rasters name the predictor responsible.
- **Calibration areas you define yourself.** Supply ecoregions, biogeographic
  provinces or basins as a vector layer and let HABITUS keep the units that
  contain an occurrence record. Rojas-Soto et al. (2024,
  [doi:10.1111/jbi.14834](https://doi.org/10.1111/jbi.14834)) compared seven
  methods across 31 species; this is the one they recommend, and **both
  methods HABITUS had before fall in the group they found weaker**.
- **Univariate AUC no longer weights variable ranking**, and its badge is no
  longer traffic-lit. A ranking users follow is a decision in all but name,
  and a low univariate AUC is not by itself a reason to drop a predictor.
- **GLM terms mean what they say.** "Quadratic" silently fitted every pairwise
  interaction as well. There are now three settings: `linear`, `quadratic`
  (no interactions) and `interactions`. With eight predictors that is 8, 16
  and 44 terms.
- **A splash screen** naming each package as it loads, instead of several
  seconds of empty screen that is indistinguishable from a failed start.
- **`HABITUS --selftest`** exercises every lazily imported path on real data
  and prints a report. If the program will not open, run this and send the
  output. The release workflows run it against the frozen binary, so a bundle
  missing a dependency fails the build instead of shipping. It found two real
  faults on its first run.

**Fixed:** startup failed on any machine with another GDAL installation (the
PROJ repair checked that `proj.db` existed, not that it was usable); applying
an update failed with `[WinError 5]` on a Program Files install; the report
asserted a variable-exclusion procedure that was not followed; a block size
derived from the variogram was recorded as user-specified.

Full detail, including what was deliberately left out, is in
[CHANGELOG.md](CHANGELOG.md).

---

## Overview

HABITUS is a free, standalone desktop application for Species Distribution Modelling. It implements **13 algorithms** within an **eight-step guided workflow** — from raw occurrence data through variable selection, model training, future climate projection, range-change analysis, accuracy assessment and automated scientific report generation — without requiring R, QGIS or command-line tools.

Everything runs locally on your machine. No data leave your computer.

---

## Download

Installers are published on the [**latest release**](https://github.com/omerorucu/habitus/releases/latest) page.

| Platform | File | Description |
|----------|------|-------------|
| Windows | `HABITUS_Setup_v1.0.2.exe` | Installer (recommended) |
| Windows | `HABITUS_v1.0.2_Windows_x64_portable.zip` | Portable — unzip and run `HABITUS.exe`, no installation |
| Linux | `HABITUS_v1.0.2_x86_64.AppImage` | Portable — `chmod +x` then run |
| Linux | `HABITUS_Setup_v1.0.2_Linux_x64.tar.gz` | Archive, extract and run |
| macOS | *not in v1.0.2* | See the note above; `HABITUS_Setup_v1.0.1_macOS.dmg` remains on the [v1.0.1 release](https://github.com/omerorucu/habitus/releases/tag/v1.0.1) |
| All | `habitus_sample.zip` | Sample dataset: *Pinus brutia*, 108 records, 23 predictors, and the calibration-polygon and absence files for trying the v1.0.2 features |

The Windows builds are code-signed with an Authenticode certificate issued to the developer, so Windows shows the publisher name rather than an unknown-publisher warning. The macOS disk images, when published, are signed and notarised by Apple.

**Requirements**

| Platform | Minimum |
|----------|---------|
| Windows | Windows 10 or 11, 64-bit |
| macOS | macOS 11 (Big Sur) or later, Apple Silicon or Intel |
| Linux | glibc 2.31 or later (Ubuntu 20.04 and later) |

8 GB RAM is recommended for high-resolution rasters on all platforms.

---

## Features

### Modelling
- **13 SDM algorithms** — GLM, GBM, BRT, RF, SVM, ANN, XGBoost, LightGBM, CatBoost, GAM, MaxEnt, ENFA, Mahalanobis Distance
- **Pseudo-absence generation** — random, disk exclusion and surface range envelope strategies, with independent settings for machine-learning, MaxEnt and presence-only algorithms
- **Stratified train/test split** — presence and background points split separately
- **Ensemble modelling** — performance-weighted mean and committee averaging
- **Multi-scenario projection** — current and unlimited future climate scenarios (CMIP6 / WorldClim / CHELSA)
- **Range change analysis** — lost / gained / stable habitat with area statistics

### Variable selection
- **VIF and correlation** — interactive variance inflation factors and Pearson/Spearman correlation with heatmaps
- **Advanced screening** — condition number, principal component analysis, LASSO (L1) and Ridge (L2) regularisation

### Evaluation and validation
- **Metrics** — ROC-AUC, TSS, continuous Boyce index, permutation variable importance, response curves (marginal, PDP, ICE)
- **Five threshold methods** — max TSS, max Kappa, sensitivity = specificity, P10, minimum ROC distance
- **Independent validation** — compare against an external reference map in binary or continuous mode; confusion matrix, overall accuracy, F1 and Cohen's kappa

### Output
- **Ten-section HTML report** — generated automatically with all figures embedded
- **GeoTIFF export** — every map written as probability and binary layers
- **300 dpi figures** — publication-ready charts

---

## Documentation

| Document | Language |
|----------|----------|
| [HABITUS_Brief_Tutorial.pdf](HABITUS_Brief_Tutorial.pdf) | Hands-on tutorial (English) |
| [HABITUS_Kisa_Kullanim_Kilavuzu.pdf](HABITUS_Kisa_Kullanim_Kilavuzu.pdf) | Hands-on tutorial (Turkish) |

Both tutorials walk through the complete eight-step workflow with screenshots
from a real analysis of *Pinus brutia* Ten., and are also attached to the
[latest release](https://github.com/omerorucu/habitus/releases/latest).

The in-application Help tab contains the full workflow description, recommended minimum sample sizes per algorithm and citation details.

---

## Citation

If HABITUS contributes to a publication, please cite:

```
Örücü, Ö. K., & Örücü, S. (2026). HABITUS: Habitat Analysis and Biodiversity
Integrated Toolkit for Unified Species Distribution Modelling. Ecological
Perspective, [Technical Report]. https://doi.org/10.53463/ecopers.20260435
```

---

## Licence

MIT Licence — see [LICENSE.txt](LICENSE.txt).

---

## Developer

**Ömer K. Örücü** — omerorucu@sdu.edu.tr — ORCID 0000-0002-2162-7553
Department of Landscape Architecture, Faculty of Architecture,
Süleyman Demirel University, 32260 Isparta, Türkiye

---

*In loving memory of Nevin Örücü*
