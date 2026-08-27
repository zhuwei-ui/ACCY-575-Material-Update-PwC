# Module 16 — Walkthrough: Autonomous Research Agents

## 1. What changes one more level up

Modules 9 through 14 used an agent to write one piece of code at a time. Module 15 was the level above: you wrote a brief and the agent ran the full pipeline end-to-end. Module 16 is the level above that. The agent picks the question, picks the method, runs the analysis, writes the report, and (in the most aggressive systems) reviews its own work. Your role collapses from "specify the analysis" to "review the final deliverable as the senior signing their name to it."

This isn't speculative. Sakana AI's *AI Scientist* (Lu et al., 2024, arXiv:2408.06292) writes complete machine-learning research papers in LaTeX; its successor, *AI Scientist-v2* (Sakana AI, 2025, arXiv:2504.08066), produced a paper that passed peer review at an ICLR 2025 workshop. James Zou's Stanford group, with co-author John Pak at the Chan Zuckerberg Biohub SF, built *Virtual Lab* (Swanson, Wu, Bulaong, Pak, Zou, *Nature* 2025), a system of agentic "scientists" that designed SARS-CoV-2 nanobody binders; two of the designs showed measurable binding to JN.1 / KP.3 variants in subsequent wet-lab validation. Boiko, MacKnight, Kline, and Gomes (*Nature* 2023) ran *Coscientist*, an autonomous chemistry agent that planned and executed its own catalysis reactions. OpenAI's Deep Research, Anthropic's Claude Research, and Google's Deep Research ship commercial-grade versions of the same architecture to consumers.

What all these systems share is a small set of architectural pieces. A *planner* decomposes a goal into sub-questions. An *executor* runs tools — code, search, retrieval, sometimes physical instruments. A *critic* reads the executor's output and asks whether it answers the planner's sub-question, often by running a second LLM call with adversarial prompting. A *memory* persists across iterations so the agent doesn't redo work it already did. Wrapping it all is a *loop* that runs until either a stopping criterion is met or the agent runs out of budget. The same pattern, with different terminology, appears in every system in the reading list. Recognizing the pattern is the goal of this module's lecture.

