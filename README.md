# moku-instrument-forge-mono-repo

Monorepo for developing Moku custom EMFI probe drivers using the forge framework.

**Status:** Architecture refactored ✅
**Version:** `v2.0.0` (2025-11-04)
**Architecture:** Clean separation - tools, libraries, and legacy code
**Latest update:** Lifted foundational libraries, introduced forge-codegen

---

## Monorepo Structure

This repository uses git submodules organized in a flat, clean hierarchy:

### Tools

#### [moku-instrument-forge-codegen](https://github.com/sealablab/moku-instrument-forge-codegen)
> **Location:** `tools/forge-codegen/`
> **Purpose:** YAML → VHDL code generator with type-safe register serialization

**NEW!** Fresh, self-contained code generator with flattened `basic_serialized_datatypes` (internal serialization engine).

### Foundational Libraries

#### [moku-instrument-forge-vhdl](https://github.com/sealablab/moku-instrument-forge-vhdl)
> **Location:** `libs/forge-vhdl/`
> **Purpose:** Reusable VHDL components

Provides common VHDL utilities:
- **packages/** - Common types, voltage utilities, LUTs
- **debugging/** - FSM observer for hardware debugging
- **loader/** - BRAM initialization utilities
- **utilities/** - Clock dividers, triggers, helpers

#### [moku-models](https://github.com/sealablab/moku-models)
> **Location:** `libs/moku-models/`
> **Purpose:** Moku platform specifications (Go/Lab/Pro/Delta)

Pydantic models for platform specs, deployment configs, signal routing.

#### [riscure-models](https://github.com/sealablab/riscure-models)
> **Location:** `libs/riscure-models/`
> **Purpose:** Riscure probe hardware specifications (DS1120A/DS1140A)

Pydantic models for probe specs, voltage safety validation.

---

## Directory Layout

```
moku-instrument-forge-mono-repo/
│
├── tools/                              # Development tools
│   └── forge-codegen/                  # Submodule: YAML → VHDL generator
│       ├── forge_codegen/
│       │   ├── basic_serialized_datatypes/  # Type system (internal)
│       │   ├── generator/              # Code generation engine
│       │   ├── models/                 # Pydantic models
│       │   ├── templates/              # Jinja2 VHDL templates
│       │   └── vhdl/                   # Frozen type packages
│       ├── tests/                      # 69 tests
│       └── docs/                       # High-quality documentation
│
├── libs/                               # Foundational libraries
│   ├── forge-vhdl/                     # Submodule: VHDL utilities
│   ├── moku-models/                    # Submodule: Platform specs
│   ├── riscure-models/                 # Submodule: Probe specs
│   └── platform/                       # Platform-specific VHDL
│
├── .claude/                            # AI agent configurations
│   ├── agents/                         # Specialized agents
│   ├── commands/                       # Slash commands
│   ├── workflows/                      # Reusable workflows
│   └── shared/                         # Architecture docs
│
├── docs/                               # Monorepo documentation
├── scripts/                            # Helper scripts
└── pyproject.toml                      # Python workspace config
```

**Architecture v2.0:** Clean separation
- **tools/** - Code generation (forge-codegen)
- **libs/** - Foundational libraries (flat, no nesting)

---

## Quick Start

### Clone with Submodules

```bash
git clone --recurse-submodules https://github.com/sealablab/moku-instrument-forge-mono-repo.git
cd moku-instrument-forge-mono-repo
```

Already cloned? Initialize all submodules:

```bash
git submodule update --init --recursive
```

Verify all submodules are properly initialized:

```bash
git submodule status --recursive
```

Expected output:
```
 <commit-hash> libs/forge-vhdl (tag)
 <commit-hash> libs/moku-models (tag)
 <commit-hash> libs/riscure-models (tag)
 <commit-hash> tools/forge-codegen (tag)
```

### Setup Python Environment

```bash
uv sync
python scripts/setup_forge_path.py  # Verify forge imports
pytest                               # Run tests
```

---

## Documentation

Each submodule follows a **3-tier documentation system**:

| Repository | Purpose | Quick Ref | Full Guide |
|------------|---------|-----------|------------|
| [forge-codegen](https://github.com/sealablab/moku-instrument-forge-codegen) | YAML → VHDL generator | [llms.txt](tools/forge-codegen/llms.txt) | [CLAUDE.md](tools/forge-codegen/CLAUDE.md) |
| [forge-vhdl](https://github.com/sealablab/moku-instrument-forge-vhdl) | VHDL utilities | [llms.txt](libs/forge-vhdl/llms.txt) | [CLAUDE.md](libs/forge-vhdl/CLAUDE.md) |
| [moku-models](https://github.com/sealablab/moku-models) | Platform specs | [llms.txt](libs/moku-models/llms.txt) | [CLAUDE.md](libs/moku-models/CLAUDE.md) |
| [riscure-models](https://github.com/sealablab/riscure-models) | Probe specs | [llms.txt](libs/riscure-models/llms.txt) | [CLAUDE.md](libs/riscure-models/CLAUDE.md) |

**Progressive disclosure:** Start with llms.txt (~500-1000 tokens), escalate to CLAUDE.md (~3-5k tokens) for complete context.

---

## Architecture Migration (v2.0.0)

**What changed (2025-11-04):**

1. **Created forge-codegen** - Fresh, self-contained YAML → VHDL generator
   - Flattened `basic_serialized_datatypes` (no longer separate repo)
   - Clean package naming: `forge_codegen` (was `forge`)
   - All imports updated, 69 tests passing

2. **Lifted foundational libraries** - Moved to top-level `libs/`
   - `libs/moku-models/` (was `forge/libs/moku-models/`)
   - `libs/riscure-models/` (was `forge/libs/riscure-models/`)
   - Now peers with `libs/forge-vhdl/`

3. **Removed legacy forge/** - Eliminated nested submodule structure
   - Old `forge/` directory with nested submodules has been removed
   - Historical probe packages archived elsewhere
   - Use `tools/forge-codegen/` for all code generation

**Benefits:**
- Clean separation (tools vs libraries)
- No nested submodules
- Simplified maintenance
- Better naming (forge_codegen vs ambiguous "forge")

---

## Development Workflow

### Using forge-codegen

```bash
# Generate VHDL from YAML spec
python -m forge_codegen.generator.codegen spec.yaml --output-dir generated/
```

See [tools/forge-codegen/](tools/forge-codegen/) for complete documentation.

### Updating Submodules

```bash
# Update a submodule to a specific version
cd tools/forge-codegen
git fetch origin
git checkout <commit-or-tag>
cd ../..

# Commit the update
git add tools/forge-codegen
git commit -m "chore: Update forge-codegen to <version>"
```

### Testing

```bash
pytest              # All tests
pytest libs/        # Library tests only
pytest tools/       # Tool tests
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

## Architecture

See [.claude/shared/ARCHITECTURE_OVERVIEW.md](.claude/shared/ARCHITECTURE_OVERVIEW.md) for complete v2.0 architecture details.

**Key Principles:**
- Clean separation: tools/ vs libs/ (no nested submodules)
- Self-contained authoritative modules
- 3-tier documentation system (llms.txt → CLAUDE.md → source)
- Git submodules for dependency management
- CocoTB + pytest for VHDL testing

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
