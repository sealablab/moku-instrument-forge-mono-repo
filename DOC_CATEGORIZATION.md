# Documentation Categorization & Cleanup Plan

**Date:** 2025-11-04
**Purpose:** Categorize all documentation for v2.0 architecture cleanup

---

## 🟢 Category A: NEW / KEEP (v2.0 Aligned, Current)

### Root Monorepo
- ✅ `./README.md` - **KEEP** (v2.0.0, current)
- ⚠️ `./llms.txt` - **NEEDS UPDATE** (references old paths like forge/libs/basic-app-datatypes)
- ✅ `./WORKFLOW_GUIDE.md` - **KEEP**
- ✅ `./ARCHITECTURE_V2_COMPLETE.md` - **KEEP** (migration record)

### Root docs/
- ✅ `./docs/README.md` - **KEEP**
- ✅ `./docs/migration/VOLTAGE_TYPE_SYSTEM_DESIGN.md` - **KEEP** (design doc)

### Root .claude/ (Monorepo Orchestration)
**Agents:**
- ✅ `./.claude/agents/deployment-orchestrator/agent.md` - **KEEP**
- ✅ `./.claude/agents/hardware-debug/agent.md` - **KEEP**
- ✅ `./.claude/agents/probe-design-orchestrator/agent.md` - **KEEP**

**Commands:**
- ✅ `./.claude/commands/cross-validate.md` - **KEEP**
- ✅ `./.claude/commands/init-probe.md` - **KEEP**
- ✅ `./.claude/commands/probe-status.md` - **KEEP**
- ✅ `./.claude/commands/sync-submodules.md` - **KEEP**
- ✅ `./.claude/commands/validate-probe-structure.md` - **KEEP**

**Shared:**
- ✅ `./.claude/shared/ARCHITECTURE_OVERVIEW.md` - **KEEP** (v2.0, 678 lines)
- ✅ `./.claude/shared/CONTEXT_MANAGEMENT.md` - **KEEP**
- ✅ `./.claude/shared/PROBE_WORKFLOW.md` - **KEEP**

**Workflows:**
- ✅ `./.claude/workflows/README.md` - **KEEP**
- ✅ `./.claude/workflows/submodule-doc-refactor-detailed.md` - **KEEP**
- ✅ `./.claude/workflows/submodule-doc-refactor-quick.md` - **KEEP**

### tools/forge-codegen/ (NEW Code Generator)
**Core:**
- ✅ `./tools/forge-codegen/README.md` - **KEEP**
- ✅ `./tools/forge-codegen/llms.txt` - **KEEP** (Tier 1)
- ✅ `./tools/forge-codegen/CLAUDE.md` - **KEEP** (Tier 2)
- ✅ `./tools/forge-codegen/MIGRATION_COMPLETE.md` - **KEEP** (historical record)

**Reference Docs:**
- ✅ All 6 files in `./tools/forge-codegen/docs/reference/` - **KEEP**

**Architecture Docs:**
- ✅ All 5 files in `./tools/forge-codegen/docs/architecture/` - **KEEP**

**Examples & Guides:**
- ✅ All 3 files in `./tools/forge-codegen/docs/examples/` - **KEEP**
- ✅ All 3 files in `./tools/forge-codegen/docs/guides/` - **KEEP**

**Debugging:**
- ✅ `./tools/forge-codegen/docs/debugging/fsm_observer_pattern.md` - **KEEP**

### libs/ (NEW Flat Libraries)
**forge-vhdl:**
- ✅ `./libs/forge-vhdl/README.md` - **KEEP**
- ✅ `./libs/forge-vhdl/llms.txt` - **KEEP**
- ✅ `./libs/forge-vhdl/CLAUDE.md` - **KEEP**
- ✅ `./libs/forge-vhdl/docs/VHDL_CODING_STANDARDS.md` - **KEEP**
- ✅ `./libs/forge-vhdl/docs/COCOTB_TROUBLESHOOTING.md` - **KEEP**
- ✅ `./libs/forge-vhdl/scripts/GHDL_FILTER.md` - **KEEP**

**moku-models:**
- ✅ `./libs/moku-models/README.md` - **KEEP**
- ✅ `./libs/moku-models/llms.txt` - **KEEP**
- ✅ `./libs/moku-models/CLAUDE.md` - **KEEP**
- ✅ `./libs/moku-models/docs/MOKU_PLATFORM_SPECIFICATIONS.md` - **KEEP**
- ✅ `./libs/moku-models/docs/routing_patterns.md` - **KEEP**

**riscure-models:**
- ✅ `./libs/riscure-models/README.md` - **KEEP**
- ✅ `./libs/riscure-models/llms.txt` - **KEEP**
- ✅ `./libs/riscure-models/CLAUDE.md` - **KEEP**

---

## 🟡 Category B: TRANSITORY (Archive for Reference)

### Root .claude/
- 📦 `./.claude/shared/ARCHITECTURE_OVERVIEW_LEGACY.md` - **ARCHIVE** (v1.0 historical reference)

