# Module 9 — Walkthrough: Predicting Misreporting from 10-K Topics

## 1. The design on one page

Before any code, you need the design in your head precisely enough to build it. Here it is, from the paper's Sections 3–4 and Appendix B. Everything you need is in this table and the briefs below; the paper is in Going Deeper if you want the full detail.

| | Brown, Crowley, and Elliott (2020) |
|---|---|
| **Question** | Does the thematic content of a 10-K (its *topic* mix) help predict intentional financial misreporting, beyond financial red flags and beyond textual style? |
| **Sample** | 10-K filings, 1994–2012. The topic model is fit on every 10-K; the prediction tests exclude financial firms (SIC 6000–6799) and firm-years missing Compustat or CRSP data. |
| **Outcome** | Two separate samples: (1) SEC enforcement actions (AAERs), and (2) "irregularity restatements": restated annual reports whose restatement disclosures or 10-K/As refer to fraud, irregularities, or an SEC, DOJ, or independent investigation, following Hennes, Leone, and Miller (2008). |
| **Financial predictors ("F-score")** | The Dechow et al. (2011) model plus size, audit-firm type, M&A, and restructuring: log assets, RSST accruals, Δreceivables, Δinventory, % soft assets, Δcash sales, ΔROA, actual issuance, operating leases, book-to-market, lagged market-adjusted return, merger/acquisition indicator, Big N and mid-size auditor, net financing, ex ante financing need, restructuring. |
| **Style predictors** | Surface features of the filing's text: length and structure (bullets, parsed size, sentence and paragraph length and their dispersion, repetition), readability (Gunning Fog, Coleman-Liau), vocabulary variety (type-token ratio), active and passive voice, dictionary tone, and counts of all-caps words, exclamation points, and question marks. |
| **Topic predictors** | Latent Dirichlet Allocation on the **full 10-K**, **31 topics**, fit separately in each **rolling five-year window** of filing years (1994–1998, 1995–1999, …) with window-specific stopwords. Each filing's topic proportions come from the window just before its year. |
| **Estimation** | Logistic regression on each five-year window, used to score filings in the following year. For restatements, a firm-year counts as misreported in training only if the restatement was **revealed by the end of the window**. |
| **Evaluation** | Out of sample: 1999–2012 for restatements, 1999–2010 for AAERs. Pooled ROC AUC across test years, with tests of AUC differences between models, plus classification tests at cutoffs. |
| **Headline** | Topic adds significant out-of-sample detection power to both the financial and the style models; the published abstract reports improvements of up to 59%. In AUC terms (in the working-paper version), topic's increment over the financial model is larger for irregularity restatements than for AAERs, which the financial model was originally built to predict. On the classification tests, where the abstract's "as much as" figure comes from, the gain is largest for AAERs. |

Three design choices in that table do most of the work, and your build has to keep all three.

1. **Rolling windows.** The topic model is re-fit every year on only the five prior years. This isn't a detail. A topic model fit on all 19 years at once would "know" which words were about to become common, which is a quiet form of looking into the future.
2. **The reveal-date rule.** A model trained in 2015 can't have learned from a restatement announced in 2018. Their training labels respect that, and yours will too, using the `reveal_date` column you built in Module 8.
3. **Out-of-sample AUC.** Every number that matters is computed on a year the model has never seen. In-sample fit is never the evidence.

## 2. Branch

```bash
cd ~/Projects/accy575/ACCY575-edgar-misreporting
git checkout main && git pull && git checkout -b feature/m9-replication
uv add scikit-learn scipy pyphen joblib
mkdir -p data/features results/m9
```

Everything in this module reads `data/final/panel.parquet` and the text files from Module 8. Nothing is downloaded.

## 3. Your data versus theirs

