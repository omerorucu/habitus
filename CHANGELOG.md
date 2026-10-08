# Changelog

All notable changes to HABITUS are recorded here.

---

## v1.1.0 — 8 October 2026

This release is mostly about **correctness**. Runs on the *Pinus brutia* sample
data, set side by side with R packages (dismo, sdm, flexsdm, wallace/ENMeval),
exposed errors that were silent. They are fixed, and the evaluation numbers that
come out are lower and honest. The interface was reorganised; new algorithms,
cross-validation designs and diagnostics were added. It also brings the macOS
build to 18 of 18 algorithms and corrects what it says about the oldest macOS it
runs on.

**If you re-run an earlier analysis, expect different numbers.** Several of the
changes below lower scores on purpose. They are in the first section so that
they are not missed.

### Fixes that change results

- **Ensemble score leakage.** The Evaluation tab scored the final ensemble map,
  which was built from models trained on all presences. Ensemble scores now come
  only from cross-validation hold-out predictions. On the sample data, EMwmean
  ROC went from 0.989 to 0.854 and TSS from 0.956 to 0.586. The old map-based
  score was removed.
- **Thresholds are no longer chosen on the test fold.** TSS, Kappa, MCC,
  sensitivity, specificity and omission rates were optimistic. The default is
  now `train_cv`: the threshold is chosen on out-of-fold predictions of the
  training rows and applied to the test fold. Expect TSS to drop by roughly
  0.05 to 0.15, and cross-validation to take about twice as long. The old
  behaviour is `threshold_source=test, or_reference=train`.
- **Boyce index follows `ecospat::ecospat.boyce`** (a line-by-line port of
  4.1.4, checked against R). The old formula remains available as `legacy`. A
  single Boyce value is noisy (fold standard deviation about 0.3).
- **Future rasters on a different grid.** A future scenario raster whose grid
  differed from the training grid (one cell wider, for example) was silently
  stretched and shifted. Layers are now aligned to the training grid by
  position, with a warning in the log and in the report (7.2 Processing
  Notices), and outputs always have the training grid's size.
- **Categorical predictors.** Class codes used to enter models as numbers. Now:
  one-hot for GLM, GLMNET, SVM and ANN; factor terms for GAM; categorical
  features for MaxEnt; BIOCLIM, Domain, ENFA and Mahalanobis exclude them, with
  a notice; tree models keep integer splits. Pseudo-absence sampling respects
  categorical NoData cells (41 of 1000 points were silently lost in the sample).
  Classes with fewer than 3 presences or 5 records are reported by name.
- **Projection maps are no longer stretched to their own min-max range**, and
  binary maps use the scale on which the threshold was learned.
- **"Full" model scores** come from the cross-validation replicates. Ensemble
  model selection no longer relies on optimistic random-split scores, and the
  ensemble weights and quality filter are derived from folds other than the one
  being scored.
- **ENFA K and Mahalanobis regularisation** were visible but inert. They are now
  passed to the models.
- **Default ensemble threshold.** A minimum TSS of 0.60 produced no ensemble and
  no warning. Defaults are now metric-specific (ROC 0.70, TSS 0.40, Boyce 0.50),
  and a clear warning appears if no model passes.
- **Tuning** uses the threshold source and Boyce formula chosen in the Models
  tab, and flags or skips grid points whose omission rate is 0 only because of
  tied predictions.
- **Input validation.** Silent fall-backs became errors or warnings: external
  test CSV, swapped longitude and latitude, a missing absence file or column, the
  accessible-area fall-back, a polygon without presences, raw error messages,
  invalid threshold names.
- **GAM variable importance** is computed. `max_kappa` no longer writes Kappa
  into the TSS column. Permutation importance is seeded.
- **Background threads.** The update check, the download and the model and
  projection workers could crash the process (Qt fast-fail `0xC0000409`) when the
  window was closed or on "New Analysis". Threads are now cancellable and
  unparented, and closing the window while work is running asks for confirmation.

### New

- **Algorithms:** BIOCLIM, Domain, GLMNET, CART and ESM, which makes 18; GAM
  `n_splines` and `lam`. None of them needs a package beyond NumPy and
  scikit-learn.
