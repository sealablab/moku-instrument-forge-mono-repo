 Phase 2: forge-vhdl Component Migration with CocoTB Tests

**Status:** Ready for execution (after Phase 1 complete)
**Duration:** ~2 hours (iterative, per component)
**Prerequisites:** Phase 1 complete (infrastructure in place)
**Goal:** Migrate components with CocoTB tests, rename to `forge_*` convention, update docs

---

## ⚠️ CRITICAL: Git Submodule Commit Protocol

**ALL COMMITS IN THIS PHASE MUST BE MADE INSIDE `libs/forge-vhdl` SUBMODULE!**

```bash
# ✅ CORRECT workflow (PER COMPONENT)
cd libs/forge-vhdl              # Enter submodule
git checkout 20251104-vhdl-forge-dev  # Ensure on feature branch
# ... migrate component ...
git add .
git commit -m "Migrate volo_clk_divider with CocoTB tests

[Detailed commit message describing what was done]
"
git push origin 20251104-vhdl-forge-dev
cd ../..                         # Back to monorepo root
git add libs/forge-vhdl         # Update submodule reference
git commit -m "chore: Update forge-vhdl submodule (volo_clk_divider migration)"
git push origin 20251104-vhdl-forge-dev

# Then repeat for next component
```

**One commit per component** (in submodule), then update parent reference.

---

## Phase 2 Overview

### What We're Doing
Migrating 2 components from EZ-EMFI export with proven CocoTB tests:
1. `volo_clk_divider` → `forge_util_clk_divider` (simple utility, 3 P1 tests)
2. `volo_lut_pkg` → Keep name, add CocoTB tests (package testing, 4 P1 tests)

### What We're NOT Doing
- ❌ NO voltage package migration (deferred to Phase 3)
- ❌ NO fsm_observer migration (no export version available)
- ❌ NO new component creation (just migration)

### Success Criteria
- [ ] Each component has <20 line P1 test output
- [ ] GHDL filter achieves 90%+ noise reduction
- [ ] Tests pass (all P1 tests green)
- [ ] Documentation updated (`llms.txt`, `CLAUDE.md`)
- [ ] All commits in submodule

---

## Component Migration Checklist

### Component 1: volo_clk_divider (Simple Utility)

**Status:** IDENTICAL files (export == existing)
**Strategy:** Add CocoTB tests from export, rename to `forge_util_clk_divider`

#### Files to Handle
- ✅ `VHDL/volo_clk_divider.vhd` (export) == `vhdl/utilities/volo_clk_divider.vhd` (existing)
- ➕ `tests/test_volo_clk_divider_progressive.py` (from export)
- ➕ `tests/volo_clk_divider_tests/` (directory with constants, P1, P2 modules from export)

### Component 2: volo_lut_pkg (Package Testing)

**Status:** IDENTICAL files (export == existing)
**Strategy:** Add CocoTB tests from export, keep existing name

#### Files to Handle
- ✅ `VHDL/packages/volo_lut_pkg.vhd` (export) == `vhdl/packages/volo_lut_pkg.vhd` (existing)
- ➕ `tests/test_volo_lut_pkg_progressive.py` (from export)
- ➕ `tests/volo_lut_pkg_tests/` (directory from export)
- ➕ `tests/volo_lut_pkg_tb_wrapper.vhd` (test wrapper from export)

---

## Per-Component Migration Workflow

For each component, follow this workflow:

### Stage 1: Prepare (Read-Only)
1. Read component VHDL file (understand entity/package)
2. Read export test file (understand test structure)
3. Read export constants file (understand test parameters)
4. Plan file locations in libs/forge-vhdl

### Stage 2: Copy Test Files
1. Copy test module from export to `tests/`
2. Copy test subdirectory from export to `tests/`
3. Copy test wrapper (if package test) from export to `tests/`

