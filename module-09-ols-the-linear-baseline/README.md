# Module 9 — OLS: The Linear Baseline

> **Hands-on:** [walkthrough.md](walkthrough.md) — running OLS on your WRDS panel and reading what comes back.

## Motivation

Linear regression is the most-used analytical tool in accounting research. Most published accounting papers run an OLS at some point. You should be able to read one, run one, and know when it's the right tool — and when it isn't.

OLS is also the right starting point for Part 2. Before you reach for a transformer, run the linear baseline. Half the time the linear model gets you 90% of the way there for 1% of the complexity, and you should know that *before* the agent talks you into something fancier. A model whose answers you can't beat with a one-liner is the model you should be reaching for last, not first.

## Goals

- Frame an accounting question as a regression and explain each coefficient in plain English.
- Read the OLS output that matters — coefficients, robust standard errors, R², residual diagnostics — and skip what doesn't.
- Recognize OLS's failure modes: omitted variables, heteroskedasticity, non-linearity, leakage.
- Direct the agent to fit the regression, but read the diagnostic plots yourself before trusting the result.

## Going Deeper *(optional)*

*The lecture covers what you need. These are here if you want to dig further.*

- *Mostly Harmless Econometrics, ch. 3* — Angrist & Pischke. Written for economists; reads cleanly. The classic reference on OLS in applied work.
- [statsmodels OLS reference](https://www.statsmodels.org/stable/regression.html) — Statsmodels project. The library you'll likely have the agent use; the diagnostic-plots section is the relevant entry.

## Materials

*Slides, in-class exercises, and recordings posted here as the module approaches.*
