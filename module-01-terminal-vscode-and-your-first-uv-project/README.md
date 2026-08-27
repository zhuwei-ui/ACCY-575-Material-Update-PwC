# Module 1 — Terminal, VS Code, and Your First `uv` Project

> **Hands-on:** [walkthrough.md](walkthrough.md) — setting up your terminal, VS Code, and your first `uv`-managed Python project.

> **Windows students:** the walkthrough starts by having you install **WSL** (the Windows Subsystem for Linux) and work at an Ubuntu prompt for the rest of the course. It's a one-time install that needs a reboot, so start it before you need it — and tell us early if your laptop won't allow it. The reason is Module 8: connecting a coding agent to the WRDS Cloud requires SSH connection multiplexing and `rsync`, and native Windows can do neither.

## Motivation

Most data classes start you in Jupyter, writing pandas in cells. That's fine for learning syntax, but it's not how anyone you'll work for actually expects you to deliver work. Your manager isn't going to clone `Untitled-7.ipynb` off your laptop.

This module is the boring stuff nobody bothers to teach but every job assumes you know: how to use a terminal, how to set up a real Python project, what an editor that isn't a notebook is for.

If you do everything in this module's [walkthrough](walkthrough.md), you'll end the module with a working development environment, a project managed by `uv`, and the ability to run and step through Python in a debugger. Sounds basic. It isn't — plenty of CS majors at UIUC can't do all of that cleanly even by senior year. You'll have it down by the end of this module.

## Goals

- Navigate the file system from the command line (`cd`, `ls`, paths, environment variables).
- Read a stack trace and identify where a program failed.
- Install VS Code with the Python extension and run code from inside it.
- Create your first project managed by `uv` (`uv init`, `uv add`, `uv run`).

## Going Deeper *(optional)*

*The lecture covers what you need. These are here if you want to dig further.*

- [The Missing Semester of Your CS Education — "The Shell"](https://missing.csail.mit.edu/2020/course-shell/) — MIT. A 30-minute primer on the terminal that's worth its weight in gold for anyone who'll spend a career in a CLI.
- [uv — Getting Started](https://docs.astral.sh/uv/getting-started/) — Astral. Official docs. Useful as a reference once you start using `uv` daily.
- [*uv: An Extremely Fast Python Package Manager*](https://youtu.be/gSKTfG1GXYQ) — Charlie Marsh, creator of `uv` (Astral). Talk-format introduction from the person who built it.
- [VS Code — Python Tutorial](https://code.visualstudio.com/docs/python/python-tutorial) — Microsoft. Goes deeper into IntelliSense, the debugger, and Python-specific features.

## Materials

*Slides, in-class exercises, and recordings posted here as the module approaches.*
