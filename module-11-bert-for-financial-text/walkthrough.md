# Module 11 — Walkthrough: BERT Embeddings as Features

## 1. What BERT does

The point of BERT is to turn a piece of text into a vector. Earlier systems did that too. TF-IDF and word2vec both produce vectors. The difference is that BERT's vectors depend on context, while the earlier ones don't. Every instance of "liquidity" in a corpus gets the same TF-IDF weight regardless of the sentence it appears in. BERT was the first widely-used model where context flowed in: the vector for *liquidity* in *liquidity covenant* came out different from the vector for *liquidity* in *liquidity event*, because BERT's attention layers conditioned each token on the surrounding words.

That conditioning happens inside the self-attention operation:

$$\text{Attention}(Q, K, V) = \mathrm{softmax}\left(\frac{Q K^\top}{\sqrt{d_k}}\right) V.$$

For each token in the input, the model looks at every other token and decides how much each one matters for the current token's representation. A dozen of these layers stacked produces the rich context-conditional vectors people call "BERT embeddings."

How did the model learn this? Pretraining on a few billion words of BookCorpus and English Wikipedia, by predicting masked words. Show it `the company recognized [MASK] of $1.2M in Q3` and ask it to fill in *revenue*. Or *impairment*. Or *expense*, depending on what the surrounding text implies. After enough of that, every layer's outputs are useful as general-purpose text features.

There are two things you can pull out of the model. One is a vector per token — an $\mathbb{R}^{768}$ vector for each input token, in `bert-base`. That's what you'd use for token-level tasks like named-entity recognition. The other is a vector per document. That's what we want here. The standard way to get the document vector is to mean-pool the token vectors, skipping padding positions. BERT was trained to put a sentence-level summary in the special `[CLS]` token and you can use that, but on long documents mean-pool is more robust.

