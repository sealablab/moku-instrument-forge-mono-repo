# moku-instrument-forge-mono-repo

**Composable Monorepo Template for Embedded Instrument Development**

---

## What Is This?

This is a **reference architecture** and **composable template** for building embedded instrument development environments using git submodules, with an integrated 3-tier documentation system optimized for AI-assisted development.

**Key Innovation:** An LLM-navigable monorepo structure where every component is self-documenting and independently composable.

**Use this as:**
- A starting point for your own instrument development monorepo
- A reference for organizing complex multi-repo projects
- A template demonstrating AI-first documentation patterns

**Repository:** https://github.com/sealablab/moku-instrument-forge-mono-repo
**Version:** 2.0.0 (Template-Ready)
**License:** MIT

---

## Using This as a Template

### For AI Agents

This monorepo is designed for AI-assisted development. Navigation strategy:

**Start here:**
1. **Read this file first** (CLAUDE.md) - Architecture overview and template usage
2. **Load `llms.txt`** (213 lines) - Quick component catalog
3. **Explore submodules** - Each has its own `llms.txt` → `CLAUDE.md` hierarchy
4. **Load detailed docs** - Only when customizing specific components

**Progressive disclosure pattern:**
```
Root CLAUDE.md (YOU ARE HERE)
    ↓
Root llms.txt (component catalog)
    ↓
Submodule llms.txt (component quick ref)
    ↓
Submodule CLAUDE.md (component deep dive)
    ↓
Source code (implementation details)
```

**AI-Friendly Commands:**
- `/customize-monorepo` - Start guided customization
- `/sync-submodules` - Update all submodules
- See `.claude/commands/` for all available commands

### For Humans

**Quick Start (GitHub Template):**
1. Click **"Use this template"** button on GitHub
2. Clone with submodules: `git clone --recurse-submodules <your-repo-url>`
3. Run: `uv sync` to install dependencies
4. Start customization: Read `TEMPLATE.md` for step-by-step guide

**Quick Start (Manual Fork):**
1. Fork this repository
2. Clone: `git clone --recurse-submodules <your-fork-url>`
3. Update remote URLs in `.gitmodules` if you fork submodules
4. Follow `TEMPLATE.md` for customization steps

**What you get:**
- Clean flat architecture (tools/ + libs/)
- Self-contained, composable submodules
- 3-tier documentation system
- AI-optimized for context-efficient development

---

## Architecture Philosophy (v2.0)

### Core Principles

**1. Clean Separation**
```
tools/          # Code generation, build tools, development utilities
libs/           # Foundational libraries (reusable, composable)
.claude/        # AI agent configurations (monorepo orchestration)
docs/           # Monorepo-level documentation
```

**No nested submodules.** Flat is better than nested.

**2. Islands of Authority**

Each submodule is an **authoritative bubble** for its domain:
- **forge-codegen** → Type system, code generation
- **forge-vhdl** → VHDL components, testing infrastructure
- **moku-models** → Platform specifications
- **riscure-models** → Probe hardware specifications

**Rule:** Never duplicate authority. If it exists in a submodule, reference it—don't recreate it.

**3. Self-Contained Modules**

Every submodule should:
- Work standalone (independent repository)
- Have 3-tier documentation (llms.txt → CLAUDE.md → source)
- Have zero parent dependencies (no ../../ references in docs)
- Be independently versioned

**4. Progressive Documentation**

Three tiers for token efficiency:
- **Tier 1 (llms.txt):** 500-1000 tokens, quick facts
- **Tier 2 (CLAUDE.md):** 3-5k tokens, complete guide
- **Tier 3 (source/docs):** Load only when needed

**Goal:** Typical AI task uses <10k tokens (5% budget) vs 50k+ with flat docs.

---

## Workspace Mode Architecture

This monorepo uses **uv workspace mode**, which means:
- The root is a **pure workspace container** (no package built at root)
- Each submodule is an independent workspace member
- Shared dependencies are defined at root level
- **No dummy package code required**

### Why Workspace Mode?

**Conceptual Clarity:**
```
Traditional: Monorepo pretends to be a package (confusing)
Workspace:   Monorepo IS a workspace (honest, clear)
```

**Key Benefits:**
- ✅ No build errors from missing package code
- ✅ Unified dependency management across submodules
- ✅ Cross-submodule imports work seamlessly
- ✅ Each submodule keeps its own `[build-system]`
- ✅ Clean separation of concerns

