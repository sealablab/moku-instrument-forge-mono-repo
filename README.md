# moku-instrument-forge-mono-repo

Monorepo for Moku custom EMFI probe drivers using the forge framework.

## Overview

This repository contains custom instrument implementations for the Moku platform:

- **DS1120-PD** - Driver for Riscure DS1120A EMFI probe
- **DS1140-PD** - Driver for Riscure DS1140A EMFI probe

The monorepo brings together:
- **forge** - Code generation framework (YAML → VHDL)
- **forge-vhdl** - Shared VHDL utilities (packages, debugging, loaders)
- **Platform VHDL** - Moku platform-specific packages
- **Probe implementations** - Custom FSM logic for each probe
- **Tests** - CocotB test infrastructure

## Repository Structure

```
moku-instrument-forge-mono-repo/
├── forge/                          # Git submodule: moku-instrument-forge
│   ├── generator/                  # Code generation (YAML → VHDL)
│   └── libs/                       # Submodules (basic-app-datatypes, moku-models, riscure-models)
├── libs/
│   ├── platform/                   # Platform-specific VHDL packages
│   │   └── common/
│   │       └── custom_inst_common_pkg.vhd
│   └── forge-vhdl/                 # Git submodule: moku-instrument-forge-vhdl
│       └── vhdl/
│           ├── packages/           # volo_voltage_pkg, volo_common_pkg, volo_lut_pkg
│           ├── debugging/          # fsm_observer
│           ├── loader/             # volo_bram_loader
│           └── utilities/          # volo_clk_divider, volo_voltage_threshold_trigger_core
├── probes/
│   ├── DS1120_PD/                  # DS1120A probe driver
│   │   ├── generated/              # Forge output (shim + main template)
│   │   ├── vhdl/                   # Custom FSM logic
│   │   └── tests/                  # Progressive CocotB tests
│   └── DS1140_PD/                  # DS1140A probe driver
│       ├── generated/
│       ├── vhdl/
│       └── tests/
├── tests/
│   ├── conftest.py                 # Shared fixtures (importable)
│   ├── run.py                      # Custom runner (excluded from pytest)
│   └── test_configs.py             # Build configuration
├── scripts/
│   └── setup_forge_path.py         # Helper for forge imports
├── .claude/
│   ├── agents/                     # Claude Code AI agents
│   │   ├── probe-design-context/   # FSM design agent
│   │   ├── deployment-context/     # Hardware deployment
│   │   └── hardware-debug-context/ # Oscilloscope debugging
│   ├── commands/                   # Context switching commands
│   └── shared/                     # Shared knowledge
├── pyproject.toml                  # Python dependencies + pytest config
└── README.md                       # This file
```

## Getting Started

### Prerequisites

- **Git** with submodule support
- **Python 3.10+** with uv package manager
- **GHDL** (VHDL compiler)
- **CocotB** (for testing)

### 1. Clone the Repository

**IMPORTANT:** You must clone with `--recurse-submodules` to initialize all submodules:

```bash
git clone --recurse-submodules https://github.com/sealablab/moku-instrument-forge-mono-repo.git
cd moku-instrument-forge-mono-repo
```

If you already cloned without `--recurse-submodules`, run:

```bash
git submodule update --init --recursive
```

### 2. Setup Python Environment

```bash
# Install dependencies with uv
uv sync

# Verify forge imports work
python scripts/setup_forge_path.py
```

You should see:
```
✅ Forge path: /path/to/moku-instrument-forge-mono-repo/forge
✅ Forge imports working correctly
```

### 3. Run Tests

```bash
# Run all tests (libs + probes)
pytest

# Run tests for specific probe
pytest probes/DS1140_PD/

# Run tests in parallel (faster)
pytest -n auto

# Run with verbose output
pytest -v
```

## Development Workflows

### Adding a New Probe

1. **Create probe directory structure:**
   ```bash
   mkdir -p probes/NEW_PROBE/{generated,vhdl,tests}
   ```

2. **Create YAML configuration in forge:**
   ```bash
   cd forge/apps
   # Create NEW_PROBE.yaml with probe specifications
   ```

3. **Generate VHDL interface:**
   ```python
   from scripts.setup_forge_path import setup_forge_path
   setup_forge_path()
   from forge.generator import BasicAppsRegPackage

   # Load YAML and generate VHDL
   pkg = BasicAppsRegPackage.from_yaml("forge/apps/NEW_PROBE.yaml")
   # ... generate shim and main template
   ```

4. **Implement custom FSM logic:**
   ```bash
   # Write your FSM in probes/NEW_PROBE/vhdl/
   ```

5. **Write CocotB tests:**
   ```bash
   # Create tests in probes/NEW_PROBE/tests/test_NEW_PROBE.py
   ```

6. **Update test_configs.py:**
   ```python
   # Add build configuration for NEW_PROBE
   ```

7. **Run tests:**
   ```bash
   pytest probes/NEW_PROBE/
   ```

### Updating Forge

The forge submodule is pinned to a specific commit. To update:

```bash
cd forge
git fetch origin
git checkout <new-commit-hash>
cd ..
git add forge
git commit -m "chore: Update forge to <commit-hash>"
```

### Updating forge-vhdl

