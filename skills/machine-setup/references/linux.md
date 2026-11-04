# Linux reference

Load when the conversation is about a Linux machine (including WSL, treated as its own machine under `linux.md`). All commands follow the action policy in `SKILL.md`: shown, offered, approved. Commands marked **user runs** need `sudo` or a password.

## Detecting the machine

Read-only; offer as one batched list.

```
cat /etc/os-release      # distro and version
uname -m                 # x86_64, aarch64, …
free -h                  # RAM
df -h /                  # storage
hostname                 # machine name for the document
echo $SHELL              # login shell
```

The distro decides the package manager. Only the families below are covered; anything else (Nix/NixOS, Alpine's `apk`, Gentoo, FreeBSD, …) is a **gap**: state it, never guess commands, and offer the gap flow.

## Package managers

**apt** (Debian, Ubuntu and derivatives)
- Install `sudo apt install <pkg>` (**user runs**); refresh `sudo apt update` (**user runs**).
- Inventory: `apt list --installed`, `apt-mark showmanual` (what was asked for), `dpkg -l <pkg>`.
- Outdated: `apt list --upgradable`.
- Distro packages are often old; third-party repos and `.deb` downloads need a recorded URL and key location in `Tools` notes.

**dnf** (Fedora, RHEL family)
- Install `sudo dnf install <pkg>` (**user runs**).
- Inventory: `dnf list --installed`, `dnf history userinstalled`.
- Outdated: `dnf check-update` (exits with code 100 when updates exist; that is not a failure).

**pacman** (Arch family)
- Install `sudo pacman -S <pkg>` (**user runs**); inventory `pacman -Qe`; outdated `checkupdates` (from pacman-contrib; `pacman -Qu` only reflects the last sync). Never partial-upgrade (`-Sy` then install without `-u`).

**zypper** (openSUSE): install `sudo zypper install <pkg>` (**user runs**); inventory `zypper se --installed-only`; outdated `zypper list-updates`.

**snap / flatpak**: sandboxing changes file and PATH access; note the method per tool. Inventory `snap list`, `flatpak list`.

**mise** and **pipx**: same role as on macOS. Install via the distro or the project's installer; inventory `mise ls`, `pipx list`. Needs shell activation (`eval "$(mise activate bash)"` in `~/.bashrc`).

## Shell and config

- bash is the usual default; zsh is a common choice. Check `echo $SHELL`.
- bash: `~/.bash_profile` or `~/.profile` runs for **login** shells, `~/.bashrc` for **interactive non-login** shells. Many distros source `.bashrc` from `.profile`; verify before assuming. Put `PATH` in the login file, aliases and activation hooks in `.bashrc`.
- Terminal emulators and display managers differ in whether they start login shells; this is a frequent cause of "works in one terminal, not in another".
- Config edits are shown in full, applied only with approval, and recorded under `Shell and config`.

## Services and quirks

- **systemd user units** (`systemctl --user status <unit>`, `~/.config/systemd/user/`) for per-user services; system units need `sudo` (**user runs**).
- **PATH in non-interactive shells** (cron, systemd, IDE-launched tools) does not include what `~/.bashrc` adds. Symptom: works in a terminal, "command not found" elsewhere.
- **sudo and PATH**: `sudo` resets `PATH`; tools installed in `~/.local/bin` or by mise are not found under `sudo`.
- **Old distro packages**: the apt/dnf version may lag; prefer mise or the vendor's repo when a recent version matters, and record the choice.
- **Permissions of global installs**: avoid `sudo pip`/`sudo npm -g`; use pipx or a version manager.

## Recording notes

Inventory for the document: distro and version; manually installed packages (`apt-mark showmanual`, `dnf history userinstalled`, `pacman -Qe`); snaps/flatpaks; mise runtimes; pipx apps; which shell files were customized (ask about each); third-party repos and manual `.deb`/tarball installs with their sources. Ask what was deliberately left out.
