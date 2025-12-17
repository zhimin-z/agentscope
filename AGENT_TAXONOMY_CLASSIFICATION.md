# AgentScope: SWE Agent Taxonomy Classification Report

This report classifies the AgentScope framework according to the SWE Agent Taxonomy across multiple dimensions: agent architecture, cognitive strategies (workflow, planning, reasoning), memory systems, and tool capabilities.

---

## 1. Agent Architecture

### Labels
SINGLE-AGENT, MULTI-AGENT-PARALLEL, MULTI-AGENT-SEQUENTIAL, MULTI-AGENT-COLLABORATIVE, MULTI-AGENT-HIERARCHICAL

### Justification

**SINGLE-AGENT**: AgentScope provides the `ReActAgent` base class that can work independently on tasks without requiring multi-agent coordination. Examples show single agents interacting with users directly (e.g., `examples/agent/react_agent/main.py`).

**MULTI-AGENT-PARALLEL**: The framework supports parallel agent execution through `fanout_pipeline` and `asyncio.gather()`. The `examples/workflows/multiagent_concurrent/main.py` demonstrates concurrent execution where multiple agents work simultaneously with results collected at the end.

**MULTI-AGENT-SEQUENTIAL**: The `sequential_pipeline` function in `src/agentscope/pipeline/_functional.py` enables agents to work in sequence, where the output of one agent becomes the input to the next agent in a pipeline.

**MULTI-AGENT-COLLABORATIVE**: The `MsgHub` class (`src/agentscope/pipeline/_msghub.py`) facilitates collaborative multi-agent systems where agents broadcast messages to each other. The debate example (`examples/workflows/multiagent_debate/main.py`) shows agents presenting different perspectives and a moderator coordinating the discussion.

**MULTI-AGENT-HIERARCHICAL**: The meta planner agent example (`examples/agent/meta_planner_agent/main.py`) demonstrates hierarchical coordination where a manager agent decomposes tasks and delegates to worker agents using the `create_worker` tool function, showing clear manager-worker structure.

---

## 2. Agent Cognition

### 2.1 Workflow Patterns

#### Labels
ITERATIVE

#### Justification

**ITERATIVE**: The `ReActAgent` implements a continuous reasoning-acting loop (lines 305-403 in `_react_agent.py`) with `max_iters` parameter controlling iterations. The agent executes a Thought→Action→Observation cycle where it reasons about the next step, executes tools, observes results, and repeats until task completion or maximum iterations reached.

### 2.2 Planning Strategies

#### Labels
HIERARCHICAL, INTERACTIVE, REPLANNING, RETRIEVAL-AUGMENTED-PLANNING

#### Justification

**HIERARCHICAL**: The `PlanNotebook` class (`src/agentscope/plan/_plan_notebook.py`) supports hierarchical task decomposition with subtasks. The system breaks down complex tasks into manageable subtasks with structured tracking of progress and dependencies.

**INTERACTIVE**: The framework natively supports realtime steering with `handle_interrupt` methods in agents, allowing human-in-the-loop planning. The `UserAgent` class enables interactive planning where agents can request user guidance at decision points.

**REPLANNING**: The `PlanNotebook` provides `revise_current_plan` tool function (line 42 in `_plan_notebook.py`) allowing agents to dynamically adjust plans when conditions change, supporting both plan revision and surgical adjustments to subtasks.

**RETRIEVAL-AUGMENTED-PLANNING**: The framework integrates long-term memory retrieval before planning. The `_retrieve_from_long_term_memory` method (line 277 in `_react_agent.py`) retrieves relevant past experiences that inform current planning, using successful past plans as templates.

### 2.3 Reasoning Techniques

#### Labels
CHAIN-OF-THOUGHT, REACT, TEST-DRIVEN, PROGRAM-AIDED, CASE-BASED, TOOL-COMPOSITION

#### Justification

**CHAIN-OF-THOUGHT**: The ReActAgent's reasoning process breaks problems into explicit steps. The system prompt formatting and multi-turn conversation structure encourage step-by-step logical reasoning visible in message histories.

**REACT**: The core `ReActAgent` class (`_react_agent.py`) implements the ReAct paradigm with explicit separation of reasoning and acting phases. The agent interleaves thought (via `_reasoning` method), action (via `_acting` method), and observation in iterative cycles.

**TEST-DRIVEN**: Structured output validation is supported through Pydantic models (lines 284-300 in `_react_agent.py`). The agent ensures solutions meet required conditions by validating against structured schemas before considering tasks complete.

**PROGRAM-AIDED**: The toolkit includes `execute_python_code` and `execute_shell_command` functions (`src/agentscope/tool/_coding/`), allowing agents to generate and execute code for calculations, data processing, and complex logic operations.