- **Cross-validation designs:** checkerboard (one and two levels), jackknife,
  longitude and latitude bands; a spatial sorting bias diagnostic; explicit
  warnings for merged or empty folds.
- **Thresholds:** `mtp`, `lpt`, `p5` (any `p<N>`), `sensitivity:<target>`,
  `max_jaccard`, `max_sorensen`, `max_fpb`.
- **Metrics:** Kappa, MCC, sensitivity, specificity, Jaccard, Sorensen, FPB,
  OR_MTP, OR_10p, AUC_train, dAUC.
- **Extrapolation:** MESS and MoD surfaces, masking of cells with MESS below 0,
  and clamping to the training range.
- **Tuning** (Models, "Tuning (optional)"): a MaxEnt `rm` by feature-class grid,
  and grids for GBM, RF, SVM, ANN, XGB, LGB, GLMNET, CART and GAM; selection
  rules `enmeval`, `enmeval_guard` and `max_auc`; a table, a plot, CSV export and
  "Apply selected setting".
- **Ensembles:** median, meansup and meanthr; a weight statistic and power; SD,
  CV and range uncertainty maps.
- **Sampling bias:** a target-group background (CSV and/or bias raster);
  pseudo-absence strategies `kmeans`, `env_const`, `geo_const`, `geo_env_const`
  and `geo_env_km_const`; class balancing, off by default.
- **MaxEnt** "Auto (maxnet default rule)" feature classes. The default is
  unchanged.
- **Categorical rasters** are listed and selectable in the Variables tab.
- **`evaluation_scores.csv`** is written to the output root with a
  `Score_source` column; a hand-edited file is backed up as `.bak.csv`. The
  Evaluation tab computes once per run, and the ROC curves use true
  cross-validation hold-out predictions (per-fold curves, their mean and a ±1 SD
  band).

### Interface

- Tab order is now Data, Variables, Models + Current Map, **Evaluation**, Future
  Scenarios, Range Change, Validation, Report, Help.
- Fewer scroll bars (none on the main workflow at 1707×960 and 1920×1080). Sizes
  come from `ui_style.py` and scale with `HABITUS_UI_SCALE`. One standard primary
  action button, a square and readable correlation heat map, a scroll-free
  variable list and a wider map layer selector.

### Defaults that changed

Old values can still be entered by hand. The evidence is a shared table of four
cross-validation designs by two pseudo-absence settings, with paired fold
differences.

| Setting | Old | New | Evidence |
|---|---|---|---|
| GBM | depth 3, 500 trees, rate 0.05, no subsampling, leaf 1 | depth 1, 200 trees, rate 0.1, subsample 0.75, leaf 5 | Boyce +0.185 [0.062, 0.335], positive in 8 of 8 cells, worst-cell AUC −0.008 |
| BRT | depth 5 | depth 2 | Boyce +0.205, AUC +0.006 |
| CatBoost | 500 iterations, depth 6, rate 0.05, l2 3 | 300 iterations, depth 4, rate 0.03, l2 5 | Boyce +0.22 [0.14, 0.32], worst-cell AUC −0.005 |
| Ensemble minimum score | TSS 0.60 | ROC 0.70 / TSS 0.40 / Boyce 0.50 | no model passed at 0.60 |
| Threshold source | test fold | `train_cv` | the test fold must not choose its own threshold |
| Boyce | fixed window | ecospat | checked against R |

### macOS

- **18 of 18 algorithms on both Apple Silicon and Intel**, MaxEnt included. This
  carries over the v1.0.3 packaging fix and adds nothing that could undo it: the
  five new algorithms use no library beyond NumPy and scikit-learn. It was
  checked on the published disk images themselves, not only on the build
  machine.
- **Code signing is retried.** Signing asks Apple's timestamp server for a
  timestamp on every one of some 290 files, and when the server did not answer
  the Intel build stopped ("A timestamp was expected but was not found"). Each
  signature is now tried up to six times.
