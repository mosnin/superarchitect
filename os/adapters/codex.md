# OpenAI Codex Runtime Adapter — SuperArchitect OS

**Version:** 1.0
**Owner:** SuperArchitect OS Core Team
**Last Updated:** 2026-03-28
**Audience:** Operators running the SuperArchitect OS via OpenAI Codex / GPT-4o API

---

## Adapter Purpose

This adapter enables the SuperArchitect OS to run on OpenAI Codex and GPT-4o models via the OpenAI API. While the Claude Code adapter is the native runtime, this adapter provides a compatibility layer so the same workflows, standards, and team structures can be orchestrated using OpenAI models.

The key difference: Claude Code natively reads `CLAUDE.md` files and executes slash commands as part of the runtime. Codex requires the operator to explicitly load OS context into the prompt. This adapter specifies exactly how to do that.

**Use this adapter when:**
- Operating in environments where Claude Code is not available
- Comparing outputs from Claude and Codex for a given task
- Using the OpenAI Assistants API or Responses API to build an agentic system on top of this OS
- Integrating SuperArchitect OS workflows into a pipeline that uses GPT-4o function calling

---

## Prompt Engineering for Codex

Unlike Claude Code (which loads `CLAUDE.md` files automatically), Codex requires the OS context to be explicitly included in the system prompt or the first user message.

### The Three-Layer Prompt Architecture

Every Codex invocation using this OS uses a three-layer prompt:

**Layer 1: OS System Prompt** — Who the agent is and what OS it runs on
**Layer 2: Context Load** — The relevant OS files for this specific task
**Layer 3: Task Instructions** — The specific work to be done

This maps to the standard OpenAI messages format:

```python
messages = [
    {"role": "system", "content": SYSTEM_PROMPT},      # Layer 1: OS identity
    {"role": "user", "content": CONTEXT_LOAD},          # Layer 2: OS context
    {"role": "assistant", "content": "Context loaded. Ready to proceed."},  # Acknowledgment
    {"role": "user", "content": TASK_INSTRUCTIONS}      # Layer 3: Task
]
```

### Context Loading Discipline

Every Codex session must:
1. Include the system prompt (see below) in every API call
2. Include only the OS files relevant to the current workflow phase — not everything
3. Load persona files for the active team
4. Load standards files relevant to the task type

Do not load all OS files in every request. This wastes context window and degrades focus.

---

## System Prompt Template

Use this system prompt as the foundation for every SuperArchitect OS invocation with Codex:

```
You are an agent in the SuperArchitect Agentic OS — a system of structured markdown files
that acts as an operating system for AI agents building world-class software systems.

## Your Identity
[PASTE ACTIVE PERSONA CONTENT HERE — e.g., contents of os/teams/architect/persona.md]

## Operating Principles
You operate under the following non-negotiable principles:

1. QUALITY FIRST: Every output must meet the quality standards defined in os/standards/quality.md
2. ARCHITECTURE MATTERS: Every structural decision follows the 12 Architecture Commandments
3. EXPLICIT OVER IMPLICIT: State assumptions, document decisions, never guess silently
4. PHASE DISCIPLINE: Complete one workflow phase fully before beginning the next
5. VERIFIABLE OUTPUTS: Every output you produce must be checkable against a defined standard

## Tool Use
You have access to the following tools:
[LIST TOOLS CONFIGURED FOR THIS SESSION]

## Output Format
All documents you create must conform to the documentation standards in
os/standards/documentation.md — including required headers, writing standards, and structure.

## Session Context
Project: [PROJECT NAME]
Current Phase: [WORKFLOW PHASE]
Active Team: [TEAM NAME]
```

---

## Context Loading Strategy

For each workflow phase, load only the relevant OS files. This table maps phases to required context:

| Workflow Phase | Required OS Files |
|---|---|
| Requirements Gathering | `os/workflows/new-system.md` (phase 1 section), `os/standards/quality.md` (NFR section) |
| Architecture Design | `os/standards/architecture.md`, `os/knowledge/patterns/design-patterns.md`, `os/knowledge/anti-patterns.md`, relevant ADR templates |
| Security Review | `os/standards/quality.md` (Security section), `os/teams/security/persona.md` |
| Implementation | `os/standards/quality.md` (Code Review section), `os/standards/architecture.md` (Boundaries section) |
| Documentation | `os/standards/documentation.md`, project architecture docs |
| Deployment | `os/standards/quality.md` (Observability section), relevant runbook templates |
| Audit | `os/standards/quality.md` (full), `os/standards/architecture.md` (full), `os/knowledge/anti-patterns.md` |

### How to Load Files in a Python Orchestrator

