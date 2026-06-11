# Week 4 — LAB: Regression Diagnostics and Violations of the OLS Assumptions

**MANG2074 Financial Econometrics 1 · University of Southampton · Module leader: Ahmad Maaitah**

*This is a lab week: a short briefing, then hands-on work in `lab.ipynb`.*

---

## 1. Why this lab matters

Last week we estimated an APT-style model for Microsoft and happily read off
t-ratios and an F-test. Every one of those numbers is only valid **if the
classical assumptions hold**. This week you become the model's auditor: for each
assumption there is (i) a formal test, (ii) a consequence if violated, and
(iii) a remedy. This diagnostic toolkit is what distinguishes an empirical
finance report that survives scrutiny from one that does not — and it is exactly
what you must include in the coursework project.

## 2. The toolkit at a glance

| Assumption | Violation | Test(s) | Consequence | Common remedy |
|---|---|---|---|---|
| Var$(u_t)=\sigma^2$ | Heteroscedasticity | Breusch–Pagan; White | SEs wrong, t/F invalid | White (HC) robust SEs |
| Cov$(u_t,u_s)=0$ | Autocorrelation | Durbin–Watson; Breusch–Godfrey | SEs wrong, t/F invalid | Newey–West (HAC) SEs; re-specify dynamics |
| $u_t \sim N$ | Non-normality | Jarque–Bera on residuals | exact small-sample tests invalid | dummies for outliers; rely on CLT |
| No exact collinearity | Multicollinearity | correlation matrix; VIF | huge SEs, unstable $\hat\beta$ | drop/combine regressors; more data |
| Correct functional form | Misspecification | Ramsey RESET | biased estimates | add nonlinear terms / re-specify |
| Constant parameters | Structural break | Chow test; sub-sample dummies | averaging across regimes | split sample; interact with dummies |

## 3. Key formulas

**Breusch–Pagan / White.** Regress squared residuals $\hat u_t^2$ on the
regressors (BP) or on regressors, their squares and cross-products (White);
$T \cdot R^2$ from this auxiliary regression is $\chi^2(m)$ under homoscedasticity.

**Durbin–Watson.** $DW \approx 2(1-\hat\rho)$ where $\hat\rho$ is the first-order
residual autocorrelation. $DW = 2$ ⇒ no autocorrelation; $DW \to 0$ ⇒ positive;
$DW \to 4$ ⇒ negative. Only tests order 1 and is invalid with lagged dependent
variables — prefer **Breusch–Godfrey**, which tests up to order $r$ via an
auxiliary regression of $\hat u_t$ on the regressors and $r$ lagged residuals.

**Newey–West (HAC) standard errors** are robust to *both* heteroscedasticity and
autocorrelation up to a chosen lag; White (HC) SEs handle heteroscedasticity
only. Coefficients are unchanged — only the SEs (hence t-ratios) move.

**VIF.** For regressor $j$: $VIF_j = 1/(1-R_j^2)$, where $R_j^2$ comes from
regressing $x_j$ on all other regressors. Rule of thumb: $VIF > 10$ is serious.

**Ramsey RESET.** Add powers of the fitted values ($\hat y_t^2, \hat y_t^3$) to
the regression; their joint significance signals neglected nonlinearity.

**Chow test.** Split the sample at a candidate break date; with $RSS_p$ from the
pooled model and $RSS_1, RSS_2$ from the sub-samples:

$$F = \frac{(RSS_p - (RSS_1+RSS_2))/k}{(RSS_1+RSS_2)/(T-2k)} \sim F(k,\,T-2k)$$

under $H_0$ of parameter constancy.

## 4. Common pitfalls in finance applications

- Reporting robust SEs *only when they help your story* — decide on the basis of
  the diagnostic tests, then stick with it.
- Concluding "no autocorrelation" from DW alone in a model with lagged returns.
- Treating residual non-normality as fatal: with $T \approx 380$ the CLT does a
  lot of work; outlier dummies usually fix the worst of it (but document them —
  each dummy literally sets that month's residual to zero).
- Chow-testing a break date you *chose by looking at the residuals* — the test's
  distribution assumes the date is fixed a priori (e.g. a known crisis date).
- Confusing multicollinearity (an X-data problem, inference still valid) with
  endogeneity (a bias problem, inference invalid).

## 5. What's in this week's lab

Working with the Week 3 APT model for Microsoft (`macro.csv`) and the Ford CAPM
regression (`capm.csv`):

1. residual plots; Breusch–Pagan and White tests;
2. White robust and Newey–West SEs, compared with OLS SEs;
3. Durbin–Watson and Breusch–Godfrey (12 lags);
4. Jarque–Bera on residuals; impulse dummies for the largest outliers;
5. correlation matrix and VIFs for the factors;
6. Ramsey RESET;
7. Chow test at the financial-crisis break (January 2008) for the CAPM model.

## References

- Brooks, C. (2019) *Introductory Econometrics for Finance*, 4th ed., CUP — Chapter 10 (diagnostic testing throughout).
- White, H. (1980) "A heteroskedasticity-consistent covariance matrix estimator
  and a direct test for heteroskedasticity", *Econometrica* 48, 817–838.
- Newey, W.K. and West, K.D. (1987) "A simple, positive semi-definite,
  heteroskedasticity and autocorrelation consistent covariance matrix",
  *Econometrica* 55, 703–708.
