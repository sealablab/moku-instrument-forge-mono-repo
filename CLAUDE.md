# CLAUDE.md

This file provides design rationale and mental models for working with this repository.

---

## Purpose & Scope

This is a **Tier 2 document** - load this when you need to understand:
- **WHY** the architecture is designed this way
- **HOW** to navigate the documentation ecosystem
- **WHEN** to use which workflow
- **WHERE** to find authoritative information

**Not in this file:**
- Quick command references → see `llms.txt`
- Architectural specifications → see `.claude/shared/ARCHITECTURE_OVERVIEW.md`
- Step-by-step procedures → see `.claude/shared/PROBE_WORKFLOW.md`
- Daily command workflows → see `WORKFLOW_GUIDE.md`

---

## Core Design Philosophy

### Why This Architecture?

This monorepo solves a specific problem: **composing independently-maintained foundational libraries into a coherent probe development workflow**.

**The key insight:** Each foundational library is an **authoritative truth bubble**:
- **basic-app-datatypes** knows the type system (23 types, register mapping)
- **moku-models** knows platform specs (clock frequencies, voltage ranges)
- **riscure-models** knows probe hardware (voltage limits, port definitions)

These libraries:
- Don't import each other (zero coupling)
- Can be used standalone (outside this monorepo)
- Are versioned independently (git submodules)
- Compose at integration time (in the forge generator)

**Why git submodules?** Because foundational libraries evolve independently. A type system change shouldn't require coordinating with probe hardware specs. Submodules enforce this boundary.

**Why 4 levels deep?** Not by design - by necessity:
```
monorepo (orchestration layer)
  └── forge (generation layer)
      └── libs/* (foundational layer)
```

Each level has a distinct purpose. The nesting is compositional, not accidental.

### Why the Tiered Documentation System?

**Problem:** AI agents with 200k token budgets waste context on irrelevant details.

**Solution:** Three-tier progressive disclosure:

**Tier 1 (llms.txt)** - Quick facts (~500 tokens)
- "What types exist?" → Answer in 30 seconds
- Load cost: 0.25% of budget

**Tier 2 (CLAUDE.md)** - Design rationale (~2-3k tokens)
- "Why this type system?" → Understand philosophy
- Load cost: 1.5% of budget

**Tier 3 (source code)** - Implementation (~5-10k tokens per file)
- "How is register mapping implemented?" → Read the code
- Load cost: 5% of budget per file

**The win:** Start minimal, expand as needed. Reserve 93%+ budget for actual work.

### Why "Never Guess, Always Read"?

AI agents hallucinate plausible answers:
- "I think there's a voltage_10v_s16 type..." ❌
- "Moku:Go probably runs at 100MHz..." ❌

**But foundational libraries are authoritative sources of truth.**

The correct pattern:
1. Load `basic-app-datatypes/llms.txt` → "23 types exist. voltage_10v_s16 is NOT one of them."
2. Load `moku-models/llms.txt` → "Moku:Go = 125 MHz (authoritative)"

**Mental model:** Foundational libraries are **databases**, not **guesses**.

### Why Option A Workspace (forge/apps/)?

We had two architectural choices:

**Option A:** Everything in one directory
```
forge/apps/DS1180_LASER/
├── DS1180_LASER.yaml          # Source
├── *_shim.vhd                 # Generated
├── *_main.vhd                 # Implementation
├── manifest.json              # Package contract
└── control_registers.json     # Defaults
```

**Option B:** Separate YAML from generated files
```
probes/DS1180_LASER.yaml       # In monorepo
forge/apps/DS1180_LASER/       # Generated files
```

**We chose Option A** because:
- Simple mental model: "one probe = one directory"
- Easy to navigate: everything colocated
- Works with forge as-is (no modifications needed)
- Natural atomic commits (YAML + generated files together)

**Tradeoff accepted:** Generated files committed to git (normally anti-pattern). We accept this because manifest.json and control_registers.json are part of the package contract, not ephemeral build artifacts.

