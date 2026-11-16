# Module 19 — Gradient-Boosted Trees *(optional)*

> **Optional — Part 4 (Agentic AI Methods and Pipelines).** Not required for the course. Part 4 is a self-paced extension: it builds its own WRDS panel in [Module 17](../module-17-wrds-data-acquisition/README.md) and applies one modern method per module to it. If you take it, do it alongside Part 3, not after it: it runs on the course's WRDS class account, which expires when the term ends, so finish Module 17's pulls, and any other work that needs WRDS itself, before then.

> **Hands-on:** [walkthrough.md](walkthrough.md) — fitting a boosted-tree model on the same WRDS panel and comparing it against your Module 18 OLS.

## Motivation

For tabular accounting and finance data, gradient-boosted trees (XGBoost, LightGBM, CatBoost) are the strongest performers — not transformers, not deep learning. They handle missing values natively, don't need feature scaling, capture non-linear interactions automatically, and consistently beat linear regression and neural networks alike on tabular benchmarks. If you only learn one post-OLS tabular method, learn this one.

The point of this module isn't to make you reach for boosting on every problem — Module 18's OLS is still the right answer plenty of the time. It's to give you the contrasting case: when the linear baseline leaves real signal on the table, gradient boosting is what picks it up. Knowing the gap between the two is itself an analytical result.

## Goals

- Understand boosting conceptually: an additive ensemble of trees fit to the residuals of the previous trees.
- Fit a gradient-boosted model on the Module 18 target and beat the OLS on a held-out set — or explain why you didn't.
- Use SHAP to interpret the model, and recognize that SHAP is attribution, not a causal claim.
- Spot leakage of target information into features before it ruins the comparison.

## Going Deeper *(optional)*

*The walkthrough covers what you need. These are here if you want to dig further.*

- [Introduction to Boosted Trees](https://xgboost.readthedocs.io/en/stable/tutorials/model.html) — XGBoost developers. Short and clear conceptual write-up of the algorithm.
- [SHAP documentation](https://shap.readthedocs.io/) — Lundberg and Lee (2017), *NeurIPS*. Reference for the interpretation library used in §6 of the walkthrough.

## Materials

*Slides and recordings from earlier offerings, if any, are posted here.*