- **The oldest macOS is now stated as the libraries state it.** The
  application's metadata said macOS 11 (Big Sur), and so did this project's own
  README. The native libraries inside the packages declare a minimum of **macOS
  14 (Sonoma) on Apple Silicon and macOS 15 (Sequoia) on Intel**, which was read
  from the libraries themselves. What happens on an older system was not tested:
  a library built for a newer macOS can still load on an older one, or can fail
  on a missing function. The bundle now declares the minimum the libraries
  declare, so an older Mac is told so up front instead of finding out later.

### Validation status

- **Comparison with R packages** (*Pinus brutia*, 108 presences, 4 folds by 2
  pseudo-absence settings, identical folds): AUC within ±0.05 for most of 13
  algorithm pairs; MaxEnt equals dismo, maxnet and flexsdm (0.82 to 0.83);
  BIOCLIM matches dismo to three decimals; HABITUS scored higher AUC for ANN
  (+0.079), GLM (+0.046) and SVM (+0.044).
- **Test suite:** 513 tests, all passing: 513 passed, 0 failed, 0 skipped, run
  in full on the release tree.
- All 18 algorithms ran on real data without crashing.
- **Interface and macOS.** The developer checked the interface by hand on
  screen, and the macOS build runs on a Mac. An automated check also downloaded
  the published macOS disk images and ran the packaging self-test on them with
  Homebrew's OpenMP library removed, on one machine of each processor type.

### Known limitations

- **One species, one data set.** The default changes and the comparison with R
  rest on *Pinus brutia* (108 presences, 4.6 km cells) and have not been
  validated on other species or scales.
- PDF report generation was not tested (the HTML report was).
- Boyce and OR_10p are very noisy with small samples (fold SD 0.2 to 0.4).
  OR = 0 for BIOCLIM and CART is mechanical (tied predictions); do not compare
  it across algorithms.
- GBM still trails flexsdm's `fit_gbm` in checkerboard Boyce (0.714 against
  0.817); AUC is equal. XGB and LGB defaults are unchanged, and LGB (31 leaves)
  is too flexible for small samples.
- MARS is not included. A long single call with no cancellation point keeps
  running in the background if the window is closed.
- The manuals (`HABITUS_MANUAL*.md`) were not renumbered for the new tab order.

---

## v1.0.3 — 6 October 2026

Three defects reported after v1.0.2, all of which made the program claim to do
something it was not doing.

### MaxEnt ignored every setting it was given

elapid renamed the regularisation multiplier from `regularization_multiplier`
to `beta_multiplier` at version 1.0. HABITUS only ever passed the old name, so
on any current installation the constructor raised `TypeError` and a fallback
ladder rebuilt the model with **every** setting at its library default. A user
who set the multiplier to 4 got 1.5; 50 hinge features became 10; the lambda
rule was discarded. Nothing in the interface, the log or the report said so.
Settings are now matched against what the installed elapid actually accepts,
and anything that genuinely has no counterpart is reported rather than dropped.

- **MaxEnt is seeded.** It was the one estimator that never received
  `random_state`, so two runs of the same analysis with the same seed produced
  different suitability maps. That is the defect v1.0.1 closed for Random
  Forest, GBM and BRT; MaxEnt was missed.
- **MaxEnt predictions are no longer rescaled.** The output was min-max
  stretched to span 0–1 across whatever was in the call. Calibration, Brier
  score and the binary threshold — all reported since v1.0.1 — were therefore
  computed on a surface whose scale had been changed after the model produced
  it, the same cell came out differently depending on which other cells were
  predicted alongside it, and every map was stretched to full range, hiding the
  case where nowhere in the study area is particularly suitable. What is
  reported is now elapid's cloglog output as it stands.
- **A failed MaxEnt fit says why.** `except Exception: pass` around three call
  styles turned an ordinary data problem — too few presences, a constant
  predictor, a column of NaN — into "failed with all known API styles. Try:
  pip install --upgrade elapid", which sent people off to reinstall a working
  library. A failed prediction no longer returns 0.5 for every cell either: a
  flat surface that looks like a result is worse than an error.

### The update button could not work

"Apply Update" downloaded individual `.py` files from the public repository
into a `patches/` folder. The public repository carries the release assets, the
changelog and the manuals, and no source, so all fifteen requests returned 404
and the user was told `0 files updated, 15 failed. Check your internet
connection and try again.` — a network diagnosis for URLs that had never
existed. The mechanism was also a hazard where it could half-work: a patches
folder left from when the source was public would shadow the newer code inside
the installed build while the version number still read as the new one.

