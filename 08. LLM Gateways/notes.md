# LLM Gateways

> A comprehensive guide to LLM Gateways for Agentic AI systems

---

## Table of Contents

1. [What is an LLM Gateway?](#1-what-is-an-llm-gateway)
2. [Why Do We Need LLM Gateways?](#2-why-do-we-need-llm-gateways)
3. [The Problem Without a Gateway](#3-the-problem-without-a-gateway)
4. [Gateway Architecture](#4-gateway-architecture)
5. [LLM Gateway vs API Gateway](#5-llm-gateway-vs-api-gateway)
6. [LLM Gateway vs LLM Router](#6-llm-gateway-vs-llm-router)
7. [Core Responsibilities](#7-core-responsibilities)
8. [Provider Abstraction](#8-provider-abstraction)
9. [Model Routing](#9-model-routing)
10. [Routing Strategies](#10-routing-strategies)
11. [Fallbacks](#11-fallbacks)
12. [Retries](#12-retries)
13. [Timeouts](#13-timeouts)
14. [Load Balancing](#14-load-balancing)
15. [Rate Limiting](#15-rate-limiting)
16. [Concurrency Control](#16-concurrency-control)
17. [Caching](#17-caching)
18. [Cost Management](#18-cost-management)
19. [Observability](#19-observability)
20. [Logging](#20-logging)
21. [Tracing](#21-tracing)
22. [Metrics](#22-metrics)
23. [Streaming](#23-streaming)
24. [Structured Outputs](#24-structured-outputs)
25. [Tool Calling](#25-tool-calling)
26. [Embeddings and Other Models](#26-embeddings-and-other-models)
27. [Security](#27-security)
28. [Prompt and Data Privacy](#28-prompt-and-data-privacy)
29. [Guardrails](#29-guardrails)
30. [Model Selection for Agents](#30-model-selection-for-agents)
31. [LLM Gateways in Agentic AI](#31-llm-gateways-in-agentic-ai)
32. [Multi-Agent Systems](#32-multi-agent-systems)
33. [Gateway Request Lifecycle](#33-gateway-request-lifecycle)
34. [Example Architecture](#34-example-architecture)
35. [Designing a Gateway](#35-designing-a-gateway)
36. [Minimal Gateway Implementation](#36-minimal-gateway-implementation)
37. [Production Gateway](#37-production-gateway)
38. [Failure Scenarios](#38-failure-scenarios)
39. [Common Design Mistakes](#39-common-design-mistakes)
40. [Gateway vs Direct Provider Calls](#40-gateway-vs-direct-provider-calls)
41. [Open-Source LLM Gateway Tools](#41-open-source-llm-gateway-tools)
42. [Learning Roadmap](#42-learning-roadmap)
43. [Interview Questions](#43-interview-questions)
44. [Key Takeaways](#44-key-takeaways)

---

## 1. What is an LLM Gateway?

An **LLM Gateway** is an infrastructure layer that sits between an application/agent and one or more Large Language Model providers.

Instead of an application directly calling:

```
Application
     |
     v
OpenAI
```

we introduce:

```
Application
     |
     v
LLM Gateway
     |
     +--------> OpenAI
     |
     +--------> Anthropic
     |
     +--------> Google
     |
     +--------> Local Models
     |
     +--------> Other Providers
```

The gateway provides a centralized interface for interacting with LLMs.

The application can send:

```json
{
  "model": "reasoning",
  "messages": [
    {
      "role": "user",
      "content": "Explain quantum computing"
    }
  ]
}
```

The gateway decides:

- which provider to use
- which model to use
- whether a fallback is necessary
- whether the request can be served from cache
- whether the user is within rate limits
- how many retries are allowed
- how much the request costs
- what telemetry should be recorded
- whether the request is allowed
- whether sensitive data needs to be filtered

## 2. Why Do We Need LLM Gateways?

A simple application might initially look like:

```
User
 |
 v
Backend
 |
 v
LLM Provider
```

This works well for prototypes.

However, production AI systems quickly become more complicated.

For example:

```
Agent
 |
 +--> GPT model
 |
 +--> Claude
 |
 +--> Gemini
 |
 +--> Local model
 |
 +--> Embedding model
 |
 +--> Reranker
```

Now the application needs to manage:

- multiple API keys
- multiple SDKs
- different APIs
- different model names
- different pricing
- different rate limits
- provider failures
- retries
- fallbacks
- observability
- cost tracking
- security
- model routing

This logic becomes duplicated throughout the application.

An LLM Gateway centralizes it.

## 3. The Problem Without a Gateway

Suppose an agent directly uses three providers.

```python
if task == "reasoning":
    client = OpenAI(...)

elif task == "cheap":
    client = Gemini(...)

elif task == "long_context":
    client = Anthropic(...)
```

Now imagine OpenAI becomes unavailable.

The application needs:

```python
try:
    response = openai_call()
except:
    response = anthropic_call()
```

Then suppose the request receives:

```
429 Too Many Requests
```

You need retry logic.

Then:

```
500 Internal Server Error
```

You need another retry.

Then:

```
Timeout
```

You need timeout handling.

Then:

```
Token budget exceeded
```

You need another model.

Then the finance team asks:

> How much did Agent A spend on LLMs last month?

You need centralized cost tracking.

Then security asks:

> Which requests contained sensitive information?

You need centralized logging or inspection.

The complexity grows rapidly.

## 4. Gateway Architecture

A production-oriented LLM Gateway can look like this:

```
                         ┌──────────────────────┐
                         │      AI Agent        │
                         └──────────┬───────────┘
                                    │
                                    v
                         ┌──────────────────────┐
                         │    LLM Gateway       │
                         │                      │
                         │ Authentication       │
                         │ Rate Limiting        │
                         │ Routing              │
                         │ Model Selection      │
                         │ Retry                │
                         │ Fallback             │
                         │ Caching              │
                         │ Guardrails           │
                         │ Observability        │
                         │ Cost Tracking        │
                         └──────────┬───────────┘
                                    │
                 ┌──────────────────┼──────────────────┐
                 │                  │                  │
                 v                  v                  v
            ┌─────────┐        ┌─────────┐        ┌─────────┐
            │ OpenAI  │        │Anthropic│        │ Gemini  │
            └─────────┘        └─────────┘        └─────────┘
                 │                  │                  │
                 v                  v                  v
            ┌─────────┐        ┌─────────┐        ┌─────────┐
            │ Models  │        │ Models  │        │ Models  │
            └─────────┘        └─────────┘        └─────────┘
```

The gateway effectively becomes the control plane for model inference.

## 5. LLM Gateway vs API Gateway

These concepts are related but different.

### API Gateway

An API Gateway manages application APIs.

For example:

```
Client
 |
 v
API Gateway
 |
 +--> User Service
 +--> Payment Service
 +--> Order Service
```

Common responsibilities:

- authentication
- authorization
- routing
- rate limiting
- request validation
- TLS
- logging

### LLM Gateway

An LLM Gateway performs similar infrastructure responsibilities but understands LLM-specific behavior.

For example:

- tokens
- context windows
- model capabilities
- prompt caching
- model pricing
- generation parameters
- tool calls
- structured output
- streaming
- model fallback
- provider-specific errors

Therefore:

```
API Gateway
    ↓
General API traffic

LLM Gateway
    ↓
LLM inference traffic
```

An LLM Gateway can exist behind an API Gateway:

```
Client
 |
 v
API Gateway
 |
 v
Application
 |
 v
LLM Gateway
 |
 +--> OpenAI
 +--> Anthropic
 +--> Gemini
```

## 6. LLM Gateway vs LLM Router

These terms are often used interchangeably, but conceptually they are different.

### LLM Router

Primarily focuses on:

> Which model should handle this request?

For example:

```
Simple request
     |
     v
Cheap model

Complex reasoning
     |
     v
Reasoning model
```

### LLM Gateway

Provides a broader infrastructure layer:

- Authentication
- Rate limits
- Routing
- Retries
- Fallbacks
- Caching
- Observability
- Cost tracking
- Security
- Provider abstraction

A router can therefore be considered one component of a gateway.

## 7. Core Responsibilities

A mature LLM Gateway may provide:

```
                    LLM Gateway
                         |
       ┌─────────────────┼─────────────────┐
       │                 │                 │
       v                 v                 v
    Routing          Reliability       Governance
       │                 │                 │
       ├─ model          ├─ retry         ├─ auth
       ├─ provider       ├─ timeout       ├─ rate limits
       ├─ load balance   ├─ fallback      ├─ budgets
       └─ capability     └─ circuit break └─ policies

       ┌─────────────────┼─────────────────┐
       │                 │                 │
       v                 v                 v
   Performance       Observability       Security
       │                 │                 │
       ├─ cache          ├─ logs           ├─ PII
       ├─ batching       ├─ metrics        ├─ filtering
       └─ streaming      └─ tracing       └─ guardrails
```

## 8. Provider Abstraction

One of the biggest benefits of a gateway is provider abstraction.

Without abstraction:

```python
openai_client.responses.create(...)
anthropic_client.messages.create(...)
gemini_client.generate_content(...)
```

Every provider has different APIs.

A gateway can expose one internal interface:

```python
response = gateway.generate(
    model="reasoning",
    messages=messages
)
```

The gateway translates that into the appropriate provider API.

### Logical Model Names

Instead of exposing provider-specific names to your application:

- `gpt-5.x`
- `claude-...`
- `gemini-...`

the application can use logical aliases:

- `fast`
- `cheap`
- `reasoning`
- `coding`
- `long_context`
- `vision`

Example:

```yaml
models:

  fast:
    provider: openai
    model: some-fast-model

  reasoning:
    provider: anthropic
    model: some-reasoning-model

  cheap:
    provider: google
    model: some-efficient-model
```

The application only knows:

```python
gateway.generate(model="reasoning")
```

This creates model indirection.

That is extremely useful because the underlying model can change without changing application code.

## 9. Model Routing

Model routing means deciding:

> Which model should process this request?

A gateway can route based on:

- task
- latency
- cost
- model capability
- context size
- token count
- user tier
- region
- availability
- historical performance
- reliability
- privacy requirements

## 10. Routing Strategies

### 10.1 Static Routing

The simplest approach.

```
Task = coding
      ↓
Coding Model
```

Configuration:

```yaml
routing:

  coding:
    model: coding-model

  summarization:
    model: fast-model

  reasoning:
    model: reasoning-model
```

**Advantages:**

- simple
- predictable
- easy to debug

**Disadvantages:**

- doesn't adapt dynamically

### 10.2 Cost-Based Routing

Choose a model based on cost.

Example:

```
Simple query
     ↓
Cheap model

Complex query
     ↓
Expensive model
```

A gateway can estimate:

```
estimated_cost =
input_tokens × input_price
+
output_tokens × output_price
```

Then select a model that satisfies:

```
cost <= budget
```

### 10.3 Latency-Based Routing

If low latency is important:

```
Provider A → 500 ms
Provider B → 800 ms
Provider C → 300 ms

Route to:
Provider C
```

However, routing solely by the latest latency measurement can create instability.

A better system may use:

```
moving average latency
+
error rate
+
availability
```

### 10.4 Capability-Based Routing

Different models have different capabilities.

Example:

```
Vision request
    ↓
Vision-capable model

Tool calling request
    ↓
Tool-capable model

Huge context request
    ↓
Long-context model

Reasoning request
    ↓
Reasoning model
```

The gateway maintains model metadata:

```json
{
  "model": "example-model",
  "capabilities": {
    "vision": true,
    "tool_calling": true,
    "structured_output": true
  },
  "context_window": 100000
}
```

### 10.5 Semantic Routing

The gateway first determines the type or complexity of the request.

Example:

```
User Request
     |
     v
Router Model / Classifier
     |
     +---- simple ------> cheap model
     |
     +---- coding ------> coding model
     |
     +---- reasoning ---> reasoning model
```

This is called **semantic routing**.

The routing model itself introduces latency and cost, so it should only be used when the expected benefit justifies it.

### 10.6 Rule-Based Routing

Example:

```python
if token_count < 1000:
    model = "fast"

elif task == "coding":
    model = "coding"

elif token_count > 50000:
    model = "long_context"

else:
    model = "reasoning"
```

This is often sufficient for many systems.

### 10.7 Weighted Routing

Traffic can be distributed:

```
Model A → 70%
Model B → 30%
```

Useful for:

- experimentation
- A/B testing
- gradual migration
- canary deployments

Example:

```
100 requests

70 → Model A
30 → Model B
```

## 11. Fallbacks

A fallback means:

> If the preferred provider/model fails, try another compatible provider/model.

Example:

```
Primary
  ↓
OpenAI
  ↓
Failure
  ↓
Anthropic
  ↓
Failure
  ↓
Google
```

### Types of Failure

Not every failure should trigger fallback.

**Usually retryable:**

- 429 Too Many Requests
- 500 Internal Server Error
- 502 Bad Gateway
- 503 Service Unavailable
- 504 Gateway Timeout
- Network timeout

**Usually not retryable:**

- Invalid API key
- Invalid request
- Malformed schema
- Unsupported capability
- Authentication failure

Blindly falling back on every error can hide application bugs.

## 12. Retries

Retries are essential because distributed systems fail.

Example:

```
Request
   |
   v
Provider
   |
   X
temporary failure
   |
   v
Retry
```

### Exponential Backoff

Instead of:

```
retry immediately
retry immediately
retry immediately
```

use:

```
attempt 1 → wait 0.5s
attempt 2 → wait 1s
attempt 3 → wait 2s
attempt 4 → wait 4s
```

A common formula is:

```
delay = base × 2^attempt
```

Usually add jitter:

```
delay = random(0, base × 2^attempt)
```

This prevents many clients from retrying simultaneously.

## 13. Timeouts

Every provider request should have a timeout.

Without a timeout:

```
Agent
  |
  v
Gateway
  |
  v
Provider
  |
  X
hangs forever
```

With timeout:

```
Gateway
  |
  |---- request
  |
  |---- timeout
  |
  v
Fallback
```

Different timeout types can exist:

- connection timeout
- request timeout
- stream timeout
- overall workflow timeout

## 14. Load Balancing

Suppose multiple providers support the same capability.

```
Gateway
 |
 +--> Provider A
 |
 +--> Provider B
 |
 +--> Provider C
```

The gateway can distribute traffic.

### Round Robin

```
A
B
C
A
B
C
```

Simple but doesn't consider performance.

### Weighted Round Robin

```
A → 50%
B → 30%
C → 20%
```

### Least Loaded

Route to the provider with the lowest current load.

### Latency-Aware

Prefer providers with lower recent latency.

### Error-Aware

Avoid providers experiencing elevated error rates.

## 15. Rate Limiting

LLM providers have limits.

For example:

- Requests per minute
- Tokens per minute
- Concurrent requests
- Daily usage

A gateway can enforce its own limits.

Example:

```
User A
100 requests/min

User B
20 requests/min

Free tier
10 requests/min
```

### Token-Based Rate Limiting

Request count isn't always enough.

Consider:

```
Request A = 100 tokens

Request B = 100,000 tokens
```

Both count as one request but have radically different resource usage.

Therefore gateways can rate-limit using:

```
tokens / minute
```

## 16. Concurrency Control

LLM calls can be expensive and slow.

Suppose an agent launches:

```
100 parallel LLM requests
```

This can overwhelm:

- your gateway
- provider limits
- network resources
- your budget

A gateway can enforce:

```
max concurrent requests = 20
```

Requests beyond that can:

- queue
- reject
- or degrade

## 17. Caching

Caching can significantly reduce:

- latency
- cost
- provider load

Suppose:

> "What is the capital of France?"

is requested repeatedly.

Instead of:

```
Request
 ↓
LLM
```

the gateway can do:

```
Request
 ↓
Cache lookup
 ↓
Hit
 ↓
Return response
```

### Exact Cache

Key:

```
hash(request)
```

If identical request:

```
cache hit
```

### Semantic Cache

Two requests may be different strings but have the same meaning.

Example:

```
"What is the capital of France?"

"Which city is France's capital?"
```

A semantic cache can embed both queries and determine similarity.

Conceptually:

```
Query
 ↓
Embedding
 ↓
Vector search
 ↓
Similar previous query?
 ↓
Return cached result
```

Semantic caching requires careful thresholds because semantically similar does not necessarily mean answer-equivalent.

## 18. Cost Management

LLM systems can become expensive quickly.

A gateway provides centralized cost accounting.

For each request:

- provider
- model
- input tokens
- output tokens
- cached tokens
- cost
- user
- agent
- workflow
- timestamp

Example:

```json
{
  "agent": "research-agent",
  "model": "reasoning-model",
  "input_tokens": 12000,
  "output_tokens": 3000,
  "cost_usd": 0.42
}
```

### Budgets

You can define:

- User budget
- Agent budget
- Project budget
- Daily budget
- Monthly budget

Example:

```
monthly_budget = $100

current_usage = $98

remaining = $2
```

The gateway can then:

- switch to cheaper model

or:

- reject request

or:

- require approval

depending on policy.

## 19. Observability

LLM systems require more than normal application monitoring.

You need visibility into:

- What model was called?
- Why?
- How many tokens?
- How long did it take?
- What did it cost?
- Did it fail?
- Did it retry?
- Did fallback occur?
- Which agent initiated it?
- Which workflow?

A gateway provides centralized observability.

## 20. Logging

A useful gateway log might contain:

```json
{
  "request_id": "req_123",
  "agent_id": "research-agent",
  "model": "reasoning-model",
  "provider": "provider-a",
  "input_tokens": 4200,
  "output_tokens": 1300,
  "latency_ms": 1840,
  "status": "success",
  "retry_count": 0,
  "fallback": false
}
```

### What Should Not Be Logged?

Be careful with:

- passwords
- API keys
- access tokens
- personal information
- confidential documents
- secrets
- sensitive prompts

Prompt logging should be configurable.

## 21. Tracing

An agentic workflow may look like:

```
User Request
    |
    v
Planner Agent
    |
    +--> Search Tool
    |
    +--> Research Agent
    |       |
    |       +--> LLM
    |
    +--> Summarizer
            |
            +--> LLM
```

A trace connects all operations.

Example:

```
Trace: trace_123

Span 1: planner
  └── LLM call
      └── 900 ms

Span 2: research
  ├── search
  └── LLM call
      └── 1.4 s

Span 3: summarizer
  └── LLM call
      └── 800 ms
```

This makes debugging agentic systems dramatically easier.

## 22. Metrics

Important metrics include:

### Reliability

- success_rate
- error_rate
- fallback_rate
- retry_rate

### Performance

- p50 latency
- p95 latency
- p99 latency
- time_to_first_token
- tokens_per_second

### Cost

- input_tokens
- output_tokens
- total_tokens
- cost_per_request
- cost_per_agent
- cost_per_user

### Model quality

Depending on your evaluation infrastructure:

- task_success_rate
- tool_success_rate
- structured_output_validity
- evaluation_score

## 23. Streaming

LLMs often stream output.

Instead of:

```
Request
   |
   | wait 10 seconds
   |
   v
Complete response
```

streaming produces:

```
Hello
Hello, how
Hello, how are
Hello, how are you...
```

A gateway must preserve the stream.

Architecture:

```
Application
     |
     v
Gateway
     |
     v
Provider
     |
     v
Token stream
     |
     v
Gateway
     |
     v
Application
```

### Streaming Challenges

The gateway must handle:

- connection termination
- partial responses
- provider stream errors
- timeout
- cancellation
- usage accounting
- retries

Retrying after partial streaming is difficult.

For example:

```
Provider
 ↓
"The answer is..."
 ↓
"Paris because..."
 ↓
NETWORK FAILURE
```

The gateway cannot blindly replay the request because the user may already have received part of the response.

## 24. Structured Outputs

Agents frequently need structured responses.

Instead of:

```
The customer appears interested in the product.
```

you may require:

```json
{
  "intent": "purchase",
  "confidence": 0.91
}
```

The gateway can standardize:

- JSON Schema
- Structured output
- Validation
- Retry on malformed output

Example:

```
LLM
 ↓
JSON
 ↓
Schema validator
 ↓
Valid?
 ├── Yes → return
 └── No → retry / repair
```

## 25. Tool Calling

Modern agents often use tools.

Example:

```
LLM
 |
 +--> search()
 |
 +--> calculator()
 |
 +--> database()
 |
 +--> email()
```

The gateway needs to preserve tool-call structures.

A request might contain:

```json
{
  "messages": [...],
  "tools": [
    {
      "name": "search",
      "description": "Search the web"
    }
  ]
}
```

The provider generates:

```json
{
  "tool_call": {
    "name": "search",
    "arguments": {
      "query": "LLM gateways"
    }
  }
}
```

The gateway must not accidentally destroy provider-specific tool semantics.

## 26. Embeddings and Other Models

LLM gateways don't necessarily have to handle only text generation.

A broader AI gateway can route:

- Text generation
- Embeddings
- Reranking
- Image generation
- Speech-to-text
- Text-to-speech
- Vision
- Moderation

Example:

```
AI Gateway
 |
 +--> Chat models
 |
 +--> Embedding models
 |
 +--> Rerankers
 |
 +--> Vision models
 |
 +--> Audio models
```

## 27. Security

The gateway is a natural security boundary.

It can enforce:

- Authentication
- Authorization
- API key management
- Tenant isolation
- Request validation
- Input filtering
- Output filtering
- PII protection
- Audit logging

### API Key Protection

Applications should generally not expose provider keys directly to users.

**Bad architecture:**

```
Browser
 |
 +--> Provider API
       API KEY
```

**Better:**

```
Browser
 |
 v
Backend
 |
 v
LLM Gateway
 |
 v
Provider
```

Provider credentials remain server-side.

## 28. Prompt and Data Privacy

A gateway may see:

- system prompt
- user prompt
- documents
- tool results
- LLM output

Therefore it becomes a sensitive infrastructure component.

You should decide:

- Should prompts be logged?
- How long are logs retained?
- Who can access them?
- Are they encrypted?
- Can providers retain submitted data?
- Can sensitive data be sent to external models?

### Data Residency

Some systems require:

```
EU data → EU infrastructure
US data → US infrastructure
```

The gateway can route accordingly:

```
Region = EU
     ↓
EU-approved provider
```

## 29. Guardrails

A gateway can enforce safety and policy rules.

Example:

```
Request
 ↓
Input guardrail
 ↓
LLM
 ↓
Output guardrail
 ↓
Application
```

**Input guardrails** can detect:

- secrets
- malicious instructions
- restricted content
- prompt injection indicators

**Output guardrails** can validate:

- schema
- sensitive information
- prohibited output
- policy requirements

## 30. Model Selection for Agents

Agentic systems should not necessarily use the same model for everything.

Consider:

```
Agent workflow
      |
      +--> Classification
      |
      +--> Planning
      |
      +--> Tool selection
      |
      +--> Coding
      |
      +--> Summarization
```

Different tasks may have different requirements.

For example:

```
Classification
    ↓
small/cheap model

Planning
    ↓
reasoning model

Summarization
    ↓
fast model

Coding
    ↓
coding-capable model
```

This is one of the strongest reasons to introduce a gateway.

## 31. LLM Gateways in Agentic AI

Agentic AI systems can generate many LLM calls.

A single user request may result in:

```
User
 |
 v
Agent
 |
 +--> Planning LLM
 |
 +--> Tool selection LLM
 |
 +--> Search
 |
 +--> Research LLM
 |
 +--> Reflection LLM
 |
 +--> Final answer LLM
```

Potentially:

```
1 user request
      ↓
10 LLM calls
```

or even:

```
1 user request
      ↓
100+ LLM calls
```

Without centralized control, cost and reliability become difficult to manage.

### Gateway as Agent Infrastructure

```
                    Agent Runtime
                         |
                         v
                  ┌─────────────┐
                  │ LLM Gateway │
                  └──────┬──────┘
                         |
       ┌─────────────────┼─────────────────┐
       │                 │                 │
       v                 v                 v
    Fast Model      Reasoning Model    Coding Model
```

The agent decides:

> what it needs

The gateway decides:

> how and where inference happens

This separation is important.

## 32. Multi-Agent Systems

Consider:

```
Supervisor Agent
       |
       +------ Research Agent
       |
       +------ Coding Agent
       |
       +------ Data Agent
       |
       +------ Reviewer Agent
```

Each agent can use a different logical model:

```
Supervisor → reasoning
Research → long_context
Coding → coding
Reviewer → reasoning
```

The gateway maps those logical names to actual providers.

This allows the agent architecture to remain stable even when model infrastructure changes.

## 33. Gateway Request Lifecycle

A production request may follow this pipeline:

```
                 Incoming Request
                        |
                        v
                Authentication
                        |
                        v
                 Authorization
                        |
                        v
                  Validation
                        |
                        v
                 Rate Limiting
                        |
                        v
                Budget Check
                        |
                        v
                 Cache Lookup
                        |
                  ┌─────┴─────┐
                  │           │
                 HIT         MISS
                  │           │
                  v           v
               Return      Routing
                              |
                              v
                       Model Selection
                              |
                              v
                        Provider Call
                              |
                     ┌────────┴────────┐
                     │                 │
                   Success            Error
                     │                 │
                     v                 v
                  Response          Retry?
                                       |
                                  ┌────┴────┐
                                  │         │
                                 Yes        No
                                  │         │
                                  v         v
                                Retry     Fallback
                                             |
                                             v
                                         Provider 2
                                             |
                                             v
                                           Result


                         Result Processing
                              |
                              v
                        Validation
                              |
                              v
                      Observability
                              |
                              v
                          Response
```

## 34. Example Architecture

A practical architecture:

```
                       ┌───────────────┐
                       │   Frontend    │
                       └───────┬───────┘
                               │
                               v
                       ┌───────────────┐
                       │   Backend     │
                       └───────┬───────┘
                               │
                               v
                    ┌─────────────────────┐
                    │     Agent Runtime   │
                    └──────────┬──────────┘
                               │
                               v
                    ┌─────────────────────┐
                    │     LLM Gateway     │
                    │                     │
                    │ Auth                │
                    │ Routing             │
                    │ Retry               │
                    │ Fallback            │
                    │ Rate Limit          │
                    │ Cache               │
                    │ Cost                │
                    │ Observability        │
                    └──────────┬──────────┘
                               │
              ┌────────────────┼────────────────┐
              │                │                │
              v                v                v
          Provider A       Provider B       Provider C
              │                │                │
              v                v                v
            Models           Models           Models
```

## 35. Designing a Gateway

When designing your own gateway, separate responsibilities.

A useful architecture is:

```
gateway/
│
├── auth/
│
├── routing/
│   ├── router
│   ├── policies
│   └── model_registry
│
├── providers/
│   ├── openai
│   ├── anthropic
│   └── google
│
├── reliability/
│   ├── retry
│   ├── timeout
│   ├── fallback
│   └── circuit_breaker
│
├── cache/
│
├── rate_limit/
│
├── cost/
│
├── observability/
│   ├── logging
│   ├── metrics
│   └── tracing
│
└── api/
```

This separation prevents gateway logic from becoming one huge class.

## 36. Minimal Gateway Implementation

A simple abstraction:

```python
class LLMGateway:

    def __init__(self, providers, router):
        self.providers = providers
        self.router = router

    async def generate(self, request):

        model = self.router.select(request)

        provider = self.providers[model.provider]

        return await provider.generate(
            model=model.name,
            messages=request.messages
        )
```

The application now uses:

```python
response = await gateway.generate(request)
```

instead of:

```python
openai_client.responses.create(...)
```

## 37. Production Gateway

A more realistic gateway:

```python
async def generate(request):

    authenticate(request)

    authorize(request)

    validate(request)

    await rate_limiter.check(request)

    await budget_manager.check(request)

    cached = await cache.get(request)

    if cached:
        return cached

    route = router.select(request)

    for attempt in range(route.max_attempts):

        try:

            response = await provider.call(
                route.provider,
                route.model,
                request
            )

            validate_response(response)

            await cache.set(request, response)

            await record_usage(
                request,
                response
            )

            return response

        except RetryableError:

            await backoff(attempt)

        except NonRetryableError:

            break

    return await fallback(request, route)
```

The exact implementation will depend on your framework and infrastructure, but the conceptual pipeline is important.

## 38. Failure Scenarios

A production gateway must assume failures.

### Scenario 1: Provider Timeout

```
Request
 ↓
Provider A
 ↓
Timeout
 ↓
Provider B
 ↓
Success
```

### Scenario 2: Rate Limit

```
Provider A
 ↓
429
 ↓
Retry/backoff
 ↓
Still 429
 ↓
Fallback Provider B
```

### Scenario 3: Invalid Request

```
Provider A
 ↓
400
 ↓
Do NOT retry blindly
 ↓
Return error
```

### Scenario 4: Provider Completely Down

```
Provider A
 ↓
5xx
 ↓
Circuit breaker opens
 ↓
Traffic routed elsewhere
```

## 39. Circuit Breakers

A circuit breaker prevents repeatedly sending requests to an unhealthy provider.

**Without one:**

```
Request
 ↓
Provider A
 ↓
Failure

Request
 ↓
Provider A
 ↓
Failure

Request
 ↓
Provider A
 ↓
Failure
```

**With a circuit breaker:**

```
Provider A
 ↓
Failures increase
 ↓
Threshold reached
 ↓
Circuit OPEN
 ↓
Stop sending traffic
 ↓
Fallback provider

After some time:

Circuit HALF-OPEN
       |
       v
Test request
       |
   ┌───┴───┐
   │       │
success  failure
   │       │
   v       v
CLOSED   OPEN
```

This protects both the gateway and provider.

## 40. Gateway vs Direct Provider Calls

### Direct

```
Agent
 |
 v
Provider
```

**Advantages:**

- simple
- low infrastructure overhead
- easy for prototypes

**Disadvantages:**

- provider lock-in
- duplicated logic
- difficult observability
- difficult fallback
- difficult cost control
- difficult multi-provider routing

### Gateway

```
Agent
 |
 v
Gateway
 |
 +--> Provider A
 +--> Provider B
 +--> Provider C
```

**Advantages:**

- centralized control
- provider abstraction
- routing
- fallbacks
- observability
- cost management
- security
- easier model migration

**Disadvantages:**

- additional infrastructure
- additional latency
- additional operational complexity
- gateway becomes a critical dependency

## 41. Open-Source LLM Gateway Tools

Several tools/projects exist in this space.

Common examples include:

- **LiteLLM**
- **Kong AI Gateway**
- **Portkey**
- **Envoy-based AI gateway architectures**
- **Cloud/provider-specific AI gateways**

### LiteLLM

LiteLLM provides a unified interface across many LLM providers.

Conceptually:

```python
completion(
    model="provider/model",
    messages=messages
)
```

It is commonly used when applications need a unified interface across providers.

### Kong AI Gateway

Kong extends API gateway concepts toward AI/LLM workloads.

Useful concepts include:

- routing
- authentication
- rate limiting
- observability
- LLM traffic management

### Portkey

Portkey focuses heavily on AI gateway capabilities such as:

- routing
- fallbacks
- observability
- guardrails
- caching

The exact feature set and integrations of these projects change over time, so always consult their current documentation before choosing one.

## 42. Learning Roadmap

If you want to properly understand LLM Gateways, learn in this order.

### Level 1 — LLM Fundamentals

Understand:

- Tokens
- Context windows
- Temperature
- Top-p
- Inference
- Streaming
- Tool calling
- Structured output
- Embeddings

### Level 2 — API Fundamentals

Learn:

- HTTP
- REST
- JSON
- authentication
- API keys
- timeouts
- HTTP status codes
- streaming
- SSE

### Level 3 — Distributed Systems

Learn:

- Retries
- Exponential backoff
- Rate limiting
- Load balancing
- Circuit breakers
- Queues
- Caching
- Concurrency

### Level 4 — LLM Infrastructure

Learn:

- Model routing
- Provider abstraction
- Fallbacks
- Cost tracking
- Token accounting
- LLM observability
- Prompt caching
- Semantic caching

### Level 5 — Agent Infrastructure

Learn:

- Agent loops
- Tool calling
- Multi-agent systems
- Planning
- Reflection
- Memory
- Workflow orchestration
- Agent tracing

### Level 6 — Production

Learn:

- Kubernetes
- Redis
- PostgreSQL
- OpenTelemetry
- Prometheus
- Grafana
- Secrets management
- Horizontal scaling
- Multi-tenancy
- Disaster recovery

## 43. Interview Questions

### Beginner

**What is an LLM Gateway?**

An infrastructure layer between an application and one or more LLM providers that centralizes routing, reliability, security, observability, and cost management.

**Why use an LLM Gateway?**

To avoid tightly coupling an application to a single provider and to centralize capabilities such as:

- routing
- fallbacks
- retries
- rate limits
- caching
- observability
- cost management

**What is model routing?**

Selecting an appropriate model/provider for a request based on criteria such as:

- capability
- cost
- latency
- availability
- context requirements

### Intermediate

**What is fallback?**

Using another model/provider when the preferred provider cannot successfully process a request.

**Why is exponential backoff used?**

To prevent repeated retries from overwhelming an already overloaded provider and to reduce synchronized retry storms.

**What is a circuit breaker?**

A reliability mechanism that temporarily stops sending requests to an unhealthy provider after repeated failures.

**Why is token-based rate limiting useful?**

Because two requests can consume drastically different amounts of compute even though both count as one request.

### Advanced

**How would you design multi-provider routing?**

Maintain a model registry containing:

- provider
- model
- capabilities
- context window
- pricing
- latency
- availability
- limits

Then combine request requirements with routing policies.

**How would you handle provider outages?**

Use:

- timeouts
- retries
- exponential backoff
- circuit breakers
- fallback models
- health monitoring

**How would you minimize LLM cost?**

Use:

- model routing
- caching
- prompt optimization
- token limits
- budget enforcement
- smaller models for simple tasks
- batching where appropriate

**How would you debug an agent making 20 LLM calls?**

Use distributed tracing.

Each workflow gets:

- trace_id

Each operation gets:

- span_id

Then inspect:

- latency
- tokens
- model
- provider
- errors
- tool calls
- retries
- fallbacks

## 44. Key Takeaways

An LLM Gateway is much more than a proxy.

At the simplest level:

```
Application
    ↓
Gateway
    ↓
LLM
```

But a production gateway becomes:

```
                    ┌────────────────────┐
                    │    LLM Gateway     │
                    ├────────────────────┤
                    │ Authentication     │
                    │ Authorization      │
                    │ Validation         │
                    │ Rate Limiting      │
                    │ Budget Control     │
                    │ Model Routing      │
                    │ Provider Routing   │
                    │ Load Balancing     │
                    │ Retries            │
                    │ Timeouts           │
                    │ Fallbacks          │
                    │ Circuit Breakers   │
                    │ Caching            │
                    │ Cost Tracking      │
                    │ Guardrails         │
                    │ Logging            │
                    │ Metrics             │
                    │ Tracing            │
                    │ Streaming          │
                    └─────────┬──────────┘
                              │
             ┌────────────────┼────────────────┐
             │                │                │
             v                v                v
          Provider A       Provider B       Provider C
```

For Agentic AI, the gateway becomes particularly valuable because one user request can trigger many model calls.

The core separation should be:

```
Agent
  │
  │ decides WHAT needs to happen
  │
  v
LLM Gateway
  │
  │ decides HOW/WHERE inference happens
  │
  v
Model Provider
```

This separation allows your agent architecture to remain stable while the underlying model infrastructure evolves.

### Mental Model

Remember this:

> An LLM Gateway is the control plane between your AI application and model infrastructure.

The application says:

> "I need a reasoning-capable model."

The gateway decides:

- Which provider?
- Which model?
- Which region?
- Is the user allowed?
- Is the budget sufficient?
- Is the provider healthy?
- Should we use cache?
- Should we retry?
- Should we fallback?
- How much did it cost?
- How long did it take?

The agent should focus on:

- reasoning
- planning
- tool usage
- state
- memory
- task execution

The gateway should focus on:

- inference infrastructure
- routing
- reliability
- governance
- cost
- security
- observability

That separation is one of the most important architectural ideas when building production-grade Agentic AI systems.
