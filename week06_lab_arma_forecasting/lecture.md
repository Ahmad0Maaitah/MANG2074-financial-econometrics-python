# Week 6 — LAB: Univariate Time-Series Modelling and Forecasting

**MANG2074 Financial Econometrics 1 · University of Southampton · Module leader: Ahmad Maaitah**

*This is a lab week: a short briefing, then hands-on work in `lab.ipynb`.*

---

## 1. From fitting to forecasting

Last week we identified and estimated ARMA models for UK house-price growth.
A model that fits well in-sample is worthless to a forecaster if it cannot
predict data it has never seen. This week we (i) automate model selection with
an information-criterion **grid search**, and (ii) evaluate genuine
**out-of-sample** forecasts: estimate on 1991–2015, forecast 2016–2018, and
score the forecasts against what actually happened.

## 2. The grid search

For every $(p, q)$ with $p, q \le 5$, fit ARMA$(p,q)$ and record AIC and BIC:

$$AIC = -2\ln\hat{L} + 2k, \qquad BIC = -2\ln\hat{L} + k \ln T.$$

Pick the minimising pair for each criterion. Expect disagreement: AIC's lighter
penalty tends to choose richer models (here ARMA(4,2)) than BIC (here AR(2)).
Convergence warnings on a few badly-conditioned $(p,q)$ pairs are normal —
suppress and ignore them; those models will not win anyway.

## 3. Static vs dynamic forecasts

With the model estimated on data up to $T_0$ (December 2015), there are two
ways to forecast the hold-out period:

- **Static (one-step-ahead)**: each forecast of $y_{t+1}$ uses the *actual*
  values up to $t$. This answers: "how good are my one-month-ahead forecasts?"
- **Dynamic (multi-step)**: forecasts beyond one step use *previous forecasts*
  in place of unavailable actuals. The $h$-step forecast from a stationary ARMA
  converges to the unconditional mean as $h$ grows — long-horizon dynamic
  forecasts are just (a path towards) the mean.

For an AR(1), $\hat{y}_{T+h} = \mu + \phi^h (y_T - \mu)$: the forecast decays
geometrically to the mean. This mean-reversion is a *feature*, not a bug.

## 4. Forecast evaluation

Compare forecasts $f_t$ with realisations $y_t$ over the hold-out sample of
size $H$:

$$RMSE = \sqrt{\frac{1}{H}\sum_t (f_t - y_t)^2}, \qquad MAE = \frac{1}{H}\sum_t |f_t - y_t|$$

RMSE punishes large misses quadratically; MAE treats all misses linearly.
Always include a **naive benchmark** — the in-sample mean, or a random walk
("no change") — because a sophisticated model that cannot beat the
unconditional mean has learned nothing useful for forecasting.

## 5. Simple exponential smoothing (SES)

An atheoretical competitor that forecasts with an exponentially weighted
average of past observations:

$$f_{t+1} = \alpha y_t + (1-\alpha) f_t, \qquad 0 < \alpha < 1.$$

One parameter, no identification step, surprisingly competitive — SES is
optimal if the truth is an ARIMA(0,1,1). Comparing ARMA against SES disciplines
our enthusiasm for elaborate models.

## 6. Common pitfalls

- **In-sample $R^2$ as forecast quality.** Only out-of-sample performance counts.
- **Look-ahead bias**: re-estimating with data beyond the forecast origin, or
  selecting the model using the full sample, contaminates the comparison
  (strictly, even our IC selection should use only the training window).
- **Comparing dynamic ARMA forecasts with static SES forecasts** — match the
  information sets.
- **Over-interpreting tiny RMSE differences**: with 27 hold-out months,
  differences are statistically fragile (Diebold–Mariano tests formalise this;
  beyond our scope).

## 7. What's in this week's lab

1. Grid-search ARMA$(p,q)$, $p,q \le 5$ on `dhp` (full sample) by AIC and BIC;
2. re-estimate the chosen models on the training sample (to 2015-12);
3. static and dynamic forecasts for 2016-01–2018-03, with plots against actuals;
4. RMSE and MAE for both, plus the in-sample-mean benchmark;
5. simple exponential smoothing comparison;
6. verdict: which forecasts would you give the building society?

## References

- Brooks, C. (2019) *Introductory Econometrics for Finance*, 4th ed., CUP — Chapters 12–13.
- Diebold, F.X. and Mariano, R.S. (1995) "Comparing predictive accuracy",
  *Journal of Business & Economic Statistics* 13, 253–263.
