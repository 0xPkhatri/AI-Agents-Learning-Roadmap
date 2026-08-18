# 01 — AI Agents: Mental Model and Terminology

## Status

- **Roadmap phase:** Phase 1 — Foundations
- **Depth:** Basic
- **Prerequisites:** None
- **Started:** 2026-08-15
- **Completed:** —
- **Status:** In progress

## Why this matters

A precise definition prevents chatbots, RAG applications, deterministic workflows, and autonomous systems from being grouped together merely because they use an LLM.

## Learning objectives

- [ ] Distinguish an LLM application, chatbot, workflow, agent, and hybrid.
- [ ] Define environment, observation, action, goal, policy, tools, state, and memory.
- [ ] Explain deterministic and probabilistic components.
- [ ] Describe degrees of agency and an agent lifecycle.
- [ ] Locate where decision-making occurs in a given AI application.

## Initial mental model

```text
Agent = Model + Instructions + State + Tools + Control Loop
      + Environment + Memory + Safety + Evaluation
```

Working distinction:

- A **workflow** follows mostly predetermined execution paths.
- An **agent** dynamically decides which actions or tools to use and what to do next.
- A **hybrid** places bounded agent decisions inside deterministic orchestration.

This is a starting hypothesis to refine through definitions, counterexamples, and the classification exercise.

## Quick notes — Agent components

The learner's original quick-reference version is preserved in [AI Agent Basics — Quick Notes](01-ai-agent-basics-quick-notes.md).

The longer ChatGPT lesson has been condensed into [AI Agents — Detailed Study Guide](01-ai-agent-mental-model-study-guide.md).

```text
AI Agent = Model + Instructions + State + Tools + Control Loop
         + Environment + Memory + Safety + Evaluation
```

Treat this formula as a **design checklist and mental model**, not a strict requirement that every small agent must implement every component separately.

### Model

The **brain or intelligence** of the system. It learns patterns from data and produces predictions, decisions, or responses.

### Instructions

Tell the AI **what to do and how to behave**.

Example:

> Summarize this article in three bullet points.

### State

The information the AI is working with **right now**.

Examples include the current conversation, current task, plan, and recent tool results.

### Tools

External capabilities the AI can use.

Examples include web search, calculators, email, APIs, databases, and file readers.

### Control loop

The repeated process the agent follows:

```text
Observe → Decide → Act → Observe → Repeat
```

The loop continues until the task succeeds, fails, requires escalation, or reaches a configured limit.

“Think” is a useful informal analogy, but **decide** is more precise in an engineering description because the system is controlled through observable inputs, actions, and outputs—not access to private model reasoning.

### Environment

The **outside world** the agent interacts with.

Examples include users, websites, files, applications, databases, and physical surroundings.

### Memory

Information retained so the agent can use it later.

```text
State  = information needed for the current computation
Memory = retained information that can be retrieved later
```

The distinction is conceptual rather than absolute: retrieved memory can become part of the current state.

### Safety

Rules, permissions, and protections intended to prevent harmful, private, unsafe, or unauthorized actions.

### Evaluation

Checks how well the agent performed. Evaluation can measure:

- correctness
- instruction following
- tool selection and usage
- safety
- efficiency
- task success

## Easy analogy

| Component | Analogy |
|---|---|
| Model | Brain |
| Instructions | Mission |
| State | Current awareness |
| Tools | Hands |
| Control loop | Decide-and-act cycle |
| Environment | World |
| Memory | Past information |
| Safety | Guardrails |
| Evaluation | Performance check |

## Final idea

These components working together provide a useful mental model for an **AI agent or agentic AI system**. The exact implementation depends on the task: simple agents may omit persistent memory, while production agents usually require explicit safety and evaluation mechanisms.

## Key terms

| Term | Working definition | Source | Confidence |
|---|---|---|---|
| Agent | A goal-directed system in which a model participates in deciding the next action within a control loop and environment. | Roadmap; refine during study | Medium |
| Workflow | A system whose possible execution paths are mostly predetermined by its designer. | Roadmap; S01 | Medium |
| Environment | The external or simulated context an agent observes and can affect. | Roadmap | Medium |
| Observation | Information returned to the agent about its current state or the result of an action. | Roadmap | Medium |
| Action | An operation selected by the agent that may affect state or the environment. | Roadmap | Medium |
| Policy | The decision mechanism mapping available context/state to the next action. | Roadmap | Medium |
| State | Explicit information required to continue the current computation or task. | Roadmap | Medium |
| Memory | Stored information that can be retrieved for use beyond the immediate input. | Roadmap | Medium |

## Questions to resolve

- [ ] Is dynamic tool choice necessary for a system to count as an agent?
- [ ] How much autonomy is enough to use the term “agent” usefully?
- [ ] Where is the boundary between a conditional LLM workflow and a low-agency agent?
- [ ] Can a system be agentic without external tools?
- [ ] Which parts of an agent should remain deterministic?

## Exercise — Classify 10 systems

Classify each as **workflow**, **agent**, or **hybrid**, and identify the exact decision point.

| # | System | Classification | Decision-maker | Evidence/reasoning |
|---:|---|---|---|---|
| 1 | Basic ChatGPT-style chat | — | — | — |
| 2 | RAG chatbot with retrieval on every query | — | — | — |
| 3 | RAG chatbot that decides whether to retrieve | — | — | — |
| 4 | Coding assistant with repository and shell tools | — | — | — |
| 5 | Fixed customer-support routing pipeline | — | — | — |
| 6 | Support system that dynamically selects resolution tools | — | — | — |
| 7 | Autonomous multi-source research system | — | — | — |
| 8 | Email summarizer triggered on a schedule | — | — | — |
| 9 | Travel assistant that plans and replans around constraints | — | — | — |
| 10 | Form-filling browser automation with fixed selectors | — | — | — |

## Completion check

- [ ] I can explain the mental model without looking at the notes.
- [ ] I can classify unfamiliar systems and defend the classification.
- [ ] I can point to the model-controlled and deterministic decisions.
- [ ] I completed and reviewed all 10 exercise cases.
- [ ] I wrote a short definition of “agent” in my own words.

## Sources

- [LangGraph: Workflows and agents](https://docs.langchain.com/oss/python/langgraph/workflows-agents) — verify and annotate when this topic is studied.
