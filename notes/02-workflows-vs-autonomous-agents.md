# 02 — Workflows vs Autonomous Agents

- **Phase:** 1 — Foundations
- **Depth:** Basic
- **Prerequisite:** Topic 01 — AI Agents: Mental Model and Terminology
- **Status:** In progress
- **Started:** 2026-08-18
- **Source:** User-provided ChatGPT lesson (`S14`)

## Learning outcome

After this topic, I should be able to design the same problem as:

- deterministic software;
- an LLM-powered workflow;
- an autonomous agent; or
- a hybrid system.

I should then be able to explain which architecture is appropriate and why.

## Central question

> When should the AI decide what happens next, and when should normal code decide?

Default engineering rule:

> Use deterministic workflows when the correct process is known. Give the model autonomy only where the process genuinely requires judgment or adaptation.

The objective is the **minimum useful level of autonomy**, not the maximum possible autonomy.

## Control flow: who decides what happens next?

Imagine these customer-support operations:

```text
A = Read customer request
B = Look up order
C = Check refund policy
D = Issue refund
E = Escalate to human
```

### Program-controlled flow

```text
A → B → C
        ├─ eligible     → D
        └─ not eligible → E
```

The developer defines the transitions. This is workflow-oriented.

### Model-controlled flow

```text
Request
  ↓
Model selects order lookup
  ↓
Observation changes what is known
  ↓
Model selects policy search
  ↓
Unexpected tracking data appears
  ↓
Model selects tracking investigation
  ↓
Continue until a stop condition
```

The model meaningfully influences the execution path. This is agent-oriented.

```text
Workflow = control flow primarily belongs to the program.
Agent    = meaningful control-flow decisions belong to the model.
```

## Fixed pipelines

A fixed pipeline sends every input through essentially the same stages.

```text
Resume
  ↓
Extract text
  ↓
Extract candidate fields
  ↓
Validate fields
  ↓
Calculate score
  ↓
Save result
```

```python
def process_resume(resume):
    text = extract_text(resume)
    candidate = llm_extract_candidate(text)
    validate(candidate)
    score = calculate_score(candidate)
    save(candidate, score)
    return score
```

An LLM may implement one step, but the overall architecture remains a workflow because the sequence is predetermined.

## Why workflows are valuable

| Property | Workflow advantage |
|---|---|
| Predictability | The executed steps and allowed transitions are known. |
| Testability | Functions, branches, and contracts can be tested independently. |
| Debuggability | Failures can be localized to a specific stage. |
| Security | Each step can receive narrowly scoped permissions. |
| Cost control | The approximate number of model and tool calls is known. |
| Latency control | Execution time is easier to estimate and constrain. |
| Reliability | Fewer possible execution paths mean fewer unexpected behaviors. |

Powerful LLM applications can—and often should—use conventional workflows.

## Conditional workflows

A workflow can branch without becoming an autonomous agent.

```text
                     ┌─ Refund workflow
Request → Classifier ├─ Order-status workflow
                     └─ Technical-support workflow
```

```python
intent = classify(request)

if intent == "refund":
    handle_refund(request)
elif intent == "order_status":
    handle_order_status(request)
elif intent == "technical_support":
    handle_technical_support(request)
```

The developer defines the allowed branches and what each branch does.

## LLM-powered workflows

The model can select among predetermined branches while the application retains overall control.

```python
intent = llm_classify(request)

if intent == "refund":
    refund_workflow()
elif intent == "shipping":
    shipping_workflow()
elif intent == "technical":
    technical_workflow()
```

This can be described as:

- an LLM-powered workflow;
- model-driven routing; or
- a bounded agentic decision inside a workflow.

The model can choose only from a controlled action set; it cannot invent an arbitrary strategy.

## Workflows as graphs

A workflow can be represented as nodes and allowed transitions.

```text
Request → Router
             ├─→ Refund flow
             ├─→ Shipping flow
             └─→ Technical flow
```

The code determines which transitions exist:

```text
A → B
A → C
B → D
C → E
```

Graph-based frameworks can mix fixed edges with model-selected conditional edges, but drawing a system as a graph does not automatically make it agentic.

## Deterministic and probabilistic components

### Deterministic rule

```python
if refund_amount > 10_000:
    require_human_approval()
```

The same relevant input produces the same outcome.

### Probabilistic judgment

```text
Does the customer's situation appear suspicious?
```

