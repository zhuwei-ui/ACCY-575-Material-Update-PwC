# Module 6 — AI-Assisted Coding and Debugging

> **Hands-on:** [walkthrough.md](walkthrough.md) — setting up GitHub Copilot, writing your first brief, debugging AI-produced code, and recognizing prompt injection.

## Motivation

AI coding assistants are now table stakes for an entry-level analytics job. Big Four interns are expected to use them on Day 1. Whoever you're competing with for the same offer already knows how.

What "use them well" means is what this module is about. It isn't "paste the assignment into ChatGPT." It's closer to managing a junior staff member who happens to be extremely fast and occasionally makes things up. You give them a clear task; they draft; you read it carefully; you fix what's wrong; you sign your name on the result. The work is still yours. The skill is no longer writing every line, it's directing and reviewing.

## Goals

- Write a clear *brief* (spec) before asking an AI to write code: what you want, what data shape, what constraints.
- Use GitHub Copilot (or another AI coding assistant) to draft, refactor, and debug code inside VS Code.
- Read AI-produced code critically: catch hallucinated APIs, wrong assumptions, and plausible-but-wrong logic.
- Use the VS Code debugger when print statements aren't enough.
- Understand the permission level your AI assistant has on your machine, and why it matters.
- Recognize prompt injection — the threat you'll face in Part 2 every time you paste client data, transcripts, or scraped content into an LLM.

## Going Deeper *(optional)*

*The lecture covers what you need. These are here if you want to dig further.*

- [GitHub Copilot — Documentation](https://docs.github.com/en/copilot) — GitHub. The "best practices" subsections are the most useful.
- [Debugging in VS Code](https://code.visualstudio.com/docs/editor/debugging) — Microsoft. Reference for the "Debugger user interface" and "Breakpoints" features.
- [OWASP Top 10 for LLM Applications](https://genai.owasp.org/) — OWASP. **LLM01: Prompt Injection** is the relevant one for accountants. The others are more developer-focused.
- [Simon Willison — *Prompt injection* (series)](https://simonwillison.net/tags/prompt-injection/) — Continuously updated archive on this threat as it evolves in the wild.

## Materials

*Slides, in-class exercises, and recordings posted here as the module approaches.*
