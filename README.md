# ACCY 575: Data Analytics Applications in Accountancy

**Syllabus — Fall 2026**

| | |
|---|---|
| **Credit** | 4 hours |
| **Term dates** | Aug 24, 2026 – Dec 9, 2026 |
| **Section A** (CRN 74620) | TR 12:30 PM – 1:50 PM · 3001 Business Instructional Facility |
| **Section I** (CRN 79414) | TR 11:00 AM – 12:20 PM · 3001 Business Instructional Facility |
| **Instructor** | *TBD* |
| **Office hours** | *TBD* |
| **Prerequisite** | ACCY 570 |

---

## 1. Course Description

The course teaches you how to do real data analysis on real accounting and finance data the way it's done in 2026: with modern tooling, a coding agent at your side, and your own judgment as the load-bearing skill.

The semester is split into two parts:

- **Part 1 (Modules 1–7) — The Modern Python Project Workflow.** You learn to set up and run a professional-grade analytical project: VS Code, `uv`, Git/GitHub, project structure, secrets handling, data validation, and AI-assisted coding. Your deliverable is a reproducible GitHub repository, not a Jupyter notebook.
- **Part 2 (Modules 8–16) — Agent-Driven Data Analysis.** You pull a real research dataset from WRDS and analyze it with a coding agent doing the implementation. Each module covers one method or tool (OLS, gradient boosting, BERT, LLM zero-shot, retrieval-augmented generation, OCR and document parsing, full agent loop) applied to the same dataset, capped by a forward-looking module on autonomous research agents. The lecture is the **concept** — when each method applies, what it assumes, what its output means, how to evaluate it. The agent writes the code; you direct, review, and correct it.

---

## 2. Learning Objectives

By the end of this course you will be able to:

- Set up a reproducible Python analytical project from scratch using modern tooling.
- Use Git and GitHub to manage, share, and review analytical work in a team workflow.
- Pull a real research dataset from WRDS and prepare it for analysis.
- Direct a coding agent through an end-to-end analysis pipeline — and critically evaluate its output rather than accepting it blindly.
- Choose appropriately among modern analytical methods (linear, tree-based, encoder-LM, LLM-based) for a given accounting question, and explain what each method is doing and why.
- Validate analytical work for correctness and reproducibility — catch the failure mode where "code ran but the answer is wrong."
- Translate business questions in accounting contexts into rigorous analyses, and communicate the results in a form a manager would actually read.

---

## 3. Required Tools

- **VS Code** (free)
- **Python**, managed via **`uv`**
- **Git** and a **GitHub** account
- A coding agent: **GitHub Copilot** (free for verified students as the *GitHub Copilot Student* plan via [GitHub Education](https://github.com/education)), or an equivalent (Claude Code, Cursor, etc.)
- **WRDS** account (provided through the College of Business; setup in Module 8)

No prior VS Code or Git experience is required. Modules 1–4 cover both from scratch.

---

## 4. Course Schedule

### Part 1 — The Modern Python Project Workflow

| Module | Topic | Focus |
|------|-------|-------|
| 1 | Terminal/CLI fundamentals; VS Code setup; first Python project with `uv` | shell, VS Code, `uv` |
| 2 | Git fundamentals: commits, branches, GitHub workflow | Git, GitHub |
| 3 | Pull requests & code review; pair-review a classmate's PR | GitHub PRs |
| 4 | Project structure & `pyproject.toml`; secrets (`.env`, `.gitignore`); a real README | `uv`, `.env` |
| 5 | Data validation & reproducibility: asserts, types, seeds, snapshot tests, `pandera` | `pandera`, `pytest` basics |
| 6 | AI-assisted coding & debugging; AI permissions; prompt-injection awareness | GitHub Copilot, VS Code debugger, OWASP LLM Top 10 |
| 7 | **Mini-project:** reproducible analysis repo (individual) | Full Part 1 workflow |

### Part 2 — Agent-Driven Data Analysis

The Part 2 panel, built in Module 8, is a sample of S&P 500 firm-years with two sides: Compustat **fundamentals** (assets, revenue, R&D, etc.) and the **10-K MD&A text** for the same firm-years, joined on `(gvkey, fyear)`. Both sides come through WRDS — the fundamentals over Postgres from your laptop, the filings off the WRDS Cloud's filesystem via a grid job your agent submits over SSH. Modules 9–10 lean on the fundamentals, 11–13 on the text, 14 extends the pipeline to documents that aren't already machine-readable, 15 ties everything together as a supervised end-to-end run, and 16 looks one level further ahead at autonomous research agents. Each module teaches one method or tool *conceptually*; the agent does the implementation.

| Module | Topic | Focus |
|------|-------|-------|
| 8 | WRDS data acquisition; pull and validate the Part 2 dataset | WRDS, `wrds` Python client |
| 9 | OLS baseline: when linear regression is the right answer (and when it isn't) | OLS, diagnostics, baseline mindset |
| 10 | Gradient-boosted trees: modern tabular SOTA | XGBoost / LightGBM, feature importance |
| 11 | BERT / FinBERT: encoder transformers for financial text | Hugging Face, embeddings as features |
| 12 | LLM zero-shot extraction & classification | structured output, evaluation |
| 13 | Embeddings + retrieval-augmented generation | semantic search, RAG over filings |
| 14 | OCR & document parsing: getting clean text from PDFs, scans, pre-EDGAR filings | LlamaParse, GCP Document AI, Tesseract |
| 15 | Agent end-to-end pipeline | Claude Code / agent SDKs running the full loop |
| 16 | Autonomous research agents: where the field is heading | Virtual Lab, AI Scientist, Deep Research, reflection |

---

## 5. Assessment

### Part 1 — Mini-project (Module 7)

Graded against objective workflow criteria: does the repo run on a clean environment, is the structure clear, do the validation checks pass, is the `results.md` write-up clear and accurate?

### Part 2 — Per-module deliverables

Each Part 2 module produces a short write-up — what method, what question, what the agent built, what you changed, what the answer is — graded on:

| Criterion | Weight | What we are looking for |
|-----------|--------|--------------------------|
| Correctness and rigor of analysis | 50% | Is the question actually answered? Are conclusions supported by the data and the method? |
| Code quality and reproducibility | 30% | Does the repo run on a clean environment? Is the structure clear? |
| Presentation / communication | 20% | Is the write-up clear and appropriately concise? |

---

## 6. AI Use Policy

AI tools (Claude, GitHub Copilot, ChatGPT, Cursor, and equivalents) are **permitted and expected** at every stage of work in this course. The skill we are evaluating is *judgment* — your ability to direct, evaluate, and correct AI output — not memorization of syntax.

This policy is consistent with the broader university policy on generative AI in coursework. If those policies update during the term, this section will be updated and announced.

---

## 7. Datasets

A single research dataset pulled from WRDS in Module 8 and reused across every Part 2 module. The panel is Compustat Fundamentals Annual (`comp.funda`) for S&P 500 firm-years 2010–2024, joined on `(gvkey, fyear)` to the Item 7 (MD&A) text of the matching 10-K filings from the SEC filing archive on the WRDS Cloud; Module 8 walks through building both halves.

---

## 8. Course Policies

- **Late submissions:** *TBD*
- **Attendance:** *TBD*
- **Academic integrity:** *TBD — note interaction with §6 AI policy*
- **Accommodations:** *TBD*

---

*This syllabus is subject to revision. Material changes will be announced and a dated version posted to the course repository.*