```bash
cd libs/forge-vhdl
git fetch origin
git checkout main
git pull
cd ../..
git add libs/forge-vhdl
git commit -m "chore: Update forge-vhdl utilities"
```

## Testing

### Test Organization

Tests are organized by component:

- **`libs/forge-vhdl/tests/`** - Tests for shared VHDL utilities
- **`probes/*/tests/`** - Tests for each probe implementation

### Test Execution

```bash
# All tests
pytest

# Library tests only
pytest libs/

# Probe tests only
pytest probes/

# Single probe
pytest probes/DS1140_PD/

# Parallel execution (4 workers)
pytest -n 4

# With coverage
pytest --cov=forge --cov=probes
```

### Test Discovery

Pytest is configured to:
- Search in `libs/` and `probes/` directories
- Match files named `test_*.py`
- Exclude custom runners (`tests/run.py`, `tests/test_configs.py`)
- Exclude simulation artifacts (`sim_build/`, `__pycache__/`)

See `pyproject.toml` for complete pytest configuration.

## Build System

### Compilation Pattern

This repository uses **CocotB + pytest** for VHDL compilation and testing:

1. **test_configs.py** - Declares VHDL sources and dependencies
2. **run_test.py** - Uses `cocotb_tools.runner` API to compile and test
3. **No Makefiles** - Python orchestrates everything

### Example Build Configuration

```python
# tests/test_configs.py
from pathlib import Path

TESTS_CONFIG = {
    "ds1140_pd_build": TestConfig(
        sources=[
            Path("libs/platform/common/custom_inst_common_pkg.vhd"),
            Path("libs/forge-vhdl/vhdl/packages/volo_voltage_pkg.vhd"),
            Path("probes/DS1140_PD/generated/DS1140_PD_custom_inst_main.vhd"),
            Path("probes/DS1140_PD/vhdl/DS1140_PD_fsm.vhd"),
        ],
        toplevel="DS1140_PD_custom_inst_main",
        ghdl_args=["--std=08"],
    )
}
```

## Architecture Decisions

This monorepo structure was validated through spike testing (see [moku-spike-redux](https://github.com/sealablab/moku-spike-redux)):

### ✅ Forge as Git Submodule
- Clean version control
- No code duplication
- Easy to update and track versions

### ✅ CocotB/Pytest Build Pattern
- No Makefiles needed
- Extremely fast builds (< 1 second)
- Python orchestration is maintainable

### ✅ Centralized Test Infrastructure
- Instant test discovery (0.00s)
- Run all tests or subset by directory
- Parallel execution support

### ✅ Agent Boundaries
- Clear R/W scopes prevent conflicts
- Agents for: probe design, deployment, hardware debugging
- See `.claude/agents/` for details

## Claude Code Integration

This repository is optimized for development with Claude Code:

### Context Switching

Use slash commands to load focused context:

- **`/vhdl`** - VHDL development context
- **`/python`** - Python tooling context
- **`/test`** - Testing context
- **`/forge`** - Code generation context

### Agent Architecture

Specialized agents handle different domains:

- **probe-design-context** - FSM design and testing
- **deployment-context** - Hardware deployment
- **hardware-debug-context** - Oscilloscope debugging

See `.claude/agents/*/agent.md` for details.

## Troubleshooting

### Submodule Issues

**Problem:** Submodules not initialized
```bash
fatal: not a git repository: forge/.git
```

**Solution:**
```bash
git submodule update --init --recursive
```

### Forge Import Errors

**Problem:** Cannot import from forge
```bash
ImportError: No module named 'forge'
```

**Solution:**
```python
from scripts.setup_forge_path import setup_forge_path
setup_forge_path()
```

### GHDL Compilation Errors

**Problem:** VHDL compilation fails

**Solution:**
1. Check compilation order in `test_configs.py`
2. Ensure packages are compiled before entities
3. Verify VHDL standard is set correctly (`--std=08`)

### Test Discovery Issues

**Problem:** Pytest finds no tests

**Solution:**
1. Check that test files are named `test_*.py`
2. Verify they're in `libs/` or `probes/` directories
3. Check `pyproject.toml` for excluded paths

## Contributing

When making changes:

1. **Choose appropriate context** - Use slash commands to load relevant context
2. **Follow domain guidelines** - VHDL, Python, or Testing patterns
3. **Write tests** - All new features need CocotB tests
4. **Update documentation** - Keep README and `.claude/` docs current
5. **Run tests** - `pytest` must pass before committing

## License

MIT License - See LICENSE file

## Related Projects

- [moku-instrument-forge](https://github.com/sealablab/moku-instrument-forge) - Code generation framework
- [moku-instrument-forge-vhdl](https://github.com/sealablab/moku-instrument-forge-vhdl) - Shared VHDL utilities
- [moku-models](https://github.com/sealablab/moku-models) - Platform data models
- [basic-app-datatypes](https://github.com/sealablab/basic-app-datatypes) - Register package types
- [riscure-models](https://github.com/sealablab/riscure-models) - Riscure probe specifications

## Questions?

- **Documentation:** See `.claude/` directory for detailed guides
- **Issues:** Open an issue on GitHub
- **Forge docs:** See `forge/README.md`
- **VHDL utils:** See `libs/forge-vhdl/README.md`
