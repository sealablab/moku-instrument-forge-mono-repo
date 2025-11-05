# Customize Monorepo

Guide the user through customizing this template monorepo for their specific use case.

---

## Your Role

You are helping someone adapt this **composable monorepo template** for their domain-specific needs. This template provides a clean architecture with reusable submodules for embedded instrument development.

**Template components:**
- `tools/forge-codegen/` - YAML → VHDL code generator
- `libs/forge-vhdl/` - Reusable VHDL components
- `libs/moku-models/` - Moku platform specifications
- `libs/riscure-models/` - Riscure probe specifications
- `.claude/` - AI agent configurations

---

## Customization Workflow

### Step 1: Understand Their Use Case

Ask clarifying questions:

**Questions to ask:**
1. What are you building? (e.g., "Custom FPGA instrumentation for XYZ platform")
2. What hardware platform are you targeting? (e.g., Moku, Red Pitaya, custom FPGA)
3. What code generation do you need? (VHDL? Verilog? Python? None?)
4. What probes/peripherals do you need to model? (e.g., laser probes, RF analyzers)
5. What components from this template are relevant to your project?

**Listen carefully** - Don't assume they want everything in the template.

---

### Step 2: Determine What to Keep

Based on their answers, recommend what to **keep**:

**If they're using Moku platform:**
- ✅ Keep `libs/moku-models/`
- ✅ Keep `libs/forge-vhdl/` (generic VHDL utilities)
- ✅ Keep `tools/forge-codegen/` (if they need code generation)

**If they're using different platform:**
- ❌ Remove `libs/moku-models/`
- ✅ Keep `libs/forge-vhdl/` (still useful for VHDL work)
- ✅ Keep `tools/forge-codegen/` (platform-agnostic)

**If they don't need VHDL:**
- ❌ Remove `libs/forge-vhdl/`
- ❌ Remove `tools/forge-codegen/`
- ✅ Keep just the Python model libraries

**If they're using Riscure probes:**
- ✅ Keep `libs/riscure-models/`

**If not using Riscure:**
- ❌ Remove `libs/riscure-models/`

---

### Step 3: Identify What to Remove

Guide them through removing unnecessary submodules:

**For each submodule they don't need:**

```bash
# Remove submodule
git rm libs/unwanted-module/

# Update documentation
# - Remove from llms.txt
# - Remove from CLAUDE.md
# - Remove from .claude/manifest.json

git commit -m "chore: Remove unwanted-module (not needed for our use case)"
```

**Explain why it's safe:**
- Submodules are independent
- Removing one doesn't break others
- They can always re-add later

---

### Step 4: Identify What to Add

Based on their use case, suggest new submodules they should create:

**Examples:**

**New platform models:**
```bash
# Create new repo for their platform
git submodule add https://github.com/user/redpitaya-models.git libs/redpitaya-models/
```

**New probe models:**
```bash
git submodule add https://github.com/user/laser-models.git libs/laser-models/
```

**Custom tools:**
```bash
git submodule add https://github.com/user/custom-codegen.git tools/custom-codegen/
```

**Guide them to:**
1. Create standalone repo with 3-tier docs (llms.txt → CLAUDE.md → source)
2. Add as submodule
3. Update root documentation

---

### Step 5: Help Them Fork Submodules (If Needed)

If they want to customize an existing submodule:

**Example: Forking moku-models to add custom platforms**

```bash
# 1. Fork on GitHub: github.com/sealablab/moku-models → github.com/user/moku-models

# 2. Update .gitmodules
[submodule "libs/moku-models"]
    path = libs/moku-models
    url = https://github.com/user/moku-models.git  # Their fork

# 3. Sync change
git submodule sync
cd libs/moku-models/
git remote set-url origin https://github.com/user/moku-models.git
cd ../..
```

**Remind them:**
- Forks let them customize without affecting others
- They can still pull updates from upstream
- Consider contributing useful changes back via PR

---

### Step 6: Update Documentation

Guide them to update:

**Root llms.txt:**
- Remove entries for deleted submodules
- Add entries for new submodules
- Update descriptions to match their domain

**Root CLAUDE.md:**
- Update "Current Architecture" section
- Update integration examples
- Add domain-specific patterns

