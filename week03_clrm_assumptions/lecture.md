# Week 3 — The CLRM II: Assumptions, Multiple Regression and F-tests

**MANG2074 Financial Econometrics 1 · University of Southampton · Module leader: Ahmad Maaitah**

---

## 1. Motivation: beyond one factor

The CAPM says one factor — the market — prices everything. Ross's **Arbitrage
Pricing Theory (APT)** disagrees: any systematic source of risk that investors
cannot diversify away (industrial production shocks, inflation surprises, credit
spreads, the slope of the yield curve…) may carry a risk premium. Testing this
requires regressing a stock's excess return on **several** variables at once —
multiple regression — and asking questions like "are the four macro variables
*jointly* useful, even if each is individually insignificant?" That question needs
the **F-test**.

But first we must be explicit about *when* OLS can be trusted: the classical
assumptions.

## 2. The multiple regression model

$$y_t = \beta_1 + \beta_2 x_{2t} + \beta_3 x_{3t} + \dots + \beta_k x_{kt} + u_t$$

or compactly in matrix form $y = X\beta + u$ with $X$ a $T \times k$ matrix. The
OLS estimator generalises to

$$\hat{\beta} = (X'X)^{-1}X'y$$

**Interpretation discipline:** $\beta_j$ is the *partial* effect of $x_j$ on $y$,
**holding all other included regressors constant**. A multiple-regression beta on
the market is therefore not the same number as the simple CAPM beta unless the
other factors are uncorrelated with the market.

## 3. The five classical (Gauss–Markov) assumptions

1. **$E(u_t) = 0$** — errors have zero mean. (Guaranteed if an intercept is included.)
2. **$\text{Var}(u_t) = \sigma^2 < \infty$** — *homoscedasticity*: constant error variance.
3. **$\text{Cov}(u_t, u_s) = 0$ for $t \neq s$** — no autocorrelation in the errors.
4. **$\text{Cov}(u_t, x_{jt}) = 0$** — regressors non-stochastic or at least
   uncorrelated with the error (no endogeneity).
5. **$u_t \sim N(0, \sigma^2)$** — normality (needed for *exact* finite-sample
   inference; asymptotically dispensable).

**Gauss–Markov theorem.** Under assumptions 1–4, OLS is **BLUE**: the **B**est
(minimum-variance) **L**inear **U**nbiased **E**stimator. Intuition: among all
estimators that are linear functions of $y$ and unbiased, none has smaller
sampling variance than OLS.

What breaks if each fails (full diagnostics next week):

| Violated | Consequence |
|----------|-------------|
| 2 (heteroscedasticity) | $\hat\beta$ still unbiased, but SEs wrong ⇒ invalid t/F tests |
| 3 (autocorrelation) | same: unbiased but invalid inference; OLS inefficient |
| 4 (endogeneity) | $\hat\beta$ **biased and inconsistent** — the worst case |
| 5 (non-normality) | exact small-sample tests invalid; OK in large samples (CLT) |

## 4. Goodness of fit: $R^2$ and adjusted $R^2$

$$R^2 = 1 - \frac{RSS}{TSS}, \qquad TSS = \sum_t (y_t - \bar{y})^2$$

$R^2$ **never falls** when you add a regressor, however irrelevant. The adjusted
version penalises model size:

$$\bar{R}^2 = 1 - \left[\frac{T-1}{T-k}(1 - R^2)\right]$$

$\bar{R}^2$ rises with an added variable only if its $|t|$-ratio exceeds 1 —
a very weak hurdle, so don't choose models on $\bar{R}^2$ alone (Week 5
introduces sharper criteria, AIC/BIC).

## 5. Testing several restrictions at once: the F-test

Suppose we want to test $m$ joint linear restrictions, e.g.
$H_0: \beta_3 = \beta_4 = \beta_5 = \beta_6 = 0$. Estimate the model twice:

