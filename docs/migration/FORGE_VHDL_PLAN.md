# forge-vhdl CocoTB Progressive Testing Migration Plan

**Version:** 1.0
**Created:** 2025-11-04
**Status:** Active
**Target:** `libs/forge-vhdl` submodule

---

## Executive Summary

Migrate proven CocoTB progressive testing infrastructure from EZ-EMFI project into `libs/forge-vhdl` submodule, enabling **token-efficient AI-assisted VHDL iteration** with <20 line test output (98% reduction from 287 lines).

**Key Innovation:** GHDL Output Filter + Progressive Testing (P1→P2→P3→P4) + TestBase framework = LLM-friendly VHDL development.

---

## Migration Philosophy

### Alignment with ARCHITECTURE_OVERVIEW.md

This migration continues the elegant hierarchical architecture established in `.claude/shared/ARCHITECTURE_OVERVIEW.md`:

**Property 1: Self-Contained Authoritative Bubble**
- `libs/forge-vhdl` becomes authoritative for VHDL utilities + testing infrastructure
- Zero dependencies on monorepo (works standalone)
- Can be used by other projects

**Property 3: Three-Tier Documentation System**
- **Tier 1**: `llms.txt` - Quick component catalog (~800 tokens)
- **Tier 2**: `CLAUDE.md` - Testing standards, design patterns (~3-5k tokens)
- **Tier 3**: Source code + CocoTB tests (load as needed)

**Property 5: Token-Efficient Context Loading**
- Start with Tier 1 (~1k tokens)
- Progressive escalation (P1 tests → P2 tests → P3 tests)
- Reserve 190k+ tokens for iterative development

---

## The Problem We're Solving

### Before: VHDL Testing Was Context-Hostile

```bash
$ ghdl -r volo_clk_divider --wave=sim.vcd
[287 lines of GHDL output]
     0.00ns INFO cocotb.gpi ...
@0ms:(assertion warning): NUMERIC_STD.TO_INTEGER: metavalue detected, returning 0
[repeated 50+ times]
[200+ more lines of elaboration noise]

Token consumption: ~4000 tokens
LLM reads: ~5% of output (misses critical info)
Context window impact: SEVERE
```

### After: LLM-Optimized Testing

```bash
$ uv run python tests/run.py volo_clk_divider
P1 - BASIC TESTS
T1: Reset behavior
  ✓ PASS
T2: Divide by 2
  ✓ PASS
T3: Enable control
  ✓ PASS
ALL 3 TESTS PASSED

Token consumption: ~50 tokens (98% reduction!)
LLM reads: 100% of output
Context window impact: MINIMAL
```

---

## Two-Phase Migration Strategy

### Phase 1: Infrastructure Setup (Foundation)
**Goal:** Install CocoTB + GHDL filter infrastructure into `libs/forge-vhdl`
**Scope:** Copy files, create directory structure, NO component migration
**Outcome:** Testing framework ready, docs in place
**Duration:** ~30 minutes (mostly copying)

### Phase 2: Component Migration (Iterative)
**Goal:** Migrate components with CocoTB tests, rename to `forge_*` convention
**Scope:** `volo_clk_divider`, `volo_lut_pkg` (+ tests from export)
**Outcome:** Validated components with <20 line P1 tests, docs updated
**Duration:** ~2 hours (per component)

---

## Git Submodule Commit Protocol

### ⚠️ CRITICAL: The #1 Mistake to Avoid

**WRONG** (commits in parent monorepo):
```bash
# ❌ BAD - commits in wrong repo!
cd /Users/johnycsh/TTOP/moku-instrument-forge-mono-repo
cp file.py libs/forge-vhdl/scripts/
git add libs/forge-vhdl/scripts/file.py
git commit -m "Add file.py"  # WRONG REPO!
```

