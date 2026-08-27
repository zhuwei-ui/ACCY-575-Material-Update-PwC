# Module 19 — Engagement Brief: MD&A Disclosure Quality Review

*You are handed a mandate and a corpus of real filings. What you measure, which methods you use, what you ask your agent — all yours. How you approach it is entirely yours; the deliverable is what's assessed.*

## Your role

You support the **Disclosure Committee** of a public company. Before the annual report is signed, the committee wants an outside read on whether the **MD&A (Item 7)** of the most recent 10-K is complete and balanced — against peers, and against the firm's own reported results. Your findings go to the committee and to the general counsel's office; **counsel owns the language** — you flag and suggest, you do not rewrite the filing.

## The task

> Benchmark your assigned firm's filed MD&A against its peers and against its own numbers. Where is the disclosure incomplete, boilerplate, or more upbeat than the results justify? Give the disclosure committee a briefing and specific passages worth a second look.

## The data

The **Part 2 MD&A panel** you built in **[Module 8](../module-08-wrds-data-acquisition/walkthrough.md)** — real 10-K MD&A (Item 7) text joined to Compustat fundamentals on `(gvkey, fyear)`. Reuse `mdna.parquet` (and `fundamentals.parquet`) from your Part 2 repo. You may also pull a firm's most recent filing or an off-panel peer directly from **EDGAR** (the [Module 14](../module-14-ocr-and-document-parsing/walkthrough.md) approach) if you want current or additional text.

- **One row = one filing** — `(gvkey, fyear)` with the MD&A text and the matching fundamentals.
- Your **assigned focal firm** and its **peer set** (defined by industry and size) are on the assignment sheet.

## What to hand in

- **A disclosure-committee briefing** — what your benchmarking found about the MD&A's completeness and balance, with the evidence behind each claim (and honest limits on what that evidence can support).
- **Flagged passages for counsel** — specific passages or omissions worth revisiting, each with a *reason* ("consider disclosing X — peers do, and the liquidity trend implies it"), for counsel to act on — not rewritten legal text.
- **The repository behind it** — reproducible in the Part 1 way (`uv`, validation, tests, honest commit history).

Bring all of it to the review session and be ready to walk us through your result.

## Ground rules

- Individual work.
- Use AI as much as you want — you own every conclusion in the briefing regardless.
- Anything you rely on from outside the data (a disclosure requirement, a definition, a method) you look up and cite yourself.
- No prescribed method, no required tools, no checklist. Figure it out.

## Grading

You are assessed on the **deliverable** — the briefing, the suggestions, and the reproducible repository behind them — and on how well it holds up to scrutiny. There is no set procedure to follow. See [Assessment across Part 3](../README.md#assessment-across-part-3).
