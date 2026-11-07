# Conformal prediction field guide

A numbered list of the traps that actually bite people who ship conformal prediction, in
the spirit of a consultant's case notes. The notebooks in this folder cite these sections
as "field guide §k" — the numbering is stable, do not reorder it. Each entry gives the
trap, the tell-tale symptom, and the guard.

## §1. The off-by-one quantile

The guarantee needs at least the ceil((n+1)(1-alpha))-th smallest calibration score, not the plain
(1-alpha) sample quantile. The difference looks cosmetic and is not: the naive quantile
systematically undercovers, worst at small n.

Symptom: empirical coverage sits consistently a little below target (say 0.888 for a 0.90
promise) and gets worse the smaller your calibration set.

Guard: take the k-th smallest score, k = ceil((n + 1) * (1 - alpha)), as every notebook does.

```python
k = math.ceil((n + 1) * (1 - alpha))
q_hat = np.sort(scores)[k - 1]                        # the k-th smallest calibration score
```

With distinct scores this rule covers a new point with probability exactly k/(n+1), which
lies between 1 - alpha and 1 - alpha + 1/(n+1). Beware the shortcuts that look equivalent
(notebook 02, section 3): `np.quantile(scores, 1 - alpha)` interpolates and can land below the
k-th score, which is invalid; `np.quantile(scores, k / n, method="higher")` returns the
(k+1)-th smallest whenever k < n, which is valid but spends one rank for nothing, a cost that
is invisible at n in the thousands and worth points of coverage at n = 40 (notebook 06).
Libraries differ, and even differ between their own classes: in MAPIE 1.4.1 the split and
cross-conformal regressors take the exact rank, while the split classifiers and conformalized
quantile regression (which by default corrects each bound at alpha/2) can land one rank higher;
at alpha = 0.10 that happens exactly when (n + 1)(1 - alpha), or (n + 1)(1 - alpha/2) for each
CQR bound, is a whole number below n (notebook 08). MAPIE 1.6.0 takes the exact rank in all of them.
Check which rank your version returns.

And if `ceil((n + 1) * (1 - alpha)) > n`, stop: your calibration set is too small for this
alpha, and the only honest prediction set is the whole outcome space: the entire real line in
regression, every label in classification. Assert on it; do not clip it.

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
inputs and 70% on the hard slice your business actually worries about. Two stronger promises
are worth telling apart. Coverage within a few groups fixed in advance and visible when you
predict (a region, a customer tier, an exam subject) is achievable: calibrate each group on
its own data (notebook 17), and expect wider sets for small groups; overlapping groups need a
method built for them (Gibbs, Cherian and Candes 2023). Per-class coverage is achievable too,
by checking each candidate label against its own class's threshold (notebook 04). Coverage at
every individual input is not achievable: with continuous features, no distribution-free
method can promise it with finite intervals (that is a theorem, not a caveat). Adaptive scores (notebook 05) and per-group calibration close most of
the practical gap.

Symptom: overall dashboards green, one segment's complaints rising.

Guard: before shipping, print a coverage-by-slice table for every slice that appears in a
contract, a regulation, or a support queue.

## §4. Your calibration set is a lottery ticket

Realized long-run coverage given one fixed calibration set is a random variable — roughly
Beta-distributed around the target. With 50 calibration points, a "90%" system routinely
delivers 83% or 96%; with ~1000 points a 90% target typically lands in 0.88-0.92.

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
human. Set size is your triage signal (notebook 04). If singletons act without a human,
measure their error rate on its own: 90% coverage does not make the singletons 90% right
(§29 and notebook 20 show why, and how to certify such a rate).

## §8. Do not average many splits

Combining intervals from many random calibration splits without a correction carries no
guarantee: averaged endpoints have none, and the intersection of N intervals at 90% each is
guaranteed only 1 - N alpha (nothing at all for N = 10). The valid Bonferroni fix runs each
split at alpha/N and intersects; under the assumptions of Lei et al. (2018, Theorem 2.4) the
result is, with probability tending to one, wider than a single split. A majority vote over
the split sets keeps coverage of at least 1 - 2 alpha under any dependence between them
(Gasparin and Ramdas 2024), but no combination is free, and none is the obvious "run it ten
times and average".

Symptom: a well-meaning teammate proposes "run it 10 times and combine".

Guard: one split, sized deliberately. If split waste hurts, use jackknife+ (proved floor
1 - 2 alpha) or CV+ (proved floor 1 - 2 alpha - min{2(1 - 1/K)/(n/K + 1), (1 - K/n)/(K + 1)},
0.717 at n = 120, K = 10 equal folds and alpha = 0.10; Barber et al. 2021, Theorem 4), both of which
usually land near the target in practice (notebook 06).

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

---

# Part II: exchangeability, forecasting and alarms (notebooks 09-15)

## §13. Exchangeable at which level?

Exchangeable is weaker than independent: rows that share a hidden common cause (the same
building, the same batch, the same month) can be correlated and still exchangeable with each
other. What matters is how the calibration rows and the future rows were chosen. Rows of one
building can be exchangeable with each other while a new building is not exchangeable with
them; buildings can be exchangeable with each other while their rows are not.

