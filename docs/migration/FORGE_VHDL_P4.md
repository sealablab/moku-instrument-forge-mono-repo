# Phase 4: Voltage Type System Implementation

**Status:** ✅ COMPLETE (2025-11-04)
**Completed by:** Phase 4 commit (6e1e99d)
**Duration:** ~4-6 hours (actual)
**Prerequisites:** Phase 3 complete (standards integrated) ✅
**Goal:** Implement function-based voltage type system with VHDL packages and Python mirror classes

**Completion Notes:**
- ✅ Design finalized (see VOLTAGE_TYPE_SYSTEM_DESIGN.md)
- ✅ voltage_types_reference.py created (Python implementation)
- ✅ CocoTB interface rules documented (Section 4 in CLAUDE.md)
- ✅ VHDL packages implemented (all 3 domains):
  - vhdl/packages/forge_voltage_3v3_pkg.vhd (10KB)
  - vhdl/packages/forge_voltage_5v0_pkg.vhd (10KB)
  - vhdl/packages/forge_voltage_5v_bipolar_pkg.vhd (10KB)
- ✅ Test wrappers created for all 3 voltage domains:
  - forge_voltage_3v3_pkg_tb_wrapper.vhd
  - forge_voltage_5v0_pkg_tb_wrapper.vhd
  - forge_voltage_5v_bipolar_pkg_tb_wrapper.vhd
- ✅ CocoTB progressive tests implemented (all 3 domains):
  - test_forge_voltage_3v3_pkg_progressive.py (P1: 4 tests, P2: 6 tests)
  - test_forge_voltage_5v0_pkg_progressive.py (P1: 4 tests, P2: 6 tests)
  - test_forge_voltage_5v_bipolar_pkg_progressive.py (P1: 4 tests, P2: 6 tests)
- ✅ Tests registered in test_configs.py
- ✅ All tests passing
- ✅ Documentation updated (CLAUDE.md, llms.txt)
- ⚠️ Python placement: Deferred (voltage_types_reference.py remains in docs/migration)

---

## Executive Summary: Finalized Design

### Approach: Function-Based Type Safety (Verilog Compatible)

**Key Decision:** Use **explicit package selection** instead of VHDL language types (records/physical) to maintain Verilog compatibility per forge-vhdl coding standards.

### Three Voltage Domain Packages

1. **forge_voltage_3v3_pkg** - 0-3.3V unipolar (TTL/digital logic)
2. **forge_voltage_5v0_pkg** - 0-5.0V unipolar (sensor supply)
3. **forge_voltage_5v_bipolar_pkg** - ±5.0V bipolar (Moku DAC/ADC, most analog)

### Implementation Pattern

**VHDL:**
```vhdl
use work.forge_voltage_3v3_pkg.all;  -- Explicit domain selection

signal trigger : real := 2.5;
signal trigger_digital : signed(15 downto 0);

trigger_digital <= to_digital(trigger);  -- Function-based conversion
```

**Python:**
```python
from voltage_types import Voltage_3V3  # Type-safe class

trigger = Voltage_3V3(2.5)  # Range validated
trigger_digital = trigger.to_digital()  # Explicit conversion
```

### Why This Design?

**✅ Advantages:**
- Verilog compatible (follows forge-vhdl standards)
- Explicit package selection (self-documenting)
- Production-proven (matches existing volo_voltage_pkg pattern)
- Python type checking (mypy catches domain mismatches)
- Arithmetic prevention (units stay attached)

**⚠️ Trade-off:**
- No VHDL compile-time type safety (accepted for Verilog compatibility)
- 80% type safety (explicit packages + runtime validation)

### Deliverables

1. Three VHDL packages (`forge_voltage_*_pkg.vhd`)
2. Python reference implementation (`voltage_types_reference.py`)
3. CocoTB tests for all packages
4. Documentation updates (forge-vhdl llms.txt, CLAUDE.md)
5. Python placement decision (defer or implement in forge/libs/)

---

## Problem Statement (Original Design Exploration)

### Current State: Type-Unsafe Voltages
```vhdl
-- Dangerous: No protection against voltage domain mismatches
signal trigger_level : signed(15 downto 0);  -- What range? Unknown!
signal dac_output : signed(15 downto 0);     -- What range? Unknown!

trigger_level <= dac_output;  -- Compiles! But is ±5V going to 0-3.3V TTL? UNSAFE!
```