---

## Navigation Strategy

### The Documentation Ecosystem

This monorepo has **5 key documentation files** at the root:

| File | Purpose | When to Load |
|------|---------|--------------|
| `llms.txt` | Entry point, command reference | Always first |
| `CLAUDE.md` | Design rationale (this file) | Understanding WHY |
| `ARCHITECTURE_OVERVIEW.md` | Complete hierarchical structure | Understanding WHAT |
| `PROBE_WORKFLOW.md` | Step-by-step procedures | Doing work |
| `WORKFLOW_GUIDE.md` | Daily command patterns | Quick reference |

**The loading pattern:**
```
User request
    ↓
Load: llms.txt (always)
    ↓
Quick command? → Answer from llms.txt → Done
    ↓
Need philosophy? → Load CLAUDE.md
    ↓
Need architecture? → Load ARCHITECTURE_OVERVIEW.md
    ↓
Need procedure? → Load PROBE_WORKFLOW.md
    ↓
Need daily commands? → Load WORKFLOW_GUIDE.md
```

### The Foundational Library Documentation

Each library follows the same pattern:

**Quick lookup:** `forge/libs/<library>/llms.txt`
- Type catalog (basic-app-datatypes)
- Platform specs (moku-models)
- Probe specs (riscure-models)

**Deep dive:** `forge/libs/<library>/CLAUDE.md`
- Type system design (basic-app-datatypes)
- Platform integration patterns (moku-models)
- Safety validation patterns (riscure-models)

**Implementation:** `forge/libs/<library>/<package>/`
- Source code, Pydantic models, tests

**Cross-library integration:** `forge/libs/MODELS_INDEX.md`
- How the trio compose
- Validation patterns
- Example integrations

### Agent Documentation

**Monorepo-level agents** (`.claude/agents/`)
- probe-design-orchestrator (primary coordinator)
- deployment-orchestrator (hardware deployment)
- hardware-debug (FSM debugging)

**Forge-level agents** (`forge/.claude/agents/`)
- forge-context (YAML validation, generation)
- deployment-context (package deployment)
- docgen-context (documentation generation)
- hardware-debug-context (FSM expert)
- workflow-coordinator (multi-stage pipelines)

**Delegation principle:** Monorepo agents coordinate, forge agents execute.

---

## Development Mental Models

### Probe Development: The Core Workflow

**Conceptual flow:**
```
Specification → Validation → Generation → Implementation → Deployment
     ↓              ↓             ↓              ↓              ↓
   YAML      Type checking   VHDL shim      Custom VHDL    Hardware
```

**Key insight:** The YAML spec is the **single source of truth**. Everything else is derived.

**The shim layer** (auto-generated VHDL):
- Extracts signals from Control Registers
- Converts raw bits to typed signals (voltage_output_05v_s16, time_milliseconds_u16)
- Presents friendly interface to your implementation

**Your implementation** (*_main.vhd):
- Uses friendly signal names (threshold, intensity, arm_probe)
- NEVER references Control Registers directly
- Works with domain types, not raw bits

**Mental model:** YAML defines the contract, shim enforces it, you implement business logic.

### Cross-Repository Development: The Two Worlds

You work in **two distinct worlds**:

**World 1: Monorepo (forge/apps/)**
- Creating new probes ✅
- Editing probe specs (YAML) ✅
- Implementing probe logic (VHDL) ✅
- Testing and deployment ✅

**World 2: Submodules (forge/libs/)**
- Adding new types ⚠️
- Adding new platforms ⚠️
- Adding new probe models ⚠️

