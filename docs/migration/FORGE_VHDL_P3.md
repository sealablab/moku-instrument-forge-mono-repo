# Phase 3: VHDL Coding Standards Integration

**Status:** ✅ COMPLETE (2025-11-04)
**Completed by:** v2.0.0 release
**Duration:** ~1 hour (actual)
**Prerequisites:** Phase 2 complete (components migrated with tests) ✅
**Goal:** Integrate proven VHDL coding standards from EZ-EMFI into forge-vhdl

**Completion Notes:**
- ✅ VHDL_CODING_STANDARDS.md installed (601 lines)
- ✅ VHDL_QUICK_REF.md created and integrated into CLAUDE.md
- ✅ All VOLO references removed from active docs
- ✅ Standards applied to voltage package test wrappers
- ✅ Port order, FSM patterns, and naming conventions documented

---

## Overview

Migrate **VHDL_CODING_STANDARDS.md** (~600 lines) from EZ-EMFI export into `libs/forge-vhdl/docs/`, updating VOLO references to forge naming convention.

**Source:** `/tmp/vhdl_coding_standards_export/VHDL_CODING_STANDARDS.md`

---

## Core Standards to Adopt

### 1. FSM Design (Critical - Verilog Compatibility)
```vhdl
-- ✅ MANDATORY: Use std_logic_vector (NOT enums)
constant STATE_IDLE : std_logic_vector(1 downto 0) := "00";
signal state : std_logic_vector(1 downto 0);

-- ❌ FORBIDDEN: Enums don't translate to Verilog
type state_t is (IDLE, ARMED);  -- DO NOT USE
```

### 2. Port Declaration Order (Mandatory)
```vhdl
entity forge_util_example is
    port (
        -- 1. Clock & Reset
        clk    : in std_logic;
        rst_n  : in std_logic;  -- Active-low

        -- 2. Control
        clk_en : in std_logic;
        enable : in std_logic;

        -- 3. Data inputs
        data_in : in std_logic_vector(15 downto 0);

        -- 4. Data outputs
        data_out : out std_logic_vector(15 downto 0);

        -- 5. Status
        busy : out std_logic
    );
end entity;
```

### 3. Signal Naming (Universal Prefixes)
| Prefix | Purpose | Example |
|--------|---------|---------|
| `ctrl_` | Control signals | `ctrl_enable`, `ctrl_arm` |
| `cfg_` | Configuration | `cfg_threshold`, `cfg_mode` |
| `stat_` | Status outputs | `stat_busy`, `stat_fault` |
| `dbg_` | Debug outputs | `dbg_state_voltage` |
| `_n` | Active-low | `rst_n`, `enable_n` |
| `_next` | Next-state | `state_next` |

**Decision:** Apply universally to ALL modules (utilities + apps) for consistency.

### 4. Reset & Enable Hierarchy
```vhdl
process(clk, rst_n)
begin
    if rst_n = '0' then
        -- 1. RESET (highest priority)
        output <= (others => '0');
    elsif rising_edge(clk) then
        if clk_en = '1' then
            -- 2. CLK_EN (clock gating)
            if enable = '1' then
                -- 3. ENABLE (functional)
                output <= input;
            end if;
        end if;
    end if;
end process;
```

---

## Execution Steps

### Step 1: Copy Standards Document
```bash
cd libs/forge-vhdl
git checkout 20251104-vhdl-forge-dev

# Copy main standards doc
cp /tmp/vhdl_coding_standards_export/VHDL_CODING_STANDARDS.md docs/

# Verify
ls docs/VHDL_CODING_STANDARDS.md
```

### Step 2: Update VOLO References
Edit `docs/VHDL_CODING_STANDARDS.md`:

**Global replacements:**
- `volo_common_pkg` → `forge_common_pkg`
- `volo_voltage_pkg` → `forge_voltage_pkg`
- `volo_clk_divider` → `forge_util_clk_divider`
- `VOLO_READY` → `FORGE_READY`

**Keep:** Core standards (FSM, ports, naming, reset hierarchy)

### Step 3: Create Quick Reference Card
Create `docs/VHDL_QUICK_REF.md` (extracted from standards):

