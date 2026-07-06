# Glossary

Plain-English terms for the conformal prediction notebooks. The goal is recall, not a
math proof.

## Core Ideas

**Conformal prediction**  
A family of methods that wrap around an existing model and return uncertainty sets or
intervals with a measurable coverage promise.

**Prediction interval**  
For regression, a lower and upper number around a prediction. Example: "this home should
sell between $480k and $690k."

**Prediction set**  
For classification, a set of possible labels. Example: `{cardiac, respiratory}` instead
of one forced class.

**Coverage**  
The fraction of future examples whose true answer lands inside the interval or set.
If 900 out of 1000 true values land inside, coverage is 90%.

**Miscoverage**  
The fraction of future examples missed by the interval or set. If coverage is 90%,
miscoverage is 10%.

**Alpha**  
The error budget. `alpha = 0.10` means you are asking for about 90% coverage.

**Confidence level**  
The coverage target, usually `1 - alpha`. MAPIE uses this term in its API.

**Distribution-free**  
The method does not require you to assume normal errors, linear models, or a known data
distribution. It still needs the calibration and future data to be comparable.

**Finite-sample guarantee**  
A promise that works for the sample size you actually have, not only in an infinite-data
limit.

## Data Roles

**Training set**  
The data used to fit the base model.

**Calibration set**  
Held-out labeled data used to measure model errors and choose the conformal threshold.
It must not be used for fitting the model.

**Test set**  
Data used only to evaluate the finished method.

**Validation set**  
Data often used for model selection or hyperparameter tuning. Do not confuse it with the
calibration set unless you are intentionally using it for calibration and nothing else.

**Exchangeability**  
The core assumption behind standard conformal prediction: calibration and future examples
are shuffled from the same population. Order should not matter.

**Covariate shift**  
The input distribution changes, but the relationship between input and output is assumed
to stay the same. Example: a housing model trained in one region is used in a new region
with larger homes.

**Concept drift**  
The relationship between input and output changes. Reweighting old calibration data is
usually not enough; you need new labeled data or a retrain.

## Scores And Quantiles

**Nonconformity score**  
A number measuring how strange or wrong a candidate answer looks to the model. Larger
usually means worse fit.

**Conformity score**  
The same idea with the sign or direction flipped. Libraries and papers vary in naming.

**Residual**  
In regression, the difference between the true value and the model prediction.

**Absolute residual**  
The size of the error, ignoring direction. This is the simplest regression score.

**Corrected quantile**  
The finite-sample conformal threshold. It uses a slightly conservative order statistic so
the coverage promise is valid.

**Naive quantile**  
The tempting ordinary quantile. It can undercover, especially with small calibration sets.

**Calibration lottery**  
The fact that one fixed calibration set gives a random realized coverage level. Small
calibration sets wobble more.

## Regression Methods

**Split conformal regression**  
Fit a model on training data, score errors on calibration data, then add the calibrated
error threshold to future predictions.

**Constant-width interval**  
An interval with the same width for every example. It can be valid but often over-covers
easy cases and under-covers hard slices.

**Adaptive interval**  
An interval whose width changes by example. It is wider where prediction is harder and
narrower where prediction is easier.

**Quantile regression**  
A model that predicts conditional quantiles, such as the 5th and 95th percentiles, instead
of only a mean.

**Conformalized quantile regression (CQR)**  
A method that starts with quantile-regression intervals and uses calibration data to repair
their coverage.

**Studentized or normalized residual**  
A residual divided by an estimate of local difficulty. This helps intervals adapt to
examples with different noise levels.

**Prediction band**  
An interval drawn across many input values, often as a shaded region in a plot.

## Classification Methods

**LAC score**  
Least ambiguous class score: roughly one minus the model probability assigned to the true
class. It is a common score for classification conformal prediction.

**Adaptive prediction sets (APS)**  
A classification method that includes labels in probability order until enough model
probability mass has been accumulated.

**RAPS**  
Regularized APS. It modifies APS to avoid overly large sets in high-class-count problems.

**Prediction-set size**  
The number of labels returned for one example. It is a useful signal: small sets usually
mean easy cases; large sets mean ambiguity.

**Singleton**  
A prediction set with exactly one label.

**Empty set**  
A prediction set with no labels. This can happen with some scores. It should trigger a
safe fallback, not a forced top-1 answer.

**Abstention**  
The model declines to make an automatic decision and routes the case elsewhere.

**Selective classification**  
A system that decides automatically on easier examples and abstains on harder ones.

**Mondrian conformal prediction**  
Conformal prediction calibrated separately within groups, such as per class or per
important segment.

**Class-conditional coverage**  
Coverage measured separately for each true class.

## Evaluation

**Marginal coverage**  
Coverage averaged over all future examples. This is the standard conformal guarantee.

**Conditional coverage**  
Coverage at a specific input or inside a narrow slice. Exact distribution-free conditional
coverage is generally impossible, but slice checks are still essential.

**Local coverage**  
Coverage measured in a neighborhood of similar examples, such as similar home sizes.

**Slice coverage**  
Coverage measured on a named subgroup: expensive homes, rare classes, new regions, and so
on.

**Average width**  
The average size of regression intervals. Lower is better only if coverage stays honest.

**Sharpness**  
How tight the intervals or sets are. Sharpness is useful only after coverage is checked.

**Adaptivity**  
Whether uncertainty gets larger on genuinely hard examples and smaller on easy ones.

**Coverage-width tradeoff**  
Higher coverage usually requires wider intervals or larger sets.

## Advanced Conformal Methods

**Full conformal prediction**  
A conformal method that avoids a calibration split by treating each candidate test label
symmetrically with the training data. It is often computationally expensive.

**Split conformal prediction**  
The practical workhorse: split data into training and calibration roles.

**Inductive conformal prediction**  
Another name often used for split conformal prediction.

**Transductive conformal prediction**  
A style of conformal prediction that reasons about each test example together with the
training examples.

**Jackknife+**  
A method that uses leave-one-out models to get prediction intervals without a simple
calibration split.

**CV+**  
A cross-validation version of jackknife+ that is often more practical than fitting one
model per training point.

**Cross-conformal prediction**  
A family of methods that use multiple folds so more data participates in both training and
calibration-like roles.

**Weighted conformal prediction**  
Conformal prediction that reweights calibration scores, often to handle covariate shift.

**Importance weight**  
A number saying how much a calibration example resembles the future distribution.

**Effective sample size**  
How many calibration examples the weighted data behaves like. Heavy weights can make a big
dataset act small.

**EnbPI**  
Ensemble batch prediction intervals, a conformal-style method for time-series forecasting
that uses bootstrap ensembles and sequential residuals.

**Block bootstrap**  
A resampling method for time series that samples chunks of consecutive observations rather
than individual shuffled rows.

**Adaptive conformal inference (ACI)**  
An online conformal method that adjusts interval size over time based on recent misses and
hits.

## Libraries And Practice

**MAPIE**  
A scikit-learn-compatible Python library for conformal prediction and uncertainty
quantification.

**Conformalize**  
MAPIE's word for calibrating a fitted model on held-out labeled data.

**Prefit**  
An API setting meaning the estimator has already been trained before it is given to the
conformal wrapper.

**Monitoring**  
Tracking coverage, width, set size, and slice behavior after launch.

**Launch checklist**  
A short operational review: alpha chosen, calibration untouched, exchangeability argued,
slices checked, hard cases routed, metrics monitored.

**Human-in-the-loop routing**  
A product design where large or empty prediction sets are sent to a human review path.
