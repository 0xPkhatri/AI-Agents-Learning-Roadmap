# 04 — LLM APIs for Agent Development

- **Phase:** 2 — LLM Application Fundamentals
- **Depth:** Intermediate
- **Prerequisites:** Python or JavaScript; Topics 01–03
- **Status:** In progress
- **Started:** 2026-08-19
- **Source:** User-provided lesson (`S16`) plus current primary documentation (`S17`–`S21`)

## Learning outcome

After this topic, I should be able to:

- explain the lifecycle of one LLM API request;
- keep trusted instructions, user input, and untrusted external data separate;
- manage conversation state explicitly;
- measure tokens, latency, and usage;
- use streaming, async execution, timeouts, and basic error handling;
- select models by capability, quality, latency, and cost; and
- build a configurable, multi-turn CLI assistant without an agent framework.

## Central mental model

```text
Application state + instructions + input
                   ↓
              API request
                   ↓
                 Model
                   ↓
     Structured response or event stream
                   ↓
      Application interprets the result
                   ↓
      Update state, act, retry, or finish
```

> The model is a service invoked by the application. The application owns state, control flow, credentials, permissions, error handling, and logging.

An agent is built from repeated, controlled versions of this atomic operation.

## 1. What an LLM API is

An LLM API lets software invoke a model programmatically.

```text
User → provider website → model
```

becomes:

```text
User → my application → HTTP/API → model provider
                              ↓
                    structured response
                              ↓
                       my application
```

Conceptually:

```python
response = client.generate(
    model="model-name",
    input="Explain REST APIs."
)

print(response.text)
```

The exact request and response schemas differ by provider. The stable concepts are:

```text
Request  = model + input/context + configuration
Response = output/content + metadata + usage + status/errors
```

## 2. Why APIs are the foundation of agents

A normal LLM application may make one call. An agent may make several:

```text
Call model
  ↓
Model requests search
  ↓
Application executes search
  ↓
Call model with the observation
  ↓
Model chooses another action or finishes
```

Before implementing an agent loop, I need to understand one model call completely.

## 3. Request lifecycle

One request generally passes through:

```text
1. Application builds context and configuration
2. Client serializes the request
3. Request travels over the network
4. Provider authenticates and validates it
5. Model processes the input tokens
6. Model generates output
7. Provider packages output, metadata, and usage
8. Application receives and validates the response
9. Application updates its own state
```

Conceptual data:

```json
{
  "model": "model-name",
  "input": "Explain REST APIs.",
  "max_output_tokens": 500
}
```

```json
{
  "id": "response_xyz",
  "output": "A REST API is...",
  "usage": {
    "input_tokens": 12,
    "output_tokens": 93
  }
}
```

Do not memorize this illustrative JSON as a provider contract.

## 4. Authentication and secret handling

Providers usually authenticate requests with an API key or another credential. The credential identifies the account, permissions, billing, and applicable limits.

Never commit keys to source control:

```python
# Bad
API_KEY = "secret-value"
```

Load them from the environment during local development:

```bash
export LLM_API_KEY="..."
```

```python
import os

api_key = os.environ["LLM_API_KEY"]
```

Production secrets belong in a managed secret store. Agent credentials deserve special care because they may also authorize databases, email, code hosts, and business APIs.

## 5. Messages, content items, and roles

Modern APIs accept structured context. A simplified message list might look like:

```python
messages = [
    {"role": "user", "content": "My project is called Aurora."},
    {"role": "assistant", "content": "Got it."},
    {"role": "user", "content": "What is my project called?"},
]
```

Important categories include:

- application or developer instructions;
- user input;
- assistant/model output;
- tool requests and results; and
- retrieved or externally supplied data.

Exact role names and content formats vary by provider.

### Authority and trust boundaries

Keep these concepts distinct:

```text
Trusted application policy
        ↓
User task
        ↓
Untrusted tool, web, and document data
```

Example:

```text
Instruction: You are a read-only SQL assistant. Never write data.
User:        Show yesterday's sales.
```

Separating instructions from task data makes policy easier to understand and later helps defend against prompt injection.

## 6. Conversation state is not automatic magic

