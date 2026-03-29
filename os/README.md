# Universal Systems Kernel — Directory Guide

> A complete reference for the kernel's file structure and the purpose of each directory and file.

---

## Purpose of This Document

This guide is the definitive reference for navigating the kernel's file system. It documents every directory, the files it contains, and the intent behind each. Any agent activating the kernel should read this document alongside `CLAUDE.md` before exploring the directory tree.

---

## Directory Tree

```
os/kernel/
├── KERNEL.md           # Operating manual
├── phases/             # 7-phase pipeline
├── agents/             # 11 cognitive agents
├── principles/         # Design laws and primitives
├── schemas/            # Canonical object schemas
├── templates/          # Pipeline artifact templates
├── manifests/          # Kernel configuration
├── audits/             # Audit backbone and reroute logic
├── runtime/            # Controller loop and modes
├── docs/               # Walkthroughs and examples
└── projects/           # Example canonical packages
```

---

## Directory Descriptions

### `KERNEL.md`

The kernel's primary operating manual. Contains the full pipeline specification, agent table, audit vector mechanics, reroute logic, iteration modes, escalation policy, anti-slop criteria, and a complete file map. Every agent activating the kernel must read this file in full before beginning any pipeline phase.

---

### `phases/`

One specification file per pipeline phase. Each file defines the phase's inputs, outputs, agent assignments, responsibilities, failure conditions, and handoff format.

```
phases/
  01_intent_compilation.md       # Phase 1: raw request → structured intent object
  02_success_model.md            # Phase 2: intent → project-specific quality dimensions
  03_architecture_search.md      # Phase 3: generate genuinely different structural candidates
  04_comparative_reasoning.md    # Phase 4: score, rank, and select or hybridize candidates
  05_structural_synthesis.md     # Phase 5: winning thesis → complete system geometry
  06_audit_and_routing.md        # Phase 6: measure quality vectors; reroute on failure
  07_packaging.md                # Phase 7: freeze package; emit blueprint and handoff manifest
```

Phase files are read sequentially as the pipeline advances. They must not be skipped or reordered.

---

### `agents/`

One definition file per kernel agent. Each file specifies the agent's cognitive function, activation conditions, inputs, outputs, decision authority, and interaction with other agents. Agents are cognitive roles, not personas.

```
agents/
  controller_architect.md        # Coherence owner, routing authority, package steward
  intent_analyst.md              # Objective extraction, ambiguity mapping, consequence rating
  success_model_architect.md     # Quality dimension derivation, threshold setting
  search_architect.md            # Candidate generation, structural thesis variation
  comparative_reasoner.md        # Scoring, ranking, tradeoff matrix, selection
  synthesis_architect.md         # System geometry: subsystems, interfaces, flows
  failure_mode_architect.md      # Adversarial testing, breakdown and drift analysis
  optimization_architect.md      # Elegance, efficiency, simplification
  audit_architect.md             # Vector production, evidence grading, reroute targeting
  mutation_architect.md          # Surgical corrective changes at reroute stage
  packaging_architect.md         # Package normalization, blueprint, handoff manifest
```

---

### `principles/`

The foundational reasoning rules that govern all kernel behavior. These files define the universal building blocks the kernel uses to reason about any system, the laws that constrain how it operates, and the standard by which it measures output quality.

```
principles/
  system_primitives.md           # 11 universal primitives present in every system
  design_laws.md                 # 10 kernel design laws — non-negotiable operating rules
  world_class_standard.md        # Definition of world-class and how the kernel measures it
```

Agents must internalize these principles before executing any phase. They are not reference material — they are active constraints on every decision the kernel makes.

---

### `schemas/`

JSON Schema definitions for every canonical object the kernel produces or consumes. Schemas define the exact structure of intent objects, success models, candidate architectures, synthesized system geometries, audit vectors, evolution ledger entries, and handoff manifests.

```
schemas/
  intent_object.json             # Compiled intent from Phase 1
  success_model.json             # Quality dimensions and thresholds from Phase 2
  candidate_architecture.json    # Structural candidate from Phase 3
  comparative_matrix.json        # Scoring matrix from Phase 4
  synthesized_architecture.json  # Full system geometry from Phase 5
  audit_vector.json              # Dimensional quality scores from Phase 6
  evolution_ledger_entry.json    # Single reroute record
  handoff_manifest.json          # Downstream consumer manifest from Phase 7
  system_package.json            # Full canonical package envelope
```

