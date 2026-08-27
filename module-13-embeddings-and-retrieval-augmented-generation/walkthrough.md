# Module 13 — Walkthrough: A Small RAG over the WRDS Corpus

## 1. What RAG does

A pretrained LLM has read a slice of the public internet up to its training cutoff. It does not know what's in a specific 10-K. It does not know what a particular firm said on a 2024 earnings call. It does not know anything proprietary you've collected. Retrieval-augmented generation closes that gap by putting the relevant documents into the prompt at query time.

The minimal pipeline has five pieces.

A chunker splits long documents into chunks small enough to retrieve and re-read individually — around 500 tokens, with a little overlap. An embedder turns each chunk into a dense vector, the same operation as Module 11 but at chunk granularity instead of document granularity. A vector store indexes those vectors so you can do fast nearest-neighbor lookup; FAISS, Chroma, or LanceDB are the common local options, Pinecone or Weaviate in the cloud. A retriever embeds an incoming user query and returns the $\text{top-}k$ most similar chunks, where similarity is usually cosine:

$$\cos(\theta) = \frac{u \cdot v}{\|u\| \cdot \|v\|}.$$

And a generator — a frontier LLM — receives the query plus the retrieved chunks and produces an answer, ideally with citations to which chunks supported which claims.

*Optional further reading: [LlamaIndex documentation](https://docs.llamaindex.ai/) — a production framework that wires these same five pieces together; the architecture page is the entry point (broad reference for the whole pipeline).*

Two failure modes are specific to RAG, and you'll see both. Retrieval can miss: the relevant chunk is in the index but doesn't make the $\text{top-}k$, and the generator either honestly says "I don't know" (best case) or hallucinates from whatever irrelevant chunks it got (worst case). Generation can be ungrounded: retrieval was fine, but the generator wrote something the retrieved chunks don't support. That happens when the LLM "knows" the answer from training and isn't actually checking the context you gave it.

These two have to be evaluated separately. Retrieval quality is measured with $\text{recall@}k$ on a small hand-built question set. Generation quality is measured by reading answers and marking which claims are actually supported.

## 2. Branch

```bash
git checkout main && git pull && git checkout -b feature/m13-rag
```

## 3. Decide your evaluation questions first

Same lesson as Module 12: build the eval before the system, or you have no way to grade what you build.

Write 15 to 20 specific questions your RAG should be able to answer from the MD&A corpus. Concrete, factual, ideally answerable from a single document:

- *"Did Microsoft's FY2023 10-K mention generative AI as a risk factor?"*
- *"Which firms in the consumer-staples sector flagged supply-chain disruption in 2022?"*
- *"What did Tesla say about regulatory credits in 2024?"*

For each question, find by hand the chunk(s) that should be retrieved. Save to `data/eval/rag_questions.csv` with columns `(question_id, question_text, gold_chunk_ids)`. That's your retrieval gold standard.

## 4. Brief the agent

Before the brief, a paragraph on why RAG is showing up in accounting and finance courses now.

A single S&P 500 firm's 10-K runs over a hundred pages. Five hundred firms across fifteen years is millions of pages. The traditional way to find a specific fact in that corpus — *"which firms in 2022 flagged supply-chain disruption as a material risk,"* or *"what did Tesla say about regulatory credits in 2024"* — was Ctrl-F across PDFs, or full-text search in an expensive subscription product like Bloomberg Terminal or Westlaw. RAG makes the same query conversational, and it makes citation automatic. Every claim in the answer is tied back to the chunk it came from. That citation discipline is exactly what an auditor or researcher needs for a workpaper, a footnote, or a literature review.

The pattern is now in production across most analytics-and-research tooling, though vendors rarely disclose the architectural details. Bloomberg's recent AI document-insights features, Thomson Reuters' Westlaw AI, Workiva's reporting AI, and AuditBoard's AI assistant all market chunk-retrieve-and-cite behavior against their proprietary corpora. Whether each one is strictly the chunk → embed → retrieve → generate pipeline you're about to build (versus a fine-tuned model that learned the corpus directly) is mostly a vendor secret. The shape is right; the implementation details vary. Inside an audit context, the same pattern powers queries like "show me prior-year findings on revenue recognition," "summarize all related-party transactions disclosed in the last five years," and similar workpaper-review tasks. In research, it's how you efficiently sweep a corpus for the first appearance of a phenomenon: first generative-AI mention, first cybersecurity incident under Item 1C, first mention of a new accounting standard's adoption.

The demo builds the smallest version of this that's still real. You'll chunk the MD&A corpus, embed each chunk, and store the vectors in a Chroma index. Then you'll expose a `query_index` and `answer_question` pair you query end-to-end. Retrieval gets evaluated with recall@5 against a hand-built question / gold-chunk set. Generation gets evaluated separately, by reading answers and marking what's actually supported. The two failure modes have different fixes, which is why they have to be measured separately. By the end you'll have an inspectable, citable interface to your firm's textual disclosures — the same shape that real workpaper-search and disclosure-research tools take.

The brief:

> **Brief.** Build a small RAG pipeline over `data/raw/mdna.parquet`. Three files:
>
> 1. `src/rag/chunk.py` — `chunk_documents(df, chunk_tokens=500, overlap=50) -> pd.DataFrame` returning `(gvkey, fyear, chunk_id, text)`. Use the `transformers` tokenizer for the same model used in encoding so token counts line up.
> 2. `src/rag/index.py` — `build_index(chunks, model_name="bert-base-uncased", store_path="data/interim/chroma")` that embeds each chunk and stores it in a Chroma collection. **Create the collection with cosine space** (`metadata={"hnsw:space": "cosine"}` works on current Chroma; the newer form is `configuration={"hnsw": {"space": "cosine"}}`) — Chroma defaults to L2, which ranks differently from the cosine similarity §1 describes. `query_index(query, k=5)` returns $\text{top-}k$ chunks; note Chroma's `query()` returns **distances** (smaller = more similar), so convert to a similarity (`score = 1 - distance`) and return it in a field named `score`, so the §5 examples that read `r['score']` line up.
> 3. `src/rag/answer.py` — `answer_question(question, k=5, model="claude-sonnet-5") -> dict` that retrieves $\text{top-}k$, builds a prompt with each chunk numbered and tagged with its `(gvkey, fyear, chunk_id)`, calls Claude with instructions to cite chunks by number for every factual claim, and returns `{"answer": str, "cited_chunk_ids": list[int], "raw_response": str}`.
>
> Then `notebooks/m13-rag.ipynb`:
>
> 1. Loads MD&A, chunks, builds index. Cache the index — building is slow, querying is fast.
> 2. Loads `data/eval/rag_questions.csv`. For each question, runs `query_index(k=5)` and records whether any gold chunk made the top 5 (recall@5).
> 3. Reports recall@5 across all eval questions. Plus recall@1 and recall@10 for context.
> 4. For each question, runs `answer_question` and records the answer plus the cited chunks.
> 5. **Hand-marks each answer**: did the cited chunks actually support the claims? Do this in a CSV by inspection.
>
> `uv add chromadb`. Reuse the embedder from Module 11.

Three deliberate choices in this brief. The 50-token overlap between chunks reduces the chance that a relevant fact straddles a chunk boundary and gets cut in half. Citation discipline — telling the generator to cite chunks by number — is what lets you tell ungrounded generation apart from real retrieval-supported answers. And caching the index keeps a sloppy notebook from re-embedding 30,000 chunks on every run.

## 5. Build the index, then poke at it before evaluating

Resist the urge to run the full pipeline end-to-end immediately. Spend ten minutes querying the index by hand first:

```python
from src.rag.index import query_index
results = query_index("supply chain disruption", k=10)
for r in results:
    # r["score"] is a cosine similarity (1 − distance): higher = more relevant
    print(r["score"], r["gvkey"], r["fyear"], r["text"][:200])
```

If the top results are obviously off-topic, your retriever is broken. Could be bad chunking, wrong embedder, or normalization mismatch. Diagnose now, before running twenty eval questions and ending up with a meaningless recall number.

## 6. Evaluate retrieval and generation separately

For retrieval, run all eval questions and compute recall@5. A small RAG with a general-purpose embedder typically lands at recall@5 between 0.6 and 0.8 on a corpus this size. Below 0.5 means the retriever needs work — bigger chunks, better embedder, or a reranker stage. Above 0.9, double-check that your gold chunks aren't trivially keyword-matchable in a way that overstates the result.

*Optional further reading: [Contextual retrieval](https://www.anthropic.com/news/contextual-retrieval) — Anthropic's practical write-up on prepending context to chunks and adding a reranker to lift recall.*

For generation, read each answer. Does every factual claim have a citation? Does the cited chunk actually contain the claim? If the retrieved chunks didn't contain the answer, did the model say so, or did it hallucinate?

Mark each answer in a CSV as `supported`, `partially_supported`, `hallucinated`, or `correctly_refused`. The hallucination rate is hallucinated / (hallucinated + supported + partially_supported). With a clean retrieval pipeline and a current generator you can often get this under 5 percent. Above 15 percent means the generator is freelancing. Try a stricter system prompt that requires every claim to be cited or the model must say "not in context."

## 7. Don't oversell what RAG can do

RAG retrieves what's already there. It doesn't synthesize across documents the way a human analyst does. It doesn't make causal claims. It doesn't fact-check the underlying documents — if a 10-K is wrong, RAG cheerfully reports that wrong claim. Worth one sentence in the write-up.

## 8. PR and merge

```bash
git push -u origin feature/m13-rag
gh pr create --title "M13: RAG over MD&A corpus" --body "$(cat <<'EOF'
## Summary
- Three-file pipeline: chunk, index (Chroma), answer (Claude).
- Eval: 20 questions, hand-marked gold chunks.
- Recall@5: [number]. Hallucination rate: [number].

## Caveats
- Embedder: `bert-base-uncased`. Modern domain-specific embedders likely improve recall.
- No reranker stage. Adding one would close some of the recall gap.
- Eval set is small; numbers are point estimates.
EOF
)"
```

Self-review: Chroma index in `data/interim/` and gitignored; eval CSV (questions + gold chunks) committed; hand-marked answer CSV committed; notebook prints recall@5 prominently so graders don't have to dig for it.

Squash-merge.

## You're done if…

- [ ] `feature/m13-rag` is merged into `main`.
- [ ] You can state recall@5 and hallucination rate as two numbers.
- [ ] You inspected at least 5 individual answers by hand and decided whether they were supported.
- [ ] The full pipeline runs end-to-end on a fresh clone (assuming the index can be rebuilt).
