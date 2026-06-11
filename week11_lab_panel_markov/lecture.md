# Week 11 — LAB: Panel Analysis and Markov Switching (Guided Mini-Research)

**MANG2074 Financial Econometrics 1 · University of Southampton · Module leader: Ahmad Maaitah**

*This is a lab week — and a dress rehearsal for the coursework. You will run two
small research projects end-to-end: question → model → estimation → diagnostics
→ interpretation.*

---

## 1. Project A — Does FDI cause growth?

A development-finance classic with direct policy stakes: countries spend
billions on investment-promotion agencies. `fd_growth.csv` is a balanced panel
of 28 countries × 16 years (2002–2017) with GDP growth, FDI inflows (% of GDP),
schooling, investment, trade openness and inflation.

**The empirical strategy ladder** (climb it in order):

1. **Pooled OLS** of growth on FDI + controls — the naive benchmark.
2. **Fixed effects** — sweep out time-invariant country characteristics
   (geography, institutions, legal origin) that plausibly correlate with both
   FDI and growth.
3. **Random effects** — efficient *if* country effects are uncorrelated with
   the regressors.
4. **Hausman test** — let the data choose between 2 and 3 (reject ⇒ FE).
5. **Cluster-robust SEs** by country — growth shocks are persistent within
   countries, so default SEs overstate precision.
6. Optionally add **time effects** — global shocks (2008–09!) hit all countries.

**What "interpretation" means here:** sign, size (is 0.04 points of growth per
point of FDI/GDP economically meaningful?), significance under *honest* SEs,
and the limits of causality (FDI chases growth too — reverse causality is not
solved by FE; mention it).

## 2. Project B — Dating bull and bear markets

Using `data_monthly.csv` (1985–2023), estimate the two-regime Markov switching
model of Week 9 and produce a **regime chronology**: dated bear spells, regime
means/volatilities, expected durations, and a comparison of smoothed
(retrospective) with filtered (real-time) probabilities — the difference is the
look-ahead bias a naive backtester would commit.

## 3. Deliverable discipline (transferable to the coursework)

A complete mini-report per project:

- **Question** (one sentence) and **data** (one sentence);
- **Model(s)** with equations;
- **Results table** — estimates with the SEs you defend;
- **Specification tests** — Hausman; regime identification checks;
- **Interpretation** — economic size, not just stars;
- **Limitations** — one honest paragraph beats ten pages of bluster.

## 4. Common pitfalls

- Running the Hausman test and then reporting the *rejected* estimator anyway.
- Forgetting that significance can evaporate under clustered SEs — report the
  clustered version as your headline (here, it changes the FDI conclusion!).
- Cross-country growth regressions are the canonical home of omitted-variable
  and reverse-causality critiques — claim association, not causation.
- In Project B, using smoothed probabilities to claim "a strategy would have
  exited the market in 2008" — only filtered probabilities were knowable.

## 5. References

- Brooks, C. (2019) *Introductory Econometrics for Finance*, 4th ed., CUP — Chapter 21; switching models chapter.
- Hamilton, J.D. (1989) "A new approach to the economic analysis of
  nonstationary time series and the business cycle", *Econometrica* 57, 357–384.
- Borensztein, E., De Gregorio, J. and Lee, J-W. (1998) "How does foreign direct
  investment affect economic growth?", *Journal of International Economics* 45, 115–135.
