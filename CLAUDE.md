# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

---

## Purpose & Scope

This is a **Tier 2 document** - load this when you need:
- Development workflow guidance
- Understanding how to navigate the documentation ecosystem
- Cross-repository development patterns
- How to extend or modify the system

**Not in this file:**
- Quick command references (see `llms.txt`)
- Architectural overview (see `.claude/shared/ARCHITECTURE_OVERVIEW.md`)
- Step-by-step procedures (see `.claude/shared/PROBE_WORKFLOW.md`)
- Token optimization strategy (see `.claude/shared/CONTEXT_MANAGEMENT.md`)

---

## Documentation Navigation Strategy

### The Three-Tier System

This monorepo uses a **tiered documentation system** optimized for AI agents:

**Tier 1: llms.txt files** (~500 tokens each)
- **Always load first**
- Quick facts, command reference, navigation pointers
- Location: Root, forge/, forge/libs/*, libs/forge-vhdl/

**Tier 2: CLAUDE.md files** (~2-5k tokens each)
- **Load for design/integration work**
- Design rationale, workflows, integration patterns
- Location: Root (this file), forge/libs/*/

**Tier 3: Source code** (~5-10k tokens per file)
- **Load for implementation/debugging**
- Actual Python/VHDL code, templates, tests

### AI Agent Loading Pattern

```
1. Start every task by loading: llms.txt (Tier 1)
   → "What is this repo? Where should I look?"

2. If quick question → Answer from llms.txt
   → Done. ~1k tokens used.

3. If design/integration question → Load CLAUDE.md (Tier 2)
   → "How do I work with this system?"
   → ~3-5k tokens used.

4. If implementation question → Load source code (Tier 3)
   → "Show me the actual code"
   → ~10-15k tokens used.

Result: Start minimal (0.5%), expand as needed (2-7%), keep 93%+ budget available
```

### Key Documentation Files

**Start here:**
- `llms.txt` - Entry point (Tier 1)

**Architectural understanding:**
- `.claude/shared/ARCHITECTURE_OVERVIEW.md` - Complete hierarchical structure
- `.claude/shared/CONTEXT_MANAGEMENT.md` - Token optimization strategy

**Operational guides:**
- `.claude/shared/PROBE_WORKFLOW.md` - Step-by-step probe development
- `WORKFLOW_GUIDE.md` - Daily development patterns
- `README.md` - Human-friendly project overview

**Cross-library integration:**
- `forge/libs/MODELS_INDEX.md` - How the foundational trio compose

**Foundational libraries (authoritative sources):**
- `forge/libs/basic-app-datatypes/llms.txt` → Type system (23 types)
- `forge/libs/moku-models/llms.txt` → Platform specs (Go/Lab/Pro/Delta)
- `forge/libs/riscure-models/llms.txt` → Probe hardware specs

---

## Development Workflows

### Workflow 1: Daily Development Setup

**Starting your day:**

```bash
# 1. Update to latest
git checkout main
git pull origin main

# 2. Ensure submodules are current
git submodule update --init --recursive

# 3. Verify Python environment
uv sync
python scripts/setup_forge_path.py

# 4. Check probe status
/probe-status  # See what probes exist and their state
```

**Decision point:**
- Working on existing probe? → Go to forge/apps/<probe_name>/
- Creating new probe? → Use `/init-probe <probe_name>`
- Modifying foundational library? → See "Cross-Repository Development" below

### Workflow 2: Creating a New Probe

**This is the primary use case for this monorepo.**

**Where you work:** `forge/apps/<probe_name>/` (Option A architecture)

**Steps:**

1. **Initialize structure**
   ```bash
   /init-probe DS1180_LASER
   # Creates: forge/apps/DS1180_LASER/
   ```

2. **Edit YAML specification**
   ```bash
   # Edit: forge/apps/DS1180_LASER/DS1180_LASER.yaml

   # Need to know what types exist?
   # → Read: forge/libs/basic-app-datatypes/llms.txt (Tier 1)

   # Need to understand type system design?
   # → Read: forge/libs/basic-app-datatypes/CLAUDE.md (Tier 2)

   # Need platform clock frequency?
   # → Read: forge/libs/moku-models/llms.txt (Tier 1)
   ```

