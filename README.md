# Claude Code Custom Commands

A collection of custom commands for [Claude Code](https://claude.ai/code) to enhance your development workflow.

## Installation

### Quick Setup (Recommended)

To add these commands to your existing `.claude` directory:

```bash
# Navigate to your .claude directory (create it if it doesn't exist)
mkdir -p ~/.claude
cd ~/.claude

# Download just the commands folder
curl -L https://github.com/jynfairchild/.claude/archive/main.tar.gz | tar xz --strip=1 jynfairchild-.claude-main/commands
```

### Alternative: Clone Method

If you prefer using git:

```bash
# Clone to a temporary directory
cd ~
git clone --depth=1 https://github.com/jynfairchild/.claude.git temp-claude

# Copy just the commands folder to your .claude directory
mkdir -p ~/.claude
cp -r temp-claude/commands ~/.claude/

# Clean up
rm -rf temp-claude
```

### Full Clone (If you don't have a .claude directory yet)

```bash
cd ~
git clone https://github.com/jynfairchild/.claude.git .claude
```

**Note:** If you already have a `.claude` directory with personal files (CLAUDE.md, diary.md, etc.), use the Quick Setup or Alternative method to preserve your data.

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