# Week 12 — Revision: The Whole Module on a Few Pages

**MANG2074 Financial Econometrics 1 · University of Southampton · Module leader: Ahmad Maaitah**

---

## 1. The module in one paragraph

We learned to take a finance question — *is this stock risky? is the hedge
right? is growth predictable? how big could tomorrow's loss be? are markets in
a bear regime? does this effect hold across firms?* — and translate it into a
statistical model, estimate that model in Python, **test whether its
assumptions hold**, and translate the numbers back into an economic answer.
The arc: returns and distributions (Wk 1) → regression and inference (Wk 2–4)
→ time-series dynamics and forecasting (Wk 5–6) → time-varying risk (Wk 7–8)
→ regimes and unobserved states (Wk 9) → panels (Wk 10–11).

## 2. Topic-by-topic summary

**Week 1 — Returns and distributions.** Model returns, not prices. Log returns
$r_t = 100\,\Delta\ln P_t$ sum over time; simple returns aggregate across
assets. Financial returns are non-normal: negatively skewed, heavily
leptokurtic; $JB = T[S^2/6 + (K-3)^2/24] \sim \chi^2(2)$ rejects normality
emphatically for daily data.

**Weeks 2–3 — CLRM.** OLS minimises RSS:
$\hat\beta = \widehat{Cov}(x,y)/\widehat{Var}(x)$ (simple) or
$(X'X)^{-1}X'y$ (multiple). Under the Gauss–Markov assumptions (zero-mean,
homoscedastic, non-autocorrelated errors, exogenous regressors; + normality for
exact tests), OLS is BLUE. t-test for one restriction (remember to test
$\beta=1$ yourself when that is the question); F-test
$\frac{(RRSS-URSS)/m}{URSS/(T-k)}$ for several at once. Applications: hedge
ratio (returns, not levels!), CAPM beta/alpha, APT-style factor models with
*changes* in macro variables.

**Week 4 — Diagnostics.** For each assumption: a test, a consequence, a
remedy. Heteroscedasticity (BP/White → robust HC SEs), autocorrelation (DW ≈
$2(1-\hat\rho)$; Breusch–Godfrey → Newey–West HAC SEs), non-normality (JB on
residuals → outlier dummies + CLT), multicollinearity (VIF $=1/(1-R_j^2)$),
functional form (RESET), parameter stability (Chow
$F=\frac{(RSS_p-RSS_1-RSS_2)/k}{(RSS_1+RSS_2)/(T-2k)}$).

**Weeks 5–6 — ARMA & forecasting.** Stationarity first: ADF with $H_0$ = unit
root; reject ⇒ stationary. Identification: ACF decays/PACF cuts at $p$ ⇒ AR(p);
ACF cuts at $q$/PACF decays ⇒ MA(q). Selection: AIC ($+2k$) vs BIC
($+k\ln T$; more parsimonious). Diagnostics: Ljung–Box on residuals. Forecasts:
static (one-step, uses actuals) vs dynamic (multi-step, converges to the
unconditional mean at rate given by the AR roots); evaluate by RMSE/MAE
against a naive benchmark, out of sample only.

**Weeks 7–8 — Volatility.** Returns are uncorrelated but their squares are
not. ARCH-LM tests for conditional heteroscedasticity. GARCH(1,1):
$\sigma_t^2 = \omega + \alpha u_{t-1}^2 + \beta\sigma_{t-1}^2$; persistence
$= \alpha + \beta$ (typically ≈ 0.99 daily); unconditional variance
$\omega/(1-\alpha-\beta)$ requires $\alpha+\beta<1$. Estimated by MLE.
Asymmetry: GJR adds $\gamma u_{t-1}^2 \mathbb{1}(u_{t-1}<0)$ — leverage means
$\gamma>0$; EGARCH works in $\ln\sigma_t^2$ (asymmetry = negative sign on
$\gamma z$). Student-t innovations for conditional fat tails. Parametric VaR:
$-(\mu + q_{0.01}\hat\sigma_{T+1})\times V/100$.

**Week 9 — Regimes & states.** Markov switching: unobserved state $s_t$ with
transition probabilities $p_{ij}$; expected duration $1/(1-p_{ii})$; smoothed
probabilities date regimes retrospectively (filtered = real-time). Equity
regimes: high-mean/low-vol vs low-mean/high-vol. TAR: switching triggered by an
observable threshold. State space + Kalman filter: continuous unobserved states
(time-varying betas, local levels).

**Weeks 10–11 — Panels.** $y_{it} = \alpha + \beta x_{it} + \mu_i + v_{it}$.
Pooled OLS biased if $Cov(\mu_i, x) \ne 0$. FE (within/LSDV — identical)
sweeps out $\mu_i$ but loses time-invariant regressors. RE efficient *iff*
$\mu_i \perp x$. Hausman: reject ⇒ **FE**. Always cluster SEs by entity.

## 3. Which model when — the decision guide

| Your question | First checks | Model | Key statistic to report | Watch out for |
|---|---|---|---|---|
| Does $x$ explain $y$ (returns)? | stationarity of both; scatter plot | OLS regression (Wk 2–3) | $\hat\beta$, robust t, $R^2$ | levels-on-levels spurious regression |
| Is the relationship trustworthy? | — | diagnostic battery (Wk 4) | BP/White, BG, JB, RESET, Chow | choosing SEs to fit the story |
| Hedge ratio / beta = 1? | returns, not prices | OLS + `t_test('x = 1')` | $(\hat\beta-1)/SE$ | software default tests $\beta=0$ |
| Joint factor relevance? | — | F-test on restrictions | $F(m, T-k)$ | judging jointly by individual t's |
| Forecast a single series? | ADF; ACF/PACF | ARMA (Wk 5–6), chosen by BIC | out-of-sample RMSE vs naive | unit roots; look-ahead bias |
| Long-horizon point forecast? | stationary? | the unconditional mean (any ARMA converges there) | — | over-selling model value |
| Measure/forecast risk? | ARCH-LM; squared-return ACF | GARCH(1,1) (Wk 7) | $\alpha+\beta$, $\bar\sigma^2$ | persistence ≥ 1; decimals vs % scaling |
| Asymmetric risk (equities)? | leverage plausible? | GJR / EGARCH (Wk 8) | $\gamma$ and its t | sign conventions differ across models |
| Tail risk / VaR? | residual kurtosis | GARCH-t + quantile (Wk 8) | 1% VaR, $\hat\nu$ | normal quantiles understate tails |
| Bull/bear dating? | regimes visible? | Markov switching (Wk 9) | $\mu_i, \sigma_i$, durations, smoothed probs | smoothed ≠ real-time |
| Known break date? | — | Chow test / dummies (Wk 4) | Chow F | data-mined break dates |
| Effect across firms/countries over time? | panel structure; within variation | FE (default) vs RE via Hausman (Wk 10) | clustered t on $\hat\beta$ | default SEs; time-invariant regressors in FE |

## 4. Exam-style checklist

Before you answer any empirical question, run this list:

1. **Data**: levels or returns? Frequency? Stationary (ADF)? Plot it.
2. **Model**: write the equation, define every symbol, state the null
   hypotheses *before* looking at p-values.
3. **Estimation**: OLS / MLE / within — and why that estimator is valid here.
4. **The test**: statistic, distribution under $H_0$, critical value, decision.
   (ADF: reject = stationary. Hausman: reject = FE. JB: reject = non-normal.
   Chow: reject = unstable. Get the directions right.)
5. **Diagnostics**: which assumptions did you check? What changed when you used
   robust/clustered SEs?
6. **Economics**: sign, size (annualise! per-month alphas mislead), and meaning.
   A coefficient is an answer to a question — restate the question.
7. **Limitations**: one honest sentence (causality, sample, look-ahead).

Classic traps, one line each: testing $\beta=0$ when the question is $\beta=1$;
reading the ADF rejection backwards; persistence is $\alpha+\beta$, not
$\beta$; $R^2\approx 1$ in levels means trouble, not triumph; smoothed
probabilities peek at the future; rejecting Hausman means *fixed* effects;
information criteria are minimised, not maximised; VaR needs horizon +
confidence level + distribution; clustering usually *raises* SEs and that is
the point.

## 5. What's in this week's lab — the mock coursework

`lab.ipynb` is a scaled-down version of the assessed empirical project
(3000-word style). One dataset trio — `capm.csv`, `ukhp.csv`,
`currencies.csv` — and four tasks: (A) descriptive statistics and normality;
(B) a CAPM study of General Electric with the full diagnostic battery and a
2008 break test; (C) Box–Jenkins modelling and a 12-month forecast of UK
house-price growth; (D) GARCH risk modelling of sterling with a one-day 1%
VaR. `solutions.ipynb` is a complete worked answer of the kind that earns a
first: every test interpreted, every number turned into a sentence.

## 6. References

- Brooks, C. (2019) *Introductory Econometrics for Finance*, 4th ed., CUP — the whole book; esp. chs. 3–7, 10–13, 19, 21.
- Engle, R.F. (1982), Bollerslev, T. (1986), Hamilton, J.D. (1989),
  Hausman, J.A. (1978), Fama, E.F. and MacBeth, J.D. (1973) "Risk, return, and
  equilibrium: empirical tests", *JPE* 81, 607–636 — the classics met this term.