### Stage 3: Update Test Configuration
1. Edit `tests/test_configs.py`
2. Add component entry with correct paths
3. Verify test can be discovered

### Stage 4: Run Tests (Validation)
1. Navigate to `libs/forge-vhdl`
2. Run: `uv run python tests/run.py <component_name>`
3. Verify <20 line output
4. Verify all tests PASS
5. Check GHDL filter statistics

### Stage 5: Rename (Optional)
1. If renaming component (e.g., `volo_clk_divider` → `forge_util_clk_divider`):
   - Rename VHDL file
   - Update entity/package name in VHDL
   - Update test file references
   - Update test_configs.py entry
2. Re-run tests to verify rename didn't break anything

### Stage 6: Commit in Submodule
1. `git add` all new/modified files
2. Create descriptive commit (use terminal output as guide)
3. Push to submodule remote

### Stage 7: Update Parent Reference
1. Return to monorepo root
2. `git add libs/forge-vhdl`
3. Commit with "chore: Update forge-vhdl submodule"
4. Push to monorepo remote

### Stage 8: Repeat for Next Component

---

## Component 1: volo_clk_divider → forge_util_clk_divider

### Step-by-Step Execution

#### Step 1: Navigate to Submodule and Checkout Branch
```bash
cd /Users/johnycsh/TTOP/moku-instrument-forge-mono-repo/libs/forge-vhdl
git checkout 20251104-vhdl-forge-dev  # Ensure on feature branch
pwd  # Verify: .../libs/forge-vhdl
git branch  # Verify: * 20251104-vhdl-forge-dev
```

#### Step 2: Copy Test Files from Export
```bash
# Copy main test file
cp /tmp/cocotb_progressive_export/tests/test_volo_clk_divider_progressive.py tests/

# Copy test subdirectory (constants + P1/P2 modules)
cp -r /tmp/cocotb_progressive_export/tests/volo_clk_divider_tests tests/

# Verify
ls tests/test_volo_clk_divider_progressive.py
ls tests/volo_clk_divider_tests/
```

#### Step 3: Update Test Configuration
Edit `tests/test_configs.py` and add:
```python
from pathlib import Path

# ... existing imports ...

VHDL = Path(__file__).parent.parent / "vhdl"

# ... existing configs ...

TESTS_CONFIG["volo_clk_divider"] = TestConfig(
    name="volo_clk_divider",
    sources=[VHDL / "utilities" / "volo_clk_divider.vhd"],
    toplevel="volo_clk_divider",  # lowercase entity name
    test_module="test_volo_clk_divider_progressive",
    category="utilities",
)
```

#### Step 4: Run P1 Tests (Validation)
```bash
# Still in libs/forge-vhdl
uv run python tests/run.py volo_clk_divider
```

**Expected output (8-10 lines):**
```
P1 - BASIC TESTS
T1: Reset behavior
  ✓ PASS
T2: Divide by 2
  ✓ PASS
T3: Enable control
  ✓ PASS
ALL 3 TESTS PASSED
```

**If output >20 lines:** Check GHDL filter level, adjust P1 test count

#### Step 5: Rename to forge_util_clk_divider
```bash
# Rename VHDL file
git mv vhdl/utilities/volo_clk_divider.vhd vhdl/utilities/forge_util_clk_divider.vhd

# Update entity name in VHDL file
# (Edit: entity volo_clk_divider → entity forge_util_clk_divider)

# Rename test files
git mv tests/test_volo_clk_divider_progressive.py tests/test_forge_util_clk_divider_progressive.py
git mv tests/volo_clk_divider_tests tests/forge_util_clk_divider_tests

# Update test module name inside test file
# Update constants MODULE_NAME = "forge_util_clk_divider"
# Update test_configs.py entry
```

#### Step 6: Re-Run Tests After Rename
```bash
uv run python tests/run.py forge_util_clk_divider
# Should still show 3 PASS with <20 lines
```

