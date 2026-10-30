# Submodule Management Guide

This document explains how to manage Git submodules in this central repository for Java development projects.

## Overview

This repository serves as a central hub that aggregates multiple Java projects as Git submodules. This allows:

- **Centralized Access**: Work with multiple related projects from a single location
- **Shared Resources**: Access the Devin CLI Java Code Analyzer skill across all projects
- **Consistent Environment**: Apply the same governance and best practices to all submodules
- **Easy Navigation**: Quickly switch between related projects

## Current Submodules

| Submodule | Path | Description | URL |
|-----------|------|-------------|-----|
| sandbox-interview | `projects/sandbox-interview` | Java interview sandbox project | https://github.com/darcskull/sandbox-interview |

## Getting Started

### Clone with Submodules

When cloning this repository for the first time, use the `--recursive` flag to automatically clone all submodules:

```bash
git clone --recursive https://github.com/darcskull/java-code-analyzer-skill.git
```

### Initialize Submodules

If you cloned without `--recursive`, you can initialize submodules later:

```bash
cd java-code-analyzer-skill
git submodule update --init --recursive
```

This will:
1. Clone each submodule repository
2. Check out the commit specified in the parent repository
3. Initialize and update each submodule

## Daily Workflow

### Check Submodule Status

```bash
# Show status of all submodules
git submodule status

# Show detailed status
git submodule foreach 'echo $name && git status'
```

### Update Submodules

#### Update to Latest Remote Commit

```bash
# Update all submodules to their latest remote commits
git submodule update --remote

# Or update a specific submodule
git submodule update --remote projects/sandbox-interview
```

#### Update to Parent Repository's Commit

```bash
# Reset submodules to the commit recorded in the parent repository
git submodule update --init --recursive
```

### Working in a Submodule

When you need to make changes to a submodule:

```bash
# Navigate to the submodule
cd projects/sandbox-interview

# Make your changes
git add .
git commit -m "Your changes"
git push origin main

# Return to parent repository
cd ../..

# Update the parent repository to track the new commit
git add projects/sandbox-interview
git commit -m "Update sandbox-interview to latest commit"
git push origin main
```

### Syncing Submodule Changes

When other developers update submodules:

```bash
# Pull latest changes from parent repository
git pull origin main

# Update submodules to the new commits
git submodule update --init --recursive
```

## Adding New Submodules

To add a new Java project as a submodule:

```bash
# Add the submodule
git submodule add https://github.com/USERNAME/PROJECT-NAME.git projects/PROJECT-NAME

# This creates:
# - .gitmodules file (if it doesn't exist)
# - projects/PROJECT-NAME directory
# - Stages the changes

# Commit the changes
git add .gitmodules projects/PROJECT-NAME
git commit -m "Add PROJECT-NAME as submodule"
git push origin main
```

### Best Practices for Adding Submodules

1. **Use descriptive names**: Choose clear, descriptive names for submodules
2. **Organize logically**: Place submodules in a consistent directory structure
3. **Document additions**: Update this document with the new submodule
4. **Test before committing**: Ensure the submodule can be cloned and initialized

## Removing Submodules

To remove a submodule:

```bash
# 1. Unregister the submodule
git submodule deinit -f projects/PROJECT-NAME

# 2. Remove the submodule directory
git rm -f projects/PROJECT-NAME

# 3. Remove the .git/modules/PROJECT-NAME directory
rm -rf .git/modules/PROJECT-NAME

# 4. Remove the entry from .gitmodules
git config -f .gitmodules --remove-section submodule.projects/PROJECT-NAME

# 5. Commit the changes
git add .gitmodules
git commit -m "Remove PROJECT-NAME submodule"
git push origin main
```

## Troubleshooting

### Submodule is in Detached HEAD State

This is normal behavior. Submodules are typically checked out at a specific commit, not a branch.

```bash
# If you need to work on a branch
cd projects/sandbox-interview
git checkout main
```

### Submodule Shows as Modified

If the submodule shows as modified in the parent repository:

