# macOS reference

Load when the conversation is about a Mac. All commands follow the action policy in `SKILL.md`: shown, offered, approved. Commands marked **user runs** need `sudo`, a password, or a GUI.

Only the package managers and OS families described here are covered; anything else is a **gap**: state it, never guess commands, and use the gap flow in `SKILL.md`.

## Detecting the machine

Read-only; offer as one batched list.

```
sw_vers                  # macOS version
uname -m                 # arm64 (Apple Silicon) or x86_64 (Intel)
sysctl hw.memsize        # RAM in bytes
df -h /                  # storage
scutil --get ComputerName   # machine name for the document
echo $SHELL              # login shell (zsh by default)
```

Use these for `Overview` and for filtering a reproduction by RAM, architecture, and storage.

## Package managers

**Homebrew** — the primary installer.
- Location depends on architecture: `/opt/homebrew` on Apple Silicon, `/usr/local` on Intel. Check with `command -v brew`.
- Install is an interactive script from brew.sh (**user runs**; it prompts for the password). Afterwards it prints a `shellenv` line to add to `~/.zprofile`; the line is part of the setup and belongs in the document.
- Inventory: `brew list --versions`, `brew list --cask`, `brew leaves` (what the user asked for, minus dependencies), `brew tap`.
- Outdated: `brew outdated`; upgrade `brew upgrade <name>`.
- Never run `brew` with `sudo`; it refuses and it breaks permissions. `sudo` applies only to genuine installers (`.pkg`), shown for the user to run.
- Formulae are CLI tools (`brew install git`); casks are GUI apps (`brew install --cask <app>`). Casks may trigger Gatekeeper prompts.

**mise** — runtime/version manager (`brew install mise`). Inventory `mise ls`. Needs shell activation: `eval "$(mise activate zsh)"` in `~/.zshrc`. Without it, tools are not on `PATH`.

**npm global** — `npm ls -g --depth=0`. Prefer a version-manager-owned Node; avoid `sudo npm -g`.

**pipx** — `pipx list`. Install CLI Python tools in isolation; `brew install pipx` then `pipx ensurepath`.

## Shell and config

- zsh is the default shell. `~/.zprofile` runs for login shells (Terminal opens login shells); `~/.zshrc` runs for interactive shells. Put `PATH` setup (Homebrew `shellenv`) in `~/.zprofile`, aliases and `mise activate` in `~/.zshrc`.
- `PATH` order matters: Homebrew's `bin` must precede `/usr/bin` for brew-installed tools to shadow the system's.
- Config edits are shown in full and applied only with approval, then recorded under `Shell and config`.

## Quirks

- **Xcode Command Line Tools**: many tools need them (`xcode-select --install`, **user runs**, GUI). `xcode-select -p` shows whether installed.
- **Rosetta 2** on Apple Silicon for x86_64-only binaries: `softwareupdate --install-rosetta` (**user runs**, `sudo`/license).
- **Gatekeeper**: unsigned apps are blocked; the user approves in System Settings → Privacy & Security. Never suggest disabling Gatekeeper wholesale.
- **`defaults write`** changes app/system preferences; each is shown with what it changes and how to revert (`defaults delete`). Record them in `Shell and config`.
- **Homebrew not on PATH after install** is the most common first-hour problem: the `shellenv` line is missing from `~/.zprofile`.
- Intel vs Apple Silicon Homebrew installs coexisting can shadow each other; check `which -a brew`.

## Recording notes

Inventory for the document: macOS version; `brew leaves`, casks, taps; mise runtimes; npm globals; pipx apps; shell files that were customized (ask the user about each); `defaults` settings they care about. Ask what was deliberately left out.
