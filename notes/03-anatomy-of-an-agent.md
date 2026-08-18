# 03 — Anatomy of an Agent

- **Phase:** 1 — Foundations
- **Depth:** Basic → Intermediate
- **Prerequisites:** Topics 01–02
- **Status:** In progress
- **Started:** 2026-08-19
- **Source:** User-provided ChatGPT lesson (`S15`)

## Learning outcome

After this topic, I should be able to:

- identify every core component inside a basic agent;
- explain the difference between model decisions and runtime execution;
- draw the complete action–observation loop;
- specify state and termination policies;
- design a small tool-using agent without a framework; and
- diagnose failures by locating the responsible component.

## Central mental model

```text
Input
  ↓
Goal
  ↓
Instructions
  ↓
Model decision
  ↓
Tool / action
  ↓
Environment
  ↓
Observation
  ↓
State update
  ↓
Repeat or stop
  ↓
Final response
```

> An agent is not merely the model. It is the complete runtime system around the model: instructions, tools, state, environment interaction, observations, control loop, and termination logic.

## The twelve core components

| # | Component | Purpose |
|---:|---|---|
| 1 | Input | Request entering the system |
| 2 | Goal | Definition of successful completion |
| 3 | Instructions | Rules governing agent behavior |
| 4 | Model | Component participating in next-action decisions |
| 5 | Tools | Controlled capabilities exposed to the model |
| 6 | State | Current execution information |
| 7 | Agent loop | Repeated decision and execution cycle |
| 8 | Actions | Operations selected by the agent |
| 9 | Observations | Results returned after actions |
| 10 | Environment | External systems and resources affected or inspected |
| 11 | Stop conditions | Rules that terminate execution |
| 12 | Final response | Completed outcome returned by the run |

Later capabilities—memory, planning, RAG, guardrails, human approval, tracing, evaluation, and persistence—extend this basic architecture.

## Input and goal

### Input

The input is what enters the agent, usually a user request.

Examples:

- “Calculate 25 × 18 and find Tokyo’s population.”
- “Find three recent papers about AI agents and compare them.”
- “Fix the failing tests in this repository.”

### Goal

The goal describes the successful outcome the agent should achieve.

```text
Input = what the user said
Goal  = what successful completion means
```

Example:

```text
Input:
My package has not arrived.

Goal:
Determine the shipment status and help resolve the delivery problem.
```

The goal may be identical to a precise request or inferred from an ambiguous one.

### Why the goal matters

The loop cannot reliably determine whether it is finished without completion criteria.

Vague:

```text
Research AI agents.
```

Measurable:

```text
Compare four agent frameworks by architecture, strengths,
weaknesses, and recommended use cases, using current primary sources.
```

A useful goal constrains the work and supplies evidence for termination.

## Instructions

Instructions are the operating policy provided by the developer.

They can define:

- role and goal;
- constraints and permissions;
- tool-use rules;
- safety requirements;
- output format;
- evidence requirements; and
- completion and escalation behavior.

Example:

```text
You are a research assistant.

Answer accurately and use search for current information.
Prefer primary sources and never invent evidence.
Explain conflicting evidence.
Stop when the requested comparison is supported sufficiently.
```

### Instructions versus user requests

Higher-priority developer or system instructions constrain user requests.

```text
System: Never execute financial transactions.
User:   Buy 100 shares of Apple.
```

The permitted response is research or explanation—not transaction execution.

```text
Developer/system policy
        ↓
User task
        ↓
Permitted agent decision
```

## Model

The model is a decision-making component inside the runtime.

Within an agent, its role is often more important than generating prose:

```text
Given:
- goal
- instructions
- current state
- available tools
- previous observations

Decide:
- the next action, or
- that the task is complete
```

Conceptually:

```python
decision = model(
    instructions=instructions,
    task=user_input,
    state=current_state,
    tools=available_tools,
)
```

The result might be a structured tool request rather than an answer:

```json
{
  "type": "tool_call",
  "tool": "calculator",
  "arguments": {"expression": "25 * 18"}
}
```

## Deciding is different from executing

The model usually does not execute tools itself.

```text
Model requests a tool call
        ↓
Runtime validates the request
        ↓
Application code executes the tool
        ↓
Runtime records the result
        ↓
Result returns to the model as an observation
```

This boundary is fundamental to security and reliability.

```text
MODEL decides.
RUNTIME validates and executes.
```

## Tools

Tools are controlled capabilities that the application exposes to the model.

Examples:

| Agent | Possible tools |
|---|---|
| Calculator/research | `calculator`, `search_web`, `open_url` |
| Coding | `search_code`, `read_file`, `write_file`, `run_tests` |
| Personal assistant | `read_calendar`, `find_contact`, `create_event`, `send_email` |
| Business | `query_database`, `search_crm`, `create_ticket` |