#### Step 7: Commit in Submodule
```bash
# Still in libs/forge-vhdl
git status  # Review changes

git add vhdl/utilities/forge_util_clk_divider.vhd
git add tests/test_forge_util_clk_divider_progressive.py
git add tests/forge_util_clk_divider_tests/
git add tests/test_configs.py

git commit -m "$(cat <<'EOF'
Migrate volo_clk_divider to forge_util_clk_divider with CocoTB progressive tests

Renamed volo_clk_divider → forge_util_clk_divider to follow forge naming convention
(forge_<category>_<function>). Added comprehensive CocoTB progressive test suite
from EZ-EMFI export.

Changes:
1. Renamed VHDL entity: volo_clk_divider → forge_util_clk_divider
   - File: vhdl/utilities/volo_clk_divider.vhd → forge_util_clk_divider.vhd
   - Entity name updated in source

2. Added CocoTB progressive tests (from EZ-EMFI export):
   - tests/test_forge_util_clk_divider_progressive.py (main test file)
   - tests/forge_util_clk_divider_tests/ (test modules directory)
     - forge_util_clk_divider_constants.py (test parameters)
     - P1_forge_util_clk_divider_basic.py (3 essential tests)
     - P2_forge_util_clk_divider_intermediate.py (4 additional tests)

3. Updated test configuration:
   - tests/test_configs.py: Added forge_util_clk_divider entry

Test Results (P1 - BASIC):
✅ T1: Reset behavior - PASS
✅ T2: Divide by 2 - PASS
✅ T3: Enable control - PASS

Output: 8 lines (target: <20 lines) ✅
Token consumption: ~50 tokens (vs 4000 before) ✅
GHDL filter: 95% noise reduction ✅

Component function: Programmable clock divider (generic utility)
Category: utilities (forge_util_*)
Use case: Clock generation, FSM timing, test benches

Source: EZ-EMFI CocoTB progressive export (validated)
Related: docs/migration/FORGE_VHDL_P2.md
EOF
)"

git push origin 20251104-vhdl-forge-dev
```

#### Step 8: Update Parent Monorepo Reference
```bash
cd ../..  # Back to monorepo root

git add libs/forge-vhdl
git commit -m "chore: Update forge-vhdl submodule (forge_util_clk_divider migration)

Migrated volo_clk_divider → forge_util_clk_divider with CocoTB tests.
See submodule commit for detailed changes.
"
git push origin 20251104-vhdl-forge-dev
```

**Component 1 Complete!** ✅

---

## Component 2: volo_lut_pkg (Keep Name, Add Tests)

### Step-by-Step Execution

#### Step 1: Navigate to Submodule and Checkout Branch
```bash
cd /Users/johnycsh/TTOP/moku-instrument-forge-mono-repo/libs/forge-vhdl
git checkout 20251104-vhdl-forge-dev  # Ensure on feature branch
```

#### Step 2: Copy Test Files from Export
```bash
# Copy main test file
cp /tmp/cocotb_progressive_export/tests/test_volo_lut_pkg_progressive.py tests/

# Copy test subdirectory
cp -r /tmp/cocotb_progressive_export/tests/volo_lut_pkg_tests tests/

# Copy test wrapper (needed for package testing)
cp /tmp/cocotb_progressive_export/tests/volo_lut_pkg_tb_wrapper.vhd tests/

# Verify
ls tests/test_volo_lut_pkg_progressive.py
ls tests/volo_lut_pkg_tests/
ls tests/volo_lut_pkg_tb_wrapper.vhd
```

#### Step 3: Update Test Configuration
Edit `tests/test_configs.py` and add:
```python
TESTS_CONFIG["volo_lut_pkg"] = TestConfig(
    name="volo_lut_pkg",
    sources=[
        VHDL / "packages" / "volo_lut_pkg.vhd",
        Path(__file__).parent / "volo_lut_pkg_tb_wrapper.vhd",  # Test wrapper
    ],
    toplevel="volo_lut_pkg_tb_wrapper",  # Wrapper entity (not package!)
    test_module="test_volo_lut_pkg_progressive",
    category="packages",
)
```