3. **Validate and generate**
   ```bash
   /validate forge/apps/DS1180_LASER/DS1180_LASER.yaml
   /generate forge/apps/DS1180_LASER/DS1180_LASER.yaml

   # This creates:
   # - *_shim.vhd (auto-generated, DO NOT EDIT)
   # - *_main.vhd (template for your implementation)
   # - manifest.json (register mappings)
   # - control_registers.json (default values)
   ```

4. **Implement custom VHDL**
   ```bash
   # Edit: forge/apps/DS1180_LASER/*_main.vhd

   # Key principle: Use friendly signal names from manifest.json
   # NOT raw Control Register references!

   # Good: if arm_probe = '1' then ...
   # Bad:  if Control6(31) = '1' then ...
   ```

5. **Cross-validate**
   ```bash
   /cross-validate DS1180_LASER

   # Checks:
   # - Signal names in VHDL match manifest.json
   # - VHDL types compatible with manifest types
   # - No direct CR references
   ```

6. **Deploy and test**
   ```bash
   /deploy DS1180_LASER --device 192.168.1.100
   /monitor-state DS1180_LASER
   ```

**Everything stays in one place:** `forge/apps/DS1180_LASER/`

### Workflow 3: Iterating on Existing Probe

**Fast iteration cycle:**

```bash
# 1. Make YAML changes
# Edit: forge/apps/DS1180_LASER/DS1180_LASER.yaml

# 2. Regenerate and redeploy in one step
/workflow iterate forge/apps/DS1180_LASER/DS1180_LASER.yaml --deploy

# 3. Verify
/monitor-state DS1180_LASER
```

**Custom VHDL changes:**

```bash
# 1. Edit implementation
# Edit: forge/apps/DS1180_LASER/*_main.vhd

# 2. Cross-validate
/cross-validate DS1180_LASER

# 3. Recompile bitstream (external step, not covered by monorepo)

# 4. Redeploy
/deploy DS1180_LASER --device 192.168.1.100 --force
```

### Workflow 4: Multi-Probe Management

**Check status of all probes:**

```bash
/probe-status

# Output shows:
# - Which probes exist in forge/apps/
# - YAML validity
# - Package generation status
# - Deployment status (if tracked)
```

**Working on multiple probes:**

Each probe is independent:
- `forge/apps/DS1140_PD/` - Complete probe A
- `forge/apps/DS1180_LASER/` - Complete probe B
- `forge/apps/DS1199_PD/` - Complete probe C

You can work on all simultaneously without conflicts.

---

## Cross-Repository Development

### Understanding the Git Structure

You're working in a **4-level nested git repository**:

```
moku-instrument-forge-mono-repo/              # Level 0 (you are here)
├── .git/
└── forge/                                     # Level 1 (submodule)
    ├── .git/
    └── libs/                                  # Level 2 (nested submodules)
        ├── basic-app-datatypes/.git/
        ├── moku-models/.git/
        └── riscure-models/.git/
```

### When to Work in Submodules

**Work in monorepo (forge/apps/):**
- Creating new probes ✅
- Editing probe YAML specs ✅
- Implementing probe VHDL ✅
- Testing probes ✅

**Work in forge submodule (forge/):**
- Modifying code generator ⚠️
- Changing VHDL templates ⚠️
- Adding new slash commands ⚠️

**Work in foundational libraries (forge/libs/):**
- Adding new types to basic-app-datatypes ⚠️⚠️
- Adding new platforms to moku-models ⚠️⚠️
- Adding new probes to riscure-models ⚠️⚠️

### Modifying a Foundational Library

**Critical pattern - must follow exactly:**

```bash
# Example: Adding a new type to basic-app-datatypes

# 1. Navigate INTO the submodule
cd forge/libs/basic-app-datatypes

# 2. Create branch IN THE SUBMODULE
git checkout -b feat/add-10v-voltage-type

# 3. Make changes IN THE SUBMODULE
# Edit: basic_app_datatypes/types.py
# Add: voltage_output_10v_s16 to BasicAppDataTypes enum
# Edit: basic_app_datatypes/metadata.py
# Add: TYPE_REGISTRY entry
# Edit: llms.txt
# Add: New type to catalog

# 4. Commit IN THE SUBMODULE
git add .
git commit -m "feat: Add voltage_output_10v_s16 type"

# 5. Push THE SUBMODULE (to its own repo!)
git push origin feat/add-10v-voltage-type

# 6. Create PR in the submodule's repository
# https://github.com/sealablab/basic-app-datatypes/pull/new

# 7. After PR is merged, return to monorepo root
cd ../../..

# 8. Update submodule reference in parent
git add forge/libs/basic-app-datatypes
git commit -m "chore: Update basic-app-datatypes to include 10V type"
git push origin main
```

