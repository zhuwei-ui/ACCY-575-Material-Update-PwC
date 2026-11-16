# Module 24 — Agent End-to-End Pipeline *(optional)*

> **Optional — Part 4 (Agentic AI Methods and Pipelines).** Not required for the course. Part 4 is a self-paced extension: it builds its own WRDS panel in [Module 17](../module-17-wrds-data-acquisition/README.md) and applies one modern method per module to it. If you take it, do it alongside Part 3, not after it: it runs on the course's WRDS class account, which expires when the term ends, so finish Module 17's pulls, and any other work that needs WRDS itself, before then.

> **Hands-on:** [walkthrough.md](walkthrough.md) — turning Modules 17–23 into a single agent-driven pipeline you direct rather than write.

## Motivation

In Modules 17–23 you used a coding agent to draft code one piece at a time. The job for Module 24 is the next step: hand the agent the *whole task* — pull the data, choose the method, fit it, evaluate it, produce the write-up — and have it run the pipeline end-to-end while you supervise.

Senior analysts at most consulting and audit shops are moving from "I write the code with AI help" to "the agent writes and runs the code; I write the brief and review the output." The skill that makes this work — and the skill that's evaluated here — is *brief writing and review*, not Python.

This module is also a useful stress test of your own judgment. An agent given enough autonomy will produce confident, plausible, complete-looking output that is wrong in subtle ways. Catching that, on a real dataset, is the actual exam.

## Goals

- Write an end-to-end brief specifying the question, data, method, success criteria, and deliverable shape.
- Watch the agent work — read every shell command before approving and stop it when it goes off the rails.
- Evaluate the result the way you'd evaluate a junior staff member's deliverable.
- Produce a final-week deliverable that is clearly yours (the brief, the review, the corrections) even though the agent wrote the code.

## Going Deeper *(optional)*

*The walkthrough covers what you need. These are here if you want to dig further.*

- [Claude Code documentation](https://docs.claude.com/en/docs/claude-code) — Anthropic. The "permission modes" and "sub-agents" pages are the most relevant.
- [Agents SDK](https://platform.openai.com/docs/guides/agents) — OpenAI. Alternative tooling for the same agent-loop pattern.
- Module 6's prompt-injection content is *more* relevant here, not less — an agent with shell access plus untrusted text in context is the canonical risk.

## Materials

*Slides and recordings from earlier offerings, if any, are posted here.*
