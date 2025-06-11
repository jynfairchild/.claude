Please provide quick project context by reading existing documentation. Follow these steps:

1. Check if .claude/explore.md exists
   - If it doesn't exist, tell the user to run /user:explore first
   - If it exists, read it to understand the project structure and setup

2. Read .claude/CLAUDE.md for project-specific instructions and preferences

3. Read .claude/diary.md to understand:
   - Recent development sessions
   - Current status
   - Unresolved issues
   - Next steps from previous sessions

4. Check git status to see uncommitted changes and current branch

5. Look at recent git commits (last 3-5) to understand recent work

6. Run git diff to see current uncommitted changes

Based on all this information, provide a concise summary:
- What the project is
- What was being worked on recently
- Current state (any uncommitted work, failing tests, etc.)
- Suggested next steps based on diary entries and current state

This is a lightweight way to get back into context without re-scanning the entire codebase.