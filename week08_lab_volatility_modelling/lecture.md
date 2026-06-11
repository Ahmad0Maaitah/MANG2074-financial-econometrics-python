# Week 8 — LAB: Modelling Volatility in Financial Markets

**MANG2074 Financial Econometrics 1 · University of Southampton · Module leader: Ahmad Maaitah**

*This is a lab week: a short briefing, then hands-on work in `lab.ipynb`.*

---

## 1. Beyond plain GARCH

Week 7 established that GARCH(1,1) captures volatility clustering. Two stylised
facts remain unexplained:

1. **Asymmetry/leverage** — bad news raises volatility more than good news of
   the same size. Plain GARCH uses $u_{t-1}^2$, which destroys the sign.
2. **Conditional fat tails** — standardised residuals are still leptokurtic, so
   the normal likelihood understates tail risk.

This lab fixes both, then puts the models to work: forecasting volatility out of
sample and computing a parametric **Value-at-Risk**.

## 2. Asymmetric GARCH models

**GJR-GARCH** (Glosten–Jagannathan–Runkle, 1993) adds a term switched on only
by negative shocks:

$$\sigma_t^2 = \omega + \alpha u_{t-1}^2 + \gamma u_{t-1}^2 \mathbb{1}(u_{t-1} < 0) + \beta \sigma_{t-1}^2$$

Good news contributes $\alpha u^2$; bad news $(\alpha + \gamma) u^2$. Leverage
effect ⇒ $\gamma > 0$. A t-test on $\gamma$ *is* the asymmetry test (a
regression-based alternative is the Engle–Ng sign-bias test: regress squared
standardised residuals on a negative-shock dummy and signed shock terms).

**EGARCH** (Nelson, 1991) models the *log* of the variance:

$$\ln\sigma_t^2 = \omega + \alpha\left(|z_{t-1}| - E|z_{t-1}|\right) + \gamma z_{t-1} + \beta \ln\sigma_{t-1}^2$$

with $z = u/\sigma$. Two advantages: no non-negativity constraints needed
(logs!), and the $\gamma z_{t-1}$ term captures asymmetry directly — for equities
expect $\gamma < 0$ (negative shocks raise log-variance). In the `arch`
package: `arch_model(r, vol='EGARCH', p=1, o=1, q=1)`; for GJR simply add
`o=1` to a standard GARCH.

## 3. Student-t errors

Replace $z_t \sim N(0,1)$ with a standardised Student-t with $\nu$ degrees of
freedom (estimated). Small $\nu$ (5–10 for equities, even ~3 for FX) means
heavy conditional tails. The likelihood gain is usually enormous, and the 1%
quantile moves materially — which matters directly for VaR.

## 4. Volatility forecasting

GARCH(1,1) $h$-step variance forecast (derived by repeated substitution):

$$E_T[\sigma^2_{T+h}] = \bar\sigma^2 + (\alpha+\beta)^{h-1}\left(\sigma^2_{T+1} - \bar\sigma^2\right)$$

— mean reversion to $\bar\sigma^2 = \omega/(1-\alpha-\beta)$ at rate
$\alpha+\beta$. With a **fixed estimation window** we estimate parameters once
(e.g. to end-2015, `fit(last_obs=...)`) and generate one-step forecasts through
the hold-out sample, comparing them with realised squared returns / |returns|.

## 5. Parametric VaR from a GARCH forecast

The 1% one-day VaR answers: "a loss we should exceed only 1 day in 100".
For a (long) position of value $V$, with conditional mean $\mu$ and forecast
volatility $\hat\sigma_{T+1}$ (in %):

$$VaR_{1\%} = -\left(\mu + q_{0.01}\,\hat\sigma_{T+1}\right)\times \frac{V}{100}$$

where $q_{0.01}$ is the 1% quantile of the standardised innovation distribution:
$-2.326$ for the normal, or the (more negative once rescaled) Student-t
quantile $t_{0.01}(\nu)\sqrt{(\nu-2)/\nu}$. GARCH-based VaR is *dynamic*: limits
tighten automatically in turbulent regimes.

## 6. Common pitfalls

- Fitting EGARCH and interpreting coefficients as if they were GARCH ones —
  the equation is in **logs of variance** and in $z$, not $u^2$.
- Expecting equity-style leverage in FX: exchange rates have no obvious
  "leverage", and asymmetry is often small or absent — let the data speak.
- Comparing models with different error distributions by log-likelihood alone
  is fine (same data), but prefer information criteria when parameter counts differ.
- Quoting VaR without stating the horizon, confidence level and distribution.
- Annualising daily VaR with $\sqrt{252}$ — VaR does not scale like volatility
  unless returns are i.i.d. normal (they are not — that is the whole point).

## 7. What's in this week's lab

Daily FX returns (all three currencies in `currencies.csv`) and the S&P 500:

1. fit GARCH(1,1), GJR-GARCH(1,1,1) and EGARCH(1,1,1) to S&P returns; test
   asymmetry and compare news responses;
2. fit GJR to the yen — is there "leverage" in FX?
3. re-estimate the yen GARCH with Student-t errors; interpret $\hat\nu$;
4. plot conditional volatilities; compare models;
5. fixed-window (to end-2015) one-step volatility forecasts for 2016–2018,
   compared with realised |returns|;
6. compute 1% one-day parametric VaR for a £1m JPY position under normal vs t.

## References

- Brooks, C. (2019) *Introductory Econometrics for Finance*, 4th ed., CUP — Chapter 19; VaR: Chapter 25 / 14.
- Glosten, L., Jagannathan, R. and Runkle, D. (1993) "On the relation between the
  expected value and the volatility of the nominal excess return on stocks",
  *Journal of Finance* 48, 1779–1801.
- Nelson, D.B. (1991) "Conditional heteroskedasticity in asset returns: a new
  approach", *Econometrica* 59, 347–370.
