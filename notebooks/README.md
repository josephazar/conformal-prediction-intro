# Conformal prediction, from zero to shipping

This folder is a short course on conformal prediction for junior AI engineers and junior
data scientists. It assumes you can use numpy, pandas, and scikit-learn, and nothing else —
no measure theory, no prior exposure to uncertainty quantification. The promise: after the
eight Part I notebooks below you will be able to explain conformal prediction in plain words,
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

Part II (notebooks 09-15) takes the same machinery to ordered data. It opens with the one
assumption everything rests on, exchangeability, shown, broken and tested, then follows Kestrel
Operations, a fictional company that forecasts building electricity and cloud CPU load and
wants alarms it can trust. If you are starting a project on calibrated forecasts, IoT or
cloud monitoring, read [PROJECT_GUIDE.md](PROJECT_GUIDE.md) first: it maps typical project
objectives to notebooks and explains how to move from the simulated series to public data.

## Part I: notebook path

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

## Part II: forecasts and alarms you can trust

9. [09_exchangeable_or_not.ipynb](09_exchangeable_or_not.ipynb) —
   exchangeable is not independent, exchangeable at which level (rows or whole buildings),
   six ways data stops being exchangeable, and three ways to test it, including a conformal
   test martingale with a false-alarm bound that holds however long you watch.
10. [10_when_tomorrow_remembers_today.ipynb](10_when_tomorrow_remembers_today.ipynb) —
    time series meet the course: autocorrelation, the four simple rules any model must beat
    (last value, same time yesterday, same time last week, 14-day median), the past-percentile
    reference band, and misses that arrive in bursts.
11. [11_the_honest_backtest_walk_forward.ipynb](11_the_honest_backtest_walk_forward.ipynb) —
    walk-forward evaluation with refits and a purge, two leakage tests, one band per horizon,
    the interval score, and why 90% per hour is far from 90% per day.
12. [12_wrapping_any_forecaster_cqr_and_pretrained.ipynb](12_wrapping_any_forecaster_cqr_and_pretrained.ipynb) —
    CQR around gradient boosting and around a zero-shot "pre-trained" stand-in, the
    pre-training overlap leak committed live, an ensemble, and the benchmark table.
13. [13_frozen_sensors_stuck_values.ipynb](13_frozen_sensors_stuck_values.ipynb) —
    detecting stuck values with a minimum-run-length rule, the innocent flat lines that fool
    it, and what frozen data does to error, coverage and band width.
14. [14_band_edges_as_alarm_limits.ipynb](14_band_edges_as_alarm_limits.ipynb) —
    band edges as alarm limits on injected freezes, steps and drifts: false alarms per day,
    detection delay, the forecaster that chases the fault, a standard detector, and choosing
    the band level by cost.
15. [15_coverage_under_drift_aci_and_pid.ipynb](15_coverage_under_drift_aci_and_pid.ipynb) —
    coverage under drift: split CQR against rolling and recency-weighted recalibration,
    adaptive conformal inference (ACI) and conformal PID, with long-run and worst-week coverage.

## Part III: language models that know what they do not know

Part III follows Quill Learning, a fictional education company whose AI study assistant answers
exam questions, writes short explainers and answers questions from a library of encyclopedia
passages. The data is real: answers, probabilities and samples from `gpt-4.1-mini`, generated once
and cached in [llm_data/](llm_data/README.md), so the notebooks run offline.

16. [16_the_confident_chatbot.ipynb](16_the_confident_chatbot.ipynb) —
    three ways a language model reports certainty (token probability, stated confidence, agreement
    of samples), how well each is calibrated, and what must stay fixed for a calibration to carry
    over: model, prompt, decoding, option order, population.
17. [17_multiple_choice_answer_sets.ipynb](17_multiple_choice_answer_sets.ipynb) —
    notebook 04's prediction sets with a language model's option probabilities, what to do when an
    API gives no probabilities, and coverage subject by subject.
18. [18_conformal_factuality.ipynb](18_conformal_factuality.ipynb) —
    keep only the claims of a generated explainer that you can vouch for, so that all kept claims
    are true with probability at least 1 - alpha, and why the claim score decides how much survives.
19. [19_retrieval_risk_control_rag.ipynb](19_retrieval_risk_control_rag.ipynb) —
    how many passages to retrieve: a coverage promise for the evidence, conformal risk control for
    the share of evidence missed, and Learn then Test for answer quality.

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

All Hearthstone, Cairn Health and Kestrel data is fully synthetic, generated in-notebook by
seeded functions whose docstrings state exactly what is baked in — the simulators are part of
the teaching material, and later notebooks test claims the docstrings make. Part III is the
exception: it reads real language-model outputs cached in `llm_data/`, whose README records
the model, the prompts, the sources and their licences.

| Data | Used by | Story |
|------|---------|-------|
| `simulate_hearthstone_listings` (synthetic) | 01, 02, 03, 05, 06, 07 | Home listings priced by Hearthstone's instant-offer desk |
| Synthetic triage clusters | 04 | Cairn Health's shortlist-for-the-nurse triage assistant |
| `load_digits` (bundled with sklearn) | 04 | The set-size-as-difficulty effect on real images |
| `fetch_california_housing` (downloads once) | 08 | Real district-level house values for the MAPIE finale |
| `load_wine` (bundled with sklearn) | 08 | A deliberately small classification set for cross-conformal |
| `simulate_building_load`, `simulate_building_portfolio` (synthetic) | 09-15 | Hourly electricity of Kestrel's office buildings; counterpart: Building Data Genome 2 |
| `simulate_vm_cpu`, `simulate_vm_fleet` (synthetic) | 10-15 | 5-minute CPU load of Kestrel's virtual machines; counterpart: public cloud traces such as the Azure Public Dataset |
| `llm_data/mmlu_*.csv` (real model outputs, MIT-licensed questions) | 16, 17 | Quill's answers to exam questions from 28 non-medical MMLU subjects |
| `llm_data/element_claims.csv` (real model outputs, PubChem ground truth) | 18 | Quill's explainers about the chemical elements, graded claim by claim |
| `llm_data/hotpot_*.csv` (real model outputs, CC BY-SA 4.0 passages) | 19 | Retrieval and answers over HotpotQA's encyclopedia passages |

## Run

The notebooks are committed fully executed, so you can read them on GitHub without running
anything. To re-run them, from the repository root:

```bash
python -m venv .venv && source .venv/bin/activate
pip install numpy pandas scikit-learn matplotlib mapie jupyter ipykernel
jupyter nbconvert --to notebook --execute --inplace notebooks/01_*.ipynb   # or open in Jupyter
```

Every notebook is self-contained (each repeats its own setup and data simulation), seeds
all randomness, and runs end-to-end in under about two minutes on a laptop; each Part II
notebook takes between 7 and 35 seconds (the model-heavy 11-14 limit scikit-learn to one
thread per fit, which keeps them fast even when the machine is busy). Notebook 08 downloads the
California housing data on first run; everything else is offline.