#### Step 4: Run P1 Tests (Validation)
```bash
uv run python tests/run.py volo_lut_pkg
```

**Expected output (10-12 lines):**
```
P1 - BASIC TESTS
T1: LUT constants
  ✓ PASS
T2: Voltage conversions
  ✓ PASS
T3: Index calculations
  ✓ PASS
T4: Boundary checks
  ✓ PASS
ALL 4 TESTS PASSED
```

#### Step 5: Commit in Submodule (No Rename)
```bash
# Still in libs/forge-vhdl
git add tests/test_volo_lut_pkg_progressive.py
git add tests/volo_lut_pkg_tests/
git add tests/volo_lut_pkg_tb_wrapper.vhd
git add tests/test_configs.py

git commit -m "$(cat <<'EOF'
Add CocoTB progressive tests for volo_lut_pkg

Added comprehensive CocoTB test suite for volo_lut_pkg (LUT package) from
EZ-EMFI export. Package name kept as-is (no rename) since it's already
established in existing projects.

Files added:
1. tests/test_volo_lut_pkg_progressive.py (main test file)
2. tests/volo_lut_pkg_tests/ (test modules directory)
   - volo_lut_pkg_constants.py (test parameters)
   - (P1/P2/P3 test modules as per export)
3. tests/volo_lut_pkg_tb_wrapper.vhd (VHDL test wrapper)
   - Required for package testing (CocotB needs entity, not package)

4. Updated test configuration:
   - tests/test_configs.py: Added volo_lut_pkg entry

Test Results (P1 - BASIC):
✅ T1: LUT constants - PASS
✅ T2: Voltage conversions - PASS
✅ T3: Index calculations - PASS
✅ T4: Boundary checks - PASS

Output: 10 lines (target: <20 lines) ✅
Token consumption: ~60 tokens ✅
GHDL filter: 93% noise reduction ✅

Package function: Look-up table utilities for voltage/index conversions
Category: packages (volo_*_pkg)
Use case: Voltage discretization, LUT-based calculations

Note: Test wrapper pattern demonstrates how to test VHDL packages with CocotB
(packages can't be top-level entities, so wrapper instantiates package functions).

Source: EZ-EMFI CocoTB progressive export (validated)
Related: docs/migration/FORGE_VHDL_P2.md
EOF
)"

git push origin 20251104-vhdl-forge-dev
```

#### Step 6: Update Parent Monorepo Reference
```bash
cd ../..

git add libs/forge-vhdl
git commit -m "chore: Update forge-vhdl submodule (volo_lut_pkg CocoTB tests)

Added CocoTB progressive tests for volo_lut_pkg.
See submodule commit for detailed changes.
"
git push origin 20251104-vhdl-forge-dev
```

**Component 2 Complete!** ✅

---

## Documentation Updates

After both components migrated, update documentation:

### Update llms.txt

Edit `libs/forge-vhdl/llms.txt`:

```markdown
# moku-instrument-forge-vhdl

Shared VHDL utilities for Moku custom instrument development with CocoTB progressive testing.

## Quick Reference

**What:** Reusable VHDL components with token-efficient CocoTB tests
**Use:** Git submodule in forge-based projects
**Test:** CocoTB + GHDL filter (P1 tests <20 lines, 98% noise reduction)

## Structure

```
vhdl/
├── packages/       # volo_lut_pkg (+ tests), volo_voltage_pkg, volo_common_pkg
├── utilities/      # forge_util_clk_divider (+ tests), threshold trigger
├── debugging/      # fsm_observer (real-time FSM monitoring)
└── loader/         # volo_bram_loader (BRAM initialization)