**Key rules:**
- ✅ Always commit in submodule FIRST
- ✅ Then update parent reference
- ✅ Each submodule has its own repo, issues, PRs
- ❌ Never force-push to submodules
- ❌ Never commit directly to submodule main (use branches + PRs)

### Syncing Submodules

**After pulling updates:**

```bash
git pull origin main
git submodule update --init --recursive

# This ensures all submodules are at the correct commit
```

**If submodules are out of sync:**

```bash
git submodule status

# Output shows:
# +abc1234 forge (heads/main)  ← Ahead of expected commit
#  def5678 libs/basic-app-datatypes ← At expected commit

# To fix:
cd forge
git checkout <expected-commit>
cd ..
git add forge
git commit -m "fix: Reset forge submodule to correct commit"
```

---

## Working with Foundational Libraries (The Authoritative Trio)

### The Three Sources of Truth

**basic-app-datatypes** - Type System Authority
- **What it knows:** 23 voltage/time/boolean types
- **Quick lookup:** `forge/libs/basic-app-datatypes/llms.txt`
- **Deep dive:** `forge/libs/basic-app-datatypes/CLAUDE.md`
- **When to use:** Validating YAML datatypes, understanding register mapping

**moku-models** - Platform Specs Authority
- **What it knows:** 4 Moku platforms (Go/Lab/Pro/Delta)
- **Quick lookup:** `forge/libs/moku-models/llms.txt`
- **Deep dive:** `forge/libs/moku-models/CLAUDE.md`
- **When to use:** Validating platform choice, understanding voltage ranges, deployment

**riscure-models** - Probe Hardware Authority
- **What it knows:** Probe specifications (DS1120A, etc.)
- **Quick lookup:** `forge/libs/riscure-models/llms.txt`
- **Deep dive:** `forge/libs/riscure-models/CLAUDE.md`
- **When to use:** Validating probe wiring, voltage safety checks

### Cross-Library Integration Patterns

**Read:** `forge/libs/MODELS_INDEX.md` for complete integration guide.

**Common pattern: Type ← Platform validation**

```python
# When validating a YAML spec, check:
# 1. Type exists in basic-app-datatypes
# 2. Platform exists in moku-models
# 3. Type voltage range compatible with platform output

from basic_app_datatypes import BasicAppDataTypes, TYPE_REGISTRY
from moku_models import MOKU_GO_PLATFORM

# Type: voltage_output_05v_s16 (±5V)
voltage_type = BasicAppDataTypes.VOLTAGE_OUTPUT_05V_S16
metadata = TYPE_REGISTRY[voltage_type]

# Platform: Moku:Go OUT1 (10Vpp = ±5V)
platform = MOKU_GO_PLATFORM
output = platform.get_analog_output_by_id('OUT1')

# Validate compatibility
assert metadata.voltage_range == "±5V"
assert output.voltage_range_vpp == 10.0
# ✓ Compatible
```

### Never Guess - Always Read

**❌ Bad (AI guessing):**
```
"I think there's a voltage_10v_s16 type..."
"Moku:Go probably runs at 100MHz..."
"The probe should accept 5V inputs..."
```

**✅ Good (AI reading authority):**
```
AI loads: basic-app-datatypes/llms.txt
→ "23 types exist. voltage_10v_s16 is NOT listed."

AI loads: moku-models/llms.txt
→ "Moku:Go = 125 MHz (authoritative)"

AI loads: riscure-models/llms.txt
→ "DS1120A digital_glitch = 0-3.3V TTL only"
```

---

## Testing Strategy

### Test Organization

Tests are distributed across the hierarchy:

```
monorepo/tests/                    # Shared test infrastructure
forge/forge/tests/                 # Forge generator tests
forge/libs/basic-app-datatypes/tests/   # Type system tests
forge/libs/moku-models/tests/           # Platform spec tests
forge/libs/riscure-models/tests/        # Probe spec tests
libs/forge-vhdl/tests/             # VHDL utility tests
forge/apps/*/tests/                # Probe-specific tests (future)
```

