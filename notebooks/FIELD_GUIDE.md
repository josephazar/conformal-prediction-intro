# Conformal prediction field guide

A numbered list of the traps that actually bite people who ship conformal prediction, in
the spirit of a consultant's case notes. The notebooks in this folder cite these sections
as "field guide §k" — the numbering is stable, do not reorder it. Each entry gives the
trap, the tell-tale symptom, and the guard.

## §1. The off-by-one quantile

The guarantee needs the ceil((n+1)(1-alpha))-th smallest calibration score, not the plain
(1-alpha) sample quantile. The difference looks cosmetic and is not: the naive quantile
systematically undercovers, worst at small n.

Symptom: empirical coverage sits consistently a little below target (say 0.888 for a 0.90
promise) and gets worse the smaller your calibration set.

Guard:

```python
q_level = math.ceil((n + 1) * (1 - alpha)) / n
q_hat = np.quantile(scores, q_level, method="higher")
```

And if `ceil((n + 1) * (1 - alpha)) > n`, stop: your calibration set is too small for this
alpha, and the only honest prediction set is infinite. Assert on it; do not clip it.

## §2. Never calibrate on training residuals

A model's errors on data it trained on are optimistically small — it has memorized part of
the noise. Thresholds computed from training residuals are too tight, and coverage on fresh
data quietly drops below target. Notebook 01 commits this mistake on purpose so you can
watch it fail.

Symptom: backtest coverage looks perfect, production coverage does not.

Guard: the calibration set must be untouched by training, feature selection, and
hyperparameter tuning. Split first, fit later, calibrate once.

## §3. Marginal is not conditional

The guarantee is an average over everything: 90% overall can decompose into 98% on easy
inputs and 70% on the hard slice your business actually worries about. No conformal method
can promise exact per-input coverage distribution-free (that is a theorem, not a caveat),
but adaptive scores (notebook 05) and per-group calibration (notebook 04) close most of
the practical gap.

Symptom: overall dashboards green, one segment's complaints rising.

Guard: before shipping, print a coverage-by-slice table for every slice that appears in a
contract, a regulation, or a support queue.

## §4. Your calibration set is a lottery ticket

Realized long-run coverage given one fixed calibration set is a random variable — roughly
Beta-distributed around the target. With 50 calibration points, a "90%" system routinely
delivers 82% or 96%; with ~1000 points a 90% target typically lands in 0.88-0.92.

Symptom: two reruns of the same pipeline report meaningfully different coverage.

Guard: size the calibration set to the tolerance you can defend, and quote coverage with
its expected spread, not as a point value. Notebook 03 shows the distribution.

## §5. Validity is free, usefulness is earned

Any nonconformity score — even random noise — delivers the coverage guarantee. A terrible
model gets 90% coverage too, via enormous sets. Coverage alone therefore proves nothing
about quality; the interval width (or set size) is where model quality shows up.

Symptom: someone declares victory because coverage hit target.

Guard: always report coverage AND average width/set size together, and track width as a
model-quality diagnostic — a better model earns narrower intervals at the same coverage.

## §6. Exchangeability is the load-bearing wall

The single real assumption: calibration and test data are shuffled from the same pool. Time
order, covariate shift, feedback loops, sorted files, and train/deploy mismatch all void the
guarantee — silently. The method raises no error; the coverage just rots.

Symptom: none. That is the problem. You have to go looking.

Guard: argue exchangeability explicitly before quoting the guarantee; compare feature
distributions between calibration and production (notebook 07); treat any deliberate
ordering in your data pipeline as a red flag.

## §7. A large set is the honest answer on a hard input

A seven-label prediction set on an ambiguous case is the system working, not failing — the
model is admitting it cannot tell. Suppressing large sets (forcing top-1, capping set size)
reinstates exactly the false confidence conformal prediction was hired to remove.

Symptom: pressure to "make the sets smaller" without a plan to make the model better.

Guard: design the product around set size — auto-accept singletons, route large sets to a
human. Set size is your triage signal (notebook 04).

## §8. Do not average many splits

Averaging or intersecting intervals from many random calibration splits (with the
multiplicity correction validity requires) is provably worse — wider — than using one
split. The obvious ensembling idea is a documented dead end (Lei et al. 2018).

Symptom: a well-meaning teammate proposes "run it 10 times and combine".

Guard: one split, sized deliberately. If split waste hurts, use CV+/jackknife+ (notebook
06), which pool data with an actual theorem attached.

## §9. Check per-class coverage in classification

Overall 90% coverage can hide one class at 70% — historically, a digit classifier promised
5% errors overall while delivering more than 11% on the digit 5. Whole classes of inputs
can be systematically failed while the average looks perfect.

Symptom: per-class complaint rates differ wildly despite a healthy overall metric.

Guard: print the per-class coverage table. If a class matters on its own, calibrate
per class (Mondrian / class-conditional conformal, notebook 04) and accept slightly
larger sets.

## §10. Covariate shift is not concept drift

Weighted conformal (notebook 07) repairs a change in the input mix — which homes come to
market — only when the input-to-output relationship P(y|x) is unchanged. If the
relationship itself moved (new tax law, new customer behavior), no reweighting of old
scores can save you; you need new data and a retrain.

Symptom: weighted conformal applied after a regime change still undercovers.

Guard: before reaching for weights, write one sentence stating why P(y|x) should be stable
across the shift. If you cannot write the sentence, you have drift, not shift.

## §11. Weights shrink your effective calibration set

Importance weighting concentrates the calibration budget on look-alike points: n weighted
points behave like n_eff = (sum w)^2 / (sum w^2) unweighted ones. Aggressive shifts push
n_eff down hard, which resurrects the §4 lottery and lengthens intervals.

Symptom: after weighting, coverage is on target on average but swings run to run; intervals
noticeably wider.

Guard: always print n_eff next to n. If n_eff is small, collect calibration data where the
test distribution actually lives instead of leaning harder on weights.

## §12. Coverage alone is not an evaluation

A fixed-width oracle band can match your coverage while being useless for triage. A full
evaluation reports three things: coverage (is the promise kept), average width or set size
(how useful), and adaptivity (does size track difficulty — check that large sets fall on
genuinely hard inputs, e.g. coverage stratified by set size or slice).

Symptom: method A "beats" method B on average width, and quietly undercovers every hard
slice to do it.

Guard: the bake-off tables in notebooks 05 and 06 are the template: coverage, width, and
slice/size breakdowns, side by side, before any winner is declared.
