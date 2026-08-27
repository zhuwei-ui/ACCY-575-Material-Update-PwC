# Module 18 — Engagement Brief: Cost Behavior & Margin Resilience

*You are handed a question and the data to answer it. Which approach you take, what you ask your agent, how you model it — all yours. How you approach it is entirely yours; the deliverable is what's assessed.*

## Your role

You sit in the **FP&A group** of a large public company. The controller is building next year's operating plan under a recession watch, and the **CFO** wants an answer to one question before the plan goes to the board's finance committee.

## The task

> If revenue falls 10–15% next year, how much of our cost actually comes out — and what does that do to operating margin? Give the controller a number they can defend, not a proportional guess.

## The data

**Compustat Fundamentals Annual (`funda`)** — the same S&P 500 firm-year panel you built in **[Module 8](../module-08-wrds-data-acquisition/walkthrough.md)**. No new subscription: reuse `fundamentals.parquet` from your Part 2 repo, or re-pull the fields you need from WRDS the way Module 8 did.

- **One row = one firm-fiscal-year** (`gvkey`, `fyear`).
- The income-statement history lives in fields like `sale` / `revt` (sales), `cogs` (cost of goods sold), `xsga` (SG&A), `xopr` (total operating expense), and `at` (total assets), keyed by `gvkey`, `fyear`, and `sich` (industry). These are S&P's standardized transcriptions of audited 10-K lines — real historical cost behavior.

Your **assigned industry (2-digit SIC) and focal firm** are on the assignment sheet.

## What to hand in

- **A decision memo to the controller / CFO** — your read on how the firm's cost actually behaves, what that implies for operating margin if sales fall, and — plainly stated — which costs are unavoidable in a downturn and which are not. Include the limits of what your estimate can support.
- **A one-page board exhibit** — the margin picture under a range of downside sales scenarios, in a single chart and table a finance committee can read in a minute.
- **The repository behind it** — reproducible in the Part 1 way (`uv`, validation, tests, honest commit history), so your numbers can be re-run.

Bring all of it to the review session and be ready to walk us through your result.

## Ground rules

- Individual work.
- Use AI as much as you want — you own every number in the memo regardless.
- Anything you rely on from outside the data (a benchmark, a definition, a method) you look up and cite yourself.
- No prescribed method, no required tools, no checklist. Figure it out.

## Grading

You are assessed on the **deliverable** — the memo, the exhibit, and the reproducible repository behind them — and on how well it holds up to scrutiny. There is no set procedure to follow. See [Assessment across Part 3](../README.md#assessment-across-part-3).