### Workspace Configuration

The root `pyproject.toml` declares workspace members:

```toml
[tool.uv.workspace]
members = [
    "libs/forge-vhdl",
    "libs/moku-models",
    "libs/riscure-models",
    "tools/forge-codegen",
]

# NO [build-system] at root - this is pure workspace!
```

### Adding/Removing Workspace Members

**Add a new member:**
```toml
[tool.uv.workspace]
members = [
    "libs/forge-vhdl",
    "libs/your-new-module",  # Add here
    "tools/forge-codegen",
]
```

**Remove a member:**
```bash
# Remove the submodule
git rm libs/unwanted-module/

# Remove from workspace members list in pyproject.toml
# Then: uv sync
```

**After changes:** Run `uv sync` to update the workspace dependency graph.

---

## Current Architecture (v2.0.0)

### Tools

**[forge-codegen](tools/forge-codegen/)**
- **Purpose:** YAML → VHDL code generator with type-safe register serialization
- **Authority:** 23-type system, register packing algorithm
- **Quick Ref:** [tools/forge-codegen/llms.txt](tools/forge-codegen/llms.txt)
- **Deep Dive:** [tools/forge-codegen/CLAUDE.md](tools/forge-codegen/CLAUDE.md)
- **Repository:** https://github.com/sealablab/moku-instrument-forge-codegen
- **Status:** ✅ Production-ready, 69 tests passing

### Foundational Libraries

**[forge-vhdl](libs/forge-vhdl/)**
- **Purpose:** Reusable VHDL components with CocoTB testing
- **Authority:** Voltage utilities, clock dividers, FSM observer, testing standards
- **Quick Ref:** [libs/forge-vhdl/llms.txt](libs/forge-vhdl/llms.txt)
- **Deep Dive:** [libs/forge-vhdl/CLAUDE.md](libs/forge-vhdl/CLAUDE.md)
- **Repository:** https://github.com/sealablab/moku-instrument-forge-vhdl
- **Status:** ✅ Production-ready, progressive testing framework

**[moku-models](libs/moku-models/)**
- **Purpose:** Pydantic models for Moku platform specifications
- **Authority:** Platform specs (Go/Lab/Pro/Delta), deployment configs, routing
- **Quick Ref:** [libs/moku-models/llms.txt](libs/moku-models/llms.txt)
- **Deep Dive:** [libs/moku-models/CLAUDE.md](libs/moku-models/CLAUDE.md)
- **Repository:** https://github.com/sealablab/moku-models
- **Status:** ✅ Production-ready, all 4 platforms supported

**[riscure-models](libs/riscure-models/)**
- **Purpose:** Pydantic models for Riscure probe specifications
- **Authority:** Probe electrical specs, voltage safety validation
- **Quick Ref:** [libs/riscure-models/llms.txt](libs/riscure-models/llms.txt)
- **Deep Dive:** [libs/riscure-models/CLAUDE.md](libs/riscure-models/CLAUDE.md)
- **Repository:** https://github.com/sealablab/riscure-models
- **Status:** ✅ Production-ready, DS1120A fully specified

### AI Agent System

**Monorepo-Level Agents** (`.claude/agents/`)
- **deployment-orchestrator** - Hardware deployment workflows
- **hardware-debug** - FSM debugging and monitoring

**Remaining Commands** (`.claude/commands/`)
- **sync-submodules** - Update all submodules to latest

**Archived** (`.claude/archive/`)
- Probe-specific agents/commands from v1.0 (kept for reference)

---

## Composability Patterns

### Adding a New Submodule

**When to add:**
- You need a reusable component across projects
- The component is domain-authoritative
- It deserves independent versioning

**How to add:**

1. **Create standalone repository** with 3-tier docs:
   ```bash
   # In new repo
   touch llms.txt CLAUDE.md README.md
   # Write documentation following 3-tier pattern
   ```

2. **Add as git submodule:**
   ```bash
   # In monorepo
   git submodule add <repo-url> libs/new-module/
   git submodule update --init libs/new-module/
   ```

3. **Update monorepo catalog:**
   - Add entry to root `llms.txt`
   - Update this CLAUDE.md (Current Architecture section)
   - Add to `.claude/manifest.json` if using programmatic discovery

4. **Commit submodule reference:**
   ```bash
   git add .gitmodules libs/new-module/
   git commit -m "feat: Add new-module submodule"
   ```

