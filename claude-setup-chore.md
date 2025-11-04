
  1. Documentation Strategy Variations

  I notice three different patterns:
  - moku-models & riscure-models: Use CLAUDE.md + llms.txt
  - forge: Uses llms.txt + extensive .claude/ structure
  - basic-app-datatypes & forge-vhdl: Use llms.txt only

  ## Q1) CLAUDE patterns
  I am __trying__ to create a composable system where the individual smaller repos (especially those that contain explicitly pydantic models), of which we have a few. In particular:
  - forge/libs/basic-app-datatypes
  - forge/libs/moku-models
  - forge/llibs/riscure-models

**those** should be given very careful consideration as they are designed to both convey the authoritative truth on abstractions and interface that define the fundamental concepts that span the project as a whole 


  2. Monorepo .claude/ Directory

  The root .claude/ has scaffolding (folders with .gitkeep) but the forge submodule has the actual
  agents.

  Questions:
  - Should the monorepo-level agents reference/import forge's agents, or duplicate them?
  - Are the monorepo agents meant to be higher-level orchestrators that delegate to submodule
  contexts?
  - Should probe-design-context work at the monorepo level (probes/DS1120_PD/) or delegate to forge
   agents?

  3. Submodule Agent Hierarchy

  The forge submodule has 5 specialized agents:
  - forge-context (YAML → VHDL generation)
  - deployment-context (deploy to hardware)
  - hardware-debug-context (FSM debugging)
  - docgen-context (documentation generation)
  - workflow-coordinator (orchestrates multi-stage workflows)

  Questions:
  - Should monorepo agents wrap these (e.g., monorepo deployment-context calls forge
  deployment-context)?
  - Or should monorepo agents be independent and focus on probe-specific workflows?
  - How do we handle the fact that deployment/debug agents exist in forge but the monorepo
  placeholder directories suggest they should also exist at monorepo level?

  4. llms.txt Hierarchy

  Currently:
  - Each submodule has its own llms.txt (self-contained)
  - No monorepo-level llms.txt yet

  Questions:
  - Should the monorepo llms.txt be a meta-index that references submodule llms.txt files?
  - Or should it be comprehensive and duplicate/summarize submodule content?
  - How should AI agents discover the layered context (monorepo → forge → nested submodules)?

  5. CLAUDE.md vs llms.txt

  Some submodules use both:
  - llms.txt: Brief reference (Quick Start format)
  - CLAUDE.md: Detailed AI context (Development workflows, integration patterns)

  Question: Should we add CLAUDE.md files to submodules that only have llms.txt, or keep llms.txt
  as the single source?

  6. Probe Development Workflow

  Based on the directory structure, it seems probes are developed at monorepo level:
  - probes/DS1120_PD/ (monorepo)
  - probes/DS1140_PD/ (monorepo)

  But code generation happens in forge:
  - forge/apps/DS1140_PD/ (generated packages)

  Questions:
  - Should the probe-design-context agent operate at monorepo level and coordinate with forge
  agents?
  - Is the workflow: monorepo (YAML) → forge (generate) → monorepo (implement VHDL) → monorepo
  (test)?
  - How do the probes/ and forge/apps/ directories relate?

  Recommendations I Need Your Input On

  Before I design the structure, I'd like to know your preferences:

  1. Agent delegation model: Wrapper agents or independent agents?
  2. llms.txt strategy: Meta-index or comprehensive?
  3. CLAUDE.md adoption: Add to all submodules or keep minimal?
  4. Probe workflow boundary: Where does monorepo responsibility end and forge begin?

  What are your thoughts on these questions?

  Todos
  ☒ Explore repo structure (README.md and directory layout)
  ☒ Read existing llms.txt files in forge/ and libs/forge-vhdl/
  ☒ Perform comprehensive recursive search for all AI config files
  ☐ Analyze findings and ask clarifying questions
  ☐ Design monorepo llms.txt hierarchy with submodule integration
  ☐ Create agent configurations in .claude/agents/
  ☐ Set up context commands in .claude/commands/