Two independent requests do not necessarily share state:

```python
client.generate("My name is Alex.")
client.generate("What is my name?")
```

The second call must be connected to the first somehow.

### Application-managed continuation

The application resends relevant history:

```python
messages = [
    {"role": "user", "content": "My name is Alex."},
    {"role": "assistant", "content": "Nice to meet you."},
    {"role": "user", "content": "What is my name?"},
]
```

Benefits: explicit, inspectable, and conceptually portable. Cost: context grows over time.

### Provider-managed continuation

Some APIs can connect a new request to a prior response or conversation identifier. This can simplify request construction, but the application must still understand ownership, storage, retention, and provider-specific behavior.

> Conversation history is one kind of application state. It is not the same as persistent personal memory.

## 7. What can enter one model call?

An agent call may contain:

```text
Instructions
User goal
Conversation history
Current task state
Available tool definitions
Previous tool results
Retrieved documents
Relevant memory
```

Context engineering means selecting the smallest set that is relevant to the current decision.

## 8. Model selection and routing

Choose a model for the workload rather than treating one model as universally best.

Compare:

- reasoning and instruction-following quality;
- speed and time to first token;
- price;
- context capacity;
- tool-calling and structured-output support;
- coding ability;
- supported modalities; and
- reliability on the actual evaluation set.

Example routing:

| Task | Useful model profile |
|---|---|
| Intent classification | Fast, inexpensive, reliable structured output |
| Complex research | Strong reasoning and tool use |
| Short title generation | Fast and inexpensive |
| Screenshot analysis | Vision-capable |

Model names and limits change. Verify current official documentation when implementing.

## 9. Generation controls

Depending on the model, APIs may expose:

- temperature or sampling controls;
- maximum output tokens;
- stop sequences;
- reasoning controls;
- response schemas or formats; and
- tool-choice controls.

### Temperature

A rough mental model:

```text
Lower temperature → choices are more concentrated
Higher temperature → choices may be more varied
```

This does not make an agent deterministic. Model behavior can vary even with conservative settings, and not every model exposes the same controls.

### Output limits

Constrain output to reduce latency and cost:

```python
response = client.generate(
    input=prompt,
    max_output_tokens=500,
)
```

A short classification or tool-selection decision should not receive the same output budget as a full report.

### Do not parse exact prose

Fragile:

```python
if response == "YES":
    execute()
```

Prefer a validated schema:

```json
{"approved": true}
```

Structured outputs are covered in Topic 06.

## 10. Tokens and context windows

Models process tokens rather than words or characters directly.

```text
text or media → token representation → model
```

Usage is often divided into:

```text
Input tokens  = instructions + history + tools + retrieved data + input
Output tokens = generated response
```

Tokens affect:

- cost;
- latency;
- rate limits; and
- how much information fits in one request.

### Context window

The context window is the bounded capacity available to one invocation. It may include inputs, tool definitions and results, images or documents, and generated output depending on the API.

More capacity does not mean “send everything.” Excess context causes:

- higher cost and latency;
- irrelevant or conflicting information;
- context pollution; and
- harder debugging.

### Cost compounds inside a loop

If successive calls contain 5k, 6k, 7k, 8k, and 9k input tokens, the run processes 35k input tokens—not merely the final 9k. Every call has its own input.

## 11. Context, rate, and spend limits are different

| Limit | Meaning |
|---|---|
| Context limit | Maximum content a single invocation can process |
| Rate limit | Requests or tokens allowed over a time period |
| Spend limit | Billing usage allowed over a period |
| Output limit | Maximum response generation for a call |

Rate limiting may produce an HTTP `429` response. Applications should respect provider guidance such as retry timing and should control concurrency rather than immediately repeating every request.

## 12. Errors, timeouts, and retries

Expected error categories include:

```text
Authentication failure
Invalid request
Context too large
Rate limit
Timeout
Network failure
Provider/server failure
Unavailable model
Safety or policy rejection
```

Always use finite timeouts for remote calls. One stuck request must not freeze an entire agent run.

Classify errors before retrying:

