# moku-instrument-forge-mono-repo

Monorepo for developing Moku custom EMFI probe drivers using the forge framework.

**Status:** Phase 4 complete, Option A architecture validated ✅
**Version:** `v0.1.0-phase4` (2025-11-03)
**Architecture:** Agent-based development workflow with forge/apps/ as workspace
**Synchronized:** `monorepo-init-v1.0.0` (2025-11-03)

---

## Monorepo Structure

This repository is composed of git submodules organized in a hierarchical structure:

### [moku-instrument-forge](https://github.com/sealablab/moku-instrument-forge)
> **Location:** `forge/`
> **Purpose:** Code generation framework (YAML → VHDL)

Generates VHDL interface code (shim + main template) from YAML specifications.

#### [basic-app-datatypes](https://github.com/sealablab/basic-app-datatypes)
> **Location:** `forge/libs/basic-app-datatypes/`
> **Purpose:** Pydantic models for register package types

#### [moku-models](https://github.com/sealablab/moku-models)
> **Location:** `forge/libs/moku-models/`
> **Purpose:** Moku platform specifications (Go/Lab/Pro/Delta)

#### [riscure-models](https://github.com/sealablab/riscure-models)
> **Location:** `forge/libs/riscure-models/`
> **Purpose:** Riscure probe hardware specifications (DS1120A/DS1140A)

### [moku-instrument-forge-vhdl](https://github.com/sealablab/moku-instrument-forge-vhdl)
> **Location:** `libs/forge-vhdl/`
> **Purpose:** Shared VHDL utilities and packages