*Optional further reading: [Hugging Face Transformers Course — encoder models](https://huggingface.co/learn/nlp-course/) — a short conceptual tour of how encoder transformers like BERT produce their vectors; a good primer before touching code.*

The wall is 512 tokens. That's about 400 words of English. A 10-K MD&A runs several thousand words, so you have to deal with it.

Three ways out. The cheap one is to truncate to the first 512 tokens, which works if the document front-loads its content (filings usually do). The honest one is to chunk into 512-token pieces and mean-pool the chunk vectors. The expensive one is to swap in a long-context model like Longformer, BigBird, or `nomic-embed-text-v1.5` with its 8K window. Part 2 uses chunking. If your text is short to begin with (analyst headlines, press release titles), truncation is fine.

## 2. Branch

```bash
git checkout main && git pull && git checkout -b feature/m11-bert
```

## 3. Pick the model

| Model | Parameters | When to use |
|---|---|---|
| `bert-base-uncased` | 110M | General-purpose default. Fast on CPU for small batches. |
| `yiyanghkust/finbert-tone` | 110M | Domain-tuned on finance text. Worth comparing if your task is sentiment-flavored. |
| `nomic-embed-text-v1.5` | 137M | Modern long-context (8K) embedder. Skips the chunking step entirely. |

Default to `bert-base-uncased`. If you want a comparison, run both. The marginal cost on a few thousand documents is small.

*Optional further reading: [FinBERT model card (`yiyanghkust/finbert-tone`)](https://huggingface.co/yiyanghkust/finbert-tone) — Huang, Wang, and Yang (2023), a finance-domain-tuned BERT; a useful baseline to compare against general-purpose `bert-base`.*

## 4. Brief the agent

A reality check before you brief the agent.

Most of what a firm discloses isn't numbers. The MD&A is dozens of pages of prose, and there's plenty more around it: risk factors, footnotes, 8-K narratives. Accounting researchers have been mining that text for a long time, but mostly with dictionary methods. The standard reference is Loughran and McDonald's (2011) finance-tuned negative-word list. You count occurrences of those words in a filing, normalize by length, and regress returns or fundamentals on the result.

It works. It also reads *liquidity covenant* and *liquidity event* as the same thing. They both contain "liquidity" and they mean opposite things, which is the kind of failure mode that's invisible until you go looking for it. BERT fixed it. Attention layers condition each token's representation on the words around it, so those two *liquidity*s end up with different vectors. That observation is the whole reason BERT-style embeddings replaced dictionary methods in the literature.

The applied side has caught up. Hedge funds run this sort of analysis on earnings calls, looking for tone shifts that precede earnings misses. Audit teams use it to flag unusual MD&A language as a focus area for substantive testing. It's not research-only anymore.

What this demo asks is simple: do the embeddings buy you anything?

You'll encode every MD&A in the Compustat sample as one 768-dim vector, append those columns to Module 10's GBM feature set, retrain, and compare RMSE with and without the text features. The question is whether the *prose* of MD&A predicts anything about next-year ROA on top of the fundamentals already in the model. If a small improvement appears, the narrative carries information the numbers don't. If no improvement appears, the narrative is mostly redundant with the balance sheet. Write up whatever you actually find.

The brief:

> **Brief.** Add `src/text/encode.py` with one function:
>
> ```python
> def encode_documents(
>     texts: list[str],
>     model_name: str = "bert-base-uncased",
>     chunk_tokens: int = 510,
>     batch_size: int = 16,
> ) -> np.ndarray:
> ```
>
> Returns a `(n_docs, hidden_dim)` array. For each document: tokenize, split into chunks of up to `chunk_tokens` tokens, encode each chunk through the model, mean-pool over non-padding tokens to get a chunk vector, then average the chunk vectors to get the document vector. Use `transformers.AutoModel` and `transformers.AutoTokenizer`. Run on GPU if available (`torch.cuda.is_available()`), otherwise CPU. **Cache** the result to disk: if the input texts and `model_name` match a previous run, reload from cache instead of re-encoding.
>
> Then `notebooks/m11-bert.ipynb`:
>
> 1. Loads `data/raw/mdna.parquet` (the textual companion from Module 8, with columns `gvkey`, `fyear`, `cik`, `fdate`, `mdna_text`, `char_count`) and `data/raw/fundamentals.parquet`.
> 2. Joins on `(gvkey, fyear)`. Dropping rows with missing `mdna_text` is fine; note how many you drop.
> 3. Calls `encode_documents(df["mdna_text"].tolist())` on the `mdna_text` column. Saves the resulting array to `data/interim/mdna_embeddings.npy`, aligned row-by-row with the joined dataframe (save the index alongside so a later module can re-join).
> 4. Re-runs the **Module 10 GBM** with the 768 embedding columns appended to the original feature set. Reports test RMSE / R² with and without text features.
> 5. Stores the comparison table in `results/m11/`.
>
> Add `tests/test_encode.py` with one test on three short strings that checks the output shape is `(3, 768)`.
>
> `uv add transformers torch sentence-transformers`. Do **not** modify earlier modules' code.
>
> *(On `transformers` 5.x, loading `bert-base-uncased` through `AutoModel` prints a one-time "unexpected keys" report for the dropped pretraining heads — that's expected, and the embeddings are unaffected.)*

Three non-obvious moves in this brief. Caching, because encoding is the expensive step. Without a cache, every notebook re-run pays the full forty-minute cost again. Same target, same train/test split, same downstream model — the only thing changing is whether embedding columns are added. And mean-pool over non-padding tokens, because padding positions carry no signal and pooling over them just dilutes the embedding.

## 5. Where to run this — and check cost first

The WRDS Cloud grid you set up in Module 8 (`wrds-cloud-sshkey.wharton.upenn.edu`, reached through the multiplexed `wrds` alias from §4 — `wrds-cloud.wharton.upenn.edu` is the password-only endpoint you used once and shouldn't need again) runs SAS, R, Python, and Stata. It has no GPU nodes in the public documentation as of 2026 — `ssh wrds 'qhost'` lists every compute node the scheduler knows about, and they are CPU machines — which means `torch.cuda.is_available()` returns `False`. That matters: a single consumer GPU runs encoding 50–100× faster than a WRDS node. This is exactly what GPUs were built for.

So you have three places you might actually run this, in order of speed:

| Where | Fits | Notes |
|---|---|---|
| **WRDS grid (CPU)** | <2k short docs | `qsub` batch job (same mechanics as Module 8 §8b), raising the memory reservation with `-l m_mem_free=16G`; plan multiple hours |
| **Local Mac (M-series, MPS)** | <5k docs | PyTorch's `mps` backend uses Apple GPU; substantially faster than pure CPU |
| **Colab / Kaggle / campus HPC** | 5k+ docs | Free T4 (Colab) or A100 (campus); minutes, not hours |

The standard workflow is to pull the MD&A text from WRDS down to your local machine or up to Colab, encode there, and save the resulting `.npy` of embeddings. WRDS lets you derive features from licensed data and bring the derived features back. Check your school's data agreement before pushing raw text to a public repo, but a 768-dim numeric vector is a derived feature, not redistributable text. If you have access to a GPU cluster (UIUC's Campus Cluster, Penn's HPC), use it.

Cost sanity check for the assignment-sized job. Five thousand MD&A documents at roughly 3,000 tokens each is six chunks per doc, around 30,000 chunks total. On WRDS CPU that's two to three chunks per second, so three to four hours. On an M-series Mac with MPS, about ten chunks per second, fifty minutes. On a Colab T4 with batch 16, two to three minutes.

If your math says days, something's wrong with the spec. Probably the agent forgot to chunk and is feeding the model six-thousand-token inputs.

## 6. Run, evaluate, compare

```bash
uv run jupyter lab notebooks/m11-bert.ipynb
uv run pytest tests/test_encode.py
```

The headline cell is the comparison:

| Features | Test RMSE | Test R² |
|---|---|---|
| Fundamentals only (M10) | 0.071 | 0.36 |
| Fundamentals + MD&A embeddings | ? | ? |

If text helps a little (a one-to-three-percent RMSE drop), that's the typical real-world result. Report it that way. If text helps a lot, check for leakage; MD&A sometimes references next year's outcome directly, and a model with access to that will look unreasonably good. If text doesn't help at all, that's a result too. The qualitative narrative may not carry marginal predictive signal beyond the fundamentals on this target.

## 7. Don't try to interpret individual dimensions

The 768 embedding columns are a learned, dense representation. Dimension 47 has no meaning you can read off. SHAP on embedding features will tell you that "the text component" matters, not what about the text matters. If you need interpretable text features — keyword counts, topic memberships, sentiment scores — Module 12 will give you them. A raw BERT vector won't.

## 8. Other things you can do with the same encoder (extensions)

The walkthrough above does the simplest thing: embeddings as features for a downstream regression. The same `encode_documents` cache is the foundation for every text task in Part 2, and a handful of variations sit on top of it. Each is roughly an evening of additional work.

**Year-over-year MD&A change.** Encode each filing; the cosine distance between filing $t$ and $t-1$ is one scalar per firm-year. Cohen, Malloy, and Nguyen ("Lazy Prices," 2020, *Journal of Finance*) show that firms whose disclosures change substantially year-over-year subsequently underperform firms whose disclosures barely change. Add it as one feature.

```python
from sklearn.metrics.pairwise import cosine_similarity
change = 1.0 - cosine_similarity(emb_t.reshape(1, -1), emb_t_minus_1.reshape(1, -1))[0, 0]
```

**Sentiment / tone classification (zero-shot).** `yiyanghkust/finbert-tone` is already fine-tuned on analyst reports for positive, negative, and neutral. The `pipeline` API gives you one-line inference.

```python
from transformers import BertTokenizer, BertForSequenceClassification, pipeline
# finbert-tone predates the `model_type` key the bare pipeline() helper now requires
# on transformers 5.x, so load the tokenizer and model explicitly:
tok = BertTokenizer.from_pretrained("yiyanghkust/finbert-tone")
mdl = BertForSequenceClassification.from_pretrained("yiyanghkust/finbert-tone", num_labels=3)
tone = pipeline("text-classification", model=mdl, tokenizer=tok)
tone("Operating margins compressed materially in Q3.")
# [{'label': 'Negative', 'score': 1.0}]
```

Aggregate per filing (share of negative sentences, say) and add as a feature.

**Topic clustering.** Split MD&A into paragraphs, encode each, run `KMeans` or `HDBSCAN` on the paragraph embeddings, then label each cluster by reading its five nearest paragraphs. Useful for descriptive work — "what themes dominate pharma 10-Ks in 2024."

**Retrieval / similarity search.** Given a query like *liquidity risk in covenant breaches*, embed it and rank all paragraphs in your corpus by cosine similarity. This is the index Module 13 (RAG) builds; Module 11 is the encoding half of that pipeline.

**Fine-tuning a custom classifier.** With at least a few hundred labeled examples (paragraphs tagged "discusses cybersecurity" vs. not, say), add a classification head on top of `bert-base` and fine-tune it. This is the one extension where you genuinely need GPU access. A three-epoch fine-tune of `bert-base` on 1k examples is about five minutes on a T4 and several hours on CPU. Use `transformers.Trainer` with `AutoModelForSequenceClassification`. Don't attempt it on WRDS CPU.

The first three reuse the encoding cache and only change the downstream computation, so they're cheap once §4 is done. Retrieval needs a small `faiss` or `sklearn.neighbors` index. Fine-tuning is its own assignment.

## 9. PR and merge

```bash
git push -u origin feature/m11-bert
gh pr create --title "M11: BERT embeddings as downstream features" --body "$(cat <<'EOF'
## Summary
- Adds `src/text/encode.py` with cached BERT encoding.
- Notebook compares GBM with vs. without `bert-base-uncased` embeddings on the M9/M10 target.
- Embeddings cached at `data/interim/mdna_embeddings.npy`; not committed.

## Headline result
[one sentence: did embeddings improve over fundamentals-only, by how much]

## Caveats
- 512-token truncation strategy = mean-pool of chunked encodings.
- Embedding columns are not individually interpretable.
EOF
)"
```

On self-review of this PR specifically, the embedding cache file should be `.gitignore`d (it's large and re-derivable), and the encoding code shouldn't run inside the notebook on every execution — the cache should hit on a re-run. The downstream comparison has to use the same train/test rows for both models. If the join with text dropped 1,500 rows, the M10 baseline needs to be recomputed on the post-join sample to be a fair comparison. That last one is the trap most people fall into. *"GBM-with-text gets RMSE 0.060 versus GBM-without-text at 0.071"* doesn't mean anything if the two were computed on different row sets.

Squash-merge, delete branch.

## You're done if…

- [ ] `feature/m11-bert` is merged into `main`.
- [ ] `data/interim/mdna_embeddings.npy` exists locally and is gitignored.
- [ ] The with-vs-without-text comparison uses the same sample for both rows.
- [ ] You can state the marginal RMSE improvement (or non-improvement) as a single number.
