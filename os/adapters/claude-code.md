# Claude Code Runtime Adapter — SuperArchitect OS

**Version:** 1.0
**Owner:** SuperArchitect OS Core Team
**Last Updated:** 2026-03-28
**Audience:** Agents and operators running the SuperArchitect OS via Claude Code CLI

---

## Adapter Purpose

This adapter makes the SuperArchitect OS fully native to the Claude Code CLI environment. It defines how Claude Code loads, interprets, and executes every workflow, persona, and standard defined in the OS.

The SuperArchitect OS is a system of markdown files that act as an operating system for AI agents. Claude Code is the primary runtime. This adapter specifies:
- How the OS boots when Claude Code starts
- How tools are used consistently across all OS operations
- How multi-agent teams are spawned and coordinated
- How hooks enforce quality and log activity automatically
- How custom slash commands activate OS workflows

When this adapter is followed, every Claude Code session operating in this repository behaves as a fully autonomous SuperArchitect agent.

---

## CLAUDE.md Integration

The root `CLAUDE.md` file is the OS bootloader. When Claude Code starts in the `/home/user/superarchitect` directory, it automatically reads `CLAUDE.md` and loads the OS context.

### Root CLAUDE.md Responsibilities

The root CLAUDE.md must:
1. Reference the OS manifest: `os/manifest.md`
2. Declare the active persona default (Commander)
3. List available slash commands
4. Specify which standards files are always-active
5. Define the project's quality thresholds

### Subdirectory CLAUDE.md Files

Each subdirectory that represents a team's workspace should have its own `CLAUDE.md` that:
- Declares the team persona active in that directory
- Lists the standards most relevant to that team's work
- Provides quick-reference links to the team's input and output conventions
- Scopes memory to the team's domain (avoids cross-team context pollution)

**Convention:**
```
/home/user/superarchitect/
  CLAUDE.md                     # Root: boots OS, defines Commander persona
  os/
    CLAUDE.md                   # OS meta: persona = OS architect
  workspaces/
    [project-name]/
      CLAUDE.md                 # Project: loads project-specific context
      teams/
        architect/CLAUDE.md    # Architect team context
        backend/CLAUDE.md      # Backend team context
        frontend/CLAUDE.md     # Frontend team context
```

### What Each CLAUDE.md Must Declare

```markdown
# [Context Name]

## Active Persona
[Persona name and reference to persona file]

## Always-Load Standards
- [path to quality standards]
- [path to architecture standards]

## Team-Specific Context
[Brief description of what this agent/team does in this directory]

## Tool Use Policy
[Any tool restrictions or requirements specific to this context]
```

---

## Tool Use Patterns

Agents in the SuperArchitect OS follow consistent, principled patterns for every Claude Code tool.

### Read

Use Read to load context before any action that depends on existing state.

**Rules:**
- Always Read the relevant architecture doc before implementing a component
- Always Read existing files before editing them (prevents overwrite of parallel work)
- Read the relevant standards before code review, audit, or quality gate
- When the target file is large, read the specific sections needed (use offset/limit)

```
Pattern: CONTEXT LOAD → READ relevant docs → ACT → VERIFY
```

### Write

Use Write only for creating new files. For modifications to existing files, use Edit.

**Rules:**
- Never Write a file that already exists (use Edit instead)
- Before Write, verify the parent directory exists with a Bash `ls`
- After Write, verify the file contents with a Read
- Log every Write in the session log (enforced by PostToolUse hook)
- Never Write secrets, credentials, or environment-specific configuration to tracked files

### Edit

Use Edit for all modifications to existing files.

**Rules:**
- Always Read the file first; Edit will fail without a prior Read
- Prefer targeted, minimal edits — change only what must change
- For large refactors, prefer multiple small Edits over one large Write rewrite
- Use `replace_all` only for renaming that should apply everywhere
- After Edit, Read the relevant section to confirm the change is correct

### Bash

Use Bash for system operations: running tests, checking git state, installing dependencies, running builds.

**Rules:**
- Never use Bash for file reading (use Read tool)
- Never use Bash for file searching (use Grep or Glob tools)
- Always use absolute paths in Bash commands
- Quote paths with spaces
- Chain dependent commands with `&&` not `;` (fail fast)
- Use `run_in_background` for long-running processes (builds, test suites)

### Grep

Use Grep for content search across files.

**Rules:**
- Never use `grep` as a Bash command — always use the Grep tool
- Use `output_mode: "files_with_matches"` for locating files, `"content"` for reading matches
- Specify `glob` or `type` filters to narrow search scope
- Use `context` parameter when surrounding lines are needed for understanding