### Desired State: Type-Safe Voltages
```vhdl
-- Safe: Physical types prevent invalid assignments
signal trigger_level : voltage_3v3;          -- Explicitly 0-3.3V
signal dac_output : voltage_5v_bipolar;      -- Explicitly ±5V

trigger_level <= dac_output;  -- COMPILE ERROR: Type mismatch!
trigger_level <= scale_5v_bipolar_to_3v3(dac_output);  -- Explicit, auditable
```

**Key Principle:** Units must not detach from type (no naked `real` voltages).

---

## Design Options (To Be Decided)

### Option 1: VHDL Physical Types (True Type Safety)
```vhdl
type voltage_3v3 is range 0 to 3_300_000 units
    uV;  -- Base unit: microvolt
    mV = 1000 uV;
    V = 1000 mV;
end units;

type voltage_5v_bipolar is range -5_000_000 to 5_000_000 units
    uV;
    mV = 1000 uV;
    V = 1000 mV;
end units;

signal trigger : voltage_3v3 := 2.5 V;  -- Compile-time type safe!
```

**Pros:**
- True VHDL compile-time safety
- Self-documenting
- Can't accidentally mix domains

**Cons:**
- Cannot directly assign to `signed(15 downto 0)` (needs conversion)
- More verbose
- Requires conversion functions for register interface

---

### Option 2: Record Types (Recommended by AI)
```vhdl
type voltage_3v3 is record
    digital : signed(15 downto 0);  -- Digital representation
    -- v_min=0.0, v_max=3.3 implicit
end record;

type voltage_5v_bipolar is record
    digital : signed(15 downto 0);
    -- v_min=-5.0, v_max=5.0 implicit
end record;

signal trigger : voltage_3v3;
signal dac : voltage_5v_bipolar;

trigger <= dac;  -- COMPILE ERROR: Type mismatch
trigger <= scale_5v_bipolar_to_3v3(dac);  -- Explicit conversion
```

**Pros:**
- Compile-time type safety
- Works with existing `signed` infrastructure
- Can extend with metadata (valid flags, etc.)

**Cons:**
- Requires `.digital` accessor for registers
- Slightly more verbose than raw signed

---

### Option 3: Subtypes (Runtime Safety Only)
```vhdl
subtype voltage_3v3_raw is signed(15 downto 0) range 0 to 21627;
subtype voltage_5v_bipolar_raw is signed(15 downto 0);

signal trigger : voltage_3v3_raw;
signal dac : voltage_5v_bipolar_raw;

trigger <= dac;  -- Compiles, but simulation ERROR if out of range
```

**Pros:**
- Minimal changes to existing code
- Works with signed directly

**Cons:**
- **No compile-time safety** (just simulation assertions)
- Easy to bypass with casts

---

## Python Mirror Requirement

**Critical:** Python side MUST mirror VHDL type safety.

```python
class Voltage_3V3:
    """0-3.3V voltage type (mirrors VHDL voltage_3v3)"""
    V_MIN = 0.0
    V_MAX = 3.3

    def __init__(self, volts: float):
        if not (self.V_MIN <= volts <= self.V_MAX):
            raise ValueError(f"{volts}V out of range [{self.V_MIN}, {self.V_MAX}]V")
        self._volts = volts

    @property
    def volts(self) -> float:
        return self._volts  # Read-only to prevent detachment

    def to_digital(self) -> int:
        """Convert to 16-bit signed for register writes"""
        normalized = self._volts / self.V_MAX
        return int(normalized * 32767)

    # CRITICAL: Prevent arithmetic that loses type
    def __add__(self, other):
        raise TypeError("Cannot add voltages (units would detach)")

    # Explicit conversions only
    def scale_to_5v_bipolar(self) -> 'Voltage_5V_Bipolar':
        """Explicitly scale 0-3.3V to ±5V centered at 0V"""
        scaled = (self._volts - 1.65) * (5.0 / 1.65)
        return Voltage_5V_Bipolar(scaled)


# Usage
trigger = Voltage_3V3(2.5)  # Type-safe, range-checked
dac = Voltage_5V_Bipolar(-3.0)

trigger = dac  # TypeError: Type mismatch!
trigger = dac.scale_to_3v3()  # Explicit, auditable

# Register writes preserve type
registers['trigger'] = trigger.to_digital()  # Explicit int conversion
```

