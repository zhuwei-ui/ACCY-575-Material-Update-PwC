# Module 3 — Walkthrough: Pull Requests and Code Review

## Before you begin: pair up with a classmate

Find a classmate to pair with for this module. Both of you:

1. Make your `ACCY575-walkthrough` repository **public** on GitHub: *Settings → scroll to the **Danger Zone** → Change visibility*.
2. Share the repository URL with each other.

You'll review each other's pull requests in Step 4. Pair review is the closest practical preview of what code review at a real job looks like.

> **The flow at a glance:** Steps 1–3 happen on **your own repo** (open a PR, self-review it). Step 4 happens on **your classmate's repo** (you review theirs while they review yours, in parallel). Step 5 is back on **your own repo** to merge.

## 1. Open a pull request on your own repo

A **pull request (PR)** is a proposal to merge one branch into another (usually a feature branch into `main`), hosted on GitHub. It's the *review step* between "I made changes" and "they land in main."

In Module 2 you merged a branch directly with `git merge`. That's fine on a personal project. In any team — or any repo someone else cares about — you don't merge directly. You open a PR, someone reviews it, *then* it gets merged.

**Why PRs exist:**

- **Code review.** Another set of eyes catches bugs, suggests improvements, and asks questions before the change ships.
- **Discussion.** A thread for "why did you do it this way?" lives next to the diff and stays there as a record.
- **Automated checks.** Tests, linters, and CI run on the PR before merge — you see results before code lands in `main`.
- **A record.** A year later, the PR description and comments tell you why a change was made.

For accounting students: think of it as the "send a draft adjusting entry to your senior for review before posting" model — translated to code. Your manager will not trust code that hasn't been reviewed.

```bash
cd ~/Projects/accy575/ACCY575-walkthrough
git checkout -b feature/add-summary
```

Edit `main.py` to add a small new function — a `summary_stats(df)` that returns mean and median per numeric column, say.

```bash
git add main.py
git commit -m "add summary_stats helper"
git push -u origin feature/add-summary
```

The CLI output prints a link like `Create a pull request for 'feature/add-summary' on GitHub by visiting: ...`. Open it.

## 2. Write a PR description that does not waste your reviewer's time

A throwaway "added stuff" description teaches you nothing and signals nothing. A real description says:

> **What:** add `summary_stats(df)` returning per-column mean and median.
>
> **Why:** I'll need this for the upcoming validation step in Module 5; pulling it out now keeps the analysis function focused.
>
> **How to test:** `uv run main.py` and confirm the output includes the summary stats. (We'll formalize tests with `pytest` in Module 5.)

Write that. Save the PR.

## 3. Review your own PR

GitHub shows the diff. Click **Files changed** and try to leave at least three comments:

- **One question** on something that is unclear or could be clearer.
- **One suggestion** — use the "suggestion" syntax in the comment box (the `±` icon, or markdown ` ```suggestion ` blocks).
- **One praise** — what did you do *well*? Knowing the good is half the skill.

Yes, you are reviewing your own code. The goal is the muscle memory.

## 4. Review your classmate's PR

> **Switch repos for this step.** Open the URL your classmate shared — you'll be working in **their** GitHub repo, not yours.

Now do the same for your pair partner's PR:

1. Open their PR on GitHub (use the URL they shared).
2. Read the diff carefully — pretend you're the senior who has to vouch for it before it goes anywhere.
3. Leave at least three comments:
   - **One question** on something the description didn't make clear.
   - **One suggestion** — use GitHub's "suggestion" syntax to propose an exact replacement.
   - **One praise** — what did they do well?
4. At the bottom of the **Files changed** view, click *Review changes* → choose *Comment* (or *Approve* if you'd vouch for it as-is) → submit.

*Optional further reading: [Google's Code Review Developer Guide](https://google.github.io/eng-practices/review/) — the industry-standard playbook for both sides of a review; the Reviewer and Author sections are each short.*

Reviewing real human-written code (not your own) is the muscle that lets you evaluate AI-generated code in Part 2. Same skill, different author.

*Optional further reading: ["Experimenting with AI code review"](https://graphite.com/blog/ai-code-review-experiments) (Graphite blog) — how the review feedback loop shifts once an AI is the author.*

## 5. Merge the PR

> **Back to your own repo.** Your classmate's review is done; now you're merging the PR you opened in Step 1.

Once your classmate has reviewed your PR (and you've responded to anything that needed a follow-up), click **Merge pull request → Confirm merge**. Then locally:

```bash
git checkout main
git pull
git branch -d feature/add-summary
```

Your branch is gone locally. On GitHub, click "Delete branch" after the merge to clean up there too. `main` is now up to date.

## 6. Let a robot run your checks (CI)

Step 1 listed "automated checks" as one reason PRs exist. Here's the concrete version. Reviewers shouldn't have to clone your branch and run the tests by hand to know they pass — **continuous integration (CI)** does it for them. With **GitHub Actions**, you commit one small file, `.github/workflows/ci.yml`, that says: *on every push and pull request, install the project with `uv sync` and run `uv run pytest`.* GitHub spins up a fresh machine, runs your checks, and stamps a green check or a red X on the PR before anyone merges — automated enforcement of everything Module 5 will teach, and it catches the "works on my laptop" failures a human reviewer's eyes slide right past.

You don't need to wire this up on the walkthrough repo today, but know it exists — it's standard on every professional repository, and it's the machinery behind the "checks passed" badges you'll see on real PRs.

*Optional further reading: [GitHub Actions — Building and testing Python](https://docs.github.com/en/actions/automating-builds-and-tests/building-and-testing-python) — GitHub's own quickstart for a test-on-push workflow, with a minimal starter you can adapt to `uv`.*

## You're done if…

- [ ] You opened, reviewed, and merged at least one PR on your own repo.
- [ ] Your PR description includes What / Why / How-to-test.
- [ ] You left review comments on a classmate's PR (one question, one suggestion, one praise).
- [ ] You can explain GitHub Flow in one sentence.
