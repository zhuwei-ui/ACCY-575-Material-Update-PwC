# Module 8 — WRDS Data Acquisition

> **Hands-on:** [walkthrough.md](walkthrough.md) — setting up a new Part 2 repo, enrolling in the WRDS class, giving your agent a key-based connection into the WRDS Cloud, and running an agent-directed two-source pull.

> **Windows students:** every local command in this module runs in the Ubuntu shell inside WSL, set up in [Module 1](../module-01-terminal-vscode-and-your-first-uv-project/walkthrough.md). Two things §4 and §5 depend on — SSH connection multiplexing and `rsync` — have no working equivalent on native Windows. If WSL isn't installed yet, do that before class.

## Motivation

Real accounting and finance research runs on Wharton Research Data Services (WRDS): Compustat, CRSP, IBES, Audit Analytics, the SEC textual datasets. Industry analytics teams and PhD programs alike pull from there. If you want to do their work, you have to know how to pull from there too.

This is the only logistical module in Part 2. The six modules that follow (M9–14) all use the panel you build here — Compustat firm-year **fundamentals** joined with **10-K MD&A text** on `(gvkey, fyear)` — with each method getting its own branch and review pass. Get this module right once and the rest of Part 2 is about methods and judgment, not data wrangling.

## Goals

- Stand up a new GitHub repository for Part 2, separate from your Part 1 work.
- Enroll in the ACCY 575 WRDS Education Classroom account and complete Duo two-factor setup.
- Install an SSH key on the WRDS Cloud and set up a multiplexed connection your agent can run commands through, without moving your editor onto a process-capped shared server.
- Configure local Python access to WRDS via `~/.pgpass`.
- Brief an agent to write two pulls — Compustat fundamentals locally, 10-K MD&A text on the cloud — and validate both with `pandera`.
- Submit the heavy 10-K parsing to the WRDS compute grid as a Grid Engine array job, monitor it with `qstat`/`qacct`, and `rsync` the result back.
- Place code and data on the right filesystems: a 10 GB backed-up home for source, a shared 500 GB scratch space for outputs.
- Keep one authoritative Git working tree on your laptop, syncing code out and results back rather than committing from two machines.
- Apply the academic / non-commercial use rules: code can be public, raw vendor data cannot.

## Going Deeper *(optional)*

*The lecture covers what you need. These are here if you want to dig further.*

- [Student Guide: Enrolling in a Class Account](https://wrds-www.wharton.upenn.edu/pages/about/wrds-account-types/student-guide-enrolling-in-a-class-account/) — WRDS. Canonical source for §2's enrollment flow.
- [Using Python on the WRDS Platform](https://wrds-www.wharton.upenn.edu/pages/grid-items/using-python-wrds-platform/) — WRDS. Reference for the `wrds` client and `.pgpass` setup.
- [`ssh_config(5)`](https://man.openbsd.org/ssh_config) — OpenBSD. The authoritative reference for the `ControlMaster` / `ControlPath` / `ControlPersist` options behind §4's multiplexed connection.
- [`rsync(1)`](https://download.samba.org/pub/rsync/rsync.1) — the full flag set behind §5's sync-to-execute loop, if you want more than `-avz --delete`.
- [`qsub(1)`](https://gridscheduler.sourceforge.net/htmlman/htmlman1/qsub.html) — Grid Engine. Submission options including the `-t` array-job flag used in §8b; [`qstat`](https://gridscheduler.sourceforge.net/htmlman/htmlman1/qstat.html) and [`qacct`](https://gridscheduler.sourceforge.net/htmlman/htmlman1/qacct.html) are the monitoring counterparts.
- [`sec-parser`](https://pypi.org/project/sec-parser/) — Alphanome.AI. The 10-K-aware parser used for Item 7 extraction in §8b.

## Materials

*Slides, in-class exercises, and recordings posted here as the module approaches.*
