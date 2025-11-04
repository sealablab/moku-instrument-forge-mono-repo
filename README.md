# moku-instrument-forge-mono-repo

Monorepo for Moku custom EMFI probe drivers using the [moku-instrument-forge](https://github.com/sealablab/moku-instrument-forge) framework.

## Overview

This monorepo provides a structured workspace for developing custom Moku instruments (EMFI probe drivers) by composing git submodules from the Moku forge ecosystem.

**Status:** ⚠️ **Infrastructure ready** - Probe implementations in progress
**Architecture validated:** [moku-spike-redux](https://github.com/sealablab/moku-spike-redux)
**Synchronized at:** `monorepo-init-v1.0.0` (2025-11-03)

---

## Repository Composition

This monorepo is **hierarchically composed** of the following submodules:

### 🔧 [moku-instrument-forge](https://github.com/sealablab/moku-instrument-forge) → `forge/`
**Code generation framework** (YAML → VHDL)

Generates interface code (shim + main template) for custom instruments.

**Nested submodules:**
- 📦 [basic-app-datatypes](https://github.com/sealablab/basic-app-datatypes) → `forge/libs/basic-app-datatypes/`
  Register package type definitions (Pydantic models)

- 🔌 [moku-models](https://github.com/sealablab/moku-models) → `forge/libs/moku-models/`
  Moku platform specifications (Go/Lab/Pro/Delta hardware models)

- 🧪 [riscure-models](https://github.com/sealablab/riscure-models) → `forge/libs/riscure-models/`
  Riscure probe specifications (DS1120A/DS1140A datasheets + models)

### ⚡ [moku-instrument-forge-vhdl](https://github.com/sealablab/moku-instrument-forge-vhdl) → `libs/forge-vhdl/`
**Shared VHDL utilities** (packages, debugging, loaders)

Reusable VHDL components:
- `vhdl/packages/` - Common types and voltage utilities
- `vhdl/debugging/` - FSM observer for hardware debugging
- `vhdl/loader/` - BRAM initialization utilities
- `vhdl/utilities/` - Clock dividers, triggers, helpers

---

## Repository Structure

```
moku-instrument-forge-mono-repo/
│
├── forge/                              # Submodule: moku-instrument-forge
│   ├── generator/                      # YAML → VHDL code generation
│   ├── templates/                      # Jinja2 templates
│   └── libs/                           # Nested submodules
│       ├── basic-app-datatypes/        # Register types
│       ├── moku-models/                # Platform specs
│       └── riscure-models/             # Probe specs
│
├── libs/
│   ├── forge-vhdl/                     # Submodule: moku-instrument-forge-vhdl
│   │   └── vhdl/{packages,debugging,loader,utilities}/
│   └── platform/                       # Platform-specific VHDL (future)
│       └── common/
│
├── probes/                             # Probe implementations (empty, ready)
│   ├── DS1120_PD/{generated,vhdl,tests}/
│   └── DS1140_PD/{generated,vhdl,tests}/
│
├── archive/                            # Reference implementations from EZ-EMFI
│   └── ez-emfi-probes/                 # See archive/README.md
│       ├── DS1120_PD/                  # DS1120A reference (volo-based)
│       └── DS1140_PD/                  # DS1140A reference (volo-based)
│
├── tests/                              # Shared test infrastructure
├── scripts/                            # Helper scripts (forge path setup)
├── .claude/                            # AI agent configurations
├── pyproject.toml                      # Python deps + pytest config
└── README.md                           # This file
```

---

## Quick Start

### 1. Clone with Submodules

**⚠️ IMPORTANT:** Clone with `--recurse-submodules` to initialize all nested submodules:

```bash
git clone --recurse-submodules https://github.com/sealablab/moku-instrument-forge-mono-repo.git
cd moku-instrument-forge-mono-repo
```

If already cloned without submodules:

```bash
git submodule update --init --recursive
```

### 2. Setup Python Environment

```bash
uv sync
python scripts/setup_forge_path.py  # Verify forge imports
```

### 3. Verify Installation

```bash
pytest  # Run tests (currently minimal)
```

---

## Submodule Documentation

Each submodule has its own README with detailed documentation:

| Submodule | Purpose | Documentation |
|-----------|---------|---------------|
| **[moku-instrument-forge](https://github.com/sealablab/moku-instrument-forge)** | Code generation (YAML → VHDL) | [forge/README.md](forge/README.md) |
| **[moku-instrument-forge-vhdl](https://github.com/sealablab/moku-instrument-forge-vhdl)** | Shared VHDL utilities | [libs/forge-vhdl/README.md](libs/forge-vhdl/README.md) |
| **[basic-app-datatypes](https://github.com/sealablab/basic-app-datatypes)** | Register types (Pydantic) | [forge/libs/basic-app-datatypes/README.md](forge/libs/basic-app-datatypes/README.md) |
| **[moku-models](https://github.com/sealablab/moku-models)** | Platform specifications | [forge/libs/moku-models/README.md](forge/libs/moku-models/README.md) |
| **[riscure-models](https://github.com/sealablab/riscure-models)** | Probe specifications | [forge/libs/riscure-models/README.md](forge/libs/riscure-models/README.md) |

---

## Development Workflow

### Adding a New Probe

1. **Create YAML specification** in `forge/apps/NEW_PROBE.yaml`
2. **Generate interface code** using forge → `probes/NEW_PROBE/generated/`
3. **Implement custom FSM** in `probes/NEW_PROBE/vhdl/`
4. **Write CocotB tests** in `probes/NEW_PROBE/tests/`
5. **Run tests** with `pytest probes/NEW_PROBE/`

See [forge/README.md](forge/README.md) for detailed code generation workflow.

### Updating Submodules

```bash
# Update specific submodule
cd forge  # or libs/forge-vhdl
git fetch origin
git checkout <commit-hash-or-tag>
cd ../..
git add forge  # or libs/forge-vhdl
git commit -m "chore: Update forge to <version>"
git push

# Tag synchronized state (optional)
# See "Submodule Synchronization" section below
```

---

## Submodule Synchronization

All submodules are tagged at synchronized states for reproducible builds:

**Current sync point:** `monorepo-init-v1.0.0` (2025-11-03)

To checkout a specific sync point:

```bash
# In each submodule
cd forge
git checkout monorepo-init-v1.0.0
cd ../libs/forge-vhdl
git checkout monorepo-init-v1.0.0
# ... repeat for nested submodules
```

Or use git submodule commands:

```bash
git submodule foreach --recursive 'git checkout monorepo-init-v1.0.0 || true'
```

---

## Testing

```bash
pytest                  # Run all tests
pytest libs/            # Library tests only
pytest probes/          # Probe tests only
pytest -n auto          # Parallel execution
```

See `pyproject.toml` for pytest configuration.

---

## Architecture

This monorepo follows patterns validated in [moku-spike-redux](https://github.com/sealablab/moku-spike-redux):

- ✅ **Git submodules** for clean dependency management
- ✅ **CocotB + pytest** for VHDL testing (no Makefiles)
- ✅ **Hierarchical composition** for modularity
- ✅ **Synchronized tagging** for reproducible builds

---

## Reference Implementations

The `archive/` directory contains reference implementations from the EZ-EMFI project:

- **DS1120_PD** - DS1120A probe driver (volo-based)
- **DS1140_PD** - DS1140A probe driver (volo-based)

⚠️ **Status:** Reference only, not actively maintained. See [archive/README.md](archive/README.md).

---

## Contributing

1. Make changes in appropriate submodule or monorepo
2. Write CocotB tests for VHDL changes
3. Run `pytest` to validate
4. Commit to submodule first, then update monorepo reference
5. Consider tagging new sync point (e.g., `monorepo-init-v1.1.0`)

---

## License

MIT License - See [LICENSE](LICENSE)

---

## Links

- **Code generation:** [moku-instrument-forge](https://github.com/sealablab/moku-instrument-forge)
- **VHDL utilities:** [moku-instrument-forge-vhdl](https://github.com/sealablab/moku-instrument-forge-vhdl)
- **Platform models:** [moku-models](https://github.com/sealablab/moku-models)
- **Probe models:** [riscure-models](https://github.com/sealablab/riscure-models)
- **Register types:** [basic-app-datatypes](https://github.com/sealablab/basic-app-datatypes)
- **Architecture validation:** [moku-spike-redux](https://github.com/sealablab/moku-spike-redux)

---

**Questions?** Open an issue or see submodule documentation for detailed guides.
