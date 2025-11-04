# Phase 4: Agent Architecture Refactoring

**Date:** 2025-11-03
**Status:** Planning - Ready for execution
**Prerequisite:** ✅ Phase 3 complete (.claude/ directory setup implemented)
**Context:** Reorganize agents by domain boundaries
**Note:** This is a work order document - delete after Phase 4 is complete

---

## Problem Statement

### Current Architecture (Problematic)

**forge/.claude/agents/** (Everything lives in forge)
```
forge/.claude/agents/
├── forge-context/          754 lines - ✅ YAML→VHDL (forge's domain)
├── deployment-context/     557 lines - ⚠️  Hardware ops (NOT forge's domain!)
├── hardware-debug-context/ 647 lines - ⚠️  Hardware debug (NOT forge's domain!)
├── docgen-context/         821 lines - ✅ Docs (package output, forge's domain)
└── workflow-coordinator/   459 lines - ⚠️  Vague name! Coordinates what?
```

**monorepo/.claude/agents/** (Phase 3 created minimal structure)
```
.claude/agents/
└── probe-design-orchestrator/   556 lines - ✅ CREATED IN PHASE 3
    └── agent.md                  (Delegates to forge agents, coordinates probes)

Still missing (should be at monorepo level):
- deployment-orchestrator/ (currently in forge as deployment-context)
- hardware-debug/ (currently in forge as hardware-debug-context)
```

### Why This Is Wrong

1. **deployment-context in forge/** - Deployment is monorepo-wide concern
   - Operates on ANY package (forge/apps/*, probes/*, future packages)
   - User needs ONE deployment agent for all probes, not per-submodule
   - Tight coupling to moku-models, NOT to forge internals

2. **hardware-debug-context in forge/** - Debugging is hardware-level concern
   - Debug ANY deployed probe, regardless of where package came from
   - Depends on deployment, not forge generation

3. **workflow-coordinator** - Name is too vague
   - What does it coordinate? (Answer: forge pipelines)
   - Sounds like it should be at monorepo level
   - Actually coordinates forge-specific workflows (/validate → /generate → /optimize)

4. **Domain boundary confusion**
   - forge/ should be: **Package generation** (YAML → VHDL + metadata)
   - monorepo/ should be: **Hardware operations** (deploy, debug) + **probe workflows**

---

## Target Architecture (Domain-Based)

### Monorepo Level: `.claude/agents/` (Hardware Operations)

**1. deployment-orchestrator/** (moved from forge, renamed)
- **Domain:** Package → Deployed hardware
- **Input:** Well-formed package (manifest.json, control_registers.json, bitstream)
- **Output:** Configured Moku device
- **Uses:** moku-models heavily (platform specs, routing, validation)
- **Why monorepo:** Deploy ANY package from anywhere in monorepo
- **Lines:** ~600 (same content as current deployment-context)
- **Commands:** `/deploy`, `/discover`

**2. hardware-debug/** (moved from forge)
- **Domain:** Deployed hardware → Debug insights
- **Input:** Deployed Moku device (from deployment-orchestrator)
- **Output:** FSM traces, state analysis, timing reports
- **Why monorepo:** Debug ANY deployed probe
- **Lines:** ~650 (same content as current hardware-debug-context)
- **Commands:** `/debug-fsm`, `/monitor-state`, `/trace-signals`, `/analyze-timing`

**3. probe-design-orchestrator/** (✅ created in Phase 3)
- **Domain:** Probe workflow coordination
- **Input:** User requests for probe development
- **Delegates to:** forge-context, deployment-context, hardware-debug-context, workflow-coordinator
- **Why monorepo:** Bridges probes/* and forge/apps/*, coordinates complete probe lifecycle
- **Lines:** 556 (implemented)
- **Commands:** Via delegation to specialists
- **Status:** ✅ Already implemented in Phase 3, currently delegates to forge agents as-is

---

### Forge Level: `forge/.claude/agents/` (Package Operations)

**1. forge-context/** (keep as-is)
- **Domain:** YAML → VHDL package generation
- **This is forge's core competency**
- **Lines:** ~750
- **Commands:** `/generate`, `/validate`, `/map-registers`, `/optimize`, `/test-forge`

**2. docgen-context/** (keep as-is)
- **Domain:** Package → Documentation (markdown, TUIs, Python APIs)
- **Why forge:** Docs are about packages (forge's output)
- **Lines:** ~820
- **Commands:** `/gen-docs`, `/gen-ui`, `/gen-python-api`

**3. forge-pipe-fitter/** (renamed from workflow-coordinator)
- **Domain:** Multi-stage forge pipelines
- **Coordinates:** /validate → /generate → /optimize → /gen-docs workflows
- **Why rename:** "workflow-coordinator" too vague; "pipe-fitter" conveys forge pipeline assembly
- **Lines:** ~460 (same content, updated delegation logic)
- **Commands:** `/workflow new-probe`, `/workflow iterate`, `/workflow optimize`, `/workflow document`

---

## Domain Separation Principle

### Forge Domain: Package Generation
```
YAML Spec
    ↓
[forge-context] → Generate package
    ↓
[docgen-context] → Generate docs
    ↓
[forge-pipe-fitter] → Coordinate above
    ↓
Well-formed package (apps/*/)
```

### Monorepo Domain: Hardware Operations
```
Well-formed package
    ↓
