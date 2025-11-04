# Validate Probe Structure

Check that probe directory follows expected structure.

**Agent:** probe-design-orchestrator

---

## Usage

```
/validate-probe-structure <probe_name>
```

**Example:**
```
/validate-probe-structure DS1140_PD
```

## What It Checks

### Required Directories
- [ ] `probes/<probe_name>/` exists
- [ ] `probes/<probe_name>/specs/` exists
- [ ] `probes/<probe_name>/vhdl/` exists
- [ ] `probes/<probe_name>/docs/` exists

### Required Files
- [ ] `probes/<probe_name>/specs/<probe_name>.yaml` exists
- [ ] `probes/<probe_name>/docs/README.md` exists

### Optional (won't fail validation)
- `probes/<probe_name>/tests/` - Test benches
- Additional VHDL files in `vhdl/`
- Additional docs in `docs/`

### Package Consistency (if package exists)
- [ ] `forge/apps/<probe_name>/` exists (if YAML has been generated)
- [ ] Package `app_name` matches probe name
- [ ] Platform matches across YAML and manifest.json

## Example Output - Valid Structure

```
✅ Probe structure validation: PASSED

Probe: DS1140_PD
Location: probes/DS1140_PD/

Required directories:
  ✅ specs/ - Found
  ✅ vhdl/ - Found
  ✅ docs/ - Found

Required files:
  ✅ specs/DS1140_PD.yaml - Found (256 lines)
  ✅ docs/README.md - Found (42 lines)

Optional:
  ✅ tests/ - Found
  ✅ vhdl/*.vhd - Found (2 files)

Package consistency:
  ✅ forge/apps/DS1140_PD/ - Found
  ✅ app_name matches: DS1140_PD
  ✅ Platform matches: moku_go

No issues found.
```

## Example Output - Issues Found

```
❌ Probe structure validation: FAILED

Probe: DS1180_LASER
Location: probes/DS1180_LASER/

Required directories:
  ✅ specs/ - Found
  ✅ vhdl/ - Found
  ❌ docs/ - MISSING

Required files:
  ✅ specs/DS1180_LASER.yaml - Found (189 lines)
  ❌ docs/README.md - MISSING

Optional:
  ⚠️  tests/ - Not found (optional)
  ⚠️  vhdl/*.vhd - No VHDL files found

Package consistency:
  ✅ forge/apps/DS1180_LASER/ - Found
  ⚠️  app_name mismatch: YAML says 'DS1180_Laser', expected 'DS1180_LASER'

---
Issues found: 3
Warnings: 3
```

## Fix Suggestions

### Missing directories
```bash
mkdir -p probes/<probe_name>/docs
mkdir -p probes/<probe_name>/tests
```

### Missing README
Use template from `/init-probe` command or create basic README:
```markdown
# <probe_name> Probe

[Description]

## Signals
[List signals from YAML]

## Usage
[Basic usage instructions]
```

### app_name mismatch
Edit YAML file to match directory name:
```yaml
app_name: DS1180_LASER  # Must match directory name exactly
```

### No VHDL files
This is expected before implementation. After package generation:
1. Review `forge/apps/<probe_name>/*_main.vhd` template
2. Create custom VHDL in `probes/<probe_name>/vhdl/`

## When to Use

### Before Starting Work
Verify probe structure is correct before beginning development.

### After Manual Changes
If you created directories/files manually, verify structure.

### Before Package Generation
Ensure structure is correct before running `/generate`.

### Troubleshooting Build Issues
Structural issues can cause generation or deployment failures.

### Code Review
Verify probe follows standard structure before committing.

## Integration with Other Commands

**After `/init-probe`:**
```
/init-probe DS1220_OPTICAL
/validate-probe-structure DS1220_OPTICAL
✅ Should pass (template creates correct structure)
```

**Before `/generate`:**
```
/validate-probe-structure DS1180_LASER
[Fix any issues]
/validate probes/DS1180_LASER/specs/DS1180_LASER.yaml
/generate probes/DS1180_LASER/specs/DS1180_LASER.yaml
```

**Part of `/probe-status`:**
Structure validation is part of overall probe status check.

## Notes

- This validates STRUCTURE only, not YAML content (use `/validate` for that)
- Package consistency checks require package to exist (optional)
- Warnings don't fail validation, only errors do
- Use `/init-probe` to create correct structure from template

## Expected Directory Structure

```
probes/<probe_name>/
├── specs/                          # Required
│   └── <probe_name>.yaml           # Required (exact name match)
├── vhdl/                           # Required (can be empty initially)
│   ├── <probe_name>_logic.vhd      # Custom VHDL (user creates)
│   └── <probe_name>_fsm.vhd        # Additional VHDL (optional)
├── docs/                           # Required
│   ├── README.md                   # Required
│   └── diagrams/                   # Optional
│       └── fsm.png
└── tests/                          # Optional
    └── <probe_name>_tb.vhd         # Test benches
```

## Validation Levels

### Level 1: Basic Structure (Required)
- Directories exist (specs/, vhdl/, docs/)
- Required files exist (YAML, README)

### Level 2: Consistency (Important)
- app_name matches directory name
- Package exists and matches
- No conflicting metadata

### Level 3: Completeness (Nice-to-have)
- VHDL files present
- Tests exist
- Comprehensive documentation

This command checks all three levels and reports issues/warnings accordingly.