### tools/forge-codegen/ Archives
- 📦 `./tools/forge-codegen/docs/archive/` (entire directory, 3 files) - **ALREADY ARCHIVED**
  - bad-first-draft-claude.md
  - HUMAN_START_HERE_plan.md
  - START_DS1150_EX_MIGRATION_PROMPT.md

- 📦 `./tools/forge-codegen/docs/phase6_archive/` (entire directory, 9 files) - **ALREADY ARCHIVED**
  - P6-Qs.md
  - PHASE6_DOCUMENTATION_PROMPT.md
  - PHASE6_QUICKSTART.md
  - PHASE6B_PROMPT.md
  - PHASE6C_PROMPT.md
  - PHASE6D_PROMPT.md
  - PHASE6D_PROMPT_FIXED.md
  - PHASE6E_PROMPT.md

- 📦 `./tools/forge-codegen/docs/PHASE6_PLAN.md` - **MOVE TO ARCHIVE**
- 📦 `./tools/forge-codegen/docs/PHASE6F_PROMPT.md` - **MOVE TO ARCHIVE**

### libs/forge-vhdl/ Archives
- 📦 `./libs/forge-vhdl/docs/archive/` (entire directory, 5 files dated 2025-11-04) - **ALREADY ARCHIVED**
  - 2025-11-04_COCOTB_PATTERNS.md
  - 2025-11-04_GHDL_OUTPUT_FILTER.md
  - 2025-11-04_PROGRESSIVE_TESTING_GUIDE.md
  - 2025-11-04_VHDL_QUICK_REF.md
  - 2025-11-04_VOLO_COCOTB_TESTING_STANDARD.md

---

## 🔴 Category C: OLD / OUTDATED (Remediate or Remove)

### ENTIRE forge/ Directory Structure
**Status:** DEPRECATED v1.0 architecture (kept as reference submodule)

**Decision needed:**
- Option 1: Keep entire `forge/` as-is (frozen reference, marked DEPRECATED in README)
- Option 2: Remove most `forge/` docs, keep only submodule pointer
- Option 3: Archive `forge/` entirely (git tag, then remove from active branch)

**Contents:**

#### forge/.claude/ (DUPLICATE of tools/forge-codegen agent system)
- 🔴 `./forge/.claude/agents/` (5 agents) - **DUPLICATE** with tools/forge-codegen structure
  - forge-context/agent.md
  - deployment-context/agent.md
  - docgen-context/agent.md
  - hardware-debug-context/agent.md
  - workflow-coordinator/agent.md + template

- 🔴 `./forge/.claude/commands/` (6 commands) - **DUPLICATE** functionality
  - forge.md
  - deployment.md
  - docgen.md
  - debug.md
  - platform.md
  - workflow.md

- 🔴 `./forge/.claude/shared/` (4 files) - **EVALUATE**
  - `package_contract.md` - May still be useful reference
  - `type_system_quick_ref.md` - Superseded by tools/forge-codegen
  - `riscure_probe_integration.md` - May still be useful
  - `SERENA_MIGRATION_ASSESSMENT.md` - Historical context only

#### forge/docs/ (COMPLETE DUPLICATE of tools/forge-codegen/docs/)
- 🔴 `./forge/docs/architecture/` (5 files) - **EXACT DUPLICATE**
- 🔴 `./forge/docs/reference/` (6 files) - **EXACT DUPLICATE**
- 🔴 `./forge/docs/examples/` (3 files) - **EXACT DUPLICATE**
- 🔴 `./forge/docs/guides/` (3 files) - **EXACT DUPLICATE**
- 🔴 `./forge/docs/debugging/fsm_observer_pattern.md` - **EXACT DUPLICATE**
- 🔴 `./forge/docs/archive/` (3 files) - **DUPLICATE ARCHIVE**
- 🔴 `./forge/docs/phase6_archive/` (9 files) - **DUPLICATE ARCHIVE**
- 🔴 `./forge/docs/PHASE6_PLAN.md` - **DUPLICATE**
- 🔴 `./forge/docs/PHASE6F_PROMPT.md` - **DUPLICATE**

#### forge/libs/ (DUPLICATE Nested Submodules)
**Status:** OLD nested submodule structure, superseded by flat libs/

**basic-app-datatypes (NO LONGER EXISTS IN v2.0):**
- 🔴 `./forge/libs/basic-app-datatypes/README.md` - **REMOVE** (flattened into forge-codegen)
- 🔴 `./forge/libs/basic-app-datatypes/llms.txt` - **REMOVE**
- 🔴 `./forge/libs/basic-app-datatypes/CLAUDE.md` - **REMOVE**
- 🔴 `./forge/libs/basic-app-datatypes/.claude/commands/library.md` - **REMOVE**

**moku-models (DUPLICATE):**
- 🔴 `./forge/libs/moku-models/*` - **All files DUPLICATE** of `./libs/moku-models/`
  - README.md, llms.txt, CLAUDE.md (identical)
  - docs/MOKU_PLATFORM_SPECIFICATIONS.md (identical)
  - docs/routing_patterns.md (identical)