Symptom: coverage averaged over many sites is on target, and any single site is not.

Guard: write down the level at which the shuffle is plausible (rows, days, buildings, VMs)
and split and calibrate at that level. Notebook 09 shows both cases.

## §14. Test before you quote

The guarantee holds under an assumption you can check. Plot the scores over time; run a
permutation test between the calibration window and recent data; train a classifier to tell
the two apart (notebook 07); or run a conformal test martingale, whose false-alarm
probability is bounded by Ville's inequality no matter how long you watch. A test that stays
quiet is evidence, not proof: it says no shift large enough to detect showed up. Summarizing a
dependent stream (daily means, say) does not make it exchangeable either.

Symptom: a guarantee quoted in a report with no evidence about the assumption behind it.

Guard: quote the guarantee after the test, not instead of it, and keep the test running in
production (notebook 09).

## §15. Walk forward or do not report

A random train/test split of time-ordered rows lets the model learn from the neighbours of
every test point; scaling or tuning on the full series lets it learn from the future. Both
produce numbers that production will not reproduce.

Symptom: backtest error far below live error.

Guard: rolling-origin blocks in time order, model re-fitted before each block, bands
calibrated on data that ends before the block starts, and an h-step purge so no training or
calibration target lies after the forecast origin (notebook 11).

## §16. Run the leakage test

If you shuffle the series in time, an honest pipeline loses all of its skill. A pipeline that
keeps its skill after the shuffle is reading the answer from somewhere. The converse does not
hold: a feature that leaks from the target's neighbours (a centred rolling mean) can lose its
skill under the shuffle too, because the shuffle destroys the neighbours it was leaking from.

Symptom: a model that "forecasts" shuffled noise, or a backtest that beats production by a
margin nobody can explain.

Guard: run the shuffle once per pipeline change and report skill relative to the best
constant forecast (the training median, for absolute error) before and after. Add the
future-scramble check, which needs no model at all: change every value after the forecast
origin and confirm that no feature of that row changes. Change the values, do not merely
reorder them: a reordering leaves the full-series mean, median, minimum and maximum untouched,
so a scaler fitted on the whole series would pass it. Notebook 11 shows a leak that only the
second test catches.

## §17. Beat the best simple rule, not the worst

Persistence (last value), seasonal naive (same time yesterday or last week) and the 14-day
median of the same time are hard to beat on regular series, and on bursty cloud series they
often win. A model is worth deploying only if it beats the best of them, per horizon and per
series family.

Symptom: a model compared with a single, weak baseline.

Guard: report all four rules for every family and horizon, and carry the best one into the
benchmark table, chosen on data before the test period. Picking the rule on the test period
itself is notebook 11's lie number three in a new costume: it flatters the rule. Show the
hindsight-best rule as a stress test if you like, labelled as such. "The model did not win" is
a valid result (notebooks 10, 11 and 12).

## §18. Report the interval score next to coverage

Coverage can always be bought with width. The interval score (Gneiting and Raftery, 2007)
charges for both: width plus 2/alpha times the distance of every miss.

Symptom: a method wins on coverage by being uselessly wide, or wins on width by missing.

Guard: report MAE, coverage, width and interval score together (notebook 11).

## §19. One band per horizon, and hourly is not daily

Forecast errors grow with the horizon, so each horizon needs its own calibration. Even then,
90% per hour does not make 90% per day: the share of days whose every hour is covered is far
lower.

Symptom: a planner surprised that "90% bands" leave most days with at least one miss.

Guard: calibrate per horizon; report joint coverage when a decision depends on a whole
trajectory (notebook 11).

## §20. Frozen values masquerade as data

A gateway that repeats its last value produces readings that look perfectly plausible. They
flatter persistence, and they distort the band in either direction: frozen calibration rows
that are easy to predict shrink it, while models trained on frozen rows can widen it a great
deal. Either way, the guarantee then covers what the sensor reported rather than what the
building did. Some flat lines are innocent: a saturated VM at exactly 100%, a meter that rounds
to whole kWh at night.

Symptom: a long run of identical readings, a baseline that is suspiciously good, or a band
that changes width sharply once flagged rows are removed.

Guard: a minimum-run-length rule with a tolerance, checked against innocent flat lines;
drop flagged rows from training and calibration; report evaluation with and without them
(notebook 13).

## §21. A band edge is an alarm with a false-alarm rate of alpha per reading

On healthy data a reading leaves a calibrated 90% band one time in ten. At hourly readings
that is more than two alarms per meter per day. And a forecaster that uses the latest
readings follows a fault and stops alarming.

Symptom: alarm fatigue, or slow drifts and freezes that never trigger.

Guard: choose the band level by the cost of false alarms and missed faults; require
consecutive exceedances; report false alarms both per flagged reading (what alpha caps) and per
alarm event (what the pager sees). Know the horizon trade-off: a forecast issued before the
fault began does not chase it, so it keeps alarming once the fault is under way, but its band is
wider, so it can catch the onset later than a next-step band. Measure both, and pair the band
with a stuck-value detector (notebooks 13 and 14).

## §22. Online calibration guarantees the long run, not this week

