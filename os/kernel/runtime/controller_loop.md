# Controller Loop

> The main execution loop of the kernel. Every system build flows through this loop.

---

## Overview

The Controller Loop is the heartbeat of the kernel. It manages the lifecycle of a system build from initial intent to final packaged architecture. The loop advances through phases, dispatches work to agents and OS teams, runs audits, handles rerouting, and enforces safety limits.

The Commander agent IS the Controller Architect. It does not delegate the loop — it runs it directly, dispatching teams at each phase and synthesizing their outputs.

---

## The Loop

```
FUNCTION controller_loop(build_request, mode):

  # STEP 1: Initialize
  package = create_package_shell(build_request)
  ledger = create_evolution_ledger()
  current_phase = PHASE_1
  iteration = 0
  reroute_count = 0
  state = INIT

  # STEP 2: Main Loop
  WHILE state != FINALIZED AND state != ABORTED:

    iteration += 1

    # STEP 3: Read current state
    context = read_package_state(package)
    thresholds = load_thresholds(mode, package.success_model)

    # STEP 4: Dispatch current phase
    state = EXECUTING
    phase_agents = resolve_phase_agents(current_phase)
    os_teams = resolve_os_teams(current_phase)  # via kernel_os_bridge

    dispatch_message = build_dispatch(
      phase = current_phase,
      context = context,
      reroute_instructions = get_reroute_instructions(ledger, current_phase),
      iteration = iteration
    )

    # STEP 5: Execute phase
    phase_outputs = execute_phase(phase_agents, os_teams, dispatch_message)

    # STEP 6: Update package
    package = merge_phase_outputs(package, current_phase, phase_outputs)

    # STEP 7: Audit (always runs after any phase)
    IF current_phase >= PHASE_5:
      audit_result = run_audit(package, thresholds)
      package.audit_results = audit_result

      # STEP 8: Evaluate audit
      IF audit_result.overall_status == "pass":
        # Advance to next phase
        current_phase = next_phase(current_phase)

        IF current_phase == PHASE_COMPLETE:
          state = FINALIZED

      ELIF audit_result.overall_status == "fail":
        # STEP 9: Compute reroute
        reroute_count += 1

        IF reroute_count > MAX_REROUTE_ITERATIONS:
          state = escalate_and_abort(package, ledger, audit_result)
          CONTINUE

        reroute_target = determine_reroute_target(
          audit_result.failed_dimensions,
          ledger
        )

        # Record in ledger
        ledger.append(create_ledger_entry(
          iteration, current_phase, audit_result, reroute_target
        ))

        # Check for overdrive trigger
        IF should_trigger_overdrive(audit_result, ledger):
          run_overdrive(package, audit_result.weakest_dimension)

        # Reroute
        current_phase = reroute_target
        state = REROUTING

    ELSE:
      # Phases 1-4: advance without full audit
      # (lightweight validation only)
      validation = validate_phase_output(current_phase, phase_outputs)

      IF validation.valid:
        current_phase = next_phase(current_phase)
      ELSE:
        # Phase output is malformed or incomplete — retry once
        phase_outputs = retry_phase(phase_agents, os_teams, dispatch_message, validation.errors)
        package = merge_phase_outputs(package, current_phase, phase_outputs)
        current_phase = next_phase(current_phase)

    # Log transition
    log_phase_transition(iteration, current_phase, state, reroute_count)

  # STEP 10: Finalize
  IF state == FINALIZED:
    final_package = run_packaging_phase(package, ledger)
    emit_outputs(final_package)
    RETURN final_package

  ELIF state == ABORTED:
    emit_partial_outputs(package, ledger, abort_reason)
    RETURN partial_package
```

---

## State Machine

```
                    ┌─────────────────────────────────────────┐
                    │                                         │
                    v                                         │
INIT ──> PHASE_1 ──> PHASE_2 ──> PHASE_3 ──> PHASE_4 ──> PHASE_5 ──> PHASE_6 ──> PHASE_7 ──> FINALIZED
                                                              │          │
                                                              │          │ (audit fail)
                                                              │          │
                                                              └──────────┘
                                                              reroute to
                                                              any earlier
                                                              phase
                                                                  │
                                                                  │ (max reroutes exceeded)
                                                                  v
                                                               ABORTED
```

### State Definitions

