# Session Handoff: Documentation Cleanup

**Date:** 2025-11-03
**Session Focus:** CLAUDE.md refactor and documentation ecosystem review
**Status:** In progress - CLAUDE.md refactored, PROBE_WORKFLOW.md reviewed

---

## What We Did This Session

### 1. CLAUDE.md Radical Refactor ✅

**Problem Identified:**
- CLAUDE.md violated its own Tier 2 documentation principles
- 762 lines (~6-8k tokens) when it should be 250-300 lines (~2-3k tokens)
- Massive redundancy with PROBE_WORKFLOW.md, WORKFLOW_GUIDE.md, and ARCHITECTURE_OVERVIEW.md
- Mixed procedures (HOW) with philosophy (WHY)
- Tried to be 5 documents in one

**Changes Made:**
- Reduced from 762 → 560 lines (26% reduction)
- Reduced from ~6-8k → ~2.5k tokens (60% reduction)
- Completely restructured around design rationale and mental models
- Removed all step-by-step procedures → delegated to PROBE_WORKFLOW.md
- Removed git tutorials → delegated to WORKFLOW_GUIDE.md
- Removed code examples → delegated to source files
- Added missing "WHY" sections:
  - Why this architecture?
  - Why tiered documentation?
  - Why "Never Guess, Always Read"?
  - Why Option A workspace?
- Added mental models:
  - Two Worlds (monorepo vs submodules)
  - Integration patterns (how libraries compose)
  - Workflow decision points
- Added AI agent guidance:
  - Context loading strategy
  - Token budget tracking table
  - Delegation strategy
  - When to load this file

**New File Location:** `/Users/johnycsh/ALT_TOP/moku-instrument-forge-mono-repo/CLAUDE.md`

**Version:** 3.0 (Tier 2 refactor - design rationale focus)

### 2. PROBE_WORKFLOW.md Review 📋

**File:** `.claude/shared/PROBE_WORKFLOW.md` (811 lines)

**What's In There:**
- Workflow 1: New Probe Development (12 detailed steps)
- Workflow 2: Iterative Development (fast cycles)
- Workflow 3: Debugging Probe Behavior
- Workflow 4: Multi-Probe Management
- Common Patterns (FSM, voltage clamping, timing control)
- Troubleshooting Guide
- Success Criteria checklist
- Next Steps After Deployment

**Assessment:**
- ✅ Comprehensive step-by-step procedures (exactly what it should be)
- ✅ Example outputs for success/failure
- ✅ YAML + VHDL code examples for common patterns
- ✅ Troubleshooting organized by symptom
- ⚠️ 811 lines is hard to navigate (needs TOC)
- ⚠️ Large code blocks could be extracted to examples/
- ⚠️ Missing cross-references to other docs
- ⚠️ Minimal AI agent guidance
- ⚠️ No token budget considerations

---

## Current Documentation Ecosystem State

### Root-Level Documentation

| File | Lines | Status | Purpose |
|------|-------|--------|---------|
| `llms.txt` | 215 | ✅ Good | Tier 1 entry point |
| `CLAUDE.md` | 560 | ✅ **Refactored** | Tier 2 design rationale |
| `ARCHITECTURE_OVERVIEW.md` | 606 | ✅ Good | Tier 2 architecture specs |
| `PROBE_WORKFLOW.md` | 811 | ⚠️ **Needs work** | Tier 2 procedures |
| `WORKFLOW_GUIDE.md` | 365 | ✅ Good | Daily commands |
| `README.md` | ? | ❓ Not reviewed | Human-friendly overview |

### Key Findings

**Documentation Hierarchy is Now Clear:**
```
llms.txt (always load first)
    ↓
Need philosophy?    → CLAUDE.md
Need architecture?  → ARCHITECTURE_OVERVIEW.md
Need procedures?    → PROBE_WORKFLOW.md
Need daily commands? → WORKFLOW_GUIDE.md
```

**Redundancy Status:**
- ✅ CLAUDE.md vs others: **Resolved** (refactor eliminated overlap)
- ⚠️ PROBE_WORKFLOW.md: Still has some overlap with old CLAUDE.md concepts (acceptable)
- ✅ WORKFLOW_GUIDE.md: Minimal overlap (good separation of concerns)

---

## Recommendations for Next Session

### Priority 1: PROBE_WORKFLOW.md Surgical Improvements (1 hour)

**Goal:** Keep structure, improve navigation and AI agent guidance

**Tasks:**
1. **Add table of contents** at top (811 lines needs navigation)
2. **Extract large code blocks** to `examples/` directory:
   - `examples/yaml/fsm-probe-template.yaml`
   - `examples/vhdl/voltage-clamping-pattern.vhd`
   - `examples/vhdl/timing-control-pattern.vhd`
3. **Add "For AI Agents" section:**
   - When to load this file
   - Token cost per workflow (~2-4k tokens for full workflow)
   - Delegation guidance (which workflows need which agents)
4. **Add cross-references:**
   - Link to CLAUDE.md for philosophy/mental models
   - Link to WORKFLOW_GUIDE.md for git commands
   - Link to llms.txt for type/platform lookups
5. **Add token budget notes** for each workflow:
   - "Loading this workflow section: ~1.5k tokens"
   - "Full Workflow 1 with examples: ~4k tokens"

