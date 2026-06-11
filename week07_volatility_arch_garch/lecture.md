# Week 7 — Modelling Volatility: ARCH and GARCH

**MANG2074 Financial Econometrics 1 · University of Southampton · Module leader: Ahmad Maaitah**

---

## 1. Motivation: risk is the thing we trade

Option prices, Value-at-Risk limits, margin requirements, portfolio weights —
all depend on **volatility**, not on expected returns. And the single most
robust empirical finding about volatility is that it is *not constant*. A risk
manager using one full-sample standard deviation treats October 2008 and a
sleepy August identically; markets do not. We need models in which the
*conditional* variance $\sigma_t^2 = \text{Var}(r_t \mid \Omega_{t-1})$ moves
with the information set $\Omega_{t-1}$.

## 2. Stylised facts of financial returns

1. **Volatility clustering** — "large changes tend to be followed by large
   changes, of either sign" (Mandelbrot). The returns themselves are nearly
   uncorrelated, but their *squares* are strongly autocorrelated.
2. **Fat tails (leptokurtosis)** — Week 1's JB rejections. Crucially, even a
   GARCH model with *normal* shocks produces unconditionally fat-tailed
   returns, because mixing normals with different variances fattens tails.
3. **Leverage effects** — volatility rises more after a price *fall* than after
   an equally sized rise (firms become more levered; also risk-aversion
   feedback). Symmetric GARCH ignores this; Week 8's GJR and EGARCH capture it.

## 3. The ARCH model (Engle, 1982)

Decompose the return as $r_t = \mu + u_t$, $u_t = \sigma_t z_t$ with
$z_t \sim N(0,1)$ i.i.d. **ARCH(q)** lets the conditional variance respond to
recent squared shocks:

$$\sigma_t^2 = \omega + \sum_{i=1}^{q} \alpha_i u_{t-i}^2, \qquad \omega > 0,\; \alpha_i \ge 0.$$

A big shock yesterday ⇒ a big variance today ⇒ big shocks today are likely —
clustering, mechanically.

**Testing for ARCH effects (the ARCH-LM test).** Regress $\hat u_t^2$ on $q$
of its own lags; under $H_0$ (no ARCH) $T R^2 \sim \chi^2(q)$. Run this *before*
fitting a GARCH — if there are no ARCH effects there is nothing to model.

ARCH's weakness: in practice $q$ must be large, with many restricted parameters.

## 4. GARCH (Bollerslev, 1986)

**GARCH(1,1)** adds yesterday's *variance* to the recursion:

$$\sigma_t^2 = \omega + \alpha u_{t-1}^2 + \beta \sigma_{t-1}^2$$

- $\alpha$ — the **news/reaction** coefficient: how hard variance reacts to
  yesterday's surprise;
- $\beta$ — the **persistence/memory** coefficient: how much of yesterday's
  variance carries over;
- GARCH(1,1) equals an ARCH(∞) with geometrically decaying weights — that is
  why one lag of each usually suffices.

**Unconditional variance.** Take unconditional expectations of both sides and
use $E(u_{t-1}^2) = E(\sigma_{t-1}^2) = \bar{\sigma}^2$ in steady state:

$$\bar{\sigma}^2 = \omega + \alpha\bar{\sigma}^2 + \beta\bar{\sigma}^2
\;\;\Longrightarrow\;\;
\bar{\sigma}^2 = \frac{\omega}{1 - \alpha - \beta}$$

valid only if $\alpha + \beta < 1$ (**covariance stationarity**). The sum
$\alpha + \beta$ is the **persistence**: an $h$-step variance forecast reverts
to $\bar\sigma^2$ at rate $(\alpha+\beta)^h$. Typical daily estimates:
$\alpha \approx 0.05$, $\beta \approx 0.90$–$0.95$, persistence ≈ 0.99 —
volatility shocks take months to fade. If $\alpha + \beta \ge 1$ (IGARCH),
shocks never fade and the unconditional variance does not exist.

## 5. Estimation: maximum likelihood intuition

OLS cannot estimate GARCH (the model is for an unobservable variance and is
nonlinear in parameters). Instead, **maximum likelihood**: given parameters
$(\mu, \omega, \alpha, \beta)$, the recursion delivers $\sigma_t^2$ for every
$t$; under conditional normality the log-likelihood is

$$\ln L = -\frac{1}{2}\sum_t \left[\ln(2\pi) + \ln \sigma_t^2 + \frac{(r_t-\mu)^2}{\sigma_t^2}\right]$$

and a numerical optimiser climbs this surface. Intuition: choose parameters so
that the predicted variances are *large exactly when the squared surprises are
large*. In Python the `arch` package does all of this:
`arch_model(r, vol='GARCH', p=1, q=1).fit()`.

(If $z_t$ is not truly normal, the same estimator is **quasi-**ML: consistent,
with robust SEs — which `arch` reports by default.)

## 6. Diagnostics

- **Standardised residuals** $\hat z_t = \hat u_t / \hat\sigma_t$ should show
  no remaining ARCH: re-run the ARCH-LM test on them.
- Their kurtosis should drop sharply relative to raw returns (clustering
  explains part, not all, of the fat tails — hence Student-t errors in Week 8).
- Check $\alpha + \beta < 1$ and all parameters ≥ 0 / significant.
- Plot $\hat\sigma_t$ against known events (2008, Brexit…) — does it spike when
  it should?

## 7. Common pitfalls in finance applications

- **Fitting GARCH to monthly or quarterly data** with few observations — ARCH
  effects are mainly a high-frequency (daily/weekly) phenomenon and MLE needs
  hundreds of observations.
- **Scaling**: feed the optimiser returns in *percent* (e.g. `100*dlog`);
  returns in raw decimals make $\omega \approx 10^{-6}$ and the optimiser
  struggles.
- **Reading $\beta$ alone as "persistence"** — persistence is $\alpha+\beta$.
- **Forgetting the stationarity check**: persistence ≥ 1 means your long-run
  forecasts are meaningless.
- **Using GARCH variance as if it were observed**: it is a model estimate,
  conditional on specification.

## 8. What's in this week's lab

Daily JPY/USD returns (`currencies.csv`) and daily S&P 500 returns (`sp500.csv`):

1. construct `rjpy = 100*dlog(JPY)`; document the stylised facts (ACF of
   returns vs squared returns, kurtosis);
2. ARCH-LM test on both series;
3. estimate ARCH(5) and GARCH(1,1) with the `arch` package;
4. compute persistence and the implied unconditional variance; compare with the
   sample variance;
5. plot conditional volatility and annotate crisis episodes;
6. diagnostics on standardised residuals.

## References

- Brooks, C. (2019) *Introductory Econometrics for Finance*, 4th ed., CUP — Chapter 19 (volatility modelling).
- Engle, R.F. (1982) "Autoregressive conditional heteroscedasticity with
  estimates of the variance of United Kingdom inflation", *Econometrica* 50, 987–1007.
- Bollerslev, T. (1986) "Generalized autoregressive conditional
  heteroskedasticity", *Journal of Econometrics* 31, 307–327.