**Critical pattern for World 2:**
```bash
# 1. Navigate INTO submodule
cd forge/libs/basic-app-datatypes

# 2. Work IN THE SUBMODULE (separate git repo!)
git checkout -b feat/new-type
# ... make changes ...
git commit -m "feat: Add new type"
git push origin feat/new-type
# ... create PR in submodule repo, merge ...

# 3. Return to monorepo, update reference
cd ../../..
git add forge/libs/basic-app-datatypes
git commit -m "chore: Update basic-app-datatypes"
git push
```

**Mental model:** Submodules are **separate git repositories** that happen to be nested. Always commit in submodule FIRST, then update parent reference.

**Common mistake:** Editing submodule files and committing from monorepo root. This creates detached HEAD state in submodule.

### Integration Patterns: How Libraries Compose

**Pattern 1: Type ← Platform Validation**

Question: "Is this type compatible with this platform output?"

```
voltage_output_05v_s16 (from basic-app-datatypes)
    ↓ check voltage range: ±5V
    ↓
Moku:Go OUT1 (from moku-models)
    ↓ check output spec: 10Vpp (±5V)
    ↓
✓ Compatible
```

**Pattern 2: Platform ← Probe Wiring Safety**

Question: "Is this Moku → probe connection safe?"

```
Moku:Go OUT1 raw DAC output: ±5V
    ↓ compare with...
    ↓
DS1120A digital_glitch input: 0-3.3V TTL
    ↓
✗ UNSAFE! Use TTL mode, not raw DAC
```

**Pattern 3: Type ← Probe Compatibility**

Question: "Can this type control this probe port?"

```
boolean_1 (from basic-app-datatypes)
    ↓ 1-bit control signal
    ↓
DS1120A trigger (from riscure-models)
    ↓ expects TTL signal
    ↓
✓ Compatible
```

**Mental model:** Each library is authoritative for its domain. Integration patterns compose them at validation time.

**See:** `forge/libs/MODELS_INDEX.md` for detailed integration examples.

---

## Workflow Decision Points

### When to Use Which Workflow?

**Scenario 1: Creating new probe**
- **Goal:** YAML spec → deployed hardware
- **Workflow:** PROBE_WORKFLOW.md → "Workflow 1: New Probe Development"
- **Commands:** `/init-probe`, `/validate`, `/generate`, `/deploy`

**Scenario 2: Iterating on existing probe**
- **Goal:** Quick YAML tweaks, redeploy
- **Workflow:** PROBE_WORKFLOW.md → "Workflow 2: Iterative Development"
- **Commands:** `/workflow iterate --deploy`

**Scenario 3: Debugging deployed probe**
- **Goal:** FSM stuck, outputs wrong
- **Workflow:** PROBE_WORKFLOW.md → "Workflow 3: Debugging"
- **Commands:** `/debug-fsm`, `/monitor-state`, `/trace-signals`

**Scenario 4: Managing multiple probes**
- **Goal:** Status dashboard, prioritize work
- **Workflow:** PROBE_WORKFLOW.md → "Workflow 4: Multi-Probe Management"
- **Commands:** `/probe-status`

**Scenario 5: Adding new type/platform/probe model**
- **Goal:** Extend foundational library
- **Workflow:** WORKFLOW_GUIDE.md → "Working with Submodules"
- **Requires:** Access to submodule repository

**Mental model:** Simple iteration stays in monorepo. Foundational changes require submodule work.

---

## Common Development Scenarios

### Scenario: "I need a type that doesn't exist"

**Problem:** YAML validation fails because `voltage_output_10v_s16` doesn't exist.

**Decision tree:**

1. **Verify it doesn't exist**
   - Load `forge/libs/basic-app-datatypes/llms.txt`
   - Check the 23 defined types
   - Confirm it's truly missing

2. **Choose approach:**
   - **Temporary workaround:** Use closest type (e.g., `voltage_output_05v_s16`), clamp in VHDL
   - **Permanent fix:** Add to basic-app-datatypes library

