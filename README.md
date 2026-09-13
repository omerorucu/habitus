# HABITUS v1.0.0

**Habitat Analysis and Biodiversity Integrated Toolkit for Unified Species Distribution Modelling (SDM)**

[![License](https://img.shields.io/badge/license-MIT-green)](LICENSE.txt)
[![Platform](https://img.shields.io/badge/platform-Windows%20%7C%20macOS%20%7C%20Linux-blue)](https://github.com/omerorucu/habitus/releases/latest)

---

## Overview

HABITUS is a free, standalone desktop application for Species Distribution Modelling. It implements **13 algorithms** within an **eight-step guided workflow** — from raw occurrence data through variable selection, model training, future climate projection, range-change analysis, accuracy assessment and automated scientific report generation — without requiring R, QGIS or command-line tools.

Everything runs locally on your machine. No data leave your computer.

---

## Download

Installers are published on the [**latest release**](https://github.com/omerorucu/habitus/releases/latest) page.

| Platform | File | Description |
|----------|------|-------------|
| Windows | `HABITUS_Setup_v1.0.0.exe` | Installer (recommended) |
| Windows | `HABITUS_v1.0.0_Windows_x64_portable.zip` | Portable — unzip and run `HABITUS.exe`, no installation |
| macOS | `HABITUS_Setup_v1.0.0_macOS.dmg` | Disk image, Apple Silicon and Intel |
| Linux | `HABITUS_v1.0.0_x86_64.AppImage` | Portable — `chmod +x` then run |
| Linux | `HABITUS_Setup_v1.0.0_Linux_x64.tar.gz` | Archive, extract and run |

The Windows builds are code-signed with an Authenticode certificate issued to the developer, so Windows shows the publisher name rather than an unknown-publisher warning. The macOS disk image is signed and notarised by Apple.

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
