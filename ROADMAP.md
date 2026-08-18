# AI Agents Learning Roadmap

This is the operational index of the submitted roadmap. Detailed definitions, exercises, code, evidence, and reflections are added to each topic note as that topic begins.

## Learning rules

- Learn the manual agent loop before adopting an orchestration framework.
- Keep deterministic behavior in ordinary code when autonomy adds no value.
- Treat tools, state, termination, permissions, and evaluation as first-class design concerns.
- Build each project as evidence that the corresponding concepts are understood.
- Recheck time-sensitive framework and protocol claims against primary documentation when studying them.

## Phase 1 — Foundations

**Goal:** Understand what makes a system an agent before writing one.

1. **AI Agents: Mental Model and Terminology** — distinguish LLM apps, chatbots, RAG, workflows, agents, and hybrids; learn environment, observation, action, goal, policy, tools, state, memory, autonomy, and lifecycle. Exercise: classify 10 systems.
2. **Workflows vs Autonomous Agents** — fixed pipelines, conditional and LLM workflows, dynamic tool selection/decomposition, autonomy spectrum, and when not to use agents. Exercise: solve one support problem three ways.
3. **Anatomy of an Agent** — input, instructions, model, tools, state, loop, observations/actions, stop conditions, response, and environment. Exercise: pseudocode a calculator/search agent.

## Phase 2 — LLM Application Fundamentals

**Goal:** Master the primitives from which agent behavior is built.

4. **LLM APIs for Agent Development** — request lifecycle, messages, roles, generation controls, tokens, streaming, async, limits, models, multimodality, and continuation. Build a configurable CLI assistant.
5. **System Prompts and Instruction Design** — roles, goals, constraints, tool and decision policies, instruction priority, data separation, examples, modularity, dynamic instructions, and versioning.
6. **Structured Outputs and Schemas** — JSON Schema, Pydantic/Zod, types, enums, validation, nesting, evolution, and failures. Build a typed task classifier.
7. **Function / Tool Calling** — tool schemas, arguments, selection, results, forced/automatic choice, parallel calls, lifecycle, and validation. Build Project 1.
8. **Building High-Quality Custom Tools** — scope, names, descriptions, schemas, idempotency, side effects, read/write distinctions, pagination, authentication, timeouts, and error contracts.
9. **External APIs and Integrations** — REST, HTTP, OAuth, API keys, webhooks, pagination, limits, SDKs, secrets, and permissions. Build a workspace agent.

## Phase 3 — Tool-Using Agents

**Goal:** Build the first real agent runtime without a framework.

10. **The Agent Loop** — observe, decide, act, update state, terminate, budget, detect loops, and trace execution. Implement a runtime in a few hundred lines or fewer.
11. **Agent State** — conversation, task, world, tool, artifact, plan, and execution state; mutation, serialization, checkpoints, and state machines.
12. **Tool Routing and Action Selection** — namespaces, filtering, availability, ranking, discovery, hierarchies, restrictions, and context-aware exposure.
13. **MCP — Model Context Protocol** — hosts, clients, servers, tools, resources, prompts, negotiation, transports, authorization, lifecycle, and boundaries. Consume and create a server.

## Phase 4 — Memory and Knowledge

**Goal:** Use information beyond the immediate prompt without polluting context.

14. **Context Windows and Context Engineering** — budgets, truncation, salience, recency, pollution, compression, summarization, assembly, and tool-result filtering.
15. **Memory Models** — working, session, long-term, semantic, episodic, procedural, preference, and entity memory; retrieval, consolidation, and forgetting.
16. **Embeddings** — vector representations, semantic similarity, cosine similarity, dimensions, chunk/query embeddings, thresholds, filters, and model selection.
17. **Vector Databases and Semantic Retrieval** — indexes, approximate neighbors, metadata, filtering, namespaces, mutations, hybrid retrieval, and persistence.
18. **RAG** — ingest, parse, clean, chunk, index, transform queries, retrieve, rerank, assemble context, ground answers, cite, and evaluate. Build Project 2.
19. **Search and Retrieval Strategies** — BM25, semantic/hybrid search, rewriting, multi-query, recursive and parent-child retrieval, graphs, freshness, reliability, and confidence.
20. **Persistent Personal Memory** — extraction and write policies, profiles, temporal/semantic retrieval, importance, conflicts, updates, deletion, and privacy. Build Project 3.

## Phase 5 — Planning and Agent Loops

**Goal:** Move from reactive tool use to goal-directed behavior.

21. **Task Decomposition** — subgoals, atomic tasks, dependencies, preconditions, task graphs, completion criteria, parallelism, and dynamic decomposition.
22. **Planning Strategies** — plan-and-execute, replanning, rolling horizons, hierarchy, planner/executor separation, verification, constraints, and DAGs.
23. **Reasoning and Decision Patterns** — ReAct-style control, planning, reflection, critique, generate/verify, candidate scoring, solution search, and escalation. Focus on observable engineering patterns, not private reasoning.
24. **Termination and Control Policies** — limits for steps, tokens, time, and cost; success/failure conditions; loop, stagnation, repetition, and confidence controls.
25. **Research Agent** — Project 4: formulate questions, plan, search, inspect and compare sources, keep notes, find evidence gaps, cite, synthesize, and stop within budgets.

## Phase 6 — Advanced Agent Architectures

