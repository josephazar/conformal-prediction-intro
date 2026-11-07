# Project guide: calibrated forecasts and alarms for energy, IoT and cloud time series

This guide is for a student starting a project on prediction bands for operational time
series (building electricity, cloud CPU, IoT sensors) who has never worked with conformal
prediction, uncertainty quantification or exchangeability. It maps the usual objectives of
such a project to the notebooks in this folder, tells you which code to start from, and
explains how to move from the simulated data used here to the public datasets.

The short version: read the core of Part I to understand what a calibrated band is and why it
works, read all of Part II to learn how it behaves on ordered data, then copy the functions
from Part II into your own pipeline and swap the simulators for real series.

## 1. Reading path

Allow about two weeks for this path before writing project code.

| Step | Notebooks | What you get out of it |
|---|---|---|
| 1. What a band promises | 01, 02 | Point forecast vs band, coverage, alpha, split conformal from scratch, the corrected quantile |
| 2. Why it works, and when it does not | 03 | The rank argument, exchangeability, the coverage lottery, marginal vs conditional coverage |
| 3. Bands that breathe | 05 | Conformalized quantile regression (CQR), the method a forecasting project usually builds on |
| 4. When the world shifts | 07 | Covariate shift, weighted conformal, covariate shift vs concept drift |
| 5. The assumption, tested | 09 | Exchangeable vs independent, the six ways data stops being exchangeable, three tests including the conformal test martingale |
| 6. Forecasting basics | 10 | Persistence, seasonal naive, the 14-day median, the percentile reference band, why misses cluster |
| 7. The protocol | 11 | Walk-forward evaluation, the leakage test, per-horizon calibration, the interval score, joint coverage |
| 8. Models with bands | 12 | Gradient boosting + CQR, a pre-trained stand-in used zero-shot, pre-training overlap, ensembles, the benchmark table |
| 9. Dirty data | 13 | Stuck-value detection with a minimum-run-length rule, and what frozen data does to error and coverage |
| 10. Alarms | 14 | Band edges as alarm limits, injected faults, detection delay and false alarms against a standard detector |
| 11. Drift | 15 | Split CQR vs rolling and recency-weighted recalibration vs ACI vs conformal PID |