**CASE-BASED**: The long-term memory system (`_long_term_memory_base.py`) enables case-based reasoning through the `retrieve_from_memory` method, allowing agents to recall similar past situations and adapt successful strategies to new problems.

**TOOL-COMPOSITION**: The `Toolkit` class supports chaining multiple tools together. The parallel and sequential tool execution modes (lines 313-324 in `_react_agent.py`) enable agents to compose tools strategically, using outputs from one tool as inputs to another.

---

## 3. Agent Memory

### Labels
EPHEMERAL, PERSISTENT, EPISODIC, SEMANTIC, VECTOR-DB, MULTI-TIERED, AGENTIC, SHARED-MEMORY, MESSAGE-PASSING

### Justification

**EPHEMERAL**: The `InMemoryMemory` class (`_in_memory_memory.py`) stores messages in a Python list during a single session without persistence across sessions, serving as working memory for current interactions.

**PERSISTENT**: The `LongTermMemoryBase` and its implementations (e.g., `_mem0_long_term_memory.py`, `_reme/` directory) provide long-term memory that persists across multiple sessions and interactions, storing knowledge that endures over time.

**EPISODIC**: The long-term memory system stores specific past events with temporal context. The `record` method in `LongTermMemoryBase` captures what happened, when, and in what context, creating a time-stamped log of interactions.

**SEMANTIC**: The knowledge base system supports semantic memory through structured fact storage. The RAG module (`src/agentscope/rag/_knowledge_base.py`) stores conceptual knowledge and entity relationships that enable reasoning about how concepts relate.

**VECTOR-DB**: The embedding-based retrieval system uses vector databases for semantic search. The `VDBStoreBase` in the RAG module (`_store` directory) and `EmbeddingModelBase` (`src/agentscope/embedding/`) convert information into dense embeddings for similarity-based retrieval.

**MULTI-TIERED**: The framework implements a hierarchical memory system with fast-access working memory (`InMemoryMemory`) for current context and slower long-term storage (`LongTermMemoryBase`) for historical information, automatically managing information between tiers.

**AGENTIC**: The long-term memory supports agent-controlled memory management through `record_to_memory` and `retrieve_from_memory` tool functions (lines 172-179 in `_react_agent.py`). The `long_term_memory_mode="agent_control"` setting enables autonomous memory organization.

**SHARED-MEMORY**: The `MsgHub` class implements shared memory accessible to multiple agents. All participants in a hub can read from and contribute to the shared message space, enabling coordination through a common knowledge pool.

**MESSAGE-PASSING**: Agents maintain private memories but communicate through explicit message passing. The `observe` method and subscriber system allow agents to send messages containing relevant information to other agents without exposing all internal state.

---

## 4. Agent Tool

### Labels
FILE-MANAGEMENT, CODE-EDITING, STRUCTURAL-RETRIEVAL, EMBEDDING-RETRIEVAL, VERSION-CONTROL, PYTHON-TOOLS, TESTING-TOOLS, WEB-TOOLS, SHELL-SCRIPTING, TEXT-PROCESSING, ENVIRONMENT-VARIABLES, NAVIGATION-TOOLS

### Justification

**FILE-MANAGEMENT**: The framework provides `view_text_file`, `write_text_file`, and `insert_text_file` functions (`src/agentscope/tool/_text_file/`) for basic file system operations including reading, creating, and modifying files.

**CODE-EDITING**: The text file tools support precise code modifications with insertion and replacement capabilities. The `write_text_file` and `insert_text_file` functions enable viewing and modifying source code files with line-based operations.

**STRUCTURAL-RETRIEVAL**: While not implementing AST-based analysis, the framework provides pattern-based code search through integration with external tools. The MCP (Model Context Protocol) integration allows connection to code analysis servers.

**EMBEDDING-RETRIEVAL**: The comprehensive embedding module (`src/agentscope/embedding/`) includes OpenAI, DashScope, Gemini, and Ollama embedding models. The RAG knowledge base uses vector similarity search (`_knowledge_base.py`) for semantic code retrieval beyond keyword matching.

**VERSION-CONTROL**: The framework integrates with git through shell command execution. The `execute_shell_command` tool allows git operations for version history inspection and code review workflows.

**PYTHON-TOOLS**: The `execute_python_code` function (`src/agentscope/tool/_coding/_python.py`) provides Python runtime execution capabilities, supporting code execution and package management operations.

**TESTING-TOOLS**: Through shell command execution, agents can run test frameworks like pytest. The training examples (`examples/training/react_agent/`) demonstrate test-driven development workflows.

