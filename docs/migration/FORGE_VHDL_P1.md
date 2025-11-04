# Phase 1: forge-vhdl CocoTB Infrastructure Setup

**Status:** Ready for execution
**Duration:** ~30 minutes
**Prerequisites:** None (clean start)
**Goal:** Install CocoTB + GHDL filter infrastructure into `libs/forge-vhdl` submodule

---

## ⚠️ CRITICAL: Git Submodule Commit Protocol

**ALL COMMITS IN THIS PHASE MUST BE MADE INSIDE `libs/forge-vhdl` SUBMODULE!**

```bash
# ✅ CORRECT workflow
cd libs/forge-vhdl              # Enter submodule
# ... make changes ...
git add .
git commit -m "descriptive message"
git push origin main
cd ../..                         # Back to monorepo root
git add libs/forge-vhdl         # Update submodule reference
git commit -m "chore: Update forge-vhdl submodule"
git push

# ❌ WRONG workflow (DO NOT DO THIS)
cd .                            # Stay in monorepo root
git add libs/forge-vhdl/scripts/file.py  # WRONG REPO!
git commit -m "..."             # Commits to WRONG repo!
```

**If you forget this, the changes will NOT appear in the submodule repository!**

---

## Phase 1 Overview

### What We're Doing
Copying proven CocoTB progressive testing infrastructure from EZ-EMFI export into `libs/forge-vhdl` without modifying any existing VHDL code.