```text
Error
  ↓
Retryable?
  ├─ Yes → wait with backoff, then retry within a budget
  └─ No  → fail clearly, use a fallback, or escalate
```

A temporary server error may be retryable. An invalid API key or malformed schema is not fixed by blind retries.

Basic boundary:

```python
try:
    response = client.generate(...)
except RateLimitError:
    handle_rate_limit()
except TimeoutError:
    handle_timeout()
except AuthenticationError:
    fail_configuration()
```

## 13. Latency, sync, async, and concurrency

Latency is the duration from sending a request until receiving the relevant response.

An eight-step agent can accumulate model and tool latency. Sequential calls are necessary when each depends on the previous result, but independent work may run concurrently.

### Synchronous

```python
response = client.generate(...)
```

Execution waits until the call completes. This is simple and suitable for many programs.

### Asynchronous

```python
response = await client.generate(...)
```

Async lets the runtime schedule other I/O while waiting. Independent operations can be gathered concurrently:

```python
results = await gather(
    research_company("A"),
    research_company("B"),
    research_company("C"),
)
```

Async is a scheduling/performance technique; it does not make the model or agent more intelligent.

## 14. Streaming

Without streaming, the application waits for a complete response. With streaming, it receives incremental events or text deltas.

```python
full_response = ""

for event in client.stream(input=prompt):
    if event.has_text:
        print(event.text, end="", flush=True)
        full_response += event.text
```

The full response is still normally stored in conversation state after the stream completes.

Streaming improves perceived responsiveness and time to first token, but it may not reduce total generation time.

Agent streams may contain lifecycle events rather than text alone:

```text
run_started
model_started
model_delta
tool_called
tool_completed
final_delta
run_completed
```

## 15. Multimodality

Depending on the model and API, inputs may include:

- text;
- images;
- audio;
- video; and
- documents.

This expands what an agent can observe. A browser agent may combine a screenshot, page structure, and goal; a coding agent may inspect an error screenshot, repository files, and terminal output.

Multimodal inputs still consume processing capacity and may affect token accounting, latency, and cost. Current provider-specific rules must be checked before implementation.

## 16. Responses are not necessarily plain text

A model response may contain:

- text;
- structured data;
- one or more tool calls;
- a refusal;
- usage or other metadata;
- audio; or
- streamed lifecycle events.

Use this mental model:

> Structured context in → structured response or event stream out.

## 17. One API, many architectural jobs

Model calls can perform:

| Job | Example question |
|---|---|
| Classification | Which category is this request? |
| Extraction | What is the order ID? |
| Tool selection | Which capability is needed next? |
| Planning | What subtasks are required? |
| Evaluation | Did the result satisfy the goal? |
| Synthesis | What final answer follows from the evidence? |

Use stronger reasoning only where it produces measured value. A difficult plan may need a different model profile from a simple label extraction.

## 18. Model quality is not a substitute for architecture

When an agent fails, the cause may be:

```text
Bad instructions
Missing or polluted context
Poor tool descriptions
Incorrect tool result
Broken state update
Weak retrieval
Missing permissions
Bad stop conditions
```

A stronger model may hide these defects temporarily. Diagnose the system boundary before changing models.

## 19. Observe every model call

Record at least:

```text
timestamp
model and configuration
latency
input tokens
output tokens
success or error category
```

Example:

```python
{
    "model": "model-alias",
    "latency_ms": 1850,
    "input_tokens": 842,
    "output_tokens": 152,
    "success": True,
}
```

Without measurements, I cannot answer why an agent is slow, costly, or unreliable.

## 20. A thin `ModelClient` abstraction

Do not scatter provider calls throughout the project. Start with one provider and isolate it behind a small client:

```python
class ModelClient:
    def generate(
        self,
        messages,
        model=None,
        max_output_tokens=None,
    ):
        ...
```

This is a natural place for:

- timeouts;
- logging;
- usage normalization;
- error mapping; and
- limited retry behavior.

Avoid premature “universal” abstractions. Learn one provider, add a second, observe the differences, then extract only the stable common interface. Preserve an escape hatch for provider-specific features.

## 21. Practical project — configurable CLI assistant

Build this without LangGraph, CrewAI, AutoGen, or an agent SDK.

