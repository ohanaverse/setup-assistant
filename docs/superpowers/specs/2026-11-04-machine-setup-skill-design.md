# Machine Setup Assistant — Skill Design

Date: 2026-11-04
Status: awaiting user approval; both builder-check questions (2026-11-04) answered and incorporated

## Intent and why

A user setting up a new machine, refreshing an old one, or debugging a broken
one does the same kind of work repeatedly, and the knowledge of how their
machine is set up lives in their head and in shell history until it is lost.

This skill is an interactive expert of machine setup on **macOS, Linux, and
Windows**. It prompts and guides the user through setup, debugging, recording
an existing setup, and reproducing a recorded setup on another machine. Its
only artifact — and the only thing it writes on its own — is a set of
human-focused markdown **setup documents, one per OS**, kept outside this
repository.

Unlike the discarded first attempt, there is **no script layer, no
machine-readable manifest, and no test/eval harness**. The expertise lives in
the skill's own markdown; the record lives in the setup documents; every other
action is proposed by the skill, shown in full, and executed only with the
user's explicit permission.

## Goals

- Guide the user through setting up a machine: propose the tools and steps
  appropriate to the OS, show the exact commands, offer to run them, and act
  only on explicit permission.
- Help debug problems: gather diagnostic output, diagnose, propose fixes under
  the same show-and-offer rule.
- Record an existing machine: inventory what is installed and ask about the
  things a scan cannot see (shell customizations, quirks, deliberate omissions),
  maintaining that OS's setup document as the durable record.
- Reproduce a setup on another machine: read the target OS's setup document
  there, apply that machine's deviations, compare against the machine, and
  walk the user through the equivalent setup — including proposing a new
  document when the new machine's OS has none yet.
- Be genuinely expert across macOS, Linux, and Windows: package managers,
  version managers, shell and config conventions, and common OS quirks.
- Be self-improving: when a gap in its own knowledge shows up, notify the user
  and offer to address it now in a new worktree, file a GitHub issue for later,
  or ignore it.

## Non-goals

- Not a package manager or unattended automation service. Nothing runs on a
  schedule; no command runs that the user has not seen.
- Not a secrets manager. Secrets stay where the user keeps them; the document
  records where, never what.
- Not a dotfiles manager. The skill may propose config-file edits, shown in
  full, applied only with explicit approval, and recorded in the document.
- Not a team or fleet provisioning tool. The subject is the user's personal
  machines.
- Not general OS troubleshooting. The focus is installing and configuring
  tools for development and other purposes; everyday OS faults (Wi-Fi,
  printers, hardware, performance, boot problems) are out of scope (user,
  2026-11-04). Debugging means toolchain and configuration problems.
- Not a replication of the discarded v1 machinery: no helper scripts, no
  `tools.json`, no Brewfile generation, no improver skill, no eval harness.

## Anti-goals

These would make the skill a failure even if it technically worked:

- Running a command the user has not seen and approved.
- Reading secret stores (`~/.ssh`, credential files, keychains) or storing any
  secret, token, or key material in the setup documents.
- Writing the setup documents, or anything else generated about the user's
  machine, into this repository.
- Committing, pushing, or merging skill changes from the self-improvement flow
  without separate explicit approval for each step.
- Pretending to know an OS or tool it does not: a gap is always surfaced, never
  papered over with a plausible-sounding guess.

## Constraints

- The skill lives in this repo at `skills/machine-setup/`.
- The setup documents live outside the repo, one per OS, default
  `~/setup-guides/<os>.md` (`macos.md`, `linux.md`, `windows.md`), or in any
  other directory the user names. They are plain markdown the user is free to
  edit by hand; the skill merges manual edits rather than rewriting over them.
- Docs-only: no scripts and no runtime dependencies beyond the agent harness.
  `gh` is optional and only used for the issue-filing option.
- A fresh agent session with no memory of the user's machine must be able to
  run every flow, using only the skill and the setup documents.
- The harness may be running on a different OS than the one being discussed
  (e.g. guiding Windows setup from macOS). Guidance must work conversationally
  even where the skill cannot execute anything on the target machine.
- Both the skill files and the setup documents are markdown.

## Approaches considered

**1. One file — everything in `SKILL.md`.** Workflow, OS expertise, and document
format in a single file.
Trade-off: smallest surface, but every session drags the whole body into
context, and three OSes' knowledge interleaved with workflow gets messy fast.

