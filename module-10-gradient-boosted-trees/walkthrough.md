# Module 10 — Walkthrough: Gradient-Boosted Trees on the WRDS Panel

## 1. What gradient boosting does

Gradient boosting builds a prediction function as a sum of small decision trees. Each new tree is trained to fix what the previous ones are still getting wrong:

$$F_M(x) = \sum_{m=1}^{M} \nu \cdot h_m(x).$$

Here $h_m$ is the $m$-th tree and $\nu \in (0, 1]$ is the learning rate. The "gradient" piece refers to how each new tree gets fit. At step $m$, the algorithm computes the negative gradient of the loss against the current prediction,

$$r_{i,m} = -\left[\frac{\partial L(y_i, F(x_i))}{\partial F(x_i)}\right]_{F = F_{m-1}},$$

and fits $h_m$ to those pseudo-residuals. When the loss is squared error, the gradient is just $y_i - F_{m-1}(x_i)$, the residual. So in the most common case, gradient boosting is "fit a small tree to the residuals, add it in, repeat."

*Optional further reading: [Introduction to Boosted Trees](https://xgboost.readthedocs.io/en/stable/tutorials/model.html) (XGBoost docs) — a short, clear derivation of the same additive-tree algorithm, including the regularized objective.*

Three knobs do most of the work. `n_estimators` is the number of trees; more helps until it overfits. `learning_rate` is how much each tree contributes; lower values are more conservative and need more trees to compensate. `max_depth` is the per-tree complexity, where 4 to 8 is the sweet spot for tabular finance data. Beyond that, trees start memorizing.

Gradient boosting dominates tabular work because it asks for almost nothing. No feature scaling. No imputation — it learns which side of a split missing values go to. Non-linear interactions get picked up automatically, which is exactly where OLS leaves signal on the table. On Kaggle-style tabular problems it has, for years, beaten linear models and deep nets both.

It is not, though, a replacement for OLS. Gradient boosting is a prediction tool. The "feature importance" it returns isn't a causal coefficient; SHAP values are an attribution decomposition, not "the effect of $X$ on $Y$ holding everything else fixed." If your question is "does R&D level affect future ROA, controlling for other factors," OLS is still the right tool. If your question is "given everything we observe about this firm, what's our best guess at next year's ROA," gradient boosting is.

## 2. Branch off main

Same workflow as Module 9. From here on this section is one line:

```bash
git checkout main && git pull && git checkout -b feature/m10-gbm
```

## 3. Train/test split — by time, not at random

Decide the split before you brief the agent. This is the single most common place a tabular model goes wrong on panel data.

A random 80/20 split picks (firm, year) cells at random, which means the model gets to peek at firm A's 2024 outcome while training on firm A's 2018–2023. In-sample performance looks great. Out-of-sample performance on actual future data is much worse. A time split — train on 2010–2020, test on 2021–2024 — fixes that, because every test observation comes from years the model hasn't seen.

If your agent silently random-splits a panel, you have to catch it. This is exactly the kind of plausible-but-wrong code Module 6 warned about.

## 4. Brief the agent

Before you hand the brief to the agent, two quick questions: what is this model actually doing for an accountant, and why are we re-running Module 9's specification with a different algorithm?

Gradient boosting is the workhorse of modern quantitative accounting and finance. Credit-risk shops use it to score borrowers; it's the model behind the modern successors to FICO and the Altman Z-score. Audit teams use it for fraud and anomaly flagging on transaction-level data — when you can't read every journal entry by hand, a GBM trained on prior-year exceptions is what triages them. Sell-side analysts use it for earnings forecast refinement. In academic accounting research, gradient boosting is the standard non-linear benchmark a linear model has to beat to claim it captured the meaningful variation.

The move from OLS to GBM is really a move in what you're optimizing for. OLS answers "what's the effect of $X$ on $Y$, controlling for $Z$." GBM answers "given everything I observe, what's my best guess at $Y$." Most of the day-to-day work an entry-level analyst or staff auditor does — fraud triage, audit scoping, deal screening, going-concern flagging — is the second kind of question. You want the best prediction, not a defensible coefficient.

The demo runs both models on the same target and features as Module 9 (next-year ROA from current-year operating ratios, plus industry FE), splits the panel by time, and reports test RMSE and R² side by side. The interesting number is the gap. If GBM beats OLS by a lot, there's non-linear signal in those three ratios — interactions, thresholds, non-monotonicities — that OLS couldn't represent. If GBM doesn't beat OLS, the linear baseline was already capturing what was capturable on this question, and the extra complexity didn't earn its keep. Write up whatever you find.

The brief:

> **Brief.** Add `src/models/gbm.py` with one function:
>
> ```python
> def fit_gbm(
>     train: pd.DataFrame, test: pd.DataFrame,
>     target: str, features: list[str],
>     params: dict | None = None,
> ) -> tuple["xgboost.XGBRegressor", pd.Series]:
> ```
>
> Returns the fitted booster and predictions on `test`. Default params: `n_estimators=500`, `learning_rate=0.05`, `max_depth=5`, `early_stopping_rounds=20`, `eval_metric="rmse"`. Use `xgboost.XGBRegressor` (sklearn API). Pass all params (including `early_stopping_rounds` and `eval_metric`) to the **constructor** — XGBoost ≥ 2.1 removed them from `.fit()`. Provide an `eval_set=[(X_val, y_val)]` to `.fit()` so early stopping has something to monitor — and carve that validation slice **from the end of `train`** (the last two training years, `fyear` 2019–2020), never from `test`. Early-stopping against the test set is itself a leak: you'd be tuning the number of trees on the rows you're grading yourself on.
>
> Then `notebooks/m10-gbm.ipynb`:
>
> 1. Loads `data/raw/fundamentals.parquet` and validates with the Module 8 schema.
> 2. Constructs the **same target and features as Module 9** (1-year-ahead ROA; current-year `revt/at`, `xrd/at`, `oancf/at`, industry `sich`).
> 3. **Splits by time:** `train = df[df.fyear <= 2020]`, `test = df[df.fyear >= 2021]`. Print row counts.
> 4. Calls `fit_gbm`. Reports test RMSE and test R². Report the same two numbers for the **Module 9 OLS** baseline on the same split, so the comparison is apples-to-apples.
> 5. Generates a SHAP summary plot (`shap.summary_plot`) on the test set and saves to `results/m10/`.
>
> Add `tests/test_gbm.py` with one passing test on synthetic data.
>
> `uv add xgboost "shap" "numba>=0.61"` (pinning `numba>=0.61` keeps the resolver on a `numba`/`llvmlite` that ships a Python 3.13 wheel — 0.61 is the first `numba` release with a `cp313` wheel). Do **not** modify `src/pull_fundamentals.py`, `src/pull_mdna.py`, or `src/models/ols.py`.

The brief is doing four things a sloppy prompt would skip. It pins the target and features to Module 9 so the comparison number actually means something. It names the time split explicitly, which kills the temptation for the agent to reach for `train_test_split(shuffle=True)`. It demands the OLS baseline on the same split, because the question of the module is whether gradient boosting buys you anything. And it requires `early_stopping_rounds`, without which the agent will cargo-cult `n_estimators=500` and happily overfit.

## 5. Run, read, and compare to OLS

> **macOS only:** XGBoost's wheel needs the OpenMP runtime, which `pip`/`uv` don't install. If `import xgboost` fails with a `libomp.dylib` error, run `brew install libomp` once.

```bash
uv run jupyter lab notebooks/m10-gbm.ipynb
uv run pytest tests/test_gbm.py
```

The interesting cell is the comparison. You want a small table:

| Model | Test RMSE | Test R² |
|---|---|---|
| OLS (M9 spec) | 0.083 | 0.12 |
| GBM (default) | 0.071 | 0.36 |

(The two columns are redundant given the same test set — $R^2 = 1 - \text{RMSE}^2 / \text{Var}(y_{\text{test}})$ — so check your own table for consistency: numbers that don't satisfy the identity mean the two models were scored on different rows.)

If GBM is worse than OLS on the test set, that's a real result worth reporting. Your features, on this target, on this sample, don't contain non-linear signal worth capturing. Say so plainly.

If GBM is meaningfully better, the next question is why. Start with the SHAP summary plot. Each row is a feature, each dot a test observation, color encodes the feature value, and the x-axis is the SHAP contribution. Look for non-monotonic patterns. If `xrd/at` shows a U-shape, that's the kind of signal OLS literally can't represent. Then look at the top-importance features. If one of them dominates and it shouldn't — `at` itself, for example — you have a leakage problem in the spec, not a tuning problem. `ni/at` and `at` on the same side of the regression is a structural issue.

## 6. SHAP is not a coefficient

Worth one paragraph in your write-up, because graders want to know you know:

> SHAP values decompose each prediction into per-feature contributions that sum to the model's deviation from the average prediction. They're an attribution tool, not a causal one. A SHAP value of +0.02 for `xrd/at` on firm $i$ does not mean "if firm $i$ raised R&D intensity by one unit, ROA would be 0.02 higher." It means "given what the model has learned, firm $i$'s R&D intensity is contributing +0.02 to the model's prediction relative to the average firm." For causal claims, OLS with an identification strategy — or the causal-ML methods you won't see until graduate work — is what you reach for.

*Optional further reading: [SHAP documentation](https://shap.readthedocs.io/) (Lundberg and Lee, 2017) — the reference for the interpretation library, including the summary plot used in §5.*

## 7. Light robustness pass

Two checks, two commits.

Refit with a different seed (`random_state=7` instead of the default). The headline RMSE should move by less than about 5%. If it moves more, the model is unstable on this sample and you should say so.

Refit on a shorter train window — 2015–2020 only, test still on 2021–2024. If results collapse, the model was leaning on 2010–2014 patterns that may not generalize.

Be honest about what you find. A model that survives stress tests is a real result. A model that doesn't, but that you write up as if it did, isn't.

## 8. Open a PR and merge

```bash
git push -u origin feature/m10-gbm
gh pr create --title "M10: Gradient-boosted trees vs OLS baseline" --body "$(cat <<'EOF'
## Summary
- Adds `src/models/gbm.py` (XGBoost with early stopping).
- Notebook compares GBM to M9 OLS on the same time-based test split.
- SHAP summary plot in `results/m10/`.

## Headline result
[one sentence: did GBM beat OLS, by how much, on which metric]

## Caveats
- SHAP is attribution, not causation.
- [other caveats — leakage suspects, instability, etc.]
EOF
)"
```

Self-review the diff. The things to actually check: the split is hard-coded by year rather than random shuffled; the comparison metric is computed on the same test rows for both models; the SHAP plots landed in `results/m10/` rather than at the top level; no `data/raw/` file accidentally got staged.

Squash-merge, delete the branch, pull main.

## You're done if…

- [ ] `feature/m10-gbm` is merged into `main`.
- [ ] You can state the GBM-vs-OLS test-set delta as a single number.
- [ ] You did not let the agent random-split a panel.
- [ ] The SHAP plot exists and you've decided whether the top feature is real or a leak.
