# Setup document format

One human-focused markdown file **per OS**, kept outside the skill's repository. Default path: `~/setup-guides/<os>.md` (`macos.md`, `linux.md`, `windows.md`). If the user names another directory, use that for every document in the session. Never write these files inside the skill's repo.

Reproducing a setup on a new machine of the same OS means copying that OS's single file to it.

## Sections

Use these headings exactly, in this order when creating a document. When editing an existing document, match sections by heading text, not position.

- `## Overview` — the OS baseline and the machines running it: name, RAM, architecture, storage.
- `## Common` — setup shared by every machine on this OS: package managers, installed tools, shell and config customizations, credential references.
- `## Machine deviations` — one subsection per machine: `### <machine-name> — <RAM>, <arch>, <storage>`. List only what differs from `Common` (tools skipped or added because of RAM/architecture/storage, machine-specific paths, quirks).
- `## Fixes and quirks` — `symptom → cause → fix`, scoped to toolchain and configuration problems, so the next machine benefits.
- `## Deliberately skipped` — things the user chose not to install or do, and why.
- `## Open todos` — things still to do.
- `## Notes` — append-only, dated `YYYY-MM-DD` entries (install history, debugging trail).

## Line formats

Tool entry, inside `## Common` (or inside a machine subsection when it is a deviation):

```
- **<name>** — method: <brew|apt|winget|…>; version: <v>; install: `<exact command>`; notes: <text or URL>
```

Credential reference — a pointer to where the user keeps a secret, never the value:

```
- `<VAR_NAME>` → <user-managed location, e.g. "1Password item 'GitHub token'">
```

## Editing rules

1. **Merge, don't overwrite.** The user may edit any file by hand. Make targeted edits to the section concerned; show the edit before applying it. Preserve prose, ordering, and unknown sections verbatim.
2. **Deltas only.** A machine deviation subsection lists only differences from `Common`. Never copy `Common` into it. A tool that cannot run on one machine stays in `Common` and is listed as skipped in that machine's subsection.
3. **Maintain in place, except Notes.** Update an entry's version or notes where it lives. Only `## Notes` is append-only.
4. **Sibling mention.** Cross-OS content (git, editor) may be duplicated across OS documents. When you edit one, mention the matching entry in the sibling document so the user can keep them in step. Do not edit the sibling yourself.

## Hand-edited or damaged documents

- Match sections by heading text. A reordered document stays reordered.
- Preserve any section or paragraph you do not recognize, verbatim.
- If a standard section is missing, ask before recreating it; never silently re-add or delete anything.
- If no document exists for the OS, propose creating one from `setup-doc-template.md` and say what it will contain before writing.

## Secrets

Documents record where a secret lives, never what it is. Never write a token, password, or key into a document, a command, or an issue draft. If the user pastes one, do not repeat it back; ask where they keep it and record the reference form above. Never read `~/.ssh`, credential files, or keychains.

## Recording outcomes

After an approved batch runs (or after the user runs a step you showed them), fold the result into the document: success, failure, and output worth keeping. Privileged or interactive steps the user ran themselves are recorded too; ask for the outcome.
