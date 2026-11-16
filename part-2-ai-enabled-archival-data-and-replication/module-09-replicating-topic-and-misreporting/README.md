# Module 9 — Predicting Misreporting from What a 10-K Talks About

> **Hands-on:** [walkthrough.md](walkthrough.md) — building financial, textual-style, and topic-model predictors of Item 4.02 restatements on the panel from Module 8, evaluated strictly out of sample, following the design in Brown, Crowley, and Elliott (2020).

## Motivation

Catching a material misstatement has always run on numbers: accruals that don't look right, receivables growing faster than sales, margins that beat the industry for no visible reason. Brown, Crowley, and Elliott (2020) added something to that list. A 10-K is mostly words, and management chooses which subjects to spend them on. That choice carries information about the business and its incentives that the ratios miss, and it predicts misreporting on top of the financial red flags. For anyone who screens filings for risk — an auditor planning an engagement, a regulator triaging referrals, an analyst picking up a new name — that is a useful idea, and this module is about learning to implement it. The paper is your guide, because the hard part isn't the topic model: it's the design around it that keeps every prediction honest.

In full: Brown, Crowley, and Elliott (2020), "What Are You Saying? Using *topic* to Detect Financial Misreporting," *Journal of Accounting Research* 58(1): 237–291. The question it answers is one every auditor, regulator, and short seller cares about: **does what a company chooses to talk about in its 10-K help predict that its financial statements are wrong?** The method is a topic model (Latent Dirichlet Allocation) fit on the full text of every 10-K, re-estimated in rolling five-year windows so that no prediction ever uses information from the future. The topic mix adds significant detection power on top of the standard financial red flags and on top of textual "style" measures like readability and sentence length. The published abstract reports that adding topic improves detection by as much as 59%.

You implement the idea on the panel you built in Module 8, which is not their sample: a different period (the XBRL era, 2011–2021, where theirs is 1994–2012), a different label (Item 4.02 non-reliance restatements, where theirs is SEC enforcement actions and irregularity restatements), and a reduced variable set (no stock returns, which EDGAR doesn't carry, and auditor type only as an optional extension extracted from the 10-K text). Those differences are the normal condition of doing this work on data you assembled yourself. Part of the assignment is to reason about which of them could change the answer, and in which direction, *before* you see your results.

## Goals

- Explain the idea and the design in your own words — the outcome, the predictors, the estimation, and the result — precisely enough that someone could build it from your description.
- Build three predictor sets from the Module 8 panel: financial (Dechow et al. 2011 variables from XBRL), textual style (length, readability, sentence structure), and topic (LDA topic proportions).
- Fit the topic model in **rolling five-year windows** with window-specific vocabularies, so that every out-of-sample prediction uses only information available at the time.
- Enforce the **reveal-date rule**: a training window may only "know" about restatements announced before the test year's first filings could be scored.
- Compare seven models (three single predictor sets and their combinations) by pooled, sampling-weighted out-of-sample **AUC**, with bootstrap confidence intervals for the differences.
- Write a short report: what your models say about the idea, what your build covers and what it leaves out, and how your data and design choices shape the answer.

## Going Deeper *(optional)*

*The lecture covers what you need. These are here if you want to dig further.*

- Brown, Crowley, and Elliott (2020), *Journal of Accounting Research* 58(1): 237–291 ([doi:10.1111/1475-679X.12294](https://doi.org/10.1111/1475-679X.12294)). The paper itself. Section 3 (sample and variables), Section 4.2 (out-of-sample design), and Appendix B (variable definitions) are the parts the walkthrough leans on. The library's Wiley access gets you the published version; a 2018 working-paper version circulates on SSRN.
- Blei, Ng, and Jordan (2003), "Latent Dirichlet Allocation," *Journal of Machine Learning Research* 3. The original LDA paper, if you want the generative model behind §6.
- Dechow, Ge, Larson, and Sloan (2011), "Predicting Material Accounting Misstatements," *Contemporary Accounting Research* 28(1). The financial model that serves as the benchmark.
- Bao, Ke, Li, Yu, and Zhang (2020), "Detecting Accounting Fraud in Publicly Traded U.S. Firms Using a Machine Learning Approach," *Journal of Accounting Research* 58(1). The companion paper in the same issue; it uses raw financial data and ensemble learning, and its discussion of evaluation metrics for rare events is worth reading alongside §7.

## Materials

*Slides, in-class exercises, and recordings posted here as the module approaches.*
