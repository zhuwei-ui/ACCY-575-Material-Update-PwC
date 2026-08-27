# Module 7 — Walkthrough: Mini-Project Build

This module is assembly. You build one project that combines everything from Modules 1–6 and submit it as a GitHub repo. See [README.md](README.md) for the full deliverable spec.

## Suggested working order

You have 5–7 days. The trap is to spend 100% of it on the analysis and 0% on the workflow. Reverse that.

### Day 1 — Bring your project up to spec

You're not starting a new project. You'll polish the `ACCY575-walkthrough` project you've been building since Module 1.

```bash
cd ~/Projects/accy575/ACCY575-walkthrough
```

Audit it against the deliverable spec:

- [ ] Layout includes `pyproject.toml`, `src/`, `tests/`, `notebooks/`, `data/`. If anything is missing, create it.
- [ ] `.gitignore` excludes `.env`, `data/raw/`, caches, `.venv/`.
- [ ] `uv sync` runs cleanly.
- [ ] The repo is on GitHub and the latest commit shows there.

Anything missing or out of date — fix it now, in its own branch, with a clean commit message.

### Day 2 — Validate the data

- Load the dataset (instructor will provide).
- Eyeball it in `notebooks/01-explore.ipynb` for 30 minutes. What columns are there? What's their shape? What's weird?
- In `src/validate.py`, write a `pandera` schema that codifies what you saw.
- Run the validation as the first step of your pipeline.
- Commit.

### Days 3–5 — Build the analysis

- One branch per significant change: `feature/load-cleanup`, `feature/aggregation`, `feature/charts`.
- Code lives in `src/`, **not** in the notebook. The notebook is for exploration; the deliverable is `src/`.
- Write tests as you go. Don't wait until the end.
- For anything non-trivial, write a brief and have an AI assistant draft it (Module 6 muscle memory). Read what it produced. Fix what's wrong.

### Day 6 — Write `results.md`

This is what your manager actually reads — not your code. The hardest part is being concise. Aim for one page max.

Structure: one short paragraph (or one table) for each of —

- **Question.** What were you asked, exactly?
- **Approach.** How did you go about it? Skip the false starts.
- **Results.** The actual answer. A small table or a single chart usually beats prose.
- **Caveats.** What's wrong with this answer? Missing data, edge cases you skipped, assumptions baked in. Be honest — that's how trust gets built.

A reader who knows nothing about your code should still understand the answer.

Example shape:

```markdown
# Monthly Spending by Category — Q1 2026

## Question
How does spending in each category trend month-over-month?

## Approach
Loaded transactions.csv, grouped by month-end and category, summed amount, filled missing cells with 0.

## Results
| Month | Food | Travel | Office |
|-------|------|--------|--------|
| Jan   | $1,200 | $0   | $450 |
| Feb   | $0     | $200 | $510 |
| ...   |        |      |      |

## Caveats
- Category labels are vendor-self-reported; "Office Supplies" and "office supplies" appear separately. Cleaned manually.
- Q1 2026 transactions before Jan 7 may be missing — system migration window.
```

`results.md` is graded the same as your code.

### Day 7 — Self-review

```bash
# Clone your own repo to a fresh location, run from scratch:
cd /tmp
git clone https://github.com/<your-username>/ACCY575-walkthrough.git
cd ACCY575-walkthrough
uv sync
uv run python -m src.analysis
```

If it doesn't run, fix it. **Do this before you submit.** This is the single most common failure mode.

Open at least one PR on a small fix you found. Merge it.

Verify `.env` is not in your Git history:

```bash
git log --all --source -- .env
```

That should print nothing. If it prints anything, you have a problem — see your instructor before submitting.

## Common mistakes (each one costs you a letter grade)

- **One giant "final" commit.** Tells the grader you treated Git as a submission portal.
- **`.env` committed.** Rotate any keys; reset history if necessary.
- **Notebook-only submission.** Doesn't meet the deliverable.
- **`results.md` written like a code comment.** Write for a non-coder manager — your future boss.
- **Tests that don't actually test anything** (`assert True` etc.). The grader runs them.

## You're done if…

- [ ] A classmate could clone your repo, run `uv sync && uv run python -m src.analysis`, and get the same answer you got.
- [ ] Your `results.md` answers the question without reference to the code.
- [ ] Your commit history tells a coherent story.
- [ ] All checks in [README.md's deliverable list](README.md) are met.
