# Setup document template

A starting point, not a straitjacket. Copy to `~/setup-guides/<os>.md` (or the directory the user names), replace the placeholders, and delete the example lines. Adapt freely where the user's setup does not fit.

````markdown
# <OS> setup

## Overview
<!-- OS baseline, e.g. "macOS 15". Then one line per machine. -->
Baseline: <OS and version>
Machines: <name> — <RAM>, <arch>, <storage>

## Common
<!-- Setup shared by every machine on this OS. -->

### Package managers
- **<manager>** — method: <install method>; version: <v>; install: `<command>`; notes: <text>

### Tools
- **git** — method: brew; version: 2.47.0; install: `brew install git`; notes: —

### Shell and config
- <customization, which file, why>

### Credential references
<!-- Where secrets live. Never the values. -->
- `GITHUB_TOKEN` → 1Password item "GitHub token"

## Machine deviations
<!-- Deltas from Common only. One subsection per machine. -->

### <machine-name> — <RAM>, <arch>, <storage>
- Skipped: <tool> — needs more RAM than this machine has
- Added: <tool> — <why>

## Fixes and quirks
<!-- symptom → cause → fix -->
- `node: command not found` after install → shell not reading mise shims → add `eval "$(mise activate zsh)"` to `~/.zshrc`

## Deliberately skipped
- <thing> — <why>

## Open todos
- <thing still to do>

## Notes
<!-- Append-only. -->
- YYYY-MM-DD: <what happened>
````
