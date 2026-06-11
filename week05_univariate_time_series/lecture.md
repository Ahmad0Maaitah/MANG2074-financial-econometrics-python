# Week 5 — Univariate Time Series: AR, MA, ARMA and ARIMA Models

**MANG2074 Financial Econometrics 1 · University of Southampton · Module leader: Ahmad Maaitah**

---

## 1. Motivation: forecasting from a series' own past

A building society wants to forecast UK house-price growth over the next two
years. No structural model of supply, demand, interest rates and migration is
available at monthly frequency — but 27 years of the series' own history are.
**Univariate time-series models** exploit one powerful idea: if a series is
correlated with its own past (autocorrelated), then its past predicts its
future. The ARMA family is the workhorse: atheoretical, cheap, and surprisingly
hard to beat at short horizons.

## 2. Stationarity first

A process is **(weakly/covariance) stationary** if its mean, variance and
autocovariances do not depend on $t$:

$$E(y_t) = \mu, \qquad \text{Var}(y_t) = \sigma^2, \qquad \text{Cov}(y_t, y_{t-s}) = \gamma_s \;\; \forall t.$$

ARMA modelling *requires* stationarity — otherwise shocks have permanent
effects, sample moments estimate nothing stable, and forecasts are anchored to
nothing. The leading violation is a **unit root**: in the AR(1) model
$y_t = \phi y_{t-1} + u_t$, the case $\phi = 1$ (a random walk). The
**Augmented Dickey–Fuller (ADF) test** has:

- $H_0$: the series contains a unit root (non-stationary);
- $H_1$: the series is stationary.

The test statistic is a t-ratio on $\psi$ in
$\Delta y_t = \mu + \psi y_{t-1} + \sum_i \alpha_i \Delta y_{t-i} + u_t$, but it
does **not** follow the t-distribution — use the Dickey–Fuller critical values
that `adfuller` supplies (≈ −2.87 at 5% with a constant). Reject (conclude
stationary) only if the statistic is **more negative** than the critical value.
House-price *levels* fail; percentage growth `dhp` passes — so we model `dhp`.
(An ARIMA$(p,1,q)$ on the level is the same thing as ARMA$(p,q)$ on the
difference — that is what the "I" means.)

## 3. The building blocks

**AR(p) — autoregressive:**

$$y_t = \mu + \phi_1 y_{t-1} + \dots + \phi_p y_{t-p} + u_t$$

Today is a weighted sum of recent past plus noise. Stationarity requires the
roots of the AR polynomial to lie outside the unit circle (AR(1): $|\phi_1|<1$).
Shocks decay geometrically. Unconditional mean (AR(1)): $\mu/(1-\phi_1)$.

**MA(q) — moving average:**

$$y_t = \mu + u_t + \theta_1 u_{t-1} + \dots + \theta_q u_{t-q}$$

Today is a finite-memory combination of recent *shocks*: effects vanish
completely after $q$ periods. MA processes are always stationary;
**invertibility** (MA(1): $|\theta_1|<1$) is needed to write them as an AR(∞)
and identify them uniquely.

**ARMA(p,q)** combines both:

$$\phi(L)\, y_t = \mu + \theta(L)\, u_t$$

with lag-operator polynomials $\phi(L) = 1 - \phi_1 L - \dots - \phi_p L^p$ and
$\theta(L) = 1 + \theta_1 L + \dots + \theta_q L^q$.

## 4. Identification: the correlogram

The **autocorrelation function (ACF)** $\tau_s = \gamma_s/\gamma_0$ and the
**partial autocorrelation function (PACF)** (correlation between $y_t$ and
$y_{t-s}$ *after* controlling for the lags in between) have signature patterns:

| Process | ACF | PACF |
|---|---|---|
| AR(p) | geometric decay | **cuts off** after lag $p$ |
| MA(q) | **cuts off** after lag $q$ | geometric decay |
| ARMA(p,q) | decays | decays |

Significance bands: an individual sample autocorrelation is approximately
$N(0, 1/T)$ under the null of no autocorrelation, so bounds are
$\pm 1.96/\sqrt{T}$. The **Ljung–Box Q-statistic** tests the first $m$
autocorrelations jointly:

$$Q^* = T(T+2)\sum_{s=1}^{m} \frac{\hat\tau_s^2}{T-s} \sim \chi^2(m) \text{ under } H_0: \text{no autocorrelation}.$$

## 5. The Box–Jenkins cycle

1. **Identification** — plot the series; test stationarity (ADF); inspect
   ACF/PACF for candidate $(p,q)$.
2. **Estimation** — fit by maximum likelihood (`ARIMA(y, order=(p,0,q)).fit()`).
3. **Diagnostic checking** — residuals should be white noise: correlogram and
   Ljung–Box on residuals; overfit by one order and check the extra term is
   insignificant.
4. **(Then) Forecasting** — next week.

Modern practice supplements step 1 with **information criteria**:

$$AIC = -2\ln \hat L + 2k, \qquad BIC = -2\ln \hat L + k\ln T$$

(lower is better; $k$ = number of estimated parameters). BIC penalises size more
heavily, so BIC picks models no larger than AIC's choice. BIC is consistent
(finds the true order asymptotically, if one exists); AIC tends to overfit
slightly but often forecasts well.

## 6. Application: UK house-price growth

For `dhp` (monthly % growth, 1991–2018): the ACF decays slowly over many lags
while the PACF is large at lags 1–2 and negligible after — the classic AR(2)
signature, and indeed BIC selects ARMA(2,0) while AIC prefers a larger
ARMA(4,2). Both fits imply highly persistent growth (sum of AR coefficients
well above 0.5): hot and cold housing markets last many months — very different
from stock returns, whose ACF is dead at all lags (efficient markets).

## 7. Common pitfalls in finance applications

- **Fitting ARMA to a non-stationary level.** Always ADF-test first; difference
  (or use returns) if a unit root is not rejected.
- **Misreading the ADF test direction** — rejection means *stationary*; "ADF
  p-value = 0.94" means you must difference.
- **Over-reading the correlogram of returns**: with $T=5000$, bands are ±0.028,
  so trivial autocorrelations look "significant"; economic size matters.
- **Choosing $p,q$ by maximising $R^2$** instead of information criteria.
- **Ignoring residual diagnostics**: an ARMA whose residuals are still
  autocorrelated is under-specified and its standard errors are wrong.
- **Seasonality**: monthly housing data can have seasonal patterns; seasonal
  dummies or SARIMA terms may be needed (we keep it simple here).

## 8. What's in this week's lab

Using `ukhp.csv`:

1. construct `dhp`; ADF tests on level and growth;
2. correlogram (ACF, PACF, Ljung–Box) and Box–Jenkins identification;
3. estimate AR(1), AR(2), MA(1), ARMA(1,1), ARMA(2,2); compare via AIC/BIC;
4. residual diagnostics for the preferred model;
5. check stationarity/invertibility of the fitted polynomial roots.

## References

- Brooks, C. (2019) *Introductory Econometrics for Finance*, 4th ed., CUP — Chapter 11 (this lab tracks the chapter's UKHP example).
- Box, G.E.P. and Jenkins, G.M. (1976) *Time Series Analysis: Forecasting and Control*, Holden-Day.
- Dickey, D.A. and Fuller, W.A. (1979) "Distribution of the estimators for
  autoregressive time series with a unit root", *JASA* 74, 427–431.
