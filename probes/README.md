# probes/ Directory - DEPRECATED

**Status:** ⚠️ DEPRECATED (as of 2025-11-03)
**Reason:** Replaced by **Option A architecture**

---

## What Happened?

During Phase 4 agent refactoring and testing (Nov 2025), we discovered that:

1. **forge's actual implementation** expects YAML specs in `forge/apps/` directory
2. **Simpler architecture:** Having YAML + generated files in one place is more intuitive
3. **No forge changes needed:** Option A works immediately with existing forge code

## Current Architecture (Option A)

**Primary workspace:** `forge/apps/<probe_name>/`

```
forge/apps/DS1140_PD/
├── DS1140_PD.yaml                      # Source specification
├── DS1140_PD_custom_inst_shim.vhd     # Auto-generated
├── DS1140_PD_custom_inst_main.vhd     # User implements logic here
└── README.md
```

**Workflow:**
1. Create YAML in `forge/apps/<probe_name>/<probe_name>.yaml`
2. Run `/generate forge/apps/<probe_name>/<probe_name>.yaml`
3. Edit generated `*_main.vhd` template
4. Deploy and debug

## What Was This Directory For?

**Original intent (Option B - not chosen):**
- Source specifications would live in `probes/*/specs/*.yaml`
- Generated packages would go to `forge/apps/*/`
- Separation of "source" vs "generated"

**Why not chosen:**
- Required forge code modifications
- More complex mental model
- Didn't align with forge's actual implementation
- Can revisit later if separation becomes important

## What's In Here Now?

This directory contains partial work artifacts from earlier experiments:
- `DS1140_PD/` - Partial probe structure (empty placeholders)
- `DS1120_PD/` - Partial probe structure (empty placeholders)

**Status:** These are **not actively used** in the current workflow.

## Should I Delete This?

**Options:**
1. **Keep it** - Historical reference, shows evolution of architecture
2. **Delete it** - Clean up, reduce confusion (recommended after confirming Option A works)
3. **Archive it** - Move to `archive/probes_option_b/`

**Recommendation:** Keep for now, delete after Option A is validated in production use.

## References

- **Option A Testing:** `OPTION_A_TEST_SUMMARY.md`
- **Phase 4 Handoff:** `P4_AGENT_REFACTOR_HANDOFF.md` (see "Post-Phase 4: Option A Testing")
- **Updated Workflow:** `.claude/shared/PROBE_WORKFLOW.md`

---

**For new probe development, use:** `forge/apps/<probe_name>/`

**Questions?** See updated agent docs in `.claude/agents/probe-design-orchestrator/`
