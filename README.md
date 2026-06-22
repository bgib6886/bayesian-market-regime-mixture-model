# Identifying Market Regimes with a Bayesian Mixture Model

**TL;DR:** I built a Gibbs sampler from scratch to identify hidden "market
regimes" (crisis, calm growth, expansion) in 15 years of S&P 500 returns.
The more sophisticated 3-regime model looked appealing on paper, but
rigorous convergence diagnostics showed it was statistically unstable — so
I chose the simpler, more reliable 2-regime model instead. The real result
here isn't the regime classification itself, it's the discipline of knowing
when a fancier model is worse than a simpler one.

## Why this matters

Financial returns aren't one consistent process — markets behave very
differently during a crash than during a calm bull run. A natural way to
capture this is a **mixture model**: assume returns are drawn from a small
number of distinct "regimes," each with its own average return and
volatility, and use the data to figure out which days belong to which
regime.

The challenge is that mixture models are notoriously easy to over-specify.
Add too many regimes and the model can't reliably tell them apart anymore —
a well-known statistical problem called **label switching**, where the
algorithm can't consistently distinguish between two regimes that are too
similar to each other. This project demonstrates both building the
technique from first principles, and the diagnostic rigor needed to know
when to trust it.

## What I did

1. **Derived the model's conditional posterior distributions by hand** —
   the mathematical foundation needed before any sampler can be written.
2. **Built a Gibbs sampler from scratch** (no off-the-shelf mixture-model
   library) to estimate a Gaussian mixture model on daily S&P 500 returns,
   with latent regime assignments, regime-specific means and volatilities,
   and mixture weights.
3. **Applied it to 15 years of daily S&P 500 returns** (2010-2024, ~3,770
   observations), testing both a 2-regime and 3-regime specification.
4. **Diagnosed model stability rigorously** — collapse rates, Geweke
   convergence tests, trace plots, and autocorrelation analysis, not just
   "does it run."
5. **Tested a published fix for label switching** (random permutation
   sampling, Frühwirth-Schnatter 2006) to see whether it could rescue the
   unstable 3-regime model.
6. **Made a judgment call on model selection** based on the diagnostics,
   rather than defaulting to the more complex specification.

Full code and output: [`bayesian_market_regimes.ipynb`](./bayesian_market_regimes.ipynb)

## Key results

**The 3-regime model looked interpretable at first glance** — a crisis
regime (-109% annualised return, high volatility, capturing COVID-19 and
2022's bear market), a calm growth regime, and a stronger expansion regime.

**But the diagnostics told a different story:**

| Diagnostic | 3-regime model | 2-regime model |
|---|---|---|
| Label-switching ("collapse") rate | 12.24% (unstable) | 0.02% (stable) |
| Geweke convergence test | Multiple parameters fail | Passes |
| Classification confidence (>90% certain) | 1.4% of observations | 58.9% of observations |
| Posterior shape | Bimodal for 2 of 3 regimes | Clean, unimodal |

The two "growth" regimes in the 3-regime model had means and volatilities
too similar to reliably distinguish — the sampler kept swapping their
labels between iterations, a classic sign of an over-specified model.

**I tested whether a known statistical fix could rescue it.** Random
permutation sampling (relabeling regimes after the fact rather than forcing
an ordering during sampling) is a published method for addressing label
switching — but it only works when regimes are genuinely well-separated.
Applied here, the fix actually got *worse* (99.97% collapse rate, against
12.24% for the simpler ordering approach), confirming the problem wasn't a
quirk of the sampling algorithm — it was that the data itself doesn't
support three distinct regimes over this period.

**Conclusion:** the 2-regime specification — a dominant "growth" regime
(~80% of days, +11% daily mean return) punctuated by a "crisis" regime
(~20% of days, -20% daily mean return, far higher volatility) — is the
reliable, well-identified model for this dataset. The 3-regime model's
extra granularity wasn't supported by the underlying data, however
tempting the more detailed story looked at first.

## Data

Daily S&P 500 adjusted closing prices, December 2009 to December 2024,
sourced from publicly available market data. Returns are calculated as
daily percentage changes in the adjusted closing price. Data file included
in this repository ([`snp500.csv`](./snp500.csv)).

## Tools

Python — custom Gibbs sampler (NumPy/SciPy), `scipy.stats` (inverse-gamma,
Dirichlet, normal distributions), Geweke convergence diagnostics, trace
plot and autocorrelation analysis, matplotlib.

## About this project

Completed as a group project for ECON5120: Bayesian Data Analysis (MSc
Data Analytics for Economics and Finance, University of Glasgow) with group
members 3118182D, 3152218A, and 3149889H. Published here with their
agreement; cleaned up and documented for portfolio purposes.

---
*Benjamin Gibson — [LinkedIn](https://www.linkedin.com/in/benjamin-gibson-ba8503336/) — [email](mailto:bgib6886@gmail.com)*
