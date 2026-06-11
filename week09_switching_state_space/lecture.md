# Week 9 — Switching Models and State-Space Methods

**MANG2074 Financial Econometrics 1 · University of Southampton · Module leader: Ahmad Maaitah**

---

## 1. Motivation: markets have moods

Everything so far assumed one set of parameters for the whole sample. But
financial markets visibly alternate between **regimes**: bull markets (steady
positive returns, low volatility) and bear/crisis markets (negative drift,
violent swings); recessions and expansions; pegged and floating exchange-rate
periods. Forcing one model onto two regimes averages incompatible worlds — the
estimated "mean return" describes neither. Three modelling responses, in
increasing subtlety:

1. **Dummy variables / Chow tests** (Week 4): fine if you *know* the break date.
2. **Threshold models**: the regime switches when an *observable* variable
   crosses a threshold.
3. **Markov switching** (Hamilton, 1989): the regime is **unobserved** and
   follows a Markov chain — the data tell us, probabilistically, which regime
   we were in at each date.

## 2. The Markov switching model

Let $s_t \in \{0, 1\}$ be an unobserved state. The simplest two-regime model
with switching mean and variance:

$$r_t = \mu_{s_t} + \varepsilon_t, \qquad \varepsilon_t \sim N(0, \sigma^2_{s_t})$$

The state evolves as a first-order Markov chain with **transition probabilities**

$$P(s_t = j \mid s_{t-1} = i) = p_{ij}, \qquad
P = \begin{pmatrix} p_{00} & 1-p_{00} \\ 1-p_{11} & p_{11} \end{pmatrix}$$

Key objects and their meaning:

- $p_{00}, p_{11}$ close to 1 ⇒ regimes are *persistent* (not week-to-week noise);
- **expected duration** of regime $i$: $E(D_i) = \dfrac{1}{1 - p_{ii}}$
  (geometric distribution — derive by summing $\sum_d d\, p_{ii}^{d-1}(1-p_{ii})$);
- **smoothed probabilities** $P(s_t = i \mid \text{all data})$: the model's
  retrospective regime dating — the headline output, plotted against history;
- filtered probabilities $P(s_t = i \mid \text{data up to } t)$: the real-time
  version, what an investor could actually have known.

Estimation is by maximum likelihood via the **Hamilton filter**, which
recursively updates regime probabilities as each observation arrives; the
likelihood mixes the two regime densities weighted by those probabilities. In
`statsmodels`:

```python
sm.tsa.MarkovRegression(r, k_regimes=2, switching_variance=True)
```

`MarkovAutoregression` adds switching AR dynamics ($r_t$ depends on lags within
each regime).

## 3. Threshold autoregression (concept)

When the trigger is *observable*, use a **TAR/SETAR** model:

$$y_t = \begin{cases}
\mu_1 + \phi_1 y_{t-1} + u_t & \text{if } y_{t-d} \le \kappa \\
\mu_2 + \phi_2 y_{t-1} + u_t & \text{if } y_{t-d} > \kappa
\end{cases}$$

with threshold $\kappa$ and delay $d$. Finance examples: exchange rates inside
target-zone bands; momentum that behaves differently after large drawdowns;
interest-rate corridors. Markov switching vs TAR is a modelling choice about
*whether the regime is hidden or triggered by something you can see*.

## 4. State-space form and the Kalman filter (a first look)

A huge class of models can be written as a **state-space system**:

$$\underbrace{y_t = Z\,\alpha_t + \epsilon_t}_{\text{measurement equation}} \qquad\qquad
\underbrace{\alpha_t = T\,\alpha_{t-1} + \eta_t}_{\text{state (transition) equation}}$$

— an observed series $y_t$ driven by an unobserved, evolving state $\alpha_t$. The **Kalman filter** is the recursive
algorithm that optimally (i) predicts the state, (ii) updates the prediction
when each new $y_t$ arrives, and (iii) delivers the likelihood as a by-product.
Finance applications: time-varying CAPM betas ($\beta_t$ as the state),
extracting an unobserved "fundamental value", local-level (random-walk-plus-
noise) trend extraction. Markov switching is the discrete-state cousin of this
continuous-state machinery; ARMA models themselves are estimated in state-space
form by `statsmodels`.

## 5. Diagnostics and interpretation discipline

- **Label regimes by their estimates**, not by their index: regime "0" is
  whatever the optimiser made it. Check $\mu$, $\sigma^2$, then name them
  (calm/bull vs turbulent/bear).
- Smoothed probabilities should classify decisively (near 0 or 1 most dates);
  probabilities hovering around 0.5 signal a weakly identified model.
- Compare regime dating with known history (recessions, crises) — face validity.
- Check persistence: $p_{ii} \approx 0.5$ means "regimes" are just noise.

## 6. Common pitfalls in finance applications

- **Too many regimes**: each extra regime adds many parameters; with a few
  hundred observations, two is usually the honest maximum.
- **Switching mean only**: financial regimes differ mainly in *variance*; set
  `switching_variance=True` or the model will use the mean to mimic variance shifts.
- **Treating smoothed probabilities as a trading signal**: they use the full
  sample — future data! Real-time strategies must use *filtered* probabilities.
- **Local maxima**: switching likelihoods are multi-modal; try several starting
  values (`search_reps`) before trusting estimates.
- **Reading durations in the wrong units** — they are in the data's frequency
  (months here).

## 7. What's in this week's lab

Using `data_monthly.csv` (monthly equity-style returns, 1985–2023):

1. fit a two-regime Markov switching model with switching mean and variance;
2. interpret $\mu_i$, $\sigma_i$, the transition matrix and expected durations;
3. plot smoothed regime probabilities and date the turbulent episodes;
4. extend to a Markov *autoregression*; compare;
5. (taster) a local-level state-space model estimated by the Kalman filter.

## References

- Brooks, C. (2019) *Introductory Econometrics for Finance*, 4th ed., CUP — switching models chapter.
- Hamilton, J.D. (1989) "A new approach to the economic analysis of
  nonstationary time series and the business cycle", *Econometrica* 57, 357–384.
- Ang, A. and Timmermann, A. (2012) "Regime changes and financial markets",
  *Annual Review of Financial Economics* 4, 313–337.