**CORRECT** (commits in submodule):
```bash
# ✅ GOOD - commits in submodule first
cd /Users/johnycsh/TTOP/moku-instrument-forge-mono-repo/libs/forge-vhdl
cp /tmp/file.py scripts/
git add scripts/file.py
git commit -m "Add file.py to testing infrastructure

Copied from EZ-EMFI CocoTB progressive export.
Enables GHDL output filtering with 98% noise reduction.
"
git push origin main  # Push submodule changes

# Then update parent reference
cd ../..
git add libs/forge-vhdl  # Update submodule reference
git commit -m "chore: Update forge-vhdl submodule with testing infrastructure"
git push origin main
```

### Commit Message Style

Per user preference: **"What you'd normally output to terminal"**

**Good commit message:**
```
Add GHDL output filter for token-efficient testing

Copied ghdl_output_filter.py from EZ-EMFI CocoTB export.
This 340-line Python script provides intelligent GHDL output filtering:
- 4 filter levels (AGGRESSIVE/NORMAL/MINIMAL/NONE)
- 90-98% output reduction (287 lines → 8 lines)
- Preserves all errors, failures, test results
- Operates at OS file descriptor level (bulletproof)

Files added:
- scripts/ghdl_output_filter.py (340 lines)

Dependencies: Python 3.7+ stdlib only (no external packages)

This is THE critical enabler for <20 line P1 test output.
```

**Key elements:**
1. What was done (natural language)
2. Why it matters (context)
3. Technical details (files, dependencies)
4. Impact (quantified if possible)

---

## Directory Structure After Migration

```
libs/forge-vhdl/                          # Git submodule (self-contained)
├── vhdl/
│   ├── packages/                         # VHDL packages
│   │   ├── volo_common_pkg.vhd           # Existing
│   │   ├── volo_lut_pkg.vhd              # Existing → will have tests
│   │   └── volo_voltage_pkg.vhd          # Existing (deferred redesign)
│   │
│   ├── utilities/                        # Generic utilities
│   │   ├── volo_clk_divider.vhd          # Existing → rename forge_util_clk_divider
│   │   └── volo_voltage_threshold_trigger_core.vhd
│   │
│   ├── debugging/                        # Debug infrastructure
│   │   └── fsm_observer.vhd              # Existing (no export version)
│   │
│   └── loader/
│       └── volo_bram_loader.vhd          # Existing
│
├── scripts/                              # NEW in Phase 1
│   └── ghdl_output_filter.py             # THE SECRET WEAPON
│
├── tests/                                # NEW in Phase 1
│   ├── test_base.py                      # Progressive test framework
│   ├── conftest.py                       # Shared utilities (clock, reset)
│   ├── run.py                            # Optimized test runner
│   ├── test_configs.py                   # Test registry
│   │
│   ├── test_forge_util_clk_divider.py    # NEW in Phase 2
│   ├── forge_util_clk_divider_tests/     # NEW in Phase 2
│   │   ├── __init__.py
│   │   ├── forge_util_clk_divider_constants.py
│   │   ├── P1_forge_util_clk_divider_basic.py
│   │   └── P2_forge_util_clk_divider_intermediate.py
│   │
│   └── test_volo_lut_pkg.py              # NEW in Phase 2
│       └── volo_lut_pkg_tests/           # NEW in Phase 2
│
├── docs/                                 # NEW in Phase 1
│   ├── GHDL_OUTPUT_FILTER.md             # Filter documentation
│   ├── VOLO_COCOTB_TESTING_STANDARD.md   # Authoritative testing rules
│   ├── PROGRESSIVE_TESTING_GUIDE.md      # Step-by-step conversion
│   ├── COCOTB_PATTERNS.md                # Quick reference
│   └── VHDL_COCOTB_LESSONS_LEARNED.md    # Common pitfalls
│
├── llms.txt                              # UPDATE in Phase 2
├── CLAUDE.md                             # CREATE in Phase 2
├── README.md                             # Existing
└── pyproject.toml                        # Existing (for Python dependencies)
```

