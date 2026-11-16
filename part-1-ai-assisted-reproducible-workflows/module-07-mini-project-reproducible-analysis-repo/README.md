# Module 7 — Mini-Project: Reproducible Analysis Repo

> **Hands-on:** [walkthrough.md](walkthrough.md) — suggested working order and self-review checklist.

## Motivation

Mini-project time. Combine everything from Modules 1–6 into one repo and submit that.

What matters most here is whether the thing actually runs end-to-end on a fresh clone, more than how clever the analysis is. That emphasis is deliberate. It's the same way real work gets evaluated: a manager who can't reproduce your numbers won't trust the rest of what you say, no matter how good the analysis was. Get the workflow right and the analysis quality follows.

This is a build module, not a reading one.

## Goal

Combine everything from Modules 1–6 into a complete, reproducible analytical project that you can hand to a manager (or a future employer) and they can run it.

## Deliverable

A public GitHub repository — the same project you've been building since Module 1 (`ACCY575-walkthrough`, or whatever you named it). It must contain:

- A working `pyproject.toml` and `uv.lock`. `uv sync && uv run python <entry>` should run your analysis end-to-end on a clean machine.
- A clear directory layout (e.g., `data/`, `src/`, `notebooks/`, `results/`).
- A `.gitignore` that excludes `.env`, raw data, caches, and anything else that should not be committed.
- At least one `pandera` schema or `assert`-based validation on input data.
- A `results.md` with: the question, the approach, the answer, and the caveats — written for a manager, not a grader.
- An honest commit history. More than one commit. No single "final upload" commit.

## Dataset

[`datasets/general_2016_2018.csv`](../../datasets/general_2016_2018.csv) — single audits of federal award recipients from the Federal Audit Clearinghouse: 110,948 rows, 47 columns, audit years 2016–2018. Copy it into your project's `data/raw/` (the command is in [datasets/README.md](../../datasets/README.md), along with the columns and why the contact fields were removed).

## Going Deeper *(optional)*

Nothing new this module. This is a build module.