The button now downloads the release installer for the platform it is running
on, into the user's own download folder, and opens that folder. A release with
nothing for this platform leads to the release page rather than an error.
Leftover patch folders are no longer placed on the import path; their presence
is reported once so they can be deleted.

### macOS

- **The DMG was Apple Silicon only, and its name did not say so.** It is built
  on an Apple Silicon runner, PyInstaller freezes for the architecture it runs
  on, and none of the wheels involved are universal2. On an Intel Mac the
  download therefore could not start, which is indistinguishable from "it will
  not install". Both architectures are now built and the architecture is part
  of the file name: `HABITUS_Setup_vX_macOS_arm64.dmg` and
  `…_macOS_x86_64.dmg`. **Intel Mac users of v1.0.0 or v1.0.1 need the x86_64
  file.**
- **The disk image now contains an Applications shortcut.** It was created with
  bare `hdiutil`, so it held the application bundle and nothing else. People ran
  HABITUS from the mounted read-only image, where macOS relocates it to a path
  it cannot write to, and concluded it had not installed.
- **Signing is done from the inside out.** `codesign --deep`, which Apple
  documents as unsuitable for anything but testing, signs nested code with the
  outer bundle's options in no guaranteed order. For a bundle of a few thousand
  shared libraries that is how an application arrives notarised and still
  refuses to launch.
- **MaxEnt now works on macOS.** It was unavailable from v1.0.0 to v1.0.2, and
  the cause was a file-name collision between wheels, found by reading the
  wheels themselves rather than by trial. rasterio, pyproj, pyogrio and shapely
  each vendor their own copies of the C libraries they need, and different
  wheels carry *different builds of the same library under the same file name*:
  `libproj` (rasterio 5.1 MB, pyproj 4.6 MB), `libgdal` (25 MB and 61 MB),
  `libtiff`, `libjpeg`, `libzstd` and `liblzma`. They are not interchangeable:
  rasterio's `libproj` is built with symbol renaming and exports
  `_internal_proj_context_create`, where pyproj's exports
  `_proj_context_create`. PyInstaller flattens the lot into one directory and
  keeps one file per name, so pyproj's extension ended up bound to rasterio's
  `libproj`, could not find the symbol it asks for, and failed to load. elapid
  imports pyproj through geopandas, so MaxEnt went with it; the other twelve
  algorithms do not touch it, which is why the program otherwise worked.
  Windows never had the problem because the tool that builds its wheels gives
  every vendored DLL a hash-suffixed name. The macOS build now does the same:
  every vendored library is renamed with a tag unique to its package and every
  reference to it is rewritten, so no two files share a name. The packaging
  self-test now reports 13 of 13 algorithms on macOS.

- **xgboost and lightgbm no longer depend on Homebrew on macOS.** Neither
  wheel ships an OpenMP runtime: each library asks for `@rpath/libomp.dylib` and
  carries only Homebrew's location as an rpath. On a Mac without Homebrew's
  `libomp`, which is most of them, importing xgboost raises an error that is not
  an `ImportError`, so the program fails to start rather than losing one
  algorithm. The release build never showed it, because the build machine had
  Homebrew's `libomp` installed and the self-test ran there. The bundle now
  shares the one `libomp` that scikit-learn ships, and the build removes
  Homebrew's before testing, so the self-test is a clean-machine test. Whether
  the disk images of v1.0.0 to v1.0.2 carried a `libomp` of their own was not
  checked; on a Mac without Homebrew, treat those as unverified.
- On Intel, scikit-learn's runtime lacks one symbol (`___kmpc_dispatch_deinit`)
  that xgboost's x86_64 build imports; the Apple Silicon one has it. The build
  checks the runtime's exported symbols against what the consumers import and,
  where it falls short, puts a newer runtime in the same file. It has to be one
  file: two OpenMP runtimes in one process abort with "OMP: Error #15".
- The bundle carries the committed `.icns` icon, and its version string is read
  from `version.py` instead of being repeated in the build file, where it had
  already gone stale once.

---