```python
import os

def load_os_file(relative_path: str) -> str:
    """Load an OS context file into a string for prompt inclusion."""
    base = os.environ.get("SUPERARCHITECT_OS_ROOT", "/home/user/superarchitect")
    full_path = os.path.join(base, relative_path)
    with open(full_path, "r") as f:
        return f.read()

def build_context_block(files: list[str]) -> str:
    """Build a context block from multiple OS files."""
    parts = []
    for f in files:
        content = load_os_file(f)
        parts.append(f"--- BEGIN: {f} ---\n{content}\n--- END: {f} ---")
    return "\n\n".join(parts)

# Example: Loading context for Architecture phase
architecture_context = build_context_block([
    "os/standards/architecture.md",
    "os/knowledge/patterns/design-patterns.md",
    "os/knowledge/anti-patterns.md"
])
```

### Selective File Sections

For very large standards files, load only the relevant section by extracting markdown headers:

```python
def load_section(file_path: str, section_header: str) -> str:
    """Extract a specific section from a markdown file."""
    content = load_os_file(file_path)
    lines = content.split("\n")
    in_section = False
    section_lines = []
    for line in lines:
        if line.startswith("## ") and section_header in line:
            in_section = True
        elif line.startswith("## ") and in_section:
            break
        if in_section:
            section_lines.append(line)
    return "\n".join(section_lines)
```

---

## Tool Use Differences

### Codex Tool Use vs. Claude Code Tool Use

Claude Code has native filesystem tools (Read, Write, Edit, Bash, Grep, Glob, Agent). Codex uses the OpenAI function calling interface. Map the OS tool use patterns to Codex as follows:

| Claude Code Tool | Codex Equivalent | Notes |
|---|---|---|
| `Read(file_path)` | `read_file(path)` custom function | Must be implemented by operator |
| `Write(file_path, content)` | `write_file(path, content)` custom function | Must be implemented by operator |
| `Edit(old_string, new_string)` | `edit_file(path, old, new)` custom function | Must be implemented by operator |
| `Bash(command)` | `run_command(command)` custom function | Implement with subprocess sandbox |
| `Grep(pattern, path)` | `search_files(pattern, directory)` custom function | Use ripgrep via subprocess |
| `Glob(pattern)` | `find_files(pattern)` custom function | Use pathlib glob |
| `Agent(prompt)` | Recursive API call with new system prompt | Implement via orchestration layer |

### Function Definitions for OpenAI API

```python
SUPERARCHITECT_TOOLS = [
    {
        "type": "function",
        "function": {
            "name": "read_file",
            "description": "Read the contents of a file. Always use this before editing.",
            "parameters": {
                "type": "object",
                "properties": {
                    "path": {"type": "string", "description": "Absolute file path"}
                },
                "required": ["path"]
            }
        }
    },
    {
        "type": "function",
        "function": {
            "name": "write_file",
            "description": "Create a new file with the given content. Do not use to overwrite existing files.",
            "parameters": {
                "type": "object",
                "properties": {
                    "path": {"type": "string", "description": "Absolute file path"},
                    "content": {"type": "string", "description": "Full file content"}
                },
                "required": ["path", "content"]
            }
        }
    },
    {
        "type": "function",
        "function": {
            "name": "run_command",
            "description": "Run a shell command and return output. Use for git, npm, pytest, etc.",
            "parameters": {
                "type": "object",
                "properties": {
                    "command": {"type": "string", "description": "Shell command to run"},
                    "working_directory": {"type": "string", "description": "Directory to run in"}
                },
                "required": ["command"]
            }
        }
    }
]
```

---

## Multi-Agent with Codex

Implementing the SuperArchitect OS's multi-agent team structure with Codex requires an orchestration layer — the framework that plays the role of the Claude Code Agent tool.

### Architecture: Codex Orchestrator

```
User Request
     │
     ▼
Orchestrator (Python / Node.js)
     │
     ├── Commander Agent (GPT-4o)
     │   ├── Receives: full system prompt + task
     │   ├── Returns: structured work plan with team assignments
     │   │
     │   ├── ── Thread Pool ──────────────────────────────
     │   │   ├── Architect Agent (GPT-4o, parallel)
     │   │   ├── Security Agent (GPT-4o, parallel)
     │   │   └── Data Agent (GPT-4o, parallel)
     │   │   ─────────────────────────────────────────────
     │   │
     │   └── Aggregates team outputs → Final deliverable
     │
     ▼
Output Files Written to Workspace
```

### Python Orchestrator Template

