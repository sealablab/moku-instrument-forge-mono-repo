# Voltage Type System Design

**Version:** 1.0
**Date:** 2025-11-04
**Status:** Design finalized, ready for implementation
**Scope:** Cross-language (VHDL + Python) voltage domain type safety

---

## Executive Summary

This document captures the **intentional design decisions** for a voltage type system that provides:

1. **Clear unit specification** - No naked `float` voltages in Python or `real` in VHDL
2. **Type-checking at assignment** - Catch domain mismatches at development time
3. **Verilog compatibility** - Follows forge-vhdl coding standards
4. **1:1 VHDL-Python mapping** - Consistent behavior across languages
5. **Minimal implementation** - Not semantically rich, just domain enforcement

**Key Innovation:** Function-based type safety (instead of language type systems) for Verilog compatibility, with explicit package selection enforcing domain boundaries.

---

## The Problem

### Before: Type-Unsafe Voltages

**VHDL:**
```vhdl
signal trigger_level : real;  -- What range? Unknown!
signal dac_output : real;     -- What range? Unknown!

trigger_level <= dac_output;  -- Compiles! But is ±5V going to 0-3.3V TTL? UNSAFE!
```

**Python:**
```python
trigger = 2.5  # Volts? Millivolts? What range?
dac = -3.0     # What domain?

registers['trigger'] = int(trigger * 6553)  # What scale factor? Guess!
```

**Problems:**
- Units detach from values (naked floats)
- No compile-time domain checking
- Easy to mix voltage ranges accidentally
- Scale factors scattered throughout code

---

## Design Constraints

### Constraint 1: Verilog Compatibility (Hard Requirement)

From `libs/forge-vhdl/docs/VHDL_CODING_STANDARDS.md`:

| VHDL Feature | Verilog Compatible? | forge-vhdl Standard |
|--------------|---------------------|---------------------|
| Physical types | ❌ NO | "Comments for units" |
| Records | ❌ NO | "Separate signals or arrays" |
| Subtypes | ⚠️ Limited | "Use base type" |
| Enums | ❌ NO | "`std_logic_vector` constants" |

**Implication:** Cannot use VHDL's physical types or records for voltage types.

**Rationale:** forge-vhdl components must work in Verilog-based projects (Moku platform requirement).

---

### Constraint 2: Units Must Not Detach from Type

**User requirement:** Prevent naked `float` voltages that lose unit information.

```python
# ❌ BAD: Unit-less float
voltage = 2.5  # Volts? Millivolts? What range?

# ✅ GOOD: Type carries units
voltage = Voltage_3V3(2.5)  # Unambiguously 0-3.3V range
```

---

### Constraint 3: Type-Checking at Assignment

**User requirement:** Catch domain mismatches at development time, not runtime.

```python
# Type checker should catch this
trigger: Voltage_3V3 = Voltage_5V_Bipolar(-3.0)  # Type error!
```

---

### Constraint 4: Minimal Implementation

**User requirement:** Not semantically rich, just domain enforcement.

- ✅ Type-checking
- ✅ Range validation
- ✅ Explicit conversions
- ❌ No arithmetic operations
- ❌ No rich semantics

---

## The Solution: Function-Based Type Safety

### Core Insight

Since we **cannot use VHDL language features** (records, physical types) due to Verilog compatibility, we use **explicit package selection** to enforce domain boundaries:

```vhdl
-- Domain declared by package selection
use work.forge_voltage_3v3_pkg.all;

signal trigger : real;  -- Context: 3.3V domain
trigger_digital <= to_digital(trigger);  -- Domain enforced by package
```

**This provides:**
- ✅ Verilog compatibility (standard types only)
- ✅ Explicit domain selection (package name in code)
- ✅ Auditable conversions (package functions)
- ✅ Self-documenting (package name = domain)
- ⚠️ No compile-time type safety (can still mix domains accidentally)

**Trade-off accepted:** 80% type safety (explicit packages + runtime checks) vs 100% (records/physical types that break Verilog compatibility).

---

## Design Decisions

### Decision 1: Three Explicit Packages (Not Parameterized)

**Packages:**
1. `forge_voltage_3v3_pkg` - 0-3.3V unipolar (TTL/digital logic)
2. `forge_voltage_5v0_pkg` - 0-5.0V unipolar (sensor supply)
3. `forge_voltage_5v_bipolar_pkg` - ±5.0V bipolar (Moku DAC/ADC)

**Rationale:**
- Covers 99% of real FPGA work
- Explicit > generic (self-documenting)
- Package name declares domain
- Easy to add more ranges later if needed

**Alternative rejected:** Parameterized package (e.g., `generic V_MAX: real`)
- **Why rejected:** Less explicit, easier to make mistakes, scale factors buried in generics