```markdown
# VHDL Quick Reference

## Port Order Template
1. clk, rst_n
2. clk_en, enable
3. inputs
4. outputs
5. status

## FSM Template
```vhdl
constant STATE_IDLE : std_logic_vector(1 downto 0) := "00";
signal state : std_logic_vector(1 downto 0);
```

## Signal Prefixes
- ctrl_* = Control
- cfg_* = Config
- stat_* = Status
- dbg_* = Debug

## Process Template
```vhdl
process(clk, rst_n)
begin
    if rst_n = '0' then
        -- reset
    elsif rising_edge(clk) then
        if clk_en = '1' and enable = '1' then
            -- logic
        end if;
    end if;
end process;
```
```

### Step 4: Update CLAUDE.md
Add standards section to `libs/forge-vhdl/CLAUDE.md`:

```markdown
## VHDL Coding Standards

### Mandatory Rules

**FSM States:** Use `std_logic_vector`, not enums (Verilog compatibility)
**Port Order:** clk, rst_n, clk_en, enable, data, status
**Signal Naming:** Universal prefixes (`ctrl_`, `cfg_`, `stat_`, `dbg_`)
**Reset Hierarchy:** rst_n > clk_en > enable

**See:** docs/VHDL_CODING_STANDARDS.md for complete rules
**Quick Ref:** docs/VHDL_QUICK_REF.md
```

### Step 5: Commit in Submodule
```bash
# Still in libs/forge-vhdl
git add docs/VHDL_CODING_STANDARDS.md
git add docs/VHDL_QUICK_REF.md
git add CLAUDE.md

git commit -m "$(cat <<'EOF'
Add VHDL coding standards from EZ-EMFI project

Integrated proven VHDL design patterns and coding standards:
- FSM design rules (std_logic_vector, not enums - Verilog compat)
- Port declaration order (clk, rst_n, clk_en, enable, data, status)
- Signal naming conventions (ctrl_*, cfg_*, stat_*, dbg_* prefixes)
- Reset & enable hierarchy (rst_n > clk_en > enable)
- Synthesis guidelines
- Common anti-patterns to avoid

Files added:
- docs/VHDL_CODING_STANDARDS.md (600 lines, comprehensive guide)
- docs/VHDL_QUICK_REF.md (quick reference card for developers)

Updated:
- CLAUDE.md (added standards section with pointers)

Changes from EZ-EMFI original:
- Updated VOLO references → forge naming convention
- volo_common_pkg → forge_common_pkg
- volo_voltage_pkg → forge_voltage_pkg
- volo_clk_divider → forge_util_clk_divider

These standards are MANDATORY for all new forge-vhdl components.

Source: EZ-EMFI vhdl_coding_standards_export
Related: docs/migration/FORGE_VHDL_P3.md
EOF
)"

git push origin 20251104-vhdl-forge-dev
```

### Step 6: Update Parent Monorepo
```bash
cd ../..
git add libs/forge-vhdl
git commit -m "chore: Update forge-vhdl submodule (VHDL coding standards)"
git push origin 20251104-vhdl-forge-dev
```

---

## Integration with forge/ Code Generator

**Note:** forge/ generates `*_main.vhd` templates for users.

**Action for future:** Update forge templates to follow these standards:
- Port order matches standard
- Signal names use prefixes (`ctrl_arm`, `cfg_threshold`)
- FSM states use `std_logic_vector`

**Not in Phase 3 scope** (defer to forge/ repo updates)

---

## Checklist

- [ ] VHDL_CODING_STANDARDS.md copied to docs/
- [ ] VOLO references updated to forge
- [ ] VHDL_QUICK_REF.md created
- [ ] CLAUDE.md updated with standards pointer
- [ ] Committed in submodule
- [ ] Parent reference updated
- [ ] Standards reviewed and approved

---

## Success Criteria

✅ Standards document in `libs/forge-vhdl/docs/`
✅ All VOLO references updated to forge
✅ Quick reference card available
✅ CLAUDE.md points to standards
✅ Future components will follow these rules

---

## Next Steps

After Phase 3:
- **Phase 4:** Voltage type system design (physical types with unit safety)
- **Phase 5:** Apply standards to existing components (refactor if needed)
- **Phase 6:** Update forge/ templates to match standards

---

**Time estimate:** 1 hour
**Complexity:** Low (mostly copy + find/replace)
**Impact:** High (establishes design consistency for all future work)
