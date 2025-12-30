---
description: Create a changelog entry for this coding session
allowed-tools: Bash(ls:*), Bash(mkdir:*)
argument-hint: [version]
---

Please write a changelog session entry for our coding work. Follow these steps:

1. Look for `.changelog/` directory in the current project
2. If it doesn't exist, create the directory:
   - This directory catalogs versioned changes made during coding sessions
   - Details the release/version identifier, date/time, and what was changed

3. Create a new entry file in that directory:
   - Filename format: `vX.Y.Z.md` (e.g., v0.6.0.md)
   - If the user provided a version as an argument, use: `v$ARGUMENTS.md`

4. Review modified files, our conversation, and any error messages to compile a comprehensive list of what was accomplished

5. Create the new entry as a separate file (don't append to existing files)

Format each entry like this:

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
