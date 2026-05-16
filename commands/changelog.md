---
description: Create a changelog entry for this coding session
allowed-tools: Bash(ls:*), Bash(mkdir:*)
argument-hint: minor | major | X.Y.Z
---

Please write a changelog session entry for our coding work. Follow these steps:

1. Look for `.changelog/` directory at the **root** of the project (not in subdirectories like frontend/ or backend/)
2. If it doesn't exist, create the directory at the project root:
   - This directory catalogs versioned changes made during coding sessions
   - Details the release/version identifier, date/time, and what was changed

3. Determine the new version number from `$ARGUMENTS`:
   - List existing entries in `.changelog/` (e.g. `ls .changelog/`) and find the highest existing `vX.Y.Z.md` file. Parse it as three integers `X`, `Y`, `Z`.
   - If `$ARGUMENTS` is `minor` → bump the **third** digit: `vX.Y.(Z+1)` (e.g. `v0.3.8` → `v0.3.9`).
   - If `$ARGUMENTS` is `major` → bump the **second** digit and reset the third to `0`: `vX.(Y+1).0` (e.g. `v0.3.8` → `v0.4.0`). Always keep the trailing `.0` — never write `v0.4` without it.
   - If `$ARGUMENTS` looks like an explicit version (e.g. `1.2.3` or `v1.2.3`), use it verbatim as `vX.Y.Z`.
   - If `$ARGUMENTS` is empty or anything else, ask the user whether they want `minor` or `major` before proceeding.
   - If `.changelog/` is empty or has no `vX.Y.Z.md` files, start at `v0.1.0` for either `minor` or `major`.

4. Create a new entry file at `.changelog/vX.Y.Z.md` using the computed version. Don't append to existing files — always a new file.

5. Review modified files, our conversation, and any error messages to compile a comprehensive list of what was accomplished.

Format the entry like this:

```markdown
## vX.Y.Z – Session Title
**Date:** [Current date and time with timezone]

### Changes Made:
- [Change 1]
- [Change 2]
- [etc...]

### Current Status:
[Brief description of what's working and what isn't]

### Unresolved Issues:
- [Any blockers or problems not yet solved]
- [Or "None" if everything is resolved]

### Commit Message:
[Concise title]

- [Bullet point summary of changes, max 10 points]
```
