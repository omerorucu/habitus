# HABITUS v1.1.1

**Habitat Analysis and Biodiversity Integrated Toolkit for Unified Species Distribution Modelling (SDM)**

[![License](https://img.shields.io/badge/license-MIT-green)](LICENSE.txt)
[![Platform](https://img.shields.io/badge/platform-Windows%20%7C%20macOS%20%7C%20Linux-blue)](https://github.com/omerorucu/habitus/releases/latest)

---

## What's new in v1.1.1

> **Earlier releases are no longer offered for download.** v1.0.0 to v1.0.2 and
> v1.1.0 each carry faults that are fixed here; the details are in
> [CHANGELOG.md](CHANGELOG.md). Please use v1.1.1.

**A second HABITUS window no longer opens in the middle of a model run (macOS).**
While models were running, a second copy of the program, with its own splash
screen, could open by itself. Python starts a small helper process the first time
anything creates a multiprocessing lock, and scikit-learn does so every time it
fits a forest. On macOS and Linux the helper is launched as a new copy of the
running program, and the packaged program did not recognise it as a helper. It
does now, and any copy of the program that a running one starts exits quietly
instead of opening a window. Windows has no such helper and was not affected.

This was checked on the published macOS disk images for both Apple Silicon and
Intel, on the Linux build, and by the person who reported it, on a Mac.

---

## v1.1.0

> **This release changes numbers, on purpose.** Runs on the *Pinus brutia*
> sample data, set side by side with R packages, exposed errors that were silent.
> They are fixed, and the scores that result are lower and honest. If you re-run
> an earlier analysis, expect different values.

**Results that change**

- **Ensemble scores no longer leak.** They used to be scored on a map built from
  models trained on every presence. They now come only from cross-validation
  hold-out predictions: on the sample data the ensemble ROC went from 0.989 to
  0.854 and TSS from 0.956 to 0.586.
- **The test fold no longer picks its own threshold.** The threshold is chosen
  on out-of-fold predictions of the training rows. Expect TSS to drop by roughly
  0.05 to 0.15, and cross-validation to take about twice as long.
- **The Boyce index follows `ecospat::ecospat.boyce`**, checked against R.
- **Future rasters on a different grid** were silently stretched and shifted;
  they are now aligned to the training grid, with a warning in the log and the
  report.
- **Categorical predictors** are encoded properly instead of entering models as
  numbers.
- **Default changes** for GBM, BRT, CatBoost, the minimum ensemble score and the
  threshold source, each with its evidence in [CHANGELOG.md](CHANGELOG.md). The
  old values can still be entered by hand.

**New**

- **Five more algorithms, 18 in all:** BIOCLIM, Domain, GLMNET, CART and ESM.
- **More cross-validation designs** (checkerboard, jackknife, longitude and
  latitude bands), a spatial sorting bias diagnostic, more thresholds and
  metrics, MESS and MoD extrapolation surfaces, and a tuning panel (MaxEnt
  regularisation by feature class, and grids for the other algorithms).
- **A reorganised interface**, with the Evaluation tab after Models and fewer
  scroll bars.

**macOS**

- **All 18 algorithms on both Apple Silicon and Intel**, MaxEnt included.
- **The oldest macOS is now stated as the bundled libraries state it:** macOS 14
  (Sonoma) on Apple Silicon and macOS 15 (Sequoia) on Intel. This page and the
  application's metadata both said macOS 11. What an older system does when it
  is allowed to try was not tested.

**Known limits.** The evidence is **one species and one data set**, so the
default changes and the comparison with R are not yet validated on other species
or scales. PDF report generation was not tested. The full list is in
[CHANGELOG.md](CHANGELOG.md).

---

## v1.0.3

> **macOS is back, and MaxEnt now loads on it.** v1.0.2 shipped for Windows and
> Linux only. This release brings macOS back as two disk images, one for each
> kind of Mac, with all 13 algorithms.

**macOS**

- **Two disk images, one per processor.** `HABITUS_Setup_v1.0.3_macOS_arm64.dmg`
  is for Apple Silicon (M1 and later) and `HABITUS_Setup_v1.0.3_macOS_x86_64.dmg`
  for Intel Macs; Apple menu → About This Mac shows which you have. The macOS
  disk images of earlier releases were built on an Apple Silicon machine and, as
  far as can be told, did not start on an Intel Mac. If a download would not
  install on an Intel Mac, that is the likely reason.
- **MaxEnt works.** The macOS build could not import `elapid`, which provides
  MaxEnt, so it carried 12 of the 13 algorithms. The cause was found by reading
  the libraries inside the packages: `rasterio` and `pyproj` each bundle a
  `libproj` under the same file name but from a different build (rasterio's
  exports its symbols with an `internal_` prefix, pyproj's does not), and the
  packaging step kept only one of them, so pyproj ended up bound to the wrong
  one. Every bundled library now has a name of its own, as it already did on
  Windows. The packaging self-test reports 13 of 13 on both builds. v1.0.0 and
  v1.0.1 were built the same way, so they almost certainly lacked MaxEnt on
  macOS too.
- **Homebrew is not needed.** XGBoost and LightGBM ask for an OpenMP library
  that neither ships and that, on a Mac, normally comes from Homebrew. They now
  use the copy that scikit-learn ships, and the build is tested with Homebrew's
  copy removed.
- **The disk image has an Applications shortcut**: drag HABITUS onto it. The
  images are signed with the developer's Apple Developer ID certificate and
  notarised by Apple.
- **macOS 14 (Sonoma) or later on Apple Silicon, macOS 15 (Sequoia) or later on
  Intel.** That is the oldest system the bundled geospatial libraries were built
  for, read from the libraries themselves. It is higher than the macOS 11 this
  page used to state, and older systems are not supported.
- So far this has been verified by the build's own self-test on a machine of
  each processor type, not yet on a range of Macs. A report from a Mac that
  does not work is welcome.

**All platforms**

- **MaxEnt ignored the settings you chose.** A renamed option in the `elapid`
  library meant the regularisation multiplier, the number of hinge features and
  the lambda rule were silently replaced by library defaults. They are applied
  now, MaxEnt is seeded so the same seed gives the same map, and its predictions
  are no longer rescaled after the fact, which had also distorted the
  calibration statistics. If you ran MaxEnt in an earlier version, run it again:
  results can differ.
- **The update button works.** It used to try to fetch source files from this
  repository, which holds none, so it always reported a failure and blamed your
  connection. It now downloads the installer for your platform into your
  download folder.

Full detail is in [CHANGELOG.md](CHANGELOG.md).

---

## v1.0.2

> v1.0.2 was released for Windows and Linux only. The macOS builds returned in v1.0.3, above.

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

HABITUS is a free, standalone desktop application for Species Distribution Modelling. It implements **18 algorithms** within an **eight-step guided workflow** — from raw occurrence data through variable selection, model training, future climate projection, range-change analysis, accuracy assessment and automated scientific report generation — without requiring R, QGIS or command-line tools.

Everything runs locally on your machine. No data leave your computer.

---

## Download

Installers are published on the [**latest release**](https://github.com/omerorucu/habitus/releases/latest) page.

| Platform | File | Description |
|----------|------|-------------|
| Windows | `HABITUS_Setup_v1.1.1.exe` | Installer (recommended) |
| Windows | `HABITUS_v1.1.1_Windows_x64_portable.zip` | Portable — unzip and run `HABITUS.exe`, no installation |
| Linux | `HABITUS_v1.1.1_x86_64.AppImage` | Portable — `chmod +x` then run |
| Linux | `HABITUS_Setup_v1.1.1_Linux_x64.tar.gz` | Archive, extract and run |
| macOS (Apple Silicon) | `HABITUS_Setup_v1.1.1_macOS_arm64.dmg` | Disk image for M1 and later; open it and drag HABITUS onto Applications |
| macOS (Intel) | `HABITUS_Setup_v1.1.1_macOS_x86_64.dmg` | Disk image for Intel Macs |
| All | `habitus_sample.zip` | Sample dataset: *Pinus brutia*, 108 records, 23 predictors, and the calibration-polygon and absence files for trying the calibration-area and absence-data features |

The Windows builds are code-signed with an Authenticode certificate issued to the developer, so Windows shows the publisher name rather than an unknown-publisher warning. The macOS disk images are signed with the developer's Apple Developer ID certificate and notarised by Apple, so Gatekeeper opens them without an unidentified-developer warning.

**Requirements**

| Platform | Minimum |
|----------|---------|
| Windows | Windows 10 or 11, 64-bit |
| macOS, Apple Silicon | macOS 14 (Sonoma) or later |
| macOS, Intel | macOS 15 (Sequoia) or later |
| Linux | glibc 2.35 or later (Ubuntu 22.04 and later) |

8 GB RAM is recommended for high-resolution rasters on all platforms.

---

## Features

### Modelling
- **18 SDM algorithms** — GLM, GLMNET, GBM, BRT, RF, SVM, ANN, XGBoost, LightGBM, CatBoost, GAM, CART, MaxEnt, ENFA, Mahalanobis Distance, BIOCLIM, Domain, ESM
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
