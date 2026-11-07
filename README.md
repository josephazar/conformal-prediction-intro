# Conformal Prediction Intro

A hands-on notebook course on conformal prediction for junior AI engineers and data
scientists. The project teaches how to turn point predictions into auditable uncertainty
intervals and prediction sets, how to diagnose the ways those promises fail, and how to
ship the methods with MAPIE without treating the library as a substitute for judgment.

The course is intentionally practical: every idea is introduced through a product scenario,
then tested in code, then translated into the language a team would use in a launch review.
The main story follows Hearthstone, a fictional property-tech startup, with a short detour
through Cairn Health for classification and triage. Part II follows Kestrel Operations, a
fictional operator of office buildings and a cloud platform, and takes the methods to time
series: exchangeability tested rather than assumed, honest backtests, calibrated forecast
bands, frozen sensors, alarms, and coverage under drift. Part III follows Quill Learning, a
fictional education company, and applies the same ideas to language models with real model
outputs: calibrated confidence, answer shortlists, factual claims you can vouch for,
retrieval with a guarantee, and a certified error rate for the answers it gives on its own.

## What Is Inside

- `notebooks/` - Part I (notebooks 01-08), from point-prediction failure to MAPIE;
  Part II (notebooks 09-15), forecasts and alarms you can trust; Part III (notebooks 16-20),
  language models that know what they do not know.
- `notebooks/llm_data/` - cached outputs of `gpt-4.1-mini` used by Part III, with the notebook
  that generated them and their provenance and licences.
- `notebooks/PROJECT_GUIDE.md` - for students starting a project on calibrated forecasts for
  energy, IoT or cloud time series: objectives mapped to notebooks, and how to move to public
  data (Building Data Genome 2, the Azure Public Dataset).
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

Part II, forecasts and alarms you can trust:

| Step | Notebook | Topic |
|---:|---|---|
| 9 | `09_exchangeable_or_not.ipynb` | Exchangeable vs independent, six violations, and how to test the assumption |
| 10 | `10_when_tomorrow_remembers_today.ipynb` | Time series, simple baselines, the percentile reference band, clustered misses |
| 11 | `11_the_honest_backtest_walk_forward.ipynb` | Walk-forward evaluation, leakage tests, per-horizon bands, the interval score |
| 12 | `12_wrapping_any_forecaster_cqr_and_pretrained.ipynb` | CQR around boosting and a zero-shot pre-trained stand-in, pre-training overlap, the benchmark table |
| 13 | `13_frozen_sensors_stuck_values.ipynb` | Stuck-value detection and its effect on error, coverage and width |
| 14 | `14_band_edges_as_alarm_limits.ipynb` | Band edges as fault-detection limits against a standard detector |
| 15 | `15_coverage_under_drift_aci_and_pid.ipynb` | Coverage under drift: recalibration, ACI, and conformal PID |

Part III, language models that know what they do not know:

| Step | Notebook | Topic |
|---:|---|---|
| 16 | `16_the_confident_chatbot.ipynb` | Token probabilities, stated confidence, calibration, and exchangeability for language models |
| 17 | `17_multiple_choice_answer_sets.ipynb` | Multiple-choice answers as prediction sets, with and without probabilities |
| 18 | `18_conformal_factuality.ipynb` | Keeping only the generated claims you can vouch for |
| 19 | `19_retrieval_risk_control_rag.ipynb` | Retrieval depth for RAG: conformal risk control and Learn then Test |
| 20 | `20_answer_or_hand_off.ipynb` | Answer alone or hand off: a certified error rate among the answers given, with Learn then Test |

Bonus notebooks:

| Bonus | Notebook | Topic |
|---:|---|---|
| 1 | `notebooks/bonus/01_regression_with_mapie.ipynb` | MAPIE regression in the Hearthstone setting |
| 2 | `notebooks/bonus/02_time_series_forecasting_with_mapie.ipynb` | One-step-ahead forecasting with MAPIE time-series intervals |
| 3 | `notebooks/bonus/03_coverage_gallery.ipynb` | A compact visual gallery for marginal, local, and failed coverage |
| 4 | `notebooks/bonus/04_decision_policy_from_set_size.ipynb` | Turning singleton, multi-label, and empty sets into workflow policy |

## How To Run

Create a virtual environment and install the pinned notebook dependencies (tested with
Python 3.10):

```bash
python -m venv .venv
source .venv/bin/activate
python -m pip install --upgrade pip
python -m pip install -r requirements.txt
```

Start Jupyter:

```bash
jupyter lab
```

The notebooks are self-contained. Most data is synthetic and generated inside each
notebook with fixed seeds. Notebook 08 uses public scikit-learn datasets, including
California housing and wine. Notebooks 16-20 read cached language-model outputs from
`notebooks/llm_data/`, so they also run offline and need no API key; only regenerating that
data (`notebooks/llm_data/generate_llm_data.ipynb`) needs an Azure OpenAI deployment and the
extra packages in `requirements-llm.txt`.

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
- Ilia Nouretdinov, Thomas Melluish, and Vladimir Vovk. "Ridge Regression Confidence Machine."
  ICML, 2001.
