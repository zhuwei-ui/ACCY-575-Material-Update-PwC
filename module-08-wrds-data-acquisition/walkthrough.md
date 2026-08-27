# Module 8 — Walkthrough: WRDS Data Acquisition

## 1. Stand up a new repo for Part 2

Part 1's `ACCY575-walkthrough` was workflow practice on small data. Part 2 is research data, multiple models, and a branch-per-module review workflow. Keeping them in separate repos avoids a tangled history and gives you a Part 2 repo that's clean enough to point a recruiter at.

Pick a name and create the local folder:

```bash
cd ~/Projects/accy575
uv init --no-package ACCY575-wrds-data-analysis
cd ACCY575-wrds-data-analysis
```

`--no-package` for the same reason as Module 1: it gives you the flat layout, so the `src/` tree you build next is yours rather than one `uv` has already scaffolded a package inside.

Mirror the Part 1 layout you set up in Module 4:

```bash
mkdir -p data/raw data/interim src jobs notebooks tests
touch data/raw/.gitkeep src/__init__.py src/pull_fundamentals.py
touch src/build_mdna_manifest.py src/parse_mdna_shard.py src/merge_mdna.py
touch jobs/parse_mdna.sh jobs/merge_mdna.sh
touch .env
```

Update `.gitignore` (this is the same shape as Module 4 but with WRDS-specific lines added — copy the whole block):

```
# Python
__pycache__/
*.pyc
.venv/

# Secrets
.env
.env.*
.pgpass

# Data — WRDS is licensed, do not commit raw pulls (test fixtures under tests/ stay committable)
data/raw/
data/interim/
data/*.csv
data/*.parquet
data/*.xlsx

# OS
.DS_Store
Thumbs.db

# IDE
.vscode/settings.json
.idea/
```

Now create the GitHub repo and push:

```bash
git init
git add .
git commit -m "Initial Part 2 scaffold"
gh repo create ACCY575-wrds-data-analysis --public --source=. --remote=origin --push
```

> **`gh` not installed?** `brew install gh` on macOS, `sudo apt install -y gh` in WSL, then `gh auth login`. Or create the repo through the GitHub UI and `git remote add origin …` manually. Either is fine.

You should now be able to open the repo on GitHub and see the empty scaffold. This is the `main` you'll branch off in every later module.

## 2. Enroll in the ACCY 575 WRDS class

Your access for this course comes through a **WRDS Education Classroom account** — a special type of WRDS account tied to a specific class, with a defined start and end date. It is *not* a personal researcher account, and it expires when the semester ends. Class accounts include the same things you need for this course: SSH to the WRDS Cloud, the JupyterHub and SAS Studio environments, disk storage, and the standard data libraries.

The instructor sets up the class on the WRDS side and gives you a **Class Code** (a short alphanumeric token) at the start of Module 8. You use that code to enroll. Without it you can't get in — registering as a generic individual on `wrds-www.wharton.upenn.edu` will not give you the access this course assumes.