**Why this matters:**
- Prevents `trigger = 2.5` (naked float - what units?)
- Forces `trigger = Voltage_3V3(2.5)` (units embedded in type)
- Cross-domain operations require explicit methods
- Auditable conversions in code

---

## Voltage Ranges (Pragmatic 3-Range System)

Based on discussion: **99% of real FPGA work uses 3 ranges**

### 1. TTL/Digital Logic (0-3.3V)
```vhdl
type voltage_3v3  -- or voltage_ttl
-- Use for: Digital glitch, probe triggers, GPIO, TTL interfaces
-- Digital range: 0 to 21627 (0x0000 to 0x54EB)
```

### 2. Unipolar Supply (0-5.0V)
```vhdl
type voltage_5v0  -- or voltage_unipolar
-- Use for: DAC outputs (unipolar), sensor power
-- Digital range: 0 to 32767 (0x0000 to 0x7FFF)
```

### 3. Bipolar Signal (±5.0V)
```vhdl
type voltage_5v_bipolar  -- or voltage_bipolar
-- Use for: Moku DAC/ADC, analog signals, AC waveforms
-- Digital range: -32768 to 32767 (0x8000 to 0x7FFF)
```

**Note:** forge/basic-app-datatypes has 12 voltage types (±20V, ±25V, etc.) for comprehensive serialization. This is **separate domain** - both coexist.

---

## Design Decisions (Finalized)

### ✅ Decision 1: VHDL Approach
**Selected:** Function-based packages (Verilog compatible)
**Rejected:** Physical types, record types (violate forge-vhdl coding standards)

### ✅ Decision 2: Python Implementation
**Selected:** Rich classes with type enforcement, arithmetic prevention
**Implemented:** See `voltage_types_reference.py` (reference implementation)

### ✅ Decision 3: Number of Packages
**Selected:** 3 explicit packages (3.3V, 5V, ±5V)
**Rationale:** Covers 99% of real work, explicit > generic

### ✅ Decision 4: Integration Strategy
**Selected:** Parallel existence (opt-in for new code)
**Rationale:** No breaking changes, gradual migration

### ✅ Decision 5: Relationship with basic-app-datatypes
**Selected:** Stay independent, cross-document
- basic-app-datatypes = serialization (12 types)
- voltage-types = runtime safety (3 types)
- Different domains, both coexist

---

## Implementation Checklist

### Phase 4a: VHDL Packages (2-3 hours)

**In libs/forge-vhdl submodule:**

- [ ] Create `vhdl/packages/forge_voltage_3v3_pkg.vhd`
  - [ ] Constants (V_MIN, V_MAX, SCALE_FACTOR)
  - [ ] Common reference voltages (1V, 2.5V, 3.3V digital values)
  - [ ] `to_digital()` function
  - [ ] `from_digital()` function
  - [ ] `clamp()` function
  - [ ] `is_valid()` function
  - [ ] `is_voltage_equal()` testbench helper

- [ ] Create `vhdl/packages/forge_voltage_5v0_pkg.vhd` (same structure)

- [ ] Create `vhdl/packages/forge_voltage_5v_bipolar_pkg.vhd` (same structure)

- [ ] Commit in submodule with descriptive message
- [ ] Update parent monorepo reference

### Phase 4b: CocoTB Tests (2-3 hours)

**In libs/forge-vhdl/tests/:**

- [ ] Create test wrappers (packages need wrapper entities)
  - [ ] `forge_voltage_3v3_pkg_tb_wrapper.vhd`
  - [ ] `forge_voltage_5v0_pkg_tb_wrapper.vhd`
  - [ ] `forge_voltage_5v_bipolar_pkg_tb_wrapper.vhd`

- [ ] Create progressive tests
  - [ ] `test_forge_voltage_3v3_pkg_progressive.py`
    - [ ] P1: Conversion accuracy (3-4 tests)
    - [ ] P2: Edge cases, clamping (5-7 tests)
  - [ ] Similar for 5v0 and 5v_bipolar

- [ ] Update `test_configs.py` to register tests

- [ ] Validate P1 output <20 lines

- [ ] Commit in submodule

### Phase 4c: Documentation Updates (30 mins)

- [x] Update `libs/forge-vhdl/llms.txt` (done)
- [x] Update `libs/forge-vhdl/CLAUDE.md` (done)
- [x] Update monorepo `llms.txt` (done)
- [x] Create `VOLTAGE_TYPE_SYSTEM_DESIGN.md` (done)
- [x] Create `voltage_types_reference.py` (done)