### What We're NOT Doing
- ❌ NO component migration (that's Phase 2)
- ❌ NO VHDL file changes
- ❌ NO voltage package work
- ❌ NO test execution (just infrastructure setup)
- ❌ NO llms.txt or CLAUDE.md updates (Phase 2)

### Files to Copy
1. **GHDL output filter** (1 file, 340 lines) - THE critical component
2. **CocoTB test framework** (4 files, ~600 lines) - Progressive testing infrastructure
3. **Documentation** (5 files, ~2500 lines) - Testing standards and guides

**Total:** 10 files, ~3500 lines

---

## Directory Structure to Create

```
libs/forge-vhdl/
├── scripts/                    # NEW: Python utilities
│   └── ghdl_output_filter.py
│
├── tests/                      # NEW: CocoTB test infrastructure
│   ├── test_base.py
│   ├── conftest.py
│   ├── run.py
│   └── test_configs.py
│
└── docs/                       # NEW: Testing documentation
    ├── GHDL_OUTPUT_FILTER.md
    ├── VOLO_COCOTB_TESTING_STANDARD.md
    ├── PROGRESSIVE_TESTING_GUIDE.md
    ├── COCOTB_PATTERNS.md
    └── VHDL_COCOTB_LESSONS_LEARNED.md
```

---

## Step-by-Step Execution

### Step 1: Navigate to Submodule
```bash
cd /Users/johnycsh/TTOP/moku-instrument-forge-mono-repo/libs/forge-vhdl
pwd  # Should show: .../libs/forge-vhdl
git status  # Should show: On branch main
```

**Checkpoint:** You are in the submodule, not the monorepo root.

---

### Step 2: Create Directory Structure
```bash
# Still in libs/forge-vhdl
mkdir -p scripts
mkdir -p tests
mkdir -p docs
```

**Checkpoint:** Three new directories created.

---

### Step 3: Copy GHDL Output Filter (The Secret Weapon)

```bash
# Still in libs/forge-vhdl
cp /tmp/cocotb_progressive_export/scripts/ghdl_output_filter.py scripts/

# Verify
ls -lh scripts/ghdl_output_filter.py
# Should show: ~10KB, 340 lines
```

**What this file does:**
- Intercepts GHDL output at OS file descriptor level
- Filters 90-98% of noise (metavalue warnings, null warnings, initialization spam)
- Preserves ALL critical info (errors, failures, PASS/FAIL results)
- Provides 4 filter levels (AGGRESSIVE/NORMAL/MINIMAL/NONE)
- Zero external dependencies (Python stdlib only)

**This is THE critical enabler for <20 line test output.**

**Checkpoint:** `scripts/ghdl_output_filter.py` exists and is ~340 lines.

---

### Step 4: Copy CocoTB Test Framework

```bash
# Still in libs/forge-vhdl
cp /tmp/cocotb_progressive_export/tests/test_base.py tests/
cp /tmp/cocotb_progressive_export/tests/conftest.py tests/
cp /tmp/cocotb_progressive_export/tests/run.py tests/
cp /tmp/cocotb_progressive_export/tests/test_configs.py tests/

# Verify
ls -lh tests/*.py
# Should show 4 files
```

**What these files do:**

**test_base.py** (Framework core)
- `TestBase` class with progressive test levels (P1/P2/P3/P4)
- Verbosity control (MINIMAL/NORMAL/VERBOSE/DEBUG)
- Token-efficient logging (`self.log()` with conditional output)
- Test execution scaffolding (`run_all_tests()`, `test()`, `setup()`)

**conftest.py** (Shared utilities)
- Clock generation (`setup_clock()`)
- Reset helpers (`reset_active_low()`, `reset_active_high()`)
- Pulse counting utilities
- Timeout management
- MCC Control0 bit scheme (for Moku integration)

**run.py** (Optimized test runner)
- Integrates GHDL filter automatically
- Supports test-level selection (P1/P2/P3/P4 via env var)
- Supports verbosity control (via env var)
- Minimal configuration needed

**test_configs.py** (Test registry)
- Central registry of all test configurations
- Maps test names to VHDL sources, entities, test modules
- Used by `run.py` for test discovery

**Checkpoint:** All 4 framework files in `tests/`.

---

### Step 5: Copy Documentation

```bash
# Still in libs/forge-vhdl
cp /tmp/cocotb_progressive_export/docs/GHDL_OUTPUT_FILTER.md docs/
cp /tmp/cocotb_progressive_export/docs/VOLO_COCOTB_TESTING_STANDARD.md docs/
cp /tmp/cocotb_progressive_export/docs/PROGRESSIVE_TESTING_GUIDE.md docs/
cp /tmp/cocotb_progressive_export/docs/COCOTB_PATTERNS.md docs/
cp /tmp/cocotb_progressive_export/docs/VHDL_COCOTB_LESSONS_LEARNED.md docs/

# Verify
ls -lh docs/*.md
# Should show 5 files
```

**What these docs provide:**

**GHDL_OUTPUT_FILTER.md** (500+ lines)
- Complete filter documentation
- How it works (OS-level FD redirection)
- Before/after examples (287 lines → 8 lines)
- Integration instructions
- Customization guide

**VOLO_COCOTB_TESTING_STANDARD.md** (258 lines) - **AUTHORITATIVE**
- Mandatory test structure (P1/P2/P3/P4)
- Directory organization rules
- Output suppression requirements
- Execution commands
- Critical DO's and DON'Ts

**PROGRESSIVE_TESTING_GUIDE.md** (576 lines)
- Step-by-step conversion from traditional to progressive tests
- How to structure P1/P2/P3 test levels
- Test value selection (small for P1, realistic for P2)
- Migration patterns

**COCOTB_PATTERNS.md** (493 lines)
- Quick reference for common CocoTB patterns
- Clock generation, reset, assertions
- Signal manipulation
- Best practices

**VHDL_COCOTB_LESSONS_LEARNED.md** (520 lines)
- Common pitfalls and how to avoid them
- GHDL quirks (metavalue warnings, initialization)
- Debugging techniques
- Performance optimization

**Checkpoint:** All 5 documentation files in `docs/`.

---

### Step 6: Verify File Structure

```bash
# Still in libs/forge-vhdl
tree -L 2 scripts tests docs
# Or use find if tree not available:
find scripts tests docs -type f | sort
```

**Expected output:**
```
scripts/ghdl_output_filter.py
tests/conftest.py
tests/run.py
tests/test_base.py
tests/test_configs.py
docs/COCOTB_PATTERNS.md
docs/GHDL_OUTPUT_FILTER.md
docs/PROGRESSIVE_TESTING_GUIDE.md
docs/VHDL_COCOTB_LESSONS_LEARNED.md
docs/VOLO_COCOTB_TESTING_STANDARD.md
```

**Checkpoint:** All 10 files present.

---

### Step 7: Commit in Submodule

```bash
# Still in libs/forge-vhdl
git status
# Should show 10 new files

git add scripts/ tests/ docs/

git commit -m "$(cat <<'EOF'
Add CocoTB progressive testing infrastructure for token-efficient AI iteration

This commit establishes the foundation for LLM-friendly VHDL testing with 98%
output reduction (287 lines → 8 lines) through intelligent GHDL filtering and
progressive test levels.

Infrastructure components:

1. GHDL Output Filter (scripts/ghdl_output_filter.py)
   - 340-line Python script for intelligent output suppression
   - 4 filter levels: AGGRESSIVE (98%), NORMAL (90%), MINIMAL (70%), NONE (0%)
   - OS-level file descriptor redirection (bulletproof)
   - Preserves ALL errors/failures/results
   - Zero external dependencies (Python 3.7+ stdlib only)

2. CocoTB Test Framework (tests/)
   - test_base.py: TestBase class with progressive levels (P1/P2/P3/P4)
   - conftest.py: Shared utilities (clock, reset, pulse counting)
   - run.py: Optimized test runner with integrated GHDL filter
   - test_configs.py: Central test registry

3. Documentation (docs/)
   - GHDL_OUTPUT_FILTER.md: Complete filter documentation (500+ lines)
   - VOLO_COCOTB_TESTING_STANDARD.md: Authoritative testing rules (258 lines)
   - PROGRESSIVE_TESTING_GUIDE.md: Step-by-step conversion guide (576 lines)
   - COCOTB_PATTERNS.md: Quick reference patterns (493 lines)
   - VHDL_COCOTB_LESSONS_LEARNED.md: Common pitfalls (520 lines)

Key Innovation:
- Progressive testing: P1 (essential, <20 lines) → P2 (standard) → P3 (full) → P4 (debug)
- Token efficiency: 50 tokens vs 4000 tokens per test run
- Context preservation: 98% GHDL noise reduction
- AI iteration speed: LLM can read 100% of test output

Files added:
- scripts/ghdl_output_filter.py (340 lines)
- tests/test_base.py (150 lines)
- tests/conftest.py (200 lines)
- tests/run.py (150 lines)
- tests/test_configs.py (100 lines)
- docs/GHDL_OUTPUT_FILTER.md (500 lines)
- docs/VOLO_COCOTB_TESTING_STANDARD.md (258 lines)
- docs/PROGRESSIVE_TESTING_GUIDE.md (576 lines)
- docs/COCOTB_PATTERNS.md (493 lines)
- docs/VHDL_COCOTB_LESSONS_LEARNED.md (520 lines)

Total: 10 files, ~3500 lines

Source: EZ-EMFI CocoTB progressive export (validated, production-tested)

This infrastructure enables Phase 2 component migration with CocoTB tests.

Related: docs/migration/FORGE_VHDL_PLAN.md
EOF
)"
```

**Checkpoint:** Commit created in submodule.

---

### Step 8: Push Submodule Changes

```bash
# Still in libs/forge-vhdl
git push origin main
```

**Checkpoint:** Changes pushed to remote submodule repository.

---

### Step 9: Return to Monorepo Root and Update Reference

```bash
cd ../..
pwd  # Should show: .../moku-instrument-forge-mono-repo

git status
# Should show: modified: libs/forge-vhdl (new commits)

git add libs/forge-vhdl

git commit -m "chore: Update forge-vhdl submodule with CocoTB testing infrastructure

Added progressive testing framework for token-efficient AI iteration:
- GHDL output filter (98% noise reduction)
- CocoTB test infrastructure (P1/P2/P3/P4 levels)
- Comprehensive testing documentation

Submodule commit: [commit hash will be inserted]

See libs/forge-vhdl for detailed commit message and file list.
Related: docs/migration/FORGE_VHDL_PLAN.md, docs/migration/FORGE_VHDL_P1.md
"

git push origin main
```

**Checkpoint:** Monorepo reference updated and pushed.

---

## Verification Checklist

Run these commands to verify Phase 1 completion:

```bash
# 1. Check submodule has new files
ls libs/forge-vhdl/scripts/ghdl_output_filter.py
ls libs/forge-vhdl/tests/test_base.py
ls libs/forge-vhdl/docs/VOLO_COCOTB_TESTING_STANDARD.md
# All should exist

# 2. Check submodule commit history
cd libs/forge-vhdl
git log --oneline -1
# Should show: "Add CocoTB progressive testing infrastructure..."

# 3. Check file counts
find scripts tests docs -type f | wc -l
# Should show: 10

# 4. Check monorepo reference
cd ../..
git log --oneline -1
# Should show: "chore: Update forge-vhdl submodule..."

# 5. Verify git state is clean
git status
# Should show: nothing to commit, working tree clean

cd libs/forge-vhdl
git status
# Should show: nothing to commit, working tree clean
```

**If all checks pass:** Phase 1 complete! ✅

---

## Phase 1 Completion Checklist

Mark each item when complete:

### File Operations
- [ ] Created `scripts/` directory in libs/forge-vhdl
- [ ] Created `tests/` directory in libs/forge-vhdl
- [ ] Created `docs/` directory in libs/forge-vhdl
- [ ] Copied `ghdl_output_filter.py` to `scripts/`
- [ ] Copied `test_base.py` to `tests/`
- [ ] Copied `conftest.py` to `tests/`
- [ ] Copied `run.py` to `tests/`
- [ ] Copied `test_configs.py` to `tests/`
- [ ] Copied 5 documentation files to `docs/`

### Git Operations (Submodule)
- [ ] Navigated to `libs/forge-vhdl` submodule
- [ ] Ran `git add` for new files
- [ ] Created commit with descriptive message
- [ ] Pushed commit to submodule remote (`git push origin main`)

### Git Operations (Parent Monorepo)
- [ ] Navigated back to monorepo root
- [ ] Ran `git add libs/forge-vhdl` to update reference
- [ ] Created commit with "chore: Update forge-vhdl submodule"
- [ ] Pushed commit to monorepo remote

### Verification
- [ ] All 10 files exist in correct locations
- [ ] Submodule commit visible in `git log`
- [ ] Monorepo commit visible in `git log`
- [ ] Both repos have clean working tree (`git status`)
- [ ] Submodule reference updated in parent

### Handoff to Phase 2
- [ ] Phase 1 checklist 100% complete
- [ ] Read `docs/migration/FORGE_VHDL_P2.md`
- [ ] Ready to begin component migration

---

## What Changed (Summary)

### Before Phase 1
```
libs/forge-vhdl/
├── vhdl/               # VHDL source files (unchanged)
├── llms.txt            # Existing (unchanged)
├── README.md           # Existing (unchanged)
└── pyproject.toml      # Existing (unchanged)
```

### After Phase 1
```
libs/forge-vhdl/
├── vhdl/               # VHDL source files (unchanged)
├── scripts/            # NEW: GHDL filter
├── tests/              # NEW: CocoTB infrastructure
├── docs/               # NEW: Testing docs
├── llms.txt            # Existing (unchanged - will update in Phase 2)
├── README.md           # Existing (unchanged)
└── pyproject.toml      # Existing (unchanged)
```

**Net change:** +10 files, +3 directories, ~3500 lines of infrastructure code.

---

## Common Issues & Solutions

### Issue: "git commit" commits to wrong repo
**Symptom:** Changes don't appear in submodule repository
**Solution:** Always `cd libs/forge-vhdl` first, verify with `pwd`

### Issue: Files copied but not committed
**Symptom:** `git status` shows untracked files
**Solution:** Run `git add` before `git commit`

### Issue: Submodule reference not updated in parent
**Symptom:** Monorepo doesn't show submodule changes
**Solution:** Run `git add libs/forge-vhdl` after submodule commit

### Issue: Permission denied when copying files
**Symptom:** `cp` command fails
**Solution:** Check `/tmp/cocotb_progressive_export` exists and is readable

### Issue: Directory already exists
**Symptom:** `mkdir` warns "File exists"
**Solution:** Safe to ignore (or use `mkdir -p` which won't error)

---

## Time Estimates

- **Step 1-2** (Navigation + directories): 1 minute
- **Step 3** (GHDL filter): 2 minutes
- **Step 4** (CocoTB framework): 3 minutes
- **Step 5** (Documentation): 5 minutes
- **Step 6** (Verification): 2 minutes
- **Step 7-9** (Git operations): 5 minutes
- **Verification checklist**: 5 minutes

**Total estimated time:** ~25 minutes

---

## Next Steps

After Phase 1 completion:

1. ✅ Mark all checklist items complete
2. 📖 Read `docs/migration/FORGE_VHDL_P2.md`
3. 🔍 Review Phase 2 component migration plan
4. 🚀 Begin Phase 2 execution (component migration with CocoTB tests)

---

## Success Criteria

Phase 1 is complete when:

✅ All 10 files copied to correct locations
✅ Submodule commit created and pushed
✅ Monorepo reference updated and pushed
✅ Git working tree clean in both repos
✅ Verification checklist 100% complete
✅ Ready to begin Phase 2

---

**The Golden Rule:**

> **"All commits during Phase 1 MUST be made inside libs/forge-vhdl submodule."**

If you follow the step-by-step guide, you cannot make this mistake.

---

**Phase 1 Status:** Ready for execution
**Next:** Execute steps 1-9, complete checklist, proceed to Phase 2