---

### Decision 2: Function-Based Conversions (Not Language Types)

**Pattern:**
```vhdl
package forge_voltage_3v3_pkg is
    constant V_MIN : real := 0.0;
    constant V_MAX : real := 3.3;
    constant SCALE_FACTOR : real := 32767.0 / 3.3;

    function to_digital(voltage : real) return signed;
    function from_digital(digital : signed(15 downto 0)) return real;
    function clamp(voltage : real) return real;
    function is_valid(voltage : real) return boolean;
end package;
```

**Rationale:**
- Verilog compatible (standard types only)
- Follows existing `volo_voltage_pkg.vhd` pattern (production-proven)
- Explicit conversions (auditable)
- Runtime validation (clamp, is_valid)

**Alternative rejected:** VHDL record types
- **Why rejected:** Violates forge-vhdl coding standards (Verilog incompatible)

**Alternative rejected:** VHDL physical types
- **Why rejected:** Violates forge-vhdl coding standards (Verilog incompatible)

---

### Decision 3: Python Classes with Type Enforcement

**Pattern:**
```python
class Voltage_3V3:
    V_MIN: Final[float] = 0.0
    V_MAX: Final[float] = 3.3

    def __init__(self, volts: float):
        if not (self.V_MIN <= volts <= self.V_MAX):
            raise ValueError(...)
        self._volts = volts

    @property
    def volts(self) -> float:
        return self._volts  # Read-only

    def to_digital(self) -> int:
        return int(self._volts * self.SCALE_FACTOR)

    # Prevent arithmetic
    def __add__(self, other):
        raise TypeError("Cannot add voltages")
```

**Rationale:**
- Python type checker (mypy) catches domain mismatches
- `__init__` validates range at assignment
- Read-only `volts` property prevents unit detachment
- Arithmetic prevention forces explicit conversions
- 1:1 mapping with VHDL packages

**Alternative rejected:** Type aliases (`Voltage_3V3 = float`)
- **Why rejected:** No runtime validation, units can detach

**Alternative rejected:** Rich classes with arithmetic
- **Why rejected:** User requested minimal implementation, not semantically rich

---

### Decision 4: Separate Python Classes (Not Inheritance)

**Pattern:**
```python
class Voltage_3V3:   # Separate class
    ...

class Voltage_5V0:   # Separate class
    ...

class Voltage_5V_Bipolar:  # Separate class
    ...
```

**Rationale:**
- Type checker sees them as distinct types (not compatible)
- No accidental polymorphism
- Clear in function signatures (`def set_trigger(v: Voltage_3V3)`)

**Alternative rejected:** Base class inheritance
- **Why rejected:** Type checker would allow subclass substitution (defeats type safety)

---

### Decision 5: Arithmetic Prevention (Raises TypeError)

**Pattern:**
```python
def __add__(self, other):
    raise TypeError("Cannot add voltages (units would detach)")
```

**Rationale:**
- Prevents `voltage1 + voltage2` (units would detach)
- Forces explicit conversions for cross-domain operations
- Makes unit handling auditable

**Example:**
```python
# ❌ Prevented: Units detach
offset = Voltage_3V3(1.0) + Voltage_3V3(0.5)  # TypeError!

# ✅ Explicit: Preserve units
v1 = Voltage_3V3(1.0)
v2 = Voltage_3V3(0.5)
sum_volts = v1.volts + v2.volts  # Explicit float addition
result = Voltage_3V3(sum_volts)  # Re-wrap with validation
```

---

## VHDL Implementation

### Package Structure

**File:** `libs/forge-vhdl/vhdl/packages/forge_voltage_3v3_pkg.vhd`