### Phase 4d: Python Placement (defer or 1 hour)

**Option A:** Keep as reference in docs/ (current state)
**Option B:** Create `forge/libs/voltage-types/` submodule
**Option C:** Place in `libs/forge-vhdl/python/`

**Recommendation:** Defer decision, keep reference implementation for now

---

## Key Insights from Discussion

### 1. Units Must Not Detach from Type
```python
# BAD: Unit-less float
voltage = 2.5  # Volts? Millivolts? What range?

# GOOD: Type carries units
voltage = Voltage_3V3(2.5)  # Unambiguously 0-3.3V range
```

### 2. Explicit Conversions Required
```vhdl
-- Cannot implicitly convert between domains
trigger_3v3 <= dac_5v_bipolar;  -- COMPILE ERROR

-- Must explicitly handle scaling
trigger_3v3 <= scale_5v_to_3v3(dac_5v_bipolar);  -- Auditable
```

### 3. forge/ vs forge-vhdl Separation
- **forge/**: Comprehensive (12 voltage types) for YAML → register serialization
- **forge-vhdl**: Pragmatic (3 voltage types) for hand-written VHDL safety
- **Both coexist:** Different use cases, cross-documented

---

## Existing Code to Review

### From EZ-EMFI Export:
- `/tmp/vhdl_coding_standards_export/VHDL/packages/volo_voltage_pkg.vhd` (327 lines)
  - Hardcoded ±5V
  - Functions: `voltage_to_digital()`, `is_voltage_equal()`, `clamp_voltage()`
  - Could be template for new type system

### From forge-vhdl Current:
- `libs/forge-vhdl/vhdl/packages/volo_voltage_pkg.vhd`
  - Same as export (identical)

---

## Proposed Phase 4 Scope (TBD)

### Minimal Scope:
1. Design voltage type system (choose VHDL approach)
2. Implement 3 voltage types in VHDL
3. Create Python mirror classes
4. Write CocoTB tests for type safety
5. Document in CLAUDE.md

### Extended Scope (Optional):
6. Conversion functions between types
7. Integration examples with fsm_observer
8. Update existing components to use types
9. forge/ template updates

---

## Next Steps Before Executing Phase 4

1. **User decides:** VHDL type approach (physical vs record vs subtype)
2. **User decides:** Python implementation scope
3. **User decides:** Integration strategy
4. **Review:** Existing `volo_voltage_pkg.vhd` for patterns to preserve
5. **Design:** Conversion function API
6. **Write:** Detailed Phase 4 execution plan

---

## Context Preservation Notes

**From earlier discussion:**
- You wanted **physical types** to prevent unit detachment
- Python must mirror VHDL for consistency
- 3 pragmatic ranges (3.3V, 5V, ±5V) cover 99% of real work
- forge/ 12 types are for comprehensive serialization (separate domain)
- Old `volo_voltage_pkg` had "faux type safety" - we want **real** safety

**EZ-EMFI lessons learned:**
- FSM states must be `std_logic_vector` (Verilog compat)
- Signal naming prefixes (`ctrl_`, `cfg_`) prevent ambiguity
- Port order standardization helps readability
- Reset hierarchy (rst_n > clk_en > enable) is proven

**VHDL Coding Standards integrated in Phase 3:**
- Mandatory rules now documented in forge-vhdl
- All future components follow standards
- Templates will enforce conventions

---

## References for Future Session

When picking up Phase 4 design:

1. **Read:** `/tmp/cocotb_progressive_export/VHDL/packages/volo_voltage_pkg.vhd`
   - Understand existing conversion functions
   - Identify patterns to preserve

2. **Read:** `/tmp/vhdl_coding_standards_export/VHDL_CODING_STANDARDS.md`
   - Section on voltage constants
   - `clamp_voltage()` function pattern

3. **Review:** `forge/libs/basic-app-datatypes/CLAUDE.md`
   - 12 voltage type rationale
   - Relationship with forge-vhdl types

4. **Decide:** VHDL type approach (Options 1-3 above)

5. **Design:** Python class hierarchy

6. **Plan:** CocoTB test strategy for type safety validation

---

**Status:** Design draft complete
**Next:** User reviews, makes key decisions, then Phase 4 execution plan can be written

**Time estimate:** Design decisions + detailed plan = 1 hour
**Execution:** TBD (depends on scope)