## v1.0.2 — 18 September 2026

This release answers the second round of feedback on v1.0.1, and in particular a
detailed assessment from Citlalli Esparza Estrada (UNAM) and a bug report and
three requests from Maxwell C. Obiakara (University of Lagos). Both supplied
references; the work below follows them rather than an opinion about them.

### Environmental novelty (the largest gap in v1.0.1)

- **ExDet novelty diagnostics on every projection.** A suitability value does not
  distinguish a cell the model can speak about from one it cannot, and until now
  nothing in the output marked an extrapolation as one. Four rasters are now
  written beside each projection:
  - `{scenario}_NT1.tif` — the cell is outside the calibration range of at least
    one predictor. This is what a MESS surface detects.
  - `{scenario}_NT2.tif` — every predictor is individually inside its range, but
    the *combination* never occurred during calibration. A univariate check
    cannot see this by construction, and it is not a corner case: in the worked
    example of Mesgaran et al., 6,617 of the 10,785 points that passed the
    univariate test had a distorted correlation structure.
  - `{scenario}_MIC_NT1.tif`, `{scenario}_MIC_NT2.tif` — which predictor is
    responsible. "There is extrapolation here" is not actionable; "because of
    precipitation seasonality" is.

  The report states what fraction of cells is novel by each test, which
  predictors drive it, and warns when more than a quarter of the projection lies
  outside the calibration range.
  (Mesgaran et al. 2014, doi:10.1111/ddi.12209)

- **NT2 is refused rather than faked when it cannot be computed.** The
  Mahalanobis distance needs the inverse of the calibration covariance matrix,
  and HABITUS allows correlated predictors, so that matrix can be numerically
  singular. When it is, NT2 is not reported and the reason is given, including
  what to do about it. A plausible-looking number from a singular matrix would
  be worse than no number.

### Calibration area

- **Supply your own calibration area as a polygon layer.** Set the accessible
  area to `polygon` and load ecoregions, biogeographic provinces or basins as
  SHP, GPKG, GeoJSON, KML or GML.

- **The biogeographic-entity method, automated.** With "keep only the polygons
  containing an occurrence record" the program selects the units the species
  actually occupies, so a continent-wide layer can be used directly instead of
  being clipped by hand first. Rojas-Soto et al. (2024) compared seven ways of
  delimiting a calibration area across 31 species: 68% of the best models came
  from the accessible-area approach, and this is the method they recommend.
  **Both methods HABITUS had before this release, `buffer` and `mcp_buffer`,
  fall in the group that paper found weaker**, and neither can express a
  barrier. They remain available and remain far better than sampling the whole
  raster. (doi:10.1111/jbi.14834)

- A layer in a different coordinate system is reprojected and the fact recorded.
  Failures stop the run and name the cause: unreadable layer, no polygons, no
  polygon containing a record (almost always a coordinate-system mismatch), or
  no overlap with the rasters.

- **The calibration area is now described in `habitus_run_config.json`**: the
  layer, whether selection by occurrence was used, how many polygons the layer
  held and how many were kept, and whether it was reprojected. Rojas-Soto et
  al.'s other finding was that published studies routinely fail to describe how
  the calibration area was built, which makes them impossible to repeat.

### Variable selection

- **Univariate AUC no longer weights the ranking.** It carried 30% of the
  priority score. The choice formally rested with the user, but a ranking users
  follow is a decision in all but name, and ordering predictors by univariate
  discrimination promotes whichever variable happens to separate presences from
  background in this sample. That is the wrong criterion for ecological
  relevance and for transferability: a predictor can discriminate well here
  through a correlation with the real driver and fail wherever that correlation
  does not hold. The score is now collinearity only (VIF and mean correlation);
  AUC stays in the table as information.

- **The AUC badge is no longer traffic-lit.** Green, amber and red said that a
  low univariate AUC is a defect. It is not: a predictor can discriminate poorly
  alone and still be the one that matters in combination. The colouring was a
  stronger steer than the ranking weight, so it went too.
  (Both raised by Citlalli Esparza Estrada.)

### Model parameterisation