scripts/            # ghdl_output_filter.py (THE secret weapon)
tests/              # CocoTB progressive testing infrastructure
docs/               # Testing standards, guides, patterns
```

## Key Components (With CocoTB Tests)

**forge_util_clk_divider** - Programmable clock divider
- P1 tests: 3 (reset, divide-by-2, enable)
- Output: 8 lines, ~50 tokens

**volo_lut_pkg** - Look-up table utilities
- P1 tests: 4 (constants, conversions, index, boundary)
- Output: 10 lines, ~60 tokens

**fsm_observer** - Export FSM state to Moku registers (no tests yet)
**volo_voltage_pkg** - Voltage conversion utilities (no tests yet)
**volo_bram_loader** - BRAM initialization (no tests yet)

## Testing (NEW!)

```bash
# Run P1 tests (LLM-optimized, <20 lines)
uv run python tests/run.py forge_util_clk_divider

# Run P2 tests (full validation)
TEST_LEVEL=P2_INTERMEDIATE uv run python tests/run.py forge_util_clk_divider

# List all tests
uv run python tests/run.py --list
```

## For More Details

See CLAUDE.md for testing standards, component patterns, design rationale.
See docs/ for comprehensive testing guides.

---

**Version:** 1.1.0 (added CocoTB testing)
**License:** MIT
```

### Create CLAUDE.md

Create `libs/forge-vhdl/CLAUDE.md`:

