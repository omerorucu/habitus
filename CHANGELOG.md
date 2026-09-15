# Changelog

All notable changes to HABITUS are recorded here.

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
