# MANG2074 — Financial Econometrics 1 (Python)

**University of Southampton — Southampton Business School**
**Academic year:** 2026–27 | **Module leader:** Ahmad Maaitah

This repository contains the complete set of weekly teaching materials for MANG2074
Financial Econometrics 1, taught in **Python**. The module gives you the skills to
conduct financial modelling and forecasting on real financial data, using a
problem-based learning approach: every technique is introduced through a genuine
finance question (hedging, asset pricing, house-price forecasting, risk management)
and then implemented hands-on in Jupyter notebooks.

## Module aims

By the end of the module you will be able to:

- manipulate, summarise and visualise financial data in `pandas`;
- estimate and interpret linear regression models (CAPM, hedge ratios, APT-style factor models);
- diagnose and remedy violations of the classical OLS assumptions;
- build, select and forecast with univariate time-series models (ARMA/ARIMA);
- model time-varying volatility with ARCH/GARCH-family models and compute simple VaR measures;
- estimate regime-switching models and basic panel-data models (fixed/random effects);
- write up an empirical finance project that combines these tools with sound diagnostics.

## Weekly schedule

| Week | Folder | Topic | Main data |
|------|--------|-------|-----------|
| 1 | [week01_intro_maths_foundations](week01_intro_maths_foundations/) | Introduction & mathematical foundations: prices, returns, distributions | `sp500.csv`, `ukhp.csv` |
| 2 | [week02_clrm_introduction](week02_clrm_introduction/) | The classical linear regression model I: OLS, hedge ratios, CAPM | `sandphedge.csv`, `capm.csv` |
| 3 | [week03_clrm_assumptions](week03_clrm_assumptions/) | CLRM II: assumptions, multiple regression, F-tests (APT-style model) | `macro.csv` |
| 4 | [week04_lab_ols_violations](week04_lab_ols_violations/) | **Lab:** diagnostic testing & violations of OLS assumptions | `macro.csv`, `capm.csv` |
| 5 | [week05_univariate_time_series](week05_univariate_time_series/) | Univariate time series: AR, MA, ARMA, Box–Jenkins | `ukhp.csv` |
| 6 | [week06_lab_arma_forecasting](week06_lab_arma_forecasting/) | **Lab:** ARMA model selection & out-of-sample forecasting | `ukhp.csv` |
| 7 | [week07_volatility_arch_garch](week07_volatility_arch_garch/) | Modelling volatility: ARCH and GARCH | `currencies.csv`, `sp500.csv` |
| 8 | [week08_lab_volatility_modelling](week08_lab_volatility_modelling/) | **Lab:** GJR/EGARCH, volatility forecasts, parametric VaR | `currencies.csv`, `sp500.csv` |
| 9 | [week09_switching_state_space](week09_switching_state_space/) | Switching & state-space models: Markov switching, TAR, Kalman filter | `data_monthly.csv` |
| 10 | [week10_panel_data](week10_panel_data/) | Panel data: pooled OLS, fixed & random effects, Hausman test | `panelx.csv`, `panel_wage.csv` |
| 11 | [week11_lab_panel_markov](week11_lab_panel_markov/) | **Lab:** mini research projects — FDI & growth panel; regime dating | `fd_growth.csv`, `data_monthly.csv` |
| 12 | [week12_revision](week12_revision/) | Revision: which model when, exam checklist, mock empirical project | `capm.csv`, `ukhp.csv`, `currencies.csv` |

Each week folder contains:

- `lecture.md` — lecture notes (rendered maths, derivations, diagnostics, pitfalls);
- `lab.ipynb` — the lab notebook with tasks and starter code (`# YOUR CODE HERE` gaps);
- `solutions.ipynb` — fully worked, executed solutions with interpretation.

## Quick start

