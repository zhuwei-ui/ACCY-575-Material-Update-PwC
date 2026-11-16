# Module 20 — BERT for Financial Text *(optional)*

> **Optional — Part 4 (Agentic AI Methods and Pipelines).** Not required for the course. Part 4 is a self-paced extension: it builds its own WRDS panel in [Module 17](../module-17-wrds-data-acquisition/README.md) and applies one modern method per module to it. If you take it, do it alongside Part 3, not after it: it runs on the course's WRDS class account, which expires when the term ends, so finish Module 17's pulls, and any other work that needs WRDS itself, before then.

> **Hands-on:** [walkthrough.md](walkthrough.md) — using a BERT-family encoder to turn the textual half of your WRDS dataset into features.

## Motivation

A lot of the signal in accounting and finance lives in text — 10-K MD&A, conference call transcripts, analyst reports, audit committee disclosures. Until 2018 the standard tool for this was a dictionary (Loughran-McDonald) or a bag-of-words model. Both still work in textbook examples and neither is what you should be using for new work.

What you should reach for is an encoder transformer — BERT, FinBERT, or a more recent variant — that turns a paragraph into a dense vector capturing meaning rather than counting words. You then feed those vectors into the same kinds of models you already know (Module 18's OLS, Module 19's boosting) as features. That's how text gets into a quantitative accounting analysis without your having to train a deep model from scratch.

## Goals

- Understand what an encoder transformer does and what its output vectors represent.
- Use a pre-trained Hugging Face model (BERT or FinBERT) to encode your MD&A corpus, deciding on chunking, truncation, and pooling.
- Add the embeddings as features to your Module 18 or Module 19 model and report how much they help.
- Estimate the GPU and runtime cost of encoding before launching the job.

## Going Deeper *(optional)*

*The walkthrough covers what you need. These are here if you want to dig further.*

- [LLM Course — Transformer Architectures (encoder models)](https://huggingface.co/learn/llm-course/chapter1/6) — Hugging Face. Short conceptual overview; read this before touching code.
- [FinBERT (`yiyanghkust/finbert-tone`)](https://huggingface.co/yiyanghkust/finbert-tone) — Huang, Wang, and Yang (2023), *Contemporary Accounting Research* 40(2). Domain-tuned BERT for financial text; a useful baseline to compare against general-purpose BERT.

## Materials

*Slides and recordings from earlier offerings, if any, are posted here.*
