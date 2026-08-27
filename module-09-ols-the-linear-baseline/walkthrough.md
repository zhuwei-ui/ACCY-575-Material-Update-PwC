# Module 9 — Walkthrough: OLS on the WRDS Panel

## 1. What OLS does

OLS fits a linear function from features to a target,

$$y_i = \beta_0 + \beta_1 x_{i1} + \beta_2 x_{i2} + \cdots + \beta_k x_{ik} + \varepsilon_i,$$

and picks the coefficients that minimize the sum of squared residuals. The closed form is famous:

$$\hat\beta = \arg\min_\beta \sum_{i=1}^{n} \big(y_i - x_i^\top \beta\big)^2 = (X^\top X)^{-1} X^\top y.$$

A whole chapter of an econometrics text builds up to that one line. There are three things from that chapter worth keeping in your head while you read your own output.

*Optional further reading: Mostly Harmless Econometrics, ch. 3 (Angrist & Pischke) — the classic treatment of OLS in applied work, written for economists but reads cleanly.*

One: the model is linear in $\beta$, not in the variables. You can throw $x^2$, $\log x$, interactions, and dummies into the right-hand side and it's still "linear regression." So when someone tells you OLS can't handle non-linearity, they mean OLS can't handle non-linearity in the coefficients. Non-linearity in the variables is fine.

Two: $\hat\beta_k$ is the expected change in $y$ for a one-unit change in $x_k$, holding the rest of the regressors fixed. The "holding fixed" part is the whole game. Change what's in the regression and every coefficient can shift. Reading a single coefficient without thinking about what else is on the right-hand side will mislead you.

Three: OLS assumes errors that are mean-zero, uncorrelated with the regressors, and (for the textbook standard errors) homoskedastic. Real panel data violates the second and third routinely, which is why we use robust standard errors below. The first assumption is the dangerous one. If your regressors correlate with the error, the coefficients aren't just imprecise. They're biased.

OLS earns its keep when the outcome is continuous, you care about the direction and magnitude of effects rather than pure prediction, and you have a story for why the regressors aren't correlated with the error. Most of accounting research lives in that zone. It doesn't earn its keep for binary outcomes (use a probability model), strongly non-linear relationships (gradient boosting will pick those up in Module 10), wide text features (Module 11), or pure prediction problems where nobody is going to read the coefficients.

## 2. The Part 2 workflow: branch, work, review, merge

Starting this module, every Part 2 model gets its own branch. The pattern is the team workflow you'd run on a real engagement: branch off `main`, brief the agent, let it commit to the branch, read the diff, fix what's wrong, open a PR, self-review it, merge.

You're working alone, but the workflow is the point. Part 3 puts you in groups on real cases, and if "branch → PR → review → merge" is something you're learning for the first time under group pressure, the mechanics will eat the time the analysis needed. The other reason is forced review. A green-and-red diff looks different from the working tree you've been staring at for an hour; bugs that hide in the editor jump out of a diff page.

Be a little sloppy on these per-module PRs. The point isn't perfect commit hygiene; it's that the loop is automatic by Part 3. One commit per PR is fine. A short description is fine. Skipping the PR isn't.

```bash
cd ~/Projects/accy575/ACCY575-wrds-data-analysis
git checkout main
git pull
git checkout -b feature/m9-ols
```

## 3. Brief the agent

Before you hand anything to the agent, be clear on the regression you're specifying. Target: next-year return on assets — `ni / at` led one year. Features: three current-year operating ratios (asset turnover, R&D intensity, operating cash flow to assets), plus industry dummies built from `sich`. Three real variables, a fixed-effects block, a one-year-ahead outcome.

This isn't an arbitrary spec. Predicting next-year ROA from current fundamentals is one of the most-replicated regressions in accounting. A version of it shows up everywhere real work happens. Equity analysts forecast next-year profitability before they forecast price; their starting point looks a lot like this regression. Auditors running analytical procedures under AU-C 520, or building a going-concern assessment under AU-C 570, need a defensible expectation of next year's profitability, and "industry-adjusted operating ratios predict next-year ROA" is exactly that kind of expectation. In academic accounting research, this family of regressions is the workhorse for asking how an accounting choice — a revenue recognition policy, capitalize-vs.-expense, an estimate — maps into future fundamentals. The modern DuPont decomposition (Nissim and Penman 2001) you saw in financial statement analysis is the conceptual ancestor of the feature set.

The accounting part of all this is the interpretability requirement. A black-box model that predicts ROA accurately might be useful for trading. It's useless for an audit workpaper, a 10-K disclosure review, or a research paper, because nobody can defend the prediction in English. OLS is the baseline here because the coefficient is the deliverable, not the prediction.

Two specification choices are worth understanding before you read the brief. The one-year lead exists because regressing this-year ROA on this-year ratios is mostly an identity — `ni` appears on both sides — so the headline coefficient would be mechanical. Leading the target by a year forces the regression to say something about the future, which is the question analysts and auditors actually have. The industry fixed effects exist because a utility's ROA and a software firm's ROA aren't on the same scale and never will be. Without industry dummies you'd mostly be measuring industry composition rather than within-industry variation in the ratios you care about.

The brief:

