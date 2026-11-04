# Windows reference

Load when the conversation is about a Windows machine. The harness often runs on another OS, in which case nothing can be executed on the PC: show every command for the user to run themselves, ask them to paste the output, and interpret it. Commands marked **user runs** need an elevated (admin) shell or a UAC prompt. Commands are PowerShell unless noted.

## Detecting the machine

Read-only; offer as one batched list for the user to run and paste back.

```
winver                                                    # GUI: Windows version
$PSVersionTable.PSVersion                                 # PowerShell version
$env:PROCESSOR_ARCHITECTURE                               # AMD64, ARM64
Get-CimInstance Win32_ComputerSystem | Select Name, TotalPhysicalMemory
Get-PSDrive -PSProvider FileSystem                        # storage: used/free per drive
winget --version                                          # is winget present
```

`TotalPhysicalMemory` is bytes. Use these for `Overview` and for filtering a reproduction by RAM, architecture, and storage.

## Package managers

**winget** — ships with current Windows 10/11 (App Installer).
- Install `winget install --id <Publisher.Name> -e`; search `winget search <name>` and confirm the package ID before recommending it. IDs are not guessable, so flag any ID you are unsure of as needing verification.
- Inventory `winget list`; outdated/upgrade: `winget upgrade` (list), `winget upgrade --id <id>`.
- User-scope installs need no admin; machine-scope installers trigger **UAC** (**user runs**).

**Chocolatey** — install per chocolatey.org (**user runs**, elevated). Inventory `choco list`; outdated `choco outdated`; install `choco install <pkg>` (elevated).

**Scoop** — user-scope, no admin: inventory `scoop list`; outdated `scoop status`; install `scoop install <pkg>`. Buckets add package sources (`scoop bucket list`).

**mise / pipx**: available through scoop/winget; same roles as elsewhere (`mise ls`, `pipx list`). Activation goes in `$PROFILE`.

Prefer one manager per tool and record which. Mixing winget and scoop for the same tool causes duplicate installs and PATH shadowing.

## Shell and config

- Windows ships PowerShell 5.1; PowerShell 7 (`pwsh`) is a separate install. Record which the user uses.
- `$PROFILE` is the per-user startup script (path: `echo $PROFILE`). The equivalent of `.zshrc`/`.bashrc`. It may not exist yet: `New-Item -Path $PROFILE -ItemType File -Force` (shown first).
- Execution policy can block `$PROFILE` and scripts: check `Get-ExecutionPolicy -List`. Changing it is a security decision: show it, explain it, and let the user run `Set-ExecutionPolicy -Scope CurrentUser RemoteSigned` themselves if they agree.
- Windows Terminal profiles live in its `settings.json`; record customizations the user cares about.

## WSL

WSL distros are Linux machines: guide them with the Linux reference's knowledge (SKILL.md loads it) and record them in `linux.md` as their own machine subsection (name, resources). Note the Windows side separately in `windows.md`. Files under `/mnt/c` are slow and permission-odd; keep projects in the Linux filesystem. Check with `wsl -l -v` (version 1 vs 2).

## Quirks

- **UAC / elevation**: anything needing admin is shown for the user to run; ask for the outcome and record it.
- **PATH**: edit via System Properties → Environment Variables, or `[Environment]::SetEnvironmentVariable(...)`. Avoid `setx` for PATH: it truncates at 1024 characters and flattens the user/system split. New values appear only in new shells.
- **Line endings**: `core.autocrlf` interplay between Windows git and WSL causes whole-file diffs; record the chosen setting.
- **Long paths** (>260 chars) may need the `LongPathsEnabled` registry setting (**user runs**, admin).
- **Antivirus/SmartScreen** may block unsigned installers; the user decides, never suggest disabling protection wholesale.

## Recording notes

Inventory for the document: Windows version; `winget list` filtered to what the user installed on purpose, `choco list`, `scoop list`; PowerShell version and `$PROFILE` contents (ask what is customized); WSL distros; execution policy chosen. Ask what was deliberately left out.
