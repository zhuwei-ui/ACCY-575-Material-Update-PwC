# Module 16 — Where This Is Heading: Autonomous Research Agents

> **Hands-on:** [walkthrough.md](walkthrough.md) — a seminar-style module: read the literature on AI agents conducting end-to-end research, run a Deep Research tool against a Part 2 question, and compare its output to what you produced in Module 15.

## Motivation

Module 15 had you direct an agent through one analysis you specified: you wrote the question, you picked the method, you bounded the constraints, and the agent executed. The frontier of agentic AI is the level above that — *the agent decides the question, picks the method, runs the analysis, and writes the report,* with the human reviewing the output rather than directing the process. By the time you graduate, you will be reviewing this kind of output.

The lab evidence is already in. Sakana's "AI Scientist" produces full machine-learning research papers end-to-end, one of which passed peer review at the ICLR 2025 "I Can't Believe It's Not Better" workshop. James Zou's group at Stanford ran a *Virtual Lab* of agentic "scientists" that designed nanobody binders to SARS-CoV-2 spike protein variants and saw two of them validated in wet-lab experiments (Swanson, Wu, Bulaong, Pak, Zou, *Nature* 2025). Boiko, MacKnight, Kline, and Gomes (*Nature* 2023) built "Coscientist," an autonomous chemistry agent that planned and ran its own catalysis experiments. The Deep Research products from OpenAI, Anthropic, and Google all ship a consumer-grade version of the same loop.

Accounting will get this slower than chemistry or biology — fewer wet-lab cycles to optimize, more interpretive judgment, and the disclosure-regulation environment is unforgiving of black-box claims. But it will get it. This module is the preview.

## Goals

- Understand what separates an "autonomous research agent" from the supervised pipeline you ran in Module 15.
- Read three to five papers from the active autoresearch literature and identify the common architectural patterns (planner, executor, critic, memory, tool-use).
- Run a commercial Deep Research product on a Part 2 question, compare its output to your Module 15 deliverable, and write up the comparison.
- Form an opinion on which parts of accounting and finance research are realistically automatable in the next two to five years, and which are not.

## Going Deeper *(required readings — pick three)*

The lecture covers all of these at a high level. Pick three to read end-to-end before class.

- **Lead reading.** *[The flagged SocArXiv preprint](https://osf.io/preprints/socarxiv/24xfq_v1)* — the discussion paper assigned by the instructor on AI agents conducting social-science research. Read first; the rest of the list is response and context.
- **Swanson, K., Wu, W., Bulaong, N. L., Pak, J. E., & Zou, J. (2025).** "The Virtual Lab of AI agents designs new SARS-CoV-2 nanobodies." *Nature*. The Stanford team's full agent-team-of-scientists system, with wet-lab validation. Most concrete demonstration to date that autonomous agent loops produce real results in a hard scientific domain.
- **Lu, C., Lu, C., Lange, R. T., Foerster, J., Clune, J., & Ha, D. (2024).** "The AI Scientist: Towards Fully Automated Open-Ended Scientific Discovery." *arXiv:2408.06292*. Sakana AI / Oxford / UBC. End-to-end ML research paper generation. (Its successor, *AI Scientist-v2* — arXiv:2504.08066, 2025 — produced the first AI-written paper to pass workshop peer review, at the ICLR 2025 ICBINB workshop.)
- **Boiko, D. A., MacKnight, R., Kline, B., & Gomes, G. (2023).** "Autonomous chemical research with large language models." *Nature*. The "Coscientist" system. Foundational paper for the autoresearch architecture you'll see in everything since.
- **Schmidgall, S., et al. (2025).** "Agent Laboratory: Using LLM Agents as Research Assistants." *arXiv*. Open-source reference implementation of the autoresearch architecture; useful if you want to read code, not just papers.

If you want a more critical view, read **Beel et al. (2025), "Evaluating Sakana's AI Scientist: Bold Claims, Mixed Results, and a Promising Future?"** (arXiv:2502.14297) — they argue the headline claims overstate what the system actually produces. Useful counterweight before you get carried away.

## Materials

*Slides, in-class discussion guide, and the post-discussion reflection prompt posted here as the module approaches.*