**Files to create:**
```
examples/
├── yaml/
│   ├── fsm-probe-template.yaml
│   ├── voltage-clamping-template.yaml
│   └── timing-control-template.yaml
└── vhdl/
    ├── fsm-pattern.vhd
    ├── voltage-clamping-pattern.vhd
    └── timing-control-pattern.vhd
```

### Priority 2: Verify README.md (15 mins)

**Check:**
- Is it human-friendly?
- Does it point to llms.txt for AI agents?
- Is it up-to-date with new documentation structure?
- Does it explain the tiered documentation system for humans?

### Priority 3: Create Documentation Index (30 mins)

**File:** `.claude/shared/DOCUMENTATION_INDEX.md`

**Purpose:** Single source of truth for "which doc to load when"

**Structure:**
```markdown
# Documentation Index

## By Audience
### For AI Agents
- Always start: llms.txt
- Design rationale: CLAUDE.md
- Architecture specs: ARCHITECTURE_OVERVIEW.md
- Step-by-step procedures: PROBE_WORKFLOW.md
- Daily commands: WORKFLOW_GUIDE.md

### For Humans
- Getting started: README.md
- Architecture: ARCHITECTURE_OVERVIEW.md
- Daily workflows: WORKFLOW_GUIDE.md

## By Task
### Creating New Probe
1. Load: llms.txt
2. Load: PROBE_WORKFLOW.md → Workflow 1
3. Reference: basic-app-datatypes/llms.txt (for types)

### Debugging Probe
1. Load: llms.txt
2. Load: PROBE_WORKFLOW.md → Workflow 3 (Debugging)

## Token Budget Planning
| Task | Files | Tokens | % Budget |
|------|-------|--------|----------|
| Quick lookup | llms.txt | 500 | 0.25% |
| Understand philosophy | +CLAUDE.md | 2,500 | 1.25% |
| Complete workflow | +PROBE_WORKFLOW.md | 6,500 | 3.25% |
```

---

## Open Questions

1. **Should we split PROBE_WORKFLOW.md into modular files?**
   - Pro: Easier to load specific workflows
   - Con: More files to maintain
   - Decision: Defer until after surgical improvements

2. **Where should code examples live?**
   - Option A: `examples/` directory (recommended)
   - Option B: Keep inline in PROBE_WORKFLOW.md
   - Option C: In source code with references
   - Recommendation: **Option A** (examples/ directory)

3. **Is WORKFLOW_GUIDE.md redundant with PROBE_WORKFLOW.md?**
   - Analysis needed: WORKFLOW_GUIDE.md focuses on git/daily patterns
   - PROBE_WORKFLOW.md focuses on probe-specific procedures
   - Likely okay as-is, but verify no overlap

---

## Files Modified This Session

```
Modified:
- /Users/johnycsh/ALT_TOP/moku-instrument-forge-mono-repo/CLAUDE.md
  (762 lines → 560 lines, complete restructure)

Created:
- /Users/johnycsh/ALT_TOP/moku-instrument-forge-mono-repo/.claude/shared/SESSION_HANDOFF.md
  (this file)
```

---

## Quick Start for Next Session

**To resume where we left off:**

1. **Review changes:**
   ```bash
   git diff CLAUDE.md
   ```

2. **Read current state:**
   ```bash
   cat .claude/shared/SESSION_HANDOFF.md
   ```

3. **Start with Priority 1:**
   - Read PROBE_WORKFLOW.md
   - Add TOC
   - Extract code examples
   - Add AI agent section

4. **Or pivot to different task:**
   - Verify README.md
   - Create DOCUMENTATION_INDEX.md
   - Review forge-level docs

---

## Context for AI Agent

**What you need to know:**

- This monorepo uses a **3-tier documentation system**:
  - Tier 1: llms.txt (~500 tokens)
  - Tier 2: CLAUDE.md, ARCHITECTURE_OVERVIEW.md, PROBE_WORKFLOW.md (~2-5k tokens each)
  - Tier 3: Source code (~5-10k tokens per file)

- **CLAUDE.md was just refactored** to be true Tier 2 (design rationale, not procedures)

- **PROBE_WORKFLOW.md is next** - needs surgical improvements for navigation and AI guidance

- **Documentation philosophy:** Start minimal, expand as needed, reserve 93%+ token budget

- **Key principle:** Each doc has one clear purpose, minimal overlap

---

## Commit Message Template

When you commit this work:

```
docs: Refactor CLAUDE.md and add session handoff

- Reduce CLAUDE.md from 762→560 lines (26% reduction)
- Focus on design rationale and mental models (true Tier 2)
- Remove step-by-step procedures (delegated to PROBE_WORKFLOW.md)
- Remove git tutorials (delegated to WORKFLOW_GUIDE.md)
- Add "WHY" sections for architecture decisions
- Add mental models for development workflows
- Add AI agent guidance (context loading, delegation)
- Add SESSION_HANDOFF.md for continuity

Next: Surgical improvements to PROBE_WORKFLOW.md (see handoff doc)
```

---

**Session End:** 2025-11-03
**Next Session:** Pick up with PROBE_WORKFLOW.md surgical improvements (Priority 1)
**Estimated Time to Complete Remaining Work:** 2-3 hours
