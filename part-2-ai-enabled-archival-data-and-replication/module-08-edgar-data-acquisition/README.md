# Module 8 — EDGAR Data Acquisition: 10-Ks, XBRL Financials, and Restatements

> **Hands-on:** [walkthrough.md](walkthrough.md) — setting up the Part 2 repo, configuring an off-the-shelf EDGAR library (edgartools) and the SEC's own financial-statement datasets, and briefing an agent through five steps that end in one CIK-year panel: the filing universe, the 10-K panel, its financial statement items as first reported, the Item 4.02 8-Ks that later declared them unreliable, and the extraction of text from both.

## Motivation

Every public company's financial reporting passes through one place: the SEC's EDGAR system. The commercial research databases (Compustat, Audit Analytics, the WRDS SEC suite) are, to a large extent, EDGAR that someone else has cleaned and then sold back to you. Learning to go to the source directly gets you three things. It's free, so what you build here keeps working after graduation with no subscription behind it. It's current, down to filings made this morning. And it's transparent: when a number looks wrong, you can open the filing it came from.

The cost is that EDGAR is raw. Nobody has decided for you which filing is the "real" 10-K for a firm-year, which XBRL tag means revenue, or which fiscal years a restatement actually touched. This module is about making those decisions deliberately and writing them down, because in research, and in audit analytics, those decisions are where results come from.

The dataset you build answers one question, and the next module asks it: **can you tell, from what a company files, that its financial statements will later have to be restated?** Module 9 puts a specific idea to work on this panel: that the topics a 10-K discusses help predict misreporting, beyond what the financial numbers already tell you. The idea, and the design that tests it honestly, come from Brown, Crowley, and Elliott (2020, *Journal of Accounting Research*).

## Goals

- Stand up a new GitHub repository for Part 2, separate from your Part 1 work.
- Follow EDGAR's fair-access rules (declared `User-Agent`, at most 10 requests per second) by configuring an existing, open-source client, **edgartools**, rather than writing one, and write down the rules your coding agent must follow when using it.
- Build a **CIK-year panel of 10-K filings**, and map each filing to the fiscal year it reports on (not the calendar year it was filed in).
- Pull **key financial statement items** as they were *first reported* in that 10-K, from the SEC's Financial Statement Data Sets, and understand why the obvious routes return restated numbers instead.
- Identify **Item 4.02 (non-reliance) 8-Ks**, the disclosure a company files when previously issued financial statements can no longer be relied on.
- **Extract** two things from text: the fiscal periods each 4.02 8-K declares unreliable, which turns an event into CIK-year labels, and the narrative text of a case-control sample of 10-Ks (every restated CIK-year plus 20% of the rest, with sampling weights) for Module 9's topic model.
- Validate the assembled panel with `pandera`, flag the cases that need judgment (restatements disclosed before the 10-K was filed, the 2021 SPAC wave, combined filings), and decide what belongs in Git.

## Going Deeper *(optional)*

*The lecture covers what you need. These are here if you want to dig further.*

- [Accessing EDGAR Data](https://www.sec.gov/search-filings/edgar-search-assistance/accessing-edgar-data) — SEC. The canonical statement of the `User-Agent` and rate-limit rules in §3, and the layout of the full-index files used in §4.
- [edgartools documentation](https://dgunning.github.io/edgartools/) — the library used throughout. Note that some of its pages lag behind the code; when they disagree, the pinned version's behavior wins.
- [Financial Statement Data Sets](https://www.sec.gov/data-research/sec-markets-data/financial-statement-data-sets) — SEC. The quarterly as-filed numbers used in §6, with a PDF that documents every field in `sub.txt` and `num.txt`.
- [EDGAR Application Programming Interfaces](https://www.sec.gov/search-filings/edgar-application-programming-interfaces) — SEC. The raw endpoints (`submissions`, `companyfacts`, `frames`) that edgartools calls for you, if you want to see what's underneath.
- [Form 8-K instructions](https://www.sec.gov/files/form8-k.pdf) — SEC. Item 4.02's actual wording: what triggers it, and what the company must say.
- Hennes, Leone, and Miller (2008), "The Importance of Distinguishing Errors from Irregularities in Restatement Research," *The Accounting Review* 83(6). The standard argument for why "restatement" is not one thing, and the basis for the irregularity flag in §8a.
- Dechow, Ge, Larson, and Sloan (2011), "Predicting Material Accounting Misstatements," *Contemporary Accounting Research* 28(1). Where the financial variables you pull in §6 come from.

## Materials

*Slides, in-class exercises, and recordings posted here as the module approaches.*
