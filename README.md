# ACCY 575: Data Analytics Applications in Accountancy

**Syllabus — Spring 2027**

| | |
|---|---|
| **Credit** | 3 hours |
| **Term dates** | Jan 19, 2027 – May 5, 2027 |
| **Section A** (CRN 66492) | MW 9:30 AM – 10:50 AM · 130 Wohlers Hall |
| **Section I** (CRN 66493) | MW 11:00 AM – 12:20 PM · 226 Wohlers Hall |
| **Instructor** | Wei Zhu |
| **Office hours** | *TBD* |
| **Prerequisite** | ACCY 570 |

---

## 1. Course Description

The course teaches you how to do real data analysis on real accounting and finance data the way it's done in 2027: with modern tooling, a coding agent at your side, and your own judgment as the load-bearing skill.

The semester has five parts — three required, plus two optional ones for students who want to go further:

- **Part 1 (Modules 1–7) — AI-Assisted Reproducible Workflows.** You learn to set up and run a professional-grade analytical project: VS Code, `uv`, Git/GitHub, project structure, secrets handling, data validation, and AI-assisted coding. Your deliverable is a reproducible GitHub repository, not a Jupyter notebook.
- **Part 2 (Modules 8–9) — AI-Enabled Archival Data and Replication.** You build a research dataset straight from the SEC's EDGAR system: a CIK-year panel of 10-K filings, financial statement items from XBRL *as first reported*, Item 4.02 restatement disclosures, and the text of both, with a coding agent doing the implementation from your briefs. Then you use it to implement an idea from a published *Journal of Accounting Research* paper: that the topics a 10-K discusses help predict misreporting, beyond what the financial numbers show.
- **Part 3 (Modules 10–16) — Accounting Analytics Cases.** Seven cases, each a self-contained engagement: a purchasing-card internal audit, a differential analysis of a travel program, a relational database for financial reporting, earnings-call text analytics, an advisory case on Chicago taxi data, a tax migration case, and the PwC case. Case materials and requirements are on Canvas.
- **Part 4 (Optional, Modules 17–25) — Agentic AI Methods and Pipelines.** A self-paced extension: build a Compustat + 10-K MD&A panel on WRDS, then apply one modern method per module (OLS baseline, gradient-boosted trees, BERT, LLM zero-shot, retrieval-augmented generation, OCR, a full agent pipeline, autonomous research agents). It runs on a WRDS class account that expires when the term ends, so take it alongside Part 3.
- **Part 5 (Optional, Modules 26–28) — Capstone Engagements.** Three open-ended engagements (managerial accounting, financial communications, and an advisory capstone) with a mandate and no prescribed method. They build on the Part 4 panel.

---

## 2. Learning Objectives

By the end of this course you will be able to:

- Set up a reproducible Python analytical project from scratch using modern tooling.
- Use Git and GitHub to manage, share, and review analytical work in a team workflow.
- Build a research dataset directly from SEC EDGAR, and explain every decision that turns raw filings into a panel: which filing, which fiscal year, which number, and what was known when.
- Implement a published empirical design on data you assembled yourself, and reason about how your own choices shape the answer.
- Direct a coding agent through an end-to-end analysis pipeline, and critically evaluate its output rather than accepting it blindly.
- Apply data analytics to core accounting problems: audit testing, differential analysis, database design, disclosure text, and advisory work.
- Validate analytical work for correctness and reproducibility, and catch the failure mode where "code ran but the answer is wrong."
- Translate business questions in accounting contexts into rigorous analyses, and communicate the results in a form a manager would actually read.

---

## 3. Required Tools

