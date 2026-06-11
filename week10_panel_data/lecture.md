# Week 10 — Panel Data: Pooled OLS, Fixed Effects, Random Effects

**MANG2074 Financial Econometrics 1 · University of Southampton · Module leader: Ahmad Maaitah**

---

## 1. Motivation: two dimensions are better than one

Does a stock's CAPM beta predict its return *across firms*? Does union
membership raise wages? Pure time-series data (one firm over time) or one
cross-section (many firms in one year) each answer weakly. A **panel** follows
$N$ entities over $T$ periods — e.g. 1,734 firms over 11 years (`panelx.csv`),
or 595 workers over 7 years (`panel_wage.csv`) — and offers three big prizes:

1. **More information**: $NT$ observations, more variation, more power.
2. **Dynamics + heterogeneity together**: how relationships evolve *and* differ.
3. The killer feature: control for **unobserved, time-invariant heterogeneity**
   (managerial quality, worker ability, firm culture) *without measuring it*.

## 2. The model and the problem with pooling

$$y_{it} = \alpha + \beta x_{it} + u_{it}, \qquad i = 1,\dots,N,\; t = 1,\dots,T$$

**Pooled OLS** stacks all observations and ignores the panel structure. It is
consistent only if the composite error is uncorrelated with $x_{it}$. Decompose

$$u_{it} = \mu_i + v_{it}$$

where $\mu_i$ is an entity-specific, time-invariant effect. If
$\text{Cov}(\mu_i, x_{it}) \ne 0$ — able workers select into certain jobs,
well-run firms have low betas — pooled OLS is **biased** (omitted-variable bias
in disguise).

## 3. Fixed effects (FE)

Treat $\mu_i$ as $N$ fixed intercepts. Two algebraically identical routes:

**LSDV (least squares dummy variables):** include a dummy for every entity:

$$y_{it} = \beta x_{it} + \sum_{j=1}^{N} \mu_j D_{j,it} + v_{it}$$

**Within transformation:** demean each variable by its entity mean. Since
$\bar{y}_i = \alpha + \beta\bar{x}_i + \mu_i + \bar{v}_i$, subtracting gives

$$(y_{it} - \bar{y}_i) = \beta (x_{it} - \bar{x}_i) + (v_{it} - \bar{v}_i)$$

— $\mu_i$ is *annihilated*, whatever it is and however it correlates with $x$.
The two estimators give the **same $\hat\beta$** (the lab verifies this
numerically); the within form is just cheaper than inverting an
$(N+k)$-dimensional matrix.

Costs of FE: (i) anything time-invariant (gender, industry) is swept out too —
its effect cannot be estimated; (ii) only *within-entity* variation identifies
$\beta$, so variables that barely move over time are estimated imprecisely;
(iii) $N$ incidental intercepts burn degrees of freedom.

## 4. Random effects (RE)

Treat $\mu_i \sim (0, \sigma_\mu^2)$ as a random draw **uncorrelated with the
regressors**. RE is a GLS estimator on quasi-demeaned data
$(y_{it} - \theta\bar y_i)$ with $\theta = 1 - \sqrt{\sigma_v^2/(T\sigma_\mu^2 + \sigma_v^2)} \in (0,1)$ — a
weighted average of the within and between information. If its assumption
holds, RE is more **efficient** than FE and can estimate time-invariant
regressors. If the assumption fails, RE is **inconsistent**.

## 5. Choosing: the Hausman test

$$H = (\hat\beta_{FE} - \hat\beta_{RE})'\left[\widehat{\text{Var}}(\hat\beta_{FE}) - \widehat{\text{Var}}(\hat\beta_{RE})\right]^{-1}(\hat\beta_{FE} - \hat\beta_{RE}) \;\sim\; \chi^2(k)$$

Logic: under $H_0$ (RE valid, $\mu_i$ uncorrelated with $x$) both estimators
are consistent and should agree; FE is consistent *either way*. A large $H$
means the estimates diverge, which can only happen if RE is inconsistent.

> **Direction to remember:** reject $H_0$ ⇒ **use fixed effects**.
> Fail to reject ⇒ RE is admissible (and more efficient).

## 6. Inference: cluster-robust standard errors

Within an entity, errors are correlated over time (persistent shocks); default
SEs that assume independence are typically **too small** — sometimes
dramatically. Standard practice in empirical finance: cluster by entity,

```python
PanelOLS.from_formula('ret ~ 1 + beta + EntityEffects', data)\
        .fit(cov_type='clustered', cluster_entity=True)
```

which allows arbitrary correlation within each firm's observations.

## 7. Applications this week

**A. Beta and returns (`panelx.csv`).** Annual returns and betas for 1,734
firms, 1996–2006: does beta explain returns in the cross-section, as the CAPM
demands? (Preview: pooled OLS says no; FE says high-beta years are, if
anything, *low*-return years within a firm — the "beta anomaly" flavour.)

**B. A wage equation (`panel_wage.csv`).** Classic Cornwell–Rupert data: 595
workers, 7 years; log wage on experience (and its square), weeks worked,
marital status, union membership. Pooled vs FE coefficients differ wildly —
textbook evidence of correlated individual ability.

## 8. Common pitfalls in finance applications

- **Forgetting the MultiIndex**: `linearmodels` needs
  `data.set_index(['entity','time'])`. And `return` is a Python keyword —
  rename the column.
- **Estimating gender/industry effects in FE** — they are collinear with the
  entity dummies; software silently drops them or errors.
- **Reading the Hausman direction backwards** (rejection ⇒ FE, not RE).
- **Default SEs in panels** — cluster by entity, or your t-stats flatter you.
- **Unbalanced panels** from `dropna()`: fine for these estimators, but check
  attrition is not informative (firms that die are not random).
- Two-way effects: add `TimeEffects` for common shocks (market-wide years).

## 9. What's in this week's lab

1. Set up both panels with MultiIndexes; describe within vs between variation.
2. Pooled OLS, FE (via `PanelOLS`), RE (via `RandomEffects`) on both datasets.
3. Verify within-demeaning reproduces the FE slope exactly.
4. Hausman tests, computed manually from the FE/RE outputs.
5. Cluster-robust SEs and how conclusions change.

## References

- Brooks, C. (2019) *Introductory Econometrics for Finance*, 4th ed., CUP — Chapter 21 (panel data).
- Hausman, J.A. (1978) "Specification tests in econometrics", *Econometrica* 46, 1251–1271.
- Petersen, M.A. (2009) "Estimating standard errors in finance panel data sets:
  comparing approaches", *Review of Financial Studies* 22, 435–480.
