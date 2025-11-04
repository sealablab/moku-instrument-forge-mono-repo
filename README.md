# moku-instrument-forge-mono-repo

Monorepo for developing Moku custom EMFI probe drivers using the forge framework.

**Status:** Infrastructure complete, ready for probe development
**Synchronized:** `monorepo-init-v1.0.0` (2025-11-03)
**Architecture:** Validated in [moku-spike-redux](https://github.com/sealablab/moku-spike-redux)

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

## Directory Layout

```
moku-instrument-forge-mono-repo/
│
├── forge/                              # Submodule: moku-instrument-forge
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
├── probes/
│   ├── DS1120_PD/                      # DS1120A probe (empty, ready)
│   │   ├── generated/                  # Forge-generated code
│   │   ├── vhdl/                       # Custom FSM implementation
│   │   └── tests/                      # CocotB tests
│   └── DS1140_PD/                      # DS1140A probe (empty, ready)
│       ├── generated/
│       ├── vhdl/
│       └── tests/
│
├── archive/
│   └── ez-emfi-probes/                 # Reference implementations (read-only)
│       ├── DS1120_PD/                  # Old DS1120A implementation
│       └── DS1140_PD/                  # Old DS1140A implementation
│
├── tests/                              # Shared test infrastructure
├── scripts/                            # Helper scripts
├── .claude/                            # AI agent configurations
└── pyproject.toml                      # Python dependencies + pytest config
```

---

## Quick Start

### Clone with Submodules

```bash
git clone --recurse-submodules https://github.com/sealablab/moku-instrument-forge-mono-repo.git
cd moku-instrument-forge-mono-repo
```

Already cloned? Initialize submodules:

```bash
git submodule update --init --recursive
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

### Adding a New Probe

1. Create YAML specification: `forge/apps/NEW_PROBE.yaml`
2. Generate interface code → `probes/NEW_PROBE/generated/`
3. Implement custom FSM → `probes/NEW_PROBE/vhdl/`
4. Write CocotB tests → `probes/NEW_PROBE/tests/`
5. Run tests: `pytest probes/NEW_PROBE/`

See [forge/README.md](forge/README.md) for detailed code generation workflow.

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
pytest probes/      # Probe tests only
pytest -n auto      # Parallel execution
```

Configuration in `pyproject.toml`.

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

## Archive

The `archive/` directory contains reference implementations from the EZ-EMFI project:

- **DS1120_PD** - DS1120A probe driver (volo-based)
- **DS1140_PD** - DS1140A probe driver (volo-based)

**Status:** Reference only, not actively maintained.
**See:** [archive/README.md](archive/README.md) for details.

---

## Architecture

Patterns validated in [moku-spike-redux](https://github.com/sealablab/moku-spike-redux):

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