An LLM's result can vary and may require validation.

A strong architecture commonly alternates the two:

```text
Deterministic input checks
        ↓
Model interpretation
        ↓
Deterministic validation
        ↓
Model investigation
        ↓
Deterministic permission policy
        ↓
Human approval when necessary
```

## Dynamic tool selection

Suppose a system exposes:

```text
get_order()
get_tracking()
search_refund_policy()
get_customer_history()
issue_refund()
escalate_to_human()
```

In a workflow, application code selects the tools. In an agent, the model chooses the next permitted tool using the goal, state, and latest observation.

```text
Customer problem
  ↓
Model chooses get_order
  ↓
Order observation
  ↓
Model chooses get_tracking
  ↓
Tracking observation
  ↓
Model chooses policy search or escalation
```

Dynamic tool selection is useful when different cases require different combinations of many possible capabilities.

## Dynamic task decomposition

Tool selection asks:

> What action is needed now?

Task decomposition asks:

> Which smaller problems must be solved to achieve the goal?

For an AWS-to-GCP migration decision, a model might create:

1. Understand the current AWS architecture.
2. Estimate current cost.
3. Map services to GCP equivalents.
4. Estimate GCP cost.
5. Assess migration complexity.
6. Compare performance and operational risks.
7. Produce a recommendation.

If the developer supplied this exact sequence, decomposition is static and workflow-oriented. If the model creates and adapts it from the specific goal, it is agentic planning.

## The autonomy spectrum

Autonomy is a continuum rather than a binary label.

| Level | Architecture | Model control |
|---:|---|---|
| 0 | Traditional deterministic software | None |
| 1 | LLM in a fixed pipeline | Performs a predefined step |
| 2 | LLM router | Chooses among predefined workflows |
| 3 | Dynamic tool selection | Chooses a tool and uses its result |
| 4 | Iterative tool-using agent | Repeatedly selects actions from observations |
| 5 | Planning agent | Creates, executes, verifies, and revises a plan |
| 6 | Long-running autonomous agent | Persists, waits for events, recovers, and collaborates over hours or days |

More autonomy can increase flexibility and capability. It also tends to increase:

- inference and tool cost;
- latency and latency variance;
- possible execution paths;
- testing and debugging difficulty;
- security risk;
- operational complexity;
- unpredictability.

## Uncertainty about “what” versus “how”

### The outcome and process are both known

Use a workflow.

```text
Verify email
→ validate payment
→ create account
→ send welcome email
```

### The goal is known but the path is not

Consider an agent.

```text
Goal: Find the root cause of a production incident.
```

The correct investigation might require logs, metrics, Git history, deployment records, traces, or cloud-status information. Each discovery can determine the next action.

## When agents are useful

Agentic behavior is a reasonable candidate when:

- the correct path cannot be written reliably beforehand;
- later actions depend heavily on intermediate discoveries;
- many tools or information sources may be relevant;
- the problem requires interpretation, investigation, or planning;
- the system must adapt or replan when assumptions fail;
- flexibility produces measurable value over a simpler workflow.

Examples include open-ended research, production-incident investigation, complex support cases, and repository-level debugging.

## When not to use an agent

### The sequence is known

```text
Upload document → extract text → summarize → save
```

Use a workflow.

### Strict correctness is required

Payroll, taxes, permission rules, balances, and policy thresholds belong in deterministic software. An LLM can explain anomalies but should not replace the calculation or policy engine.

### The task is simple CRUD

Changing a phone number usually needs validation and a database update—not planning and a tool loop.

### Latency is tightly constrained

Repeated model and tool calls introduce substantial and variable latency.

### Volume is large and predictable

A fixed process operating on millions of similar records will normally be cheaper, faster, easier to scale, and easier to audit.

### Actions are dangerous or irreversible

For database deletion, large transfers, account termination, or legal publication:

```text
Agent investigates or recommends
        ↓
Deterministic policy checks
        ↓
Human approval
        ↓
Controlled execution
```

### The action space is small

If requests fall into four clear categories, an LLM router followed by four workflows may outperform a general autonomous agent.

## Production architecture patterns

### Router plus workflows

```text
                   ┌─ Billing workflow
                   ├─ Shipping workflow
User → LLM router ─┼─ Refund workflow
                   └─ Technical workflow
```

The model answers only “Which known workflow applies?” Execution remains controlled.

### Workflow containing an agent