**2. `SKILL.md` plus references, loaded on demand.** `SKILL.md` holds the
interaction workflow; separate references hold macOS, Linux, and Windows
expertise plus the document format. Only the OS-relevant reference is read
when the conversation is about that OS.
Trade-off: more files, but each stays small, OS depth can grow independently,
and it follows the standard skill convention.

**3. Template shipped as a fixture.** Option 2 plus a canonical setup-document
template the skill starts from.
Trade-off: consistent document shape across machines and sessions; costs a
little rigidity if the user's setup does not fit — the template is a starting
point, not a straitjacket.

**Chosen: 2 + 3**, as decided with the user. The template is itself just
another reference file, so it adds no machinery.

## The parts of the how

### Layout

```
skills/machine-setup/
  SKILL.md                        # interaction workflow, action policy, gap flow
  references/
    macos.md                      # macOS expertise: Homebrew, mise, defaults, quirks
    linux.md                      # Linux expertise: apt/dnf, systemd, rc files, quirks
    windows.md                    # Windows expertise: winget/choco/scoop, PowerShell, WSL
    debugging.md                  # toolchain triage: symptom -> gather -> isolate -> fix
    setup-doc-format.md           # section-by-section format and editing rules
    setup-doc-template.md         # canonical starting document
README.md                         # what the skill is and how to install it
docs/superpowers/...              # specs and plans for this skill itself
```

### Action policy (the trust rule)

- Every action the skill proposes is shown first: the exact command or edit,
  what it does, and anything at risk.
- The skill then offers to run it. It runs only what the user explicitly
  approved in that exchange — approval covers the shown list, not future lists.
- Anything privileged, interactive, or GUI-driven (`sudo`, password prompts,
  installer wizards, Windows UAC) is shown for the user to run themselves,
  since the skill cannot complete it anyway; the outcome is recorded either
  way.
- Read-only inspection commands (`sw_vers`, `brew list`, `dpkg -l`, `uname`,
  registry queries) follow the same rule: shown, offered, approved. They are
  never silently run.
- After an approved batch runs, results (success, failure, output worth
  keeping) are folded into the setup documents.

### The four flows

The skill does not expose commands; it recognizes the flow from the
conversation and runs it.

1. **Setup guidance.** User describes the goal → skill identifies the OS and
   what is already present (via approved inspection) → proposes steps one
   decision at a time, each with its exact command → show-and-offer → record
   outcomes in the document.
2. **Recording an existing machine.** Inventory via approved read-only
   commands (package managers, installed tools, versions), then interview the
   user for what a scan cannot see: shell and config customizations, why an
   odd tool is there, what was deliberately left out, known quirks. The
   machine's OS document is created or updated from both halves, with anything
   specific to this machine going into its deviation subsection rather than
   `Common`.
3. **Reproducing on another machine.** Read the target OS's setup document on
   the new machine → inspect the target (approved read-only commands) →
   present the difference: already present / missing / not applicable here,
   filtered through the new machine's specs (RAM, arch, storage), with a
   substitution proposal (`brew install x` → `winget install x`) when no
   document exists yet for that OS → walk through the missing parts as in
   flow 1 → add a deviation subsection for the new machine to that OS's
   document.
4. **Debugging (toolchain and configuration only — see non-goals).** User
   reports a symptom → skill asks for the specific
   diagnostic outputs (shown, user-run or approved) → narrows causes using
   `references/debugging.md` and the OS reference → proposes the fix under the
   action policy → records the symptom and fix in that OS document's fixes
   section so the next machine benefits.

### The setup documents

One human-focused markdown file **per OS** at `~/setup-guides/<os>.md`
(`macos.md`, `linux.md`, `windows.md`), or wherever the user points. This
topology is the user's answer to the builder check (2026-11-04): most steps
for an OS are shared, while machines differ in deviations — e.g. macOS setup
on a 64GB and a 16GB machine is mostly the same, but some tools cannot run on
the 16GB machine. Each file:

- **Overview** — the OS baseline and the machines running it (name, key
  specs: RAM, architecture, storage).
- **Common** — setup shared by every machine on this OS: package managers,
  installed tools (name, method, version, the reproduction command,
  notes/URL for non-standard tools), shell and config customizations,
  credential references (variable name → the user-managed path holding the
  value).