ACI and conformal PID bound the average miss rate over a long stream, for any sequence, with
no exchangeability assumption. They promise nothing about a given week. ACI can return
infinite bands, and both methods can return empty ones: whenever the offset falls below minus
half the raw band's width (for ACI also when its working alpha reaches 1), the lower edge lands
above the upper (notebook 15: up to 114 hours, depending on the variant). An empty
band shows no range at all for that hour, a guaranteed miss.

Symptom: long-run coverage on target and a bad week that someone asks about.

Guard: report weekly (worst-week) coverage, width, and the counts of infinite and empty bands
next to long-run coverage. Average width and interval score over the finite, non-empty bands
only, and say so; count empty bands as misses, and never swap or clip them away. Dropping a
method's worst hours from a score can flatter it, so check that the ranking holds on the hours
where every method issued a finite, non-empty band (notebook 15).

## §23. Pre-training overlap is leakage

A pre-trained forecaster evaluated zero-shot on a series that was part of its training data
is being tested on its homework. Several public cloud and building traces appear in the
pre-training corpora of popular time-series foundation models.

Symptom: a zero-shot model that looks far better on some datasets than on others.

Guard: check the model's published pre-training data lists; evaluate on series released
after the model or not listed; otherwise report the overlap (notebook 12).

---

# Part III: language models (notebooks 16-20)

## §24. Token probabilities, stated confidence and agreement are opinions

A language model can tell you how sure it is in three ways: the probability it assigns to its
answer, a confidence number it writes when asked, and how often repeated samples agree. All
three are useful and none is a promise. Stated confidence in particular tends to crowd near the
top of the scale whatever the truth.

Symptom: "the model said 95%" quoted as the chance that an answer is right.

Guard: check calibration with a reliability diagram, then conformalize whichever score ranks
easy and hard questions best. Even a perfectly calibrated probability promises nothing about a
single question; a conformal set does promise coverage (notebooks 16 and 17).

## §25. Freeze the procedure

A calibration is valid for one procedure: one model version, one prompt, one set of decoding
settings, one option order, one population of questions. Change any of them and the calibration
scores may no longer be exchangeable with the new answers. Hosted models are not even
bit-for-bit repeatable: the same prompt and settings can come back with different probabilities.

Symptom: coverage drifts after a "harmless" prompt edit or a silent model upgrade.

Guard: log the procedure with every calibration run (model version string, prompt hash,
decoding settings, option order, date, population); recalibrate after any change; measure how
much the same prompt, sent again, varies (notebook 16).

## §26. The benchmark is not your users

Public benchmarks are convenient calibration sets, and some of them have probably been seen in
training. Coverage measured on a benchmark transfers only to questions exchangeable with it, and
your users' questions usually are not.

Symptom: a guarantee quoted from a benchmark run and applied to live traffic.

Guard: calibrate on a sample of your own traffic, labelled; treat benchmark results as a
rehearsal (notebooks 09 and 16).

## §27. A factuality guarantee is about whole answers, and only as good as its grader

Conformal factuality promises that, with probability at least 1 - alpha, every claim kept in an
answer is true. It says nothing about any single claim, and "true" means "accepted by the
grader": a wrong reference value or an ambiguous question becomes part of the guarantee. Know
what the grader reads, too: notebook 18 checks the value the model returned, not the sentence
shown to the reader, so the promise covers the sentence only as far as it repeats that value.
The promise has a price in content: at stricter levels the filter removes more true claims and
can empty whole answers.

Symptom: a filtered explainer published with a guarantee nobody can trace to its grading rules.

Guard: state the grader, its tolerances and its known ambiguities next to the guarantee; treat
answers (not claims) as the exchangeable units (notebook 18).

## §28. Controlling an expected loss is not controlling every answer

Conformal risk control bounds the average loss (for example the share of evidence missed), so
some questions can still miss a lot. It needs a loss that only improves as the parameter
loosens. When the risk can go up and down (more retrieved context can confuse a model), picking
the parameter with the best calibration score is not a certificate.

Symptom: "we control the miss rate" read as "no question misses evidence", or the best-looking
setting chosen and then reported as guaranteed.

Guard: say "on average" when you use conformal risk control; for non-monotone risks use Learn
then Test, one test per candidate setting with a family-wise error correction (notebook 19).

## §29. Coverage is not the error rate among the answers you give

A 90% shortlist promises that the true answer is in the set nine times in ten. It does not
make nine in ten of its single-option answers right: misses over all questions are at most
alpha on average, so among the questions answered alone the error can reach
alpha / P(answered alone). Picking the loosest threshold whose calibration error looks fine is
not a certificate either: it breaks the promise far more often than the stated delta.

Symptom: "90% coverage, so 90% of the answers shown alone are right", or an answer threshold
read off a calibration curve.

Guard: fix the promise (epsilon, delta) and a grid of thresholds in advance; test them strictest
first with a binomial p-value on the answered calibration questions, at the full delta, and
stop at the first failure; ship the loosest certified threshold, or hand everything off when
none is certified. The binomial test needs an i.i.d. sample of the traffic you will serve
(notebook 20).