- **Unrestricted** model: residual sum of squares $URSS$;
- **Restricted** model (restrictions imposed): $RRSS$.

$$F = \frac{(RRSS - URSS)/m}{URSS/(T-k)} \;\sim\; F(m,\, T-k) \text{ under } H_0$$

Intuition: imposing valid restrictions should barely raise the RSS; a large rise
(big $F$) is evidence against $H_0$. The "regression F-statistic" printed in every
summary is the special case where *all* slopes are restricted to zero.

**Individually insignificant but jointly significant** is common when regressors
are correlated (multicollinearity, Week 4): each variable's separate contribution
is poorly measured, but the group clearly matters. The reverse can also happen.
Always test joint hypotheses with F, not by scanning t-ratios.

In `statsmodels`: `results.f_test('dprod = dcredit = dmoney = dspread = 0')`.

## 6. Application: an APT-style model for Microsoft

Using `macro.csv` (monthly, 1986–2018) we explain Microsoft's excess return with
the market plus macro surprise proxies. Financial theory says asset prices react
to **news** — unanticipated changes — so we first transform each macro series
into a change/growth rate (following Brooks ch. 7):

| Variable | Construction | Captures |
|----------|--------------|----------|
| `ermsoft` | $100\,\Delta\ln(MSFT) - USTB3M/12$ | excess stock return |
| `ersandp` | $100\,\Delta\ln(S\&P) - USTB3M/12$ | market factor |
| `dprod` | $\Delta INDPRO$ | industrial production shock |
| `dcredit` | $\Delta CCREDIT$ | consumer credit shock |
| `dinflation` | $\Delta(100\,\Delta\ln CPI)$ | inflation surprise |
| `dmoney` | $\Delta M1SUPPLY$ | money supply shock |
| `dspread` | $\Delta BMINUSA$ | credit (default) spread change |
| `rterm` | $\Delta(USTB10Y - USTB3M)$ | term-structure (slope) change |

Then estimate

$$ermsoft_t = \beta_1 + \beta_2\,ersandp_t + \beta_3\,dprod_t + \beta_4\,dcredit_t + \beta_5\,dinflation_t + \beta_6\,dmoney_t + \beta_7\,dspread_t + \beta_8\,rterm_t + u_t$$

and test whether the non-market macro factors are jointly priced.

## 7. Common pitfalls in finance applications

- **Levels of macro variables on returns**: CPI or M1 in levels trend; the
  regression mixes orders of integration. Transform to changes first.
- **"Holding constant" language**: with correlated factors, the partial effects
  can flip sign relative to simple regressions — that is mathematics, not a bug.
- **Maximising $R^2$**: with monthly stock returns, an honest $R^2$ of 0.3–0.4 is
  good; an $R^2$ of 0.95 usually signals a look-ahead or a levels variable
  smuggled in.
- **Data mining / step-wise selection**: searching many factor combinations and
  reporting the best t-stats invalidates the stated significance levels.
- **Forgetting degrees of freedom**: with $k$ regressors you need $T \gg k$;
  asset-pricing panels with 8 factors and 60 observations are fragile.

## 8. What's in this week's lab

1. Build the transformed dataset from `macro.csv` exactly as in §6.
2. Estimate the full APT-style regression; interpret size, sign and significance.
3. Compute $\bar R^2$ by hand and verify against the summary.
4. F-test: are `dprod`, `dcredit`, `dmoney`, `dspread` jointly zero?
5. Replicate that F-test manually via restricted vs unrestricted RSS.
6. Re-estimate the trimmed model and compare AIC/BIC.

## References

- Brooks, C. (2019) *Introductory Econometrics for Finance*, 4th ed., CUP — Chapters 6–7.
- Ross, S.A. (1976) "The arbitrage theory of capital asset pricing", *Journal of
  Economic Theory* 13, 341–360.
- Chen, N-F., Roll, R. and Ross, S.A. (1986) "Economic forces and the stock
  market", *Journal of Business* 59, 383–403.