Optional: 04 (prediction sets for classifiers; useful for the label-shift example in 09),
06 (CV+ and jackknife+ when data is scarce), 08 (the MAPIE library), bonus 02 (EnbPI through
MAPIE's time-series wrapper). Keep [FIELD_GUIDE.md](FIELD_GUIDE.md) open while you work:
sections §13-§23 are the traps specific to this kind of project.

## 2. Objectives mapped to notebooks

| Typical project objective | Notebook | Code to start from | Field guide |
|---|---|---|---|
| Leakage-free evaluation: rolling-origin blocks, refit before each block, bands calibrated on earlier data | 11 | `walk_forward_blocks`, the h-step purge, the leakage test (shuffle the series in time) and the future-scramble check | §15, §16 |
| Simple baselines: last value, same hour yesterday / last week, 14-day median of the same hour | 10 | `make_direct_table` (the baselines are columns of the table) | §17 |
| A reference band from percentiles of past data | 10 | `p05_14d_same_time`, `p95_14d_same_time` columns | §1, §4 |
| Gradient boosting with a 90% CQR band | 12 | `quantile_gbm`, the CQR score `max(lo - y, y - hi)`, `corrected_quantile` | §2, §19 |
| A pre-trained model used zero-shot, wrapped with CQR, with a pre-training overlap check | 12 | the interface contract (return q05, q50, q95 per origin and horizon), the overlap demo | §23 |
| MAE, coverage, width and interval score per horizon and dataset family | 11, 12 | `band_report`, `interval_score`, the benchmark table layout | §17, §18, §19 |
| Detect stuck-value windows and measure their effect | 13 | the minimum-run-length rule, raw vs gated vs oracle pipelines | §20 |
| Inject faults and use band edges as alarm limits; delay and false alarms vs a standard detector | 14 | `inject_fault`, the alarm rules, the rolling robust z-score detector | §21 |
| Coverage under drift: split CQR vs ACI vs conformal PID | 15 | ACI and conformal PID from scratch, the worst-week report | §22 |
| Argue (and test) the exchangeability assumption | 09 | the permutation test, the conformal test martingale | §13, §14 |

Every function above is plain numpy, pandas and scikit-learn. The shared building blocks
(`simulate_building_load`, `simulate_vm_cpu`, `make_direct_table`, `quantile_gbm`,
`walk_forward_blocks`, `interval_score`, `band_report`, `corrected_quantile`) are copied
verbatim between notebooks, so you can lift them unchanged. The specialised tools each live in
the one notebook that teaches them: the permutation test and the conformal test martingale in
09, the stuck-value detector in 13, fault injection and the robust z-score detector in 14, and
the ACI and conformal PID loops in 15.

## 3. From simulated to real data

The notebooks simulate two families on purpose: you know the ground truth, so you can check
every claim. The simulators mimic the two kinds of public data a project like this uses.

| Simulated here | Real counterpart | Granularity |
|---|---|---|
| `simulate_building_load` (two years, hourly kWh, weekly rhythm, HVAC seasons) | Building Data Genome 2 (BDG2) | hourly, 2016-2017 |
| `simulate_vm_cpu` (30 days, 5-minute CPU %, bursts, redeploy level shifts) | Azure Public Dataset VM CPU traces (2017 and 2019 releases); also Google ClusterData 2019 and Alibaba 2018 traces | 5 minutes |

**The data contract.** Every Part II function takes one series as a pandas `Series` with a
regular `DatetimeIndex` (no gaps, one value per step). Get each real series into that shape
first: resample to the grid, decide explicitly what to do with missing steps (do not
forward-fill silently: that manufactures stuck values, see §20), then call
`make_direct_table(y, h, period)` with `period = 24` for hourly data or `288` for 5-minute data.

**Building Data Genome 2.** Repository: `github.com/buds-lab/building-data-genome-project-2`.
Electricity is in `data/meters/cleaned/electricity_cleaned.csv` (about 175 MB), metadata in
`data/metadata/metadata.csv`, weather in `data/weather/`. The files are stored with Git LFS:
cloning without LFS, or downloading the "raw" link, gives you a 130-byte pointer file instead
of data. Use `git lfs pull` after cloning, or the `media.githubusercontent.com/media/...` link.
The meter file is wide: a `timestamp` column, then one column per building, named
`Site_usage_Name` (for example `Robin_office_Dina`), so the site is the first token of the
column name. The file covers 19 sites; four of them (Bear, Fox, Panther, Rat) are in a common
pre-training corpus (see the overlap note below), so filter by site before any zero-shot test.

```python
# display-only sketch: one BDG2 building as a Series for the Part II functions
meters = pd.read_csv("electricity_cleaned.csv", parse_dates=["timestamp"], index_col="timestamp")
site = meters.columns.str.split("_").str[0]
seen_in_pretraining = site.isin(["Bear", "Fox", "Panther", "Rat"])  # check your model's own list
unseen = meters.loc[:, ~seen_in_pretraining]
y = unseen["Robin_office_Dina"].asfreq("h")           # regular hourly grid; NaN where missing
```

**Azure Public Dataset.** Repository: `github.com/Azure/AzurePublicDataset`
(`AzurePublicDatasetV1.md` for 2017, `AzurePublicDatasetV2.md` for 2019; download links in
`AzurePublicDatasetV1Links.txt` and `AzurePublicDatasetLinksV2.txt`). The CPU readings are
split into 125 files (2017, roughly 0.7 GB each, gzipped) and 195 files (2019, roughly 0.86 GB
each). Each reading has a timestamp in seconds (every 5 minutes), an encrypted VM id and the
minimum, maximum and average CPU of the interval; check the column order against
`schema.csv` before parsing. Never load a whole release: stream one file, pick a sample of VMs
that have complete 30-day histories, and save those series. The min/max/avg triple helps the
stuck-value detector of notebook 13 against freezes away from the limits: a frozen gateway
repeats all three, while a busy VM's minimum and maximum keep moving. It does not help at exact
saturation, where an average of 100 forces the minimum and maximum to 100 as well; there, use
another signal such as memory or network columns, or the physical-limit guard of notebook 13.

**Pre-training overlap.** For this data the overlap is not hypothetical. The LOTSA
collection on Hugging Face (`Salesforce/lotsa_data`, used to pre-train Moirai) contains
`azure_vm_traces_2017`, `alibaba_cluster_trace_2018`, `borg_cluster_data_2011` and four BDG2
sites (`bdg-2_bear`, `bdg-2_fox`, `bdg-2_panther`, `bdg-2_rat`), plus the synthetic
`buildings_900k`. The TimesFM 2.0 model card lists `azure_vm_traces`, `borg_cluster_data` and
`buildings` from LOTSA among its pre-training datasets (its `buildings` is most likely the
synthetic set, not BDG2), and the same card states that its quantile heads have not been
calibrated after pre-training. Before reporting any
zero-shot score, check the pre-training data list published with the exact model version you
use, prefer releases or sites that are not listed, and report the overlap when you cannot
avoid it. Notebook 12 shows the size of the effect: putting the target series in the
pre-training pool flattered the interval score by 6% to 47% while coverage stayed near 0.90,
so a coverage check alone will not catch it (§23).

**Fault labels.** These datasets carry no labelled faults, which is why notebook 14 injects
them. Say so in your report: injected faults are usually cleaner than real ones.

## 4. A first-week checklist

1. Reproduce notebook 11 end to end and change one protocol constant (block length or
   calibration length); watch which numbers move.
2. Load one BDG2 building and one Azure VM into the data contract above.
3. Run the four baselines and the percentile band on both (notebook 10's code).
4. Run both leakage checks from notebook 11 on your pipeline before trusting any model
   number: the shuffle test and the future-scramble check (§16).
5. Write down, in one sentence, at which level you will argue exchangeability, and run the
   permutation test or the conformal test martingale from notebook 09 on your calibration
   scores (§13, §14).

## 5. What to read next

| Paper | Why | Notebook |
|---|---|---|
| Angelopoulos and Bates (2023), *A gentle introduction to conformal prediction* | the general recipe and its extensions | Part I |
| Romano, Patterson and Candes (2019), *Conformalized quantile regression* | the band method most forecasting projects build on | 05, 12 |
| Gneiting and Raftery (2007), *Strictly proper scoring rules, prediction, and estimation* | the interval score | 11 |
| Gibbs and Candes (2021), *Adaptive conformal inference under distribution shift* | ACI | 15 |
| Zaffran et al. (2022), *Adaptive conformal predictions for time series* | ACI's step size, AgACI, why online split conformal loses coverage under dependence | 15 |
| Angelopoulos, Candes and Tibshirani (2023), *Conformal PID control for time series prediction* | quantile tracking, integrator, scorecaster | 15 |
| Barber, Candes, Ramdas and Tibshirani (2023), *Conformal prediction beyond exchangeability* | recency-weighted conformal and its coverage-gap bound | 09, 15 |
| Xu and Xie (2021), *Conformal prediction interval for dynamic time-series* | EnbPI | bonus 02 |
| Foygel Barber et al. (2021), *The limits of distribution-free conditional predictive inference* | why no method can promise coverage at a given hour or building | 03, 15 |
| Vovk, Gammerman and Shafer (2005), *Algorithmic learning in a random world*, chapter 7 | testing exchangeability with conformal martingales | 09 |
