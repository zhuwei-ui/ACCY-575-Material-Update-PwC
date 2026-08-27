# Module 15 — Walkthrough: Direct an Agent End-to-End

## 1. What changes at this level

In Modules 9 through 14, the coding agent was a subroutine writer. You wrote the brief, the agent produced one file, you reviewed it. Module 15 is the level above. You hand the agent the entire task — read the dataset, pick a method, fit it, evaluate it, write the report — and it operates inside a tool-use loop, executing shell commands and writing files autonomously while you supervise.

Most modern coding agents implement variants of the ReAct loop:

```
loop:
    1. THOUGHT — agent reasons about the next action
    2. ACTION — agent calls a tool (read file, run shell, edit, search the web)
    3. OBSERVATION — environment returns the result
    4. agent updates its state
until the agent decides the task is complete (or you stop it)
```

*Optional further reading: [OpenAI Agents SDK guide](https://platform.openai.com/docs/guides/agents) — alternative tooling that implements this same agent-loop pattern, if you want to see it framed in a different stack.*

The math isn't the interesting part here. The operational details are.

The agent executes real commands on your machine. Files get written, shells run, network requests go out. That's a different beast from a chatbot suggesting code you copy-paste, and it changes how careful you have to be.

Supervision is part of the loop. Most agents pause and ask before running shells or edits, if you've configured them to. The default in some tools is "ask once at the start, then auto-approve everything," and you have to opt out of that explicitly for any serious work.

The brief is the deliverable. When the run finishes, the artifact graders see is your repo plus the write-up, but the brief is the central evidence of *your* analytical contribution. Treat it like a senior engineer's design doc: precise, unambiguous, scoped.

A senior analyst writes a brief, the agent runs the analysis, the senior reviews and signs the deliverable. The job has moved up the abstraction stack. Module 15 is your supervised practice run before that becomes your job.

## 2. Branch

```bash
git checkout main && git pull && git checkout -b feature/m15-agent
```

This branch accumulates the agent's full pipeline. It will be larger and messier than Modules 9–14's branches. That's expected. The PR review at the end is correspondingly more important.

## 3. Pick the question

For the deliverable to be evaluable, the question has to be sharp. Examples that fit the Part 2 dataset:

> *Across S&P 500 firms 2010–2024, do firms whose MD&A discusses generative AI show different next-year revenue growth than firms that don't, controlling for size, industry, and prior-year fundamentals?*

> *Build a predictor of one-year-ahead negative earnings surprise (next-year EPS < analyst consensus) using fundamentals + MD&A features. Report calibration and a small case study of three firms the model flagged for FY2024.*

> *Pull every mention of "supply chain" from MD&A 2019–2023; quantify how the language shifted post-COVID; report which industries had the largest shifts.*

A vague question — *"analyze the relationship between R&D and performance"* — produces a vague pipeline. Spend twenty minutes sharpening the question before you write the brief.

## 4. Configure the agent's permissions

Whichever agent you're using, before you start the run:

| Concern | What to set |
|---|---|
| **Working directory** | The agent should not be able to read or write outside `~/Projects/accy575/ACCY575-wrds-data-analysis/`. Configure this in the tool's settings (e.g., Claude Code project settings / `cwd` restriction). |
| **Shell commands** | Approve each one the first time, or use an allowlist. Do **not** enable "auto-approve all shells" on a fresh agent. |
| **Network** | Allow `pypi`, `huggingface`, your LLM provider. Block everything else by default. You're not running this in a vacuum, but you also don't want the agent `curl`-ing arbitrary URLs. |
| **Secrets** | `.env` and `~/.pgpass` should be readable by the agent if it needs the WRDS connection or the API key. They should **never** be writable, copied, or echoed in commit messages. Watch for this in the diff. |

Open the agent in your project. Confirm it reports the working directory and the permission mode you expect.

*Optional further reading: [Claude Code documentation](https://docs.claude.com/en/docs/claude-code) — the "permission modes" and "sub-agents" pages cover exactly the settings you're configuring here.*

## 5. Write the brief

Before you write it, one paragraph on why this workflow matters in accounting and finance specifically.

All four of the Big Four have publicly announced agentic platforms — Deloitte's Zora, PwC's Agent OS, EY's EY.ai Agentic Platform, KPMG's Workbench — and have publicly described pilots in routine substantive testing (confirmations, recalculations, sample selection on transaction-level data). Investment banking groups have announced agentic tooling for first-pass model build-outs and pitch-book drafting. Corporate FP&A teams use them for variance-analysis-and-explanation. Forensic and dispute-consulting practices use them to triage document productions. The role emerging at the entry-and-mid level isn't "the person who writes the SQL." It's "the person who knows what to ask for, reviews the output, and is willing to sign their name to it." The brief you're about to write is the central artifact of your analytical judgment in that role: what question you chose to ask, what method you chose to bound, what constraints you imposed, what "done" means to you. The agent's code is replaceable. The brief isn't.

The demo runs the entire workflow end-to-end. Pick a sharp accounting question the Module 8 dataset can actually answer. Configure the agent's permissions so a runaway can't do real damage. Write the brief on one screen. Watch the agent execute it — you will intervene; everyone intervenes. Verify the result on a fresh clone of the repo. Read the resulting `results.md` as the senior who is signing it. The brief itself lives at `docs/m15-brief.md`. Everything else in this module is downstream of that one file.

This is the exam. The brief should fit on one screen. It should specify:

```
## Task
[One paragraph: what you want, who it's for, what "done" looks like.]

## Data
- `data/raw/fundamentals.parquet` — Compustat fundamentals 2010–2024 (M8).
- `data/raw/mdna.parquet` — MD&A text, joined on (gvkey, fyear).
- Validation: `src/schema.py`. Run it as the first step of your pipeline.

## Method
[Either: pick a method you specify, e.g., "OLS with industry FE per Module 9 spec, plus
 a robustness check using GBM per Module 10."  Or: "choose appropriately and justify in
 the write-up — but justify before fitting, not after."]

## Deliverables
- `src/run_analysis.py` — one entry point that runs the pipeline end-to-end on a fresh
  clone given access to the WRDS data files.
- `notebooks/m15-final.ipynb` — exploratory companion. Optional.
- `results/m15/results.md` — the write-up: question, approach, headline result, three
  caveats. One page max. Written for a manager, not a grader.
- `tests/test_pipeline.py` — at minimum, a test that the entry point runs end-to-end on
  a synthetic 50-row fixture without errors.

## Constraints
- Do not modify Modules 8–14 code. Import from there if needed.
- Do not commit data files. Do not commit `.env` or `.pgpass`.
- Do not auto-merge or auto-push. Stop and surface a PR description for me to review.
- If you find evidence of leakage or a fundamental sample-selection problem, stop and
  write up the problem rather than working around it.
- Cost ceiling: $20 in API calls. If you'd exceed it, stop and tell me.

## What "done" looks like
- `uv run python -m src.run_analysis` produces `results/m15/results.md` from scratch.
- `uv run pytest` is green.
- The PR description names the headline answer in one sentence.
```

The constraints section is doing most of the work. An agent left without "do not auto-push" will sometimes auto-push. An agent left without a cost ceiling will sometimes burn \$200 fine-tuning a model the brief never asked for. These are the sharp edges of agent-driven workflows; the brief is where you blunt them.

Save the brief itself to `docs/m15-brief.md` and commit it on the branch. It is the central artifact of this module.

## 6. Run, supervise, intervene

Hand the brief to the agent. Watch every action.

What supervision actually means: read each shell command before approving it. `pip install xgboost` is fine. `curl` to an unfamiliar URL is not. `pip install something-with-a-typo` is not. `git push` before you've reviewed the PR is not. Watch the file edits. Most agents show you a diff before writing; read it. If the agent is rewriting a Module 8–14 file when the brief said don't, stop it.

*Optional further reading: [Module 6's prompt-injection section](../module-06-ai-assisted-coding-and-debugging/walkthrough.md#6-recognize-prompt-injection) — more relevant here, not less: an agent with shell access plus untrusted text (MD&A, scraped filings) in context is the canonical injection risk this supervision step defends against.*

Stop when something feels wrong. "Feels wrong" is a real signal. Usually it means the agent's plan diverged from the brief and is heading somewhere expensive. Fixing it cheaply now beats reverting later.

Expect the agent to take 15–60 minutes to produce a full pipeline. Expect to intervene at least three times. If you don't intervene at all, either you wrote a remarkably good brief or you weren't paying attention. Honestly assess which.

## 7. Verify on a fresh clone

When the agent says it's done, do not believe it. Verify:

```bash
cd /tmp
git clone https://github.com/<your-username>/ACCY575-wrds-data-analysis.git verify-m15
cd verify-m15
git checkout feature/m15-agent
uv sync
# you'll need WRDS credentials and an API key for the run
uv run python -m src.run_analysis
uv run pytest
```

If it doesn't run cleanly on the fresh clone, the agent shipped something that depends on your local state. Half-cached intermediate files, a hardcoded path, a forgotten `uv add`. Fix it on the branch. This is the most common place agentic pipelines fail; budget time for it.

## 8. Read the results.md as a manager would

Print it. Read it on paper if you can. You're playing the role of a senior who is about to sign their name to this work.

Does the headline answer the question that was asked? Are the caveats honest, or are they minimizing real problems? Could a non-coder reader follow the logic? Is there a number anywhere you can't trace back to the data?

If any of those are off, send the agent back with a follow-up brief that names the specific fix. Don't rewrite the file by hand if you can help it; the directed-revision loop is part of the exercise.

## 9. PR, full self-review, merge

```bash
git push -u origin feature/m15-agent
gh pr create --title "M15: End-to-end agent pipeline" --body "$(cat <<'EOF'
## Question
[one sentence]

## Headline answer
[one sentence]

## Method
[two sentences — what method, what data, why this method]

## How to reproduce
1. WRDS credentials in `.env` (or `~/.pgpass`).
2. ANTHROPIC_API_KEY in `.env`.
3. `uv sync && uv run python -m src.run_analysis`.

## Caveats
[three bullets]

## How the agent was directed
- Brief: `docs/m15-brief.md`.
- Where I stopped or redirected the agent during the run: [bullet list].
EOF
)"
```

Self-review with extra scrutiny here — this PR is more code than any single Module 9–13 PR. Does the diff include any file the brief said don't touch? Is `tests/test_pipeline.py` real? (Run it once, then change one line in `src/run_analysis.py` and confirm it fails. Then revert.) Are there committed credentials, API keys, or data files? `git diff main..HEAD --name-only | xargs -I {} grep -l -E "ANTHROPIC|WRDS_|sk-" {}` is a quick check. Does `results.md` make claims the cited numbers actually support?

Fix anything off. When clean, squash-merge into `main`.

## You're done if…

- [ ] `feature/m15-agent` is merged into `main`.
- [ ] `uv run python -m src.run_analysis` runs end-to-end on a fresh clone.
- [ ] `results/m15/results.md` exists, fits on one page, and answers the question.
- [ ] You intervened at least once during the agent run and noted the intervention in the PR description.
- [ ] The PR description, the brief in `docs/m15-brief.md`, and `results/m15/results.md` together let a senior trust this work without re-running it themselves.
- [ ] No credentials, API keys, or raw WRDS data anywhere in `git log`.

This module is the Part 2 capstone. If the rest of the semester evaporated and only your `ACCY575-wrds-data-analysis` repo survived, the grader should be able to pick it up, read `results/m15/results.md`, and tell what kind of analyst you are.