> **Brief.** Add `src/models/ols.py` with one function:
>
> ```python
> def fit_ols(df: pd.DataFrame, target: str, features: list[str]) -> "statsmodels.regression.linear_model.RegressionResultsWrapper":
> ```
>
> The function should fit OLS using `statsmodels.api.OLS` with HC3 (heteroskedasticity-consistent) standard errors and return the fitted results object. Add an intercept with `sm.add_constant(X)` — `statsmodels.api.OLS` does **not** add one automatically (unlike the formula API), so without it the regression is forced through the origin. Drop rows with any NaN in `target` or `features` *inside* the function and log how many were dropped.
>
> Then add `notebooks/m9-ols.ipynb` (or a script under `notebooks/`) that:
>
> 1. Loads `data/raw/fundamentals.parquet` and validates with the Module 8 schema.
> 2. Constructs target = 1-year-ahead `ni / at` (return on assets, lead 1).
> 3. Constructs features = current-year `revt / at` (asset turnover), `xrd / at` (R&D intensity), `oancf / at` (operating cash flow to assets), plus industry dummies from `sich` built with `pd.get_dummies(..., drop_first=True)` and cast to `float` — dropping one category keeps the dummy block from being collinear with the intercept (the dummy trap), which `statsmodels` won't raise a runtime warning about — though `.summary()` would flag it with a multicollinearity footnote.
> 4. Calls `fit_ols`, prints the summary, and saves four diagnostic plots to `results/m9/`: residuals vs fitted, Q-Q plot, scale-location, leverage. (statsmodels has no one-call helper for all four — residuals-vs-fitted and `sm.qqplot` are direct, but scale-location and leverage are hand-rolled from `results.get_influence()`.)
>
> Add `tests/test_ols.py` with one passing test that calls `fit_ols` on a tiny synthetic frame and checks the coefficients are within a small tolerance of the known answer.
>
> Use `uv add statsmodels matplotlib`. Do **not** modify anything from Module 8 (e.g. `src/pull_fundamentals.py`, `src/pull_mdna.py`).

A short prompt would miss four things this one catches. HC3 standard errors, because default OLS errors assume homoskedasticity and accounting panels never are. The one-year lead on the target, for the reason above. Industry fixed effects, for the reason above. And the test. The agent will skip the test unless you insist. Insist.

## 4. Run, read, and decide what to trust

```bash
uv run jupyter lab notebooks/m9-ols.ipynb       # or run as a script
uv run pytest tests/test_ols.py
```

What to look at in the OLS output:

| Output | What it tells you | What can mislead |
|---|---|---|
| **Coefficients** | Direction and size of each effect, holding the others fixed. | Signs that flip when you add or drop a control — your spec is fragile. |
| **HC3 std. errors** | Coefficient uncertainty. *t* = coef / se. | "Significant" with $n=8{,}000$ is cheap. Look at magnitude, not stars. |
| **R²** | Share of variance explained. | High R² from leakage (e.g., next-year ROA on next-year revenue) means nothing. |
| **Residuals vs fitted** | Should look like a featureless cloud around zero. | A funnel ⇒ heteroskedasticity. You have HC3, but consider logging the target. |
| **Q-Q plot** | Should hug the diagonal. | Heavy tails ⇒ outliers; consider winsorizing. |
| **Leverage plot** | A few high-leverage points are normal. | If one or two points dominate, the result is *those points*, not the population. |

*Optional further reading: [statsmodels OLS reference](https://www.statsmodels.org/stable/regression.html) — the library the agent uses to fit the model; the four diagnostic plots above are built from its `OLSResults` / `get_influence()` objects rather than a single plotting call.*

Pick the coefficient you actually care about and write the result in plain English: *"A 1-percentage-point increase in R&D intensity is associated with a [X]-percentage-point change in next-year ROA, holding turnover and operating cash flow to assets fixed."* If you can't write that sentence without looking at the code, you don't yet understand your own regression.

## 5. Stress-test before you trust

Three quick robustness checks. One commit each.

Drop 2020. If the headline coefficient depends on the COVID year, say so. What you have then is a COVID result, not a long-run one.

Drop a high-leverage industry. Usually financials (`sich` 6000s) or utilities. If the coefficient flips or doubles when you take them out, your headline was that industry.

Winsorize the target at the 1st and 99th percentiles. If the coefficient halves, your result was being driven by a handful of extreme firms rather than the bulk of the panel.

These are fast for the agent — one-line briefs each time. Something like: *"Re-run the regression with `df.query('fyear != 2020')`, compare the coefficient on `xrd_intensity`, and append a one-liner to the notebook."* Three commits, three notes.

## 6. Open a PR and merge

When the branch looks right:

```bash
git push -u origin feature/m9-ols
gh pr create --title "M9: OLS baseline on Compustat fundamentals" --body "$(cat <<'EOF'
## Summary
- Adds `src/models/ols.py` (HC3 OLS).
- Notebook `notebooks/m9-ols.ipynb` with fundamentals → next-year ROA regression.
- Three robustness checks: drop 2020, drop financials, winsorize target.

## Headline result
[one sentence — you write this]

## Caveats
[two or three bullets — leakage suspects, sample selection, industry coverage]
EOF
)"
```

Open the PR in your browser and read your own diff one screen at a time. Things that hide in the working tree but jump out of a diff page:

- did the agent quietly change anything outside `src/models/`, `notebooks/`, or `tests/`?
- is `data/raw/` accidentally staged?
- does the test actually test something, or is it `assert True`?
- are there imports nothing uses anymore?

Fix anything that looks off with another commit on the same branch. When it's clean:

```bash
gh pr merge --squash --delete-branch
git checkout main
git pull
```

Squash because six commits ("wip", "fix lint", "actually fix lint") collapse to one readable commit on `main`. The PR page keeps the full history if you ever want it.

## You're done if…

- [ ] `feature/m9-ols` was opened as a PR, self-reviewed, and merged into `main`.
- [ ] You can write the headline-coefficient sentence in plain English without looking at the code.
- [ ] At least two of three robustness checks ran and you noted whether the headline survived.
- [ ] `uv run pytest` is green on `main`.
