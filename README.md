# setup-assistant

An interactive skill that helps you set up, record, reproduce, and debug the developer toolchain on your macOS, Linux, and Windows machines.

## What it does

- **Setup guidance** — proposes tools and steps for your OS, shows the exact commands, and offers to run them.
- **Record a machine** — inventories what is installed and interviews you about what a scan cannot see (shell customizations, quirks, deliberate omissions).
- **Reproduce a setup** — reads the setup document for an OS on a new machine and walks you through the difference, filtered through that machine's RAM, architecture, and storage.
- **Debug** — toolchain and configuration problems only (command not found, wrong version, failed installs). Not Wi-Fi, printers, hardware, or performance.

The skill lives in [`skills/machine-setup/`](skills/machine-setup/SKILL.md). It is markdown only: no scripts, no dependencies beyond your agent harness.

## The setup documents

The skill's only artifact is one human-focused markdown **setup document per OS**: `~/setup-guides/macos.md`, `linux.md`, `windows.md` (or any directory you name). They live **outside this repository**, are plain markdown you can edit freely, and hold shared setup plus a short deviations section per machine. They record *where* secrets live, never the secrets themselves. To reproduce a setup on a new machine of the same OS, copy that OS's file to it.

## The trust rule

Every command or edit is shown in full first, then offered. The skill runs only what you approved in that exchange, and never a later list. Privileged or interactive steps (`sudo`, UAC, installer wizards) are shown for you to run yourself.

## Install

The skill is plain markdown and works with any agentic harness (coding agents, chat assistants, or your own). It assumes only that the agent can read the files and talk to you; running commands and writing files are optional, and without them the skill works conversationally (you run the commands and paste output; it hands you the document text to save).

- **Harnesses with a skills directory:** copy `skills/machine-setup/` into the folder your harness loads skills from (symlink it if your harness follows symlinked skill directories), e.g. `cp -r skills/machine-setup <your-harness-skills-dir>/machine-setup`. For example, on Pi that folder is `~/.pi/skills` (user) or `.pi/skills` (project); `~/.agents/skills/` and `.agents/skills/` also work, and the skill can be distributed as a Pi package (npm or git). `SKILL.md` has standard `name`/`description` frontmatter and loads its `references/` files on demand.
- **Harnesses that read an instructions file** (an `AGENTS.md`-style file or a project/system prompt): point it at `skills/machine-setup/SKILL.md`, or paste that file in; keep `references/` alongside it so relative links resolve.
- **Chat-only:** paste `SKILL.md` as the instructions and paste a reference file when the skill asks for it.

Optional: the [`gh`](https://cli.github.com/) CLI, used only if you choose to file an issue when the skill finds a gap.

## Self-improvement

When the skill hits something it does not know (an unfamiliar package manager, a tool missing from its references), it says so and offers three choices: **address it now** in a new worktree of this repo (commit, push, and PR each need your separate approval), **file a GitHub issue** with a sanitized draft, or **ignore** it.

## Verification

This is a docs-only skill, so it is checked by scripted walkthrough scenarios in [`docs/superpowers/verification/scenarios.md`](docs/superpowers/verification/scenarios.md), with results in `results.md` beside it. Design and plan are under `docs/superpowers/`.
