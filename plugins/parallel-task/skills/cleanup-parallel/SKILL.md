---
name: cleanup-parallel
description: Remove the parallel workspace created by /parallel-task. Run this after you've merged your PR and are done with the parallel workspace.
disable-model-invocation: true
allowed-tools: Bash, Read
---

# Cleanup Parallel Workspace

Remove the complete duplicate directory and free up disk space.

## Current Parallel Workspace

Tracked workspace: !`cat .claude/current-parallel-workspace.txt 2>/dev/null || echo "No parallel workspace tracked"`

## Instructions

### Step 1: Verify the workspace

Check what will be removed:

```bash
if [ -f .claude/current-parallel-workspace.txt ]; then
    WORKSPACE_PATH=$(cat .claude/current-parallel-workspace.txt)
    echo "Workspace to remove: $WORKSPACE_PATH"

    if [ -d "$WORKSPACE_PATH" ]; then
        echo "✓ Workspace exists"
        echo ""
        echo "Size: $(du -sh "$WORKSPACE_PATH" | cut -f1)"

        # Show git status if it's a repo
        if [ -d "$WORKSPACE_PATH/.git" ]; then
            cd "$WORKSPACE_PATH"
            echo ""
            echo "Git status:"
            git status -s || echo "Could not read git status"

            # Check for unpushed commits
            UNPUSHED=$(git log --branches --not --remotes 2>/dev/null | wc -l | tr -d ' ')
            if [ "$UNPUSHED" -gt 0 ]; then
                echo ""
                echo "⚠️  WARNING: This workspace has $UNPUSHED unpushed commit(s)!"
            fi
        fi
    else
        echo "⚠ Workspace directory not found (may have been deleted already)"
    fi
else
    echo "⚠ No parallel workspace tracked. Nothing to clean up."
fi
```

### Step 2: Remove the workspace

**⚠️ WARNING**: This will PERMANENTLY DELETE the entire workspace directory!

Make sure you've:
- ✓ Pushed all changes
- ✓ Created and verified your PR
- ✓ Merged the PR (if ready)
- ✓ Manually tested everything works

Then run:

```bash
if [ -f .claude/current-parallel-workspace.txt ]; then
    WORKSPACE_PATH=$(cat .claude/current-parallel-workspace.txt)

    if [ -d "$WORKSPACE_PATH" ]; then
        # Clean up Herd site first (if Herd is installed)
        if command -v herd &> /dev/null; then
            echo "Cleaning up Laravel Herd site..."
            cd "$WORKSPACE_PATH"

            # Stop the site if it's running
            herd stop 2>/dev/null || true

            # Unlink/forget the site from Herd
            herd unlink 2>/dev/null || true

            echo "✓ Herd site stopped and unlinked"
            cd - > /dev/null
        fi

        # Remove the entire directory
        echo "Removing: $WORKSPACE_PATH"
        rm -rf "$WORKSPACE_PATH"

        echo "✓ Parallel workspace deleted!"
        echo "✓ Disk space freed up"

        if command -v herd &> /dev/null; then
            echo "✓ Herd site completely removed"
        fi
    else
        echo "⚠ Directory already removed."
    fi

    # Clean up tracking file
    rm .claude/current-parallel-workspace.txt
    echo "✓ Cleanup complete!"

    # Restart Herd to ensure clean state
    if command -v herd &> /dev/null; then
        echo "Restarting Herd services..."
        herd restart
        echo "✓ Herd services restarted"
    fi
else
    echo "⚠ No parallel workspace to clean up."
fi
```

### Step 3: Verify cleanup

Confirm the directory is gone:

```bash
echo "Checking for remaining parallel workspaces..."
ls -la ../ | grep parallel || echo "✓ No parallel workspace directories found"

# If Herd is installed, verify the site is unlinked
if command -v herd &> /dev/null; then
    echo ""
    echo "Checking Herd sites..."
    if herd list | grep -i parallel; then
        echo "⚠️  Found parallel sites still linked in Herd (may need manual cleanup)"
    else
        echo "✓ No parallel sites found in Herd"
    fi
fi
```

---

**Done!** Your parallel workspace has been completely removed. You can create a new one anytime with `/parallel-task:parallel-task`.
