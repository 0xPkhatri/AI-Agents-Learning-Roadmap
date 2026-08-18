# 01 — AI Agents: Detailed Study Guide

- **Phase:** 1 — Foundations
- **Topic:** AI Agents — Mental Model and Terminology
- **Status:** In progress
- **Source:** [ChatGPT learning conversation](https://chatgpt.com/c/6a801644-a558-83ee-a714-86bd9b7025fb) (`S13`)
- **Processed:** 2026-08-15

## Learning outcome

After this topic, I should be able to inspect an AI system and classify it as:

- a normal LLM application or chatbot;
- a RAG application;
- a deterministic workflow;
- an agent; or
- a hybrid system.

I should also be able to identify where decisions occur and explain the vocabulary used in agent engineering.

## The central definition

> An agent is a goal-directed software system in which an AI model participates in deciding what actions to take based on its current state and observations.

The most important architectural property is not merely using an LLM or calling a tool. It is that the system can repeatedly select actions based on intermediate results.

```text
Goal
  ↓
Observe current state
  ↓
Decide next action
  ↓
Act through a tool
  ↓
Observe result
  ↓
Update state
  ↓
Continue or terminate
```

Compact form:

```text
Observe → Decide → Act → Observe → Repeat
```

## System types

### LLM application

Any software that uses a large language model as one component.

```text
User input → LLM → Response
```

```python
user_input = input()
response = llm.generate(user_input)
print(response)
```

An LLM application is not automatically an agent. A single model call may only generate content without selecting or performing actions in an environment.

### Chatbot

A conversational LLM application that normally includes conversation history.

```text
Conversation history + Current message → LLM → Response
```

Remembering previous messages does not by itself make a chatbot an agent.

### RAG application

RAG means **Retrieval-Augmented Generation**. It retrieves external information and supplies it to the model before generation.

```text
Question → Retrieve relevant information → LLM → Grounded answer
```

A fixed `retrieve → generate → answer` pipeline is RAG, not necessarily an agent. Retrieval answers the knowledge question—“What information should the model receive?”—while agents address the action question—“What should the system do next?”

RAG can also be exposed as one tool inside an agent.

### Workflow

A workflow follows a sequence or graph of execution paths defined by the developer.

```text
Input → Step A → Step B → Step C → Output
```

The workflow may contain LLM calls and conditional branches. The defining property is that the developer has substantially predetermined the possible execution paths.

### Agent

An agent receives a goal, observes its current situation, selects an action, executes it, observes the result, updates its state, and repeats until it succeeds or must stop.

```python
while not done:
    decision = llm(goal=goal, state=state, tools=tools)

    if decision.action == "finish":
        return decision.answer

    result = execute(decision.tool)
    state.update(result)
```

The loop is the critical architectural difference.

### Hybrid system

A hybrid combines deterministic control with bounded agent decisions.

```text
Validate input             ← deterministic
      ↓
Agent investigates         ← agentic
      ↓
Approval policy            ← deterministic
      ↓
Agent performs action      ← agentic
      ↓
Store audit record         ← deterministic
```

Production systems are commonly hybrids because known rules are safer and cheaper to encode as ordinary software.

> Use deterministic software where the required behavior is known. Use agentic decisions where flexibility is genuinely required.

## Comparison table

| System | Execution path | External knowledge/actions | Iterative decisions | Typical classification |
|---|---|---|---|---|
| Single-prompt application | Fixed | Usually none | No | LLM application |
| Conversational application | Fixed | Conversation history | Usually no | Chatbot |
| Fixed retrieval pipeline | Fixed | Document retrieval | No | RAG application |
| Predetermined multistep process | Developer-defined | May use APIs and LLMs | No model-controlled loop | Workflow |
| Dynamic action loop | Model participates | Tools and environment | Yes | Agent |
| Agent inside policy boundaries | Mixed | Permissioned tools | Bounded | Hybrid |

## Core vocabulary

### Goal

The desired outcome the agent is attempting to achieve.

Examples:

- Find the best laptop under ₹80,000.
- Research competitors and produce a report.
- Fix failing tests in a repository.
- Investigate increased server latency.

Complex goals can benefit from decomposition and adaptive action selection. Complexity alone, however, does not prove that an agent is needed.

### Environment

Everything outside the agent that it can observe or affect.

Software-agent environments can include:

- websites and the internet;
- filesystems and code repositories;
- databases and APIs;
- browsers and operating systems;
- email, calendars, Slack, GitHub, and CRMs;
- cloud infrastructure.

### Observation

Information received from the environment, often as the result of an action.

Examples:

- search results;
- file contents;
- database rows;
- API responses;
- test output and error messages.

### Action

An operation performed by the agent, normally through a tool.

Examples include searching, reading or writing a file, querying a database, executing code, opening a webpage, or creating a calendar event.

### Tool

A capability that the application exposes to the model. The model selects a tool call, but the surrounding application validates and executes it.

```text
Model selects tool
        ↓
Application validates arguments
        ↓
Application executes tool
        ↓
Tool result becomes an observation
        ↓
Model selects the next action
```

Tool calling is a primitive used to construct agents; a single tool call does not automatically imply a strong agent architecture.

### Policy

The mechanism that maps the current situation to the next action.

Classical shorthand:

```text
policy(state) → action
```

For an LLM agent, the model may participate in the policy:

```text
model(goal, instructions, state, observations, available tools)
    → next action
```

The overall system policy may also include deterministic rules, permissions, budgets, and approval requirements.

### State

Information representing the agent's current task execution.

State can include:

- goal and conversation;
- current plan and step;
- completed and remaining tasks;
- tool calls and outputs;
- intermediate artifacts;
- errors and retry counts;
- user approvals;
- time, token, and cost budgets.

> State is what the agent currently knows about its execution.

### Memory

Information retained so it can be retrieved and reused later.

```text
State  = current execution workspace
Memory = information retained across time
```

Examples of memory include user preferences, stable project facts, prior decisions, and recurring entities. When memory is retrieved for a run, it becomes part of the current state or context.

Later memory categories include working, short-term, long-term, semantic, and episodic memory.

### Autonomy

The degree of freedom a system has to select actions and determine its execution path.

Greater autonomy can increase capability and flexibility, but it also tends to increase cost, complexity, risk, and unpredictability.

### Agent lifecycle

```text
1. Receive goal
2. Initialize state
3. Observe
4. Decide next action
5. Execute action
6. Receive observation
7. Update state
8. Check completion and safety conditions
9. Continue, finish, fail, or request help
```

## A useful autonomy scale

This scale is a learning aid, not an official industry standard.

| Level | Description | Typical shape |
|---:|---|---|
| 0 | Pure LLM | Prompt → response |
| 1 | LLM with retrieval | Retrieve → generate |
| 2 | LLM with tool selection | Choose tool → result → answer |
| 3 | Iterative agent | Decide → act → observe → repeat |
| 4 | Planning agent | Plan → execute → verify → replan |
| 5 | Long-running autonomous system | Persistent state, events, memory, recovery, approvals, and multiple tools |

## Single-agent and multi-agent systems

### Single agent

One primary agent controls the task and its tools.

```text
Research Agent
 ├── Web search
 ├── Browser
 ├── Files
 └── Database
```

Most systems should begin here.

### Multiple agents

Different agents receive distinct responsibilities, instructions, tools, or state.

```text
Supervisor
 ├── Researcher
 ├── Coder
 └── Reviewer
```

Multi-agent designs add coordination overhead. They are justified when specialization and isolated contexts provide a measurable advantage over one well-designed agent.

## Three recognition questions

When inspecting an unfamiliar system, ask:

1. **Does the model choose actions?** If not, it is probably not an agent.
2. **Can the next step change because of a previous result?** If yes, it is strongly agentic.
3. **Is there an iterative decide–act–observe loop?** If yes, it almost certainly uses an agent architecture.

Also identify the exact decisions controlled by the model and those controlled by deterministic code.

## Important non-equivalences

### Agent is not the same as intelligence

A powerful model used for one response can be intelligent but not agentic. A smaller model combined with tools, state, and a control loop can form an agent.

Agenticness describes architecture and behavior—not merely model capability.

### Agent is not the same as automation

Scheduled or automatic execution can still follow a fixed path. Agents may run inside automations, but automation alone does not imply dynamic decisions.

### Agent is not the same as RAG

RAG supplies relevant information. An agent selects and performs actions. RAG can be one of an agent's tools.

### Agent is not the same as tool calling

One model-selected tool call is weakly agentic, but the stronger pattern includes repeated selection based on observations, termination logic, and explicit state updates.

## Examples

### Customer support

| Version | Behavior | Classification |
|---|---|---|
| A | Responds directly to a support question | Chatbot |
| B | Extracts order ID, queries database, returns status in a fixed sequence | LLM workflow |
| C | Selects among order, refund, tracking, history, and escalation tools | Agent |
| D | Investigates dynamically but requires approval for large refunds | Hybrid |

### Coding assistance

| Version | Behavior | Classification |
|---|---|---|
| Chatbot | Reads pasted code and suggests a change | Chatbot |
| RAG assistant | Searches indexed code or documentation before suggesting | RAG application |
| Coding workflow | Always read file → generate patch → apply → test | Workflow |
| Coding agent | Explores, edits, tests, observes failure, revises, and verifies | Agent |

### Flight research

A fixed workflow searches a predefined set of airlines and compares their results. An agent can search current flights, filter based on observations, inspect baggage rules for promising options, and present a tradeoff-aware recommendation.

The agent's execution path emerges from intermediate results rather than being completely hard-coded.

## Why agents are harder

Each model decision, tool, API, state transition, and environment interaction expands the failure surface.

Common failures include:

- selecting the wrong tool;
- supplying invalid arguments;
- receiving unavailable, incomplete, stale, or misleading data;
- ignoring observations;
- repeating actions or looping forever;
- overflowing or polluting context;
- obeying malicious instructions found on webpages or in documents;
- taking dangerous or unauthorized actions.

This is why production agent engineering requires permissions, security, observability, evaluation, retries, error handling, and explicit termination policies.

## Mental model

```text
                 GOAL
                   │
                   ▼
          ┌─────────────────┐
          │      AGENT      │
          │ model           │
          │ instructions    │
          │ state + memory  │
          └────────┬────────┘
                   │ selects action
                   ▼
          ┌─────────────────┐
          │      TOOLS      │
          │ search/browser  │
          │ files/database  │
          │ code/APIs       │
          └────────┬────────┘
                   │ affects
                   ▼
              ENVIRONMENT
                   │
                   └── observation ──► AGENT
```

## Exercise 1 — Classify ten systems

Use:

```text
A. Normal LLM application/chatbot
B. RAG application
C. Deterministic workflow
D. Agent
E. Hybrid
```

| # | System | My classification | Why? Where is the decision made? |
|---:|---|---|---|
| 1 | Directly explains Docker | — | — |
| 2 | Retrieves an HR policy document before answering | — | — |
| 3 | Always extracts, validates, saves, and emails an invoice | — | — |
| 4 | Decides whether to call a weather API | — | — |
| 5 | Iteratively researches PostgreSQL versus MongoDB | — | — |
| 6 | Explores a repository, edits code, tests, and revises | — | — |
| 7 | Investigates refunds but requires approval above ₹20,000 | — | — |
| 8 | Always retrieves five semantic-search chunks before answering | — | — |
| 9 | Runs a fixed sales-report automation daily at 9 AM | — | — |
| 10 | Dynamically investigates unusual sales activity | — | — |

### Answer key

Do not review until the exercise has been attempted.

| # | Answer | Reason |
|---:|---|---|
| 1 | A — LLM application | Direct prompt and answer |
| 2 | B — RAG | Retrieval before generation |
| 3 | C — Workflow | Fixed sequence |
| 4 | D — Simple agent | Model selects an external action |
| 5 | D — Agent | Iterative decisions based on observations |
| 6 | D — Agent | Acts, observes tests, and adjusts |
| 7 | E — Hybrid | Agentic investigation with deterministic approval |
| 8 | B — RAG | Fixed retrieval pipeline |
| 9 | C — Workflow/automation | Automatic but predetermined |
| 10 | D — Agent | Goal-directed, dynamic investigation |

## Exercise 2 — Laptop system three ways

Task:

> Find the best laptop under ₹80,000 for AI development.

Design three architectures:

### Version A — Normal LLM application

- What input does the model receive?
- Does it use current information?
- What output does it produce?

### Version B — Workflow

- Which fixed steps will the developer implement?
- Which sources will always be queried?
- Which comparisons will always run?

### Version C — Agent

- Which decisions can the model make?
- Which tools can it select?
- How can observations change later actions?
- What are the success and stopping conditions?
- Which actions require approval?

## Completion checklist

- [ ] I can define LLM application, chatbot, RAG, workflow, agent, and hybrid.
- [ ] I can locate model-controlled and deterministic decisions in an architecture.
- [ ] I can explain goal, environment, observation, action, tool, policy, state, memory, autonomy, and agent loop.
- [ ] I can explain why automation, RAG, intelligence, and tool calling do not automatically imply an agent.
- [ ] I completed the classification exercise before checking its answers.
- [ ] I designed the laptop system as an LLM application, workflow, and agent.
- [ ] I can state the central definition without referring to the notes.

## Final takeaway

> An agent is not simply an LLM with a sophisticated prompt. It is a goal-directed system in which the model can select actions, observe their consequences, update state, and repeatedly decide what to do next within explicit limits.

The next topic is **Workflows vs Autonomous Agents**: when flexible model-controlled decisions are justified and when ordinary software is the better design.