- **GLM terms are now what they say.** The option named "quadratic" expanded to
  linear + quadratic + *every pairwise interaction*, and nothing in the interface
  said so. There are now three settings: `linear` (x), `quadratic` (x, x², no
  interactions) and `interactions` (the previous behaviour). In MaxEnt
  feature-class terms, L, LQ and LQP.

  This matters for sample size. With eight predictors the three produce 8, 16 and
  44 terms; under the ten-events-per-variable rule that is roughly 80, 160 and
  440 occurrence records. A curved response is now available without paying for
  the interactions. *(Requested by Maxwell C. Obiakara.)*

- **The MaxEnt regularisation multiplier is relabelled and its range widened**
  to 0.1–20. It was already adjustable, as "Regularisation β", but that is not
  the name ENMeval and kuenm use and it was being missed. It is now
  "Regularisation multiplier (β / RM)" and the tooltip says what it does, that
  HABITUS does not yet search over it, and that the best value depends on the
  calibration area (Rojas-Soto et al. 2024), so it should be revisited when that
  changes.

### Fixed

- **HABITUS could fail to start, or fail on every coordinate operation, on any
  machine with another GDAL installation.** The startup repair for a stale
  `PROJ_LIB` checked only that a `proj.db` file existed at the configured path,
  not that it was usable. PostGIS ships one of an older schema, so on those
  machines the repair was skipped and PROJ then failed with "proj.db lacks
  DATABASE.LAYOUT.VERSION.MAJOR ... It comes from another PROJ installation".
  The path is now accepted only if the database carries the schema metadata the
  linked PROJ requires.

### Startup and updates

- **A splash screen while the scientific stack loads.** HABITUS imports
  rasterio, scikit-learn, three gradient-boosting libraries, elapid, pygam and
  matplotlib before its window can appear, and on a cold start that is several
  seconds of nothing at all on screen. A user cannot tell that apart from a
  program that failed to start, so they click the icon again. The splash shows
  the mark, the authors, the version and the name of the package currently
  loading; when a packaged build is missing a dependency, the last line names
  the one it stopped on.

  This required moving matplotlib and the habitus package out of module-level
  imports in `main.py`. They were pulled in before `QApplication` existed, so
  no window of any kind could be shown during the wait.

- **Applying an update no longer fails on a normal Windows install.** Patches
  were written to a `patches` folder beside the executable, which for an
  installation in Program Files is not writable by an ordinary process:
  "Apply update" ended in `[WinError 5] Access is denied` with no way forward
  short of running the whole program as administrator, which it does not
  otherwise need. The updater now uses the install directory only when it is
  genuinely writable and a per-user directory otherwise
  (`%LOCALAPPDATA%\HABITUS\patches`, or the platform equivalent). Both
  locations are on the import path, so a portable installation keeps working
  exactly as before. If even the per-user directory is barred, the message now
  says what failed, that nothing was changed, and what to do instead.

- The version shown in the application metadata was hard-coded to 1.0.0 and is
  now read from `version.py`.

### Packaging and diagnostics

- **`HABITUS --selftest`.** A new startup mode that exercises every lazily
  imported path on real data and prints a report. It checks the PROJ database,
  a coordinate reprojection, the vector stack, a complete calibration-polygon
  read-select-buffer-rasterize cycle, an ExDet run including the NT2 branch, a
  GeoTIFF round trip, the GLM term counts, which algorithms this build can run,
  and finally Qt.

  Two reasons. Several dependencies are imported inside the function that uses
  them, so PyInstaller cannot see them and a bundle missing one starts
  normally and only fails when the user reaches that feature. And "it installs
  but it will not open" is not a diagnosis: a windowed application that dies
  during startup leaves nothing to read. The self-test runs before Qt is
  imported, writes its report to a file and prints it, so it still works when
  the graphics stack is what is broken, and names software rendering as the
  thing to try when Qt is the only failure.

  The Linux and macOS release workflows now run it against the frozen binary,
  so a bundle missing a dependency fails the build instead of shipping.

- **The build no longer depends on which developer tools are installed.**
  PyInstaller imports every submodule it scans, and `xgboost.testing` calls
  `pytest.importorskip("hypothesis")`, which raises pytest's `Skipped` rather
  than `ImportError`. With pytest absent that is a tolerated import failure;
  with pytest present it aborted the build. Test submodules are now excluded
  from collection, which is also what a shipped application wants.

