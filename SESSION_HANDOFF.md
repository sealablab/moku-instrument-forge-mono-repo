# Session Handoff: .claude/ Directory Setup

**Date:** 2025-11-03
**Session:** Claude Code configuration for hierarchical monorepo
**Status:** Phase 1 & 2 Complete, Phase 3 Ready for Implementation
**Context Used:** ~114k/200k tokens

---

## What Was Accomplished

### ✅ Phase 1: Standardized Foundational Pydantic Model Libraries

**Goal:** Establish consistent two-tier documentation across the three authoritative model libraries.

**Completed Work:**

1. **basic-app-datatypes/** - Type system library
   - ✅ Created `CLAUDE.md` (601 lines) - Design rationale, complete type catalog, integration patterns
   - ✅ Refactored `llms.txt` (255→150 lines) - Tier 1 quick reference format
   - ✅ Commits: b784ce3, 15d9fab
   - ✅ Pushed to: https://github.com/sealablab/basic-app-datatypes

2. **moku-models/** - Platform specifications library
   - ✅ Refactored `llms.txt` (146→172 lines) - Tier 1 quick reference format
   - ✅ Enhanced `CLAUDE.md` (+60 lines) - Cross-library integration section
   - ✅ Commits: c24713b, a4267b0
   - ✅ Pushed to: https://github.com/sealablab/moku-models

3. **riscure-models/** - Probe hardware specs library
   - ✅ Refactored `llms.txt` (186→192 lines) - Tier 1 quick reference format
   - ✅ Enhanced `CLAUDE.md` (+65 lines) - Cross-library integration section
   - ✅ Commits: fc475ca, 600375b
   - ✅ Pushed to: https://github.com/sealablab/riscure-models

4. **forge/libs/MODELS_INDEX.md**
   - ✅ Created meta-document (324 lines) explaining the foundational trio
   - ✅ Integration patterns, AI agent decision tree, tiered loading strategy
   - ✅ Commit: d212c2f
   - ✅ Pushed to: https://github.com/sealablab/moku-instrument-forge

**Two-Tier Pattern Established:**
- **Tier 1:** llms.txt (~120-150 lines) - Quick reference, always load first
- **Tier 2:** CLAUDE.md (~250-600 lines) - Deep context, load when designing/integrating

**Cross-Library Integration:**
- basic-app-datatypes ↔ moku-models (voltage type validation)
- moku-models ↔ riscure-models (voltage safety before wiring)
- basic-app-datatypes ↔ riscure-models (type compatibility with probe specs)

### ✅ Phase 2: Created Monorepo-Level llms.txt

**Goal:** Meta-index that delegates to submodule llms.txt files.

**Completed Work:**
- ✅ Created `llms.txt` at monorepo root (213 lines)
- ✅ Repository structure overview (submodule hierarchy)
- ✅ Tiered context loading strategy (Tier 1→2→3)
- ✅ AI agent decision tree (what to load when)
- ✅ Common workflows (type lookup, generation, deployment)
- ✅ Git submodule workflow documentation
- ✅ Commit: 009600f
- ✅ Pushed to: chore/claude-setup branch

---

## Current Repository State

### All Commits Pushed Successfully

**Nested Submodules (in forge/libs/):**
```
basic-app-datatypes @ 15d9fab (main)
  ├── CLAUDE.md (new)
  └── llms.txt (refactored)

moku-models @ a4267b0 (main)
  ├── CLAUDE.md (enhanced)
  └── llms.txt (refactored)

riscure-models @ 600375b (main)
  ├── CLAUDE.md (enhanced)
  └── llms.txt (refactored)
```

**forge @ d212c2f (main)**
```
libs/MODELS_INDEX.md (new)
libs/basic-app-datatypes @ 15d9fab
libs/moku-models @ a4267b0
libs/riscure-models @ 600375b
```

**monorepo @ 009600f (chore/claude-setup branch)**
```
llms.txt (new)
forge @ d212c2f
```

### Git Status
- All nested submodules: ✅ On main branch, pushed to origin
- forge: ✅ On main branch, pushed to origin
- monorepo: ✅ On chore/claude-setup branch, pushed to origin

---

## Phase 3: Agent Structure (READY FOR IMPLEMENTATION)

### Proposed Minimal Structure

**Recommendation:** Thin orchestration layer that delegates to forge agents.

```
.claude/
├── agents/
│   └── probe-design-orchestrator/
│       └── agent.md                    # NEW - Monorepo-level coordinator (~300 lines)
│
├── commands/
│   └── README.md                       # Points to forge/.claude/commands/
│
└── shared/
    ├── CONTEXT_MANAGEMENT.md           # Tiered loading strategy
    ├── FOUNDATIONAL_MODELS.md          # Deep dive on the trio
    └── PROBE_WORKFLOW.md               # End-to-end probe development
```

### Design Rationale

**Why minimal structure?**
- forge/ already has 5 mature agents (~4,800 lines):
  - forge-context (YAML → VHDL generation)
  - deployment-context (package → hardware)
  - hardware-debug-context (FSM debugging)
  - docgen-context (docs, TUIs, Python APIs)
  - workflow-coordinator (multi-stage pipelines)

- Monorepo role = **coordinate**, not duplicate
- Avoids maintenance burden of duplicate agent logic
- Clear separation: monorepo orchestrates, forge executes

### Agent: probe-design-orchestrator

**Scope:** Monorepo-level probe workflow coordination

**Responsibilities:**
1. Guide users through probe development workflow
2. Coordinate between monorepo probe directories (`probes/DS1120_PD/`) and forge generated packages (`forge/apps/`)
3. Manage probe-specific VHDL implementations
4. Track probe development state
5. Delegate to forge agents for specialized tasks

**Delegates to:**
- forge-context: YAML validation and VHDL generation
- deployment-context: Hardware operations
- hardware-debug-context: FSM debugging
- docgen-context: Documentation generation
- workflow-coordinator: Multi-stage pipelines

**Key Workflow:**
```
User Request: "Implement new probe"
    ↓
probe-design-orchestrator (monorepo level)
    ↓
Delegates to: forge-context (YAML → VHDL)
    ↓
Coordinates: User writes custom VHDL in probes/*/vhdl/
    ↓