A tool normally has:

```text
Name
Description
Input schema
```

Example:

```text
Name: calculator
Description: Evaluate a mathematical expression.
Input: {"expression": "string"}
```

Tools bridge the gap between model knowledge and current, private, computational, or actionable capabilities.

The developer still controls which tools exist, their permissions, validation, and execution environment.

## State

State answers:

> What does the agent currently know about this execution?

Initial state:

```python
state = {
    "task": "Calculate 25 × 18 and find Tokyo's population",
    "steps": [],
    "tool_results": [],
    "final_answer": None,
}
```

After a calculation:

```python
state = {
    "task": "Calculate 25 × 18 and find Tokyo's population",
    "steps": ["calculator(25 * 18)"],
    "tool_results": [
        {"tool": "calculator", "result": 450}
    ],
    "final_answer": None,
}
```

State evolves as actions and observations accumulate.

### State is more than conversation history

Messages are only one possible part of state. A production agent may track:

```python
state = {
    "goal": "...",
    "current_plan": [],
    "completed_steps": [],
    "failed_steps": [],
    "tool_results": [],
    "files_created": [],
    "approvals": [],
    "cost": 0.42,
    "steps_used": 7,
    "status": "running",
}
```

Explicit state makes execution inspectable, resumable, testable, and enforceable.

## Actions

An action is what the agent chooses to do next.

Possible action types include:

- call a tool;
- ask the user for missing information;
- update or revise a plan;
- delegate a bounded task;
- wait for an event;
- request approval; or
- finish.

A minimal agent can restrict its decision schema to only two possibilities:

```json
{
  "type": "tool_call",
  "tool": "calculator",
  "arguments": {"expression": "25 * 18"}
}
```

or:

```json
{
  "type": "final_response",
  "answer": "25 × 18 = 450."
}
```

Structured decisions are easier to validate and dispatch than free-form prose.

## Environment and observations

### Environment

The environment is everything outside the agent runtime that it can inspect or affect.

| Agent | Environment |
|---|---|
| Research | Internet, websites, search engines, documents |
| Coding | Repository, filesystem, runtime, compiler, Git, tests |
| Business | CRM, database, email, calendar, Slack, APIs |

### Observation

An observation is information returned after an action.

```text
Action:      calculator("25 * 18")
Observation: 450

Action:      run_tests()
Observation: 47 passed, 2 failed
```

```text
Action
  ↓
Environment changes or is inspected
  ↓
Observation
  ↓
State update
```

## The agent loop

The control loop is the heart of the runtime:

1. Inspect current state.
2. Build the model context.
3. Ask the model for the next decision.
4. Validate the decision.
5. Execute an allowed action.
6. receive the observation.
7. Update state.
8. Check success and runtime limits.
9. Repeat or terminate.

```text
          MODEL
            │
       chooses action
            ▼
          TOOL
            │
       affects/reads
            ▼
       ENVIRONMENT
            │
       observation
            ▼
       UPDATE STATE
            │
            └──────────► MODEL
```

### Minimal pseudocode

```python
state = initialize_state(user_input)

while True:
    decision = model.decide(
        instructions=instructions,
        state=state,
        tools=tools,
    )

    if decision.type == "final":
        return decision.answer

    if decision.type == "tool_call":
        result = execute_tool(
            decision.tool,
            decision.arguments,
        )

        state.add_observation(
            tool=decision.tool,
            arguments=decision.arguments,
            result=result,
        )
```

This captures the core architecture, but `while True` creates a serious risk: the model may never finish.

## Stop conditions

Stop conditions determine when a run succeeds, fails, pauses, or is forcibly terminated.

### Model-requested completion

```json
{
  "type": "final_response",
  "answer": "..."
}
```

### Runtime-enforced limits

- maximum steps;
- maximum time;
- token budget;
- cost budget;
- maximum calls per tool;
- repeated-action or stagnation detection;
- human cancellation;
- permission or safety failure.

Example:

```python
MAX_STEPS = 10

for step in range(MAX_STEPS):
    decision = model(...)

    if decision.type == "final":
        return decision.answer

    # validate, execute, and record action

raise AgentError("Maximum steps exceeded")
```

### Bounded tool usage

```python
if state["search_calls"] >= 5:
    disable_search()
```

### Cost and time limits

```python
if state["cost_usd"] > MAX_COST:
    return best_partial_result()

if elapsed_seconds() > MAX_SECONDS:
    return best_partial_result()
```

Important principle:

> The model must not be the only component deciding whether execution may continue.

```python
while steps < MAX_STEPS and cost < MAX_COST:
    decision = model(...)
    if decision.type == "finish":
        return decision.answer
```

