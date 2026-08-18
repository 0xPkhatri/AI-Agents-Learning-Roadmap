# 00 — Roadmap Review

- **Reviewed:** 2026-08-15
- **Overall assessment:** Strong and technically current
- **Rating:** Approximately 8.5/10

## Verdict

The roadmap is suitable for learning AI-agent engineering. Its central principle is sound:

> Build the agent loop yourself before depending on agent frameworks.

Its principal weakness is not technical correctness but execution scope. Reliability practices appear too late, four frameworks are given more attention than necessary, and the capstone is large enough to become a production platform rather than one learning project.

## Strongest parts

- Clear distinction between workflows, agents, and hybrid systems
- Direct API → tools → manual loop → explicit state progression
- Frameworks delayed until after foundational concepts
- Serious treatment of security, evaluation, reliability, and production
- Project-based checkpoints
- A capstone that integrates the complete system

## Current documentation check

The roadmap's framework and protocol descriptions were broadly current when reviewed:

- LangGraph distinguishes predetermined workflows from dynamic agents and describes itself as low-level infrastructure for long-running, stateful agents.
- MCP's July 2026 specification introduced a stateless protocol core.
- CrewAI documentation emphasizes agents, crews, and flows.
- AutoGen describes AgentChat as its higher-level API for multi-agent applications.
- Current official OpenAI guidance recommends the Responses API for reasoning, tool calling, and multi-turn workflows. It also stresses relevant tool exposure, explicit approval boundaries, and evaluation on representative tasks.

Because these products and protocols change, their details must be checked again when their respective topics are studied.

## Recommended adjustments

### 1. Add Phase 0 — Software prerequisites

Study or refresh:

- Python fundamentals
- Type hints and Pydantic
- HTTP, REST, and JSON
- Async programming
- Git
- SQL basics
- Unit testing
- Environment variables and secrets

### 2. Introduce reliability earlier

Immediately after the first manual agent loop, add:

- tool error handling
- step limits
- structured logging
- permission boundaries
- 10–20 evaluation cases

Topics 40–48 should deepen these skills rather than introduce them for the first time.

### 3. Treat the agent equation as a checklist

```text
Agent = Model + Instructions + State + Tools + Control Loop
      + Environment + Memory + Safety + Evaluation
```

This is a useful design checklist, not a strict definition. Not every agent requires long-term memory, RAG, planning, or multiple agents.

### 4. Do not study every framework equally deeply

Recommended approach:

- Learn either OpenAI Agents SDK or LangGraph deeply.
- Learn one multi-agent framework as a comparison if multi-agent work is relevant.
- Survey the remaining frameworks conceptually.

Framework APIs change faster than foundational agent concepts.

### 5. Make multi-agent systems conditional

Use multiple agents only when a task divides naturally into specialized, independent contexts and the benefits exceed the coordination overhead.

First establish that one well-designed agent is insufficient. Multi-agent systems introduce additional communication, context, latency, cost, and evaluation problems.

### 6. Build reliability vertically

Every project should contain a small version of:

```text
Functionality
+ Tests
+ Tracing
+ Permissions
+ Cost measurement
```

Later phases should improve these capabilities rather than bolt them onto a completed system.

### 7. Build the capstone as releases

The full capstone specification is too large for one implementation pass. Follow the existing milestones as separate working releases. Each release should be usable and evaluated before adding the next subsystem.

## Recommended execution order

```text
Software prerequisites
        ↓
Agent concepts and direct LLM APIs
        ↓
Structured outputs and tool calling
        ↓
Manual agent loop
        ↓
State + context + basic tests/tracing/safety
        ↓
RAG and memory
        ↓
Planning + browser/code/database environments
        ↓
Reliability and security
        ↓
Multi-agent systems, only if justified
        ↓
One framework deeply
        ↓
Deployment, scaling, and capstone
```

## Practical rule

Use the roadmap as a curriculum map, not as a requirement to master all 57 topics before building useful systems.

Build continuously, introduce safety and evaluation early, and avoid learning four frameworks at equal depth.

## Sources

- [LangGraph: Workflows and agents](https://docs.langchain.com/oss/python/langgraph/workflows-agents)
- [LangGraph overview](https://docs.langchain.com/oss/python/langgraph/overview)
- [MCP 2026-07-28 specification release](https://blog.modelcontextprotocol.io/posts/2026-07-28/)
- [CrewAI documentation](https://docs.crewai.com/index)
- [AutoGen AgentChat](https://microsoft.github.io/autogen/stable/user-guide/agentchat-user-guide/index.html)
- [Official OpenAI model guidance](https://developers.openai.com/api/docs/guides/latest-model)

