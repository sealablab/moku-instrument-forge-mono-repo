# Documentation Categorization - SIMPLIFIED (v2.0)

**Date:** 2025-11-04

---

---

## 🟢 Category A: ACTIVE (Keep & Maintain)

### Root Monorepo Files
- ✅ `./README.md` - Main monorepo README (v2.0.0)
- ✅ `./llms.txt` - **UPDATED** (now reflects v2.0 architecture)
- ✅ `./WORKFLOW_GUIDE.md` - Workflow guide
- ✅ `./ARCHITECTURE_V2_COMPLETE.md` - Migration completion record

### Root docs/
- ✅ `./docs/README.md` - Docs index
- ✅ `./docs/migration/VOLTAGE_TYPE_SYSTEM_DESIGN.md` - Voltage type system design

### Root .claude/ (Monorepo Orchestration)
**Agents (3):**
- ✅ `./.claude/agents/deployment-orchestrator/agent.md`
- ✅ `./.claude/agents/hardware-debug/agent.md`
- ✅ `./.claude/agents/probe-design-orchestrator/agent.md`

**Commands (5):**
- ✅ `./.claude/commands/cross-validate.md`
- ✅ `./.claude/commands/init-probe.md`
- ✅ `./.claude/commands/probe-status.md`
- ✅ `./.claude/commands/sync-submodules.md`
- ✅ `./.claude/commands/validate-probe-structure.md`

**Shared (4):**
- ✅ `./.claude/shared/ARCHITECTURE_OVERVIEW.md` (v2.0, 678 lines)
- ⚠️ `./.claude/shared/ARCHITECTURE_OVERVIEW_LEGACY.md` (v1.0, archive candidate)
- ✅ `./.claude/shared/CONTEXT_MANAGEMENT.md`
- ✅ `./.claude/shared/PROBE_WORKFLOW.md`

**Workflows (3):**
- ✅ `./.claude/workflows/README.md`
- ✅ `./.claude/workflows/submodule-doc-refactor-detailed.md`
- ✅ `./.claude/workflows/submodule-doc-refactor-quick.md`

### tools/forge-codegen/ (Code Generator - v2.0)
**Core (4):**
- ✅ `./tools/forge-codegen/README.md`
- ✅ `./tools/forge-codegen/llms.txt` (Tier 1)
- ✅ `./tools/forge-codegen/CLAUDE.md` (Tier 2)
- ✅ `./tools/forge-codegen/MIGRATION_COMPLETE.md` (historical)

**Reference Docs (6):**
- ✅ All in `./tools/forge-codegen/docs/reference/`
  - manifest_schema.md
  - python_api.md
  - register_mapping.md
  - type_system.md
  - vhdl_generation.md
  - yaml_schema.md

**Architecture Docs (5):**
- ✅ All in `./tools/forge-codegen/docs/architecture/`
  - overview.md
  - code_generation.md
  - design_decisions.md
  - submodule_integration.md
  - agent_system.md

**Examples & Guides (6):**
- ✅ `./tools/forge-codegen/docs/examples/` (3 files)
  - common_patterns.md
  - minimal_walkthrough.md
  - multi_channel_walkthrough.md
- ✅ `./tools/forge-codegen/docs/guides/` (3 files)
  - getting_started.md
  - troubleshooting.md
  - user_guide.md

**Debugging (1):**
- ✅ `./tools/forge-codegen/docs/debugging/fsm_observer_pattern.md`

**README:**
- ✅ `./tools/forge-codegen/docs/README.md`

### libs/forge-vhdl/ (VHDL Utilities)
**Core (3):**
- ✅ `./libs/forge-vhdl/README.md`
- ✅ `./libs/forge-vhdl/llms.txt` (Tier 1)
- ✅ `./libs/forge-vhdl/CLAUDE.md` (Tier 2)

**Detailed Docs (3):**
- ✅ `./libs/forge-vhdl/docs/VHDL_CODING_STANDARDS.md`
- ✅ `./libs/forge-vhdl/docs/COCOTB_TROUBLESHOOTING.md`
- ✅ `./libs/forge-vhdl/scripts/GHDL_FILTER.md`

**.claude:**
- ✅ `./libs/forge-vhdl/.claude/settings.local.json` (only)

### libs/moku-models/ (Platform Specs)
**Core (3):**
- ✅ `./libs/moku-models/README.md`
- ✅ `./libs/moku-models/llms.txt` (Tier 1)
- ✅ `./libs/moku-models/CLAUDE.md` (Tier 2)