| | Theirs | Yours | Why the difference |
|---|---|---|---|
| Period | 1994–2012 | fiscal 2011–2021 | XBRL makes financials free from 2011 onward. |
| Window year | Filing year | Compustat `fyear` | Lines up with the XBRL panel. §6 gates each window by filing date so it can't use filings from after the cutoff. |
| Outcome | AAERs; irregularity restatements | Item 4.02 restatements (all), and the subset flagged as irregularities (Module 8 §8a) | 4.02s are public and dated. AAER data comes from a curated research dataset. |
| Sample | Topics: all 10-Ks. Prediction: non-financial firms with Compustat/CRSP data | Topics: the Module 8 text sample. Prediction: non-financial firms in that sample with complete XBRL inputs | Case-control: every restated CIK-year plus 20% of the rest, with sampling weights (Module 8 §8b). |
| Financial | 17 variables | The subset EDGAR supports (§4) | No stock prices or returns. Auditor type needs text extraction (an optional extension); leases, M&A, restructuring, and ex ante financing are left out for simplicity. |
| Style | ~20 variables incl. dictionary tone and voice | Surface and readability measures (§5) | Word-list tone is left out to keep this module on the text representations it builds; you'll build Loughran-McDonald tone by hand in the Module 13 case. Passive voice is an optional extension. Bullets, newlines, HTML tags, header length, and paragraph measures can't be computed, because Module 8 strips HTML and collapses whitespace. |
| Topics | 31 per window, full 10-K | 31 per window, full 10-K, the paper's LDA settings except the learning offset (§6c) | Fit on a label-independent 20% sample of your window's filings (`random20`, Module 8 §8b) rather than every 10-K; no dictionary filter on words; stopword cutoffs rescaled to shares of the window (§6b); proportions from `LatentDirichletAllocation.transform` rather than the paper's projection (§8 offers the projection as a robustness check). |
| Test years | 1999–2012 | 2016–2021 | Five-year windows starting in 2011. |