Delegates to: deployment-context (deploy to hardware)
    ↓
Delegates to: hardware-debug-context (FSM monitoring)
```

### Shared Knowledge Docs

**CONTEXT_MANAGEMENT.md** (~150 lines)
- Detailed explanation of Tier 1→2→3 loading
- When to load llms.txt vs CLAUDE.md vs source code
- Token budget optimization strategies
- Examples for common questions

**FOUNDATIONAL_MODELS.md** (~200 lines)
- Deep dive on basic-app-datatypes, moku-models, riscure-models
- Complete integration patterns
- Cross-validation examples
- Development workflows for each library

**PROBE_WORKFLOW.md** (~200 lines)
- Step-by-step probe development guide
- YAML spec → VHDL generation → custom implementation → deployment
- Best practices for probe drivers
- Common pitfalls and solutions

### Rejected Alternative

**Full duplication:** Create deployment-context, hardware-debug-context at monorepo level
- ❌ Would duplicate ~1,500 lines of agent prompts
- ❌ Two sources of truth for same functionality
- ❌ Maintenance burden when forge agents evolve
- ❌ forge agents already scoped correctly

---

## Next Steps for New Session

### Immediate Tasks

1. **Review Phase 3 proposal** with user
   - Confirm minimal structure approach
   - Get approval on agent count (just probe-design-orchestrator?)
   - Confirm shared docs are valuable

2. **Implement Phase 3** (if approved)
   - Create `.claude/agents/probe-design-orchestrator/agent.md`
   - Create `.claude/shared/CONTEXT_MANAGEMENT.md`
   - Create `.claude/shared/FOUNDATIONAL_MODELS.md`
   - Create `.claude/shared/PROBE_WORKFLOW.md`
   - Create `.claude/commands/README.md` (delegation to forge)

3. **Commit and push**
   - All Phase 3 files to chore/claude-setup branch
   - Prepare for PR to main

### Questions to Address

1. **Agent count:** Just probe-design-orchestrator, or monorepo-level wrappers for deployment/debug?
2. **Shared docs:** All three (CONTEXT_MANAGEMENT, FOUNDATIONAL_MODELS, PROBE_WORKFLOW), or fewer?
3. **Commands directory:** Actual command files, or just delegation README?

### Future Enhancements (Post-Phase 3)

- Add `.claude/skills/` if needed
- Consider MCP server integration
- Create example probe workflow documentation
- Add testing/validation workflows

---

## Key Insights from This Session

### Documentation Architecture

**Two-Tier Pattern Works Well:**
- Tier 1 (llms.txt): Fast lookup, minimal tokens (~150 lines)
- Tier 2 (CLAUDE.md): Deep dive, loaded as needed (~250-600 lines)
- Tier 3 (source code): Only when actually coding

**Meta-Index Strategy:**
- Monorepo llms.txt delegates to submodule llms.txt files
- AI agents load monorepo llms.txt first, drill down as needed
- Avoids token bloat from duplicating submodule docs

### Foundational Libraries are Special

**The trio (basic-app-datatypes, moku-models, riscure-models):**
- Authoritative source of truth
- Never guess types, platforms, or probe specs
- Always read actual definitions
- Cross-library integration is critical (voltage safety!)

### Agent Delegation Principle

**Don't duplicate, coordinate:**
- forge/ has mature, well-scoped agents
- Monorepo role is orchestration, not re-implementation
- Clear handoff points between monorepo and forge contexts
- Avoids maintenance burden and version drift

---

## Files Modified/Created This Session

### Created
- `forge/libs/basic-app-datatypes/CLAUDE.md` (601 lines)
- `forge/libs/MODELS_INDEX.md` (324 lines)
- `llms.txt` (213 lines, monorepo root)
- `SESSION_HANDOFF.md` (this file)

### Modified
- `forge/libs/basic-app-datatypes/llms.txt` (255→150 lines)
- `forge/libs/moku-models/llms.txt` (146→172 lines)
- `forge/libs/moku-models/CLAUDE.md` (+60 lines)
- `forge/libs/riscure-models/llms.txt` (186→192 lines)
- `forge/libs/riscure-models/CLAUDE.md` (+65 lines)

### Git Commits
- basic-app-datatypes: 2 commits (b784ce3, 15d9fab)
- moku-models: 2 commits (c24713b, a4267b0)
- riscure-models: 2 commits (fc475ca, 600375b)
- forge: 1 commit (d212c2f)
- monorepo: 1 commit (009600f)

All commits pushed to origin ✅

---

## Context Window Management

**This Session:**
- Used: ~114k/200k tokens
- Remaining: ~86k tokens
- Strategy: Created comprehensive documentation, committed frequently

**For Next Session:**
- Start fresh with full context window
- Load this handoff document first
- Review Phase 3 proposal before implementing
- Budget ~400-500 lines per agent/shared doc

---

## References

**Existing forge Agents:**
- [forge/.claude/agents/forge-context/agent.md](forge/.claude/agents/forge-context/agent.md) - 754 lines
- [forge/.claude/agents/deployment-context/agent.md](forge/.claude/agents/deployment-context/agent.md) - 557 lines
- [forge/.claude/agents/hardware-debug-context/agent.md](forge/.claude/agents/hardware-debug-context/agent.md) - 647 lines
- [forge/.claude/agents/docgen-context/agent.md](forge/.claude/agents/docgen-context/agent.md) - 821 lines
- [forge/.claude/agents/workflow-coordinator/agent.md](forge/.claude/agents/workflow-coordinator/agent.md) - 459 lines

**Foundational Libraries:**
- [forge/libs/basic-app-datatypes/llms.txt](forge/libs/basic-app-datatypes/llms.txt)
- [forge/libs/basic-app-datatypes/CLAUDE.md](forge/libs/basic-app-datatypes/CLAUDE.md)
- [forge/libs/moku-models/llms.txt](forge/libs/moku-models/llms.txt)
- [forge/libs/moku-models/CLAUDE.md](forge/libs/moku-models/CLAUDE.md)
- [forge/libs/riscure-models/llms.txt](forge/libs/riscure-models/llms.txt)
- [forge/libs/riscure-models/CLAUDE.md](forge/libs/riscure-models/CLAUDE.md)
- [forge/libs/MODELS_INDEX.md](forge/libs/MODELS_INDEX.md)

**Monorepo:**
- [llms.txt](llms.txt) - Entry point for AI agents

---

**Ready for handoff to new session!** 🚀