**Detailed Docs (2):**
- ✅ `./libs/moku-models/docs/MOKU_PLATFORM_SPECIFICATIONS.md`
- ✅ `./libs/moku-models/docs/routing_patterns.md`

**.claude:**
- ✅ `./libs/moku-models/.claude/settings.local.json` (only)

### libs/riscure-models/ (Probe Specs)
**Core (3):**
- ✅ `./libs/riscure-models/README.md`
- ✅ `./libs/riscure-models/llms.txt` (Tier 1)
- ✅ `./libs/riscure-models/CLAUDE.md` (Tier 2)

---

## 🟡 Category B: ARCHIVE (Keep in archive/ directories)

### tools/forge-codegen/docs/archive/ (Already Archived)
- 📦 `bad-first-draft-claude.md`
- 📦 `HUMAN_START_HERE_plan.md`
- 📦 `START_DS1150_EX_MIGRATION_PROMPT.md`

### tools/forge-codegen/docs/phase6_archive/ (Already Archived)
- 📦 9 files (P6-Qs.md, PHASE6_*.md prompts)

### tools/forge-codegen/docs/ (Move to Archive)
- 📦 `PHASE6_PLAN.md` → **MOVE** to `phase6_archive/`
- 📦 `PHASE6F_PROMPT.md` → **MOVE** to `phase6_archive/`

### libs/forge-vhdl/docs/archive/ (Already Archived)
- 📦 5 files dated 2025-11-04 (consolidated docs)

### Root .claude/shared/ (Candidate)
- 📦 `ARCHITECTURE_OVERVIEW_LEGACY.md` → **CONSIDER** moving to `archive/` or keeping as historical reference

---

## 🔴 Category C: REMOVED (Already Cleaned)

### Entire forge/ Directory - ✅ REMOVED
- All ~60 duplicate files **ELIMINATED**
- Git submodule unregistered
- .gitmodules updated

**What was removed:**
- forge/.claude/ (duplicate agents/commands)
- forge/docs/ (exact duplicates of tools/forge-codegen/docs/)
- forge/libs/basic-app-datatypes/ (flattened into forge-codegen)
- forge/libs/moku-models/ (duplicate of libs/moku-models/)
- forge/libs/riscure-models/ (duplicate of libs/riscure-models/)
- forge/apps/ (example probes)
- forge/README.md, forge/llms.txt

---

## Remaining Actions

### Immediate
1. ✅ **DONE** - Remove forge/ submodule
2. ✅ **DONE** - Update root llms.txt to v2.0 paths
3. ⏳ **TODO** - Move PHASE6*.md files to archive
4. ⏳ **TODO** - Decide on ARCHITECTURE_OVERVIEW_LEGACY.md

### Short-term
5. ⏳ **TODO** - Write root CLAUDE.md
6. ⏳ **TODO** - Update README.md if needed (check for forge/ references)
7. ⏳ **TODO** - Update ARCHITECTURE_V2_COMPLETE.md if needed

### Long-term
8. Initialize libs/ submodules: `git submodule update --init --recursive`
9. Verify all documentation is consistent
10. Tag as v2.0.0 final

---

## Current Git Status

```
M  .gitmodules          # forge entry removed
D  forge                # submodule deleted
M  llms.txt            # updated to v2.0 paths
?? DOC_CATEGORIZATION.md
?? DOC_CATEGORIZATION_v2.md
```

---

## Documentation Statistics (Post-Cleanup)

| Category | Count | Status |
|----------|-------|--------|
| **Active markdown files** | ~75 | ✅ All v2.0 aligned |
| **Active llms.txt files** | 5 | ✅ No duplication |
| **.claude directories** | 4 | ✅ Clean hierarchy |
| **Archived files** | ~20 | 📦 Organized |
| **Removed (duplicates)** | ~60 | ✅ Eliminated |

---

## Clean Architecture Achieved ✅

### Before (v1.0)
```
forge/                           # Nested submodules
├── libs/
│   ├── basic-app-datatypes/
│   ├── moku-models/
│   └── riscure-models/
├── .claude/ (5 agents)
└── docs/ (complete duplicate)
```

### After (v2.0)
```
tools/
└── forge-codegen/              # Clean code generator

libs/                           # Flat foundational libraries
├── forge-vhdl/
├── moku-models/
└── riscure-models/

.claude/                        # Monorepo orchestration
```

**Result:**
- No duplication
- Clear separation
- Clean git submodule structure
- Ready for root CLAUDE.md