3. **If permanent fix:**
   - Load `forge/libs/basic-app-datatypes/CLAUDE.md` → "Adding New Types" section
   - Follow cross-repository workflow (WORKFLOW_GUIDE.md → "Working with Submodules")
   - Submit PR to basic-app-datatypes repository
   - After merge, update monorepo submodule reference

**Mental model:** Don't hack around missing types. Either use temporary workaround with explicit clamping, or extend the authoritative library.

### Scenario: "My probe won't deploy"

**Diagnostic approach:**

1. **Check package generation**
   - Does `manifest.json` exist?
   - Does `control_registers.json` exist?
   - Run `/probe-status` to verify

2. **Check platform compatibility**
   - YAML spec says `platform: moku_go`
   - Device is actually Moku:Go?
   - Run `/discover` to verify devices

3. **Check bitstream**
   - Does `<probe_name>.tar.gz` exist?
   - Bitstream compiled for correct platform?
   - (External compilation step, not covered by monorepo)

4. **Check network**
   - Is device reachable?
   - Correct IP address?
   - Run `/discover` to find devices

**See:** PROBE_WORKFLOW.md → "Troubleshooting Guide" for detailed solutions.

### Scenario: "Generated VHDL has wrong bit slices"

**Diagnostic approach:**

1. **Understand the mapping**
   - Load `llms.txt` (Tier 1) → Register mapping overview
   - Load generated `manifest.json` → See actual bit slices
   - Compare with YAML spec

2. **If mapping is wrong:**
   - This is a forge generator bug
   - Load `forge/.claude/agents/forge-context/agent.md` (Tier 2)
   - Delegate to forge-context agent
   - May need to load `forge/generator/codegen.py` (Tier 3)

3. **If VHDL is wrong but manifest correct:**
   - Template bug
   - Load `forge/templates/shim.vhd.j2`
   - Check bit slice extraction logic

**Mental model:** manifest.json is the contract. If it's wrong, generator bug. If manifest is right but VHDL wrong, template bug.

### Scenario: "FSM stuck in wrong state"

**Diagnostic approach:**

1. **Observe current state**
   - Run `/monitor-state <probe_name>`
   - Run `/debug-fsm <probe_name>`
   - What state is it in? What state should it be in?

2. **Check control signals**
   - Are defaults correct? (check control_registers.json)
   - Did user change values? (check current CR values)
   - Are signals wired correctly? (check manifest.json)

3. **Review FSM logic**
   - Load `forge/apps/<probe_name>/*_main.vhd`
   - Check transition conditions
   - Verify reset logic

4. **Quick fix: Force reset**
   - Set `reset_fsm` signal (if implemented)
   - Redeploy with fresh defaults: `/deploy <probe_name> --force`

**See:** PROBE_WORKFLOW.md → "Troubleshooting Guide" for detailed FSM debugging.

---

## For AI Agents

### Context Loading Strategy

**Principle:** Start minimal, expand as needed.

**The pattern:**

```
User request arrives
    ↓
Load: llms.txt (Tier 1, always, ~500 tokens)
    ↓
Can answer? → Yes → Answer (99% budget remaining)
             → No → Continue
    ↓
What kind of question?
    ↓
Philosophy/Design → Load CLAUDE.md (Tier 2, +2k tokens)
Architecture/Specs → Load ARCHITECTURE_OVERVIEW.md (Tier 2, +3k tokens)
Procedures/Steps  → Load PROBE_WORKFLOW.md (Tier 2, +4k tokens)
Daily commands    → Load WORKFLOW_GUIDE.md (Tier 2, +2k tokens)
    ↓
Can answer? → Yes → Answer (95% budget remaining)
             → No → Load source code (Tier 3)
```

**Token budget tracking:**

| Stage | Files Loaded | Tokens Used | Budget Remaining |
|-------|--------------|-------------|------------------|
| Start | llms.txt | ~500 | 199,500 (99.75%) |
| Design | +CLAUDE.md | ~2,500 | 197,500 (98.75%) |
| Procedure | +PROBE_WORKFLOW.md | ~6,500 | 193,500 (96.75%) |
| Source | +3 code files | ~21,500 | 178,500 (89.25%) |