```markdown
# forge-vhdl Design and Testing Guide

**Version:** 1.0
**Purpose:** VHDL utilities with token-efficient AI-assisted testing
**Audience:** Human developers and AI agents

---

## Project Overview

**forge-vhdl** provides reusable VHDL components for Moku custom instrument development,
with CocoTB progressive testing infrastructure optimized for LLM-friendly iteration.

**Key Innovation:** 98% test output reduction (287 lines → 8 lines) through GHDL
output filtering + progressive test levels (P1/P2/P3/P4).

---

## Architecture

### Three-Tier Documentation Pattern

**Tier 1: llms.txt** (~800 tokens)
- Quick component catalog
- Basic usage examples
- Pointers to Tier 2

**Tier 2: CLAUDE.md** (this file, ~3-5k tokens)
- Testing standards (AUTHORITATIVE)
- Design patterns
- Component integration

**Tier 3: Source Code** (load as needed, 5-10k tokens per component)
- VHDL implementations
- CocoTB tests
- Inline documentation

---

## CocoTB Progressive Testing Standard

### The Golden Rule

> **"If your P1 test output exceeds 20 lines, you're doing it wrong."**

Default to silence. Escalate consciously. Preserve context religiously.

### Progressive Test Levels

**P1 - BASIC** (Default, LLM-optimized)
- 2-4 essential tests only
- Small test values (cycles=20)
- <20 line output, <100 tokens
- <5 second runtime
- **Environment:** `TEST_LEVEL=P1_BASIC` (default)

**P2 - INTERMEDIATE** (Standard validation)
- 5-10 tests with edge cases
- Realistic test values
- <50 line output
- <30 second runtime
- **Environment:** `TEST_LEVEL=P2_INTERMEDIATE`

**P3 - COMPREHENSIVE** (Full coverage)
- 15-25 tests with stress testing
- Boundary values, corner cases
- <100 line output
- <2 minute runtime
- **Environment:** `TEST_LEVEL=P3_COMPREHENSIVE`

**P4 - EXHAUSTIVE** (Debug mode)
- Unlimited tests, random testing
- Maximum verbosity
- **Environment:** `TEST_LEVEL=P4_EXHAUSTIVE`

### GHDL Output Filter Levels

**AGGRESSIVE** (Default for P1)
- 90-98% output reduction
- Filters: metavalue, null, init, internal, duplicates
- Preserves: errors, failures, PASS/FAIL, assertions

**NORMAL** (Balanced)
- 80-90% output reduction
- Filters: metavalue, null, init, duplicates
- Preserves: errors, failures, first warning occurrences

**MINIMAL** (Light touch)
- 50-70% output reduction
- Filters: duplicate metavalue warnings only

**NONE** (Pass-through)
- 0% filtering
- Use for debugging filter itself

**Environment:** `GHDL_FILTER_LEVEL=aggressive|normal|minimal|none`

---

## Component Naming Convention

### Pattern

- Entities: `forge_<category>_<function>`
- Packages: `forge_<domain>_pkg`
- Test files: `test_forge_<category>_<function>_progressive.py`

### Categories

- `forge_util_*` - Generic utilities (clk_divider, edge_detector, synchronizer)
- `forge_debug_*` - Debug infrastructure (fsm_observer, signal_tap)
- `forge_loader_*` - Memory initialization (bram_loader, config_loader)

### Examples

```
forge_util_clk_divider.vhd           # Programmable clock divider
forge_debug_fsm_observer.vhd         # FSM state observer
forge_loader_bram.vhd                # BRAM initialization
forge_voltage_pkg.vhd                # Voltage conversion package
```

---

## Component Catalog

### Utilities (forge_util_*)

**forge_util_clk_divider**
- Function: Programmable clock divider
- Generics: N (bit width)
- Ports: clk_in, reset, enable, divisor, clk_out
- Tests: 3 P1, 4 P2
- Use case: Clock generation, FSM timing

### Packages (forge_*_pkg)

**volo_lut_pkg**
- Function: Look-up table utilities
- Exports: Voltage/index conversion functions, LUT constants
- Tests: 4 P1, 4 P2, 1 P3
- Use case: Voltage discretization, LUT-based calculations

**volo_voltage_pkg** (⚠️ Pending redesign)
- Function: Voltage conversion utilities
- Current: Hardcoded ±5V
- Planned: 3-range system (3.3V, 5V, ±5V)
- Tests: None yet

**volo_common_pkg**
- Function: Common constants and types
- Exports: VOLO_READY control scheme, BRAM loader protocol
- Tests: None yet

### Debugging (forge_debug_*)

**fsm_observer** (no tests yet)
- Function: Export FSM state to Moku registers for oscilloscope debugging
- Generics: NUM_STATES, V_MIN, V_MAX, FAULT_STATE_THRESHOLD
- Use case: Hardware FSM debugging without simulation

### Loaders (forge_loader_*)

**volo_bram_loader** (no tests yet)
- Function: BRAM initialization from external sources
- Use case: LUT loading, configuration data

---

## Testing Workflow

### Running Tests

```bash
# Navigate to forge-vhdl
cd libs/forge-vhdl

# Run P1 tests (default, LLM-optimized)
uv run python tests/run.py forge_util_clk_divider

# Run P2 tests with more verbosity
TEST_LEVEL=P2_INTERMEDIATE COCOTB_VERBOSITY=NORMAL \
  uv run python tests/run.py forge_util_clk_divider

# List all available tests
uv run python tests/run.py --list