## Final response

The final response is the completed outcome of the run. It can be:

- natural-language text;
- structured JSON;
- a report or recommendation;
- a generated file;
- a code patch;
- a database change; or
- another application artifact.

A final response should reflect the stated goal and make partial completion or uncertainty explicit.

## Complete architecture

```text
USER INPUT
    ↓
INTERPRETED GOAL
    ↓
INSTRUCTIONS + INITIAL STATE
    ↓
┌────────────────────────────────┐
│ MODEL chooses ACTION           │
│       ↓                        │
│ RUNTIME validates              │
│       ↓                        │
│ TOOL interacts with ENVIRONMENT│
│       ↓                        │
│ OBSERVATION returns            │
│       ↓                        │
│ RUNTIME updates STATE          │
│       ↓                        │
│ Stop condition? ── No ─────────┘
└────────────── Yes
               ↓
         FINAL RESPONSE
```

## Calculator and search agent

Goal:

> Answer questions requiring calculation, current information, or both.

Tools:

```text
calculator(expression)
search_web(query)
```

Instructions:

```text
You are a calculator and research agent.

Answer the user's request accurately.
Use calculator for arithmetic.
Use search_web for information that must be looked up.
You may call multiple tools.
Never invent tool results.
Return a final answer when sufficient information is available.
```

### Example task

```text
Find Japan's approximate current population and calculate 2% of it.
```

Initial state:

```python
state = {
    "user_request": "Find Japan's population and calculate 2% of it.",
    "tool_history": [],
    "step_count": 0,
}
```

### Execution trace

```text
1. Model identifies missing current information.

   Action:
   search_web("Japan current population")

2. Search returns an observation.

   Observation:
   Approximately 123,000,000

3. Runtime records the search result in state.

4. Model sees the updated state and selects calculation.

   Action:
   calculator("123000000 * 0.02")

5. Calculator returns an observation.

   Observation:
   2,460,000

6. Runtime records the calculation.

7. Model determines that the goal is satisfied.

   Final:
   Japan's population is approximately 123 million;
   2% is approximately 2.46 million.
```

The exact population value and evidence would need a current, cited source during a real run.

### More realistic pseudocode

```python
TOOLS = {
    "calculator": calculator,
    "search_web": search_web,
}

MAX_STEPS = 10


def run_agent(user_input):
    state = {
        "user_input": user_input,
        "history": [],
        "step_count": 0,
    }

    for step in range(MAX_STEPS):
        state["step_count"] = step + 1

        decision = call_model(
            instructions=AGENT_INSTRUCTIONS,
            user_input=user_input,
            state=state,
            tools=TOOLS,
        )

        if decision.type == "final":
            return decision.answer

        if decision.type != "tool_call":
            state["history"].append({
                "error": "Unsupported decision type"
            })
            continue

        tool = TOOLS.get(decision.tool_name)

        if tool is None:
            result = {"error": "Unknown tool"}
        else:
            try:
                result = tool(**decision.arguments)
            except Exception as error:
                result = {"error": str(error)}

        state["history"].append({
            "action": decision,
            "observation": result,
        })

    return "Agent stopped because the maximum step limit was reached."
```

### What `call_model()` represents

```text
WHO ARE YOU?             → instructions
WHAT SHOULD YOU ACHIEVE? → goal/task
WHAT DO YOU KNOW?        → state and observations
WHAT CAN YOU DO?         → tools
WHAT SHOULD HAPPEN NEXT? → structured decision
```

`call_model()` is the central model-controlled decision point. The surrounding runtime remains responsible for validation and execution.

## Model and runtime responsibilities

| Model side | Runtime/code side |
|---|---|
| Interpret the task | Execute tools |
| Identify missing information | Validate tool names and arguments |
| Select a permitted tool | Maintain and serialize state |
| Propose tool arguments | Enforce permissions and approval rules |
| Determine whether evidence is sufficient | Enforce steps, time, token, and cost limits |
| Generate the final response | Normalize errors and record usage |

> Do not give the model responsibilities that ordinary code can enforce more reliably.

## The same anatomy across agent types

### Coding agent

```text
Input:        Fix the failing test.
Goal:         Relevant tests pass without regressions.
Instructions: Inspect first, change minimally, test afterward.
Tools:        search_code, read_file, write_file, run_tests
Environment:  Repository, filesystem, test runner
State:        Files inspected, failures, patches, tool history
Loop:         Inspect → edit → test → inspect failure → revise
Stop:         Tests pass or attempt limit is reached
Final:        Summary of changes and verified test outcome
```

### Research agent

```text
Tools:  search_web, open_page, search_docs
State:  questions, sources, evidence, notes, gaps
Loop:   search → read → identify missing evidence → search → compare
Stop:   required claims are supported or research budget is exhausted
```