```vhdl
package forge_voltage_3v3_pkg is

    -- Constants
    constant V_MIN : real := 0.0;
    constant V_MAX : real := 3.3;
    constant SCALE_FACTOR : real := 32767.0 / 3.3;  -- ~9930.0 digital/volt

    -- Common voltage reference points
    constant DIGITAL_1V0 : signed(15 downto 0) := to_signed(9930, 16);
    constant DIGITAL_2V5 : signed(15 downto 0) := to_signed(24825, 16);
    constant DIGITAL_3V3 : signed(15 downto 0) := to_signed(32767, 16);

    -- Conversion functions
    function to_digital(voltage : real) return signed;
    function from_digital(digital : signed(15 downto 0)) return real;

    -- Validation functions
    function clamp(voltage : real) return real;
    function is_valid(voltage : real) return boolean;

    -- Testbench helpers
    function is_voltage_equal(
        digital : signed(15 downto 0);
        expected_voltage : real;
        tolerance_volts : real := 0.001
    ) return boolean;

end package;

package body forge_voltage_3v3_pkg is

    function to_digital(voltage : real) return signed is
        variable digital_real : real;
        variable digital_int : integer;
    begin
        -- Clamp to valid range
        if voltage > V_MAX then
            digital_real := V_MAX * SCALE_FACTOR;
        elsif voltage < V_MIN then
            digital_real := V_MIN * SCALE_FACTOR;
        else
            digital_real := voltage * SCALE_FACTOR;
        end if;

        -- Round to nearest integer
        digital_int := integer(digital_real + 0.5);

        -- Clamp to 16-bit range [0, 32767]
        if digital_int > 32767 then
            digital_int := 32767;
        elsif digital_int < 0 then
            digital_int := 0;
        end if;

        return to_signed(digital_int, 16);
    end function;

    function from_digital(digital : signed(15 downto 0)) return real is
    begin
        return real(to_integer(digital)) / SCALE_FACTOR;
    end function;

    function clamp(voltage : real) return real is
    begin
        if voltage > V_MAX then
            return V_MAX;
        elsif voltage < V_MIN then
            return V_MIN;
        else
            return voltage;
        end if;
    end function;

    function is_valid(voltage : real) return boolean is
    begin
        return (voltage >= V_MIN) and (voltage <= V_MAX);
    end function;

    function is_voltage_equal(
        digital : signed(15 downto 0);
        expected_voltage : real;
        tolerance_volts : real := 0.001
    ) return boolean is
        variable actual_voltage : real;
    begin
        actual_voltage := from_digital(digital);
        return abs(actual_voltage - expected_voltage) <= tolerance_volts;
    end function;

end package body;
```

**Similar structure for:**
- `forge_voltage_5v0_pkg.vhd` (0-5.0V)
- `forge_voltage_5v_bipolar_pkg.vhd` (±5.0V)

---

## Python Implementation

See: `docs/migration/voltage_types_reference.py` (reference implementation)

**Key features:**
- Three classes: `Voltage_3V3`, `Voltage_5V0`, `Voltage_5V_Bipolar`
- Range validation in `__init__`
- Read-only `volts` property
- `to_digital()` / `from_digital()` methods
- Arithmetic prevention (`__add__`, `__sub__`, etc. raise `TypeError`)

**Example usage:**
```python
from voltage_types import Voltage_3V3, Voltage_5V_Bipolar

# Type-safe assignment
trigger = Voltage_3V3(2.5)
dac = Voltage_5V_Bipolar(-3.0)

# Type checker catches this
trigger = dac  # mypy error: incompatible types

# Explicit conversion
trigger_digital = trigger.to_digital()  # int for register write

# Range validation
bad = Voltage_3V3(5.0)  # ValueError at runtime
```

---

## Where Python Implementation Lives

**Options:**

### Option A: Inside forge-vhdl submodule (Colocated)
```
libs/forge-vhdl/
├── vhdl/packages/
│   ├── forge_voltage_3v3_pkg.vhd
│   ├── forge_voltage_5v0_pkg.vhd
│   └── forge_voltage_5v_bipolar_pkg.vhd
└── python/
    └── voltage_types.py
```
**Pros:** VHDL + Python together, single submodule
**Cons:** Python not usable without VHDL context

---

### Option B: New submodule in forge/libs/ (Independent Library)
```
forge/libs/
├── basic-app-datatypes/    # Serialization types (12 voltage enums)
├── moku-models/            # Platform specs
├── riscure-models/         # Probe specs
└── voltage-types/          # NEW: Physical runtime types
    ├── llms.txt            # Tier 1 quick ref
    ├── CLAUDE.md           # Tier 2 design doc
    ├── python/
    │   └── voltage_types/
    │       ├── __init__.py
    │       ├── voltage_3v3.py
    │       ├── voltage_5v0.py
    │       └── voltage_5v_bipolar.py
    └── vhdl/               # Cross-reference to forge-vhdl
        └── README.md       # Points to libs/forge-vhdl/vhdl/packages/
```
**Pros:** Independent, reusable, follows "authoritative bubble" pattern
**Cons:** Another submodule to manage

---

### Option C: Standalone repository (Most Independent)
```
github.com/sealablab/voltage-types/
├── python/
│   └── voltage_types/
│       └── ...
└── docs/
    ├── VHDL_MAPPING.md  # Cross-reference to forge-vhdl
    └── DESIGN.md        # This document
```
**Pros:** Fully independent, pip-installable, max reusability
**Cons:** Not integrated with monorepo, separate versioning

---

**Recommendation:** **Option B** (new forge/libs/voltage-types/ submodule)