**Still have 89% budget available** for actual work after full context load.

### Delegation Strategy

**You are likely in:** probe-design-orchestrator (monorepo-level agent)

**Delegate to forge agents for:**
- YAML validation → forge-context
- Package generation → forge-context or workflow-coordinator
- Hardware deployment → deployment-context
- FSM debugging → hardware-debug-context
- Documentation → docgen-context

**Handle at monorepo level:**
- Multi-probe coordination
- Cross-validation (VHDL ↔ package)
- Workflow orchestration
- Git operations

**Mental model:** Monorepo orchestrates, forge executes. Don't duplicate forge logic.

**See:** `.claude/agents/probe-design-orchestrator/agent.md` for detailed delegation patterns.

### When to Load This File

**Load CLAUDE.md when:**
- User asks "How do I develop in this repo?" (need workflow philosophy)
- User asks "Why is it designed this way?" (need design rationale)
- User asks about submodule workflows (need mental models)
- User needs to understand documentation navigation (need tier system)
- You need to decide which agent to delegate to (need delegation strategy)

**Don't load when:**
- Quick command lookup → use llms.txt
- Architectural specifications → use ARCHITECTURE_OVERVIEW.md
- Step-by-step procedures → use PROBE_WORKFLOW.md
- Daily commands → use WORKFLOW_GUIDE.md

---

## Summary: Key Principles

**1. Composability without Coupling**
- Foundational libraries are independent, compose at integration time
- Git submodules enforce boundaries
- MODELS_INDEX.md documents integration patterns

**2. Tiered Documentation for Token Efficiency**
- Start with llms.txt (Tier 1, ~500 tokens)
- Expand to CLAUDE.md/ARCHITECTURE_OVERVIEW.md (Tier 2, ~2-5k tokens)
- Deep dive to source code (Tier 3, ~5-10k tokens per file)
- Reserve 93%+ budget for actual work

**3. Never Guess, Always Read**
- Foundational libraries are authoritative databases
- Load llms.txt to verify types/platforms/probes
- Don't hallucinate plausible answers

**4. YAML is Single Source of Truth**
- Everything else is derived (shim, manifest, control_registers)
- Regeneration is idempotent
- Custom VHDL (*_main.vhd) is the only hand-written code

**5. Two Worlds: Monorepo vs Submodules**
- Probe work stays in monorepo (forge/apps/)
- Foundational changes require submodule work (forge/libs/)
- Always commit in submodule FIRST, then update parent

**6. Delegation: Monorepo Orchestrates, Forge Executes**
- Monorepo agents coordinate workflows
- Forge agents handle specialized tasks
- Don't duplicate logic across layers

---

## Quick Reference

| Task | Documentation | Commands |
|------|---------------|----------|
| Create new probe | PROBE_WORKFLOW.md | `/init-probe`, `/validate`, `/generate` |
| Iterate on probe | PROBE_WORKFLOW.md | `/workflow iterate` |
| Deploy to hardware | PROBE_WORKFLOW.md | `/deploy`, `/discover` |
| Debug FSM | PROBE_WORKFLOW.md | `/debug-fsm`, `/monitor-state` |
| Daily commands | WORKFLOW_GUIDE.md | Git workflows, branch naming |
| Modify submodule | WORKFLOW_GUIDE.md | Submodule workflow |
| Type lookup | basic-app-datatypes/llms.txt | 23 types catalog |
| Platform specs | moku-models/llms.txt | 4 platforms catalog |
| Probe specs | riscure-models/llms.txt | Probe hardware catalog |

---

**Last Updated:** 2025-11-03
**Version:** 3.0 (Tier 2 refactor - design rationale focus)
**See also:** `.claude/shared/ARCHITECTURE_OVERVIEW.md` for complete architectural specifications
