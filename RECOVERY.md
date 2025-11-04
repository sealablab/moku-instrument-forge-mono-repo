# Recovery and Restoration Guide

This document explains how to recover to known-good states if submodules become desynchronized or corrupted.

---

## Quick Recovery to v1.0.0

If something goes wrong and you need to get back to the initial stable state:

```bash
# From monorepo root
git fetch origin
git checkout v1.0.0
git submodule update --init --recursive --force
```

This will:
1. Reset monorepo to the v1.0.0 tagged commit
2. Initialize all submodules to their synchronized commits
3. Force overwrite any local changes in submodules

---

## Submodule Synchronization Points

The monorepo uses two types of tags for versioning:

### Monorepo Tags
- **`v1.0.0`** - Initial release (2025-11-03)
  - All submodules working
  - Fresh clone verified
  - Archive in place
  - Documentation complete

### Submodule Sync Tags
- **`monorepo-init-v1.0.0`** - Tagged on all 5 submodules
  - Used to mark synchronized state across repos
  - Each submodule has this tag on their respective commits

---

## Recovery Scenarios

### Scenario 1: Submodules Are Out of Sync

**Symptoms:** `git status` shows modified submodules, or submodule commits don't match expected state.

**Solution:**
```bash
# Reset all submodules to match parent repo's expectations
git submodule update --init --recursive --force
```

### Scenario 2: Submodules Are Corrupted or Missing

**Symptoms:** Submodule directories are empty, or `git submodule update` fails.

**Solution:**
```bash
# Deinitialize all submodules
git submodule deinit -f .

# Reinitialize from scratch
git submodule update --init --recursive
```

### Scenario 3: Need to Start Fresh

**Symptoms:** Everything is broken, want a clean slate.

**Solution:**
```bash
# Delete the entire repo and re-clone
cd ..
rm -rf moku-instrument-forge-mono-repo
git clone --recurse-submodules https://github.com/sealablab/moku-instrument-forge-mono-repo.git
cd moku-instrument-forge-mono-repo

# Optionally checkout v1.0.0
git checkout v1.0.0
```

### Scenario 4: Verify Current State

**Check monorepo version:**
```bash
git describe --tags
# Should show: v1.0.0 (or v1.0.0-N-gXXXXXXX if commits ahead)
```

**Check submodule commits:**
```bash
git submodule status
# Shows commit hashes for each submodule
```

**Check submodule tags:**
```bash
git submodule foreach 'echo "Repo: $name" && git describe --tags 2>/dev/null || echo "  No tags"'
```

---

## Understanding Submodule Commits

**Important:** Git submodules track **commit hashes**, not branches or tags.

When you see:
```bash
git submodule status
 e2301e312e650b0d820d376e84bc5a151dd04977 forge (monorepo-init-v1.0.0)
 2a46df56186a5681edd554e4682047f55f4c28bd libs/forge-vhdl (monorepo-init-v1.0.0)
```

This means:
- `forge/` is at commit `e2301e3` (which has tag `monorepo-init-v1.0.0`)
- `libs/forge-vhdl/` is at commit `2a46df5` (which has tag `monorepo-init-v1.0.0`)

The tag names in parentheses are for human reference only.

---

## Manual Verification Steps

After recovery, verify everything is working:

### 1. Verify Submodules Initialized

```bash
ls -la forge/generator libs/forge-vhdl/vhdl
# Should show files, not empty directories
```

### 2. Verify Python Environment

```bash
uv sync
python scripts/setup_forge_path.py
# Should show: ✅ Forge imports working correctly
```

### 3. Verify Tests Can Run

```bash
pytest --collect-only
# Should discover tests without errors
```

---

## Preventive Measures

### Before Making Changes

```bash
# Create a branch for experimental work
git checkout -b experiment/my-feature

# Work on changes...

# If something breaks, easy to get back
git checkout main
git submodule update --init --recursive --force
```

### Regular Checks

```bash
# Check if submodules are clean
git submodule foreach 'git status'

# Check if submodules match expected commits
git status
# Should show "nothing to commit, working tree clean"
```

---

## Advanced: Restoring Specific Submodule

If only one submodule is problematic:

```bash
# Example: Reset only the forge submodule
cd forge
git fetch origin
git checkout e2301e3  # Or use the tag: monorepo-init-v1.0.0
cd ..
git add forge
git commit -m "chore: Reset forge to v1.0.0 state"
```

---

## Getting Help

If recovery procedures fail:

1. Check the error message carefully
2. Verify network connectivity to GitHub
3. Ensure you have access to all submodule repositories
4. Try a completely fresh clone as last resort

**Last Resort:**
```bash
# Nuclear option - delete everything and start over
cd ..
rm -rf moku-instrument-forge-mono-repo
git clone --recurse-submodules https://github.com/sealablab/moku-instrument-forge-mono-repo.git
```

---

**Document Version:** 1.0
**Last Updated:** 2025-11-03
**Monorepo Version:** v1.0.0
