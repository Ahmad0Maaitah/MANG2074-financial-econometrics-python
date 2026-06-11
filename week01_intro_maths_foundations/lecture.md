# Week 1 — Introduction & Mathematical Foundations

**MANG2074 Financial Econometrics 1 · University of Southampton · Module leader: Ahmad Maaitah**

---

## 1. Motivation: why econometrics for finance?

Suppose you are an analyst asked any of the following:

- Has the S&P 500 become riskier over the last decade?
- Are UK house prices growing faster than inflation, and is that growth predictable?
- Is the distribution of daily equity returns really "bell-shaped", as the textbook
  Black–Scholes world assumes?

None of these can be answered by staring at a price chart. **Financial econometrics**
is the application of statistical techniques to financial data to measure, test and
forecast. This week we build the foundations: how to transform prices into returns,
how to summarise the distribution of returns, and how to test formally whether
returns are normally distributed. Everything is done in Python with `pandas`.

## 2. Prices, simple returns and log returns

Let $P_t$ denote the price of an asset (or the level of an index) at time $t$.

**Simple (arithmetic) return:**

$$R_t = \frac{P_t - P_{t-1}}{P_{t-1}} = \frac{P_t}{P_{t-1}} - 1$$

**Log (continuously compounded) return:**

$$r_t = \ln P_t - \ln P_{t-1} = \ln\!\left(\frac{P_t}{P_{t-1}}\right)$$

Both are usually multiplied by 100 and quoted in percent.

### Why two definitions?

- **Log returns aggregate over time.** The $k$-period log return is just the sum of
  one-period log returns: $r_t(k) = r_t + r_{t-1} + \dots + r_{t-k+1}$. This makes
  time-series modelling (Weeks 5–8) much cleaner.
- **Simple returns aggregate across assets.** The return on a portfolio is the
  value-weighted average of the *simple* returns of its constituents — this is *not*
  true for log returns.
- For small returns the two are nearly identical, since $\ln(1+x) \approx x$ for
  small $x$. For daily data the difference is negligible; for monthly or annual data
  it can matter.

A useful exact relationship: $r_t = \ln(1 + R_t)$, hence $r_t \le R_t$ always.

### Why not model prices directly?

Price series trend and wander without settling around a fixed mean — they are
**non-stationary** (formally: they contain a unit root, a topic we treat properly in
Week 5). Standard regression theory assumes stationarity; applying it to trending
prices produces spurious results. Returns, by contrast, are typically stationary:
their mean and variance do not drift over time. **Rule of thumb: model returns, not
prices** (unless you have a good reason, e.g. cointegration analysis).

## 3. Summary statistics for returns

For a sample of returns $r_1, \dots, r_T$:

**Mean:** $\bar{r} = \frac{1}{T}\sum_{t=1}^{T} r_t$

**Variance and standard deviation (volatility):**

$$\hat{\sigma}^2 = \frac{1}{T-1}\sum_{t=1}^{T}(r_t - \bar{r})^2, \qquad \hat{\sigma} = \sqrt{\hat{\sigma}^2}$$

In finance, the standard deviation of returns is called **volatility** and is the
workhorse measure of risk. Annualisation: multiply a daily volatility by
$\sqrt{252}$ (trading days), a monthly volatility by $\sqrt{12}$.

**Skewness** (third standardised moment) measures asymmetry:

$$\text{skew} = \frac{\frac{1}{T}\sum_t (r_t-\bar{r})^3}{\hat{\sigma}^3}$$

Negative skew — large crashes are more common than equally large booms — is typical
of equity index returns.

**Kurtosis** (fourth standardised moment) measures tail thickness:

$$\text{kurt} = \frac{\frac{1}{T}\sum_t (r_t-\bar{r})^4}{\hat{\sigma}^4}$$

A normal distribution has kurtosis exactly 3. **Excess kurtosis** is kurt $-$ 3.
Financial returns almost always display kurt $\gg 3$: they are **leptokurtic** or
"fat-tailed" — extreme days happen far more often than the normal distribution
predicts. (Beware: `pandas`' `.kurtosis()` and `scipy.stats.kurtosis` report
*excess* kurtosis by default.)

## 4. Testing normality: the Jarque–Bera test

The Jarque–Bera (1980) statistic combines skewness $S$ and excess kurtosis $K-3$:

$$JB = T\left[\frac{S^2}{6} + \frac{(K-3)^2}{24}\right]$$

Under the null hypothesis that the data are normally distributed,
$JB \sim \chi^2(2)$ asymptotically. The 5% critical value of $\chi^2(2)$ is 5.99.

- $H_0$: normality (skewness $= 0$ **and** excess kurtosis $= 0$).
- Large $JB$ (small p-value) $\Rightarrow$ **reject** normality.

For daily equity returns $JB$ is typically in the thousands — normality is rejected
overwhelmingly, driven mostly by the kurtosis term. This single fact motivates much
of the second half of the module (GARCH models generate fat tails endogenously).

## 5. Key assumptions and caveats

The statistics above are only meaningful estimates if the return series is:

1. **Stationary** — moments exist and are constant over time;
2. **Not dominated by a few data errors** — always plot the data first; a misplaced
   decimal point produces a "return" of 900% and wrecks every moment estimate;
3. **Sampled at a consistent frequency** — mixing daily and monthly observations,
   or ignoring missing days, distorts volatility.

## 6. The pandas workflow

Every empirical exercise in this module follows the same pipeline:

1. **Load**: `df = pd.read_csv('../data/sp500.csv', index_col=0, parse_dates=True)`
2. **Inspect**: `df.head()`, `df.info()`, `df.describe()` — check types, missing
   values, date range.
3. **Transform**: create returns, e.g.
   `df['ret'] = 100 * np.log(df['sp500']).diff()` then `df = df.dropna()`.
4. **Visualise**: `df.plot()` for levels, returns and histograms.
5. **Analyse**: summary statistics, tests, models.
6. **Interpret**: numbers without economic interpretation earn no marks.

## 7. Common pitfalls in finance applications

- **Using `pct_change()` when you meant log returns** (or vice versa) and then
  summing simple returns over time — they don't sum.
- **Forgetting `dropna()`** after differencing: the first observation is `NaN` and
  some routines fail silently or loudly.
- **Comparing volatilities at different frequencies** without annualising.
- **Reading kurtosis output without checking the convention** (raw vs excess).
- **Treating the house-price *level* as if it were a return** — levels trend, and
  trending data invalidate the usual t-statistics (more in Weeks 2 and 5).

## 8. What's in this week's lab

Using `sp500.csv` (daily S&P 500, 1950–2018) and `ukhp.csv` (monthly UK average
house prices, 1991–2018) you will:

1. load and inspect both datasets in `pandas`;
2. plot price levels and compute simple and log returns;
3. compare simple vs log returns numerically;
4. compute mean, volatility (incl. annualised), skewness and kurtosis;
5. plot histograms of returns against a fitted normal density;
6. run the Jarque–Bera test and interpret it;
7. write a short verdict: are S&P 500 returns normal? Are house-price changes?

## References

- Brooks, C. (2019) *Introductory Econometrics for Finance*, 4th ed., CUP — Chapters 1–2.
- Jarque, C.M. and Bera, A.K. (1980) "Efficient tests for normality, homoscedasticity
  and serial independence of regression residuals", *Economics Letters* 6, 255–259.
- Cont, R. (2001) "Empirical properties of asset returns: stylized facts and
  statistical issues", *Quantitative Finance* 1, 223–236.