### Personal assistant

```text
Tools:       read_calendar, find_contact, create_event
Environment: Calendar and contacts
State:       Date, available slots, contact, choice, approval status
Loop:        Find contact → check calendar → select slot → approve → create
```

The environment and tools change, but the architectural primitives remain the same.

## Frameworks abstract these primitives

Agent frameworks use different names and structures, but generally package some combination of:

```text
model
+ instructions
+ tools
+ state
+ action routing
+ control loop
+ termination
+ tracing and persistence
```

For example, a graph runtime may represent work as nodes and transitions over shared state, while an agent SDK may package model, instructions, tools, guardrails, and a runner.

Frameworks should therefore be understood as implementation abstractions—not magic.

## How the loop grows toward production

```python
while not terminated:
    validate_state()
    context = build_context(state)
    decision = model(context, tools)
    validate_decision(decision)

    if decision.is_final:
        validate_output(decision.output)
        return decision.output

    if not permitted(decision):
        handle_permission_failure()
        continue

    try:
        result = execute_tool(decision)
    except RetryableError:
        result = retry_tool(decision)
    except Exception as error:
        result = normalize_error(error)

    state.record(decision=decision, result=result)
    trace(state=state, decision=decision, result=result)
    enforce_budgets(state)
```

Future roadmap topics attach naturally:

| Topic | Improvement to the basic runtime |
|---|---|
| Structured output | Reliable decision objects |
| Tool calling | Standard model-to-tool protocol |
| Memory | Information retrieved across sessions |
| RAG | External knowledge retrieval |
| Planning | Goal decomposition and replanning |
| Guardrails | Action and output policies |
| Human-in-the-loop | Approval and correction points |
| Observability | Records of calls, actions, and transitions |
| Evaluation | Evidence that the agent succeeds safely |
| Deployment | Persistence, recovery, concurrency, and scale |

## Five debugging questions

When an agent behaves unexpectedly, ask:

1. **Instructions:** Did the model receive the correct behavioral policy?
2. **Tools:** Did it have the correct and clearly described capabilities?
3. **State/context:** Did it receive the information required for this decision?
4. **Environment:** Was the tool result correct, complete, and trustworthy?
5. **Control loop:** Did the runtime validate the action and enforce termination?

This maps strange behavior to a component rather than treating the whole agent as an opaque model problem.

## Exercise 1 — Design four calculator/search runs

For each task, write:

```text
Input
Goal
Action 1
Observation 1
Action 2
Observation 2
...
Stop condition
Final response
```

### Task 1

```text
Calculate 476 × 83.
```

### Task 2

```text
Who created the Python programming language?
```

### Task 3

```text
Find the approximate population of India and calculate 0.5% of it.
```

### Task 4

```text
Find the current price of Bitcoin and calculate the value of 0.025 BTC.
```

For current values, record the source and observation time. The final answer should distinguish sourced facts from derived calculations.

## Exercise 2 — Write the loop in pseudocode

Complete this without a framework or real API integration:

```python
TOOLS = {
    "calculator": calculator,
    "search_web": search_web,
}


def agent(user_input):
    state = {
        # Define explicit execution state.
    }

    for step in range(10):
        # Build context.
        # Ask the model for a structured decision.
        # Validate the decision.
        # Return if final.
        # Execute the permitted tool if requested.
        # Record the action and observation.
        pass

    # Stop safely and report partial progress.
```

The solution must make clear:

- who decides;
- who validates and executes;
- where observations are recorded;
- why the loop repeats; and
- why execution stops.

## Completion checklist

- [ ] I can define all 12 core components without looking at the notes.
- [ ] I can distinguish input from goal.
- [ ] I can explain why the model selects a tool but the runtime executes it.
- [ ] I can design explicit state beyond conversation history.
- [ ] I can draw the action–environment–observation–state loop.
- [ ] I can define success-based and runtime-enforced stop conditions.
- [ ] I can separate model responsibilities from runtime responsibilities.
- [ ] I completed all four calculator/search execution traces.
- [ ] I wrote and reviewed the framework-free pseudocode loop.
- [ ] I can apply the five debugging questions to a failed run.

## Final summary

```text
MODEL decides.
RUNTIME validates and executes.
TOOLS affect or inspect the environment.
OBSERVATIONS return new information.
STATE records progress.
THE LOOP repeats.
STOP CONDITIONS prevent unbounded execution.
THE FINAL RESPONSE demonstrates completion.
```

With this architecture understood, the conceptual foundation of Phase 1 is in place. The next topic is **LLM APIs for Agent Development**: messages, requests and responses, streaming, usage, model selection, timeouts, rate limits, and how the runtime communicates with a model.