1. **Install Python** (3.10+). The easiest route is the
   [Anaconda distribution](https://www.anaconda.com/download), which ships with
   Jupyter. Alternatively install plain Python from python.org.
2. **Clone or download** this repository.
3. **Install the required packages** (from the repository root):

   ```bash
   pip install -r requirements.txt
   ```

4. **Open the notebooks**:

   ```bash
   jupyter notebook
   ```

   then navigate to the week folder and open `lab.ipynb`. Notebooks load data with
   relative paths (`../data/xxx.csv`), so launch Jupyter from the repository root
   (or from the week folder) and do not move notebooks out of their folders.

5. **Google Colab** (optional): upload the whole repository to Google Drive, open a
   notebook with Colab, then either mount your Drive
   (`from google.colab import drive; drive.mount('/content/drive')`) and `%cd` into
   the week folder, or upload the relevant CSV and change the path from
   `../data/xxx.csv` to `xxx.csv`. Run `!pip install arch linearmodels` first —
   everything else is pre-installed on Colab.

## Data dictionary (main datasets)

All data files live in [`data/`](data/). Dates are `YYYY-MM-DD`.

| File | Frequency / span | Columns | Used in |
|------|------------------|---------|---------|
| `sp500.csv` | Daily, 1950–2018 (17,246 obs) | `date`, `sp500` (index level) | Wk 1, 7, 8 |
| `ukhp.csv` | Monthly, 1991–2018 | `Month`, `Average House Price` (£, Nationwide) | Wk 1, 5, 6, 12 |
| `sandphedge.csv` | Monthly, 1997–2018 | `Date`, `Spot`, `Futures` (S&P500 index & futures) | Wk 2 |
| `capm.csv` | Monthly, 2002–2018 | `Date`, `SANDP`, `FORD`, `GE`, `MICROSOFT`, `ORACLE` (prices), `USTB3M` (3-month T-bill, % p.a.) | Wk 2, 4, 12 |
| `macro.csv` | Monthly, 1986–2018 | `Date`, `MICROSOFT`, `SANDP`, `CPI`, `INDPRO`, `M1SUPPLY`, `CCREDIT`, `BMINUSA`, `USTB3M`, `USTB10Y` | Wk 3, 4 |
| `currencies.csv` | Daily, 1998–2018 | `Date`, `EUR`, `GBP`, `JPY` (US dollar exchange rates) | Wk 7, 8, 12 |
| `data_monthly.csv` | Monthly, 1985–2023 | `Date`, `return` (% equity return series) | Wk 9, 11 |
| `panelx.csv` | Annual firm panel | `firm_ident`, `return`, `beta`, `year` | Wk 10 |
| `panel_wage.csv` | Annual worker panel (595 ids × 7 yrs) | `lwage`, `exp`, `exp2`, `ed`, `wks`, demographics | Wk 10 |
| `fd_growth.csv` | Country panel, 2002–2017 (28 ids) | `gdpgrowth`, `FDI`, `schooling`, `population`, `investment`, `trade`, `inflation`, … | Wk 11 |
| `fred.csv` | Monthly Treasury yields | `GS3M`–`GS10` | extra |
| `arch.csv`, `currencies_garch.csv`, `monthlyfactors.csv`, `vw_sizebm_25groups.csv`, `msc_fail_sheet1.csv`, `efficient_sheet1.csv`, `event_*.csv` | — | supplementary datasets for further practice | extra |

## 🎮 Interactive practice site

Every week has a matching interactive practice page — simulations with sliders, a scored game, and code challenges. They open directly in the browser (no installation): **[Practice hub](https://ahmad0maaitah.github.io/MANG2074-practice/)**. Each week folder's README also has a direct **▶ Open the practice page** link.

## Acknowledgement and use

Teaching material and example applications are adapted from **Chris Brooks,
*Introductory Econometrics for Finance*, 4th edition, Cambridge University Press**,
and its accompanying Python guide. The datasets are the standard teaching datasets
distributed with that textbook. This repository is provided for the educational use
of students enrolled on MANG2074 only and must not be redistributed commercially.