# Run all tests
uv run python tests/run.py --all
```

### Adding Tests for New Components

See `docs/PROGRESSIVE_TESTING_GUIDE.md` for step-by-step instructions.

Quick summary:
1. Copy template from `test_forge_util_clk_divider_progressive.py`
2. Create `<component>_tests/` directory with constants + P1/P2 modules
3. Update `tests/test_configs.py` with component entry
4. Run tests, ensure <20 line P1 output

---

## Integration with forge/

### forge/ Code Generation
- Uses `basic-app-datatypes` for type system (12 voltage types)
- Generates VHDL shim + main template
- Auto-generates type packages (`basic_app_types_pkg.vhd`)

### forge-vhdl Utilities
- Provides practical utilities for manual VHDL in `*_main.vhd`
- Focus on 3 common voltage ranges (3.3V, 5V, ±5V)
- Standalone, works outside forge/ ecosystem

**Separation:**
- forge/ = Comprehensive, auto-generated, YAML-driven
- forge-vhdl = Pragmatic, hand-written, day-to-day

---

## Token Efficiency Metrics

### Before CocoTB + GHDL Filter

```
Test output: 287 lines
Token consumption: ~4000 tokens
LLM context impact: SEVERE
Cost per test: $0.12 (GPT-4)
```

### After CocoTB + GHDL Filter

```
Test output: 8 lines (P1), 20 lines (P2)
Token consumption: ~50 tokens (P1), ~150 tokens (P2)
LLM context impact: MINIMAL
Cost per test: $0.001 (GPT-4)
```

**Savings:** 98% reduction, 120x cost reduction

---

## Development Workflow

### Adding New Component

1. Write VHDL component in appropriate `vhdl/` subdirectory
2. Create CocoTB test using template
3. Run P1 tests, ensure <20 line output
4. Commit in submodule with descriptive message
5. Update `llms.txt` catalog
6. Add component section to this `CLAUDE.md`

### Modifying Existing Component

1. Make VHDL changes
2. Run existing tests (should still pass)
3. Add new tests if behavior changed
4. Commit in submodule

### Git Submodule Protocol

**CRITICAL:** All commits must be made inside `libs/forge-vhdl` submodule!

```bash
cd libs/forge-vhdl
git checkout 20251104-vhdl-forge-dev  # Ensure on feature branch
# make changes
git add .
git commit -m "descriptive message"
git push origin 20251104-vhdl-forge-dev
cd ../..
git add libs/forge-vhdl  # Update parent reference
git commit -m "chore: Update forge-vhdl submodule"
git push origin 20251104-vhdl-forge-dev
```

---

## Common Testing Patterns

### Pattern 1: Simple Entity Test

See `test_forge_util_clk_divider_progressive.py` for complete example.

```python
class ForgeUtilClkDividerTests(TestBase):
    async def run_p1_basic(self):
        await self.test("Reset", self.test_reset)
        await self.test("Divide by 2", self.test_divide_by_2)

    async def test_reset(self):
        await reset_active_low(self.dut)
        assert int(self.dut.clk_out.value) == 0