**Rationale:**
- Follows ARCHITECTURE_OVERVIEW.md "authoritative bubble" pattern
- Python can be used without VHDL context
- Still integrated with monorepo for cross-referencing
- Clear separation: basic-app-datatypes = serialization, voltage-types = runtime safety

**But defer this decision:** For now, keep reference implementation in `docs/migration/voltage_types_reference.py`. Decide placement when ready to commit.

---

## Relationship with forge/basic-app-datatypes

**Question:** How do these relate to the 12 voltage types in `basic-app-datatypes`?

### Answer: Separate Domains, Both Coexist

**basic-app-datatypes** (Serialization types):
- 12 voltage types (`voltage_output_05v_s16`, `voltage_output_10v_s16`, etc.)
- For YAML → register encoding
- Comprehensive coverage (±20V, ±25V ranges)
- Enum + metadata

**voltage-types** (Runtime safety types):
- 3 voltage types (3.3V, 5V, ±5V)
- For hand-written VHDL/Python safety
- Pragmatic common ranges
- Function-based packages (VHDL), classes (Python)

**They complement each other:**
```python
# Serialization (basic-app-datatypes)
from basic_app_datatypes import BasicAppDataTypes
voltage_type_enum = BasicAppDataTypes.VOLTAGE_OUTPUT_05V_S16

# Runtime safety (voltage-types)
from voltage_types import Voltage_5V_Bipolar
actual_voltage = Voltage_5V_Bipolar(-3.0)
```

**Cross-reference in documentation:**
- `basic-app-datatypes/CLAUDE.md` → Link to voltage-types for runtime safety
- `voltage-types/CLAUDE.md` → Link to basic-app-datatypes for serialization

---

## Success Criteria

When implementation is complete:

### VHDL Side:
- [ ] `forge_voltage_3v3_pkg.vhd` implemented
- [ ] `forge_voltage_5v0_pkg.vhd` implemented
- [ ] `forge_voltage_5v_bipolar_pkg.vhd` implemented
- [ ] All packages have CocoTB tests
- [ ] Tests validate conversion accuracy
- [ ] Packages follow forge-vhdl coding standards (Verilog compatible)

### Python Side:
- [ ] Three classes implemented (`Voltage_3V3`, `Voltage_5V0`, `Voltage_5V_Bipolar`)
- [ ] Range validation in `__init__`
- [ ] Arithmetic prevention (`__add__`, etc. raise `TypeError`)
- [ ] `to_digital()` / `from_digital()` methods match VHDL
- [ ] Type checking works (mypy catches domain mismatches)
- [ ] Unit tests validate behavior

### Documentation:
- [ ] This design document (`VOLTAGE_TYPE_SYSTEM_DESIGN.md`)
- [ ] Reference implementation (`voltage_types_reference.py`)
- [ ] forge-vhdl updated (llms.txt, CLAUDE.md)
- [ ] Monorepo awareness (ARCHITECTURE_OVERVIEW.md, FORGE_VHDL_P4.md)
- [ ] Cross-references to basic-app-datatypes

### Integration:
- [ ] Example usage in fsm_observer or other component
- [ ] Conversion patterns documented
- [ ] Python type checking validated (mypy passes)

---

## Open Questions (For Future)

### Q1: Should we add more voltage ranges?
**Examples:** 1.8V, 10V, 15V?
**Answer:** Start with 3, add on demand. Most real work covered by these 3.

### Q2: Should we provide cross-domain conversion functions?
**Example:** `convert_3v3_to_5v_bipolar(voltage: Voltage_3V3) -> Voltage_5V_Bipolar`?
**Answer:** Not initially. Require explicit unwrap + rewrap (auditable).

### Q3: Should Python implementation use Pydantic?
**Pros:** Validation, serialization
**Cons:** Dependency, might be overkill
**Answer:** Defer. Reference implementation is pure Python (no dependencies).

---

## Key Takeaways

1. **Function-based safety** (not language types) for Verilog compatibility
2. **Explicit package selection** enforces domain boundaries
3. **Python classes** provide type-checking at assignment
4. **Arithmetic prevention** keeps units attached
5. **1:1 VHDL-Python mapping** for consistency
6. **80% type safety** accepted (vs 100% that breaks Verilog compatibility)

**The pattern works because:**
- Verilog compatibility maintained
- Domain selection is explicit (package/class name)
- Runtime validation catches errors
- Type checkers catch mismatches
- Production-proven (follows existing `volo_voltage_pkg` pattern)

---

**Version History:**
- v1.0 (2025-11-04): Initial design finalized

**Next Steps:**
1. Review and approve this design
2. Implement VHDL packages in forge-vhdl
3. Decide Python implementation placement
4. Create CocoTB tests
5. Update documentation ecosystem