```text
Receive ticket
  ↓
Validate account                 ← deterministic
  ↓
Agent investigates               ← agentic
  ├─ search logs
  ├─ query database
  └─ inspect documentation
  ↓
Generate recommendation
  ↓
Human approval                   ← controlled boundary
  ↓
Execute action                   ← deterministic
```

Only the portion that benefits from discovery is autonomous.

### Agent calling workflows as tools

Expose high-level business capabilities:

```text
cancel_order(order_id)
create_return(order_id)
replace_item(order_id)
escalate_case(reason)
```

Each tool can internally execute a deterministic workflow:

```python
def create_return(order_id):
    validate_order(order_id)
    generate_return_label()
    update_order_status()
    schedule_pickup()
    send_confirmation()
```

The model chooses the appropriate business operation but does not control every low-level step.

## Keep autonomy at the highest useful level

Avoid exposing unnecessary primitives such as arbitrary database fields, raw HTTP requests, or unrestricted status mutation.

Prefer a narrow high-level capability:

```text
create_return(order_id)
```

over many low-level capabilities:

```text
set_database_field()
send_http_request()
modify_status()
generate_pdf()
send_email()
```

High-level tools reduce the action space, simplify policies, and make behavior easier to test and audit.

## Autonomy expands the failure surface

A fixed pipeline has a limited set of transitions:

```text
A → B → C
```

An autonomous system might take an unexpected path:

```text
A → D → D → B → F → G → D → ...
```

New failure possibilities include:

- wrong tool selection or arguments;
- unnecessary or repeated actions;
- loops and premature completion;
- incorrect task decomposition;
- tool failures and misleading observations;
- context overflow;
- corrupted or inconsistent state;
- unauthorized actions.

Reliability engineering becomes more important as autonomy increases.

## Architecture decision framework

Ask these questions in order:

1. **Can the correct sequence be written beforehand?** If yes, prefer a workflow.
2. **Does the next action depend on discoveries?** If yes, agentic behavior may help.
3. **Are many possible tools or actions relevant?** If yes, dynamic selection may help.
4. **Does the decision require judgment?** Use a model for ambiguity, not basic arithmetic or known rules.
5. **Can mistakes cause serious harm?** Reduce autonomy and add validation, permissions, and approval.
6. **Does autonomy produce meaningful value?** If a simpler design is equally useful, do not use an agent.

Compact rule:

```text
If the path is known:
    WORKFLOW

If AI only chooses among known paths:
    LLM ROUTER + WORKFLOWS

If the path depends on intermediate discoveries:
    AGENT

If actions have significant risk:
    AGENT + DETERMINISTIC GUARDRAILS + HUMAN APPROVAL
```

## Decision boundaries

A mature system deliberately assigns decisions to code, models, or humans.

| Owner | Best suited for | Examples |
|---|---|---|
| Deterministic code | Rules, validation, permissions, calculations, business constraints | Threshold checks, ownership, payment settlement, return-window expiry |
| Model | Ambiguity, interpretation, investigation, planning, tool selection | User intent, missing information, relevant evidence, next investigation step |
| Human | High-risk, exceptional, uncertain, or irreversible decisions | Large exceptions, account suspension, legal threats, production rollback |

```text
Strong system = code decisions + model decisions + human decisions
```

## Bounded autonomy

Autonomous does not mean unrestricted or unsupervised. It means the system can independently select some actions toward a goal.

Example research-agent boundary:

```text
May:
✓ search the web
✓ open pages
✓ read reports
✓ store notes

May not:
✗ send email
✗ purchase products
✗ modify databases
✗ disclose secrets
```

Production agents should have constraints, permissions, budgets, deadlines, approval checkpoints, and human oversight.

## Examples

### Password reset

```text
Verify identity → send reset link → finish
```

The path is known. A workflow is appropriate.

### Travel planning

A fixed workflow can always ask for dates, search flights and hotels, find attractions, and generate an itinerary. An agent is useful only when it should adapt—such as considering another airport after discovering expensive flights or changing neighborhoods after checking travel time.

### Research

Investigating a company's revenue decline is agent-shaped because earnings reports, competitors, industry data, regions, pricing, and management commentary may generate new questions.

### Production incident

An API error increase may come from a deployment, database overload, dependency outage, traffic spike, configuration, DNS, or certificates. The correct diagnostic path emerges from metrics, logs, and other observations.