All phase outputs must conform to the relevant schema. Schema validation is a quality gate before any phase transition.

---

### `templates/`

YAML and Markdown templates that phases populate as they execute. Templates define the exact fields each artifact must contain. Agents fill templates — they do not invent their own artifact formats.

```
templates/
  final_package_template.yaml    # Master template for the canonical system package
  system_blueprint_template.md   # Human-readable blueprint structure
  handoff_manifest_template.yaml # Handoff artifact for downstream consumers
  audit_report_template.md       # Audit summary for review
  evolution_ledger_template.yaml # Append-only mutation log
```

Using templates ensures consistency across all runs and makes outputs machine-readable by downstream processes.

---

### `manifests/`

Kernel configuration files that control operating parameters, mode thresholds, escalation triggers, and agent activation rules. Manifests are read by the Controller Architect during initialization.

```
manifests/
  kernel_config.yaml             # Global kernel settings and version
  mode_thresholds.yaml           # Score thresholds per runtime mode
  escalation_policy.yaml         # Conditions that trigger human escalation
  agent_registry.yaml            # Agent identifiers and activation rules
```

Manifests are not modified during a pipeline run. They are configuration, not state.

---

### `audits/`

Specifications and reference material for the kernel's audit system. This directory defines how quality vectors are computed, how evidence is graded, how uncertainty is rated, and how reroute targets are selected.

```
audits/
  audit_backbone.md              # 8 universal backbone dimensions audited on every project
  confidence_vectors.md          # Evidence strength and uncertainty rating rules
  reroute_logic.md               # Mapping from failing dimensions to reroute targets
  anti_slop_criteria.md          # Structural defects that trigger automatic reroute
```

The Audit Architect operates from these specifications. They define the floor of acceptable output quality and the precise conditions under which the kernel corrects itself.

---

### `runtime/`

Specifications for the Controller Architect's execution loop, runtime mode selection, iteration limits, and escalation handling.

```
runtime/
  controller_loop.md             # Phase-by-phase execution logic and transition rules
  modes.md                       # Standard, Search Heavy, Conservative, Overdrive
  escalation_handling.md         # When and how to surface decisions to the human
  iteration_limits.md            # Maximum corrective iterations before forced escalation
```

Runtime files govern how the kernel behaves under different conditions — not what it produces, but how it decides, iterates, and stops.

---

### `docs/`

Explanatory guides, walkthroughs, and worked examples. This directory is for understanding, not execution. Agents reading docs will find annotated examples of pipeline runs, explanations of design choices, and tutorials for applying the kernel to unfamiliar problem types.

```
docs/
  pipeline_walkthrough.md        # Annotated end-to-end pipeline example
  reading_audit_vectors.md       # How to interpret quality vector output
  reroute_examples.md            # Worked examples of reroute scenarios
  applying_to_new_domains.md     # How to apply the kernel to an unfamiliar domain
```

Docs are informational. They do not define behavior. Behavior is defined by phase specs, agent specs, principles, and manifests.

---

### `projects/`

Example canonical packages from completed pipeline runs. Each project subdirectory contains the final YAML package, the generated blueprint, and the handoff manifest. These serve as reference examples for what complete kernel output looks like.

```
projects/
  example-01/
    final_system_package.yaml
    system_blueprint.md
    handoff_manifest.yaml
  example-02/
    final_system_package.yaml
    system_blueprint.md
    handoff_manifest.yaml
```

Example packages span different domain types to demonstrate the kernel's domain universality. They are not templates — they are completed outputs used for comparison and calibration.

---

## Reading Order for Activation

Agents activating the kernel should read files in this order:

1. `CLAUDE.md` (root) — kernel identity, pipeline overview, design laws summary
2. `os/kernel/KERNEL.md` — full operating manual
3. `os/kernel/principles/` — system primitives, design laws, world-class standard
4. `os/kernel/phases/` — phase-by-phase specifications
5. `os/kernel/agents/` — cognitive agent definitions
6. `os/kernel/templates/` — artifact formats (referenced during execution)

Read completely. Do not skip files. The kernel is a coherent system — partial reading produces partial understanding and defective output.

---

*Universal Systems Kernel v1.0 — os/README.md*
*Structural cognition, separated from domain cognition, measured at every stage.*
