# Module 4 — Walkthrough: Project Structure and Secrets

## 1. Read your `pyproject.toml`

Open the `pyproject.toml` in your `ACCY575-walkthrough` project:

```toml
[project]
name = "accy575-walkthrough"
version = "0.1.0"
description = "Add your description here"
readme = "README.md"
requires-python = ">=3.13"
dependencies = ["pandas>=3.0.3"]
```

Your exact version numbers will differ — `uv` pins whatever is current when you run it (e.g. a recent Python 3.13+ and pandas 3.x), and `name` is the lowercased project name from Module 1.

Four things to know:

- `name` — project slug.
- `requires-python` — minimum Python version. Pin it.
- `dependencies` — what `uv add` writes to.
- `[project]` is one *table*. Other tools (`ruff`, `pytest`, `mypy`) get their own under `[tool.<name>]`.

*Optional further reading: [Packaging Python Projects](https://packaging.python.org/en/latest/tutorials/packaging-projects/) (PyPA) — the full reference for `pyproject.toml` fields and project layout when you eventually want to publish or share a package.*

## 2. Move to a real project layout

Right now your project has just `main.py` at the top. As soon as a project has more than one file, organize it:

```
ACCY575-walkthrough/
├── pyproject.toml
├── uv.lock
├── README.md
├── .env                  # secrets — never committed
├── .gitignore
├── data/                 # raw/processed data — usually not committed
│   └── .gitkeep
├── src/
│   └── analysis.py       # actual code lives here
└── notebooks/            # exploratory work
```

Create the layout:

```bash
cd ~/Projects/accy575/ACCY575-walkthrough     # the project you created in Module 1
mkdir -p data src notebooks
touch data/.gitkeep src/__init__.py src/analysis.py
mv explore.ipynb notebooks/   # move your Module 1 notebook in (skip if you don't have one)
```

### What each line does *(optional reference)*

*Lecture covers this. Skim if a specific command was unclear.*

- **`cd ~/Projects/accy575/ACCY575-walkthrough`** — `cd` changes directory; `~` expands to your home folder. Every subsequent command now runs *inside* your project.
- **`mkdir -p data src notebooks`** — creates three folders in one shot. The `-p` flag means "don't error if the folder already exists" — safe to re-run.
- **`touch data/.gitkeep src/__init__.py src/analysis.py`** — `touch` creates empty files. Three of them, each there for a specific reason:
  - **`data/.gitkeep`** — Git doesn't track empty folders, so we drop a placeholder file inside `data/` to make Git pick up the folder. (`.gitkeep` is just a convention; the name has no special meaning to Git.)
  - **`src/__init__.py`** — in Python, a folder is only an importable *package* if it contains an `__init__.py`. Without this, `from src.analysis import ...` fails. The file can be empty; what matters is that it exists.
  - **`src/analysis.py`** — the (still empty) file where your actual analysis code will live.
- **`mv explore.ipynb notebooks/`** — `mv` moves (or renames) a file. The trailing `/` on `notebooks/` makes it explicit you're moving the file *into* that folder. If you skipped Module 1's notebook step, this command will fail with `No such file or directory` — harmless.

Move logic from `main.py` into `src/analysis.py`. Keep `main.py` as the thin entry point (or remove it and run modules directly with `uv run python -m src.analysis`).

## 3. Set up `.env` for secrets

An **API key** is a long string of characters that identifies your account when your code calls an external service — OpenAI, Anthropic, GitHub, AWS, Stripe, and so on. It functions like a password attached to a billable account: anyone holding it can make requests on your behalf, and on your bill. Three things to keep in mind:

- **It identifies you.** The service uses the key to know which account the call belongs to.
- **It authorizes you.** Possession alone grants access — there's no second factor in most cases.
- **It bills you.** Every call counts against your quota or wallet.

This is why leaking an API key is a real incident, not a theoretical one. Bad actors scrape public GitHub repositories for leaked keys and run up charges (or worse) on your account before you notice. The single rule for this section: **never put an API key directly in your code, and never commit one to Git.**

The standard pattern is `.env` — a file that lives only on your machine, listed in `.gitignore` so Git never sees it, that your code reads from at runtime.

*Optional further reading: [The Twelve-Factor App — III. Config](https://12factor.net/config) (Adam Wiggins) — the classic short argument for keeping config and secrets in the environment rather than in code.*

In your project root:

```bash
touch .env
echo ".env" >> .gitignore   # do this BEFORE adding a real key — uv's default .gitignore does NOT exclude .env
```

(You'll expand `.gitignore` properly in the next section; this one line makes sure a key can't slip into a commit in the meantime.)

Open `.env` in VS Code and add a fake key (a real one would never be hard-coded into a tutorial, of course):

```
OPENAI_API_KEY=sk-fake-do-not-use-this-in-real-code
```

Install the loader:

```bash
uv add python-dotenv
```

In `src/analysis.py`:

```python
import os
from dotenv import load_dotenv

load_dotenv()
api_key = os.environ["OPENAI_API_KEY"]
print(f"loaded a key of length {len(api_key)}")
```

Run it: `uv run python -m src.analysis`. You should see the length printed.

## 4. Update your `.gitignore` for secrets and data

You've been using the baseline `.gitignore` from `uv init` for two modules. Now that you have real secrets (and soon, real data files), expand it. Open `.gitignore` and add the **Secrets** and **Data** sections:

```
# Python
__pycache__/
*.pyc
.venv/

# Secrets
.env
.env.*

# Data — keep raw/interim pulls and bulk data files out of Git, but not test fixtures
data/raw/
data/interim/
data/*.csv
data/*.parquet
data/*.xlsx

# OS
.DS_Store
Thumbs.db

# IDE
.vscode/settings.json
.idea/
```

The single rule: **`.env` must never be committed.** Once a real API key lands in Git history, it is compromised forever, even if you delete the file in a later commit.

## 5. Write a real README

Your `README.md` is the front door of your repository. When a teammate, a recruiter, or future-you opens the repo on GitHub, that rendered README is what they see first — before they click into a single line of code. A stub README (`# ACCY575-walkthrough`) signals nothing; a clear one signals competence.

A useful project README answers four questions in under one screen of text:

1. **What is this project?** One sentence.
2. **How do I set it up?** The commands to get from a fresh `git clone` to a working environment.
3. **How do I run it?** The command that actually produces output.
4. **Where do the results live?** A pointer to `results.md` or wherever the analysis writeup goes.

Open `README.md` (uv init created it empty) and fill it in with something like:

````markdown
# ACCY575-walkthrough

Coursework and analyses for ACCY 575 — Data Analytics Applications in Accountancy.

## Setup

```bash
uv sync
```

## Run

```bash
uv run python -m src.analysis
```

## Results

Latest writeup in [results.md](results.md). *(Created in Module 7.)*
````

A few rules of thumb that survive any project:

- **Write for someone who doesn't know your project.** Don't assume context the reader doesn't have.
- **Keep it under one screen.** Long READMEs go unread. Move detail to a `docs/` folder if the project grows.
- **Update it whenever setup or run commands change.** A wrong README is worse than no README — it actively misleads.

This is the same skill you'd use to document an audit data pipeline, an FP&A model, or any deliverable where someone other than you needs to be able to pick it up.

## You're done if…

- [ ] Your project has `pyproject.toml`, `src/`, `data/`, `notebooks/` layout.
- [ ] `.env` exists with at least one variable, loaded via `python-dotenv`.
- [ ] `.gitignore` excludes `.env` and raw/interim data (`data/raw/`, `data/interim/`, `data/*.csv`, `data/*.parquet`, `data/*.xlsx`).
- [ ] Your `README.md` answers: what is this, how to set up, how to run, where results live.