[deployment-orchestrator] → Deploy to Moku
    ↓
[hardware-debug] → Debug FSM/signals
    ↓
Running probe on hardware
```

### Monorepo Domain: Probe Workflow
```
User: "Implement new probe"
    ↓
[probe-design-orchestrator]
    ↓ Delegates to forge-pipe-fitter
    ↓ User writes custom VHDL in probes/*/vhdl/
    ↓ Delegates to deployment-orchestrator
    ↓ Delegates to hardware-debug
    ↓
Complete probe development lifecycle
```

---

## Migration Plan

### Step 1: Create Monorepo Agents Directory
```bash
mkdir -p .claude/agents/deployment-orchestrator
mkdir -p .claude/agents/hardware-debug
mkdir -p .claude/agents/probe-design-orchestrator
```

### Step 2: Migrate deployment-context → deployment-orchestrator

**Source:** `forge/.claude/agents/deployment-context/agent.md`
**Destination:** `.claude/agents/deployment-orchestrator/agent.md`

**Changes needed:**
1. Rename header: "Deployment Context Agent" → "Deployment Orchestrator"
2. Update scope: "Read packages" → "Read packages from monorepo (forge/apps/*, probes/*)"
3. Update commands section: Reference monorepo-level commands
4. Add section: "Works with ANY package in monorepo"
5. Strengthen moku-models integration section
6. Update file paths in examples (may reference probes/* not just forge/apps/*)

**What stays the same:**
- Core deployment logic (discover, connect, deploy, configure routing)
- Platform specifications
- Routing patterns
- Error handling
- moku API usage

### Step 3: Migrate hardware-debug-context → hardware-debug

**Source:** `forge/.claude/agents/hardware-debug-context/agent.md`
**Destination:** `.claude/agents/hardware-debug/agent.md`

**Changes needed:**
1. Rename header: "Hardware Debug Context Agent" → "Hardware Debug Agent"
2. Update scope: "Debug forge packages" → "Debug ANY deployed probe"
3. Update dependency: References "deployment-context" → "deployment-orchestrator"
4. Update file paths in examples (may reference probes/* not just forge/apps/*)

**What stays the same:**
- FSM debugging logic
- State monitoring
- Signal tracing
- Timing analysis
- Oscilloscope integration

### Step 4: Rename workflow-coordinator → forge-pipe-fitter

**Source:** `forge/.claude/agents/workflow-coordinator/agent.md`
**Destination:** `forge/.claude/agents/forge-pipe-fitter/agent.md`

**Changes needed:**
1. Rename header: "Workflow Coordinator Agent" → "Forge Pipe-Fitter Agent"
2. Update description: "Coordinates forge-specific pipelines (validate, generate, optimize, document)"
3. Update delegation logic:
   - deployment-context → deployment-orchestrator (at monorepo level)
   - hardware-debug-context → hardware-debug (at monorepo level)
4. Make clear: "This agent coordinates FORGE operations, not hardware operations"
5. Update workflow templates to delegate hardware ops to monorepo agents

**What stays the same:**
- Workflow templates (/workflow new-probe, /workflow iterate, etc.)
- State tracking for forge operations
- Error handling for generation/validation

### Step 5: Update probe-design-orchestrator (already exists)

**File:** `.claude/agents/probe-design-orchestrator/agent.md` (✅ created in Phase 3)

**Changes needed:** (~400-500 lines already implemented, needs reference updates)
**Current implementation (Phase 3):**
- ✅ 556 lines, fully implemented
- ✅ Delegates to forge agents (as they currently exist in forge/)
- ✅ Has rich delegation examples with all current agent names
- ✅ Includes monorepo-specific workflows

**Updates needed after Phase 4 migration:**
- Update delegation references: deployment-context → deployment-orchestrator
- Update delegation references: hardware-debug-context → hardware-debug
- Update delegation references: workflow-coordinator → forge-pipe-fitter
- Update file path references (agents moved from forge/ to monorepo/)

**No content changes needed** - just update agent names/paths in delegation examples

### Step 6: Update forge-context References

**File:** `forge/.claude/agents/forge-context/agent.md`

**Changes needed:**
1. Remove references to deployment-context (it moved)
2. Add note: "For deployment, use monorepo-level deployment-orchestrator"
3. Update "Integration with Other Contexts" section:
   - deployment-context → deployment-orchestrator (monorepo-level)
   - hardware-debug-context → hardware-debug (monorepo-level)

### Step 7: Update docgen-context References

**File:** `forge/.claude/agents/docgen-context/agent.md`

**Changes needed:**
1. Update "Integration with Other Contexts" section:
   - deployment-context → deployment-orchestrator (monorepo-level)

### Step 8: Update Command Files

**forge/.claude/commands/** (update delegation references)

Check these files for references to old agent names:
- `deployment.md` - Update to reference deployment-orchestrator
- `debug.md` - Update to reference hardware-debug
- `workflow.md` - Update to reference forge-pipe-fitter

### Step 9: Delete Old Agent Directories (after migration verified)

**In forge/.claude/agents/:**
```bash
# After verifying migration successful
rm -rf forge/.claude/agents/deployment-context
rm -rf forge/.claude/agents/hardware-debug-context
rm -rf forge/.claude/agents/workflow-coordinator  # renamed to forge-pipe-fitter
```

---

## Verification Checklist

Before considering migration complete:

### Agent Files
- [ ] `.claude/agents/deployment-orchestrator/agent.md` exists (~600 lines)
- [ ] `.claude/agents/hardware-debug/agent.md` exists (~650 lines)
- [ ] `.claude/agents/probe-design-orchestrator/agent.md` exists (~400-500 lines)
- [ ] `forge/.claude/agents/forge-pipe-fitter/agent.md` exists (~460 lines)
- [ ] `forge/.claude/agents/forge-context/agent.md` updated (references fixed)
- [ ] `forge/.claude/agents/docgen-context/agent.md` updated (references fixed)

### Cross-References
- [ ] All agents reference correct peer agents (new names, new locations)
- [ ] forge agents reference monorepo agents correctly (../../../.claude/agents/*)
- [ ] monorepo agents reference forge agents correctly (forge/.claude/agents/*)

### Commands
- [ ] forge/.claude/commands/*.md updated with new agent references
- [ ] No broken delegation references

### Old Files Removed
- [ ] `forge/.claude/agents/deployment-context/` deleted
- [ ] `forge/.claude/agents/hardware-debug-context/` deleted
- [ ] `forge/.claude/agents/workflow-coordinator/` deleted

---

## Benefits of This Reorganization

### 1. Clear Domain Boundaries
- **forge/:** "I generate packages"
- **monorepo/:** "I operate hardware and coordinate probes"

### 2. Reusability
- deployment-orchestrator can deploy ANY package, not just forge-generated
- hardware-debug can debug ANY probe, not tied to forge
- forge agents don't need to know about hardware

### 3. Scalability
- Adding new package sources (beyond forge) is easy
- Adding new hardware operations doesn't pollute forge
- Multi-probe workflows are natural at monorepo level

### 4. Maintenance
- forge changes don't affect deployment logic
- Hardware API changes isolated to deployment-orchestrator
- Each agent has single responsibility

### 5. Mental Model
- "forge makes packages, monorepo deploys them" - simple!
- No confusion about where to look for deployment logic
- Agent names clearly convey purpose (forge-pipe-fitter vs vague workflow-coordinator)

---

## Future Enhancement: Phase 5 (Post-Refactor)

### moku-models Library Factoring

**Goal:** Move deployment logic from agent prompt to Python library

**Create in moku-models:**
```
moku-models/
├── moku_models/
│   ├── deployment/          # NEW
│   │   ├── __init__.py
│   │   ├── deployer.py      # Typed deployment operations
│   │   ├── discovery.py     # Device discovery with caching
│   │   └── validator.py     # Pre-deployment validation
│   └── ...
```

**deployer.py example:**
```python
from moku_models import MokuConfig, PlatformSpec
from pathlib import Path
import json

class Deployer:
    """Type-safe Moku deployment operations."""

    def deploy(
        self,
        device_ip: str,
        config: MokuConfig,
        bitstream_path: Path
    ) -> DeploymentResult:
        """Deploy package to Moku device."""
        # Implementation using moku API
        # All type-checked with Pydantic models
        ...

    def discover(
        self,
        platform: PlatformSpec | None = None
    ) -> list[MokuDevice]:
        """Discover Moku devices on network."""
        ...

    def validate_deployment(
        self,
        package_dir: Path,
        target_platform: PlatformSpec
    ) -> list[ValidationError]:
        """Validate package before deployment."""
        ...
```

**Then deployment-orchestrator/agent.md becomes:**
```markdown
# Deployment Orchestrator

Uses `moku_models.deployment` library for all operations.

## Example Usage

```python
from moku_models.deployment import Deployer
from moku_models import MokuConfig

deployer = Deployer()
devices = deployer.discover(platform='moku_go')
config = MokuConfig.from_manifest('apps/DS1140_PD/manifest.json')
result = deployer.deploy(
    device_ip='192.168.1.100',
    config=config,
    bitstream_path='apps/DS1140_PD/DS1140_PD.tar.gz'
)
```

## Why This Approach
- Agent is thin wrapper (100-200 lines vs 600 lines)
- Library is reusable outside this monorepo
- Type safety via Pydantic
- Easier testing (test library, not agent prompts)
```

**Benefits:**
- Deployment logic becomes reusable Python library
- Agent prompts stay concise (focus on workflow, not implementation)
- Testing is easier (unit test library code)
- moku-models becomes "complete deployment solution" (types + logic)

**When to do this:**
- After Phase 4 refactoring complete
- When deployment logic stabilizes
- If other projects need deployment capabilities

---

## File Manifest (After Phase 4)

### Monorepo Level
```
.claude/
├── agents/
│   ├── deployment-orchestrator/
│   │   └── agent.md                (~600 lines, moved from forge)
│   ├── hardware-debug/
│   │   └── agent.md                (~650 lines, moved from forge)
│   └── probe-design-orchestrator/
│       └── agent.md                (~400-500 lines, new)
│
├── commands/
│   ├── sync-submodules.md          (✅ 80 lines, created in Phase 3)
│   ├── init-probe.md               (✅ 180 lines, created in Phase 3)
│   ├── probe-status.md             (✅ 150 lines, created in Phase 3)
│   ├── validate-probe-structure.md (✅ 220 lines, created in Phase 3)
│   └── cross-validate.md           (✅ 420 lines, created in Phase 3)
│
└── shared/
    ├── CONTEXT_MANAGEMENT.md       (✅ 430 lines, created in Phase 3)
    └── PROBE_WORKFLOW.md           (✅ 580 lines, created in Phase 3)
```

### Forge Level
```
forge/.claude/
├── agents/
│   ├── forge-context/
│   │   └── agent.md                (~750 lines, updated references)
│   ├── docgen-context/
│   │   └── agent.md                (~820 lines, updated references)
│   └── forge-pipe-fitter/          (renamed from workflow-coordinator)
│       └── agent.md                (~460 lines, updated delegation)
│
├── commands/
│   ├── forge.md                    (existing)
│   ├── deployment.md               (updated references)
│   ├── debug.md                    (updated references)
│   ├── docgen.md                   (existing)
│   ├── platform.md                 (existing)
│   └── workflow.md                 (updated references)
│
└── shared/
    ├── package_contract.md         (existing)
    └── type_system_quick_ref.md    (existing)
```

---

## Migration Strategy Options

### ~~Option C: Phase 3 First, Then Refactor~~ ✅ COMPLETED
**Status:** We chose this option, Phase 3 is now complete!

**What was done:**
1. ✅ Implemented Phase 3 (minimal monorepo .claude/)
2. ⏭️ Use it for a bit, validate approach (NEXT)
3. ⏭️ Then execute Phase 4 refactoring with lessons learned (FUTURE)

**Current state:**
- Phase 3 agents working with current forge structure
- probe-design-orchestrator delegates to agents as-is (deployment-context, etc.)
- Ready to use and validate before Phase 4

---

### Recommended Approach for Phase 4 Execution

### Option A: Big Bang Migration (one session)
**Approach:** Implement all refactoring in one go
**Pros:** Clean cut, no intermediate state
**Cons:** High risk, long session, potential errors
**Best for:** If very confident in plan after using Phase 3

### Option B: Incremental Migration (multi-session)
**Approach:** Migrate one agent at a time, test each
**Pros:** Lower risk, easier to verify
**Cons:** Temporary inconsistency in naming/locations
**Best for:** If cautious, want to test each step

**Recommendation:** Validate Phase 3 first through actual usage, then choose A or B based on experience

---

## Next Steps (Current State)

### ✅ Phase 3 Complete (Done!)
- ✅ Created probe-design-orchestrator (556 lines, delegates to forge agents)
- ✅ Created CONTEXT_MANAGEMENT.md and PROBE_WORKFLOW.md
- ✅ Created 5 monorepo-specific commands
- ✅ Committed to chore/claude-setup branch (commits: ec3fe00, 9fcf3f3)

### ⏭️ Before Phase 4 (Recommended)
1. **Use Phase 3 structure** for actual probe development
   - Work on a probe using new .claude/ setup
   - Identify friction points with current agent delegation
   - Validate that the approach works
   - Gather learnings before refactoring

2. **Decide if Phase 4 is needed**
   - Is current delegation working well?
   - Are domain boundaries causing confusion?
   - Is the value worth the refactoring effort?

### ⏭️ When Ready for Phase 4
1. **Review this document** - Check migration plan is still accurate
2. **Start fresh session** - Full context window recommended
3. **Execute migration** - Follow 9-step plan above
4. **Verify thoroughly** - Use checklist before considering complete

### ⏭️ Phase 5 (Future Enhancement)
- Factor deployment library into moku-models
- Simplify agent prompts
- Make deployment logic reusable across projects

---

## Open Questions to Resolve in Phase 4

1. **Should docgen-context stay in forge or move to monorepo?**
   - Argument for forge: Generates docs about packages (forge's output)
   - Argument for monorepo: Docs are for users, not forge-internal
   - Current decision: Keep in forge (docs ARE about packages)

2. **Should forge-pipe-fitter be renamed to something else?**
   - Current decision: forge-pipe-fitter
   - Alternatives: forge-orchestrator, forge-workflows, package-pipeline
   - "pipe-fitter" conveys: assembling forge pipeline stages

3. **How should monorepo agents reference forge agents?**
   - Relative path: `../../../forge/.claude/agents/forge-context/agent.md`
   - Documented convention: "Delegate to forge-context agent"
   - Current decision: Documented delegation pattern (not file references)

4. **Should we create wrapper commands at monorepo level?**
   - Example: `/generate` at monorepo that calls forge's `/generate`
   - Pro: Convenience, works from monorepo root
   - Con: Duplication, maintenance burden
   - Current decision: No wrappers, use forge commands directly (via delegation)

---

## Success Criteria (Phase 4 Complete)

- [ ] All agents organized by domain (hardware @ monorepo, packages @ forge)
- [ ] No broken cross-references between agents
- [ ] forge-pipe-fitter coordinates forge pipelines only
- [ ] deployment-orchestrator handles ANY package deployment
- [ ] hardware-debug handles ANY probe debugging
- [ ] probe-design-orchestrator coordinates complete probe lifecycle
- [ ] Documentation updated (if any user-facing docs exist)
- [ ] All commands work with new agent structure
- [ ] Verification checklist 100% complete

---

## References

**Related Documents:**
- `SESSION_HANDOFF.md` - Phase 1 & 2 complete, Phase 3 design
- `forge/libs/MODELS_INDEX.md` - Foundational model libraries
- `llms.txt` - Monorepo meta-index
- `forge/.claude/agents/*/agent.md` - Current agent structure (to be refactored)

**Key Decisions:**
- Domain separation: Hardware ops @ monorepo, package ops @ forge
- Agent renaming: workflow-coordinator → forge-pipe-fitter (clarity)
- Agent migration: deployment + debug → monorepo (domain fit)
- Future enhancement: moku-models library factoring (Phase 5)

---

**Ready for Phase 4 execution when Phase 3 is complete and validated!** 🚀