- **Machine deviations** — one short subsection per machine, keyed by machine
  name and specs, listing only what differs from `Common`: tools skipped or
  added because of RAM/architecture/storage, machine-specific paths, quirks.
  Deltas only, never a full copy of `Common`.
- **Fixes and quirks** — symptom → cause → fix, scoped to toolchain and
  configuration, so the next machine benefits.
- **Deliberately skipped**, **open todos**, and append-only chronological
  notes wherever chronology matters (install history, debugging trail);
  everything else is maintained in place.

Reproducing on a new machine means copying that OS's file to it. Rules: never
secrets, only references to where the user keeps them; the skill never reads
secret stores; the user may edit any of these files freely and the skill
merges rather than overwrites; the documents never live in this repo.

### Self-improvement: the gap flow

A **gap** is something the skill should know or do but does not: an unknown OS
or package manager, a tool missing from its references, a fix it cannot
diagnose confidently, or a workflow step it cannot perform.

When a gap surfaces the skill:

1. States it briefly and plainly (never guesses past it).
2. Offers three choices:
   - **Address now** — create a worktree of this repo
     (`.worktrees/<slug>`, branch `improve/<slug>`), show the proposed edit to
     the skill's files, apply it on approval, then show the diff. Commit,
     push, and PR are each offered separately and only happen with their own
     explicit approval. Nothing auto-merges.
   - **File a GitHub issue** — draft a sanitized title and body (no secrets,
     home paths, usernames, or hostname), show the draft and the exact
     `gh issue create` command, run it only on approval. If `gh` is missing,
     show the draft for the user to file by hand.
   - **Ignore** — drop it, no record, no residue.
3. Records nothing about the gap in the setup documents — those are about
   the user's machines, not this skill's internals.

Changes from this flow are limited to this repository.

## Verification

Docs-only means there is no script layer to test, and the discarded eval
harness is deliberately not coming back. Verification is a manual walkthrough
executed by the builder against each flow (setup, record, reproduce, debug,
each gap option) using a fixed scenario script, plus a link/format check that
every cross-reference inside the skill resolves. Results are reported in the
implementation PR/summary for the user to judge.

## What is left to the builder

- Exact wording of `SKILL.md` and each reference, following skill-authoring
  conventions (frontmatter, description, progressive disclosure).
- The depth and contents of each OS reference: which package managers and
  tools are covered (Homebrew/mise/npm/pipx; apt/dnf and friends;
  winget/Chocolatey/Scoop, PowerShell profile, WSL) and which quirks merit
  inclusion.
- The exact text of `setup-doc-template.md`.
- The scenario script used for the manual verification walkthrough.
- README install instructions for the target harnesses.

## Decisions made by the assistant (not by the user)

Each is cheap to override:

1. **Skill name `machine-setup`** (repo is `setup-assistant`).
2. **One document per OS, with per-machine deviation sections** — the user's
   answer (2026-11-04) to the builder check. Deviations are deltas only, never
   a full copy of `Common`, keyed by machine name and specs (RAM, arch,
   storage).
3. **Document default paths `~/setup-guides/macos.md`, `linux.md`,
   `windows.md`**, overridable by the user naming another directory in
   conversation. Reproducing on a machine of a recorded OS = copying that
   OS's single file.
4. **Approval granularity is the shown list**: the user approves the batch of
   commands they just reviewed, not each command individually, and never a
   future batch.
5. **The secrets policy from v1 is carried over unchanged** (reference forms
   only, never read secret stores, never write secret values) — it was a
   user-driven requirement there and safety-relevant here.
6. **No test harness, no CI, no Makefile.** Verification is the manual
   walkthrough described above.
7. **Update maintenance is not a separate flow**: checking for upgrades is
   ordinary guidance under flow 1 (show `brew outdated` etc., offer, record).
8. **Windows support is conversational-first**: where the harness cannot
   execute on Windows, every flow still works by showing the user commands to
   run themselves.
9. **Gap flow on a machine without this repo**: the skill offers to clone
   `ohanaverse/setup-assistant` first; if the user declines, it falls through
   to the issue/ignore options. Builder-check resolution (minor, assistant's
   call).
10. **Cross-OS content may be duplicated** across OS documents (e.g. git or
    editor setup appears in both `macos.md` and `linux.md`); when the skill
    edits one, it mentions the sibling entry so the user can keep them in
    step. No shared cross-OS file — that would reintroduce machinery.
