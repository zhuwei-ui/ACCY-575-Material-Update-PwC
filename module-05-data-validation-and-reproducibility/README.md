# Module 5 — Data Validation and Reproducibility

> **Hands-on:** [walkthrough.md](walkthrough.md) — adding asserts, a `pandera` schema, deterministic seeds, and your first `pytest` suite.

## Motivation

Crashes are easy. The code stops, you read the error, you fix it. The dangerous bug is the one where the code happily produces the wrong number and you ship it to your manager without noticing. Most accounting-quality bugs are that kind.

This module is about catching them. The other angle is reproducibility — if you run your analysis today and get one answer, and your manager runs it tomorrow and gets a different one, you have a problem regardless of which answer is correct. Seeds, sorting, schemas, a few simple tests. None of it is hard. The discipline is doing it before you ship, not after a number turns out to be wrong.

By the end of this module you will have asserts and a `pandera` schema guarding your inputs, deterministic random seeds, and a small `pytest` suite running locally.

## Goals

- Catch the failure mode where "code ran but the answer is wrong" — the single most dangerous outcome in analytical work.
- Write `assert`-based and schema-based checks on incoming data.
- Use random seeds and deterministic ordering so your results are reproducible run-to-run.
- Use `pytest` to run a basic test suite on your analysis functions.

## Going Deeper *(optional)*

*The lecture covers what you need. These are here if you want to dig further.*

- [pandera — Quickstart](https://pandera.readthedocs.io/en/stable/) — Reference for lightweight schema validation on pandas DataFrames.
- [pytest — Get Started](https://docs.pytest.org/en/stable/getting-started.html) — Pytest's own quickstart. The first ten minutes of pytest gets you most of the value.
- [The Turing Way — Guide for Reproducible Research](https://the-turing-way.netlify.app/) — Community-written guide; the Reproducible Research chapter is the relevant entry point.

## Materials

*Slides, in-class exercises, and recordings posted here as the module approaches.*