### Glob

Use Glob for finding files by name or path pattern.

**Rules:**
- Use Glob before Read when the exact file path is uncertain
- Use specific patterns: `os/standards/*.md` not `**/*.md` when scope is known
- Sort results by modification time to identify most recently changed files

### Agent (Multi-agent Spawning)

The Agent tool is the core of SuperArchitect's parallel execution model. Use it to spawn specialist teams that work concurrently.

---

## Multi-Agent Spawning

The SuperArchitect OS achieves parallelism by spawning multiple specialist agents using the Agent tool. Each agent operates in its own context with a scoped prompt.

### Spawning Pattern

```
Commander (orchestrator)
  ├── Spawns Agent: Architect Team
  │   └── Task: Design system, produce architecture docs
  ├── Spawns Agent: Security Team
  │   └── Task: Security requirements and threat model
  └── Spawns Agent: Data Team
      └── Task: Data model and storage strategy
```

### Agent Prompt Template

When spawning a specialist agent, use this template for the agent's prompt:

```
You are the [TEAM NAME] of the SuperArchitect OS.

## Your Persona
[Paste or reference the team's persona from os/teams/[team].md]

## Your Task
[Specific, bounded task for this agent]

## Your Inputs
[List of files this agent should Read first]

## Your Outputs
[Exact files this agent should Write or Edit, with paths]

## Standards You Must Follow
- Quality: os/standards/quality.md
- Architecture: os/standards/architecture.md
- [Additional relevant standards]

## Constraints
- Do not exceed your domain boundary
- Report completion with a structured summary
- If you encounter a blocking issue, describe it precisely rather than guessing
```

### Coordination Rules