```bash
# Check what changed
cd projects/sandbox-interview
git status

# If you want to update the parent to track the new commit
cd ../..
git add projects/sandbox-interview
git commit -m "Update submodule commit"
```

### Submodule Directory is Empty

If a submodule directory is empty after cloning:

```bash
# Initialize and update submodules
git submodule update --init --recursive
```

### Submodule Won't Update

If a submodule won't update to the remote version:

```bash
# Force update to remote
cd projects/sandbox-interview
git fetch origin
git checkout origin/main
cd ../..
git add projects/sandbox-interview
git commit -m "Force update submodule"
```

## Advanced Operations

### Update Specific Submodule to Specific Branch

```bash
cd projects/sandbox-interview
git checkout develop
cd ../..
git add projects/sandbox-interview
git commit -m "Update sandbox-interview to develop branch"
```

### Create a Branch in a Submodule

```bash
cd projects/sandbox-interview
git checkout -b feature/new-feature
# Make changes
git add .
git commit -m "Add new feature"
git push origin feature/new-feature
cd ../..
git add projects/sandbox-interview
git commit -m "Update sandbox-interview to feature/new-feature"
```

### Merge Submodule Changes

```bash
cd projects/sandbox-interview
git checkout main
git merge feature/new-feature
git push origin main
cd ../..
git add projects/sandbox-interview
git commit -m "Merge feature/new-feature into main"
```

## Best Practices

### 1. Commit Submodule Changes
Always commit submodule changes in the parent repository after making changes to a submodule. This ensures other developers get the same version.

### 2. Use Meaningful Commit Messages
When updating submodules, describe what changed in the submodule:
```bash
git commit -m "Update sandbox-interview: fix authentication bug"
```

### 3. Regular Updates
Regularly update submodules to stay current with their upstream repositories.

### 4. Test Before Committing
After updating a submodule, test that it works correctly before committing the update.

### 5. Document Dependencies
If submodules have dependencies on each other, document them in this file.

### 6. Branch Strategy
Consider creating a branch in the parent repository when working on multiple submodules simultaneously.

## Automation

### Script to Update All Submodules

Create a script `update-submodules.sh`:

```bash
#!/bin/bash
echo "Updating all submodules..."
git submodule update --remote --merge
git add .
git commit -m "Update all submodules to latest commits"
git push origin main
echo "All submodules updated!"
```

### Script to Check Submodule Status

Create a script `check-submodules.sh`:

```bash
#!/bin/bash
echo "Checking submodule status..."
git submodule status
echo ""
echo "Checking for uncommitted changes..."
git submodule foreach 'git status --short'
```

## CI/CD Integration

### GitHub Actions Example

```yaml
name: Update Submodules

on:
  schedule:
    - cron: '0 0 * * 0'  # Weekly
  workflow_dispatch:

jobs:
  update:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
        with:
          submodules: recursive
          
      - name: Update submodules
        run: |
          git submodule update --remote --merge
          
      - name: Commit changes
        run: |
          git config --local user.email "action@github.com"
          git config --local user.name "GitHub Action"
          git add .
          git diff --quiet && git diff --staged --quiet || git commit -m "Auto-update submodules"
          
      - name: Push changes
        uses: ad-m/github-push-action@master
        with:
          github_token: ${{ secrets.GITHUB_TOKEN }}
          branch: main
```

## Security Considerations

### 1. Review Submodule Code
Before adding a submodule, review its code for security issues.

### 2. Keep Submodules Updated
Regularly update submodules to get security patches.

### 3. Use Verified Sources
Only add submodules from trusted sources and verified repositories.

### 4. Monitor Submodule Changes
Monitor when submodules are updated to ensure no malicious changes are introduced.

## References

- [Git Submodule Documentation](https://git-scm.com/book/en/v2/Git-Tools-Submodules)
- [GitHub Submodule Guide](https://github.com/git-guides/git-submodules)
- [Atlassian Git Submodules Tutorial](https://www.atlassian.com/git/tutorials/git-submodules)

---

*Last updated: 2026-10-30*