```python
from openai import OpenAI
from concurrent.futures import ThreadPoolExecutor, as_completed

client = OpenAI()

def run_team_agent(team_name: str, persona: str, task: str, context: str) -> dict:
    """Run a single specialist team agent and return its output."""
    system_prompt = build_team_system_prompt(team_name, persona)
    messages = [
        {"role": "system", "content": system_prompt},
        {"role": "user", "content": f"Context:\n{context}\n\nTask:\n{task}"}
    ]
    response = client.chat.completions.create(
        model="gpt-4o",
        messages=messages,
        tools=SUPERARCHITECT_TOOLS,
        tool_choice="auto",
        temperature=0.2
    )
    return {"team": team_name, "response": response}

def spawn_parallel_teams(team_tasks: list[dict]) -> list[dict]:
    """Spawn multiple team agents in parallel."""
    results = []
    with ThreadPoolExecutor(max_workers=len(team_tasks)) as executor:
        futures = {
            executor.submit(
                run_team_agent,
                t["team"], t["persona"], t["task"], t["context"]
            ): t["team"]
            for t in team_tasks
        }
        for future in as_completed(futures):
            results.append(future.result())
    return results
```

---

## Codex-Specific Best Practices

### Temperature Settings

| Task Type | Recommended Temperature |
|---|---|
| Architecture design, ADRs | 0.1 — low temperature for precise, consistent decisions |
| Code generation | 0.2 — slight creativity for idiomatic code |
| Documentation writing | 0.3 — some variety for natural prose |
| Requirements gathering | 0.4 — allow for generative thinking |
| Creative problem solving | 0.5 — balance creativity with coherence |

Never use temperature > 0.7 for SuperArchitect OS tasks. Higher temperatures produce inconsistent outputs that fail quality gates.

### Token Budget Management

| Phase | Recommended Max Tokens | Notes |
|---|---|---|
| Requirements | 4,096 | Requirements should be concise |
| Architecture design | 8,192 | ADRs and diagrams need room |
| Implementation plan | 8,192 | File-by-file plan |
| Code generation (per file) | 4,096 | One file per request |
| Documentation | 4,096 | One doc per request |
| Full audit report | 16,384 | Comprehensive findings |

### Few-Shot Examples

Include 1–2 examples of the expected output format at the beginning of complex requests. This dramatically improves output consistency.

```python
def add_few_shot_example(messages: list, example_input: str, example_output: str) -> list:
    """Add a few-shot example to the conversation."""
    return messages + [
        {"role": "user", "content": example_input},
        {"role": "assistant", "content": example_output}
    ]
```

### Context Refresh Strategy

Codex does not have persistent memory. For multi-phase workflows:
1. At the end of each phase, write a structured summary to a markdown file
2. At the start of the next phase, load the summary (not the full prior conversation)
3. Include phase outputs (file paths + brief descriptions) in the next phase's context
4. Never rely on Codex "remembering" something from earlier in a long conversation

### Rate Limit and Retry Policy

```python
import time
from openai import RateLimitError

def call_with_retry(client, **kwargs, max_retries=3):
    """Call OpenAI API with exponential backoff on rate limits."""
    for attempt in range(max_retries):
        try:
            return client.chat.completions.create(**kwargs)
        except RateLimitError:
            if attempt == max_retries - 1:
                raise
            wait_time = 2 ** attempt  # 1s, 2s, 4s
            time.sleep(wait_time)
```

---

## Compatibility Layer

The same workflow files work for both Claude Code and Codex because they are plain markdown. The compatibility layer is in how they are loaded, not in their content.

### Workflow File Compatibility

All workflow files in `os/workflows/` are written in plain markdown with structured phases. They are:
- **Claude Code**: Automatically loaded when referenced in CLAUDE.md or a slash command
- **Codex**: Explicitly included in the context load for each phase

### Standards File Compatibility

Standards in `os/standards/` are format-agnostic. They define what must be true, not how the agent achieves it. Both Claude and Codex can reason from the same standards files.

### Persona File Compatibility

Persona files in `os/teams/*/persona.md` work for both runtimes:
- **Claude Code**: Loaded via subdirectory CLAUDE.md files automatically
- **Codex**: Included in the system prompt as the "Your Identity" section

### Differences That Require Adapter-Specific Handling

| Concern | Claude Code | Codex |
|---|---|---|
| File system access | Native tools (Read/Write/Edit) | Custom functions required |
| Memory/context persistence | CLAUDE.md hierarchy | Explicit summary files |
| Multi-agent spawning | Native Agent tool | Python orchestration layer |
| Slash commands | Native .claude/commands/ | Not supported; use API workflows |
| Quality gate hooks | settings.json hooks | Pipeline-level validation |
| Session state | Managed by CLI | Managed by orchestrator |

### Migration Checklist: Claude Code → Codex

When moving a workflow from Claude Code to Codex:

- [ ] Replace all CLAUDE.md auto-loading with explicit context injection
- [ ] Implement custom tool functions for Read, Write, Edit, Bash, Grep, Glob
- [ ] Replace Agent tool calls with orchestrator.spawn_parallel_teams()
- [ ] Replace slash commands with API workflow functions
- [ ] Add explicit phase-summary writes for cross-phase context
- [ ] Set temperature per task type (see table above)
- [ ] Add retry logic with exponential backoff
- [ ] Add structured output parsing (Codex may deviate from format without explicit schema)