Provides reusable VHDL components:
- **packages/** - Common types, voltage utilities, LUTs
- **debugging/** - FSM observer for hardware debugging
- **loader/** - BRAM initialization utilities
- **utilities/** - Clock dividers, triggers, helpers

---

## Directory Layout (Option A Architecture)

```
moku-instrument-forge-mono-repo/
│
├── forge/                              # Submodule: moku-instrument-forge
│   ├── apps/                           # Probe packages (YAML + generated VHDL)
│   │   └── DS1140_PD/                  # Example probe package
│   │       ├── DS1140_PD.yaml          # Source specification
│   │       ├── *_shim.vhd              # Auto-generated interface
│   │       ├── *_main.vhd              # Template for implementation
│   │       └── README.md               # Documentation
│   ├── generator/                      # YAML → VHDL code generation
│   ├── templates/                      # Jinja2 templates
│   └── libs/
│       ├── basic-app-datatypes/        # Submodule: Register types
│       ├── moku-models/                # Submodule: Platform specs
│       └── riscure-models/             # Submodule: Probe specs
│
├── libs/
│   ├── forge-vhdl/                     # Submodule: VHDL utilities
│   └── platform/common/                # Platform-specific VHDL (future)
│
├── .claude/                            # AI agent configurations (Phase 4)
│   ├── agents/                         # Monorepo-level agents
│   │   ├── deployment-orchestrator/    # Package → hardware deployment
│   │   ├── hardware-debug/             # FSM debugging
│   │   └── probe-design-orchestrator/  # Probe workflow coordination
│   ├── commands/                       # Slash commands
│   └── shared/                         # Shared documentation
│
├── tests/                              # Shared test infrastructure
├── scripts/                            # Helper scripts
└── pyproject.toml                      # Python dependencies + pytest config
```

**Key Change (Option A):** All probe development happens in `forge/apps/<probe_name>/`:
- YAML specifications
- Generated VHDL files
- User implementation

---

## Quick Start

### Clone with Submodules

**Note:** This repository uses **nested submodules** - the `forge/` submodule contains three additional submodules (`libs/basic-app-datatypes`, `libs/moku-models`, `libs/riscure-models`). The `--recurse-submodules` flag handles this automatically.

```bash
git clone --recurse-submodules https://github.com/sealablab/moku-instrument-forge-mono-repo.git
cd moku-instrument-forge-mono-repo
```

Already cloned? Initialize all submodules (including nested ones):

```bash
git submodule update --init --recursive
```

Verify all submodules are properly initialized:

```bash
git submodule status --recursive
```

Expected output:
```
 <commit-hash> forge (tag)
 <commit-hash> forge/libs/basic-app-datatypes (tag)
 <commit-hash> forge/libs/moku-models (tag)
 <commit-hash> forge/libs/riscure-models (tag)
 <commit-hash> libs/forge-vhdl (tag)
```

### Setup Python Environment

```bash
uv sync
python scripts/setup_forge_path.py  # Verify forge imports
pytest                               # Run tests
```

---

## Documentation

Each submodule maintains its own documentation:

| Repository | Purpose | Documentation |
|------------|---------|---------------|
| [moku-instrument-forge](https://github.com/sealablab/moku-instrument-forge) | Code generation (YAML → VHDL) | [forge/README.md](forge/README.md) |
| [moku-instrument-forge-vhdl](https://github.com/sealablab/moku-instrument-forge-vhdl) | Shared VHDL utilities | [libs/forge-vhdl/README.md](libs/forge-vhdl/README.md) |
| [basic-app-datatypes](https://github.com/sealablab/basic-app-datatypes) | Register package types | [forge/libs/basic-app-datatypes/README.md](forge/libs/basic-app-datatypes/README.md) |
| [moku-models](https://github.com/sealablab/moku-models) | Platform specifications | [forge/libs/moku-models/README.md](forge/libs/moku-models/README.md) |
| [riscure-models](https://github.com/sealablab/riscure-models) | Probe specifications | [forge/libs/riscure-models/README.md](forge/libs/riscure-models/README.md) |

---

## Development Workflow

### Adding a New Probe (Option A)

1. **Initialize probe directory:**
   ```bash
   /init-probe NEW_PROBE  # Creates forge/apps/NEW_PROBE/
   ```

2. **Write YAML specification:**
   - Edit `forge/apps/NEW_PROBE/NEW_PROBE.yaml`
   - Define datatypes, platform, mapping strategy

3. **Validate YAML:**
   ```bash
   /validate forge/apps/NEW_PROBE/NEW_PROBE.yaml
   ```

4. **Generate VHDL interface:**
   ```bash
   /generate forge/apps/NEW_PROBE/NEW_PROBE.yaml
   ```
   Creates:
   - `*_shim.vhd` (auto-generated, DO NOT EDIT)
   - `*_main.vhd` (template for your implementation)

5. **Implement custom logic:**
   - Edit `forge/apps/NEW_PROBE/NEW_PROBE_custom_inst_main.vhd`
   - Use friendly signal names from YAML

6. **Deploy and test:**
   ```bash
   /deploy NEW_PROBE --device <ip>
   /monitor-state NEW_PROBE
   ```

See [.claude/shared/PROBE_WORKFLOW.md](.claude/shared/PROBE_WORKFLOW.md) for detailed workflow.

### AI Agent Commands

Phase 4 includes AI agents for probe development:

```bash
/probe-status                          # Show all probe states
/init-probe <name>                     # Create new probe
/validate forge/apps/<name>/<name>.yaml  # Validate YAML
/generate forge/apps/<name>/<name>.yaml  # Generate VHDL
/deploy <name> --device <ip>           # Deploy to hardware
```

See [.claude/commands/](.claude/commands/) for all available commands.

### Updating Submodules

```bash
# Update a submodule to a specific version
cd forge
git fetch origin
git checkout <commit-or-tag>
cd ..

# Commit the update
git add forge
git commit -m "chore: Update forge to <version>"
```

### Testing

```bash
pytest              # All tests
pytest libs/        # Library tests only
pytest forge/       # Forge tests
pytest -n auto      # Parallel execution
```

Configuration in `pyproject.toml`.

**Note:** Probe-specific tests would go in `forge/apps/<probe_name>/tests/` (future).

---

## Submodule Synchronization

All submodules are tagged at synchronized states for reproducible builds.

**Current sync point:** `monorepo-init-v1.0.0` (2025-11-03)

### Checkout Synchronized State

```bash
# Checkout all submodules to the synchronized tag
git submodule foreach --recursive 'git checkout monorepo-init-v1.0.0 || true'
```

### Create New Sync Point

When updating multiple submodules, tag the new synchronized state:

```bash
# In each updated submodule
cd forge
git tag -a monorepo-init-v1.1.0 -m "Synchronized for monorepo v1.1.0"
git push origin monorepo-init-v1.1.0
cd ..

# Repeat for other submodules...
```

---

## Architecture

### Option A (Current)

**Primary workspace:** `forge/apps/<probe_name>/`
- YAML specifications
- Generated VHDL files
- User implementations
- Documentation

**Benefits:**
- Simple mental model: "everything in one place"
- Works with forge as-is (no modifications needed)
- Clear separation: source + generated in same directory
- Validated and tested

### Phase 4 Agent System

Hierarchical AI agent organization for probe development:

**Monorepo-level agents** (`.claude/agents/`):
- `deployment-orchestrator` - Hardware deployment
- `hardware-debug` - FSM debugging and monitoring
- `probe-design-orchestrator` - Workflow coordination

**Forge-level agents** (`forge/.claude/agents/`):
- `forge-context` - YAML validation and package generation
- `docgen-context` - Documentation generation
- `forge-pipe-fitter` - Multi-stage pipeline coordination

### Foundation Patterns

From [moku-spike-redux](https://github.com/sealablab/moku-spike-redux):

- Git submodules for dependency management
- CocotB + pytest for VHDL testing (no Makefiles)
- Hierarchical composition for modularity
- Synchronized tagging for reproducible builds

---

## Contributing

1. Make changes in appropriate submodule
2. Write CocotB tests for VHDL changes
3. Validate with `pytest`
4. Commit to submodule, then update monorepo reference
5. Tag new sync point if multiple submodules updated

---

## License

MIT License - See [LICENSE](LICENSE)
