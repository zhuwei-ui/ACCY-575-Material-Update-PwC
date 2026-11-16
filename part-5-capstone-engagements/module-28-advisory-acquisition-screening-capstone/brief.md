# Module 28 — Engagement Brief: Acquisition Screening & Diligence (Capstone)

*Your team is handed a two-sentence mandate and access to the data. The universe, the screen, the diligence, the recommendation — everything is yours to scope and defend. How you approach it is entirely up to the team; the deliverable, and how it holds up in the room, are what count.*

## Your role

Your **advisory / transaction-services team** has been engaged by a strategic acquirer (or a private-equity sponsor). The client's investment committee will hear your recommendation in a **live session** and will push back on every choice you made.

## The task

> We want to deploy capital by acquiring a public company. Screen the universe, bring us a defensible shortlist with a first read on earnings quality and working-capital health, and tell us who to meet and who to pass.

Everything the mandate does not specify — which companies are even in scope, what makes a target attractive, what "quality" means, where to draw the line — is yours to define and to justify.

## The data

- The **Part 4 Compustat panel** you built in **[Module 17](../../part-4-agentic-ai-methods-and-pipelines/module-17-wrds-data-acquisition/walkthrough.md)** (S&P 500 firm-years). Reuse it, and pull any additional fundamentals you need — working-capital and cash-flow lines such as `rect`, `invt`, `ap`, `act`, `lct`, `capx`, `dlc`, `dltt` — from WRDS the way Module 17 did. Same subscription, no new entitlement.
- **EDGAR filing signals** (the [Module 8](../../part-2-ai-enabled-archival-data-and-replication/module-08-edgar-data-acquisition/walkthrough.md) approach: the `submissions` data's `form` and `items` fields, §5 and §7b) — things a diligence team watches for, like late-filing notices (NT 10-K), non-reliance/restatement disclosures (8-K Item 4.02), auditor changes (8-K Item 4.01), and, from the 10-K text itself (Module 8 §8b), internal-control weaknesses (Item 9A).

The client did not hand you a target list. Defining the universe is part of the job.

## What to hand in

- **An investment-committee report and deck** — the scoped universe and screen (every criterion justified), the earnings-quality and working-capital read on the finalists, the filing red flags, and a **meet-or-pass recommendation per name** with its rationale and its caveats.
- **A live defense** — an IC-style Q&A in which the committee interrogates your choices. Every member should be able to defend the whole recommendation, not just their own part of it.
- **The team repository** — reproducible in the Part 1 way, built the collaborative way (branches and pull requests, the [Module 3](../../part-1-ai-assisted-reproducible-workflows/module-03-pull-requests-and-code-review/walkthrough.md) workflow) so contributions are visible and the analysis re-runs.

## Ground rules

- **Teams of 3–4** where possible (ask the instructor); solo by arrangement.
- Use AI as much as you want — the team owns every number in the report regardless.
- Anything you rely on from outside the data (a screening rule, a diligence metric, a definition) you look up and cite yourselves.
- No prescribed method, no required tools, no checklist. Scope it, build it, defend it.

