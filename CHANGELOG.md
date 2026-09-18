# Changelog

All notable changes to HABITUS are recorded here.

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
