# Module 4 — Project Structure and Secrets

> **Hands-on:** [walkthrough.md](walkthrough.md) — moving your project into a real layout, handling `.env` secrets, and writing a README that doesn't waste a reader's time.

## Motivation

Two modules of pushing to GitHub means anyone can find your repo now. Worth thinking about what they see when they get there.

A repo that's just one `script.py` and an empty README looks like coursework — because it is. A repo with a sensible folder layout, a README that tells you what the thing does, and obvious care about not committing secrets looks like someone who's been writing software for a while. Most of the difference is cosmetic, and cosmetic stuff matters when someone is deciding whether to interview you.

The other thing this module covers is API keys. The cost of getting this wrong is real: bots scrape GitHub for committed keys within minutes of the push, and you get the bill.

By the end of this module you will (1) understand the layout of a real Python project and (2) have an `.env`-based workflow for handling secrets.

## Goals

- Understand `pyproject.toml` and what makes a project reproducible by someone who is not you.
- Manage secrets safely with `.env` and `.gitignore`. Never commit API keys.
- Write a `README.md` that lets a stranger set up and run your project without asking you.

## Going Deeper *(optional)*

*The lecture covers what you need. These are here if you want to dig further.*

- [Packaging Python Projects](https://packaging.python.org/en/latest/tutorials/packaging-projects/) — PyPA. Reference for `pyproject.toml` and project layout when you want to publish or share a package.
- [The Twelve-Factor App — III. Config](https://12factor.net/config) — Adam Wiggins. The classic short argument for keeping config and secrets out of code.

## Materials

*Slides, in-class exercises, and recordings posted here as the module approaches.*
