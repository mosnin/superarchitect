# Agent Framework Handoff Adapter

**Version:** 1.0
**Owner:** Foundation Team (Team 1)
**Last Updated:** 2026-03-29

---

## Purpose

This adapter enables SuperArchitect OS to integrate with external agent frameworks: LangChain, AutoGen, CrewAI, and similar orchestration systems. It defines how to translate SuperArchitect OS concepts into the native primitives of each framework.

---

## 1. Concept Mapping

### SuperArchitect OS -> Agent Framework Primitives

| SuperArchitect OS Concept | LangChain Equivalent | AutoGen Equivalent | CrewAI Equivalent |
|--------------------------|---------------------|-------------------|------------------|
| Commander Agent | AgentExecutor / Supervisor | GroupChatManager | Crew (manager) |
| Team Agent | Agent with tools | ConversableAgent | Agent with role |
| Team File (.md) | System prompt | system_message | backstory + goal |
| Workflow | Chain / Graph (LangGraph) | Conversation flow | Process (sequential/hierarchical) |
| Dispatch | Router / Conditional edges | Speaker selection | Task assignment |
| Quality Gate | Output parser + validator | Reply validation | Task callback |
| Standards file | Few-shot examples in prompt | Teachable agent memory | Agent context |
| Escalation | Human-in-the-loop node | Human proxy agent | Human input task |

---

## 2. LangChain / LangGraph Integration

### Architecture
```python
from langchain.agents import AgentExecutor
from langgraph.graph import StateGraph, END

# Define the SuperArchitect state
class SuperArchitectState(TypedDict):
    system_description: str
    phase: str
    requirements: Optional[dict]
    architecture: Optional[dict]
    security_review: Optional[dict]
    implementation: Optional[dict]
    qa_report: Optional[dict]
    artifacts: list[str]

# Create team agents
def create_team_agent(team_name: str, team_file_path: str) -> AgentExecutor:
    """Load a SuperArchitect team as a LangChain agent."""
    with open(team_file_path) as f:
        system_prompt = f.read()

    # Load supporting files (standards, patterns)
    supporting_context = load_team_supporting_files(team_name)

    llm = ChatAnthropic(model="claude-sonnet-4-20250514")
    tools = get_team_tools(team_name)  # File read/write, bash, etc.

    return AgentExecutor.from_agent_and_tools(
        agent=create_agent(llm, system_prompt + supporting_context, tools),
        tools=tools,
    )

# Build the workflow graph
def build_system_graph() -> StateGraph:
    graph = StateGraph(SuperArchitectState)

    # Add nodes for each phase
    graph.add_node("discovery", discovery_phase)
    graph.add_node("architecture", architecture_phase)
    graph.add_node("security", security_phase)
    graph.add_node("implementation", implementation_phase)
    graph.add_node("qa", qa_phase)
    graph.add_node("documentation", documentation_phase)
    graph.add_node("delivery", delivery_phase)

    # Add edges (workflow sequence)
    graph.add_edge("discovery", "architecture")
    graph.add_edge("architecture", "security")
    graph.add_edge("security", "implementation")
    graph.add_edge("implementation", "qa")
    graph.add_edge("qa", "documentation")
    graph.add_edge("documentation", "delivery")
    graph.add_edge("delivery", END)

    # Add quality gate conditional edges
    graph.add_conditional_edges(
        "qa",
        quality_gate_check,
        {"pass": "documentation", "fail": "implementation"}
    )

    graph.set_entry_point("discovery")
    return graph.compile()
```

### Tool Mapping
```python
# Map SuperArchitect capabilities to LangChain tools
tools = [
    FileReadTool(),           # Read OS files, codebase
    FileWriteTool(),          # Write code, documentation
    BashTool(),               # Run builds, tests, linters
    GrepTool(),               # Search codebase
    GlobTool(),               # Find files by pattern
]
```

---

## 3. AutoGen Integration

### Architecture
```python
import autogen

# Load team configurations from OS files
def create_autogen_agent(team_name: str, team_file_path: str) -> autogen.ConversableAgent:
    with open(team_file_path) as f:
        system_message = f.read()

    return autogen.AssistantAgent(
        name=team_name,
        system_message=system_message,
        llm_config={"model": "claude-sonnet-4-20250514"},
    )

# Create the Commander as GroupChatManager
commander = autogen.GroupChatManager(
    groupchat=autogen.GroupChat(
        agents=[
            create_autogen_agent("architect", "os/teams/architect/ARCHITECT.md"),
            create_autogen_agent("engineer", "os/teams/engineer/ENGINEER.md"),
            create_autogen_agent("security", "os/teams/security/SECURITY.md"),
            create_autogen_agent("qa", "os/teams/qa/QA.md"),
            create_autogen_agent("devops", "os/teams/devops/DEVOPS.md"),
            create_autogen_agent("product", "os/teams/product/PRODUCT.md"),
        ],
        messages=[],
        max_round=50,
        speaker_selection_method="auto",  # Commander selects speaker
    ),
    llm_config={"model": "claude-sonnet-4-20250514"},
)

# Human proxy for escalation
user_proxy = autogen.UserProxyAgent(
    name="human_operator",
    human_input_mode="TERMINATE",  # Only for escalation
    code_execution_config={"work_dir": "workspace"},
)

# Start the build
user_proxy.initiate_chat(
    commander,
    message="Build a SaaS platform for restaurant reservation management",
)
```