## Customer-support problem: three architectures

Problem:

> A laptop was ordered five days ago. Tracking has not updated for three days, and the customer needs it tomorrow.

Available capabilities:

```text
get_order(order_id)
get_tracking(tracking_id)
search_shipping_policy(query)
check_replacement_stock(product_id)
create_replacement(order_id)
create_refund(order_id)
escalate_to_human(reason)
```

### A — Deterministic workflow

```text
Extract order ID
  ↓
Get order
  ↓
Get tracking
  ↓
Tracking stale for more than 48 hours?
  ├─ no  → explain current status
  └─ yes → check policy
              ↓
           replacement allowed?
              ├─ yes → check inventory → offer replacement if available
              └─ no  → escalate
```

Advantages: predictable, inexpensive, fast, and easy to test. Disadvantage: all important cases must be anticipated.

### B — LLM-powered workflow

The model extracts structured case data and writes a personalized response:

```json
{
  "intent": "shipping_problem",
  "urgency": "high",
  "reason": "customer needs item tomorrow",
  "order_id": "123"
}
```

Code retrieves the order and tracking data, applies the business policy, and determines the allowed resolution.

```text
Model: language interpretation, urgency detection, response writing
Code:  policy, validation, permissions, execution
```

This is often an excellent production architecture.

### C — Autonomous support agent

The agent dynamically checks the order, tracking, delayed-shipment policy, and inventory. If it discovers that only a more expensive replacement exists, it avoids unauthorized substitution and escalates with the collected evidence.

Its next actions depend on observations, so it is an agent.

### Comparison

| Property | Workflow | LLM workflow | Autonomous agent |
|---|---|---|---|
| Path predetermined | Yes | Mostly | No |
| LLM required | No | Yes | Yes |
| Dynamic tool choice | No | Limited | Yes |
| Replanning | No | Limited | Yes |
| Predictability | Very high | High | Lower |
| Flexibility | Low | Medium | High |
| Cost and latency | Low | Medium | Higher |
| Testing difficulty | Low | Medium | High |
| Ambiguous cases | Weak | Moderate | Strong |

The strongest deployment is likely a hybrid:

```text
LLM understands request
  ↓
Agent investigates with read tools
  ↓
Agent proposes resolution
  ↓
Deterministic policy engine
  ├─ safe action      → controlled execution
  └─ sensitive action → human approval
```

## Exercise — Investigate a slow SaaS API

Problem:

> My SaaS application's API has become much slower since yesterday. Find out what happened.

Available capabilities:

```text
query_metrics()
search_logs()
get_recent_deployments()
query_database_metrics()
check_cloud_status()
search_code()
create_incident_report()
restart_service()
rollback_deployment()
```

### A — Fixed workflow

Write the predetermined diagnostic sequence:

```text
1.
2.
3.
4.
5.
```

### B — LLM-powered workflow

Document the boundary:

```text
Model decides:
-

Code decides:
-
```

### C — Autonomous agent

Describe at least three possible observation-dependent branches. For example, how should the next action differ after discovering database saturation versus a recent deployment?

### D — Actions that should not be autonomous

Decide whether these require human approval and explain why:

- `restart_service()`
- `rollback_deployment()`

Also specify read-only permissions, action budgets, stopping conditions, and the evidence required before recommending a production change.

## Completion checklist

- [ ] I can explain who owns control flow in a workflow and in an agent.
- [ ] I can distinguish fixed, conditional, and LLM-powered workflows.
- [ ] I can explain dynamic tool selection and dynamic task decomposition.
- [ ] I can place a design on the autonomy spectrum.
- [ ] I can explain when an agent should not be used.
- [ ] I can identify decisions that belong to code, a model, or a human.
- [ ] I can describe router-plus-workflows, workflow-containing-agent, and agent-calling-workflows patterns.
- [ ] I completed all four parts of the slow-API exercise.
- [ ] I can defend the minimum useful autonomy for my chosen architecture.

## Final takeaway

> Do not ask only, “Can I use an agent here?” Ask, “Which decisions genuinely require model autonomy, and which should remain deterministic?”

Workflows and agents are complementary:

```text
Workflows solve predictable processes.
Agents solve adaptive decision-making.
Strong systems combine them deliberately.
```

The next topic is **Anatomy of an Agent**, where these control decisions become concrete components: goal, model, instructions, tools, state, environment, observations, actions, loop, and termination conditions.