### Running Tests

**Monorepo-level tests:**
```bash
pytest                  # All tests
pytest -n auto          # Parallel execution
pytest libs/            # Library tests only
```

**Forge tests:**
```bash
cd forge
uv run pytest forge/tests/ -v
```

**Foundational library tests:**
```bash
cd forge/libs/basic-app-datatypes
pytest tests/ -v
```

**Testing after changes:**

If you modified:
- YAML spec → Run `/validate` and `/generate`, then test deployment
- Custom VHDL → Run `/cross-validate`, recompile bitstream, test on hardware
- Forge generator → Run `pytest forge/tests/`
- Foundational library → Run tests in that library's repo

---

## Extending the System

### Adding a New Slash Command

**Location:** `.claude/commands/<command-name>.md`

**Template:**
```markdown
# Command Name

Brief description.

**Agent:** [which agent handles this]

---

## Usage

\`\`\`
/command-name <args>
\`\`\`

## What It Does

[Detailed description]

## Examples

[Usage examples]

## Notes

[Important considerations]
```

**After creating:** Test by invoking `/command-name` in Claude Code

### Adding a New Agent

**Location:** `.claude/agents/<agent-name>/agent.md`

**Key sections:**
- Role and responsibilities
- Domain boundaries
- Delegation strategy
- Commands it handles
- Context loading strategy

**Reference existing agents:**
- `.claude/agents/probe-design-orchestrator/agent.md` (monorepo-level)
- `forge/.claude/agents/forge-context/agent.md` (forge-level)

### Adding a New Probe Template

**Location:** `.claude/commands/init-probe.md`

Edit the template YAML and README that get generated when `/init-probe` runs.

---

## Common Development Scenarios

### Scenario 1: "I need a type that doesn't exist"

**Problem:** YAML validation fails because voltage_output_10v_s16 doesn't exist.

**Solution:**
1. Load `forge/libs/basic-app-datatypes/llms.txt` (Tier 1)
   → Confirm type doesn't exist in the 23 defined types
2. Load `forge/libs/basic-app-datatypes/CLAUDE.md` (Tier 2)
   → Find "Adding New Types" section
3. Follow cross-repository development workflow (see above)
4. Submit PR to basic-app-datatypes repository
5. After merge, update monorepo submodule reference

**Alternative (temporary):**
Use existing type with closest range (e.g., voltage_output_05v_s16) and clamp in VHDL.

### Scenario 2: "My probe won't deploy"

**Common causes:**

1. **Package not generated**
   ```bash
   /generate forge/apps/<probe_name>/<probe_name>.yaml
   ```

2. **Platform mismatch**
   ```bash
   # Check YAML platform vs device
   # YAML: platform: moku_go
   # Device: Actually moku_lab
   # Fix: Update YAML or deploy to correct device
   ```

3. **Bitstream missing**
   ```bash
   # Bitstream compilation is external step
   # Ensure <probe_name>.tar.gz exists
   ```

4. **Network issues**
   ```bash
   /discover  # Find available devices
   # Verify device IP and reachability
   ```

### Scenario 3: "Generated VHDL has wrong bit slices"

**Debugging approach:**

1. Load `llms.txt` (Tier 1) - Understand register mapping
2. Load `forge/.claude/agents/forge-context/agent.md` (Tier 2)
3. Load `forge/generator/codegen.py` (Tier 3) - Trace bit slice calculation
4. Load generated `*_shim.vhd` (Tier 3) - Compare with manifest.json
5. Identify bug in register mapper or template

**Fix location:** Depends on where bug is:
- Register mapping logic → `forge/libs/basic-app-datatypes/mapper.py`
- Template → `forge/templates/shim.vhd.j2`
- Generator → `forge/generator/codegen.py`

### Scenario 4: "FSM is stuck in wrong state"

**Debugging approach:**

1. Use hardware debugging:
   ```bash
   /debug-fsm <probe_name>
   /monitor-state <probe_name>
   ```

2. Check control registers:
   - Are default values correct? (check control_registers.json)
   - Did user change values via Python API?

