---
name: parallel-task
description: Create a parallel workspace by duplicating the entire directory. Perfect for Laravel Herd environments where you need to run and test the app in parallel.
disable-model-invocation: true
allowed-tools: Bash, Read, Write
argument-hint: "[task-description]"
---

# Create Parallel Task Workspace

You're creating a complete duplicate of the current directory so you can work on a separate task and test it in a parallel environment (e.g., Laravel Herd will auto-serve it at a new URL).

## Current Repository Info

Current directory: !`pwd`
Repository name: !`basename $(pwd)`
Current branch: !`git branch --show-current 2>/dev/null || echo "unknown"`

## Task Description
$ARGUMENTS

## Instructions

### Step 1: Duplicate the entire directory

Run these commands to create a complete copy in your Code directory:

```bash
# Get current directory info
CURRENT_DIR=$(pwd)
REPO_NAME=$(basename "$CURRENT_DIR")
TIMESTAMP=$(date +%s)
PARENT_DIR=$(dirname "$CURRENT_DIR")
WORKSPACE_PATH="${PARENT_DIR}/${REPO_NAME}-parallel-${TIMESTAMP}"

# Duplicate the entire directory (excluding .git at first, then init fresh repo)
echo "Creating duplicate at: $WORKSPACE_PATH"
rsync -a --exclude='.git' "$CURRENT_DIR/" "$WORKSPACE_PATH/"

# Initialize fresh git repo in the duplicate
cd "$WORKSPACE_PATH"
git init
git add .
git commit -m "Initial commit for parallel task: $ARGUMENTS"

# Store the workspace path for cleanup later
mkdir -p "$CURRENT_DIR/.claude"
echo "$WORKSPACE_PATH" > "$CURRENT_DIR/.claude/current-parallel-workspace.txt"

echo ""
echo "✓ Parallel workspace created at: $WORKSPACE_PATH"

# If using Laravel Herd, link the site and show the URL
if command -v herd &> /dev/null; then
    WORKSPACE_NAME=$(basename "$WORKSPACE_PATH")
    HERD_URL="${WORKSPACE_NAME}.test"

    # Explicitly link the site in Herd (parked directories may not include ~/code)
    echo "Linking site in Laravel Herd..."
    cd "$WORKSPACE_PATH"
    herd link

    echo "✓ Laravel Herd URL: http://${HERD_URL}"
    echo ""
    echo "  Opening in browser..."

    # Open the URL in default browser
    open "http://${HERD_URL}"

    echo "✓ Browser opened to parallel workspace!"
else
    echo "✓ Ready to use!"
fi
```

### Step 2: Open new Claude Code session

**Open a new terminal window** and run:

```bash
WORKSPACE_PATH=$(cat .claude/current-parallel-workspace.txt)
cd "$WORKSPACE_PATH"
claude
```

This launches a fresh Claude Code session in your parallel workspace.

### Step 3: Work on your task

In the new session:
1. Work on the task: **$ARGUMENTS**
2. Test changes manually (if using Herd, visit the URL shown above)
3. Make commits as you work
4. When ready, push to a new branch and create a PR

Example workflow:
```bash
# Create feature branch
git checkout -b fix/parallel-task

# Make your changes...

# Commit
git commit -am "Fix: $ARGUMENTS"

# Add remote (use your actual repo URL)
git remote add origin <your-repo-url>

# Push and create PR
git push -u origin fix/parallel-task
gh pr create --title "Fix: $ARGUMENTS" --body "Fixes issue in parallel workspace"
```

### Step 4: After PR is created and verified

**IMPORTANT**: Once you're happy with the PR and ready to clean up, remember to run:

```
/parallel-task:cleanup-parallel
```

This will remove the parallel workspace directory and free up disk space.

---

**Note**: This is a complete duplicate with its own git repository. Changes here are isolated until you push and create a PR. If using Laravel Herd, the site will be automatically available at the URL shown above.
