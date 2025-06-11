Please help organize Claude-related files by moving them into the .claude directory. Follow these steps:

1. Check if a CLAUDE.md file exists in the project root
2. If it exists:
   - Create .claude directory if it doesn't exist
   - Move CLAUDE.md to .claude/CLAUDE.md
   - Verify the move was successful
   - Confirm the root directory no longer has CLAUDE.md

3. Also check for any other Claude-related files in the root that should be moved:
   - Look for files like claude.md, CLAUDE.json, claude_config.*, etc.
   - Move any found files to .claude/

4. After moving files, update any references:
   - Check if any files reference the old CLAUDE.md location
   - Update paths from "CLAUDE.md" to ".claude/CLAUDE.md"

5. Report what was moved and any issues encountered

Note: Be careful not to break any existing functionality that might depend on CLAUDE.md being in the root directory.