**riscure-models (DUPLICATE):**
- 🔴 `./forge/libs/riscure-models/*` - **All files DUPLICATE** of `./libs/riscure-models/`
  - README.md, llms.txt, CLAUDE.md (identical)

**Meta:**
- 🔴 `./forge/libs/MODELS_INDEX.md` - **EVALUATE** (may have useful integration patterns)

#### forge/ Core Docs
- 🔴 `./forge/README.md` - **UPDATE or REMOVE** (should just say "DEPRECATED, see tools/forge-codegen")
- 🔴 `./forge/llms.txt` - **UPDATE or REMOVE** (should redirect to tools/forge-codegen/llms.txt)

#### forge/apps/
- ⚠️ `./forge/apps/DS1140_PD/README.md` - **EVALUATE** (example probe, may be useful reference)

---

## Recommended Actions

### Immediate (Before Writing Root CLAUDE.md)

1. **Update outdated references in active files:**
   - ✏️ `./llms.txt` - Update paths from `forge/libs/basic-app-datatypes` → `tools/forge-codegen/forge_codegen/basic_serialized_datatypes` (internal)
   - ✏️ `./llms.txt` - Update references to nested structure → flat structure

2. **Move transitory files to archive:**
   - 📦 Move `./tools/forge-codegen/docs/PHASE6_PLAN.md` → `phase6_archive/`
   - 📦 Move `./tools/forge-codegen/docs/PHASE6F_PROMPT.md` → `phase6_archive/`

3. **Decision: forge/ directory handling**
   - **Recommend:** Add `forge/DEPRECATED.md` at root explaining:
     - This is legacy v1.0 architecture
     - Use `tools/forge-codegen/` for new development
     - Kept as git submodule for reference only
     - Do not update documentation here

### Near-term (Post Root CLAUDE.md)

4. **Create forge/DEPRECATED.md:**
```markdown
# DEPRECATED: Legacy Architecture

This directory contains the **deprecated v1.0 architecture** with nested submodules.

**Do not use this for new development.**

## Replacement

Use the **v2.0 architecture** instead:
- **Code generation:** `tools/forge-codegen/`
- **Foundational libraries:** `libs/forge-vhdl/`, `libs/moku-models/`, `libs/riscure-models/`

## Why Deprecated?

The v1.0 architecture had:
- Nested git submodules (complex maintenance)
- Ambiguous "forge" naming (forge vs forge generator)
- basic-app-datatypes as separate library (now internal to forge-codegen)

The v2.0 architecture has:
- Flat library structure (clean separation)
- Clear naming (forge-codegen, forge-vhdl)
- No nested submodules

## Contents

This directory is kept as a **frozen reference** for:
- Historical context
- Migration documentation
- Legacy probe examples in `apps/`

**Last updated:** 2025-11-04 (frozen at v1.0 state)
```

5. **Update forge/README.md:**
   - Add prominent DEPRECATED notice at top
   - Link to tools/forge-codegen/

6. **Update forge/llms.txt:**
   - Add DEPRECATED notice
   - Redirect to tools/forge-codegen/llms.txt

### Optional (Future Cleanup)

7. **Remove duplicate docs in forge/:**
   - All of `forge/docs/` (duplicated in tools/forge-codegen/docs/)
   - All of `forge/.claude/` (duplicated agent structure)
   - All of `forge/libs/` documentation (duplicated in libs/)

8. **Keep minimal forge/ structure:**
   - forge/DEPRECATED.md
   - forge/README.md (with deprecation notice)
   - forge/apps/ (example probes)
   - forge/platform/ (if has unique content)
   - forge/scripts/ (if has unique utilities)

---

## Summary Statistics

| Category | Count | Action |
|----------|-------|--------|
| **Keep (Active v2.0)** | ~85 files | No change, maintain |
| **Transitory (Archive)** | ~20 files | Already archived or move to archive |
| **Outdated (Deprecate/Remove)** | ~60+ files | Deprecate forge/, remove duplicates |

**Key Insight:** Almost all duplication stems from the v1.0 → v2.0 migration. The `forge/` directory contains nearly complete duplicates of what's now in `tools/forge-codegen/` and flat `libs/`.

---

## Questions for Review

1. **forge/ directory fate?**
   - Keep as frozen DEPRECATED reference?
   - Remove most docs, keep only submodule pointer?
   - Archive entirely (git tag, remove from main branch)?

2. **forge/.claude/ agents?**
   - Are they still used by any workflows?
   - Or completely superseded by root .claude/ + tools/forge-codegen structure?

3. **forge/libs/MODELS_INDEX.md?**
   - Any unique content vs current docs?
   - Worth migrating useful parts to root docs/?

4. **forge/apps/ examples?**
   - Still useful as reference?
   - Or migrate to tools/forge-codegen/examples/?

5. **Root llms.txt scope?**
   - Should it be comprehensive monorepo overview?
   - Or minimal "navigator" pointing to submodule llms.txt files?