**WEB-TOOLS**: The multi-modality tools (`src/agentscope/tool/_multi_modality/`) include web-related capabilities. The MCP integration supports HTTP/SSE transport protocols for web service interaction (lines 102-104 in README.md).

**SHELL-SCRIPTING**: The `execute_shell_command` function (`_coding/_shell.py`) provides comprehensive shell scripting capabilities, enabling automation through bash commands with support for control structures and environment management.

**TEXT-PROCESSING**: The text file tools support text manipulation operations. The message formatting system processes and transforms text content across different formats for various model providers.

**ENVIRONMENT-VARIABLES**: The framework uses environment variables for configuration (e.g., `DASHSCOPE_API_KEY` in examples). The shell execution tools allow agents to read and set environment variables for runtime configuration.

**NAVIGATION-TOOLS**: The file management tools support directory navigation and exploration. The `view_text_file` function with path parameters enables workspace traversal and codebase exploration.

---

## Novel Features

### Agent Skill System

**Definition**: A dynamic capability extension system where agents can load specialized instruction sets, scripts, and resources on-demand to improve performance on specific tasks. Each skill is a self-contained package with metadata (`SKILL.md`) describing usage patterns.

**Evidence**: The `Toolkit` class (`_toolkit.py` lines 77-88) implements agent skill registration and management. Skills are organized in folders with `SKILL.md` files, and the framework automatically generates skill prompts that agents can load dynamically to enhance specialized task performance.

**Distinction from existing labels**: Unlike TOOL-EXPLORATION (which focuses on discovering available tools), the agent skill system provides structured domain expertise packages that combine instructions, context, and potentially multiple coordinated tools. Skills represent higher-level capability modules rather than individual tool functions.

### Agentic Tool Management

**Definition**: Self-controlled tool activation and deactivation where agents autonomously decide which tools to equip based on task requirements, using meta-level tool functions to manage their own toolkit configuration.

**Evidence**: The `enable_meta_tool` parameter (line 100 in `_react_agent.py`) registers the `reset_equipped_tools` function, allowing agents to dynamically modify their available tools. The group-wise tool management system (lines 206-219) enables agents to activate/deactivate entire tool groups through meta-tools.

**Distinction from existing labels**: TOOL-EXPLORATION focuses on discovering tools, while agentic tool management involves autonomous decisions about which tools to keep active. This meta-cognitive capability allows agents to optimize their toolkit based on current context, reducing prompt overhead and improving focus.

### Realtime Steering with State Preservation

**Definition**: Native support for interrupting agent execution mid-process while preserving complete internal state, converting interruptions into observable events that agents can seamlessly integrate into their reasoning process.

**Evidence**: The `handle_interrupt` method in `AgentBase` and robust memory preservation ensure agents can resume after interruption. The framework treats interruptions as first-class events rather than errors (mentioned in README.md lines 243-250).

**Distinction from existing labels**: INTERACTIVE planning assumes planned pause points, while realtime steering allows interruption at any moment during execution with automatic state recovery. This enables more fluid human-agent collaboration without predetermined interaction points.

### Stateful MCP Integration

**Definition**: Support for both stateful and stateless Model Context Protocol (MCP) clients with fine-grained control over tool function lifecycles and connection management.

**Evidence**: The MCP module (`src/agentscope/mcp/`) includes `_stateful_client_base.py` and stateless variants. The framework supports streamable HTTP/SSE/StdIO transport with client- and function-level control (README.md lines 102-104).

**Distinction from existing labels**: Standard tool integration assumes stateless function calls. Stateful MCP support enables persistent connections with external services, maintaining context across multiple interactions and supporting streaming responses.

---

## Summary

AgentScope demonstrates a comprehensive multi-paradigm architecture supporting both single and multi-agent coordination patterns (parallel, sequential, collaborative, hierarchical). The framework implements iterative workflow with ReAct-based reasoning, hierarchical and interactive planning with replanning capabilities, and retrieval-augmented planning.

The cognitive system combines chain-of-thought reasoning, ReAct loops, program-aided execution, and tool composition. The memory architecture is sophisticated with ephemeral, persistent, episodic, and semantic layers, vector-based retrieval, multi-tiered organization, agentic control, and both shared-memory and message-passing modes.

Tool capabilities span file management, code editing, embedding-based retrieval, Python execution, shell scripting, and web tools. Novel features include an agent skill system for dynamic capability loading, agentic tool management for self-controlled toolkit configuration, realtime steering with state preservation, and stateful MCP integration for persistent external service connections.

This classification reveals AgentScope as a highly flexible, production-ready framework designed for building sophisticated LLM applications with strong emphasis on transparency, modularity, and agent autonomy.