3. Review VHDL FSM logic:
   - Load `forge/apps/<probe_name>/*_main.vhd`
   - Check transition conditions
   - Verify reset logic

4. Check signal wiring:
   - Load manifest.json - Verify register mappings
   - Cross-reference with VHDL signal usage

---

## Git Workflow Best Practices

### Branch Naming

- New probe: `feature/<probe-name>` (e.g., `feature/ds1180-laser`)
- Bug fix: `fix/<description>` (e.g., `fix/voltage-clamp-off-by-one`)
- Enhancement: `feat/<description>` (e.g., `feat/add-timing-controls`)
- Chore: `chore/<description>` (e.g., `chore/update-submodules`)

### Commit Messages

```
<type>: <short summary>

<optional detailed description>

<optional footer>
```

Types: `feat`, `fix`, `docs`, `chore`, `refactor`, `test`

**Examples:**

```
feat: Add DS1180 laser probe with FSM control

- Define 8 datatypes for laser control
- Implement FSM with READY/ARMED/FIRING/COOLING states
- Target platform: moku_go
- Safety clamp: 3.0V max output
```

```
fix: Correct voltage threshold calculation in DS1180

The threshold was off by one due to signed/unsigned conversion.
Changed comparison from >= to > in FSM state machine.

Fixes: voltage trigger not firing at exact threshold
```

```
chore: Update basic-app-datatypes submodule

Update to include new voltage_output_10v_s16 type added in v1.1.0
```

### Before Committing

**Checklist:**
- [ ] Run `/probe-status` to verify probe validity
- [ ] Run `pytest` if you modified Python code
- [ ] Run `/cross-validate <probe>` if you modified VHDL
- [ ] Check `git status` to see what's staged
- [ ] Write clear commit message explaining WHY, not just WHAT

---

## AI Agent Guidance

### When to Load This File

**Load CLAUDE.md when:**
- User asks "How do I develop in this repo?"
- User is creating/modifying probes
- User asks about git submodule workflows
- User needs to understand documentation navigation
- User is extending the system

**Don't load when:**
- Quick command lookup (use llms.txt instead)
- Architectural overview (use ARCHITECTURE_OVERVIEW.md instead)
- Step-by-step procedures (use PROBE_WORKFLOW.md instead)

### Delegation Strategy

You are likely in the **probe-design-orchestrator** agent. Delegate to:

- **forge-context** - YAML validation, package generation
- **deployment-orchestrator** - Hardware deployment
- **hardware-debug** - FSM debugging
- **docgen-context** - Documentation generation

See agent prompts in `.claude/agents/` for delegation patterns.

### Context Loading Decision Tree

```
User request arrives
    ↓
Load: llms.txt (Tier 1, always)
    ↓
Is this a quick command? → Answer from llms.txt
    ↓
Is this about development workflow? → Load CLAUDE.md (Tier 2)
    ↓
Is this about architecture? → Load ARCHITECTURE_OVERVIEW.md (Tier 2)
    ↓
Is this about types/platforms/probes? → Load foundational library llms.txt (Tier 1)
    ↓
Need design details? → Load foundational library CLAUDE.md (Tier 2)
    ↓
Need implementation details? → Load source code (Tier 3)
```

---

## Summary: Development Workflow Quick Reference

**Creating new probe:**
```bash
/init-probe <name> → Edit YAML → /validate → /generate → Edit VHDL → /cross-validate → /deploy
```

**Iterating on probe:**
```bash
Edit YAML → /workflow iterate --deploy → /monitor-state
```

**Modifying foundational library:**
```bash
cd forge/libs/<lib>/ → git checkout -b feat/... → edit → commit → push → PR → merge → cd ../../.. → git add forge/libs/<lib> → commit
```

**Everything in one place:**
```
forge/apps/<probe_name>/  # YAML + VHDL + manifest + docs
```

**Never guess types/platforms:**
```
Load: forge/libs/*/llms.txt (authoritative sources)
```

**Documentation hierarchy:**
```
llms.txt (Tier 1) → CLAUDE.md (Tier 2) → source code (Tier 3)
```

---

**Last Updated:** 2025-11-03
**Version:** 2.0 (Tier 2 rewrite)
**See also:** `.claude/shared/ARCHITECTURE_OVERVIEW.md` for complete architectural understanding