- **VS Code** (free)
- **Python**, managed via **`uv`**
- **Git** and a **GitHub** account
- A coding agent: **GitHub Copilot** (free for verified students as the *GitHub Copilot Student* plan via [GitHub Education](https://github.com/education)), or an equivalent (Claude Code, Cursor, etc.)
- An **Anthropic API key** with a few dollars of credit, for the LLM extraction step in Module 8 (setup in that module)
- **Microsoft Excel** (Module 10; optional in Module 11) and **Tableau Public** (free; Module 14)
- **DB Browser for SQLite** (free; the Module 12 notebook is written around it, though its brief shows how to do every step in SQL instead)
- *Optional parts only:* a **WRDS** class account (the instructor provides the class code; setup in Module 17)

No prior VS Code or Git experience is required. Modules 1–4 cover both from scratch. EDGAR needs no account at all.

---

## 4. Course Schedule

The repository is organized into five sections, one folder each; every module folder keeps its course-wide number, so Module 17 is `part-4-agentic-ai-methods-and-pipelines/module-17-wrds-data-acquisition/`.

| Folder | Modules | Required? |
|--------|---------|-----------|
| `part-1-ai-assisted-reproducible-workflows/` | 1–7 | Yes |
| `part-2-ai-enabled-archival-data-and-replication/` | 8–9 | Yes |
| `part-3-accounting-analytics-cases/` | 10–16 | Yes |
| `part-4-agentic-ai-methods-and-pipelines/` | 17–25 | Optional |
| `part-5-capstone-engagements/` | 26–28 | Optional |

### Part 1 — AI-Assisted Reproducible Workflows

| Module | Topic | Focus |
|------|-------|-------|
| 1 | Terminal/CLI fundamentals; VS Code setup; first Python project with `uv` | shell, VS Code, `uv` |
| 2 | Git fundamentals: commits, branches, GitHub workflow | Git, GitHub |
| 3 | Pull requests & code review; pair-review a classmate's PR | GitHub PRs |
| 4 | Project structure & `pyproject.toml`; secrets (`.env`, `.gitignore`); a real README | `uv`, `.env` |
| 5 | Data validation & reproducibility: asserts, types, seeds, snapshot tests, `pandera` | `pandera`, `pytest` basics |
| 6 | AI-assisted coding & debugging; AI permissions; prompt-injection awareness | GitHub Copilot, VS Code debugger, OWASP LLM Top 10 |
| 7 | **Mini-project:** reproducible analysis repo (individual) | Full Part 1 workflow |

### Part 2 — AI-Enabled Archival Data and Replication

Module 8 builds one panel, and Module 9 uses it. The panel is one row per **CIK-year** (company × fiscal year, 2011–2021), with four pieces: the 10-K itself; its key financial statement items from XBRL, *as originally reported* (not as later restated), matched to the 10-K by accession number; whether an Item 4.02 8-K later declared that year's statements unreliable, and whether the disclosure points to an irregularity (matched by company and fiscal year); and the full 10-K text for a case-control sample of CIK-years.

| Module | Topic | Focus |
|------|-------|-------|
| 8 | EDGAR data acquisition: 10-K CIK-year panel; XBRL financials as first reported; Item 4.02 restatement 8-Ks; extracting restated periods (regex vs. LLM, against a hand-labeled gold set) and 10-K text | edgartools, SEC Financial Statement Data Sets, `pandera`, Anthropic API |
| 9 | Financial, style, and rolling-window LDA topic predictors of restatements, evaluated out of sample, following Brown, Crowley & Elliott (2020, *JAR*) | `scikit-learn` LDA and logistic regression, pooled AUC, cluster bootstrap |

### Part 3 — Accounting Analytics Cases

> 📝 **My note:** Adjust these cases after going through all modules.

Each module is one case. Case materials, requirements, deliverables, and due dates are on **Canvas**.

| Module | Case |
|------|--------|
| 10 | Purchasing-card audit analytics |
| 11 | Differential analysis of a centralized travel program |
| 12 | Relational databases and SQL for financial reporting |
| 13 | Processing earnings conference calls |
| 14 | Yellow Cab Chicago |
| 15 | Tax migration |
| 16 | PwC case |

### Part 4 (Optional) — Agentic AI Methods and Pipelines

Not required. The panel, built in Module 17, is a sample of S&P 500 firm-years with two sides: Compustat **fundamentals** and the **10-K MD&A text** for the same firm-years, joined on `(gvkey, fyear)`. Both come through WRDS: the fundamentals over Postgres from your laptop, the filings off the WRDS Cloud's filesystem via a grid job your agent submits over SSH. Each module teaches one method *conceptually*; the agent does the implementation.

| Module | Topic | Focus |
|------|-------|-------|
| 17 | WRDS data acquisition; pull and validate the Part 4 dataset | WRDS, `wrds` Python client |
| 18 | OLS baseline: when linear regression is the right answer (and when it isn't) | OLS, diagnostics, baseline mindset |
| 19 | Gradient-boosted trees: modern tabular SOTA | XGBoost / LightGBM, feature importance |
| 20 | BERT / FinBERT: encoder transformers for financial text | Hugging Face, embeddings as features |
| 21 | LLM zero-shot extraction & classification | structured output, evaluation |
| 22 | Embeddings + retrieval-augmented generation | semantic search, RAG over filings |
| 23 | OCR & document parsing: getting clean text from PDFs, scans, pre-EDGAR filings | LlamaParse, GCP Document AI, Tesseract |
| 24 | Agent end-to-end pipeline | Claude Code / agent SDKs running the full loop |
| 25 | Autonomous research agents: where the field is heading | Virtual Lab, AI Scientist, Deep Research, reflection |

### Part 5 (Optional) — Capstone Engagements

Not required. Each module is a mandate and a real dataset with no walkthrough and no prescribed method. Each folder holds a short **engagement brief**.

| Module | Domain | The engagement | Real data | Format |
|------|--------|----------------|-----------|--------|
| 26 | Managerial accounting | Cost behavior & margin resilience — what a revenue downturn does to operating margin; a CFO decision memo | Compustat fundamentals (the Part 4 panel) | Individual |
| 27 | Financial communications | MD&A disclosure-quality review vs. peers and vs. the numbers; a disclosure-committee briefing | The Part 4 10-K MD&A panel + EDGAR | Individual |
| 28 | Advisory (**capstone**) | Acquisition screening & diligence from a two-sentence mandate; an investment-committee report and deck | Part 4 Compustat panel + incremental WRDS pull + EDGAR | Group |

---

## 5. AI Use Policy

AI tools (Claude, GitHub Copilot, ChatGPT, Cursor, and equivalents) are **permitted and expected** at every stage of work in this course. The skill we are evaluating is *judgment* — your ability to direct, evaluate, and correct AI output — not memorization of syntax.

You are accountable for every number in anything you hand in, as if you had produced it yourself.

This policy is consistent with the broader university policy on generative AI in coursework. If those policies update during the term, this section will be updated and announced.

---

## 6. Datasets

### Part 2 — EDGAR

One research panel, built in Module 8 from the SEC's EDGAR system and reused in Module 9: 10-K filings for fiscal years 2011–2021 mapped to CIK-years, key financial statement items from each filing's own XBRL, Item 4.02 non-reliance 8-Ks turned into restated-year labels, and the full text of a case-control sample of 10-Ks. EDGAR is public and free. The data isn't committed to Git because of its size; the code that rebuilds it is.

### Part 3 — One dataset per case

Each case comes with its own dataset. The data, together with the case requirements, is on **Canvas**. Several cases use third-party material licensed for enrolled students only, so don't redistribute it.

### Getting the course materials

The modules and walkthroughs live in this repository. Clone it once, next to your own projects (your instructor gives you read access; this needs the GitHub account and the signed-in `gh` CLI you set up in Module 2):

```bash
gh repo clone jingran-h/ACCY-575-Data-Analytics-Applications-in-Accountancy ~/Projects/accy575/course-materials
```

Run `git -C ~/Projects/accy575/course-materials pull` when the course materials are updated. This repository is private, so don't fork it publicly.

**Part 3 case materials are on Canvas, not here.** Each Part 3 folder in this repository is a placeholder that names the case and its module number; the data, requirements, and due date all come from Canvas.

### Optional parts

Part 4 builds its own panel on WRDS in Module 17: Compustat Fundamentals Annual (`comp.funda`) for S&P 500 firm-years 2010–2024, joined on `(gvkey, fyear)` to the Item 7 (MD&A) text of the matching 10-K filings on the WRDS Cloud. It requires the course's WRDS class account. Part 5 reuses that panel.

---

## 7. Course Policies

- **Late submissions:** *TBD*
- **Attendance:** *TBD*
- **Academic integrity:** *TBD — note interaction with §5 AI policy*
- **Accommodations:** *TBD*

---

*This syllabus is subject to revision. Material changes will be announced and a dated version posted to the course repository.*
