Please create a git commit using the commit message from the latest diary entry. Follow these steps:

1. Check if .claude/diary.md exists
2. If it doesn't exist, tell the user to run /user:wdiary first to create a diary entry
3. If it exists, read the file and find the most recent diary entry (the last entry in the file)
4. Extract the "Commit Message:" section from that entry
5. Run git status to check for untracked files
6. If there are untracked files, run: git add -A
7. Then commit with: git commit -m "[extracted commit message]", dont add anything about claude
8. If $ARGUMENTS is provided, use that as the commit message instead
9. Show the user what files were added (if any) and the result of the git commit

This will stage ALL changes (new files, modifications, and deletions) before committing.
