Please create a git commit using the commit message from the most recent changelog session entry. Follow these steps:

1. Check if `.changelog/` exists.
2. If it exists, determine the most recent entry file (highest semantic version or latest modified file if versions are tied).
3. Extract the "Commit Message:" section from that entry.
4. Run `git status` to check for untracked files.
5. If there are untracked files, run: `git add -A`.
6. Then commit with: `git commit -m "[extracted commit message]"` 
6a. When committing, pass the entire changelog message in a single -m argument (including all bullet points separated
  by \n). Don’t chain multiple -m flags.” That keeps the CLI from chaining sub‑messages into the subject line and ensures our exact changelog text
  lands verbatim in the commit.
7. If `$ARGUMENTS` is provided, use that as the commit message instead.
8. Show the user what files were added (if any) and the result of the git commit.
9. Do not mention any co author or the llm that assisted in this process, do not mention the words claude, codex, or gemini.

This will stage ALL changes (new files, modifications, and deletions) before committing.