26. **Browser and Web Agents** — DOM/vision interaction, page state, navigation, forms, authentication, downloads, sandboxing, dynamic pages, and recovery. Build Project 5.
27. **Database Agents** — schema discovery, SQL generation and validation, read-only access, planning, summarization, limits, timeouts, transactions, and isolation.
28. **Code-Execution Agents** — sandboxing, filesystem/shell/Python isolation, dependencies, tests, analysis, limits, and artifacts.
29. **Coding Agents** — repository and symbol search, dependency inspection, patches, test loops, compiler/linter feedback, diffs, verification, and context management. Build Project 6.
30. **Human-in-the-Loop Systems** — approval gates, interrupts, checkpoints, confirmations, queues, escalation, corrections, and high-risk boundaries.
31. **Multi-Agent Fundamentals** — specialization, supervisors, routers, agents as tools, delegation, handoffs, context isolation, overhead, redundancy, and failure modes.
32. **Agent-to-Agent Communication** — typed messages, contracts, handoffs, shared state, blackboards, event buses, queues, aggregation, and conflict resolution.
33. **Multi-Agent Orchestration** — supervisor/worker, router/specialist, and planner/worker/reviewer architectures. Build Project 7 with minimum necessary context per agent.

## Phase 7 — Agent Frameworks

**Entry condition:** The manual agent loop and core architectures above can be implemented and explained.

34. **Framework Architecture Patterns** — agent abstractions, graph runtimes, tool registration, state, persistence, memory, handoffs, retries, approval, tracing, streaming, and deployment integrations.
35. **OpenAI Agents SDK** — agents, runners, tools, handoffs, guardrails, sessions, state, interruptions, tracing, streaming, and usage. Rebuild the research agent.
36. **LangGraph** — state, nodes, edges, conditional transitions, graph execution, checkpoints, interrupts, persistence, subgraphs, and durable execution.
37. **CrewAI** — agents, tasks, crews, flows, tools, knowledge, memory, and delegation. Rebuild the multi-agent research system.
38. **AutoGen** — AgentChat, agents, teams, tools, termination, human involvement, and multi-agent conversations.
39. **Framework vs From-Scratch Decisions** — compare control, complexity, dependencies, latency, persistence, branching, observability, handoffs, and infrastructure.

## Phase 8 — Reliability, Security, and Evaluation

**Goal:** Turn demonstrations into dependable systems.

40. **Error Handling and Recovery** — exceptions, invalid arguments, empty results, API/model/parser failures, partial completion, corrupt state, compensation, and degradation.
41. **Retries, Backoff, and Fallbacks** — exponential backoff, jitter, retry classification and budgets, tool/model/provider fallback, circuit breakers, idempotency, and dead letters.
42. **Guardrails and Permissions** — input/output checks, tool policy, read/write separation, allow/deny rules, least privilege, scoped authorization, approvals, limits, and policy engines.
43. **Agent Security** — threat modeling, untrusted input, secrets, sandboxes, credentials, capability security, exfiltration, tenant isolation, audits, and supply-chain risks.
44. **Prompt Injection and Tool-Use Attacks** — direct/indirect injection, poisoned retrieval/tools, exfiltration, instruction/data separation, escalation, trust boundaries, confused deputies, and permission gates.
45. **Observability, Logging, and Tracing** — IDs, spans, model/tool calls, transitions, latency, usage, cost, errors, retries, metadata, and audit logs.
46. **Testing Agents** — unit, contract, integration, scenario, regression, synthetic, mock, golden, and adversarial testing with stochastic tolerance.
47. **Agent Evaluation** — task success, correctness, tool and argument accuracy, retrieval, faithfulness, plans, safety, instruction adherence, efficiency, judges, humans, and pairwise comparisons.
48. **Measuring Reliability** — success, failures, hallucinations, recovery, intervention, steps, tokens, cost, and latency distributions.
49. **Cost and Latency Optimization** — model routing, context pruning, caches, parallelism, retrieval, batching, speculative work, early termination, budgets, and async execution.

## Phase 9 — Production and Deployment

50. **Production Agent Architecture** — API gateway/service, model and tool services, retrieval, relational/vector stores, queues, workers, caches, object storage, durable state, and observability.
51. **Deployment** — Docker, containers, environment configuration, secrets, CI/CD, health checks, scaling, shutdown, rolling releases, and provider configuration.
52. **Async Execution and Job Queues** — jobs, queues, workers, IDs, progress, cancellation, retries, priority, concurrency, and scheduling.
53. **Persistent and Long-Running Agents** — durable execution, checkpoints, crash recovery, sleep/wake, schedules, events, human/API waits, persistence, and event sourcing.
54. **Event-Driven Agents and Background Tasks** — cron, webhooks, streams, pub/sub, triggers, queues, notifications, schedules, and reactive agents.
55. **Scaling Agent Systems** — horizontal workers, partitioning, concurrency and rate coordination, locks, idempotency, caching, shared state, isolation, pooling, backpressure, and load shedding.
56. **Production Monitoring and Continuous Improvement** — production evals, regression and failure clustering, version tracking, feedback, trace-derived datasets, canaries, and A/B tests.

## Phase 10 — Capstone

57. **Production-Ready Autonomous Agent From Scratch** — build an autonomous research and execution agent with planning, explicit state, context and memory management, RAG, routing, safety, evaluation hooks, broad but permissioned tools, resumability, recovery, observability, and production deployment.

### Capstone milestones

1. Core — user → LLM → tools → final answer
2. Agent — loop through actions and observations
3. Planning — plan, execute, verify, and replan
4. Knowledge — RAG, search, and persistent memory
5. Reliability — retries, fallbacks, checkpoints, and recovery
6. Safety — permissions, sandboxing, HITL, and injection defenses
7. Observability — trace every model call, tool call, and state change
8. Evaluation — automated suites and reliability metrics
9. Production — API, database, queue, workers, Docker, and monitoring
10. Scale — concurrency, isolation, rate limits, retries, idempotency, and cost controls

Only after this manual version works should it be rebuilt with a framework to reveal exactly what that framework abstracts.