- **`pyogrio` and `shapely` are now collected properly**, with their GDAL DLLs
  and data files. `pyogrio` was not collected at all, which would have made
  the calibration-polygon feature fail in the packaged build while working
  perfectly from source.

### Known limitations, unchanged

- No systematic hyperparameter search. ENMeval and kuenm exist to do this and
  HABITUS does not yet do it; the regularisation multiplier and feature classes
  have to be chosen deliberately.
- No sampling-bias surface.
- `grinnell`-style dispersal simulation for delimiting the accessible area is not
  implemented. It performed best in Rojas-Soto et al. but requires an
  ellipsoid niche model and an iterative spread simulation, which is a separate
  subsystem rather than an added option; the recommended BE method is
  implemented instead.
- Plot export is still PNG only. TIFF with selectable compression and vector PDF
  are queued. *(Requested by Maxwell C. Obiakara.)*
- The "Stage 2 — Variables" wheel-scroll trap is not yet fixed.
  *(Reported by Maxwell C. Obiakara.)*
- Occurrence records are still not matched to the environmental conditions of
  their own date.
- Multi-species batch runs are still not available.
- Rasters must still be aligned before loading; HABITUS detects misalignment but
  does not correct it.

---

## v1.0.1 — 14 September 2026

This release answers the feedback received on v1.0.0 from roughly 370 researchers.
Almost all of it concerned the same thing: the software produced a finished-looking
report whose numbers were more confident than the method behind them justified.
The changes below are about making the reported numbers honest, and about letting
the user supply data the software was previously guessing at.

**If you re-run a v1.0.0 analysis in v1.0.1, expect the evaluation scores to drop.**
That is the point. The old numbers were inflated by a validation design that tested
the model on data it had effectively already seen.

### Model validation

- **Spatial block cross-validation, now the default.** Folds are whole geographic
  blocks held out in turn, rather than randomly scattered points. Occurrence records
  are spatially autocorrelated: under a random split a test record usually sits within
  a few hundred metres of a training record with nearly identical predictor values, so
  the model is scored on information it already holds and AUC and TSS come out too
  high. Block size is measured from the empirical variogram of your own predictors; if
  the study area cannot hold enough blocks at that size the block is halved until it
  can, and the reduction is reported. Where blocking is impossible the run falls back
  to random folds and says so, in the log and in the report. `random` remains
  selectable for comparability with older results.
  (Roberts et al. 2017, doi:10.1111/ecog.02881; Valavi et al. 2019)
- **Environmental block cross-validation** as a third option: folds are k-means
  clusters in predictor space, which tests extrapolation into unseen conditions more
  directly — the question a climate projection actually asks.
- **Uncertainty on every metric.** The report now leads with per-algorithm mean,
  standard deviation and range across folds and replicates. A single number invited
  readers to rank algorithms that differ by less than their own run-to-run variation.
- **Calibration.** Brier score, calibration slope and intercept, and binned
  calibration error are computed alongside AUC/TSS/Boyce. A model can discriminate
  perfectly and still return probabilities that do not match observed frequencies,
  and a suitability surface read as a probability depends on the second property.
  (Pearce & Ferrier 2000)
- **Prevalence and threshold are reported per model.** TSS and kappa both move with
  prevalence, so a TSS quoted without it cannot be compared across species or across
  runs with different background sizes. The binary threshold is reported because a
  binary map cannot be interpreted, or reproduced, without it.
- Cross-validation runs now honour the binary threshold rule selected in the Models
  tab. They previously fell back to `max_tss` regardless, so the CV rows and the
  full-data row of the same algorithm could be thresholded by different rules.

### Absence and background data

- **You can supply your own absence records.** A new "My Own Absence Data" tab takes
  a CSV of surveyed absences, empty atlas squares, or your own background sample.
  These are labelled **true absences** throughout the outputs, never pseudo-absences,
  because the distinction changes what the evaluation statistics mean. Two modes:
  `replace` (your records are the whole absence class) and `augment` (your records
  plus generated points).
