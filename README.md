# Claude Parallel Task Plugin

A Claude Code plugin for creating parallel workspaces by duplicating your entire project directory. Perfect for Laravel Herd and other development environments where you need to work on multiple tasks simultaneously.

## Problem This Solves

Ever had Claude working on a large feature in your main repo with lots of uncommitted changes, but needed to quickly fix a small bug? This plugin lets you:

- **Work in parallel** - Create isolated workspace while main repo is busy
- **Test manually** - Run and test changes in your browser (auto-serves with Laravel Herd)
- **No conflicts** - Complete isolation from your main workspace
- **Easy cleanup** - One command to remove the workspace when done

## Use Case

**Main repo** (Claude working on big feature):
- URL: `http://myapp.test`
- Status: Busy with large task, many uncommitted changes

**Parallel workspace** (you fix urgent bug):
- URL: `http://myapp-parallel-1234567890.test` (auto-generated)
- Status: Clean, isolated environment for quick fix
- Can test immediately in browser

## Installation

### 1. Add the marketplace

From within Claude Code:

```bash
/plugin marketplace add ElliotPutt/claude-parallel-task
```

### 2. Install the plugin

```bash
/plugin install parallel-task@claude-parallel-task
```

### 3. (Optional) Add to project settings

To share with your team, add to `.claude/settings.json`:

```json
{
  "extraKnownMarketplaces": {
    "claude-parallel-task": {
      "source": {
        "source": "github",
        "repo": "ElliotPutt/claude-parallel-task"
      }
    }
  },
  "enabledPlugins": {
    "parallel-task@claude-parallel-task": true
  }
}
```

## Usage

### Create a parallel workspace

While Claude is busy in your main repo:

```bash
/parallel-task:parallel-task fix login redirect bug
```

This will:
1. Duplicate your entire directory to `../myrepo-parallel-1234567890/`
2. Initialize a fresh git repo in the duplicate
3. Show you the new Herd URL (if using Laravel Herd)
4. Prompt you to open a new Claude Code session

### Open new session

In a **new terminal window**:

```bash
cd ../myrepo-parallel-1234567890
claude
```

### Work on your task

In the new Claude Code session:
1. Make your changes
2. Test manually (visit the Herd URL shown)
3. Commit and push
4. Create a PR

### Clean up after PR merged

Back in your **original repo/session**:

```bash
/parallel-task:cleanup-parallel
```

This removes the entire parallel workspace directory.

## Features

### 🚀 Full Duplication
- Complete directory copy using `rsync`
- Excludes `.git` and initializes fresh repo
- Complete isolation from main workspace

### 🌐 Laravel Herd Support
- Auto-detects Herd installation
- Shows new `.test` URL automatically
- No configuration needed
- Herd auto-removes URL on cleanup

### 🛡️ Safety Checks
- Warns if workspace has unpushed commits
- Shows workspace size before deletion
- Checks git status before cleanup
- Prevents accidental data loss

### 📝 State Tracking
- Tracks workspace location in `.claude/current-parallel-workspace.txt`
- Easy cleanup even if you forget the path
- Supports multiple parallel workspaces

## How It Works

### `/parallel-task:parallel-task [description]`

1. Gets current directory name and generates timestamp
2. Creates duplicate: `../project-parallel-1234567890/`
3. Uses `rsync -a --exclude='.git'` to copy everything
4. Initializes fresh git repo in duplicate
5. Stores workspace path for cleanup
6. Detects and shows Herd URL if available

### `/parallel-task:cleanup-parallel`

1. Reads tracked workspace path
2. Verifies workspace exists
3. Checks for unpushed commits (warns if found)
4. Deletes entire directory with `rm -rf`
5. Removes tracking file
6. Herd auto-removes `.test` domain

## Requirements

- Claude Code CLI
- `rsync` (pre-installed on macOS/Linux)
- Git
- (Optional) Laravel Herd for auto `.test` domains

## Use Cases

### Urgent Bug Fix
Main repo has large feature in progress, need to fix production bug quickly.

### Code Review
Want to test someone's branch while keeping your main workspace intact.

### Testing Multiple Approaches
Try different solutions in parallel without branch switching.

### Client Demo
Need to demo stable version while continuing development work.

## Tips

- **Database**: Both workspaces can share the same database (configurable in `.env`)
- **Dependencies**: Each workspace has its own `node_modules`, `vendor`, etc.
- **Multiple workspaces**: You can create multiple parallel workspaces simultaneously
- **Git remotes**: Each workspace starts with a fresh git repo (add remotes as needed)

## Comparison to Git Worktrees

Git worktrees are great for git-only isolation, but this plugin provides:

| Feature | Git Worktrees | Parallel Task |
|---------|---------------|---------------|
| Full directory duplication | ❌ (shares git objects) | ✅ |
| Separate dependencies | ❌ | ✅ |
| Herd auto-detection | ❌ | ✅ |
| Fresh git repo | ❌ | ✅ |
| Multiple dev servers | ⚠️ (manual setup) | ✅ |

## Examples

### Example 1: Quick Bug Fix

```bash
# In main repo (Claude busy with feature)
/parallel-task:parallel-task fix auth redirect

# New terminal
cd ../myapp-parallel-1709316480
claude

# In new session: fix bug, commit, push PR
git checkout -b fix/auth-redirect
# ... make changes ...
git commit -am "Fix auth redirect bug"
git remote add origin git@github.com:user/repo.git
git push -u origin fix/auth-redirect
gh pr create

# Back in main repo
/parallel-task:cleanup-parallel
```

### Example 2: Test PR Manually

```bash
/parallel-task:parallel-task test pr-branch

# New terminal
cd ../myapp-parallel-1709316480
git remote add origin git@github.com:user/repo.git
git fetch origin
git checkout pr-branch

# Test at http://myapp-parallel-1709316480.test
# When done
cd ../myapp
/parallel-task:cleanup-parallel
```

## Contributing

Issues and PRs welcome! This plugin is designed to be simple and focused on one task: creating parallel workspaces for testing.

## License

MIT

## Author

Created by [ElliotPutt](https://github.com/ElliotPutt)

## Changelog

### 1.0.0 (Initial Release)
- `/parallel-task:parallel-task` - Create parallel workspace
- `/parallel-task:cleanup-parallel` - Remove parallel workspace
- Laravel Herd auto-detection
- Safety checks for unpushed commits
- Workspace tracking