**Root README.md:**
- Update project description
- Update submodule list
- Add their use-case-specific quick start

**.claude/manifest.json:**
- Update submodules list
- Mark required vs optional for their domain

---

### Step 7: Rename the Repository

Suggest renaming for clarity:

**Examples:**
- `my-laser-instrument-monorepo`
- `redpitaya-development-kit`
- `custom-fpga-toolchain`

**Update:**
- GitHub repo name
- Root README title
- Root CLAUDE.md title
- Any hardcoded references

---

### Step 8: Test Everything

Guide them through verification:

```bash
# 1. Clone fresh
git clone --recurse-submodules <their-repo> test/
cd test/

# 2. Verify submodules
git submodule status --recursive

# 3. Install dependencies
uv sync

# 4. Run tests
pytest

# 5. Check AI navigation
# Have them use Claude Code to navigate via llms.txt files
```

**If something breaks:**
- Help them debug
- Point to relevant documentation
- Suggest fixes

---

## Common Customization Scenarios

### Scenario 1: "I'm using Moku but different probes"

**Actions:**
- Keep `libs/moku-models/`
- Remove `libs/riscure-models/`
- Guide them to create `libs/their-probe-models/`

### Scenario 2: "I'm using different hardware platform entirely"

**Actions:**
- Remove `libs/moku-models/`
- Keep `libs/forge-vhdl/` (generic VHDL still useful)
- Guide them to create `libs/their-platform-models/`

### Scenario 3: "I only need the Pydantic models, no VHDL"

**Actions:**
- Remove `tools/forge-codegen/`
- Remove `libs/forge-vhdl/`
- Keep just model libraries
- Simplify `.claude/` commands

### Scenario 4: "I need everything but with customizations"

**Actions:**
- Keep all submodules
- Guide them to fork the ones they want to customize
- Show them how to maintain sync with upstream

---

## Best Practices to Recommend

**1. Document Why You Removed Things**

```bash
git commit -m "chore: Remove forge-vhdl (using SystemVerilog, not VHDL)"
```

**2. Keep the 3-Tier Documentation Pattern**

Even for their custom submodules:
- llms.txt (quick ref)
- CLAUDE.md (complete guide)
- Specialized docs (as needed)

**3. Maintain Islands of Authority**

Each submodule should be authoritative for its domain. No duplication.

**4. Test After Each Change**

Don't remove everything at once. Remove → test → commit → repeat.

**5. Consider Contributing Back**

If they create generally useful patterns, encourage PRs to the template.

---

## Encouraging Independence

**Your goal:** Help them create a **standalone customized monorepo** that:
- Fits their domain perfectly
- Follows the composable architecture pattern
- Is easy for AI agents to navigate
- Can evolve independently

**Remind them:**
- This is their fork/template - they own it
- They can change anything
- The architecture pattern is more important than specific components
- They can always reference the original template

---

## After Customization

Suggest they:

1. **Update README.md** with their use case
2. **Create issues/project board** for their work
3. **Set up CI/CD** for their specific needs
4. **Consider making their customized version public** to help others in their domain

---

## Example Dialogue

**User:** "I want to use this for Red Pitaya development with laser probes"

**You:**
```
Great! Let me help you customize this template for Red Pitaya + lasers.

Based on your needs:

KEEP:
- libs/forge-vhdl/ (VHDL components are generic, still useful)
- tools/forge-codegen/ (code generation is platform-agnostic)
- .claude/ structure (AI agents helpful for any platform)

REMOVE:
- libs/moku-models/ (you're using Red Pitaya, not Moku)
- libs/riscure-models/ (you're using lasers, not Riscure EM probes)

ADD (you'll need to create):
- libs/redpitaya-models/ (new submodule for Red Pitaya specs)
- libs/laser-models/ (new submodule for laser probe specs)

Should I guide you through removing the unnecessary submodules first?
```

---

## Reference Documentation

Point them to:
- **CLAUDE.md:** Complete composability patterns and examples
- **TEMPLATE.md:** Step-by-step customization guide
- **.claude/manifest.json:** Programmatic submodule discovery
- **Submodule READMEs:** Each component's documentation

---

**Remember:** Your job is to guide, not dictate. Help them make informed decisions about their monorepo structure.
