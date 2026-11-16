# Module 23 — OCR and Document Parsing *(optional)*

> **Optional — Part 4 (Agentic AI Methods and Pipelines).** Not required for the course. Part 4 is a self-paced extension: it builds its own WRDS panel in [Module 17](../module-17-wrds-data-acquisition/README.md) and applies one modern method per module to it. If you take it, do it alongside Part 3, not after it: it runs on the course's WRDS class account, which expires when the term ends, so finish Module 17's pulls, and any other work that needs WRDS itself, before then.

> **Hands-on:** [walkthrough.md](walkthrough.md) — extracting clean text from messy PDFs and scanned documents, then folding the results into your Module 22 RAG index.

## Motivation

Modules 18 through 22 ran on clean text. WRDS hands you 10-K MD&A as parsed plaintext, joined to numerical fundamentals on `(gvkey, fyear)`, and you build models on top. Real accounting documents don't always arrive in that shape. Audit workpapers are scanned PDFs. Pre-EDGAR filings (anything before about 1994) only exist as page images. K-1s, paper invoices, expense receipts, prior-period exhibits — all of it shows up as PDFs or scans where the text isn't directly addressable.

OCR (optical character recognition) is what turns those images back into text. Modern OCR has moved well beyond the Tesseract-style "convert pixels to characters" of ten years ago. LlamaParse, GCP Document AI, and the OCR features inside Claude and GPT do layout-aware parsing: tables come out as tables, headings as headings, and the natural reading order is preserved across multi-column pages. The result drops cleanly into a RAG index, an LLM classifier, or any of the methods you've already built.

## Goals

- Understand the difference between digital-PDF text extraction and image OCR, and know which tool you reach for in each case.
- Run the same document through LlamaParse, GCP Document AI, and Tesseract and compare what each produces.
- Pick the right tradeoff between cost, quality, and operational complexity for your specific accounting task.
- Extend Module 22's RAG index with text recovered from non-machine-readable source documents.

## Going Deeper *(optional)*

*The walkthrough covers what you need. These are here if you want to dig further.*

- [LlamaParse documentation](https://www.llamaindex.ai/llamaparse) — LlamaIndex. The "supported file types" and "parsing instructions" pages are most relevant.
- [Google Cloud Document AI](https://cloud.google.com/document-ai) — Google. Enterprise OCR with specialized processors for invoices, receipts, contracts, and form-type documents.
- [Tesseract OCR](https://tesseract-ocr.github.io/tessdoc/) — open-source baseline. Useful as a free local fallback and as the floor your paid services should clearly beat.
- [SEC EDGAR full-text search](https://www.sec.gov/edgar/search/) — search the full text of EDGAR filings; note it only covers filings submitted electronically since 2001. Handy for pulling test documents like the modern 10-K.

## Materials

*Slides and recordings from earlier offerings, if any, are posted here.*