One more difference isn't in any table, because it's a fact about the period, not a choice: the 2021 wave of SPAC restatements (Module 8 §7b). A large share of all 4.02 positives in fiscal 2020–2021 belong to blank-check shells, whose 10-Ks a topic model could pick out from vocabulary alone ("trust account," "business combination"). Your prediction sample excludes them twice over: they are SIC 6770, inside the paper's financial-firm exclusion (6000–6799), and they tag no revenue, PP&E, or pretax income, so they fail §4's complete-inputs rule. When you build the evaluation sample, confirm `blank_check` is zero, and report how many restated 2020–2021 CIK-years the exclusion removed. (Companies that have since merged out of a SPAC carry their new SIC code, because `sic` in the submissions data is the company's current code, so `blank_check` misses their shell years; the complete-inputs rule still drops those.) The same staff statement also reached companies that had *already* completed their SPAC merger. Nikola, Virgin Galactic, Vertiv, Clarivate, Hostess, AdaptHealth, and a couple of dozen others restated fiscal 2020, and some also 2019, to reclassify their warrants. They have operating SIC codes and full financials, and their 4.02s came after their fiscal-2020 10-Ks, so roughly 30 of them stay in your evaluation sample as fiscal-2020 positives and as training positives for the 2021 window. They are genuine 4.02s, but mechanical ones, and their 10-Ks share a vocabulary ("business combination," "warrants," "PIPE"). Report how many of your restated rows have a `reveal_date` between 2021-04-12 and 2021-06-30.

**Before you run anything**, open `results.md` in your repo and write a short section headed *Predictions*. For each difference above, say whether you expect it to make topic look *more* or *less* useful than in the paper, and why. For example: Item 4.02s include plenty of honest errors, like a lease-accounting mistake or a misclassified cash flow. Would a topic model detect those as well as it detects fraud? Would the financial model? Commit that section, on this branch, **before** you see results. It's the equivalent of pre-registration, and it's the difference between explaining a result and explaining it away.

## 4. Financial predictors and the evaluation sample

> **Brief.** Write `src/features_financial.py` that reads `data/final/panel.parquet` and writes `data/features/financial.parquet` keyed by `(cik, fyear)`. Drop rows with `at <= 0` or `at_lag <= 0` first (a few shells report zero assets, and every ratio below divides by assets), and report how many. With `avg_at = (at + at_lag) / 2`:
>
> - `log_at = log(at)`
> - `rsst_acc = (noac − noac_lag) / avg_at`, where `noac = at − che − lt − pstk` (and `noac_lag` from the lagged values). This is Richardson et al.'s (2005) accruals measure, ΔWC + ΔNCO + ΔFIN, written in the form that needs the fewest inputs. With WC = (current assets − cash − short-term investments) − (current liabilities − current debt), NCO = (total assets − current assets − long-term investments) − (total liabilities − current liabilities − long-term debt), and FIN = (short-term investments + long-term investments) − (long-term debt + current debt + preferred stock), every term except assets, cash, liabilities, and preferred stock cancels. Prove the identity in a unit test with made-up numbers.
> - `ch_rec = (rect − rect_lag) / avg_at`
> - `ch_inv = (invt − invt_lag) / avg_at`
> - `soft_assets = (at − ppent − che) / at`
> - `ch_cash_sales = cs / cs_lag − 1`, where `cs = sale − (rect − rect_lag)` and `cs_lag` is the prior-`fyear` row's `cs` for the same company (sort by `cik` and `fyear` first, then use `groupby("cik").shift()`, only where that row's `fyear` is exactly `fyear − 1`). Where it's missing (fiscal 2011, or a gap in filings), fall back to `cs_lag = sale_lag`, and report how often the fallback is used. Leave `ch_cash_sales` missing when `cs_lag <= 0`.
> - `ch_roa = (pi − pi_lag) / avg_at` (the paper's Appendix B definition: the change in pretax income, scaled by average assets)
> - `issue = 1 if sstk > 0 or dltis > 0 else 0`
> - `tot_financing = fincf / avg_at`
>
> Don't winsorize here; §7 does it inside each window. Report, for each variable, the share missing and which input caused it.

What you *can't* build from EDGAR alone: book-to-market and the lagged market-adjusted return need stock prices. Auditor type (Big N, mid-size) is buildable but not free. XBRL cover pages carry the auditor's name (`dei:AuditorName`) only for fiscal periods ending on or after December 15, 2021, so at most your last year has it. For earlier years the name is on the audit report's signature line ("/s/ Ernst & Young LLP") in the 10-K text from Module 8 §8b. That makes a good optional extension. The Dechow model's market variables are some of its stronger predictors, so your financial model is weaker than theirs *by construction*, which makes it easier for topic to look good against it. Write that in your *Predictions* section if you haven't.

Every model in §7 must be scored on exactly the same CIK-years, or the comparison is meaningless: a model that drops hard cases because a variable is missing will look better for the wrong reason. So fix the evaluation sample once, in a file, before any model sees it.

> **Brief.** Write `src/eval_sample.py`. From `data/final/panel.parquet` and `data/features/financial.parquet`, keep CIK-years that:
>
> - have text (`text_path` not null), and so have a sampling weight `w`;
> - have every §4 variable non-missing;
> - are not `revealed_before_10k` (`~panel["revealed_before_10k"]`; Module 8 §9: those 10-Ks were written after the restatement was public);
> - are not financial firms (`pd.to_numeric(sic, errors="coerce")` outside 6000–6799, matching the paper's prediction sample; `sic` is a string and about 1% of rows have none, so keep those and report how many).
>
> Write `data/features/eval_sample.parquet` with columns `(cik, fyear, accession, filing_date, sic, blank_check, w, random20, restated, reveal_date, irregularity, irregularity_reveal_date, text_path)`. Print rows dropped by cause and by `fyear`, and the number of restated rows that survive in each year.

Look at the survivors per year before you go on. If a year has only a handful of restated rows left, its AUC will be noise, and you should say so in the write-up rather than let it vanish into the pooled number.

## 5. Style predictors

> **Brief.** Write `src/features_style.py` that reads the text file of each CIK-year in `data/features/eval_sample.parquet` and computes, per document:
>
> - `log_words` (log word count) and `parsed_chars` (character count)
> - `sent_len_mean`, `sent_len_sd` (words per sentence), `word_len_sd` (characters per word)
> - `pct_short_sent`, `pct_long_sent` (share of sentences ≤ 30 and ≥ 60 words)
> - `repetition`: mean number of times each distinct sentence appears in the document
> - `type_token` = unique words / total words, computed on the **first 10,000 words** so it's comparable across documents of different lengths (the paper uses the whole document; say so in your write-up)
> - `fog = 0.4 × (words/sentences + 100 × complex_words/words)`, with complex words having 3+ syllables (count syllables with `pyphen`, and say so)
> - `coleman_liau = 5.88 × (chars/words) − 29.6 × (sentences/words) − 15.8`, where `chars` counts every character that isn't a space or punctuation (the paper's definition, so digits count)
> - `n_allcaps` (words of 2+ letters, all capitals), `n_exclaim`, `n_question`
>
> Sentence splitting: split on `.`, `?`, `!` followed by whitespace and a capital letter, protecting common abbreviations (`Inc.`, `Corp.`, `No.`, `U.S.`, `e.g.`, `i.e.`). A word, for every measure above: a whitespace-separated token with leading and trailing punctuation stripped that contains at least one letter or digit (so `2,345` is a word and a lone `$` is not). Keep both rules simple and deterministic, and document them. Write `data/features/style.parquet`. Process files one at a time; don't hold the corpus in memory.

Implement the Coleman-Liau and Fog formulas exactly as written above, which follow the paper's Appendix B, and add a unit test on a sentence whose values you work out by hand. Readability formulas are the kind of code where an off-by-one in the syllable counter silently shifts every value, and a hand-checked test is the only defense.

*Optional extension:* the paper's `%ActiveVoice` and `%PassiveVoice` need a dependency parser. `spaCy`'s `en_core_web_sm` model marks passive subjects (`nsubjpass`). Running it over hundreds of millions of words takes hours on a laptop, so if you add it, run it on the first 20,000 words of each document and say so.

## 6. Topics, in rolling windows

This is the paper's contribution and the most involved part of the build. The plan:

| Window (train fiscal years) | Test fiscal year | Label and filing cutoff for training |
|---|---|---|
| 2011–2015 | 2016 | 2016-06-30 |
| 2012–2016 | 2017 | 2017-06-30 |
| 2013–2017 | 2018 | 2018-06-30 |
| 2014–2018 | 2019 | 2019-06-30 |
| 2015–2019 | 2020 | 2020-06-30 |
| 2016–2020 | 2021 | 2021-06-30 |

The cutoff is set by *when the test-year filings could first be scored*. Under the Compustat convention from Module 8, fiscal 2016 runs from year-ends in June 2016 through May 2017, so the earliest fiscal-2016 10-Ks arrive in the summer of 2016. A model that scores them can only have learned from what was public before then. That puts the cutoff at June 30 of the test year, and it applies to two things:

- **Labels.** A training row counts as restated only if its `reveal_date` is on or before the cutoff (for the irregularity outcome, its `irregularity_reveal_date`).
- **Filings.** Because your windows are fiscal years, not filing years, a few training rows were *filed* after the cutoff: late filers, mostly, and January–May year-ends filed in the summer. Drop from each window's LDA fit and regressions any row whose `filing_date` is after the cutoff, and report how many.

Test-year labels are the eventual truth, meaning every restatement known as of your Module 8 download. The cutoff is conservative for the December year-end majority, whose 10-Ks arrive months after it, and conservative is the right direction to err.

### 6a. Tokenize once, choose vocabulary per window

Re-tokenizing tens of thousands of 10-Ks for each of six windows would take all day. Instead, tokenize every document **once** into a sparse document-term matrix, and let each window pick its own columns.

> **Brief.** Write `src/topics_dtm.py`. For every CIK-year in `data/features/eval_sample.parquet`, read its text, and build a `scipy.sparse` CSR document-term count matrix with `sklearn.feature_extraction.text.CountVectorizer(token_pattern=r"(?u)\b[a-z]{3,}\b", lowercase=True, min_df=1, dtype=np.int32)` (alphabetic tokens of 3+ letters: numbers and numeric dates drop out, but month names and tickers stay, and §6b's stopword rules remove the common ones). Stream the documents from disk with a generator. Save the matrix (`scipy.sparse.save_npz`), the vocabulary, and the row order `(cik, fyear)` to `data/features/`. Print the matrix shape and number of non-zeros.

(If you fit the topic model on a broader set than the evaluation sample, say by keeping financial firms the way the paper does, build the matrix over that broader set and select rows later. Either way, write down which rows the topic model saw.)

> **Is a global vocabulary look-ahead?** Only if you let future documents *choose* the columns. The union vocabulary just assigns every word an index. Which columns a window actually *uses* is decided in §6b from that window's documents alone, and a word that first appears in 2019 has zero counts in every earlier window, so it can never be selected there. This is the general pattern for time-series text work: compute once, but make every *decision* inside the window.

### 6b. Window-specific stopwords

The paper removes three kinds of words, all computed within each window: (1) the most frequent words, removed in order of frequency until 60% of all word occurrences in the window are gone; (2) words occurring fewer than 1,100 times in the window; (3) words appearing in fewer than 100 filings. Those thresholds were tuned, by simulation, on their corpus of every 10-K in each window. Your windows are smaller, because you sampled filings in Module 8 §8b, so the same absolute cutoffs would prune far more of your vocabulary than of theirs. Restate rules (2) and (3) as shares of your window (for example, a word must appear in at least 1% of the window's filings), choose values, report them, and check in §8 whether the answer is sensitive to them.

Rule (1) is aggressive, and it's worth seeing why before you implement it. In a legal document, a small set of words ("company," "financial," "may," "including") make up a huge share of all tokens and say nothing about content. Removing the head of the frequency distribution until 60% of occurrences are gone is a blunt way of forcing the model to spend its topics on the words that actually distinguish documents.

### 6c. Fit, transform, repeat

> **Brief.** Write `src/topics_rolling.py`. For each window in the table above:
>
> 1. Take the rows of the DTM whose `fyear` is in the window, whose `filing_date` is on or before the cutoff, and whose `random20` is true: a 20% sample drawn without looking at labels. Compute the window's stopword set (§6b) from those rows only, and keep the remaining columns.
> 2. Fit `sklearn.decomposition.LatentDirichletAllocation(n_components=31, learning_method="online", batch_size=100, learning_offset=10, learning_decay=0.7, doc_topic_prior=0.05, topic_word_prior=0.05, max_iter=10, random_state=575, n_jobs=1)` on those rows, **shuffled** with a seeded RNG. Those settings follow the paper, which uses the online variational algorithm of Hoffman, Bach, and Blei (2010) in batches of 100 filings drawn in random order, with symmetric priors of 1/20 and a learning decay of 0.7, with one exception. The paper's learning offset of 1024 damps every early update. Across tens of thousands of 10-Ks per window that's harmless, but on your few thousand it leaves most of the 31 topics stuck near their random start, so use 10. (`random_state` makes the fit reproducible only at a fixed `n_jobs`, because sklearn gives each parallel worker its own share of the work; `n_jobs=1` keeps your topics identical to a classmate's.)
> 3. Transform the **test year's** rows and **all** of the window's rows (same columns). `transform` returns rows that already sum to 1; these are the topic proportions.
> 4. Save, per window: the fitted model (`joblib`), the kept vocabulary, the top 15 words per topic as a CSV, and the proportions for window and test rows to `data/features/topics_{test_year}.parquet`.
>
> Log the time per window. Accept `--windows` to run a subset, for testing.

Why `random20`: the text sample keeps every CIK-year that was *eventually* restated, including restatements revealed years after the cutoff. An LDA fit on all of it would let future labels decide which documents shape the topics, which is the same quiet look-ahead as fitting one topic model on every year at once. The logistic regressions can correct for the sampling with weights; `LatentDirichletAllocation` takes no weights, so it gets a sample that never looked at labels instead.

Why shuffle: sklearn's online LDA visits the batches in the order you give them, and each update partly forgets the earlier ones, so documents fed in year order pull the topics toward the window's last year. (The paper also draws its batches in random order, framing the risk in its own tool as overweighting the early years.) Either way, the order of the documents shouldn't shape the topics.

Run one window first (`--windows 2016`) and check that the fit converged: print the window's perplexity and the number of topics whose mean share across documents exceeds 1%. If only a handful of topics carry the documents, or several topics share the same top words, the fit hasn't converged. That's a settings problem, not a cleaning problem. Then open the top-words CSV and **read the topics**. You should see recognizable themes: banking and loans, oil and gas exploration, clinical trials and FDA approval, real estate and leases, retail stores, and a few that are about the 10-K's own boilerplate (risk-factor language, internal-control certifications). If a topic's top words are run-together XBRL names ending in `member` (`maximummember`, `minimummember`, `seniornotesmember`), or `srt` and `gaap`, the inline-XBRL header survived Module 8's cleaning. Go back and fix it there, not here.

## 7. Estimate and evaluate

> **Brief.** Write `src/evaluate.py`. Restrict everything, training and testing, to rows in `data/features/eval_sample.parquet`, and `assert` that every model scores the identical set of `(cik, fyear)` in each test year. For each test year, and for each model specification below:
>
> 1. Take the window's training rows (filed on or before the cutoff) and the test year's rows.
> 2. Winsorize every continuous financial and style variable at the 1st and 99th percentiles, with caps computed on the training rows only, as percentiles weighted by `w` (`np.percentile(x, [1, 99], weights=w, method="inverted_cdf")`), and applied to both.
> 3. Standardize every feature with the training rows' `w`-weighted mean and SD.
> 4. Fit `sklearn.linear_model.LogisticRegression(C=1.0, max_iter=5000)` (L2 is the default penalty; don't pass `penalty=`, which is deprecated) with `sample_weight=w`, and score the test rows.
>
> | Model | Features |
> |---|---|
> | FIN | financial (§4) |
> | STYLE | style (§5) |
> | TOPIC | 31 topic proportions (§6) |
> | FIN+STYLE | |
> | FIN+TOPIC | |
> | STYLE+TOPIC | |
> | FIN+STYLE+TOPIC | |
>
> Training label: `restated == 1` **and** `reveal_date` on or before the window's cutoff. Test label: `restated == 1`. Run everything a second time with the irregularity outcome: `irregularity` with `irregularity_reveal_date` on or before the cutoff in training, and `irregularity` in testing.
>
> Pool the out-of-sample scores across all six test years and compute the pooled AUC per model, weighted: `sklearn.metrics.roc_auc_score(y, score, sample_weight=w)`. Report per-year weighted AUCs too. For each pair (FIN vs FIN+TOPIC, STYLE vs STYLE+TOPIC, FIN vs TOPIC, FIN+STYLE vs FIN+STYLE+TOPIC), compute the AUC difference with a **cluster bootstrap by CIK**: resample CIKs with replacement 1,000 times (seed 575), recompute both weighted pooled AUCs on each resample, and report the 2.5th and 97.5th percentiles of the difference.
>
> Skip any bootstrap resample, or any test year, that lacks both classes; report the number of resamples used, and report such a year's AUC as undefined rather than letting a NaN into an average. If a training window has no positives for an outcome, skip that window for that outcome and say so. Write `results/m9/auc.csv` and `results/m9/auc_diff.csv`, with a column saying which outcome each row is.

Four choices in that brief are worth understanding rather than just accepting.

- **Why weights.** Module 8 kept every restated CIK-year and one in five of the others. For the exact outcome you sampled on, the unweighted AUC would be fine, because AUC doesn't care how many of each class there are. But you also use a narrower outcome (irregularities), whose negatives include restated rows sampled at 100%, and training labels that count some not-yet-revealed restatements as negatives. Weighting each row by the inverse of its sampling probability makes the sample stand in for the population in every one of those cases. It's one argument, and leaving it out biases the comparison in ways that differ across models.
- **Topic proportions sum to one.** Put all 31 into a logistic regression with an intercept and they're perfectly collinear, because any one is 1 minus the others. An unpenalized fit either fails or returns meaningless coefficients. The default L2 penalty resolves this: it picks the smallest-norm solution among the equivalent ones. Dropping one topic would work too. What doesn't work is ignoring the problem.
- **Why bootstrap by CIK, not by row.** The same company appears in several test years, and its years are not independent draws. Resampling rows pretends you have more independent evidence than you do and gives confidence intervals that are too narrow. Resampling companies keeps each company's years together.
- **Why AUC and not accuracy.** Restatements are a few percent of CIK-years. A model that predicts "no restatement" for everyone is 95%+ accurate and useless. AUC asks the question that matters for screening: if you rank firms by the model's score, do the restaters rank higher?

## 8. Read the results

Build the table the paper would have built:

| Model | Pooled AUC (all 4.02s) | Pooled AUC (irregularities) |
|---|---|---|
| FIN | | |
| STYLE | | |
| TOPIC | | |
| FIN+STYLE | | |
| FIN+TOPIC | | |
| STYLE+TOPIC | | |
| FIN+STYLE+TOPIC | | |

And the differences with their bootstrap intervals. Then answer, in `results.md`:

1. **Does topic add anything?** Does adding TOPIC raise the AUC of FIN, and of STYLE, with a bootstrap interval that excludes zero? Say "yes," "partially," or "no," with numbers.
2. **Which of your *Predictions* held?** Go through them one by one. Where you were wrong, what does that tell you?
3. **All restatements versus irregularities.** Is topic more useful for one outcome than the other? The paper argues topic works because what a company chooses to discuss reflects its business and its incentives. Is that argument stronger for fraud than for honest error? And why does the financial-firm exclusion matter more in 2020–2021 than in the paper's period: what would "this company is a shell" have done to TOPIC's AUC? What did the de-SPAC warrant restatements that stayed in your sample do to fiscal 2020?
4. **What are the predictive topics?** For the FIN+TOPIC model in two or three windows, list the topics with the largest positive and negative coefficients, their top words, and a human-readable label for each. Compare them with what the paper reports for irregularity restatements: restating firms discussed advertising, securitized or guaranteed securities, strategic alliances, and growth in franchised operations more, and mergers, foreign-country risk, hedging, legal proceedings, credit arrangements, and stock-option plans less; discussions of income performance mattered in most years, but without a consistent sign. A topic that turns out to mean "small shell company" is a finding too, but it's a different finding from "the disclosure is suspicious."
5. **One robustness check of your choosing.** Some examples: 20 or 50 topics instead of 31; only the MD&A instead of the full 10-K; the paper's topic projection instead of `transform` (`X @ (lda.components_ / lda.components_.sum(axis=1, keepdims=True)).T`, each row rescaled to sum to 1); the paper's Fisher test (per test year, a univariate logistic regression of the outcome on the out-of-sample score, with the six p-values combined as −2 Σ ln p ~ χ²(12), using test labels for inference only, never to build the score; this one needs `uv add statsmodels` for p-values, and don't pass `w` as `freq_weights`, which counts each control five times and shrinks the p-values); keeping financial firms (SIC 6000–6799); dropping positives whose `reveal_date` falls between 2021-04-12 and 2021-06-30 (the warrant wave); fitting LDA on every window row instead of the `random20` subsample; different stopword thresholds. Pick one, run it, and report whether the answer changes.

What matters on this module is not whether your AUCs match theirs. A careful build that finds a weaker effect on your data and explains why is a better result than a sloppy one that happens to match.

## 9. Open the PR

```bash
git add src tests results/m9 results.md
git commit -m "M9: rolling-window LDA topic predictors of Item 4.02 restatements"
git push -u origin feature/m9-replication
gh pr create --title "M9: Topic vs. financial predictors of Item 4.02 restatements" --body "$(cat <<'EOF'
Topic, financial, and style predictors of misreporting on EDGAR 2011–2021, following the design in Brown, Crowley & Elliott (2020, JAR).

- Financial, style, and rolling-window LDA topic features
- Reveal-date-aware training labels and filing-date-gated windows; weighted out-of-sample pooled AUC, 2016–2021
- Cluster-bootstrap (by CIK) intervals for AUC differences

See results.md for predictions (committed before results), the AUC tables, and the discussion.
EOF
)"
```

Review your own diff on GitHub before you merge, the Module 3 way. The three places to look hardest: anything that computes a statistic across years (does any window see a later year's data?), anything that filters rows (are all seven models scored on the same CIK-years?), and every call to `roc_auc_score` and `fit` (is `sample_weight` there?).

## 10. Where this goes next *(optional)*

LDA is a 2003 method, and the paper's choice of it reflects when the work was done. Its contribution was the *idea* that disclosure content predicts misreporting, together with a design that tests it honestly. The design is the lasting part. Swap the topic step for any modern text representation and the rest of this module is unchanged. Examples are sentence-embedding averages (the optional [Module 20](../../part-4-agentic-ai-methods-and-pipelines/module-20-bert-for-financial-text/README.md) builds these) or an LLM that labels each filing's themes (the optional [Module 21](../../part-4-agentic-ai-methods-and-pipelines/module-21-llm-zero-shot-extraction-and-classification/README.md) covers that). If you want an extension project, that's the natural one: does a 2026 text representation beat 2003 LDA on the same windows, labels, and metric?
