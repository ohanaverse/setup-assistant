# Debugging toolchain and configuration problems

Load for flow 4. Scope: installing and configuring tools. Everything here follows the action policy in `SKILL.md`: diagnostics are shown, offered, and approved (or run by the user and pasted back); fixes likewise.

## Triage loop

1. **Symptom.** Get the exact message, the command that produced it, and what changed recently (an install, an upgrade, a new shell, a new machine).
2. **Gather.** Ask for the smallest set of targeted read-only diagnostics that discriminate between the likely causes below. One batched shown list.
3. **Isolate.** From the outputs, name the single most likely cause and what evidence supports it. If the evidence does not discriminate, say so and ask for one more targeted check; do not state a guess as fact.
4. **Fix.** Propose the exact command or edit, what it changes, and how to undo it. Apply only on approval.
5. **Record.** Show the `symptom → cause → fix` entry together with the fix list. Apply it to `Fixes and quirks` in that OS's setup document (default `~/setup-guides/<os>.md`, or the directory the user named; create the section if absent, otherwise merge) once the user confirms the fix worked. That confirmation is the trigger; no second prompt. Note the machine if the fix is machine-specific.

## Common toolchain symptoms

Each entry: symptom → likely causes (most likely first) → discriminating command.

- **`<tool>: command not found` right after install** → (1) shell not reloaded; (2) install dir not on `PATH` (e.g. Homebrew `shellenv` or `mise activate` missing from the shell startup file); (3) installed under a different user/scope → `command -v <tool>`, `echo $PATH`, check the startup file for the activation line.
- **Wrong version runs** → (1) a system or older copy earlier on `PATH`; (2) version manager not activated in this shell; (3) a per-directory version file (`.tool-versions`, `.nvmrc`) pinning something else → `which -a <tool>` (`where.exe <tool>` on Windows), `<tool> --version`, `mise ls` / `mise current`.
- **Shim exists but fails** → the version manager has no version installed for this directory → `mise ls`, `mise current`.
- **Works in a terminal but not in an IDE, cron, or systemd** → non-interactive environment lacks interactive `PATH` additions → compare `echo $PATH` there vs. in the terminal; fix `PATH` in the login-level file.
- **Permission denied on global install** → global prefix owned by root (e.g. system Node with `npm -g`) → `npm config get prefix`; fix by using a version manager or a user-owned prefix, never `sudo npm -g`.
- **Install fails with TLS/proxy errors (`ECONNRESET`, certificate errors)** → corporate proxy or custom CA → targeted config queries only: `npm config get registry`, `npm config get proxy`, `npm config get https-proxy`, `git config --get http.proxy`, `pip config debug` is **not** safe if it prints index URLs with credentials, so prefer `pip config get global.index-url` after warning the user it may contain credentials and asking them to redact any `user:pass@` before pasting.
- **Package manager conflicts / duplicate installs** → the same tool installed by two managers → `which -a`, then inventory each manager.
- **Homebrew works but a brew tool is the wrong architecture (Apple Silicon)** → Intel brew under `/usr/local` shadowing `/opt/homebrew` → `which -a brew`, `uname -m`, `file $(which <tool>)`.

## Diagnostics to request

Prefer these targeted, read-only commands:

| Need | macOS/Linux | Windows (PowerShell) |
|---|---|---|
| Where is it | `command -v <tool>`, `which -a <tool>` | `Get-Command <tool> -All` |
| Version | `<tool> --version` | `<tool> --version` |
| PATH | `echo $PATH` | `$env:PATH -split ';'` |
| Shell files | `grep -n '<keyword>' ~/.zshrc ~/.zprofile ~/.bashrc ~/.profile` | `Select-String <keyword> $PROFILE` |
| Version manager | `mise ls`, `mise current` | same |
| Package state | `brew list --versions <pkg>`, `dpkg -l <pkg>`, `dnf list installed <pkg>` | `winget list <id>` |

**Never request** dump-everything commands that can leak secrets: `env`, `printenv`, `set` (without a filter), `git config -l`, `npm config list`, reading `~/.npmrc`, `~/.pypirc`, `~/.aws/credentials`, `~/.git-credentials`, `~/.ssh/*`, or keychain contents. If the answer requires a value that might be secret, ask a yes/no or targeted question ("is a proxy set? what is the host only?") and tell the user to redact credentials before pasting.

## Out of scope

Everyday OS faults: Wi-Fi, printers, hardware, performance, boot problems, OS update failures unrelated to toolchains. Decline plainly, propose no diagnostics, and say what you can help with.

## When to raise a gap

If the symptom involves an OS, package manager, or tool the references do not cover, or you cannot discriminate between causes with the available diagnostics, say so plainly and use the gap flow in `SKILL.md`. Never fill the hole with a plausible-sounding guess.