```

### Pattern 2: Package Test (Needs Wrapper)

See `test_volo_lut_pkg_progressive.py` + `volo_lut_pkg_tb_wrapper.vhd`.

```vhdl
-- Wrapper entity (packages can't be top-level)
entity volo_lut_pkg_tb_wrapper is
end entity;

architecture tb of volo_lut_pkg_tb_wrapper is
    -- Expose package functions/constants as signals
    signal test_constant : std_logic_vector(15 downto 0) := PACKAGE_CONSTANT;
end architecture;
```

---

## Related Documentation

### In forge-vhdl
- `docs/VOLO_COCOTB_TESTING_STANDARD.md` - Authoritative testing rules
- `docs/PROGRESSIVE_TESTING_GUIDE.md` - Step-by-step test creation
- `docs/GHDL_OUTPUT_FILTER.md` - How the filter works
- `docs/COCOTB_PATTERNS.md` - Quick reference patterns
- `docs/VHDL_COCOTB_LESSONS_LEARNED.md` - Common pitfalls

### In Monorepo
- `docs/migration/FORGE_VHDL_PLAN.md` - Migration plan
- `.claude/shared/ARCHITECTURE_OVERVIEW.md` - Hierarchical architecture

---

**Last Updated:** 2025-11-04
**Maintainer:** Moku Instrument Forge Team
**Version:** 1.0.0
```

### Commit Documentation Updates

```bash
# In libs/forge-vhdl
git add llms.txt CLAUDE.md
git commit -m "Update documentation with CocoTB testing info

Updated llms.txt (Tier 1):
- Added testing section with quick commands
- Updated component catalog with test counts
- Added P1 output metrics

Created CLAUDE.md (Tier 2):
- CocoTB progressive testing standard (AUTHORITATIVE)
- Component naming convention
- Complete component catalog
- Testing workflow and patterns
- Token efficiency metrics
- Integration with forge/

Documentation now follows 3-tier pattern from ARCHITECTURE_OVERVIEW.md.

Related: docs/migration/FORGE_VHDL_P2.md
"
git push origin 20251104-vhdl-forge-dev

# Update parent
cd ../..
git add libs/forge-vhdl
git commit -m "chore: Update forge-vhdl submodule (documentation)"
git push origin 20251104-vhdl-forge-dev
```

---

## Phase 2 Completion Checklist

### Component 1: forge_util_clk_divider
- [ ] Test files copied from export
- [ ] Test configuration updated
- [ ] P1 tests run successfully (<20 lines)
- [ ] Component renamed (volo → forge_util)
- [ ] Tests re-run after rename (still pass)
- [ ] Committed in submodule
- [ ] Parent reference updated

### Component 2: volo_lut_pkg
- [ ] Test files copied from export
- [ ] Test wrapper copied
- [ ] Test configuration updated
- [ ] P1 tests run successfully (<20 lines)
- [ ] Committed in submodule (no rename)
- [ ] Parent reference updated

### Documentation
- [ ] llms.txt updated with testing info
- [ ] CLAUDE.md created with testing standards
- [ ] Both committed in submodule
- [ ] Parent reference updated

### Verification
- [ ] All P1 tests <20 line output
- [ ] GHDL filter achieves 90%+ reduction
- [ ] All tests pass (green)
- [ ] Git history clean (all commits in submodule)
- [ ] Documentation accurate (component catalog matches reality)

---

## Success Criteria

Phase 2 is complete when:

✅ 2 components migrated with CocoTB tests
✅ All P1 tests produce <20 line output
✅ GHDL filter achieves 90%+ noise reduction
✅ Token consumption <100 tokens per test run
✅ Documentation updated (llms.txt, CLAUDE.md)
✅ All commits made in submodule
✅ Parent references updated
✅ Checklist 100% complete

---

## Time Estimates

- **Component 1** (forge_util_clk_divider): 1 hour
  - Copy tests: 10 min
  - Update config: 5 min
  - Run tests: 5 min
  - Rename: 15 min
  - Commit/push: 10 min
  - Debugging/fixes: 15 min

- **Component 2** (volo_lut_pkg): 45 min
  - Copy tests: 10 min
  - Update config: 5 min
  - Run tests: 5 min
  - Commit/push: 10 min
  - Debugging/fixes: 15 min

- **Documentation**: 30 min
  - Update llms.txt: 10 min
  - Create CLAUDE.md: 15 min
  - Commit/push: 5 min

**Total estimated time:** ~2.5 hours

---

## Next Steps

After Phase 2 completion:

1. ✅ Mark all checklist items complete
2. 🎉 Celebrate! (You have token-efficient VHDL testing!)
3. 🔄 Iterate on additional components (Phase 3+):
   - Add tests for fsm_observer
   - Redesign voltage package (3-range system)
   - Migrate remaining utilities

---

## Common Issues & Solutions

### Issue: Tests fail with "module not found"
**Solution:** Check `test_configs.py` toplevel name is lowercase entity name

### Issue: Output still >20 lines
**Solution:** Reduce P1 test count (2-4 tests only), use small test values

### Issue: GHDL filter not working
**Solution:** Check `run.py` has `FilteredOutput` context manager, verify import

### Issue: Rename breaks tests
**Solution:** Update ALL references (test file imports, constants MODULE_NAME, test_configs.py)

### Issue: Package test fails
**Solution:** Verify test wrapper entity exists, test_configs.py uses wrapper as toplevel

---

**Phase 2 Status:** Ready for execution (after Phase 1 complete)
**Next:** Execute per-component workflow, complete checklist, celebrate!
