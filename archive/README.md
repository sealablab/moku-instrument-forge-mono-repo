# Archive Directory

This directory contains **reference implementations** from the EZ-EMFI project.

## Status: ⚠️ REFERENCE ONLY

These files are **archived for reference** and are **not actively maintained**. They represent earlier implementations before the monorepo migration and forge-based architecture.

## What's Archived

### ez-emfi-probes/

Contains probe driver implementations from the EZ-EMFI project:

- **DS1120_PD/** - DS1120A EMFI probe driver (volo-based)
- **DS1140_PD/** - DS1140A EMFI probe driver (volo-based)

Each probe directory contains:
- `vhdl/` - VHDL source files (shim, main, packages, FSM)
- `tests/` - CocotB progressive tests
- `docs/` - Project documentation (requirements, summaries, testing guides)
- `*.yaml` - Probe configuration files

## Why Archived?

The monorepo is transitioning to a **forge-based architecture** where:

1. **Code generation** - Forge generates interface code from YAML specifications
2. **Modular VHDL** - Shared utilities in `libs/forge-vhdl/` (git submodule)
3. **Clean separation** - Generated code vs custom logic clearly separated
4. **Better testing** - Centralized pytest infrastructure

The archived implementations served as:
- Proof-of-concept for DS1120A and DS1140A drivers
- Learning platform for VHDL/Moku development
- Reference for FSM design patterns

## Using Archived Code

**These files should NOT be copied directly into the monorepo.**

Instead, use them as **reference** when:
- Designing new probe FSMs
- Understanding probe-specific timing requirements
- Learning from past implementation patterns
- Troubleshooting hardware issues

## Migration Path (If Needed)

If you need to bring these probes into the monorepo:

1. **Extract requirements** from archived docs
2. **Create forge YAML** for probe specification
3. **Generate interface code** using forge
4. **Implement custom FSM** in `probes/*/vhdl/`
5. **Port tests** to new structure in `probes/*/tests/`
6. **Validate** with CocotB + pytest

See main `README.md` for monorepo workflows.

## Source

**Original project:** EZ-EMFI (private repository)
**Archived:** 2025-11-03
**Architecture validated in:** [moku-spike-redux](https://github.com/sealablab/moku-spike-redux)

---

**Questions?** See main repository `README.md` or `.claude/` documentation.