---

## 4. CrewAI Integration

### Architecture
```python
from crewai import Agent, Task, Crew, Process

# Load team as CrewAI agent
def create_crew_agent(team_name: str, team_file_path: str) -> Agent:
    with open(team_file_path) as f:
        backstory = f.read()

    role_map = {
        "architect": "System Architect",
        "engineer": "Senior Software Engineer",
        "security": "Security Engineer",
        "qa": "QA Engineer",
        "devops": "DevOps Engineer",
        "product": "Product Strategist",
        "designer": "UX Designer",
    }

    return Agent(
        role=role_map[team_name],
        goal=f"Execute {team_name} responsibilities per SuperArchitect OS standards",
        backstory=backstory,
        verbose=True,
        allow_delegation=True,
        tools=get_team_tools(team_name),
    )

# Define tasks from workflow phases
discovery_task = Task(
    description="Analyze system requirements and produce PRD",
    agent=product_agent,
    expected_output="Product Requirements Document with user stories and success metrics",
)

architecture_task = Task(
    description="Design system architecture based on requirements",
    agent=architect_agent,
    expected_output="Architecture blueprint with ADRs",
    context=[discovery_task],  # Dependency
)

# Create the crew (Commander equivalent)
crew = Crew(
    agents=[product_agent, architect_agent, security_agent, engineer_agent, qa_agent, devops_agent],
    tasks=[discovery_task, architecture_task, security_task, implementation_task, qa_task, docs_task],
    process=Process.sequential,  # Or Process.hierarchical for Commander-managed
    verbose=True,
)

result = crew.kickoff()
```

---

## 5. Generic Adapter Protocol

For any agent framework not listed above, implement these interfaces:

### Agent Interface
```
create_agent(team_file: str, tools: list) -> Agent
  - Load the team file as the agent's system prompt/persona
  - Attach the specified tools
  - Return an agent instance that can execute tasks

execute_task(agent: Agent, task: str, context: dict) -> Result
  - Send the task description to the agent
  - Provide context from previous phases
  - Return the agent's output
```

### Workflow Interface
```
create_workflow(workflow_file: str) -> Workflow
  - Parse the workflow file to extract phases, dependencies, and quality gates
  - Return a workflow graph

execute_workflow(workflow: Workflow, input: str) -> Result
  - Execute phases in order, respecting dependencies
  - Check quality gates between phases
  - Handle failures and escalation
  - Return the final output
```

### Quality Gate Interface
```
check_quality_gate(phase: str, output: Result, criteria: list) -> GateResult
  - Evaluate the output against the quality gate criteria
  - Return PASS, FAIL, or CONDITIONAL_PASS
  - Include specific findings for any failures
```

### Escalation Interface
```
escalate(reason: str, context: dict, options: list) -> Decision
  - Present the escalation to the human operator
  - Include context and recommended options
  - Return the human's decision
```

---

## 6. File Loading Strategy

When integrating with any framework, OS files should be loaded as follows:

**Priority 1 (always load):**
- `CLAUDE.md` — OS activation and principles
- `os/commander/COMMANDER.md` — Commander persona
- Target team file (e.g., `os/teams/architect/ARCHITECT.md`)

**Priority 2 (load for the active phase):**
- Team standards (e.g., `os/teams/architect/patterns.md`)
- Team templates (e.g., `os/teams/architect/templates/microservices.md`)
- Global standards (e.g., `os/standards/quality.md`, `os/standards/security.md`)

**Priority 3 (load on demand):**
- Knowledge base files
- Anti-patterns catalog
- Case studies

**Context Window Management:**
- If the framework has limited context, load only Priority 1 + the most relevant Priority 2 files
- Summarize large files if needed
- Use RAG (retrieval-augmented generation) for the knowledge base

---

*This adapter translates SuperArchitect OS capabilities into any agent framework. The OS is framework-agnostic by design — the intelligence lives in the Markdown files, not in the orchestration code.*
