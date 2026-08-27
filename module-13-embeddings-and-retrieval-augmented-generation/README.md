# Module 13 — Embeddings and Retrieval-Augmented Generation

> **Hands-on:** [walkthrough.md](walkthrough.md) — building a small RAG system over the textual half of your WRDS dataset and using it to answer accounting questions.

## Motivation

LLMs by themselves do not know what's in a specific 10-K filing or a specific earnings call. Their training data is broad and outdated. Retrieval-augmented generation is the standard fix: convert your documents into embeddings (the same operation as Module 11), store them in a vector index, and at query time fetch the most relevant chunks and hand them to the LLM as context.

It's how analyst tools, legal-tech products, and most enterprise "ask your documents" features work internally. For accounting, RAG turns "summarize what 200 firms said about supply-chain risk in 2024" from a manual reading exercise into a query.

## Goals

- Understand the RAG pipeline at the level of components: chunker, embedder, vector store, retriever, generator.
- Build a small RAG system over your Module 8 MD&A corpus.
- Recognize RAG-specific failure modes: bad chunking, retrieval misses, ungrounded generation, unsupported citations.
- Evaluate retrieval and generation quality separately on a hand-built question set.

## Going Deeper *(optional)*

*The lecture covers what you need. These are here if you want to dig further.*

- [Contextual retrieval](https://www.anthropic.com/news/contextual-retrieval) — Anthropic. Short, practical write-up of what works in modern RAG.
- [LlamaIndex documentation](https://docs.llamaindex.ai/) — LlamaIndex. Framework the agent will likely use; the architecture page is the relevant entry.

## Materials

*Slides, in-class exercises, and recordings posted here as the module approaches.*
