# Probe Status

Show development status of all probes in the monorepo.

**Agent:** probe-design-orchestrator

---

## Usage

```
/probe-status
```

No arguments required - scans entire `probes/` directory.

## What It Shows

For each probe discovered:
1. **YAML Status** - Valid, invalid, or missing
2. **Package Status** - Generated, stale, or missing
3. **Deployment Status** - Deployed device IP or not deployed

## Example Output

```
Probe Development Status
========================

DS1140_PD
  ✅ YAML: Valid (probes/DS1140_PD/specs/DS1140_PD.yaml)
  ✅ Package: Generated (forge/apps/DS1140_PD/, 2025-11-03 14:30)
  ✅ Deployed: 192.168.1.100 (Slot 2)

DS1180_LASER
  ⚠️  YAML: Not validated
  ✅ Package: Generated (forge/apps/DS1180_LASER/, 2025-11-02 10:15)
  ❌ Deployed: No

DS1200_EM
  ✅ YAML: Valid (probes/DS1200_EM/specs/DS1200_EM.yaml)
  ⚠️  Package: Stale (YAML modified after generation)
  ❌ Deployed: No

DS1220_OPTICAL
  ⚠️  YAML: Validation errors (2 errors found)
  ❌ Package: Not generated
  ❌ Deployed: No

---
Total: 4 probes
Ready: 1 (DS1140_PD)
Need attention: 3
```

## Status Indicators

### YAML Status

**✅ Valid**
- YAML file exists
- Recently validated by forge-context
- No schema errors

**⚠️ Not validated**
- YAML file exists
- Not validated recently
- May have errors

**⚠️ Validation errors**
- YAML file exists but has errors
- Shows error count
- Need to fix and re-validate

**❌ Missing**
- No YAML file found in specs/
- Need to create specification

### Package Status

**✅ Generated**
- Package exists in forge/apps/<probe_name>/
- manifest.json and VHDL files present
- Up-to-date with YAML

**⚠️ Stale**
- Package exists but YAML modified after generation
- Need to regenerate package

**❌ Not generated**
- No package directory found
- Need to run `/generate`

### Deployment Status

**✅ Deployed**
- Device IP shown
- Slot number shown (if known)
- Last deployment timestamp

**❌ Not deployed**
- No active deployment found
- Need to run `/deploy`

## Checks Performed

1. **Scan probes/ directory** for subdirectories
2. **For each probe:**
   - Check `probes/<probe_name>/specs/<probe_name>.yaml` exists
   - Read YAML (if exists) and basic syntax check
   - Check `forge/apps/<probe_name>/` exists
   - Compare timestamps (YAML vs package)
   - Check deployment cache (if maintained)

## Use Cases

### Project Overview
Run at start of work session to see all probe states.

### Identify Stale Probes
Find probes where YAML changed but package not regenerated.

### Multi-Probe Development
Track status when working on multiple probes simultaneously.

### Onboarding
New team members can see all available probes.

## Next Actions by Status

### Probe with validation errors
```
/validate probes/<probe_name>/specs/<probe_name>.yaml
```
Fix errors, re-validate.

### Probe with stale package
```
/generate probes/<probe_name>/specs/<probe_name>.yaml --force
```

### Probe not deployed
```
/deploy <probe_name> --device <ip>
```

### Probe ready
No action needed, or monitor with:
```
/monitor-state <probe_name>
```

## Notes

- Deployment status requires deployment tracking (may be limited)
- YAML validation status cached from recent `/validate` calls
- Package staleness determined by file timestamps
- Use this before starting work to prioritize tasks

## Example Session

```
User: What probes need attention?

Agent: /probe-status

Output shows:
- DS1140_PD: All green, ready
- DS1180_LASER: Package stale, need regeneration
- DS1200_EM: Not deployed
- DS1220_OPTICAL: YAML errors

User: Fix DS1220_OPTICAL first

Agent: /validate probes/DS1220_OPTICAL/specs/DS1220_OPTICAL.yaml
[Shows errors]

User: [Fixes YAML]

Agent: /validate probes/DS1220_OPTICAL/specs/DS1220_OPTICAL.yaml
✅ Now valid

Agent: /probe-status
[Shows DS1220_OPTICAL now has valid YAML, ready to generate]
```