- **Accessible-area restriction, on by default.** Background points are drawn only
  from within a buffer around the occurrence records — the "M" of the BAM framework.
  Cells the species could not plausibly have reached are not evidence of
  unsuitability, but the model was treating them as such, which inflated the scores
  and distorted the response curves. Buffer method and radius are user-set; `none`
  restores the previous behaviour. (Barve et al. 2011)
- **Spatial thinning of occurrence records**, manual or automatic. Records clustered
  around roads, reserves and institutes pull the fitted response curves towards the
  conditions of well-surveyed places. Automatic mode derives the distance from the
  same variogram range that sizes the CV blocks, and reduces it if it would discard
  more than half the records. (Aiello-Lammens et al. 2015)
- Minimum-distance ("disk") background sampling now uses great-circle distance. The
  previous `degrees / 111` conversion ignored the cos(latitude) shrinkage of
  longitude, so the exclusion disc was too small everywhere away from the equator.

### Ensembles

- **Two-layer ensemble, on by default.** Each algorithm's replicates are averaged
  first, then the per-algorithm means are combined across algorithms. A single run is
  one draw from the algorithm's own variability: the background points are redrawn,
  the fold split changes, and the stochastic learners start from a different state.
  Building the cross-algorithm layer from the averages also stops an algorithm that
  happens to have more replicates from dominating the combined map.
- **New output rasters**: `{algo}_mean_prob.tif`, `{algo}_mean_bin.tif`,
  `{algo}_sd.tif` (where that algorithm's replicates disagree), and `EMsd.tif`
  (where the algorithms disagree with each other).
- Optionally project the cross-validation models as well as the full-data models, for
  more replicates behind each algorithm's mean and spread.

### Reproducibility

- **One seed for the whole run**, set in the Data tab and applied to background
  sampling, fold assignment and every stochastic estimator.
- **Random Forest, GBM and BRT were never seeded at all.** They are bagged or
  subsampled learners, so two runs on identical data produced different suitability
  maps and nothing in the output explained the difference. Fixed.
- **`habitus_run_config.json`** is written next to the maps with every run: seed,
  every setting, the cross-validation design actually used, package versions, and an
  explicit list of the settings left at their shipped defaults.
- The report's Methods section now describes what the run actually did. It previously
  read several data-stage settings from the wrong object and printed the shipped
  defaults regardless of what had been chosen.

### Data handling

- **Raster grid consistency is checked before anything else.** Predictors are read
  cell by cell into one table, so a raster on a different grid silently paired cell
  (i,j) of one layer with a geographically different cell of another and produced a
  finished-looking map from mismatched data. The run now stops and names the offending
  file, the specific mismatch, and the QGIS tool that fixes it.
- **Resolution advisory.** When predictor cells are about 4 km or larger the run says
  so, because on steep topographic or coastal gradients most of the variation that
  separates occupied from unoccupied sites happens below that scale.
- **Small-sample protection.** Fewer than ten presences, or fewer than ten presences
  per predictor, now raises an advisory that appears in the report as well as on
  screen.

### Errors and interface

- **Error messages are readable and copyable.** `QMessageBox.critical` truncated the
  text, and the part that was cut off was the line naming the exception — the only
  line that says what went wrong. The new dialog shows a headline, a plain-language
  hint for recognised failures, the full traceback in a selectable pane, a Copy
  button, and the path to the run log.
- Cross-validation, ensemble and projection failures in the Validation and Advanced
  Variable Analysis tabs now use the same dialog.

### Known limitations, unchanged in this release

- Occurrence records are not matched to the environmental conditions of their own
  date; a time-calibrated analysis is not supported.
- Suitability is assigned to climatically suitable but functionally unreachable
  patches; there is no connectivity or dispersal mask beyond the accessible-area
  buffer.
- Only presence–background and presence–absence modelling; abundance is not modelled.
- Rasters must be aligned before loading. HABITUS now detects misalignment but does
  not correct it.
- Multi-species batch runs are not available; one species per run.

---

## v1.0.0 — 11 August 2026

First public release. Thirteen algorithms, eight-step guided workflow, ensemble
modelling, multi-scenario projection, range-change analysis, automated HTML report.
Windows, macOS and Linux builds.