---

## Component Version Validation

Pre-migration analysis comparing CocoTB export vs existing `libs/forge-vhdl`:

| Component | Export Path | Existing Path | Status |
|-----------|-------------|---------------|--------|
| `volo_clk_divider.vhd` | `/tmp/cocotb_progressive_export/VHDL/` | `libs/forge-vhdl/vhdl/utilities/` | ✅ **IDENTICAL** (md5: 967830ee) |
| `volo_lut_pkg.vhd` | `/tmp/cocotb_progressive_export/VHDL/packages/` | `libs/forge-vhdl/vhdl/packages/` | ✅ **IDENTICAL** (md5: da61349d) |
| `fsm_observer.vhd` | ❌ Not in export | `libs/forge-vhdl/vhdl/debugging/` | ✅ **Keep existing** |
| `volo_voltage_pkg.vhd` | `/tmp/cocotb_progressive_export/VHDL/packages/` | `libs/forge-vhdl/vhdl/packages/` | ⚠️ **Deferred** (needs redesign) |

**Decision:** Use existing VHDL files (already identical), focus on adding CocoTB tests from export.

---

## Success Criteria

### Phase 1 Complete When:
- [ ] GHDL filter copied to `libs/forge-vhdl/scripts/`
- [ ] CocoTB framework copied to `libs/forge-vhdl/tests/`
- [ ] Documentation copied to `libs/forge-vhdl/docs/`
- [ ] Directory structure validated
- [ ] All commits made **inside submodule**
- [ ] Submodule reference updated in parent monorepo
- [ ] `FORGE_VHDL_P1.md` checklist 100% complete

### Phase 2 Complete When:
- [ ] `volo_clk_divider` migrated with CocoTB tests
- [ ] `volo_lut_pkg` migrated with CocoTB tests
- [ ] All P1 tests produce <20 line output
- [ ] GHDL filter achieves 90%+ noise reduction
- [ ] `llms.txt` updated with testing info
- [ ] `CLAUDE.md` created with testing standards
- [ ] All commits made **inside submodule**
- [ ] `FORGE_VHDL_P2.md` checklist 100% complete

### Overall Success Metrics:
- **Token efficiency**: P1 test output <50 tokens (vs 4000 tokens before)
- **Context preservation**: 98% reduction in GHDL noise
- **AI iteration speed**: <20 line test output enables fast LLM feedback
- **Documentation**: Tier 1 (llms.txt) + Tier 2 (CLAUDE.md) complete

---

## The GHDL Output Filter: The Secret Weapon

### Why This Matters

**Before filter:**
```
287 lines of output
4000 tokens consumed
LLM sees: 5% signal, 95% noise
```

**After filter (AGGRESSIVE mode):**
```
8 lines of output
50 tokens consumed
LLM sees: 100% signal, 0% noise
```

### How It Works