### Required features

- API key loaded from the environment;
- configurable model;
- system/developer instructions;
- multi-turn conversation;
- conversation reset;
- streaming on/off;
- input and output token tracking;
- latency measurement;
- graceful error handling;
- `/exit`; and
- `/stats`.

### Optional features

- async API client;
- two providers;
- model switching;
- image input; and
- save/load conversation JSON.

### Suggested structure

```text
llm_cli/
├── main.py
├── client.py
├── config.py
└── .env.example
```

The real `.env` must be ignored by Git.

### Conversation loop

```python
messages = []

while True:
    user_input = input("You: ")

    if user_input == "/exit":
        break

    if user_input == "/clear":
        messages = []
        continue

    messages.append({
        "role": "user",
        "content": user_input,
    })

    response = client.generate(
        instructions=SYSTEM_INSTRUCTIONS,
        messages=messages,
        model=config["model"],
    )

    print("Assistant:", response.text)

    messages.append({
        "role": "assistant",
        "content": response.text,
    })
```

### Usage and timing

```python
stats = {
    "calls": 0,
    "input_tokens": 0,
    "output_tokens": 0,
    "latency_seconds": 0.0,
}
```

`/stats` should show call count, cumulative token usage, and average latency. Measure rather than guess.

### Useful commands

```text
/exit
/clear
/stats
/model
/config
```

## 22. Experiments to perform

### A. Conversation continuation

```text
My project is called Aurora.
What is my project called?
/clear
What is my project called?
```

This demonstrates that conversation context is not permanent memory.

### B. Context growth

Record input tokens each turn in a long conversation. History is repeatedly processed, so token usage normally grows unless context is pruned or summarized.

### C. Streaming

Compare:

```text
TTFT          = time to first token
Total latency = time until completion
```

### D. Model selection

Run the same models on extraction, explanation, difficult reasoning, and long-document summarization. Compare quality, latency, and usage.

### E. Output limits

Ask for a detailed explanation with a very small output budget, then increase it. Observe truncation and latency differences.

### F. Failure handling

In a safe local setup, test an invalid model or invalid credential. Record the error type and ensure no secret is logged.

## 23. Transition from LLM app to agent

Current CLI:

```python
while True:
    user_input = input()
    response = model(messages)
    print(response)
```

Later agent loop:

```python
while not done:
    response = model(messages, tools=tools)

    if response.has_tool_call:
        result = execute_tool(response.tool_call)
        messages.append(result)
        continue

    return response.text
```

```text
LLM application
  ↓ structured decisions
  ↓ tools
  ↓ repeated action/observation loop
Agent
```

## 24. Completion checklist

I should be able to answer:

- What happens from request construction through state update?
- Who owns conversation history?
- How are instructions different from user input and tool data?
- Why do tokens matter for context, cost, latency, and limits?
- What is the difference between context, rate, spend, and output limits?
- What do streaming and async each solve?
- Which errors are retryable?
- Why are timeouts mandatory?
- How do multimodal inputs affect agent observations?
- Why should a model client remain thin?
- Can I build the CLI assignment without copying a framework tutorial?

## Final takeaway

```text
Application owns:
state + control flow + credentials + errors + logs + permissions

Model provides:
generated or structured decisions from the supplied context
```

> An agent is ultimately built from repeated, controlled model API calls. The application constructs context, invokes the model, interprets the response, updates state, and controls everything around that invocation.

## Sources

- `S16` — user-provided lesson, condensed and reorganized here.
- `S17` — [OpenAI Responses API reference](https://platform.openai.com/docs/api-reference/responses).
- `S18` — [Claude API rate limits](https://platform.claude.com/docs/en/api/rate-limits).
- `S19` — [Claude vision](https://platform.claude.com/docs/en/build-with-claude/vision).
- `S20` — [Claude context windows](https://platform.claude.com/docs/en/build-with-claude/context-windows).
- `S21` — [Gemini token counting](https://ai.google.dev/gemini-api/docs/tokens).

Provider-specific model names, limits, controls, pricing, and response schemas are intentionally not frozen in these notes. Verify them in current primary documentation during implementation.