Accounting will see this slower than biology and chemistry. The reasons are interpretive: a wet-lab cycle gives the agent unambiguous feedback (the protein bound or it didn't), while an accounting research question almost never has that kind of clean ground truth. The disclosure-regulation environment is also unforgiving — an audit conclusion has to be defensible to the PCAOB, and an SEC filing review has to be defensible in court, neither of which tolerates an unexplained black-box claim. But the trajectory is one direction. Routine financial-statement analysis, ratio-style anomaly detection, and first-pass disclosure review are already being automated, and the wrapper that makes the automation *autonomous* rather than supervised is the same wrapper you're reading about in this module.

## 2. Branch

```bash
git checkout main && git pull && git checkout -b feature/m16-autoresearch
```

This module's deliverable is a reflection, not code. The branch is light.

## 3. Read three of the papers

Pick three from the [Going Deeper](README.md#going-deeper-required-readings--pick-three) list. The flagged SocArXiv preprint is required; pick two others. Take notes as you read on the following:

- What is the system's *planner-executor-critic-memory* decomposition? Where is each component implemented?
- What's the human in the loop for? At which decisions does the human still appear, and which decisions are fully automated?
- What does the paper measure to claim success, and how would you criticize that measurement?
- What would have to change for this system to work on an accounting research question rather than the domain the paper studied?

You'll need those notes for §6.

## 4. Pick a Part 2 question and run Deep Research on it

Pick one of the questions you (or a classmate) tackled in Module 15. Reuse the same wording.

Open a commercial Deep Research product. Three reasonable choices: Anthropic's [Claude Research](https://claude.com/blog/research), OpenAI's [Deep Research](https://openai.com/index/introducing-deep-research/), or Google's [Deep Research in Gemini](https://gemini.google/overview/deep-research/). The exact product matters less than the comparison; pick whichever you have access to.

Hand the question to the tool. Let it run. Read the deliverable when it finishes.

Save the full output (including the citation list) to `results/m16/deep_research_run.md`. Note which product you used and the date of the run; these systems are improving fast and a result from May won't reproduce in August.

## 5. Compare to your Module 15 deliverable

Open the `results/m15/results.md` from your Module 15 PR side-by-side with the Deep Research output you just generated. Compare on the following dimensions:

| Dimension | What to look at |
|---|---|
| **Question fidelity** | Did the Deep Research product answer the question you asked, or a different question it found easier? |
| **Data sources** | Does the Deep Research output cite the WRDS Compustat data you used in M15? Or did it work from secondary sources (news articles, summaries, blog posts) it found by searching? |
| **Method transparency** | Can a reader trace each headline number back to a specific data source and operation? Or are the numbers free-floating with general citations? |
| **Caveats / honesty** | Does the Deep Research output acknowledge limitations of its own approach? How does that compare to the caveats in your M15 write-up? |
| **Time** | How long did each take to produce? Deep Research is usually fast on questions answerable from the web and slow-or-failing on questions that require the underlying dataset. |

Write the comparison up in `results/m16/comparison.md`. One page, five paragraphs — one per dimension. Be specific: quote text from both deliverables, name the failures concretely.

The point of this exercise isn't to declare a winner. The current commercial Deep Research products are good at literature surveys, news summaries, and questions where the answer exists somewhere online. They're weak at — sometimes incapable of — questions that require running an analysis on a dataset they don't have access to. Module 15's pipeline is the opposite. Knowing where each tool sits on that axis is the skill.

## 6. Write the reflection

The reflection is the deliverable. One-page markdown at `results/m16/reflection.md`. Address each of the following directly:

1. **Architecture you saw across the readings.** Name the planner / executor / critic / memory decomposition from §1 and point to where each appears in the systems you read about. One paragraph.
2. **Where Deep Research outperformed your M15 pipeline.** Where it underperformed. One paragraph each.
3. **Which parts of accounting research are realistically automatable in the next two to five years, and which are not.** Be specific. "Audit confirmations" is too vague — name a specific audit procedure and explain what an autoresearch system would need to do it and what it currently can't. One paragraph.
4. **Your own forward-looking question.** What would you have to learn between graduation and your two-year-out career checkpoint to remain a useful supervisor of these systems? One paragraph.

Three pages of class discussion compress into one page of writing. Write tight.

## 7. PR and merge

```bash
git push -u origin feature/m16-autoresearch
gh pr create --title "M16: Autonomous research agents — readings, Deep Research run, reflection" --body "$(cat <<'EOF'
## Summary
- Read three papers from the autoresearch reading list (one flagged + two of own choice).
- Ran [Claude / OpenAI / Google] Deep Research on Module 15's question; output committed to `results/m16/deep_research_run.md`.
- Five-dimension side-by-side comparison committed to `results/m16/comparison.md`.
- Reflection committed to `results/m16/reflection.md`.

## Headline takeaway
[one sentence — your single most-important conclusion about where autoresearch lands relative to your own pipeline]

## Reading list
- [paper 1]
- [paper 2]
- [paper 3]
EOF
)"
```

Self-review: every file under `results/m16/` is committed and renders cleanly on GitHub; the reflection answers all four prompts in §6 rather than the easy three; the Deep Research output includes the run date and product name so a grader can interpret it; you cited at least three readings in the reflection, not just one.

Squash-merge.

## You're done if…

- [ ] `feature/m16-autoresearch` is merged into `main`.
- [ ] Three papers read end-to-end, including the flagged SocArXiv preprint.
- [ ] Deep Research run on a Module 15 question, output committed with date and product name.
- [ ] One-page comparison and one-page reflection committed.
- [ ] You can answer, in one sentence each: what the planner / executor / critic / memory pattern is, where commercial Deep Research is weak, and what part of accounting work you think will be autonomously automated first.

This module is the course's forward look. The point isn't to make you an expert on autoresearch — the field will look completely different by the time you graduate. The point is that you know what to read, what to compare against, and how to position yourself as the person who reviews the output rather than the person whose job the output replaces.