1. **OS-Level FD Redirection**: Intercepts stdout/stderr at file descriptor level (even C code like GHDL can't bypass)
2. **Smart Filtering**: Removes metavalue warnings, null warnings, initialization noise, duplicates
3. **Critical Preservation**: NEVER filters ERROR, FAIL, PASS, assertion failures
4. **Progressive Levels**: AGGRESSIVE (98% reduction) → NORMAL (90%) → MINIMAL (70%) → NONE (0%)

### Integration

The filter is automatically integrated into `tests/run.py`:
```python
from ghdl_output_filter import GHDLOutputFilter, FilterLevel

with FilteredOutput(filter_level=FilterLevel.AGGRESSIVE):
    runner.test(...)  # All GHDL output is filtered!
```

**Zero configuration needed by users** - it just works!

---

## Progressive Testing Methodology

### The P1→P2→P3→P4 Progression

**P1 - BASIC** (Default, LLM-optimized)
- 2-4 essential tests only
- Small test values (cycles=20, not 2000)
- <20 line output, <100 tokens
- <5 second runtime
- **Use case:** Daily development, AI iteration

**P2 - INTERMEDIATE** (Standard validation)
- 5-10 tests with edge cases
- Realistic test values
- <50 line output
- <30 second runtime
- **Use case:** Pre-commit validation

**P3 - COMPREHENSIVE** (Full coverage)
- 15-25 tests with stress testing
- Boundary values, corner cases
- <100 line output
- <2 minute runtime
- **Use case:** CI/CD, release validation

**P4 - EXHAUSTIVE** (Debug mode)
- Unlimited tests, random testing
- Maximum verbosity
- Unlimited output
- Unlimited runtime
- **Use case:** Deep debugging, investigating failures

### How to Run

```bash
# Default: P1 tests, MINIMAL output (for LLMs)
uv run python tests/run.py forge_util_clk_divider

# P2 tests, NORMAL verbosity (for humans)
TEST_LEVEL=P2_INTERMEDIATE COCOTB_VERBOSITY=NORMAL \
  uv run python tests/run.py forge_util_clk_divider

# P3 tests, full output (for CI/CD)
TEST_LEVEL=P3_COMPREHENSIVE COCOTB_VERBOSITY=VERBOSE \
  uv run python tests/run.py forge_util_clk_divider

# P4 debug mode (investigating failures)
TEST_LEVEL=P4_EXHAUSTIVE COCOTB_VERBOSITY=DEBUG GHDL_FILTER_LEVEL=none \
  uv run python tests/run.py forge_util_clk_divider
```

---

## Naming Convention Migration

### Current State (Inconsistent)
```
volo_clk_divider.vhd         # "volo" prefix
fsm_observer.vhd             # no prefix
volo_voltage_pkg.vhd         # "volo" prefix
volo_lut_pkg.vhd             # "volo" prefix
```

### Target State (Phase 2)
```
forge_util_clk_divider.vhd        # forge + category + function
forge_debug_fsm_observer.vhd      # forge + category + function
forge_voltage_pkg.vhd             # forge + domain + pkg
forge_lut_pkg.vhd                 # forge + domain + pkg
```

**Pattern:** `forge_<category>_<function>` for entities, `forge_<domain>_pkg` for packages

**Categories:**
- `util_` - Generic utilities (clk_divider, edge_detector)
- `debug_` - Debug infrastructure (fsm_observer, signal_tap)
- `loader_` - Memory initialization (bram_loader)

**Transition Strategy:**
- Phase 2 renames during migration
- Original names preserved in git history
- No backwards compatibility aliases (clean cut)

---

## Deferred Items (Future Phases)

### Voltage Package Redesign (Phase 3)
**Current:** `volo_voltage_pkg.vhd` hardcoded for ±5V
**Needed:** 3-range pragmatic system (3.3V, 5V, ±5V)
**Status:** Deferred until Phase 2 complete
**Rationale:** 99% of real FPGA control uses these 3 ranges

### Additional Component Migrations (Phase 4+)
- `volo_bram_loader.vhd` → `forge_loader_bram.vhd`
- `volo_voltage_threshold_trigger_core.vhd` → `forge_util_voltage_threshold.vhd`
- New components: `forge_util_edge_detector.vhd`, `forge_debug_signal_tap.vhd`

### Integration with forge/ Code Generator (Phase 5)
- Use `libs/forge-vhdl` utilities in generated `*_main.vhd` templates
- Reference in `forge/llms.txt`: "Use forge-vhdl for utilities"
- Cross-validation: VHDL utilities ↔ basic-app-datatypes types

---

## Handoff Between Phases

### Phase 1 → Phase 2 Handoff

**Phase 1 deliverables:**
- [ ] All infrastructure files copied and committed (in submodule)
- [ ] Directory structure validated
- [ ] Documentation accessible
- [ ] Git submodule reference updated in parent
- [ ] `FORGE_VHDL_P1.md` checklist complete

**Phase 2 prerequisites:**
- [ ] Phase 1 complete and verified
- [ ] CocoTB test runner works (can run `uv run python tests/run.py --list`)
- [ ] GHDL filter validated (reduces output)
- [ ] User reviews Phase 2 plan

**Handoff validation:**
```bash
# Verify infrastructure
cd libs/forge-vhdl
ls scripts/ghdl_output_filter.py  # Should exist
ls tests/test_base.py              # Should exist
ls docs/VOLO_COCOTB_TESTING_STANDARD.md  # Should exist

# Verify git state
git status  # Should be clean (all changes committed)
cd ../..
git status  # Should show updated submodule reference
```

---

## Risk Mitigation

### Risk: Forgetting to commit in submodule
**Mitigation:** Each phase document has prominent ⚠️ warnings
**Validation:** Check `git log` in both submodule and parent

### Risk: GHDL filter too aggressive (filters real errors)
**Mitigation:** PRESERVE_PATTERNS list (never filters ERROR/FAIL)
**Validation:** Run with `GHDL_FILTER_LEVEL=none` to compare output

### Risk: Test output still too verbose
**Mitigation:** Adjust P1 test count (2-4 tests), use small test values
**Validation:** Measure output lines (<20 requirement)

### Risk: Submodule reference not updated in parent
**Mitigation:** Explicit step in each phase checklist
**Validation:** `git diff libs/forge-vhdl` shows updated commit hash

---

## Related Documentation

### Monorepo-Level
- `.claude/shared/ARCHITECTURE_OVERVIEW.md` - Hierarchical architecture philosophy
- `.claude/shared/CONTEXT_MANAGEMENT.md` - Token optimization strategy
- `llms.txt` - Monorepo entry point

### Submodule-Level (libs/forge-vhdl)
- `llms.txt` - Tier 1 quick reference (update in Phase 2)
- `CLAUDE.md` - Tier 2 design rationale (create in Phase 2)
- `docs/VOLO_COCOTB_TESTING_STANDARD.md` - Authoritative testing rules (add in Phase 1)

### Phase-Specific
- `docs/migration/FORGE_VHDL_P1.md` - Phase 1 execution plan
- `docs/migration/FORGE_VHDL_P2.md` - Phase 2 execution plan

---

## Phase Execution Commands

### Run Phase 1
```bash
# Read the plan
cat docs/migration/FORGE_VHDL_P1.md

# Execute (AI agent or manual)
# Follow checklist step-by-step
# Commit in submodule
# Update parent reference
```

### Run Phase 2
```bash
# Verify Phase 1 complete
cat docs/migration/FORGE_VHDL_P1.md  # Check all checkboxes

# Read Phase 2 plan
cat docs/migration/FORGE_VHDL_P2.md

# Execute per-component
# Commit each component in submodule
# Update parent reference after each
```

---

## Success Markers

When this migration is complete:

✅ **Token Efficiency**: <50 tokens per test run (vs 4000 before)
✅ **AI Iteration Speed**: LLM can read 100% of test output
✅ **Context Preservation**: 98% GHDL noise reduction
✅ **Testing Infrastructure**: Progressive P1→P2→P3→P4 framework
✅ **Documentation**: Tier 1 (llms.txt) + Tier 2 (CLAUDE.md) complete
✅ **Component Migration**: 2+ components with <20 line P1 tests
✅ **Git History**: Clean commits in submodule, parent references updated
✅ **Reusability**: forge-vhdl works standalone, portable to other projects

---

**The Golden Rule:**

> **"If your P1 test output exceeds 20 lines, you're doing it wrong."**

Default to silence. Escalate consciously. Preserve context religiously.

---

**Next Step:** Read `docs/migration/FORGE_VHDL_P1.md` and begin Phase 1 execution.

---

**Version History:**
- v1.0 (2025-11-04): Initial plan created, component versions validated