- The Commander spawns agents for parallelizable work packages
- Agents that have dependencies must be sequenced (spawn A, wait, spawn B with A's output)
- Each agent writes to its own output directory to avoid conflicts
- Commander aggregates outputs and resolves conflicts before proceeding
- Use the `run_in_background: false` default to ensure agent completion before proceeding

### Work Package Identification

A task is parallelizable if:
1. It has no dependency on another concurrent task's output
2. It writes to different files than concurrent tasks
3. It represents a distinct domain boundary

A task must be sequential if:
1. It depends on the output of a prior task
2. It modifies shared configuration or documentation
3. It requires a decision from the Commander before proceeding

---

## Hooks Configuration

Hooks automate quality gate enforcement and session logging. They are configured in `.claude/settings.json`.

### SessionStart Hook: OS Boot

Fires when Claude Code starts a session in the superarchitect directory.

**Purpose:** Load OS context, verify OS integrity, display active configuration.

**Behavior:**
1. Print OS banner and version
2. Verify all required OS files exist
3. Load and display the active persona
4. Show available slash commands

```json
{
  "hooks": {
    "SessionStart": [
      {
        "type": "command",
        "command": "echo '=== SuperArchitect OS v1.0 ===' && echo 'Booting OS...' && ls /home/user/superarchitect/os/manifest.md > /dev/null 2>&1 && echo 'OS manifest: OK' || echo 'WARNING: OS manifest missing'"
      }
    ]
  }
}
```

### PreToolUse Hook: Write Quality Gate

Fires before every Write tool invocation.

**Purpose:** Log file creation intent and prevent overwriting existing files without explicit intent.

```json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Write",
        "type": "command",
        "command": "echo \"[$(date -u +%Y-%m-%dT%H:%M:%SZ)] PRE-WRITE: $CLAUDE_TOOL_INPUT_FILE_PATH\" >> /tmp/superarchitect-session.log"
      }
    ]
  }
}
```

### PostToolUse Hook: Agent Completion Log

Fires after every Agent tool invocation completes.

**Purpose:** Log team completion for session audit trail.

```json
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Agent",
        "type": "command",
        "command": "echo \"[$(date -u +%Y-%m-%dT%H:%M:%SZ)] AGENT COMPLETED: Task logged\" >> /tmp/superarchitect-session.log"
      }
    ]
  }
}
```

---

## Custom Slash Commands

Slash commands are defined as markdown files in `.claude/commands/`. Each file's content is the prompt template for the command.

### Available Commands

| Command | File | Purpose |
|---|---|---|
| `/build-system` | `.claude/commands/build-system.md` | Activates new-system workflow |
| `/audit-system` | `.claude/commands/audit-system.md` | Activates audit workflow |
| `/spawn-team` | `.claude/commands/spawn-team.md` | Manually invokes any specialist team |

### How Commands Work in Claude Code

1. User types `/build-system` in Claude Code
2. Claude Code reads `.claude/commands/build-system.md`
3. The file's content becomes the active prompt/instruction set
4. Claude executes the workflow defined in the command file
5. The user's input (everything after the command name) is passed as `$ARGUMENTS`

---

## Settings

Recommended Claude Code settings for SuperArchitect OS operation, configured in `.claude/settings.json`:

```json
{
  "permissions": {
    "allow": [
      "Bash(ls:*)",
      "Bash(mkdir:*)",
      "Bash(git:*)",
      "Bash(npm:*)",
      "Bash(python:*)",
      "Bash(echo:*)",
      "Bash(cat:*)",
      "Write(*)",
      "Edit(*)",
      "Read(*)",
      "Grep(*)",
      "Glob(*)",
      "Agent(*)"
    ]
  },
  "env": {
    "SUPERARCHITECT_OS_ROOT": "/home/user/superarchitect",
    "SUPERARCHITECT_LOG": "/tmp/superarchitect-session.log",
    "SUPERARCHITECT_VERSION": "1.0"
  }
}
```

**Key settings rationale:**
- Agent tool is explicitly allowed: enables multi-agent spawning
- Git commands are allowed: enables phase-completion commits
- Write/Edit/Read are unrestricted: OS must be able to create all system files
- Environment variables expose OS root path for consistent file resolution

---

## Memory Management

Claude Code maintains context through `CLAUDE.md` files in the directory hierarchy. The SuperArchitect OS uses this to scope memory appropriately.

### Context Scope Rules

- **Root CLAUDE.md**: Always loaded. Contains OS-wide context and persona.
- **Workspace CLAUDE.md**: Loaded when working in a project workspace. Contains project context.
- **Team CLAUDE.md**: Loaded by spawned agents for that team. Contains team-specific context.

### What Belongs in Each Level

**Root level CLAUDE.md:**
- OS version and active standards
- Default persona (Commander)
- Available slash commands
- Global quality thresholds

**Workspace level CLAUDE.md:**
- Project name and stage
- System description (one paragraph)
- Active workflow phase
- Links to project-specific architecture docs

**Team level CLAUDE.md:**
- Team persona details
- Domain boundary description
- Input/output conventions
- Team-specific quality requirements

### Context Window Management

When context windows are large, agents should:
1. Read only the sections of documents relevant to the current task
2. Use Grep to find specific content rather than loading full files
3. Summarize completed work in the workspace CLAUDE.md to compress prior context
4. Delegate subtasks to spawned agents rather than accumulating context in one agent

---

## Claude Code Best Practices for the SuperArchitect OS

### Always Read Architecture Docs Before Implementing

Before writing any implementation code, the implementing agent must:
1. Read `os/standards/architecture.md` — verify the approach follows architecture commandments
2. Read any existing ADRs relevant to the domain
3. Read the system's architecture documentation at Level 1 and Level 2

This prevents implementation that conflicts with established architectural decisions.

### Use Agent Tool for Parallelizable Work Packages

When the Commander identifies work that can be done independently:
- Do not do it sequentially in a single context
- Spawn parallel agents using the Agent tool
- Each agent gets a bounded prompt with clear inputs and outputs
- This reduces total build time and keeps each agent's context focused

### Commit After Each Phase Completion

At the end of each workflow phase:
```bash
git add [specific files changed in this phase]
git commit -m "Phase [N]: [Phase name] complete — [one-line summary]"
```

Benefits:
- Creates a rollback point for each phase
- Makes progress visible to human reviewers
- Scopes commits to logical units of work
- Enables parallel branches for experimental approaches

### Use TodoWrite to Track Cross-Phase Progress

When a multi-phase workflow is in progress, maintain a TodoWrite list that:
- Shows completed phases (checked)
- Shows the current active phase
- Shows upcoming phases with their key dependencies
- Includes any blocking issues discovered during the current phase

This creates a persistent progress tracker that survives context switches.

### Error Handling Protocol

When an agent encounters an error:
1. Do not guess or try random fixes
2. Describe the error precisely (exact message, file, line)
3. Read the relevant documentation to understand the expected behavior
4. If the issue is environmental (missing dependency, wrong config), surface it to the Commander
5. If the issue is a design conflict, escalate to the architect agent
6. Never silently skip a failing step

### Quality Gate Enforcement

Before marking any phase complete:
1. Run the phase's quality gates (see `os/standards/quality.md`)
2. If any gate fails, fix it before proceeding
3. Document any deliberately accepted failures with a rationale
4. The Commander does not advance the workflow until all gates are green