**Example (adding hypothetical laser-models):**
```bash
git submodule add https://github.com/yourorg/laser-models.git libs/laser-models/
# Edit llms.txt, CLAUDE.md
git add .gitmodules libs/laser-models/ llms.txt CLAUDE.md
git commit -m "feat: Add laser-models for optical probe support"
```

### Removing a Submodule

**When to remove:**
- You don't need the component for your use case
- You're replacing it with a different implementation
- It's not relevant to your domain

**How to remove:**

1. **Remove from git:**
   ```bash
   git rm libs/unwanted-module/
   git commit -m "chore: Remove unwanted-module (not needed for our use case)"
   ```

2. **Clean up references:**
   - Remove from root `llms.txt`
   - Remove from this CLAUDE.md
   - Remove from `.claude/manifest.json` if present

3. **Update documentation** to explain why it was removed (optional but recommended)

**Example (removing riscure-models if you don't need probe specs):**
```bash
git rm libs/riscure-models/
# Edit llms.txt, CLAUDE.md
git commit -m "chore: Remove riscure-models (not using Riscure probes)"
```

### Forking a Submodule

**When to fork:**
- You need domain-specific customizations
- The upstream is moving in a different direction
- You want to experiment without affecting others

**How to fork:**

1. **Fork the submodule repository** on GitHub

2. **Update .gitmodules** to point to your fork:
   ```bash
   # Edit .gitmodules
   [submodule "libs/moku-models"]
       path = libs/moku-models
       url = https://github.com/YOUR-USERNAME/moku-models.git  # Your fork
   ```

3. **Sync the change:**
   ```bash
   git submodule sync
   cd libs/moku-models/
   git remote set-url origin https://github.com/YOUR-USERNAME/moku-models.git
   git fetch origin
   cd ../..
   git add .gitmodules
   git commit -m "chore: Switch to forked moku-models"
   ```

4. **Make your customizations** in the fork

5. **Consider contributing back** - If your changes are generally useful, submit a PR to upstream

### Replacing a Submodule

**When to replace:**
- You found a better implementation
- You want to use a different technology
- You're switching domains (e.g., Moku → different platform)

**How to replace:**

1. Remove old submodule (see "Removing a Submodule")
2. Add new submodule (see "Adding a New Submodule")
3. Update integration code that referenced the old module
4. Test thoroughly

**Example (replacing moku-models with hypothetical picoscope-models):**
```bash
git rm libs/moku-models/
git submodule add https://github.com/yourorg/picoscope-models.git libs/picoscope-models/
# Update code that imported from moku_models
git commit -m "feat: Replace moku-models with picoscope-models"
```

---

## Submodule Integration Patterns

### Pattern 1: Type System + Platform Validation

**Use case:** Validate that code generation types are compatible with platform specs

```python
# In your custom code
from forge_codegen.basic_serialized_datatypes import BasicAppDataTypes, TYPE_REGISTRY
from moku_models import MOKU_GO_PLATFORM

# Get type metadata
voltage_type = BasicAppDataTypes.VOLTAGE_OUTPUT_05V_S16
metadata = TYPE_REGISTRY[voltage_type]
# → voltage_range: "±5V"

# Get platform DAC specs
platform = MOKU_GO_PLATFORM
dac = platform.get_analog_output_by_id('OUT1')
# → voltage_range_vpp: 10.0 (±5V)

# Validate compatibility
assert metadata.voltage_range == "±5V"
assert dac.voltage_range_vpp == 10.0
print("✓ Type compatible with platform")
```

**Libraries involved:** forge-codegen + moku-models

### Pattern 2: Platform + Probe Safety Validation

**Use case:** Ensure safe wiring between platform and probe hardware

```python
from moku_models import MOKU_GO_PLATFORM
from riscure_models import DS1120A_PLATFORM

# Get platform output specs
moku_output = MOKU_GO_PLATFORM.get_analog_output_by_id('OUT1')
# Can output 3.3V TTL or ±5V analog

# Get probe input specs
probe_input = DS1120A_PLATFORM.get_port_by_id('digital_glitch')
# Accepts 0-3.3V TTL only

# Validate voltage compatibility
ttl_voltage = 3.3
if probe_input.is_voltage_compatible(ttl_voltage):
    print("✓ Safe: Moku OUT1 (TTL) → DS1120A digital_glitch")
else:
    raise ValueError("⚠ Unsafe voltage for probe!")
```

**Libraries involved:** moku-models + riscure-models

### Pattern 3: Code Generation + VHDL Components

**Use case:** Generated code references reusable VHDL components

```vhdl
-- In generated VHDL (from forge-codegen)
library work;
use work.forge_voltage_5v_bipolar_pkg.all;  -- From forge-vhdl
use work.basic_app_types_pkg.all;           -- From forge-codegen

signal threshold_voltage : real := 2.5;  -- Volts
signal threshold_digital : signed(15 downto 0);

-- Use voltage conversion from forge-vhdl
threshold_digital <= to_digital(threshold_voltage);
```

**Libraries involved:** forge-codegen + forge-vhdl

---

## Customization Guide

### Scenario 1: Different Platform (Not Moku)

**Goal:** Use this template for a different hardware platform (e.g., Red Pitaya, PicoScope)

**Steps:**

1. **Keep:**
   - `tools/forge-codegen/` (still useful for code generation)
   - `libs/forge-vhdl/` (generic VHDL components still work)
   - `.claude/` structure (AI agents, commands)

2. **Remove:**
   - `libs/moku-models/` (Moku-specific)
   - `libs/riscure-models/` (if not using Riscure probes)

3. **Add:**
   - `libs/your-platform-models/` (create new submodule for your platform)

4. **Update:**
   - Root `llms.txt` and `CLAUDE.md` (document new platform)
   - Integration code referencing moku-models

### Scenario 2: Different Code Generator

**Goal:** Replace forge-codegen with your own generator or different tool

**Steps:**

1. **Remove:**
   - `tools/forge-codegen/`

2. **Add:**
   - `tools/your-codegen/` (your own code generator)

3. **Keep:**
   - `libs/` (all foundational libraries still useful)
   - `.claude/` structure

4. **Update:**
   - Root documentation
   - Integration patterns

### Scenario 3: Minimal VHDL-Only Setup

**Goal:** Just want VHDL utilities and platform models, no code generation

**Steps:**

1. **Keep:**
   - `libs/forge-vhdl/`
   - `libs/moku-models/` (or your platform models)

2. **Remove:**
   - `tools/forge-codegen/`
   - `libs/riscure-models/` (if not needed)

3. **Simplify:**
   - Remove code generation commands from `.claude/commands/`
   - Update root documentation

### Scenario 4: Pure Python (No VHDL)

**Goal:** Just want the Pydantic models for platform/probe specs

**Steps:**

1. **Keep:**
   - `libs/moku-models/`
   - `libs/riscure-models/` (if relevant)

2. **Remove:**
   - `tools/forge-codegen/`
   - `libs/forge-vhdl/`

3. **Simplify:**
   - Most of `.claude/` can be removed
   - Update root docs to reflect Python-only usage

---

## Documentation Strategy

### 3-Tier System Explained

**Tier 1: llms.txt** (~500-1000 tokens)
- **Purpose:** Quick orientation, component catalog
- **When to read:** Always (load first, minimal cost)
- **Contains:** Exports, basic usage, pointers to Tier 2

**Tier 2: CLAUDE.md** (~3-5k tokens)
- **Purpose:** Complete design guide, integration patterns
- **When to read:** When designing, integrating, customizing
- **Contains:** Architecture, rationale, workflows, examples

**Tier 3: Source + Specialized Docs** (~10k+ tokens per file)
- **Purpose:** Implementation details, debugging
- **When to read:** When actually coding or debugging
- **Contains:** Source code, detailed references, troubleshooting

**Token Efficiency:**
```
Quick question:     Tier 1 only (1k tokens, 0.5% budget)
Design question:    Tier 1 + 2 (5k tokens, 2.5% budget)
Implementation:     Tier 1 + 2 + 3 (15k tokens, 7.5% budget)
Still have 92.5% budget remaining!
```

### Creating 3-Tier Docs for New Submodules

**Template for new submodule:**

1. **llms.txt:**
   ```markdown
   # your-module-name

   > One-line description

   ## What is this?
   Brief explanation

   ## Core Exports
   - Key classes/functions
   - Main types

   ## Basic Usage
   Quick example

   ## Common Tasks
   - Task 1: How to do it
   - Task 2: How to do it
   ```

2. **CLAUDE.md:**
   ```markdown
   # CLAUDE.md

   ## Project Overview
   Complete description, purpose, design rationale

   ## Quick Start
   Installation, basic setup

   ## Architecture
   Design decisions, key patterns

   ## Integration
   How to use with sibling libraries

   ## Development
   How to contribute, testing
   ```

3. **Specialized docs/** (optional)
   - API references
   - Troubleshooting guides
   - Deep-dive tutorials

---

## Git Submodule Workflows

### Initial Setup

```bash
# Clone with all submodules
git clone --recurse-submodules <repo-url>

# Or if already cloned
git submodule update --init --recursive
```

### Updating Submodules

```bash
# Update specific submodule to latest
cd libs/moku-models/
git pull origin main
cd ../..
git add libs/moku-models/
git commit -m "chore: Update moku-models to latest"

# Update all submodules to latest
git submodule update --remote --merge
git add .
git commit -m "chore: Update all submodules"
```

### Working on a Submodule

```bash
# Make changes IN the submodule
cd libs/your-module/
git checkout -b feature/new-feature
# Make changes
git add .
git commit -m "feat: Add new feature"
git push origin feature/new-feature
# Create PR in submodule repo

# After PR merged, update parent
git checkout main
git pull origin main
cd ../..
git add libs/your-module/
git commit -m "chore: Update your-module submodule"
```

### Checking Submodule Status

```bash
# See all submodule commits
git submodule status --recursive

# See which submodules have updates available
git submodule update --remote --dry-run
```

---

## Common Pitfalls & Solutions

### Pitfall 1: Submodule Out of Sync

**Problem:** `git status` shows submodule as modified but you didn't change it

**Solution:**
```bash
# Reset submodule to committed version
git submodule update --init libs/module-name/

# Or reset all submodules
git submodule update --init --recursive
```

### Pitfall 2: Cloning Without Submodules

**Problem:** Directories exist but are empty

**Solution:**
```bash
git submodule update --init --recursive
```

### Pitfall 3: Detached HEAD in Submodule

**Problem:** Submodule is in "detached HEAD" state

**Solution:**
```bash
cd libs/module-name/
git checkout main  # or appropriate branch
cd ../..
```

### Pitfall 4: Forgot to Commit Submodule Update

**Problem:** Updated submodule but parent still points to old commit

**Solution:**
```bash
# After updating submodule, always commit in parent
git add libs/module-name/
git commit -m "chore: Update module-name submodule"
```

---

## Testing the Template

Before using this as a template for real work:

1. **Clone fresh:** `git clone --recurse-submodules <url> test-clone/`
2. **Verify submodules:** `git submodule status --recursive`
3. **Install deps:** `uv sync`
4. **Run tests:** `pytest`
5. **Check docs:** AI should be able to navigate using llms.txt files
6. **Try customization:** Remove a submodule and verify everything still works

---

## Migration from v1.0

If you have an existing monorepo using the old nested `forge/` structure:

**Key changes in v2.0:**
- Eliminated `forge/` nested submodule
- Moved libs to flat `libs/` structure
- Created separate `tools/forge-codegen/`
- Archived probe-specific workflows

**See:** `.claude/shared/archive/ARCHITECTURE_OVERVIEW_v1.0.md` for v1.0 structure

---

## Resources

### Documentation
- **This file:** Complete architecture and template guide
- **llms.txt:** Quick component catalog (start here for AI navigation)
- **ARCHITECTURE_OVERVIEW.md:** Technical architecture deep-dive
- **TEMPLATE.md:** Step-by-step customization guide (coming soon)

### Community
- **Issues:** https://github.com/sealablab/moku-instrument-forge-mono-repo/issues
- **Discussions:** https://github.com/sealablab/moku-instrument-forge-mono-repo/discussions

### Related Projects
- forge-codegen: https://github.com/sealablab/moku-instrument-forge-codegen
- forge-vhdl: https://github.com/sealablab/moku-instrument-forge-vhdl
- moku-models: https://github.com/sealablab/moku-models
- riscure-models: https://github.com/sealablab/riscure-models

---

## Contributing to the Template

Improvements welcome! This template should evolve with best practices.

**Good PRs:**
- Additional composability patterns
- Better customization examples
- Improved documentation clarity
- New integration patterns

**Before submitting:**
- Ensure 3-tier docs are updated
- Test that template still works after changes
- Update this CLAUDE.md if architecture changes

---

## License

MIT License - See [LICENSE](LICENSE)

---

**Version:** 2.0.0 (Template-Ready)
**Last Updated:** 2025-11-04
**Maintained By:** Sealab Team

**This is a template. Fork it. Customize it. Make it yours.** 🚀