- Matteo Gasparin and Aaditya Ramdas. "Merging Uncertainty Sets via Majority Vote." 2024.
- Vladimir Vovk. "Conditional Validity of Inductive Conformal Predictors." ACML, 2012.
- Jing Lei and Larry Wasserman. "Distribution-Free Prediction Bands for Non-parametric
  Regression." Journal of the Royal Statistical Society Series B, 2014.
- Isaac Gibbs, John J. Cherian, and Emmanuel J. Candes. "Conformal Prediction with
  Conditional Guarantees." 2023.
- Chen Xu and Yao Xie. "Conformal Prediction Interval for Dynamic Time-Series." ICML, 2021.
- Vianney Taquet, Vincent Blot, Thomas Morzadec, Louis Lacombe, and Nicolas Brunel.
  "MAPIE: an Open-Source Library for Distribution-Free Uncertainty Quantification." 2022.

Part II also draws on:

- Tilmann Gneiting and Adrian E. Raftery. "Strictly Proper Scoring Rules, Prediction, and
  Estimation." Journal of the American Statistical Association, 2007.
- Isaac Gibbs and Emmanuel Candes. "Adaptive Conformal Inference Under Distribution Shift."
  NeurIPS, 2021.
- Margaux Zaffran, Olivier Feron, Yannig Goude, Julie Josse, and Aymeric Dieuleveut.
  "Adaptive Conformal Predictions for Time Series." ICML, 2022.
- Anastasios N. Angelopoulos, Emmanuel J. Candes, and Ryan J. Tibshirani. "Conformal PID
  Control for Time Series Prediction." NeurIPS, 2023.
- Rina Foygel Barber, Emmanuel J. Candes, Aaditya Ramdas, and Ryan J. Tibshirani.
  "Conformal Prediction Beyond Exchangeability." Annals of Statistics, 2023.
- Rina Foygel Barber, Emmanuel J. Candes, Aaditya Ramdas, and Ryan J. Tibshirani. "The
  Limits of Distribution-Free Conditional Predictive Inference." Information and Inference,
  2021.
- Stephen Bates, Emmanuel Candes, Lihua Lei, Yaniv Romano, and Matteo Sesia. "Testing for
  Outliers with Conformal p-values." Annals of Statistics, 2023.

Part III also draws on:

- Bhawesh Kumar, Charlie Lu, Gauri Gupta, Anil Palepu, David Bellamy, Ramesh Raskar, and
  Andrew Beam. "Conformal Prediction with Large Language Models for Multi-Choice Question
  Answering." 2023.
- Christopher Mohri and Tatsunori Hashimoto. "Language Models with Conformal Factuality
  Guarantees." ICML, 2024.
- Anastasios N. Angelopoulos, Stephen Bates, Adam Fisch, Lihua Lei, and Tal Schuster.
  "Conformal Risk Control." ICLR, 2024.
- Anastasios N. Angelopoulos, Stephen Bates, Emmanuel J. Candes, Michael I. Jordan, and
  Lihua Lei. "Learn then Test: Calibrating Predictive Algorithms to Achieve Risk Control." 2021.
- John J. Cherian, Isaac Gibbs, and Emmanuel J. Candes. "Large Language Model Validity via
  Enhanced Conformal Prediction Methods." NeurIPS, 2024.
- Yonatan Geifman and Ran El-Yaniv. "Selective Classification for Deep Neural Networks."
  NeurIPS, 2017.
- Stephen Bates, Anastasios Angelopoulos, Lihua Lei, Jitendra Malik, and Michael I. Jordan.
  "Distribution-Free, Risk-Controlling Prediction Sets." Journal of the ACM, 2021.
- Dan Hendrycks, Collin Burns, Steven Basart, Andy Zou, Mantas Mazeika, Dawn Song, and Jacob
  Steinhardt. "Measuring Massive Multitask Language Understanding." ICLR, 2021 (MMLU).
- Zhilin Yang, Peng Qi, Saizheng Zhang, Yoshua Bengio, William W. Cohen, Ruslan Salakhutdinov,
  and Christopher D. Manning. "HotpotQA: A Dataset for Diverse, Explainable Multi-hop Question
  Answering." EMNLP, 2018.

## License

- Code (the Python in the notebooks): MIT, see [LICENSE](LICENSE).
- Course text and figures (the notebook explanations, the field guide, the project guide and
  the glossary): [Creative Commons Attribution 4.0 International (CC BY 4.0)](https://creativecommons.org/licenses/by/4.0/).
- Data in `notebooks/llm_data/`: the `hotpot_*.csv` files adapt HotpotQA, which is built
  from Wikipedia, and are shared under
  [CC BY-SA 4.0](https://creativecommons.org/licenses/by-sa/4.0/), as their source requires;
  the MMLU questions are MIT-licensed and the PubChem values are public domain. See
  [notebooks/llm_data/README.md](notebooks/llm_data/README.md) for attribution.
- Third-party datasets and passages that the notebooks load or quote keep their own licences.
