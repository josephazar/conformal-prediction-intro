# Conformal Prediction Intro

A hands-on notebook course on conformal prediction for junior AI engineers and data
scientists. The project teaches how to turn point predictions into auditable uncertainty
intervals and prediction sets, how to diagnose the ways those promises fail, and how to
ship the methods with MAPIE without treating the library as a substitute for judgment.

The course is intentionally practical: every idea is introduced through a product scenario,
then tested in code, then translated into the language a team would use in a launch review.
The main story follows Hearthstone, a fictional property-tech startup, with a short detour
through Cairn Health for classification and triage.

## What Is Inside

- `notebooks/` - the main eight-notebook course, from point-prediction failure to MAPIE.
- `notebooks/bonus/` - four extra notebooks on MAPIE regression, time-series forecasting,
  a compact coverage gallery, and decision policy from prediction-set size.
- `notebooks/FIELD_GUIDE.md` - the recurring operational traps and guardrails.
- `notebooks/README.md` - the notebook-specific learning path.
- `GLOSSARY.md` - simple explanations of the key terms in conformal prediction.

## Learning Path

| Step | Notebook | Topic |
|---:|---|---|
| 1 | `01_the_confident_model_that_lost_money.ipynb` | Why point predictions hide business risk |
| 2 | `02_the_fifteen_line_fix_split_conformal.ipynb` | Split conformal prediction from scratch |
| 3 | `03_why_it_works_the_leaderboard_argument.ipynb` | Exchangeability, rank arguments, and marginal coverage |
| 4 | `04_classification_the_honest_shortlist.ipynb` | Classification prediction sets and set-size triage |
| 5 | `05_adaptive_intervals_one_width_does_not_fit_all.ipynb` | Normalized scores and conformalized quantile regression |
| 6 | `06_small_data_full_conformal_jackknife_cv.ipynb` | Full conformal, jackknife+, and CV+ |
| 7 | `07_when_the_world_shifts_covariate_shift.ipynb` | Covariate shift and weighted conformal prediction |
| 8 | `08_shipping_it_conformal_prediction_with_mapie.ipynb` | MAPIE, real data, and the launch checklist |

Bonus notebooks:

| Bonus | Notebook | Topic |
|---:|---|---|
| 1 | `notebooks/bonus/01_regression_with_mapie.ipynb` | MAPIE regression in the Hearthstone setting |
| 2 | `notebooks/bonus/02_time_series_forecasting_with_mapie.ipynb` | One-step-ahead forecasting with MAPIE time-series intervals |
| 3 | `notebooks/bonus/03_coverage_gallery.ipynb` | A compact visual gallery for marginal, local, and failed coverage |
| 4 | `notebooks/bonus/04_decision_policy_from_set_size.ipynb` | Turning singleton, multi-label, and empty sets into workflow policy |

## How To Run

Create a virtual environment and install the notebook dependencies:

```bash
python -m venv .venv
source .venv/bin/activate
python -m pip install --upgrade pip
python -m pip install numpy pandas scikit-learn scipy matplotlib mapie jupyter ipykernel
```

Start Jupyter:

```bash
jupyter lab
```

The notebooks are self-contained. Most data is synthetic and generated inside each
notebook with fixed seeds. Notebook 08 uses public scikit-learn datasets, including
California housing and wine.

To execute a notebook from the command line and save its outputs:

```bash
jupyter nbconvert --to notebook --execute --inplace notebooks/01_the_confident_model_that_lost_money.ipynb
```

## Teaching Principles

- Start with a concrete failure, not a theorem.
- Split data roles clearly: train, calibrate, test.
- Show the tempting mistake before the repair.
- Report coverage and width together.
- Slice coverage before declaring victory.
- Treat large or empty prediction sets as product states, not annoyances.
- Use MAPIE as an audited implementation, not as a promise generator.

## References

This course is inspired by and cites the following conformal prediction literature:

- Anastasios N. Angelopoulos and Stephen Bates. "A Gentle Introduction to Conformal
  Prediction and Distribution-Free Uncertainty Quantification." Foundations and Trends in
  Machine Learning, 2023.
- Glenn Shafer and Vladimir Vovk. "A Tutorial on Conformal Prediction." Journal of Machine
  Learning Research, 2008.
- Vladimir Vovk, Alexander Gammerman, and Glenn Shafer. *Algorithmic Learning in a Random
  World.* Springer, 2005.
- Jing Lei, Max G'Sell, Alessandro Rinaldo, Ryan J. Tibshirani, and Larry Wasserman.
  "Distribution-Free Predictive Inference for Regression." Journal of the American
  Statistical Association, 2018.
- Ryan J. Tibshirani, Rina Foygel Barber, Emmanuel Candes, and Aaditya Ramdas.
  "Conformal Prediction Under Covariate Shift." NeurIPS, 2019.
- Yaniv Romano, Evan Patterson, and Emmanuel Candes. "Conformalized Quantile Regression."
  NeurIPS, 2019.
- Rina Foygel Barber, Emmanuel J. Candes, Aaditya Ramdas, and Ryan J. Tibshirani.
  "Predictive Inference with the Jackknife+." Annals of Statistics, 2021.
- Chen Xu and Yao Xie. "Conformal Prediction Interval for Dynamic Time-Series." ICML, 2021.
- Vianney Taquet, Vincent Blot, Thomas Morzadec, Louis Lacombe, and Nicolas Brunel.
  "MAPIE: an Open-Source Library for Distribution-Free Uncertainty Quantification." 2022.
