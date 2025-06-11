# Claude Code Custom Commands

A collection of custom commands for [Claude Code](https://claude.ai/code) to enhance your development workflow.

## Installation

1. Clone this repository to your home directory:
```bash
cd ~
git clone https://github.com/yourusername/claude-commands.git .claude
```

2. If you already have a `.claude` directory, back it up first:
```bash
mv ~/.claude ~/.claude.backup
git clone https://github.com/yourusername/claude-commands.git .claude
# Copy any personal files back
cp ~/.claude.backup/CLAUDE.md ~/.claude/ 2>/dev/null || true
```

## Available Commands

### `/user:explore`
Comprehensive project exploration that analyzes the entire codebase structure, technologies, and recent changes. Creates `.claude/explore.md` for future reference.

### `/user:context`
Lightweight context retrieval that reads existing exploration data, diary entries, and recent git activity. Use this for quick project resumption.

### `/user:wdiary`
Write a diary entry documenting your coding session, including changes made, current status, and next steps. Saves to `.claude/diary.md`.

### `/user:commit`
Create a git commit using the commit message from your latest diary entry, or provide a custom message. Automatically stages all changes.

### `/user:move_claude`
Move CLAUDE.md from project root to `.claude/` directory for cleaner project organization.

## Usage Examples

### Starting work on a new project:
```
/user:explore
```

### Resuming work the next day:
```
/user:context
```

### After a coding session:
```
/user:wdiary
/user:commit
```

### Custom commit message:
```
/user:commit Fixed the login bug and updated tests
```

## Contributing

Feel free to submit issues and pull requests for new commands or improvements!

## Privacy Note

This repository only contains the command definitions. All your personal data (conversation logs, diary entries, settings) are excluded via .gitignore.