*Optional further reading: [WRDS Student Guide: Enrolling in a Class Account](https://wrds-www.wharton.upenn.edu/pages/about/wrds-account-types/student-guide-enrolling-in-a-class-account/) — the canonical step-by-step for the enrollment flow below.*

### 2a. If you have no WRDS account yet

1. Go directly to the class-student registration form: <https://wrds-www.wharton.upenn.edu/register/?user_type=class-student>. The user-type radio button arrives pre-selected to "Class - Students with Code."
2. Enter your identifying information. For **Subscriber**, choose **University of Illinois at Urbana-Champaign** from the drop-down. Use your `@illinois.edu` email.
3. Paste the **Class Code** the instructor posted to the course site.
4. Click **Register for WRDS**.

### 2b. If you already have a WRDS account from prior coursework or research

1. Log into <https://wrds-www.wharton.upenn.edu>.
2. Top right: **Your Account → Your Account Info**.
3. Scroll to **Your Classes** and click **Enroll in a Class**.
4. Enter the Class Code, confirm "ACCY 575 — Fall 2026," and submit.

### 2c. Set up Duo two-factor authentication

WRDS requires two-factor authentication. Once your enrollment goes through, WRDS will walk you through linking Duo Mobile:

1. Install the **Duo Mobile** app on your phone (free, App Store / Play Store).
2. Follow the on-screen QR-code flow at the [Duo setup guide](https://wrds-www.wharton.upenn.edu/pages/about/log-in-to-wrds-using-two-factor-authentication/) WRDS surfaces during first login.
3. From now on every WRDS access channel — the website, SSH to the WRDS Cloud, and `wrds.Connection()` from Python — challenges you for Duo. The website and SSH accept either a Duo Push or a 6-digit passcode; **Postgres / `wrds.Connection()` accepts Push only.**
4. WRDS retains your MFA session for up to **30 days** as long as your IP address (and, on the web, your browser + cookies) doesn't change. So in practice you'll see the Duo prompt once per network, not once per login.

### 2d. Confirm you can log in

Within an hour of enrollment, log into the WRDS website and click into any data product (e.g., Compustat → Fundamentals Annual). If the query form loads, you're in. If it says you need a subscription, the enrollment isn't through yet — wait an hour, then email the instructor.

> **Heads up — account expiration.** Your class account expires at the end of the term. Anything you want to keep — pulled data, cached embeddings, code — has to live in your local Git repo, not on the WRDS Cloud. Plan accordingly: do not treat your `~/` on the WRDS server as a long-term home directory.

> **Heads up — academic, non-commercial use only.** WRDS data is licensed for **academic and non-commercial research**. That's the binding line. The clear-cut "no" cases: don't trade on it, don't run it as input to a personal investing strategy, don't hand it to an internship or employer for their analysis, don't bundle it into a commercial product. The clear-cut "yes": coursework, papers, theses, any non-commercial research output — with a WRDS acknowledgement when you publish.
>
> A separate rule that *is* immediate-termination: **never share your WRDS credentials with anyone.** Each student has their own account through the class enrollment in §2.
>
> One more nuance: the underlying data vendors (Compustat, CRSP, IBES, etc.) each have their own redistribution clauses on top of the WRDS umbrella, and some are stricter than others about reposting raw extracts. When in doubt — and especially before pushing raw vendor data to a public repo — check the specific vendor's terms or ask the instructor. For this course, §10 covers what's safe to commit and what isn't, and you can follow that without reading every vendor license.

## 3. Set up key-based access to the WRDS Cloud (one time)

**What is SSH?** Short for *Secure Shell*. You open an encrypted terminal session on a remote computer; you type commands locally and they execute on the server. It's how every cloud server and research compute cluster is reached — once you've done it once, the pattern carries everywhere.

The WRDS Cloud is a Linux compute environment Wharton hosts. Two facts about it shape everything else in this module:

- **The 10-K filing files exist only on its filesystem** (under `/wrds/sec/`). No amount of Postgres access from your laptop reaches them. §8b has to run there — that part is not optional.
- **Its login node is one shared machine**, and WRDS caps what any single account may consume on it: processes, memory, CPU. You are going to work *through* that node, not *on* it. Section 4 is entirely about respecting that budget.

There are two SSH front doors to the WRDS Cloud, and the difference between them is the hinge this module turns on:

| Hostname | What it accepts |
|---|---|
| `wrds-cloud.wharton.upenn.edu` | Password + Duo, typed by a human. **Nothing can be automated against it.** |
| `wrds-cloud-sshkey.wharton.upenn.edu` | An SSH key. No password to type. |

You will use the first one exactly once — to install your key. Everything after that goes through the second.

### 3a. Generate an SSH key

[Module 2](../module-02-git-fundamentals/walkthrough.md) mentioned SSH keys and deliberately steered you to `gh auth login` instead. This is where you actually make one, because here it's the only path to a connection your agent can drive.

> **Windows: every "on your laptop" command in this module runs in the Ubuntu shell inside WSL** — the one you set up in [Module 1](../module-01-terminal-vscode-and-your-first-uv-project/walkthrough.md), not PowerShell. This module is precisely where that decision pays for itself, and §4a explains exactly which piece of it Windows cannot do natively. Your key belongs in Ubuntu's `~/.ssh/`, i.e. `/home/<your-unix-name>/.ssh/`. If you ever find yourself generating one into `C:\Users\<you>\.ssh\`, stop — you're in the wrong shell.

On your **laptop**:

```bash
ssh-keygen -t ed25519 -C "wrds-accy575" -f ~/.ssh/wrds_ed25519
```

`-t ed25519` picks the modern key type; `-C` is just a human-readable label baked into the key; `-f` names the files so this key sits beside, rather than on top of, any key you already have. You'll be asked for a passphrase — set one if you like, or press Enter twice for none. (With a passphrase, run `ssh-add ~/.ssh/wrds_ed25519` once per login session so you're not retyping it; on macOS `ssh-add --apple-use-keychain ~/.ssh/wrds_ed25519` remembers it across reboots. WSL has no equivalent keychain — you'd need `eval "$(ssh-agent -s)"` then `ssh-add` in each new Ubuntu shell — so on Windows the path of least resistance is genuinely to press Enter twice and rely on the file permissions instead.)

You now have two files:

- `~/.ssh/wrds_ed25519` — the **private** key. Never leaves your laptop. Never gets committed. Treat it exactly like your password.
- `~/.ssh/wrds_ed25519.pub` — the **public** key. Safe to paste anywhere. This is the half that goes onto the server.

### 3b. Log in once with your password

Copy the public key to your clipboard first — you'll need to paste it in a moment:

```bash
pbcopy < ~/.ssh/wrds_ed25519.pub                        # macOS
# Windows (WSL): clip.exe < ~/.ssh/wrds_ed25519.pub
# Linux:         xclip -selection clipboard < ~/.ssh/wrds_ed25519.pub
```

`clip.exe` is a Windows program, and calling it from an Ubuntu prompt is not a typo — WSL lets either side run the other's executables, so the Linux `<` redirect feeds a Linux file into the Windows clipboard. If none of these work, `cat ~/.ssh/wrds_ed25519.pub` and select the output by hand; just make sure you get the whole single line and no wrapped line breaks.

Then connect:

```bash
ssh <your-wrds-username>@wrds-cloud.wharton.upenn.edu
```

`<your-wrds-username>` is the username WRDS assigned you when your enrollment in §2 was approved (visible in **Your Account → Your Account Info**).

First connection asks you to confirm the host fingerprint — type `yes`. Then it prompts for your **WRDS password** (the one you set during enrollment), prefixed with the account you're logging in as:

```
(<your-username>@wrds-cloud.wharton.upenn.edu) Password:
```

After the password, you'll see a Duo two-factor menu in the terminal — not a popup, plain text:

```
Duo two-factor login for <your-username>

Enter a passcode or select one of the following options:

1. Duo Push to XXX-XXX-1234
2. Phone call to XXX-XXX-1234
3. SMS passcodes to XXX-XXX-1234

Passcode or option (1-3): 
```

Type `1` and press Enter. Approve the push on your phone. WRDS then prints a long welcome banner and drops you at a prompt that looks roughly like:

```
[your-username@wrds-cloud-login2-w ~]$
```

The exact login-node name varies — WRDS has several and balances you across them, so don't be surprised if you land somewhere different next time. **Read that banner rather than scrolling past it.** It lists the grid commands you'll use in §8b (`qsub`, `qstat`, `qdel`, `qrsh`), tells you how to check your disk quota, and — importantly — announces scheduled maintenance windows, during which *all* WRDS Cloud jobs, batch and interactive, are terminated. A multi-hour job submitted the evening before a maintenance window does not survive it.

You're now on a Wharton server. Look around, and take note of two numbers in particular:

```bash
pwd                  # /home/<group>/<your-username>  — WRDS assigns the group segment for your class
ls /wrds             # data libraries available — compustat, crsp, ibes, audit, ...
nproc                # how many CPU cores this shared login node has
ulimit -u            # the ceiling on processes your account may run on it
quota                # your disk quota — WRDS's own command, more useful here than df -h
```

Note that `quota` takes **no flags** here; WRDS wraps it in a script that reports home and scratch together, and passing the usual `-s` gets you an error instead of output. You should see something in this shape:

```
DIRECTORY  USED / LIMIT
    Home:  1.81GB / 10GB
 Scratch:  32.18MB / 500GB
```

**A 10 GB home directory is smaller than it sounds.** A Python virtualenv is a few hundred MB, and §8b writes twenty shard files plus a merged Parquet on top of that. Scratch is far roomier but is shared with everyone else at your institution and is not backed up — fine for intermediate files, not for anything you'd be upset to lose.

**Now write down the `ulimit -u` number.** At the time of writing it is **100** — one hundred processes, total, for your entire account on this machine. That is the single most important number in this module, and §4 exists because of it.

### 3c. Install your public key

Still inside that session:

```bash
mkdir -p ~/.ssh && chmod 700 ~/.ssh
echo '<paste your public key line here>' >> ~/.ssh/authorized_keys
chmod 600 ~/.ssh/authorized_keys
exit
```

Paste the whole single line you copied in §3b — it starts with `ssh-ed25519` and ends with `wrds-accy575`. `authorized_keys` is the standard OpenSSH list of "public keys allowed to log in as me"; appending yours is what makes §3d work.

The two `chmod`s are **not** optional. `sshd` silently ignores an `authorized_keys` file that other users on the machine could read or a `.ssh` directory they could write to — and "silently ignores" means your next login falls back to asking for a password with no explanation of why. If §3d prompts for a password, this is almost always the reason.

### 3d. Confirm key auth works

Back on your laptop:

```bash
ssh -i ~/.ssh/wrds_ed25519 <your-wrds-username>@wrds-cloud-sshkey.wharton.upenn.edu 'hostname; ls /wrds | head -3'
```

Note the shape of that command: a quoted string *after* the hostname. SSH runs it on the server, prints the output locally, and exits — no interactive session, no shell to sit in. **This one-command form is the whole basis of the workflow in §4**, because it's something a program can call.

You should get a hostname and three directory names back, with no password prompt.

**The key is necessary but not sufficient, and it's worth understanding why.** This endpoint requires *two* factors chained: your key, and then Duo. What saves you is that WRDS ties the Duo half to your IP address and keeps that session for up to 30 days — so from a network you've already authenticated on, the second factor is satisfied silently and the command returns in under a second with no interaction at all. From a *new* network (a café, a VPN switch, a different building), the same command stops and waits for a Duo approval.

That distinction is the reason §4 exists in the form it does. A command that is *usually* non-interactive is not something you can hand to an agent — the one time it prompts, the agent has no phone, no way to see the prompt, and no way to answer it, so it hangs until something kills it. §4 removes the "usually."

> **Why bother with the cloud at all, if we mostly work locally?** Two reasons, one of which is binding. The binding one is §8b: the raw filings are on that filesystem and nowhere else. The general one is that some queries are too big to pull — CRSP daily for 30 years is 50+ GB — and in other contexts (regulatory filings, controlled-data audits) the data is *not allowed* to leave the host environment, so compute-goes-to-the-data is the only legal workflow.

## 4. Give your agent a connection to ride

The obvious move at this point is VS Code's **Remote-SSH** extension: it connects to a server, installs a VS Code Server in your home directory there, and hands you a normal-looking editor window whose file tree, terminal, and Python interpreter all live on the remote machine. On most servers it is the right answer, and it's the same pattern Big Four data teams use for regulated datasets that can't leave a hardened environment.

**On the WRDS Cloud it is the wrong answer, and it will fail.** Look again at the `ulimit -u` number from §3b: **100 processes**, for your whole account, on a login node with 8 cores that the entire subscribing institution shares.

A VS Code Server is not one process. It's a Node server, a separate extension host, a terminal host, a file watcher, and one language server per language you've enabled — each of them multi-threaded. A single Remote-SSH window routinely lands in the high dozens of processes *before you open a file*. Against a budget of 100 that is not "a bit heavy," it is most of your allowance spent on an editor. What you get is not a clean error message: it's `fork: retry: Resource temporarily unavailable`, a connection that dies thirty seconds after it opens, or an editor that connects and then silently refuses to open a terminal.

The cap is specific to the **login node**. Jobs on the compute nodes behind the scheduler run with an effectively unlimited process ceiling and no virtual-memory cap — which is the whole argument for §8b's design in one sentence: the restricted machine is a doorway, not a workspace.

So we invert the arrangement:

| | Remote-SSH | What we'll do instead |
|---|---|---|
| Where files are edited | On WRDS Cloud | **On your laptop** |
| Where the editor runs | On WRDS Cloud | On your laptop |
| Where code executes | On WRDS Cloud | On WRDS Cloud |
| Processes held open on WRDS | Dozens, continuously | **One** connection, plus a short-lived shell per command |

You keep the agent-driven workflow you've had since Module 6 — you brief an agent, it writes files, it runs things, it reads output back. The only change is that when the agent needs the cloud, it reaches through a single SSH connection instead of you moving your whole editor onto the server. **The agent becomes your proxy into WRDS.**

### 4a. Write the SSH config

The `ssh …` command in §3d works but is unwieldy, and — more importantly — a fresh one of those opens a *new* connection every time, re-authenticating and re-spawning processes on each call. An agent that runs twenty commands would open twenty connections.

OpenSSH solves this with **connection multiplexing**: the first connection becomes a "master" that leaves a socket file on your laptop, and every later connection to the same host rides that existing socket instead of opening its own. One authentication, one set of server-side processes, unlimited commands.

Create or edit `~/.ssh/config` on your laptop and add:

```
Host wrds
    HostName wrds-cloud-sshkey.wharton.upenn.edu
    User <your-wrds-username>
    IdentityFile ~/.ssh/wrds_ed25519
    ControlMaster auto
    ControlPath ~/.ssh/cm-%r@%h:%p
    ControlPersist 8h
    ServerAliveInterval 60
```

Line by line:

- `Host wrds` — the nickname. From now on `ssh wrds` means all of the above. This is also what makes the agent's commands short and readable.
- `ControlMaster auto` — reuse an existing master connection if one is live; otherwise become the master.
- `ControlPath` — where the socket file lives. `%r@%h:%p` expands to user@host:port so different hosts don't collide. (If `ssh` ever complains that the control path is too long, swap it for `~/.ssh/cm-%C` — `%C` is a short hash of the same four values. Unix sockets have a ~104-character path limit, which a long username plus a long hostname can genuinely exceed.)
- `ControlPersist 8h` — keep the master alive in the background for 8 hours after the last command finishes, so a full working session needs one authentication.
- `ServerAliveInterval 60` — ping every 60 seconds so a NAT or firewall doesn't quietly drop an idle connection.

Set permissions, because `ssh` refuses a config other users can write:

```bash
chmod 600 ~/.ssh/config
```

> **Windows: this is the section that required WSL.** Connection multiplexing works by leaving a **Unix domain socket** on your laptop — a special file that two local processes use to talk to each other, and, crucially, to hand each other live network connections. Microsoft's OpenSSH port has never implemented multiplexing on top of it, so on native Windows the three `Control*` lines above cannot work; `ssh` tries to create the socket, fails, and prints `getsockname failed: Not a socket`. This has been [open since 2016](https://github.com/PowerShell/Win32-OpenSSH/issues/405) with no fix in sight, and Git Bash is not a way around it either — its Cygwin-emulated sockets break multiplexing differently, [dropping the connection](https://cygwin.com/pipermail/cygwin/2022-January/250542.html) when a second command tries to ride the master. Inside WSL you are running real Linux OpenSSH against a real socket, and all of this simply works. So: `~/.ssh/config` here means the Ubuntu one, `/home/<your-unix-name>/.ssh/config`. Windows keeps its own at `C:\Users\<you>\.ssh\config`, which nothing in this course reads.
>
> **And if you see that error anyway, read the fix your agent offers you carefully.** Ask a coding agent about `getsockname failed: Not a socket` and it will correctly diagnose Windows OpenSSH and then confidently propose deleting the `ControlMaster`, `ControlPath`, and `ControlPersist` lines. That does silence the error — and it also removes the entire mechanism §4 exists to build, quietly turning your setup into "a fresh connection, and a fresh set of login-node processes, for every command your agent runs." The error is not telling you the config is wrong. It's telling you the config is being read by the wrong `ssh`. Close PowerShell, open the Ubuntu tab, and run it there. This is the Module 6 lesson arriving in the wild: the agent's answer was locally true and globally wrong, and only you know what the file was for.

*Optional further reading: [`ssh_config(5)`](https://man.openbsd.org/ssh_config) — the authoritative reference for every option above, if you want to see what else an SSH config can do.*

### 4b. Open the master connection

Once per working session:

```bash
ssh wrds
```

Approve the Duo push if you get one (first time on a new network — see §3d). You'll land on the login node. Now **leave it open in its own terminal window and don't work in it.** That window's only job is to hold the master connection alive.

> **Or open it without a shell at all.** `ssh -fN wrds` authenticates, backgrounds itself, and never allocates a terminal — the master exists, but there's no session sitting on the server for you to be tempted to run things in. `-f` means "go to background after authenticating," `-N` means "don't run a command." This is the tidier habit; the interactive window is easier to reason about the first few times.

### 4c. Prove the agent can use it

Open a **second** terminal on your laptop — your normal working one — and check:

```bash
ssh -O check wrds
# Master running (pid=12345)
```

`-O check` asks the multiplexer about the master rather than connecting. Now run a command through it:

```bash
time ssh wrds 'hostname; ulimit -u; ps --no-headers -u $USER | wc -l'
```

Three things to notice in that output:

1. **No password, no Duo.** It rode the existing socket.
2. **It returns in well under a second.** A fresh SSH connection takes a couple of seconds to negotiate; multiplexed ones are effectively free. This is why the agent can afford to call it repeatedly.
3. **The process count is tiny** — your master shell plus the one running the command. Compare that to the `ulimit -u` printed next to it. That headroom is the whole point.

Two more commands worth knowing before you hand this to an agent:

```bash
ssh -O exit wrds     # close the master deliberately (end of the day)
ssh -O check wrds    # "Control socket connect(...): No such file or directory" = no master
```

If the agent ever reports that a WRDS command hung or asked for a password, the first thing to check is `ssh -O check wrds`. A dead master is the cause roughly every time — the fix is to re-run §4b, not to debug the command.

> **Windows:** expect to re-run §4b more often than your classmates on macOS. `ControlPersist 8h` counts wall-clock hours in which the process stays alive, and Windows suspends the whole WSL virtual machine when the laptop sleeps, which usually takes the master with it. Closing the lid over lunch and coming back to a dead master is normal, not a broken setup. (`wsl --shutdown` kills it outright, for the same reason.) One `ssh wrds` and you're back.

### 4d. Tell the agent the rules

Your agent doesn't know any of this yet, and left to itself it will do something reasonable-but-wrong, like trying to `pip install` on the login node or opening a fresh `ssh <user>@<host>` for every command. Write the conventions down where the agent will read them. In your local repo root, create `AGENTS.md` (Claude Code also reads `CLAUDE.md`; Cursor reads `.cursorrules` — use whichever your tool picks up):

```markdown
# WRDS Cloud access

Reach the WRDS Cloud **only** through the multiplexed SSH host alias `wrds`:

The remote working directory is `~/accy575`.

- Run a remote command:  `ssh wrds '<command>'`
- Sync code up:          `rsync -avz --delete --exclude '.venv' --exclude '__pycache__' \
                            --exclude '.git' --exclude 'data' --exclude 'logs' ./ wrds:~/accy575/`
- Pull results down:     `rsync -avz wrds:~/accy575/data/raw/ ./data/raw/`

Rules:
- NEVER use `ssh <user>@wrds-cloud.wharton.upenn.edu` — that endpoint is
  password + Duo only and will hang waiting for human input.
- NEVER start a long-running or interactive process on the login node.
  The login node is shared and process-capped. Heavy work goes to the
  compute grid via `qsub`.
- NEVER drop `--exclude 'data'` or `--exclude 'logs'` from the upward sync.
  Combined with `--delete` that erases results produced on the cloud, and
  `~/accy575/data` is a symlink to scratch — deleting it strands the data.
- NEVER add `--delete` to the downward sync.
- Home on WRDS has a 10 GB quota. Data output goes to `~/accy575/data`,
  which is a symlink into /scratch. Never write large files elsewhere.
- Edit all files locally. The copy on WRDS is a disposable working copy,
  never the source of truth.
- Do not `git push` from WRDS. The repo lives on the laptop.
- Anything submitted with `qsub` runs unattended: no prompts, no Duo, no
  `breakpoint()`, no Postgres connection. Log to stdout instead.
- If a command hangs, check `ssh -O check wrds` before anything else.
```

This is the same discipline as Module 6: the agent is fast and literal, so the constraints have to be written down rather than assumed. A brief that says "pull the MD&A text" without these rules gets you an agent cheerfully trying to open an interactive SSH session and waiting forever for a Duo prompt it can't see.

> **Windows: launch your coding agent from the Ubuntu shell, not from PowerShell.** Everything above assumes the agent's `ssh` is the same `ssh` that owns the master connection you opened in §4b. An agent started in PowerShell gets Windows' OpenSSH, which cannot see the WSL socket at all — so every command it runs opens a fresh connection, and the first one it runs from a new network hangs on a Duo prompt with nobody to answer it. Start the agent from the Ubuntu prompt in the project directory (or from VS Code's integrated terminal while the window shows `WSL: Ubuntu`), and it inherits the right `ssh`, the right `~/.ssh/config`, and the right master.

## 5. Sync code to the cloud with `rsync`

Your code needs to get onto the WRDS Cloud somehow. The tempting answer is to authenticate GitHub over there (`gh auth login`) and `git clone` — and it works, but it buys you a second Git working tree with its own staged files, its own commits, and its own opportunities to diverge from your laptop. You'd then spend the rest of the module reconciling two checkouts of the same repo.

Skip it. **Your laptop's repo is the single source of truth.** The directory on WRDS is a disposable working copy that you push code *to* and pull results *from*. Nothing is ever authored there, so nothing there ever needs committing, and your class account expiring at the end of term costs you nothing.

The tool for this is `rsync`.

> **What is `rsync`?** A file-copy tool that only transfers what actually differs. Where `scp` blindly re-sends every byte, `rsync` compares the two sides first and sends deltas — so re-syncing a project after a one-line edit moves a few hundred bytes instead of the whole tree. It speaks over SSH, which means it inherits your `wrds` alias, your key, and your multiplexed connection for free. The flags you'll see below: `-a` (archive — recurse into directories and preserve timestamps and permissions), `-v` (verbose — list what moved), `-z` (compress in transit), `--delete` (remove files on the destination that no longer exist on the source, so the remote copy is a mirror rather than an ever-growing pile), and `--exclude` (skip paths).
>
> macOS ships it. Ubuntu usually does, but some WSL images are trimmed — if you get `rsync: command not found`, run `sudo apt install -y rsync` once. (Windows ships no `rsync` at all, and no reasonable substitute: this is the second of the two reasons Module 1 put you in WSL.)

### 5a. Create the working directory — and put the data somewhere it fits

Your home directory has a **10 GB quota** (§3b). That is fine for code and a virtualenv, and it is not fine for §8b's output: twenty shard files plus a merged Parquet, on top of a few hundred MB of Python packages, will crowd it and may run you out mid-job. Running out of quota partway through an array job is a genuinely annoying failure — some tasks succeed, some write truncated Parquets, and nothing tells you why until you read the logs.

WRDS gives you a second, much larger space for exactly this: **scratch**, at `/scratch/<your-institution>/`, with a 500 GB quota. Two things to know about it. It is *shared* — that 500 GB covers everyone at your school, so it's a commons, not your personal drive. And it is working space, not storage: treat anything there as deletable, and keep the only durable copy of your results in your local repo.

So: **code in home, data on scratch.** Create both, and link them together:

```bash
ssh wrds 'mkdir -p ~/accy575/{src,jobs,logs}'
ssh wrds 'mkdir -p /scratch/$(id -gn)/$USER/accy575-data/{raw,interim}'
ssh wrds 'ln -sfn /scratch/$(id -gn)/$USER/accy575-data ~/accy575/data'
ssh wrds 'ls -l ~/accy575/'
```

Those are your first real agent-shaped commands: one line each, run remotely, return immediately, hold nothing open. `{src,jobs,logs}` is shell brace expansion — Bash turns it into three separate paths before `mkdir` ever sees it. `$(id -gn)` is your Unix group, which is also your institution's scratch directory name, so this works unchanged for every student.

The last command creates a **symbolic link**: `~/accy575/data` is not a directory but a signpost pointing at the scratch directory. Anything written to `~/accy575/data/raw/` lands on scratch. This matters more than it looks — it means every script, job file, and `rsync` path in the rest of this module can say `data/raw/…` and be right, with the storage decision made once in one place instead of threaded through three scripts. `-s` makes it symbolic, `-f` replaces any existing link, and `-n` stops it from burrowing *inside* a link that's already there if you re-run the command.

> **Why not just work on scratch entirely?** Because home is backed up and scratch is not, and because your code is small. Splitting on "is this expensive to lose, or expensive to store?" is the general form of this decision, and you'll meet it on every cluster you ever use.

### 5b. Push your code up

From your local repo root:

```bash
rsync -avz --delete \
  --exclude '.venv' --exclude '__pycache__' --exclude '.git' \
  --exclude 'data' --exclude 'logs' \
  ./ wrds:~/accy575/
```

The trailing slash on `./` matters: `./` means "the contents of this directory," while `.` would mean "this directory itself" and would nest everything one level deeper. This is the single most common `rsync` mistake.

**Read the exclude list as a protection list, not just a skip list.** `--delete` makes the destination a mirror of the source — anything on the remote that isn't in your local tree gets removed. But rsync will not delete a path you've excluded, and that is exactly what saves you here: `data/` and `logs/` are *outputs* the cloud produces and your laptop doesn't have, and `.venv/` is the Linux environment you'll build there in §8b. Drop any one of those from the list and your next sync quietly deletes hours of work. (`.git` is excluded for a different reason — the remote copy isn't a repo at all, which is the point of §11.)

### 5c. Pull results back down

The mirror image, with the paths reversed:

```bash
rsync -avz wrds:~/accy575/data/raw/ ./data/raw/
```

No `--delete` in this direction — you don't want a cloud-side cleanup to erase your local Parquet files.

### 5d. Let the agent drive it

Those three commands (`ssh wrds '…'`, push, pull) are the entire interface to the WRDS Cloud for the rest of this course, which is exactly why §4d wrote them into `AGENTS.md`. From here on you don't type them — you say things like:

> Sync the repo up to WRDS and run the smoke test there.

and the agent assembles the `rsync` and `ssh` calls itself. Read what it proposes before approving, the same as any other command it wants to run; `--delete` pointed at the wrong path is a genuinely destructive flag.

> **What just happened.** You now have a remote execution environment without a remote editor. The parts of the Remote-SSH experience you actually wanted — run code on the server, see the output, move files both ways — are all still here. What's gone is the heavyweight server-side process that the WRDS Cloud won't let you keep. This laptop-authoritative, sync-to-execute pattern is how most HPC and grid computing has always worked; Remote-SSH is the newer convenience, and it's the one that doesn't survive contact with a process cap.

## 6. Configure local Python access (`.pgpass`)

For the rest of this course we work locally. The `wrds` Python package connects to WRDS over the Postgres protocol. The clean way to authenticate is a `~/.pgpass` file — credentials saved with OS-level permissions, never in your repo.

The first call to `wrds.Connection()` will prompt for username and password and offer to save a `.pgpass` for you. Accept. Or set it up manually:

```bash
echo "wrds-pgdata.wharton.upenn.edu:9737:wrds:<your-username>:<your-password>" > ~/.pgpass
chmod 600 ~/.pgpass
```

The `chmod 600` is required — Postgres refuses to read a `.pgpass` that's world-readable. Confirm:

```bash
ls -l ~/.pgpass
# -rw-------  1 you  staff  ...
```

> **Why not put credentials in `.env`?** `.env` is fine if you read it into Python explicitly. `.pgpass` is the Postgres-native mechanism — the `wrds` client picks it up automatically with no code change, and it's a system-wide credential rather than a per-project one. Both are acceptable; `.pgpass` is the WRDS-recommended path.

*Optional further reading: [Using Python on the WRDS Platform](https://wrds-www.wharton.upenn.edu/pages/grid-items/using-python-wrds-platform/) — WRDS's own reference for the `wrds` client and `.pgpass` setup.*

## 7. Install the Python tooling

Back in your `ACCY575-wrds-data-analysis` project. **First, pin the Python version** — open `pyproject.toml` and set:

```toml
requires-python = ">=3.11,<3.13"
```

This one line saves you a genuinely nasty failure in §8b, and the reason is worth understanding because the shape of it recurs constantly.

By default `uv` picks the newest Python it can find, and the WRDS Cloud ships a very new one. The `sec-parser` package you'll add in §8b depends on `lxml`, and `lxml` is not pure Python — it's a C extension wrapping the `libxml2` and `libxslt` system libraries. For common Python versions the maintainers publish prebuilt **wheels**, so installing it is just a download. For a Python version too new to have wheels yet, `pip`/`uv` falls back to compiling from source, which needs the `libxml2` and `libxslt` *development headers* on the machine — which the WRDS Cloud does not have, and which you cannot install without root. The failure looks like this and stops the whole sync:

```
× Failed to build `lxml==5.4.0`
  Error: Please make sure the libxml2 and libxslt development packages are installed.
```

Pinning below the bleeding edge keeps you on versions that have wheels, so nothing needs compiling and nothing needs root. With the pin in place, `uv` downloads a self-contained CPython 3.12 into your home directory and installs every dependency from a wheel.

> **The general lesson.** On a machine where you don't have root, "newest Python" is a liability, not a feature. Any dependency with a compiled extension — `lxml`, `numpy`, `pyarrow`, `psycopg2` — is a download on a well-trodden version and a build-from-source adventure on a fresh one. Pin deliberately.

Then add the dependencies. **Note the version floor on `wrds`** — it is not decoration:

```bash
uv add "wrds>=3.5" pandas pyarrow pandera
uv add --dev pytest ipykernel jupyterlab
```

Without it, the resolver picks `wrds` 3.1.6 and your first query dies with a error that names nothing you wrote:

```
AttributeError: 'Connection' object has no attribute 'cursor'
```

The chain is worth tracing once, because dependency conflicts always look like this. `wrds` 3.5 requires `pandas<2.3`; other packages in this project are happy with newer pandas; the resolver takes the newest pandas it can and then walks *`wrds`* backwards to 3.1.6 to fit. But 3.1.6 requires `sqlalchemy<2`, and SQLAlchemy 1.4 connections are not something modern pandas recognises as a database connection — so `pandas.read_sql_query` falls through to a raw-driver code path and calls `.cursor()` on an object that has none. Three packages each behaving correctly, one broken program. Pinning `wrds>=3.5` forces the resolver to solve for the combination that works: `wrds` 3.5, `pandas` 2.2.x, `sqlalchemy` 2.0.x.

> **The habit.** When a library fails inside its own internals on a call you made correctly, check versions before you check your code. `uv pip list | grep -E 'wrds|pandas|sqlalchemy'` costs five seconds and is right more often than any amount of staring at the traceback.

(`ipykernel` is what VS Code needs to run notebook cells; `jupyterlab` adds the standalone `jupyter lab` command that Modules 9–11 use to open notebooks from the terminal. Without it, `uv run jupyter lab` fails with `Error executing Jupyter command 'lab'`.)

Quick sanity check:

```bash
uv run python -c "import wrds; db = wrds.Connection(); print(db.list_libraries()[:3]); db.close()"
```

What you should see on a fresh `.pgpass`'d setup, the first time you run this from a new network:

1. The command appears to hang for a few seconds — *that's the Postgres server waiting on Duo.*
2. A **Duo Push** lands on your phone. Approve it. (Postgres connections don't accept passcode entry — Push is the only second-factor option here.)
3. Back in the terminal, three library names print and the script exits.

After that, WRDS keeps your MFA session warm for ~30 days from the same IP — re-runs from the same network return instantly with no Duo prompt. From a different network (a coffee shop, a VPN switch), the Duo Push step comes back.

> **Aside — `python -c "…"`.** The `-c` flag tells Python to run the string that follows as a complete program and exit, no file or REPL involved. It's the fastest way to execute one or two lines of Python without opening an editor or `python` shell — useful for sanity checks like this, for grabbing a quick value out of a Parquet file (you'll see this pattern again in §8a), or for running anything you'd otherwise type into iPython once and never again. Statements in the string are separated by `;` since `-c` is one line; for anything multi-line, put the code in a `.py` file instead.

## 8. Build the Part 2 dataset

Open your coding agent (Copilot Chat, Claude Code, Cursor — whichever you're using). Write briefs the way Module 6 trained you to: a clear spec gets you a clean result; a vague prompt gets you a confidently wrong one.

The pull has **two parts that live in different places**:

- **Compustat fundamentals.** Sit in WRDS Postgres. Reachable from your laptop over `.pgpass`. Pull this locally.
- **MD&A text from 10-K filings.** WRDS Postgres only stores the *index* (`wrdssec_all.wrds_forms` — about 238,000 10-K rows). The filing files themselves live on the WRDS Cloud's own filesystem under `/wrds/sec/`, reachable only from a process *running there*. **You'll write this one locally and have the agent execute it on the cloud through the §4 connection.**

This is where §4 (the multiplexed `wrds` connection) and §5 (`rsync`) stop being setup and become required. If you skipped them, go back.

### 8a. Compustat fundamentals — *local*

In your local `ACCY575-wrds-data-analysis` repo, brief the agent:

> **Brief.** Write `src/pull_fundamentals.py`. Connect to WRDS using `wrds.Connection()`, pull annual Compustat fundamentals (`comp.funda` table) for S&P 500 firms over 2010–2024, save to `data/raw/fundamentals.parquet`.
>
> Variables to pull: `gvkey`, `datadate`, `fyear`, `tic`, `conm`, `at` (assets), `lt` (liabilities), `revt` (revenue), `ni` (net income), `oancf` (operating cash flow), `xrd` (R&D), `sich` (industry).
>
> Compustat filters: `indfmt='INDL'`, `datafmt='STD'`, `popsrc='D'`, `consol='C'`, `fyear` between 2010 and 2024 inclusive.
>
> S&P 500 membership comes from **CRSP, not Compustat** — `comp.idxcst_his` is present but materially incomplete, and using it silently costs you a fifth to a third of the index (see below). Use `crsp_a_indexes.dsp500list_v2` (or the legacy `dsp500list`), join to Compustat through `crsp.ccmxpf_lnkhist` — the CRSP permno column there is named `lpermno`, not `permno` — with the standard `linktype in ('LU','LC')` and `linkprim in ('P','C')` filters. Apply the membership window with the columns that match the table you chose: `mbrstartdt <= datadate <= mbrenddt` for `dsp500list_v2`, or `start <= datadate <= ending` for the legacy `dsp500list`.
>
> Print the row count at the end. Don't commit the parquet file.

What this brief gets right that a sloppy prompt wouldn't:

- The four standard Compustat filter flags (`INDL`/`STD`/`D`/`C`). Without them you get duplicates from the supplemental and restated formats.
- **CRSP-not-Compustat for S&P 500 membership.** Most tutorials online tell you to use `comp.idxcst_his`. That table is right there, it answers your query, and it is wrong. Count the members it reports for `gvkeyx='000003'` (S&P 500 Comp-Ltd) against CRSP's on the same dates:

  | Date | `comp.idxcst_his` | `crsp_a_indexes.dsp500list_v2` |
  |---|---|---|
  | 2015-06-30 | 326 | 502 |
  | 2019-06-30 | 396 | 504 |
  | 2021-06-30 | 424 | 505 |
  | 2024-06-30 | 465 | 503 |

  CRSP returns the ~500 you'd expect. Compustat returns a number that is plausible enough to pass a glance and is missing a fifth to a third of the index. Nothing errors, nothing warns, and every downstream statistic is computed on a silently truncated, non-randomly-selected panel. **Run this comparison yourself rather than taking the table above on faith** — it takes one query each, and the habit of checking a reference table against a second source before building on it is the actual lesson. This is the sampling-bias mistake §8 exists to teach you to catch.
- The link-history join (`ccmxpf_lnkhist`) with `linktype`/`linkprim` filters. Naive joins on `permno` or `cusip` produce duplicate-firm and history-misalignment bugs.
- "Don't commit the parquet" — the agent will sometimes try to `git add` the data file. Stop it.

Read the file the agent produces. Make sure it does what the brief said and nothing else. Then run it:

```bash
uv run python -m src.pull_fundamentals
```

Expected: a Duo Push on first run (§7), then 1–3 minutes of querying, then a row count and a written `data/raw/fundamentals.parquet`. Inspect:

```bash
uv run python -c "import pandas as pd; print(pd.read_parquet('data/raw/fundamentals.parquet').head())"
uv run python -c "import pandas as pd; print(pd.read_parquet('data/raw/fundamentals.parquet').shape)"
```

Sanity checks worth running by hand:

- Row count plausibility: ~500 firms × ~15 years ≈ 7,000–8,000 rows (less, because of constituent changes).
- Year coverage: `df['fyear'].value_counts().sort_index()` — should be roughly even across 2010–2024.
- No duplicate `(gvkey, fyear)` pairs.

If any of these look wrong, that's a brief problem, not a data problem — go back to the brief and fix the spec.

### 8b. MD&A text — *on the WRDS Cloud*

This is the more involved half of the pull, and three later modules depend on it:

- **Module 11 (BERT / FinBERT)** encodes each MD&A paragraph into a 768-dim vector and adds those embeddings as features in the M9/M10 models — the question being whether what management *says* adds signal beyond the numbers.
- **Module 12 (LLM zero-shot)** classifies and extracts structured fields from MD&A: risk-factor categorization, forward-looking-statement detection, segment disclosures.
- **Module 13 (RAG)** indexes the MD&A chunks and lets you query the corpus with natural language ("which firms discussed supply chain in 2022?").

Why MD&A and not the full 10-K or earnings-call transcripts: it's a standardized location (Item 7) the parser can find reliably, it's where management writes in their own words rather than reciting GAAP, and it joins cleanly to Compustat on `(gvkey, fyear)` so the text and fundamentals signals are directly comparable.

Everything here is authored **in your local repo**, in your normal editor, with your normal agent. It executes on the cloud through the §4 connection. You never open an editor on the server.

#### The shape of the job, and why it's split in three

The naive version is one script that queries Postgres and then parses 7,000 files. Three things about the WRDS Cloud make that the wrong shape.

**The parsing is too heavy for the login node.** Each filing means reading a multi-megabyte file off a shared network filesystem and running a parser over it; 7,000 of them is on the order of 20 GB of reads and a couple of hours of work. The login node is one shared machine with a 100-process cap and a 4 GB per-process memory ceiling (§3b) — running that there is both against the spirit of a shared resource and a reliable way to get your processes killed mid-run. WRDS Cloud, like most research clusters, has a **job scheduler** for exactly this, and heavy work belongs on the compute nodes behind it, where neither of those limits applies.

**The query should run once, not twenty times.** If each array task connected to Postgres and re-derived the panel itself, you'd run the same CRSP–Compustat link join twenty times concurrently against a shared database to produce twenty identical answers. Worse, you'd have no guarantee the twenty results are byte-identical — and if they aren't, your "deterministic" shard boundaries silently overlap or drop rows. Query once, freeze the answer to disk, and let the tasks read it.

**A bad query should cost ten seconds, not twenty tasks.** Run interactively and you see the row count immediately; bury the same query inside the array job and you find out after the scheduler has dispatched twenty tasks, and you debug it by reading twenty logs.

> **An asymmetry worth noticing.** From your laptop, `wrds.Connection()` needs a `.pgpass` and a Duo push (§6–§7). From *inside* the WRDS Cloud — login node or compute node — it needs neither: the Postgres server trusts connections originating on its own network, so `wrds.Connection(wrds_username="…")` just works, unattended, with no credentials on disk. That's convenient, and it's also why the rule in §4d about batch jobs never prompting is about your *code*, not about WRDS's authentication. The general principle still holds everywhere else: anything submitted to a scheduler runs with no human attached, so a library that decides to ask a question mid-run doesn't fail — it hangs until the scheduler kills it.

Hence three pieces:

| | Script | Where it runs | Why |
|---|---|---|---|
| Stage 1 | `src/build_mdna_manifest.py` | Login node, interactive | One Postgres query, frozen to disk. Seconds of work, and you see the row count immediately if it's wrong. |
| Stage 2 | `src/parse_mdna_shard.py` | Compute nodes, via `qsub` | Reads files and parses them. No database, no network. Hours of work, parallelised. |
| Stage 3 | `src/merge_mdna.py` | Compute node, via `qsub` | Concatenates shard outputs. Seconds of work, but needs more memory than the login node allows. |

> **What is a job scheduler?** On a shared cluster you don't run programs directly — you *submit* them, and a scheduler decides when and on which machine they run. WRDS Cloud runs **Grid Engine**, whose commands are `qsub` (submit a job), `qstat` (check on your jobs), and `qdel` (cancel one). Submitting returns immediately with a job ID; the job itself starts whenever slots free up and runs to completion with nothing attached to it — no terminal, no SSH session, nothing that dies when you close your laptop. Run `ssh wrds 'qconf -sql'` to list the queues that actually exist — `all.q` is the general-purpose one and the only one this module needs; `interactive.q` backs the `qrsh` command in the login banner, and the rest serve WRDS's own services. Don't copy a queue name out of a tutorial without checking it against that list; queue names are per-site and a job submitted to a queue that doesn't exist is rejected outright.
>
> The compute nodes behind `all.q` are a different class of machine from the login node — 24 to 48 cores and around 500 GB of RAM each, against the login node's 8 cores and a 4 GB per-process memory cap. The scheduler is also what makes your work a good citizen: it knows how many slots exist and won't let the class collectively oversubscribe the cluster the way twenty simultaneous `multiprocessing.Pool`s on the login node would.

> **What is an array job?** One submission that expands into N nearly-identical tasks, numbered `1..N`, each scheduled independently. `qsub -t 1-20` gives you twenty tasks; inside each, the environment variable `SGE_TASK_ID` tells that task which number it is. It is the natural fit for embarrassingly parallel work — 7,000 filings that don't need to talk to each other — and it's *better* than a `multiprocessing.Pool` here, because the scheduler is doing the parallelism across whole machines instead of one process fighting for cores on one machine. It also degrades gracefully: if one task dies, you re-submit that task, not the whole run.
>
> The `multiprocessing` idea is still worth knowing — Python's Global Interpreter Lock stops threads from running Python bytecode in parallel, so CPU-bound speedups need separate *processes*, which is what `multiprocessing.Pool` gives you. On a laptop it's the right tool. On a grid, the scheduler is.

*Optional further reading: [`qsub(1)`](https://gridscheduler.sourceforge.net/htmlman/htmlman1/qsub.html), [`qstat(1)`](https://gridscheduler.sourceforge.net/htmlman/htmlman1/qstat.html), and [`qacct(1)`](https://gridscheduler.sourceforge.net/htmlman/htmlman1/qacct.html) — the Grid Engine manual pages for the three commands used below.*

#### Stage 1 — build the manifest

In your local repo, brief the agent:

> **Brief.** Write `src/build_mdna_manifest.py`. It runs **on the WRDS Cloud login node** and does no file parsing — it only figures out *which* filings to parse.
>
> Connect with `wrds.Connection(wrds_username="<your-wrds-username>")` — pass the username explicitly, because with no argument the client stops and asks for it, which is fine when you're typing and useless when an agent is. Use the same S&P 500 / 2010–2024 panel as `src/pull_fundamentals.py` — re-derive `(gvkey, fyear)` from CRSP + `ccmxpf_lnkhist` so this script is independent. Map each `gvkey` to its historical `cik` via **`wrdssec.wciklink_gvkey`** (note the library: it is `wrdssec`, *not* `wrdssec_all`, which holds only `wrds_13f_link`). This is the WRDS-maintained linking table; it carries `link_start_date` and `link_end_date`, so pick the link active for each firm-year rather than the most recent one, and drop the rows where `gvkey` is null — plenty of CIKs are trusts and funds with no Compustat counterpart at all. Then query `wrdssec_all.wrds_forms` for `form='10-K'` filings whose filing date falls in fiscal year `fyear` (or shortly after — 10-Ks typically file 60–90 days after fiscal year-end, so allow `fyear+1` Q1). Each row of the result has a `wrdsfname` column naming the filing's file. Resolve it against **`/wrds/sec/wrds_clean_filings/{wrdsfname}`**, not the raw `/wrds/sec/warchives/` tree — see the note below on why.
>
> Write `data/interim/mdna_manifest.parquet` with columns `(gvkey, fyear, cik, fdate, filepath)`, sorted deterministically by `(gvkey, fyear)` so that shard boundaries are reproducible. Print the row count.

#### Stage 2 — parse one shard

> **Brief.** Write `src/parse_mdna_shard.py`. It runs **on a compute node inside a Grid Engine array task** and must be completely non-interactive — no Postgres connection, no prompts, no network.
>
> Accept `--shard N --nshards M --manifest <path> --out <path>` on the command line via `argparse`. Read the manifest, take every row where `index % M == N` (a deterministic stride, so the shards partition the manifest exactly and no row is done twice), and process only those.
>
> For each row, open `filepath` from the local filesystem, read the contents, and extract the "Item 7" section (Management's Discussion and Analysis). Use a 10-K-aware parser — `sec-parser` from PyPI is the cleanest path; regex on `r"item\s*7[.\s]"` … `r"item\s*7a[.\s]"` is the fallback. If extraction fails for a filing, log the `gvkey/fyear` to stderr and skip the row — **never fabricate text**.
>
> Write `--out` as Parquet with columns `(gvkey, fyear, cik, fdate, mdna_text, char_count)`. Print the number of rows written and the number of failures. Exit non-zero if more than 20% of the shard failed, so the scheduler records a failure rather than a silent empty result.

> **Brief.** Write `jobs/parse_mdna.sh`, a Grid Engine array-job script for `src/parse_mdna_shard.py` with 20 tasks. It must:
>
> - Set `#$ -cwd` (run from the submit directory), `#$ -N parse_mdna` (job name), `#$ -j y` (merge stderr into stdout), `#$ -o logs/` (write logs there), `#$ -t 1-20` (array of 20 tasks), `#$ -q all.q` (queue).
> - Use the virtualenv interpreter by absolute path — `"$HOME/accy575/.venv/bin/python"` — not `uv run` and not a bare `python`, because a compute node's `PATH` is not your login shell's.
> - Convert `SGE_TASK_ID` (which Grid Engine numbers from 1) to a 0-based shard index once, into a shell variable, and use that same variable for both `--shard` and the output filename so the two can never drift apart. Pass `--nshards 20`.
> - Write each task's output to `data/interim/mdna_shard_${SHARD}.parquet`, so the twenty files are numbered `0`–`19` and match the `--shard` values exactly.

What these briefs get right:

- **Where each stage runs.** The `filepath` values only resolve on WRDS Cloud's filesystem; from your laptop, even with a valid `.pgpass`, the file reads would fail. This is *the* reason §4/§5 exist.
- **`wrdsfname` is already a relative path, not a bare filename.** A row comes back looking like `000031/315374/0001144204-19-000709.txt` — a CIK-bucket directory, the CIK, then the accession number — so joining it to an archive root is a plain concatenation and nothing needs to be reconstructed. Check one by hand before you trust 7,000 of them:
  ```bash
  ssh wrds 'head -5 /wrds/sec/wrds_clean_filings/000031/315374/0001144204-19-000709.txt'
  ```
  That should print an SEC document header ending in `CONFORMED SUBMISSION TYPE: 10-K`. If your agent invented a path-building scheme instead of using the column as given, this is where you'll catch it.
- **Cleaned filings, not raw archives.** WRDS keeps two copies of every filing under the same `wrdsfname`: `/wrds/sec/warchives/` is the complete submission, exhibits and encoded attachments included, and `/wrds/sec/wrds_clean_filings/` is the same document with that ballast stripped. Sampling 25 real 10-Ks, the raw copies average **24 MB** each and the cleaned copies **2.8 MB** — nearly nine times smaller — and the cleaned version still contains the full Item 7 text you're after. Across 7,000 filings that is the difference between reading ~170 GB over the network filesystem and reading ~20 GB. This job is far more I/O-bound than CPU-bound, so that ratio, not your parser, sets the runtime.
- **No Postgres in the batch stage.** Not because the connection would fail there — it works fine — but because twenty tasks re-deriving the same panel is twenty times the database load for one answer, and twenty chances for the shard boundaries to disagree.
- **A deterministic shard rule.** `index % M == N` over a sorted manifest guarantees the 20 shards partition the work with no overlap and no gaps. "Split into 20 chunks" implemented against an unsorted query result does not.
- **Absolute interpreter path.** Compute nodes don't source your `.bashrc` the way an interactive login does. Half of all "it worked when I ran it by hand" grid bugs are `PATH` bugs. On this cluster a bare `python3` can resolve to either of two very different interpreters depending on `PATH` — `/usr/bin/python3` and `/usr/local/bin/python3` are several major versions apart — so "whichever `python3` the job happens to find" is not a thing you want to leave to chance.
- **`-cwd` and the shared filesystem.** Your home directory is mounted on the compute nodes as well as the login node, which is what makes this whole design work: the job script, the manifest, the virtualenv, and the output directory are all the same files from either side. It's also why nothing needs to be copied to a compute node before the job runs, and why the shard Parquets are simply *there* when it finishes.
- **Historical CIK linking.** Companies change CIKs (mergers, restructurings). `wciklink_gvkey` keeps history; using a current-only mapping produces silent gaps for firms that switched.
- **Filing-date window.** 10-K filing date is usually in early `fyear+1`, not in `fyear`. Naively filtering by `fdate ∈ fyear` drops most of your panel.
- **Skip-and-log on parse failure.** Some 10-Ks have non-standard structure that the parser will choke on. Better to log them than to ship a Parquet with hallucinated text in a few rows.
- **Two different failure thresholds, on purpose.** The brief's *non-zero exit above 20%* is a circuit breaker: it tells the scheduler this task is broken so `qacct` records a failure instead of a silently-thin Parquet. The *~5%* figure in the sanity checks below is a quality bar: that's roughly what a real 10-K-aware parser achieves, and anything much worse means the extraction logic needs work even though every task "succeeded." For calibration, crude `item 7` … `item 7a` regex matching on a sample of real filings fails around **28%** of the time — which is exactly why `sec-parser` is the primary path and regex is only the fallback.

Read every file the agent produced before any of it goes near the cluster. A bug you ship to 20 compute nodes is a bug 20 times over.

#### Build the environment on the cloud

`sec-parser` isn't in the lockfile yet. Add it **locally** first, so the lockfile that gets synced up is the one you committed:

```bash
uv add sec-parser
```

> `sec-parser`'s last PyPI release is `0.58.1` (June 2024) and the project's GitHub README now states it is no longer maintained — it still works, so pin the version and keep the regex fallback from the brief ready in case a filing trips it up.

*Optional further reading: [`sec-parser`](https://pypi.org/project/sec-parser/) — Alphanome.AI's 10-K-aware parser, the package doing the Item 7 extraction in Stage 2.*

Now push the code up and build a virtualenv **on the cloud**. Your local `.venv` is excluded from the sync in §5b for a good reason: it holds compiled binaries built for *your* machine, and hard-codes absolute paths to *your* home directory. Neither survives the trip — even from WSL, where the binaries are at least Linux ones, the paths are wrong and the C libraries they were built against are a different vintage. The lockfile travels; the environment gets rebuilt.

`uv` is **not** preinstalled on the WRDS Cloud, and you don't have root there, so system package managers are out. Install it into your home directory once — the official installer script does exactly that:

```bash
ssh wrds 'curl -LsSf https://astral.sh/uv/install.sh | sh'
```

Then sync the code up and build the environment:

```bash
rsync -avz --delete --exclude '.venv' --exclude '__pycache__' --exclude '.git' --exclude 'data' --exclude 'logs' ./ wrds:~/accy575/
ssh wrds 'cd ~/accy575 && ~/.local/bin/uv sync'
```

Note the **absolute path** `~/.local/bin/uv` rather than adding the directory to `PATH` in `~/.bashrc`. Most Linux `.bashrc` files open with a guard like `case $- in *i*) ;; *) return;; esac` that bails out immediately for non-interactive shells — and `ssh wrds '<command>'` is exactly that. So a `PATH` line appended to `.bashrc` works when you log in by hand and silently does nothing when your agent runs a command, which is a maddening class of bug to chase. Absolute paths sidestep it, and you'll see the same reasoning again in the job script.

(Grid Engine's own commands — `qsub`, `qstat`, `qacct` — are the exception: WRDS puts them on the system-wide `PATH`, so `ssh wrds 'qsub …'` works without any of this. Don't generalise from that to your own tools.)

If `uv sync` dies on `Failed to build lxml`, you skipped the `requires-python` pin in §7 — go set it, re-sync locally so `uv.lock` updates, and push again.

Confirm the interpreter the job script will use actually exists and can import what the job needs:

```bash
ssh wrds '~/accy575/.venv/bin/python -c "import sec_parser, wrds, sys; print(\"env ok\", sys.version.split()[0])"'
```

You should get `env ok 3.12.x`. (The `wrds` package emits a few `SyntaxWarning: invalid escape sequence` lines on recent Pythons — cosmetic, from unescaped backslashes in its own docstrings, and safe to ignore.) Note where that interpreter actually points:

```bash
ssh wrds 'readlink -f ~/accy575/.venv/bin/python'
# /home/<group>/<you>/.local/share/uv/python/cpython-3.12.13-linux-x86_64-gnu/bin/python3.12
```

It's a Python `uv` downloaded into **your home directory**, not a system one — which matters because home is mounted on the compute nodes too. The interpreter your array tasks run is the same file you just tested, and nothing needs to be installed on the nodes themselves.

#### Run Stage 1, then submit Stage 2

Stage 1 is seconds of work, and you run it directly so you see the row count before committing twenty tasks to it:

```bash
ssh wrds 'cd ~/accy575 && .venv/bin/python -m src.build_mdna_manifest'
```

This runs unattended — no password, no Duo, because you're already inside WRDS's network. It takes **a minute or two**, not seconds: the CRSP–Compustat–link join is doing real work server-side even though the result is small. A reference run over the S&P 500 / 2010–2024 panel returns **7,463 filings across 762 firms**; if your number is wildly off from that, fix it here rather than discovering it twenty tasks later. Then submit the array job:

```bash
ssh wrds 'cd ~/accy575 && qsub jobs/parse_mdna.sh'
# Your job-array 4815162.1-20:1 ("parse_mdna") has been submitted
```

That command returns **immediately**. The job is now the scheduler's problem, not your connection's — you can close your laptop, change networks, or let the SSH master expire, and it keeps running. This is what replaces the old habit of wrapping long jobs in `tmux`: a scheduled job was never attached to your session in the first place.

#### Watch it

```bash
ssh wrds 'qstat -u $USER'
```

The `state` column is what you're reading: `qw` means queued and waiting for a slot, `r` means running, `Eqw` means the task errored on submission (usually a bad path in the job script). An empty result means every task has finished — Grid Engine drops completed jobs from `qstat`.

Ask your agent to poll it rather than doing it yourself:

> Check the WRDS job queue every couple of minutes until `parse_mdna` is gone, then show me the last 20 lines of each log in `~/accy575/logs/`.

Each task writes its own log into `logs/`, named `<job-name>.o<job-id>.<task-id>` — so a 20-task run of `parse_mdna` leaves `parse_mdna.o34970638.1` through `…20`. The task ID on the end is what lets you tie a specific failure back to a specific shard, which is the whole reason `-t` and `-o logs/` were worth setting.

When it's done, look at the logs before you trust the output — a task that crashed 30 seconds in also disappears from `qstat`:

```bash
ssh wrds 'ls ~/accy575/data/interim/mdna_shard_*.parquet | wc -l'   # expect 20
ssh wrds 'grep -il "traceback\|error" ~/accy575/logs/* || echo "no errors"'
ssh wrds 'qacct -j parse_mdna 2>/dev/null | grep -E "taskid|exit_status|ru_wallclock" | head -60'
```

`qacct` is the accounting record for *finished* jobs — per-task exit status and wall-clock time, which is the only place to see why a task that's no longer in `qstat` failed. `exit_status 0` on all 20 tasks is what you want.

**Realistic runtime.** Measured on a full 7,463-filing manifest against the cleaned filings tree, with a regex extractor:

| | Observed |
|---|---|
| `ru_wallclock` per task (≈370 filings each) | **31–35 seconds** |
| Wall-clock for all 20 tasks, lightly loaded cluster | **~2 minutes** |
| Same work sequentially in one process | ~12 minutes |

Two caveats before you plan around those numbers. On a **busy** cluster the array serialises — only some tasks get slots at once, and your wall-clock becomes mostly `qw`, not `r`, which no amount of optimising your Python touches. And a real 10-K-aware parser is substantially slower per filing than a regex; if you followed the brief and used `sec-parser`, expect the per-task figure to be minutes rather than seconds.

What the measurement does settle is the shape of the thing: this is a **minutes** job on the grid, not the multi-hour ordeal it becomes on one core — and if you pointed the script at `/wrds/sec/warchives/` instead of the cleaned tree, multiply everything by roughly nine.

If a shard or two failed, you don't re-run everything — re-submit just those tasks, overriding the `-t` in the script from the command line:

```bash
ssh wrds 'cd ~/accy575 && qsub -t 7-7  jobs/parse_mdna.sh'
ssh wrds 'cd ~/accy575 && qsub -t 13-13 jobs/parse_mdna.sh'
```

Grid Engine's `-t` takes **one** range per submission — `n`, `n-m`, or `n-m:step` — not a comma-separated list, so two failed tasks means two `qsub` calls (or `-t 7-13:6`, if you enjoy that sort of thing). A command-line `-t` overrides the `#$ -t` directive inside the script, which is why the script doesn't need editing.

That granularity is the practical payoff of the array-job design. Two failed tasks out of twenty means you re-run a tenth of the work, not all of it — and on a job measured in tens of minutes, that is the difference between an annoyance and an afternoon.

#### Stage 3 — merge and pull down

> **Brief.** Write `src/merge_mdna.py`. Read every `data/interim/mdna_shard_*.parquet`, concatenate them, assert the result has no duplicate `(gvkey, fyear)` pairs, sort by `(gvkey, fyear)`, validate against `MdnaSchema` (§9), and write `data/raw/mdna.parquet`. Print the final row count, the median `char_count`, and the count of shard files it read, and fail loudly if that count isn't the number of tasks you submitted.
>
> Also write `jobs/merge_mdna.sh` to submit it: same directives as `jobs/parse_mdna.sh` but with no `-t` (it's a single task) and with `#$ -l m_mem_free=8G`.

**This one has to go through the scheduler too, and the reason is instructive.** The merge is "just a concatenation," it takes seconds, and running it on the login node is the obvious move. It fails:

```
pyarrow.lib.ArrowMemoryError: realloc of size 1027080192 failed
```

Concatenating ~7,000 MD&A documents means holding about a gigabyte of text in memory, then handing a copy of it to Parquet to serialise — comfortably past the login node's 4 GB per-process cap once you count the copy. The job is *fast*, not *small*, and the login node limits memory, not time. Those are different axes and it's easy to conflate them.

```bash
ssh wrds 'cd ~/accy575 && qsub jobs/merge_mdna.sh'
# poll qstat as before, then:
ssh wrds 'cat ~/accy575/logs/merge_mdna.o*'
rsync -avz --progress wrds:~/accy575/data/raw/mdna.parquet ./data/raw/
```

For reference, a full run of the S&P 500 / 2010–2024 panel produces:

```
shard files found: 20 (expected 20)
rows          : 7,157
firms         : 753
median chars  : 77,675
parquet bytes : 271,703,291
```

That `rsync` is the only place a raw Parquet crosses the boundary from server to laptop. A sample of real 10-Ks puts the median extracted MD&A at about **69,000 characters**, so 7,000 of them is roughly **half a gigabyte of raw text** — and Parquet compresses prose well, so expect the file itself to land in the low hundreds of MB. `--progress` is worth adding on a transfer that size. It's a one-time download; for the rest of Part 2 your laptop is the primary work environment.

Once it's safely on your laptop and the schema in §9 passes, you can delete the shards from scratch — that's what scratch is for, and the class shares its quota:

```bash
ssh wrds 'rm -f ~/accy575/data/interim/mdna_shard_*.parquet'
```

(Re-running the parse locally is *not* a fallback. The filings live under `/wrds/sec/` on the WRDS Cloud's filesystem and nowhere else, and pulling them in bulk runs into tens to hundreds of GB of transfer plus vendor licensing.)

**Keeping an eye on a shared server.** Even with the heavy work on compute nodes, you can still exhaust your own disk quota — and a 20-task array job writing shards is exactly the thing that does it. `quota` (no flags — see §3b) is WRDS's own command for this and the one to reach for first, since it reports your 10 GB home and the shared scratch space together; `free -h` and `top` cover memory and live processes on the login node.

Three habits that transfer to any cluster:

- `qstat -u $USER` **before** submitting again, so you don't stack duplicate jobs on top of a run that's still going.
- `qstat -u \*` to see what the rest of the cluster is doing. If your tasks sit in `qw` for twenty minutes, this tells you whether it's you or a queue full of someone else's work — the escaped `\*` is there because an unquoted `*` would be expanded by your shell before `qstat` ever sees it. `qhost -j` gives the same picture organised by machine.
- `qacct`'s `ru_wallclock` and `maxvmem` **after** a run, to learn what your job actually costs. That's the number that turns "I think I need more memory" into a specific request — `qsub -l m_mem_free=8G` reserves 8 GB per task, and `m_mem_free` is the resource name this cluster actually uses (confirm with `ssh wrds 'qconf -sc'`, which lists every requestable resource). Reserving without measuring first is guessing with extra steps.

*Optional further reading: [`memory_profiler`](https://pypi.org/project/memory-profiler/) — line-by-line RAM profiling for when a Compustat or text pull blows past a node's memory limit.*

Sanity checks once `mdna.parquet` is on your laptop:

- **Row count** close to but slightly under the manifest's. Some firms file 10-K-equivalents (10-KSB, 10-K405) instead, and a fraction of extractions fail. A reference run: 7,463 in the manifest, 7,157 written, 306 logged failures — **4.1%**, inside the 5% quality bar.
- **Year coverage** roughly flat: `df.fyear.value_counts().sort_index()`. The reference run gives 470–486 rows per year across 2010–2024. A sagging tail usually means the filing-date window is wrong, not that the data is missing.
- **Spot-check three rows** by printing `mdna_text[:300]`. They should open on "Item 7. Management's Discussion and Analysis…" and read like prose.

Then do the check almost everyone skips, because it is where the real problem is:

- **Look at the distribution of `char_count`, not just the failure count.** In the same reference run the logged failure rate was a respectable 4.1% — and the length distribution said something else:

  | `char_count` | Share | What it is |
  |---|---|---|
  | under 1,000 | **7.8%** | Matched a table-of-contents entry, not the section |
  | 1,000–5,000 | 1.8% | Truncated — hit an early false `Item 7A` |
  | 5,000–300,000 | 88.9% | Plausible MD&A |
  | over 300,000 | **1.5%** | Ran past the end; the worst was **8.3 MB** |

  So roughly **one row in nine is unusable while every task exits 0 and the failure counter says 4.1%.** A failed extraction is loud and gets logged; a *wrong* extraction returns a string, passes every check that only asks "did we get something," and quietly poisons whatever you build on it. Modules 11–13 will happily embed and index an 8 MB blob of exhibit tables and tell you nothing is wrong.

  This is the single most transferable lesson in the module: **an error rate you measured is not the same as a quality rate you didn't.** The §9 schema is where you turn this observation into an automated guard, which is why it bounds `char_count` from both sides.

If any of these look wrong, fix the brief above and re-run. Because everything was authored locally, "fix and re-run" is edit → `rsync` → `qsub`, with no reconciling of two copies of the code.

## 9. Validate with `pandera`

Same pattern as Module 5. Add `src/schema.py` with one schema per Parquet:

```python
import pandera.pandas as pa
from pandera.typing import Series

class FundamentalsSchema(pa.DataFrameModel):
    gvkey: Series[str]
    datadate: Series[pa.DateTime]
    fyear: Series[int] = pa.Field(ge=2010, le=2024)
    tic: Series[str]
    at: Series[float] = pa.Field(ge=0, nullable=True)
    lt: Series[float] = pa.Field(ge=0, nullable=True)
    revt: Series[float] = pa.Field(nullable=True)
    ni: Series[float] = pa.Field(nullable=True)

    class Config:
        strict = "filter"
        coerce = True
        unique = ["gvkey", "fyear"]

class MdnaSchema(pa.DataFrameModel):
    gvkey: Series[str]
    fyear: Series[int] = pa.Field(ge=2010, le=2024)
    cik: Series[str]
    fdate: Series[pa.DateTime]
    mdna_text: Series[str] = pa.Field(str_length={"min_value": 5_000, "max_value": 300_000})
    char_count: Series[int] = pa.Field(ge=5_000, le=300_000)

    class Config:
        strict = "filter"
        coerce = True
        unique = ["gvkey", "fyear"]
```

`coerce = True` matters here, and it's worth seeing exactly why. Ask the `wrds` client for six columns and check what you actually get back:

```python
df.dtypes
# gvkey       string[python]
# datadate    string[python]     <- not a date!
# fyear                Int64     <- nullable integer, not int64
# tic         string[python]
# at                 Float64
# lt                 Float64
```

Two mismatches against the schema. `datadate` arrives as a **string**, not a `datetime` — the driver doesn't parse dates for you. And the numerics come back as pandas' *nullable* extension types (`Int64`, `Float64`, capital letters) rather than plain NumPy `int64`/`float64`, because SQL columns can be NULL and NumPy integers can't. A Parquet round-trip can shift them again. Without `coerce`, a schema declaring `Series[pa.DateTime]` and `Series[int]` fails on data that is completely fine. Coercion casts each column to the declared type as part of validation — and still fails loudly on the real problems, like an `fyear` that's missing or fractional or a `datadate` that isn't a parseable date at all.

> Don't take those dtypes on faith either — they depend on your `wrds` and `pandas` versions. `print(df.dtypes)` on your own pull is one line, and it's the difference between fixing a schema and guessing at one.

Wire each schema into the script that produces the file it describes: `pull_fundamentals.py` validates against `FundamentalsSchema` *before* writing, and `merge_mdna.py` validates the concatenated frame against `MdnaSchema` *before* writing `mdna.parquet`. Validating at the merge rather than in each shard is deliberate — a schema check is cheap, and one failure message about the whole panel is easier to act on than twenty identical ones scattered across compute-node logs. Re-run; if validation fails, you have a real problem to look at — don't paper over it by relaxing the schema.

**The bounds on `mdna_text` are the point of this schema, and they are bounded on both sides on purpose.** An extraction that returns 400 characters matched a table-of-contents line, not a section. One that returns 8 MB ran past Item 7A and swallowed the exhibits. Neither is an *error* as far as the parsing job is concerned — both return a perfectly good Python string, and both sail through any check that only asks whether extraction "succeeded." The §8b measurements are where the numbers come from: 5,000 and 300,000 characters bracket about 89% of a real S&P 500 panel, and the rows outside them are overwhelmingly junk rather than unusual firms.

Expect this schema to **fail the first time you run it**, rejecting roughly one row in nine. That is the schema doing its job. Resist the urge to widen the bounds until it passes — that converts a loud failure into a silent one, which is the exact trade you spent Module 5 learning not to make. Either improve the extraction, or drop the offending rows deliberately and say so in your `README.md`.

## 10. Decide what gets committed

WRDS itself permits academic, non-commercial use. The catch is the **underlying data vendors** — Compustat, CRSP, IBES, Audit Analytics — each have their own redistribution clauses on top of the WRDS umbrella, and several restrict reposting raw extracts even for academic users. Reading every vendor license to figure out what's allowed is more work than just keeping raw pulls off public repos.

So the default for this course is: **don't commit raw vendor data**. Partly that's a licensing call — some vendors are stricter than the WRDS umbrella, and default-private spares you reading each license. But the bigger reason is that the data file isn't worth committing: anyone reproducing your work — a grader, a teammate, future-you — has their own WRDS access and can re-run the pull from your script in a minute. A frozen Parquet snapshot is *less* flexible than the script that generated it.

What you commit:

- `src/pull_fundamentals.py` (local) and the three-stage MD&A pipeline — `src/build_mdna_manifest.py`, `src/parse_mdna_shard.py`, `src/merge_mdna.py`.
- `jobs/parse_mdna.sh` and `jobs/merge_mdna.sh` — the Grid Engine submission scripts. Half a dozen directives and one command each, and they are the difference between a reproducible run and "I forget how many shards I used."
- `AGENTS.md` — the WRDS access conventions from §4d, so a teammate's agent inherits the same rules.
- `src/schema.py` — the validation.
- A small **fixture** file for tests: `tests/fixtures/fundamentals_sample.parquet`, ~50 rows of synthetic-or-anonymized data for `pytest` to chew on.
- `README.md` describing how to reproduce.

What you do **not** commit:

- `data/raw/*.parquet` — already in `.gitignore`.
- `.pgpass` — already in `.gitignore`. Double-check.
- WRDS username, password, or any URL with credentials baked in.

Verify:

```bash
git status                   # nothing in data/raw/ should appear
git log --all -- '*.pgpass'  # should print nothing
```

## 11. Commit and push

**There is only one working tree.** Everything in this module was authored on your laptop — the local pull, the three MD&A scripts, the job script, the schemas — and the copy on the WRDS Cloud is a synced mirror with no Git history of its own. So there's one commit, from one place:

```bash
git add src/ jobs/ tests/ AGENTS.md pyproject.toml uv.lock README.md .gitignore
git status                   # eyeball one more time before committing
git commit -m "M8: WRDS pulls, grid job script, validation schemas"
git push
```

**Don't forget `uv.lock`.** It carries the `sec-parser` dependency you added in §8b; without it, a teammate cloning the repo and running `uv sync` on their own WRDS account gets `ModuleNotFoundError: sec_parser` on every one of their twenty array tasks and no obvious explanation of why.

That single-tree property is worth naming, because it's the second real benefit of the §4/§5 arrangement — after the process budget. With Remote-SSH you'd be maintaining two checkouts of the same repo on two machines, remembering which one has the newer `parse_mdna_shard.py`, and resolving the merge when you got it wrong. Sync-to-execute makes that class of mistake unrepresentable: the cloud has no commits to diverge with.

Now `main` on your laptop and on GitHub are in sync. This is the `main` that Modules 9–14 will each branch from — and **work on `main` directly is rare from here on out**: every model gets its own branch.

> **Reproducibility check.** What you just pushed has to be enough for a grader or teammate with their own WRDS account to reproduce `mdna.parquet` from scratch: the three scripts, the job script, the lockfile, and the `AGENTS.md` conventions. The Parquet itself is gitignored and stays out. If you're unsure, read your `README.md` as if you'd never seen the repo — can you get from clone to `qsub` without asking anyone a question?

### Letting the agent do this

You can also just tell your coding agent **"push all"** or **"commit and push the schema work"** — most agents (Claude Code, Cursor in agent mode, Copilot agent) will run `git status`, draft a commit message, stage, commit, push, and report back. For routine work, this is genuinely faster than typing the four lines above.

It's not a free pass, though. Two things you watch for, every time:

- **The staged file list, before you approve the commit.** Most agents pause and show what's staged; some quietly run `git add -A` and sweep in *everything*, including a forgotten `data/raw/foo.parquet`, a `.env`, or a notebook with credentials in cell output. A leaked WRDS password or API key in a `git push origin main` to a public repo is compromised within minutes — there's no "untoast the bread" command for it.
- **The commit message, before you approve it.** Vague messages ("update files", "fix stuff") aren't useful to a teammate, future-you, or a grader. If the agent's draft is bad, send it back — same brief-and-review loop as Module 6.

For Part 2 work going forward, **"push all" is fine on a feature branch** (low blast radius — anything wrong gets caught at the PR review in §6 of each subsequent module). For pushes to `main`, slow down. The blast radius is the public repo and your reputation as someone whose history doesn't contain accidents.

## You're done if…

- [ ] `https://github.com/<your-username>/ACCY575-wrds-data-analysis` exists, public, with `src/pull_fundamentals.py`, the three MD&A scripts, and both job scripts under `jobs/` visible.
- [ ] You SSHed into WRDS Cloud at least once with your password and saw `/wrds`, and you know your `ulimit -u`.
- [ ] `ssh -i ~/.ssh/wrds_ed25519 <user>@wrds-cloud-sshkey.wharton.upenn.edu 'hostname'` returns without asking for a password.
- [ ] `ssh -O check wrds` reports a running master, and `ssh wrds 'hostname'` returns in well under a second.
- [ ] `ssh wrds 'ls -l ~/accy575/'` shows `data` as a symlink into `/scratch/…`, so no large file ever lands in your 10 GB home.
- [ ] `AGENTS.md` (or your agent's equivalent) contains the WRDS access rules from §4d.
- [ ] `uv run python -m src.pull_fundamentals` produces `data/raw/fundamentals.parquet` locally.
- [ ] `qsub jobs/parse_mdna.sh` ran on the grid and all 20 tasks show `exit_status 0` in `qacct`.
- [ ] `qsub jobs/merge_mdna.sh` produced `data/raw/mdna.parquet` on the cloud, and you read its log rather than assuming it worked.
- [ ] You `rsync`'d `mdna.parquet` down to your local `data/raw/`.
- [ ] Both `pandera` schemas pass on their respective files.
- [ ] No raw WRDS data, no `.pgpass`, no credentials of any kind appear in `git log`.
- [ ] Your repo's `main` is the foundation Module 9 will branch off.