| State | Description |
|---|---|
| `INIT` | Package shell created. No phases executed yet. |
| `PHASE_1` through `PHASE_7` | Currently executing or about to execute this phase. |
| `EXECUTING` | A phase is actively running (agents and teams are working). |
| `REROUTING` | Audit failed. System is returning to an earlier phase. |
| `FINALIZED` | All phases complete. All audits pass. Package is ready. |
| `ABORTED` | Max reroutes exceeded or unresolvable escalation. Partial output emitted. |

### Phase Definitions

| Phase | Name | Purpose | Full Audit? |
|---|---|---|---|
| 1 | Intent Compilation | Understand what the system must do and why | No (validation only) |
| 2 | Success Model | Define measurable success criteria and thresholds | No (validation only) |
| 3 | Architecture Search | Generate candidate architectures | No (validation only) |
| 4 | Comparative Reasoning | Evaluate and rank candidates, select winner | No (validation only) |
| 5 | Structural Synthesis | Elaborate the selected architecture into full structure | Yes |
| 6 | Audit | Full 8-dimension audit against universal backbone | Yes (this IS the audit) |
| 7 | Packaging | Compile canonical system package with all artifacts | Yes (final check) |

---

## Phase Dispatch Details

### How the Controller Dispatches Work

For each phase, the Controller:

1. **Resolves agents**: Determines which kernel phase agents handle the structural reasoning.
2. **Resolves OS teams**: Uses the kernel-OS bridge (`integration/kernel_os_bridge.md`) to determine which OS teams provide domain expertise.
3. **Builds context**: Assembles all relevant outputs from previous phases, plus any reroute instructions from the evolution ledger.
4. **Sends dispatch message**: Structured message conforming to `os/commander/protocols.md`.
5. **Receives outputs**: Waits for all dispatched agents/teams to return results.
6. **Merges outputs**: Integrates results into the canonical package.

### Context Flow Between Phases

Each phase receives:
- All outputs from ALL previous phases (cumulative context).
- The current state of the canonical package.
- Reroute instructions (if this is a rerouted execution) including:
  - Which dimensions failed and why.
  - What the evolution ledger shows about previous attempts.
  - Specific instructions for what to fix.
- Runtime mode and threshold configuration.

### Phase Output Validation (Phases 1-4)

Before full audit is possible (Phases 1-4), each phase output is validated for structural completeness:
- Does the output contain all required fields for this phase?
- Is the output internally consistent?
- Does the output build coherently on previous phase outputs?

Validation failures trigger a single retry with error feedback. If the retry also fails, the Controller logs the issue and advances (the full audit in Phase 6 will catch substantive problems).

---

## Timeout and Safety

### Max Reroute Iterations: 5

Defined in `audits/reroute_logic.md`. After 5 reroutes, the system escalates.

### Max Total Iterations: 20

Safety valve against infinite loops. If total iterations (including non-reroute forward progress) exceeds 20, the system halts and emits current state.

### Per-Phase Timeout

Each phase has a maximum execution time. If a phase does not produce output within its budget, the Controller:
1. Logs the timeout.
2. Retries once with a simplified scope.
3. If the retry also times out, escalates.

### Resource Budget

The Controller tracks cumulative resource usage (computation, API calls, context window consumption). If the budget is exhausted before finalization, the Controller:
1. Saves the current package state.
2. Emits a CHECKPOINT message.
3. The build can be resumed later via `/resume`.

---

## Logging

Every state transition is logged with:

```json
{
  "timestamp": "2026-03-29T14:32:00Z",
  "iteration": 3,
  "from_state": "PHASE_5",
  "to_state": "PHASE_6",
  "trigger": "phase_5_complete",
  "reroute_count": 1,
  "notes": "Phase 5 reroute addressed failure_awareness. Advancing to audit."
}
```

The log is separate from the evolution ledger. The log records ALL transitions (including forward progress). The ledger records ONLY reroute events with full audit context.

---

## Checkpoint and Resume

The Controller supports interruption and resumption:

### Checkpoint
At the end of each phase, the Controller saves:
- Current package state
- Evolution ledger
- Current phase and iteration counters
- Runtime mode and thresholds

### Resume
When `/resume` is invoked:
1. Load the checkpoint.
2. Restore all state.
3. Re-enter the loop at the saved phase.
4. Continue execution.

Checkpoints enable long builds to survive context window limits, session timeouts, and manual interruptions.

---

*Controller Loop v1.0 — Kernel Runtime Engine*
