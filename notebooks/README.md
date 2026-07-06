# Conformal prediction, from zero to shipping

This folder is a short course on conformal prediction for junior AI engineers and junior
data scientists. It assumes you can use numpy, pandas, and scikit-learn, and nothing else —
no measure theory, no prior exposure to uncertainty quantification. The promise: after the
eight notebooks below you will be able to explain conformal prediction in plain words,
implement its core recipe from scratch in about fifteen lines of numpy, diagnose the ways
it fails, and use the MAPIE library correctly on real data.

The course is told as a story. You have just joined the data science team at Hearthstone, a
fictional property-tech startup that makes instant cash offers on homes, and the
acquisitions desk has one question your point-prediction model cannot answer: how far off
can a single price be? Each notebook picks up where the previous one left off — a detour
through a telehealth triage shortlist (notebook 04), a data-poor coastal market (notebook
06), and a launch in a new region that quietly breaks your guarantee (notebook 07) —
ending with real public data and a production library (notebook 08). Every pitfall is
committed live in code and then fixed, not just described; the recurring traps are collected
in [FIELD_GUIDE.md](FIELD_GUIDE.md), which the notebooks cite by section number.

## Notebook path

1. [01_the_confident_model_that_lost_money.ipynb](01_the_confident_model_that_lost_money.ipynb) —
   why point predictions hide risk, and why the two obvious homegrown fixes (± MAE,
   training-residual quantiles) quietly fail.
2. [02_the_fifteen_line_fix_split_conformal.ipynb](02_the_fifteen_line_fix_split_conformal.ipynb) —
   split conformal prediction from scratch: calibration sets, nonconformity scores, the
   corrected quantile, and a coverage promise you can actually make.
3. [03_why_it_works_the_leaderboard_argument.ipynb](03_why_it_works_the_leaderboard_argument.ipynb) —
   the entire theory as one rank argument, verified by simulation: exchangeability, the
   coverage lottery, marginal versus conditional coverage, and how to break the machine.
4. [04_classification_the_honest_shortlist.ipynb](04_classification_the_honest_shortlist.ipynb) —
   prediction sets for classifiers: softmax is an opinion, set size is a difficulty meter,
   and per-class calibration protects the patients the average hides.
5. [05_adaptive_intervals_one_width_does_not_fit_all.ipynb](05_adaptive_intervals_one_width_does_not_fit_all.ipynb) —
   diagnosing one-width-fits-all intervals and fixing them twice: normalized scores and
   conformalized quantile regression.
6. [06_small_data_full_conformal_jackknife_cv.ipynb](06_small_data_full_conformal_jackknife_cv.ipynb) —
   when you cannot afford a calibration split: full conformal, jackknife+, and CV+, with
   their real costs measured.
7. [07_when_the_world_shifts_covariate_shift.ipynb](07_when_the_world_shifts_covariate_shift.ipynb) —
   covariate shift silently voids the guarantee; weighted conformal prediction repairs it,
   at a measurable price.
8. [08_shipping_it_conformal_prediction_with_mapie.ipynb](08_shipping_it_conformal_prediction_with_mapie.ipynb) —
   the same math through the MAPIE library on California housing and wine data, a
   vocabulary translation table, and the pre-launch checklist.

## Bonus notebooks

The `bonus/` folder contains four optional notebooks that keep the same teaching style but
zoom in on applied review moments:

1. [bonus/01_regression_with_mapie.ipynb](bonus/01_regression_with_mapie.ipynb) —
   MAPIE regression back in the Hearthstone setting, with an audit against the fifteen-line
   split conformal recipe and a CQR comparison.
2. [bonus/02_time_series_forecasting_with_mapie.ipynb](bonus/02_time_series_forecasting_with_mapie.ipynb) —
   one-step-ahead cash-demand forecasting with MAPIE's time-series wrapper, block
   bootstrap intervals, and online updates.
3. [bonus/03_coverage_gallery.ipynb](bonus/03_coverage_gallery.ipynb) —
   a compact visual gallery showing marginal coverage, local adaptivity, and a failed
   overconfident interval system.
4. [bonus/04_decision_policy_from_set_size.ipynb](bonus/04_decision_policy_from_set_size.ipynb) —
   a Cairn Health workflow notebook that turns singleton, multi-label, and empty
   prediction sets into routing policy.

## Data

All Hearthstone and Cairn Health data is fully synthetic, generated in-notebook by seeded
functions whose docstrings state exactly what is baked in — the simulators are part of the
teaching material, and later notebooks test claims the docstrings make. No CSVs to manage.

| Data | Used by | Story |
|------|---------|-------|
| `simulate_hearthstone_listings` (synthetic) | 01, 02, 03, 05, 06, 07 | Home listings priced by Hearthstone's instant-offer desk |
| Synthetic triage clusters | 04 | Cairn Health's shortlist-for-the-nurse triage assistant |
| `load_digits` (bundled with sklearn) | 04 | The set-size-as-difficulty effect on real images |
| `fetch_california_housing` (downloads once) | 08 | Real district-level house values for the MAPIE finale |
| `load_wine` (bundled with sklearn) | 08 | A deliberately small classification set for cross-conformal |

## Run

The notebooks are committed fully executed, so you can read them on GitHub without running
anything. To re-run them, from the repository root:

```bash
python -m venv .venv && source .venv/bin/activate
pip install numpy pandas scikit-learn matplotlib mapie jupyter ipykernel
jupyter nbconvert --to notebook --execute --inplace notebooks/01_*.ipynb   # or open in Jupyter
```

Every notebook is self-contained (each repeats its own setup and data simulation), seeds
all randomness, and runs end-to-end in under about two minutes on a laptop. Notebook 08
downloads the California housing data on first run; everything else is offline.
