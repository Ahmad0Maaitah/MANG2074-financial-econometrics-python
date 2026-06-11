# Week 2 — The Classical Linear Regression Model I

**MANG2074 Financial Econometrics 1 · University of Southampton · Module leader: Ahmad Maaitah**

---

## 1. Motivation: two real finance problems

**Problem A — Hedging.** A fund holds a portfolio that tracks the S&P 500 spot
index. To protect the portfolio it sells index *futures*. How many futures
contracts per unit of spot exposure should it sell? The **optimal hedge ratio** is
the slope of a regression of spot returns on futures returns. Is the optimal
ratio equal to 1 (a "naive" one-for-one hedge), or something else?

**Problem B — Asset pricing.** The CAPM says a stock's expected excess return is
proportional to the market's expected excess return, with proportionality constant
**beta**. How do we estimate Ford's beta? Is it significantly different from 1
(the market's own beta)? Is its alpha zero, as the CAPM requires?

Both problems reduce to estimating a **simple linear regression** — the single
most important tool in this module.

## 2. The model

$$y_t = \alpha + \beta x_t + u_t, \qquad t = 1, \dots, T$$

- $y_t$: dependent variable (regressand) — e.g. spot return, stock excess return.
- $x_t$: explanatory variable (regressor) — e.g. futures return, market excess return.
- $u_t$: random disturbance, capturing everything else affecting $y_t$.
- $\alpha, \beta$: unknown population parameters we want to estimate.

## 3. The OLS estimator: derivation

Ordinary Least Squares chooses $\hat{\alpha}, \hat{\beta}$ to minimise the
**residual sum of squares**

$$RSS = \sum_{t=1}^{T} \hat{u}_t^2 = \sum_{t=1}^{T} (y_t - \hat{\alpha} - \hat{\beta} x_t)^2.$$

Differentiate w.r.t. $\hat{\alpha}$ and $\hat{\beta}$ and set to zero (the *normal
equations*):

$$\frac{\partial RSS}{\partial \hat{\alpha}} = -2\sum_t (y_t - \hat{\alpha} - \hat{\beta}x_t) = 0,
\qquad
\frac{\partial RSS}{\partial \hat{\beta}} = -2\sum_t x_t (y_t - \hat{\alpha} - \hat{\beta}x_t) = 0$$

Solving simultaneously:

$$\hat{\beta} = \frac{\sum_t (x_t - \bar{x})(y_t - \bar{y})}{\sum_t (x_t - \bar{x})^2}
= \frac{\widehat{\text{Cov}}(x,y)}{\widehat{\text{Var}}(x)},
\qquad
\hat{\alpha} = \bar{y} - \hat{\beta}\bar{x}$$

Two immediate insights:

- $\hat{\beta}$ is just the sample covariance of $x$ and $y$ scaled by the variance
  of $x$. The CAPM beta is *literally* $\text{Cov}(r_i, r_m)/\text{Var}(r_m)$ —
  OLS and the finance formula coincide.
- The fitted line passes through the point of means $(\bar{x}, \bar{y})$.

## 4. Key assumptions (preview)

OLS estimates are *unbiased and efficient* under the classical assumptions
(treated in full next week): the errors have zero mean, constant variance, no
autocorrelation, are independent of the regressors, and (for exact inference)
are normally distributed. This week we take them as given.

## 5. Precision: standard errors

The estimated standard errors are

$$SE(\hat{\beta}) = \frac{s}{\sqrt{\sum_t (x_t - \bar{x})^2}}, \qquad
SE(\hat{\alpha}) = s\sqrt{\frac{\sum_t x_t^2}{T \sum_t (x_t - \bar{x})^2}}$$

where $s^2 = RSS/(T-2)$ estimates the error variance. Precision improves with
more observations, more variation in $x$, and smaller residual variance.

## 6. Hypothesis testing: the t-test

To test $H_0: \beta = \beta^*$ against $H_1: \beta \ne \beta^*$, form

$$t = \frac{\hat{\beta} - \beta^*}{SE(\hat{\beta})} \sim t(T-2) \text{ under } H_0.$$

Reject $H_0$ at the 5% level if $|t|$ exceeds the critical value (≈1.97 for the
sample sizes we use; ≈1.96 asymptotically).

**Crucial finance point:** the software's printed t-statistic and p-value always
test $\beta^* = 0$ (statistical significance). For the hedge ratio and for CAPM
beta we usually care about $\beta^* = 1$ — you must construct that test yourself:

$$t = \frac{\hat{\beta} - 1}{SE(\hat{\beta})}$$

In `statsmodels`: `results.t_test('Futures = 1')`.

## 7. Application A — the optimal hedge ratio

With spot return $\Delta s_t$ and futures return $\Delta f_t$ (both in %, log
differences), the variance-minimising hedge ratio $h^*$ solves

$$\min_h \text{Var}(\Delta s_t - h\,\Delta f_t) \;\Rightarrow\; h^* = \frac{\text{Cov}(\Delta s, \Delta f)}{\text{Var}(\Delta f)}$$

— exactly the OLS slope in $\Delta s_t = \alpha + h \Delta f_t + u_t$.

We will run the regression in **levels** (spot on futures prices) *and* in
**returns**. The levels regression produces an $R^2$ of essentially 1 and a huge
t-ratio — but it is close to meaningless, because both prices trend together
(non-stationarity / spurious regression). The returns regression gives the valid
hedge ratio. We then test $H_0: h = 1$.

## 8. Application B — estimating CAPM beta

The testable (ex-post) CAPM regression for stock $i$:

$$(r_{i,t} - r_{f,t}) = \alpha_i + \beta_i (r_{m,t} - r_{f,t}) + u_{i,t}$$

Using `capm.csv`: returns are $100\,\Delta\ln P_t$; the risk-free rate `USTB3M`
is **annualised**, so the monthly rate is `USTB3M/12`. The CAPM predicts
$\alpha_i = 0$; $\beta_i$ measures systematic risk:

- $\beta > 1$: aggressive stock — amplifies market movements;
- $\beta < 1$: defensive stock;
- $\beta \approx 0$: uncorrelated with the market.

## 9. Diagnostics to glance at (full treatment in Week 4)

- $R^2$: share of variation in $y$ explained — typically 0.2–0.5 for individual
  stock CAPM regressions; very high $R^2$ in a levels regression is a red flag.
- Durbin–Watson statistic near 2 suggests no first-order autocorrelation.
- Residual plots: look for outliers and changing variance.

## 10. Common pitfalls in finance applications

- **Regressing prices on prices**: trending levels produce spurious "significance".
  Use returns (or formally test for cointegration — Year 3 material).
- **Forgetting to convert the risk-free rate to the data frequency** (divide an
  annualised % rate by 12 for monthly data).
- **Reading the wrong test**: the printed p-value tests $\beta=0$, not $\beta=1$.
- **Interpreting $\alpha$ in % per month** and forgetting to annualise when
  comparing with reported fund "alphas".
- **Causality**: regression measures association; a significant beta does not say
  the market "causes" Ford's return in a structural sense.

## 11. What's in this week's lab

1. Load `sandphedge.csv`; compute log returns of spot and futures.
2. Regress spot on futures in levels, then in returns; compare and explain.
3. Test $H_0$: hedge ratio $= 1$ with `t_test`.
4. Load `capm.csv`; build excess returns for Ford and the S&P 500.
5. Estimate Ford's beta; test $\beta = 0$, $\beta = 1$, and $\alpha = 0$.
6. Scatter plot with fitted security characteristic line.

## References

- Brooks, C. (2019) *Introductory Econometrics for Finance*, 4th ed., CUP — Chapters 3–5.
- Sharpe, W.F. (1964) "Capital asset prices: a theory of market equilibrium under
  conditions of risk", *Journal of Finance* 19, 425–442.
- Ederington, L.H. (1979) "The hedging performance of the new futures markets",
  *Journal of Finance* 34, 157–170.
