# LangGraph — Complete Notes

> **LangGraph is a low-level orchestration framework and runtime for building long-running, stateful AI agents and workflows.**

LangGraph allows us to explicitly model an AI application as a **graph of computation** where:

* **State** stores information about the current execution.
* **Nodes** perform work and update the state.
* **Edges** determine what happens next.
* **Conditional edges** allow dynamic routing.
* **Loops** allow iterative agent behavior.
* **Persistence** allows state to survive across executions.
* **Interrupts** allow human-in-the-loop workflows.
* **Streaming** exposes intermediate execution results.
* **Subgraphs** allow modular and multi-agent architectures.
* **Time travel** allows replaying and forking previous executions.

LangGraph can be used with LangChain, but it does **not require LangChain**. LangChain provides higher-level agent abstractions, while LangGraph provides lower-level orchestration and runtime capabilities.

---

# 1. Why LangGraph?

Traditional LLM applications often look like:

```text
User
  ↓
Prompt
  ↓
LLM
  ↓
Response
```

This works for simple applications.

But real AI agents often require:

```text
User
  ↓
Understand request
  ↓
Plan
  ↓
Search data
  ↓
Call tools
  ↓
Evaluate result
  ↓
Retry if necessary
  ↓
Ask human if necessary
  ↓
Continue
  ↓
Generate final response
```

The problem becomes harder when the application needs:

* multiple steps
* branching
* loops
* state
* tool calls
* retries
* human approval
* memory
* persistence
* parallel execution
* long-running tasks
* fault recovery
* debugging
* multiple agents

LangGraph provides primitives for modeling these systems explicitly.

---

# 2. LangGraph Mental Model

The most important mental model is:

```text
                ┌──────────────┐
                │    START     │
                └──────┬───────┘
                       │
                       ▼
                ┌──────────────┐
                │     Node     │
                └──────┬───────┘
                       │
                       ▼
                ┌──────────────┐
                │    State     │
                └──────┬───────┘
                       │
                       ▼
              ┌──────────────────┐
              │ Conditional Edge │
              └───────┬──────────┘
                      │
              ┌───────┴────────┐
              ▼                ▼
        ┌───────────┐    ┌───────────┐
        │  Node A   │    │  Node B   │
        └─────┬─────┘    └─────┬─────┘
              │                │
              └───────┬────────┘
                      ▼
                ┌──────────────┐
                │     END      │
                └──────────────┘
```

At its core:

```text
Graph
 ├── State
 ├── Nodes
 └── Edges
```

LangGraph's Graph API explicitly models workflows around these three concepts.

---

# 3. LangGraph vs LangChain

## LangChain

LangChain provides higher-level abstractions for:

* models
* tools
* prompts
* agents
* integrations
* agent loops

It is useful when you want to build an agent quickly.

## LangGraph

LangGraph focuses on:

* orchestration
* state
* control flow
* persistence
* durable execution
* human-in-the-loop
* streaming
* complex workflows

### Simple comparison

```text
LangChain
    │
    │ High-level
    ▼
Agent / Tool / Model abstractions
    │
    ▼
LangGraph
    │
    │ Low-level orchestration
    ▼
State + Nodes + Edges + Runtime
```

LangChain's current agents are built on top of LangGraph, which allows them to inherit capabilities such as persistence and durable execution.

### When should I use LangGraph?

Use LangGraph when you need:

* deterministic + agentic workflows
* complex branching
* loops
* multiple agents
* long-running execution
* state persistence
* human approval
* fault tolerance
* fine-grained execution control

If a simple prebuilt agent is enough, a higher-level LangChain agent may be preferable.

---

# 4. Installation

Install LangGraph:

```bash
pip install -U langgraph
```

If using LangChain:

```bash
pip install -U langgraph langchain
```

If using Groq:

```bash
pip install -U langgraph langchain langchain-groq
```

Verify:

```bash
python -c "import langgraph; print('LangGraph installed')"
```

Do not rely on:

```python
langgraph.__version__
```

because the package does not necessarily expose a `__version__` attribute.

---

# 5. First LangGraph Program

The smallest useful graph looks like this:

```python
from typing_extensions import TypedDict

from langgraph.graph import StateGraph, START, END


class State(TypedDict):
    message: str


def hello(state: State):
    return {
        "message": "Hello LangGraph!"
    }


builder = StateGraph(State)

builder.add_node("hello", hello)

builder.add_edge(START, "hello")
builder.add_edge("hello", END)

graph = builder.compile()

result = graph.invoke({
    "message": ""
})

print(result)
```

Conceptually:

```text
START
  │
  ▼
hello
  │
  ▼
 END
```

The standard Graph API uses `StateGraph`, nodes, edges, `START`, `END`, and `compile()`.

---

# 6. The Three Core Concepts

## 6.1 State

State represents the current snapshot of the application.

Example:

```python
class State(TypedDict):
    user_query: str
    answer: str
```

The state can contain:

```text
user input
messages
tool results
intermediate results
decisions
metadata
execution flags
agent outputs
```

---

## 6.2 Nodes

A node is a Python function that:

1. receives state
2. performs some work
3. returns a state update

Example:

```python
def process(state: State):
    result = state["user_query"].upper()

    return {
        "answer": result
    }
```

Nodes can perform:

* LLM calls
* database queries
* API calls
* tool calls
* calculations
* validation
* routing preparation
* human interaction

---

## 6.3 Edges

Edges determine what executes next.

Fixed edge:

```python
builder.add_edge("node_a", "node_b")
```

Start edge:

```python
builder.add_edge(START, "node_a")
```

End edge:

```python
builder.add_edge("node_a", END)
```

---

# 7. START and END

`START` represents the beginning of graph execution.

```python
builder.add_edge(START, "first_node")
```

`END` represents the terminal point.

```python
builder.add_edge("last_node", END)
```

Think:

```text
START
  ↓
Node A
  ↓
Node B
  ↓
END
```

---

# 8. State Schemas

The state schema defines what information travels through the graph.

## TypedDict

Most basic approach:

```python
from typing_extensions import TypedDict


class State(TypedDict):
    question: str
    answer: str
```

---

## Optional fields

```python
from typing import Optional
from typing_extensions import TypedDict


class State(TypedDict):
    question: str
    answer: Optional[str]
```

---

## Multiple data types

```python
class State(TypedDict):
    question: str
    documents: list[str]
    score: float
    approved: bool
```

---

# 9. State Updates

A node usually returns only the fields it wants to update.

```python
def retrieve(state: State):
    docs = search(state["question"])

    return {
        "documents": docs
    }
```

It does not need to return the entire state.

Conceptually:

```text
Current State
     │
     ▼
Node
     │
     ▼
Partial Update
     │
     ▼
New State
```

---

# 10. Reducers

Reducers determine how multiple updates to the same state field are combined.

This becomes especially important with:

* parallel execution
* multiple nodes
* tool calls
* fan-out/fan-in workflows

Example:

```python
from typing import Annotated
from operator import add
from typing_extensions import TypedDict


class State(TypedDict):
    results: Annotated[list[str], add]
```

Now:

```text
Node A → ["A"]
Node B → ["B"]
Node C → ["C"]
```

can be combined into:

```text
["A", "B", "C"]
```

without simply overwriting the previous value.

### Important distinction

Without reducer:

```text
old value
   ↓
new value
```

With reducer:

```text
old value
   +
new value
   ↓
combined value
```

Reducers are especially important when multiple branches write to the same state key.

---

# 11. MessagesState

For conversational agents, LangGraph provides:

```python
from langgraph.graph import MessagesState
```

Example:

```python
class State(MessagesState):
    user_id: str
```

This is convenient when your application revolves around messages.

Example:

```python
from langgraph.graph import StateGraph, MessagesState, START, END


def chatbot(state: MessagesState):
    return {
        "messages": [
            {
                "role": "assistant",
                "content": "Hello!"
            }
        ]
    }


builder = StateGraph(MessagesState)

builder.add_node("chatbot", chatbot)

builder.add_edge(START, "chatbot")
builder.add_edge("chatbot", END)

graph = builder.compile()
```

---

# 12. Fixed Workflows

A deterministic workflow:

```text
START
  ↓
Input
  ↓
Retrieve
  ↓
Process
  ↓
Generate
  ↓
END
```

Code:

```python
builder.add_edge(START, "retrieve")
builder.add_edge("retrieve", "process")
builder.add_edge("process", "generate")
builder.add_edge("generate", END)
```

This is useful for:

* document pipelines
* ETL
* RAG
* data processing
* deterministic business logic

---

# 13. Conditional Edges

Conditional edges allow the graph to make decisions.

Example:

```python
def route(state: State):
    if state["score"] >= 0.8:
        return "good"

    return "bad"
```

Then:

```python
builder.add_conditional_edges(
    "evaluate",
    route,
    {
        "good": "finalize",
        "bad": "retry"
    }
)
```

Graph:

```text
             ┌──────────────┐
             │   Evaluate   │
             └──────┬───────┘
                    │
              ┌─────┴─────┐
              ▼           ▼
            Good         Bad
              │           │
              ▼           ▼
          Finalize       Retry
```

This is one of the most important patterns for agentic workflows.

---

# 14. Loops

Agents often need iterative behavior.

Example:

```text
Generate
   ↓
Evaluate
   ↓
Good? ───── Yes ───→ END
   │
   No
   │
   ▼
Improve
   │
   └────────────→ Generate
```

A conditional edge can create the loop:

```python
builder.add_conditional_edges(
    "evaluate",
    route,
    {
        "retry": "generate",
        "done": END
    }
)
```

This enables:

* reflection
* self-correction
* validation
* retry loops
* iterative research
* planning/execution cycles

---

# 15. Example: Self-Correcting Agent

```python
from typing_extensions import TypedDict

from langgraph.graph import StateGraph, START, END


class State(TypedDict):
    question: str
    answer: str
    score: int


def generate(state: State):
    answer = f"Answer for: {state['question']}"

    return {
        "answer": answer
    }


def evaluate(state: State):
    score = len(state["answer"])

    return {
        "score": score
    }


def route(state: State):
    if state["score"] > 20:
        return "done"

    return "retry"


builder = StateGraph(State)

builder.add_node("generate", generate)
builder.add_node("evaluate", evaluate)

builder.add_edge(START, "generate")
builder.add_edge("generate", "evaluate")

builder.add_conditional_edges(
    "evaluate",
    route,
    {
        "retry": "generate",
        "done": END
    }
)

graph = builder.compile()
```

This is the basic architecture behind many reflection-style agents.

---

# 16. Node Functions

A node can be:

```python
def my_node(state):
    ...
```

It can also be async:

```python
async def my_node(state):
    ...
```

The node should generally:

```text
Read State
    ↓
Perform Work
    ↓
Return Update
```

Avoid unnecessarily mutating the state object directly.

Prefer:

```python
return {
    "answer": result
}
```

over:

```python
state["answer"] = result
return state
```

---

# 17. Nodes Calling LLMs

LangGraph itself is not an LLM provider.

You can use:

* OpenAI
* Anthropic
* Groq
* Google
* local models
* other LangChain-compatible models

Example with Groq:

```python
from langchain_groq import ChatGroq


llm = ChatGroq(
    model="llama-3.3-70b-versatile",
    temperature=0
)
```

Then:

```python
def call_model(state: State):

    response = llm.invoke(
        state["question"]
    )

    return {
        "answer": response.content
    }
```

LangGraph handles the orchestration; the model handles generation/reasoning.

---

# 18. LangGraph + Tools

An agent often follows:

```text
User
 ↓
LLM
 ↓
Tool?
 ├── No → Final response
 │
 └── Yes
      ↓
   Execute Tool
      ↓
      LLM
      ↓
   Tool?
      ...
```

This loop is a natural fit for LangGraph.

A simplified architecture:

```text
          ┌─────────────┐
          │    Agent    │
          └──────┬──────┘
                 │
          Tool required?
           /           \
         No             Yes
         │               │
         ▼               ▼
       END          Tool Executor
                         │
                         ▼
                       Agent
```

---

# 19. Command

`Command` allows a node to combine:

1. a state update
2. a routing decision

Example:

```python
from langgraph.types import Command


def router(state: State):

    if state["score"] > 0.8:
        return Command(
            update={"approved": True},
            goto="finalize"
        )

    return Command(
        update={"approved": False},
        goto="retry"
    )
```

This is useful when routing depends on logic inside a node and you want to update state at the same time.

The Graph API documents `Command` as a control mechanism for combining state updates with transitions.

---

# 20. Command vs Conditional Edges

## Conditional Edge

Use when routing logic is conceptually separate:

```text
Node
 ↓
Router
 ↓
A / B / C
```

## Command

Use when the node itself decides:

```text
Node
 ├── Update State
 └── Decide Next Node
```

### Rule of thumb

Use:

```python
add_conditional_edges()
```

when routing is a separate concern.

Use:

```python
Command(...)
```

when the node naturally owns both the update and transition.

---

# 21. Send API

`Send` is useful for dynamic fan-out and map-reduce workflows.

Suppose we have:

```text
Documents
    ↓
Split documents
    ↓
Process each document independently
    ↓
Combine results
```

Conceptually:

```text
                 ┌── Document 1 ──┐
                 │                │
Input ───────────┼── Document 2 ──┼──→ Aggregate
                 │                │
                 └── Document 3 ──┘
```

`Send` can dynamically create work for multiple items.

This is useful for:

* map-reduce
* parallel document processing
* multi-agent fan-out
* batch processing
* dynamic task generation

The Graph API specifically supports `Send` for map-reduce-style workflows.

---

# 22. Runtime

LangGraph applications can receive runtime information through `Runtime`.

Runtime can provide:

* context
* persistent store
* stream writer
* execution information
* server information

Runtime context is useful for dependency injection.

Example:

```python
from dataclasses import dataclass

from langgraph.runtime import Runtime


@dataclass
class Context:
    user_id: str


def node(state: State, runtime: Runtime[Context]):

    user_id = runtime.context.user_id

    return {
        "user_id": user_id
    }
```

Invoke:

```python
graph.invoke(
    {"question": "Hello"},
    context=Context(user_id="user-123")
)
```

This is preferable to hardcoding dependencies or relying heavily on global variables.

---

# 23. State vs Context vs Store

This distinction is extremely important.

## State

Mutable information during the current execution.

Examples:

```text
messages
current question
intermediate results
tool outputs
scores
decisions
```

## Runtime Context

Static information provided for one invocation.

Examples:

```text
user_id
database connection
configuration
tenant information
API clients
```

## Store

Persistent information that can survive across conversations.

Examples:

```text
user preferences
long-term memories
profile information
application knowledge
```

Conceptually:

```text
                 LangGraph
                     │
        ┌────────────┼─────────────┐
        ▼            ▼             ▼
      State        Context        Store
        │            │             │
   Current run    Run-level     Cross-run
    mutable       metadata      persistent
```

LangChain's context documentation distinguishes these based on mutability and lifetime.

---

# 24. Persistence

One of LangGraph's most important capabilities is persistence.

A checkpointer can save graph state at execution steps.

This enables:

* memory
* human-in-the-loop
* fault tolerance
* replay
* time travel
* recovery

The persisted execution is organized around **threads**.

---

# 25. Checkpointers

A graph can be compiled with a checkpointer.

Example:

```python
from langgraph.checkpoint.memory import InMemorySaver


checkpointer = InMemorySaver()

graph = builder.compile(
    checkpointer=checkpointer
)
```

For development, in-memory persistence is convenient.

For production, use a durable persistence backend appropriate to your deployment.

---

# 26. Threads

A thread identifies a persistent execution context.

Example:

```python
config = {
    "configurable": {
        "thread_id": "user-123"
    }
}
```

Then:

```python
graph.invoke(
    {"question": "Hello"},
    config=config
)
```

Using the same thread allows the graph to access the persisted state associated with that thread.

Using a new thread creates a separate execution history.

The `thread_id` is essential for checkpoint-based resume behavior.

---

# 27. Short-Term Memory

A conversational agent can use checkpointed state as short-term memory.

Example:

```text
Thread: user-123

Message 1
   ↓
Checkpoint

Message 2
   ↓
Checkpoint

Message 3
   ↓
Checkpoint
```

The conversation can continue using the same thread.

---

# 28. Long-Term Memory

Long-term memory is different.

Example:

```text
User
 │
 ├── Preferences
 ├── Profile
 ├── Past decisions
 └── Important facts
```

A persistent store can be used for information shared across conversations.

This is different from simply storing messages in the current graph state.

---

# 29. Durable Execution

Durable execution means a long-running workflow can persist its progress and recover instead of necessarily starting from scratch after a failure.

Example:

```text
Task A
 ↓
Task B
 ↓
Task C
 ↓
Task D
```

Suppose Task C fails.

Without persistence:

```text
Restart
 ↓
A
 ↓
B
 ↓
C
```

With appropriate checkpointing:

```text
Checkpoint after B
       ↓
Failure at C
       ↓
Resume
       ↓
C
 ↓
D
```

This is particularly useful for:

* long-running agents
* API workflows
* approval processes
* research agents
* enterprise automation

Durable execution is one of LangGraph's central capabilities.

---

# 30. Human-in-the-Loop

AI agents should not always be allowed to act autonomously.

Example:

```text
Agent
 ↓
Generate action
 ↓
Risky operation?
 ↓
Human approval
 ├── Approve → Execute
 └── Reject  → Modify
```

LangGraph supports this using interrupts and persistence.

---

# 31. Interrupts

Import:

```python
from langgraph.types import interrupt
```

Example:

```python
def approval_node(state: State):

    approved = interrupt(
        "Do you approve this action?"
    )

    return {
        "approved": approved
    }
```

Execution pauses until external input is provided.

The graph state is persisted so execution can resume later.

---

# 32. Resuming an Interrupt

Use `Command`.

```python
from langgraph.types import Command


graph.invoke(
    Command(resume=True),
    config=config
)
```

Conceptually:

```text
Node
 ↓
interrupt()
 ↓
SAVE STATE
 ↓
WAIT
 ↓
Human decision
 ↓
Command(resume=...)
 ↓
Continue
```

---

# 33. Important Interrupt Rule

Do not write:

```python
try:
    answer = interrupt(...)
except:
    ...
```

Interrupts work through the runtime's control-flow mechanism.

Also remember:

> When execution resumes, the node containing the interrupt may restart from the beginning.

Therefore, side effects performed before `interrupt()` should be designed to be safe to repeat or made idempotent.

---

# 34. Human Approval Pattern

A common production pattern:

```text
             Agent
               │
               ▼
        Generate Action
               │
               ▼
         Risk Evaluation
               │
        ┌──────┴──────┐
        │             │
       Safe          Risky
        │             │
        ▼             ▼
      Execute      interrupt()
                      │
                      ▼
                    Human
                   /     \
              Approve   Reject
                 │         │
                 ▼         ▼
              Execute    Revise
```

Useful for:

* sending emails
* deleting data
* financial operations
* publishing content
* database mutations
* security actions
* production deployments

---

# 35. Streaming

LangGraph supports streaming intermediate execution information.

Instead of waiting:

```text
START → Node A → Node B → Node C → Final
```

you can expose:

```text
Node A completed
Node B started
LLM tokens arriving
Node C completed
Final result
```

Streaming is useful for:

* chat interfaces
* agent dashboards
* progress indicators
* debugging
* real-time UI

LangGraph provides multiple streaming projections for state, messages, subgraphs, and other execution information.

---

# 36. Basic Streaming

Example:

```python
for chunk in graph.stream(
    {"question": "Explain RAG"},
    stream_mode="updates"
):
    print(chunk)
```

Possible conceptual output:

```text
{"retrieve": {...}}

{"generate": {...}}

{"evaluate": {...}}
```

---

# 37. Streaming Modes

Depending on the version/API being used, LangGraph supports different projections such as:

```text
updates
values
messages
custom
debug
tasks
checkpoints
subgraphs
```

The exact available modes and APIs should be checked against the version of LangGraph installed in the project.

---

# 38. Subgraphs

Large agent systems should not become one enormous graph.

Instead:

```text
Main Graph
    │
    ├── Research Subgraph
    │
    ├── Analysis Subgraph
    │
    ├── Writing Subgraph
    │
    └── Review Subgraph
```

A subgraph is essentially a graph embedded inside another graph.

Useful for:

* multi-agent systems
* modular workflows
* reusable components
* domain-specific agents
* team-based architectures

LangGraph supports stateful and stateless subgraph patterns.

---

# 39. Multi-Agent Architecture

A multi-agent system can be represented as:

```text
                    Supervisor
                        │
           ┌────────────┼────────────┐
           ▼            ▼            ▼
       Researcher     Coder       Reviewer
           │            │            │
           └────────────┼────────────┘
                        ▼
                     Final
```

Possible agents:

```text
Planner
Researcher
Retriever
Coder
Critic
Reviewer
Executor
Supervisor
```

LangGraph provides the orchestration layer connecting these agents.

---

# 40. Per-Invocation vs Per-Thread Subgraphs

### Per-invocation

Each subgraph call starts independently.

Useful when:

```text
Parent
 ├── Call researcher
 ├── Call researcher
 └── Call researcher
```

and each call does not need memory from previous calls.

### Per-thread

The subgraph retains state across calls.

Useful for:

```text
Research Agent
 ↓
Conversation 1
 ↓
Conversation 2
 ↓
Conversation 3
```

where the same subagent needs persistent conversation state.

LangGraph recommends per-invocation persistence for many independent subagent calls and per-thread persistence when a subagent needs multi-turn memory.

---

# 41. Time Travel

Persistence enables time travel.

Two major operations are:

```text
Replay
Fork
```

## Replay

Start execution from a previous checkpoint.

```text
Checkpoint 1
    ↓
Checkpoint 2
    ↓
Checkpoint 3
    ↓
Checkpoint 4
```

Replay from checkpoint 2:

```text
Checkpoint 2
    ↓
Node 3
    ↓
Node 4
```

Nodes before the checkpoint do not need to execute again.

## Fork

Modify state at an earlier checkpoint and explore another trajectory.

```text
             Checkpoint 2
                  │
          ┌───────┴───────┐
          ▼               ▼
       Original         Fork
        path             path
```

This is useful for:

* debugging
* experimentation
* evaluating agent decisions
* analyzing failures

LangGraph documents replay and fork as time-travel capabilities built on checkpoints.

---

# 42. State Inspection

You can inspect persisted state.

Example:

```python
state = graph.get_state(config)

print(state)
```

History:

```python
history = graph.get_state_history(config)

for snapshot in history:
    print(snapshot)
```

This is extremely useful when debugging complex agents.

---

# 43. State Updates

You can update state programmatically:

```python
graph.update_state(
    config,
    {
        "approved": True
    }
)
```

This creates a new checkpoint rather than modifying the historical checkpoint in place.

---

# 44. Retry Policies

LLM and API calls can fail because of:

* network errors
* rate limits
* temporary service failures
* database errors
* external API failures

LangGraph allows retry policies on nodes.

Example:

```python
from langgraph.types import RetryPolicy


builder.add_node(
    "retrieve",
    retrieve,
    retry_policy=RetryPolicy(
        max_attempts=3
    )
)
```

Retry behavior can be customized according to the failure type.

---

# 45. Retry Strategy

Do not blindly retry everything.

Good retry candidates:

```text
Network timeout
Rate limit
Temporary service unavailable
Transient database error
```

Bad retry candidates:

```text
Invalid input
Authentication failure
Malformed request
Business-rule violation
```

Production agents should distinguish between:

```text
Transient error
Permanent error
Recoverable by LLM
Recoverable by human
Unexpected error
```

---

# 46. Node Caching

Expensive nodes may be cached.

Example:

```python
from langgraph.cache.memory import InMemoryCache
from langgraph.types import CachePolicy


builder.add_node(
    "expensive_operation",
    expensive_operation,
    cache_policy=CachePolicy(
        ttl=120
    )
)

graph = builder.compile(
    cache=InMemoryCache()
)
```

Caching can reduce:

* latency
* API calls
* model costs
* repeated computation

LangGraph supports node-level cache policies and cache implementations.

---

# 47. Error Handling

A production node should consider failure explicitly.

Basic:

```python
def call_api(state: State):

    try:
        result = api_call()

        return {
            "result": result
        }

    except Exception as e:

        return {
            "error": str(e)
        }
```

But not every error should simply be swallowed.

A better architecture is:

```text
API Call
   │
   ├── Success → Continue
   │
   ├── Temporary Failure → Retry
   │
   ├── LLM-recoverable → Agent Retry
   │
   ├── User-fixable → Human
   │
   └── Unexpected → Fail
```

---

# 48. Thinking in LangGraph

A useful design process is:

## Step 1 — Start with the business process

Do not begin with:

> "Which LangGraph API should I use?"

Begin with:

> "What process am I automating?"

---

## Step 2 — Break it into discrete steps

Example:

```text
Receive email
↓
Classify
↓
Retrieve information
↓
Generate response
↓
Evaluate
↓
Escalate if necessary
```

---

## Step 3 — Define state

Ask:

> What information needs to travel between steps?

Example:

```python
class State(TypedDict):
    email: str
    category: str
    documents: list[str]
    draft: str
    confidence: float
```

---

## Step 4 — Build nodes

```text
classify()
retrieve()
generate()
evaluate()
escalate()
```

---

## Step 5 — Connect nodes

```text
START
 ↓
classify
 ↓
retrieve
 ↓
generate
 ↓
evaluate
 ↓
┌──────────────┐
│ confidence?  │
└──────┬───────┘
       │
  ┌────┴────┐
  ▼         ▼
final     escalate
```

This workflow-first mindset is recommended in LangGraph's own "Thinking in LangGraph" guidance.

---

# 49. Keep State Raw

A good design principle is:

> Store raw information in state and format prompts when needed.

Avoid stuffing state with unnecessary prompt-formatted strings.

Prefer:

```python
class State(TypedDict):
    question: str
    documents: list[str]
```

Then:

```python
def generate(state):

    prompt = f"""
    Question:
    {state["question"]}

    Documents:
    {state["documents"]}
    """

    ...
```

instead of permanently storing:

```python
prompt = """
You are an expert...
...
"""
```

inside the state.

This keeps the graph easier to reason about and debug.

---

# 50. Deterministic + Agentic Workflows

One of LangGraph's strongest patterns is combining deterministic software with probabilistic LLM behavior.

Example:

```text
              Deterministic
                  │
                  ▼
            Validate Input
                  │
                  ▼
             LLM Agent
                  │
                  ▼
           Tool Selection
                  │
                  ▼
          Deterministic API
                  │
                  ▼
            LLM Evaluation
                  │
                  ▼
          Deterministic Guard
                  │
                  ▼
               Output
```

This is usually better than giving an LLM complete control over the entire application.

---

# 51. Agent Loop

A classic tool-using agent can be modeled as:

```text
START
  ↓
Agent
  ↓
Does agent need tool?
  ├── No → END
  │
  └── Yes
       ↓
   Tool Executor
       ↓
      Agent
```

The graph explicitly represents the agent loop.

---

# 52. RAG with LangGraph

A RAG workflow:

```text
             User Query
                 │
                 ▼
             Retrieve
                 │
                 ▼
          Retrieved Docs
                 │
                 ▼
             Rerank
                 │
                 ▼
             Generate
                 │
                 ▼
             Evaluate
                 │
            ┌────┴────┐
            ▼         ▼
          Good       Bad
            │         │
            ▼         ▼
           END     Retrieve Again
```

Possible nodes:

```python
retrieve()
rerank()
generate()
evaluate()
rewrite_query()
```

LangGraph is especially useful when RAG is not simply:

```text
retrieve → generate
```

but instead requires:

```text
retrieve → evaluate → retry → rewrite → retrieve
```

---

# 53. Corrective RAG

Corrective RAG:

```text
Question
   ↓
Retrieve
   ↓
Evaluate Documents
   ↓
Relevant?
 ┌─┴───────────────┐
 │                 │
Yes               No
 │                 │
 ▼                 ▼
Generate        Rewrite Query
                   │
                   ▼
                Retrieve
```

This is a natural LangGraph pattern.

---

# 54. Reflection Agent

A reflection agent:

```text
Generate
   ↓
Critic
   ↓
Good?
 ┌─┴────┐
Yes     No
 │       │
 ▼       ▼
END    Improve
          │
          └──→ Generate
```

State might contain:

```python
class State(TypedDict):
    task: str
    draft: str
    critique: str
    score: float
```

---

# 55. Planner → Executor Architecture

A common agent architecture:

```text
User
 ↓
Planner
 ↓
Plan
 ↓
Executor
 ↓
Tool Calls
 ↓
Results
 ↓
Reviewer
 ↓
Final Answer
```

State:

```python
class State(TypedDict):
    task: str
    plan: list[str]
    results: list[str]
    final_answer: str
```

---

# 56. Supervisor Architecture

For multi-agent systems:

```text
                 Supervisor
                 /    |    \
                /     |     \
               ▼      ▼      ▼
          Research   Code   Review
               \      |      /
                \     |     /
                 ▼    ▼    ▼
                  Supervisor
                      │
                      ▼
                     END
```

The supervisor decides:

```text
Which agent should act next?
```

This can be implemented with:

* conditional edges
* `Command`
* subgraphs
* LLM-based routing

---

# 57. LangGraph Functional API

LangGraph provides two major styles:

```text
Graph API
Functional API
```

## Graph API

Explicitly define:

```text
State
Nodes
Edges
```

Best when you want:

* visual workflows
* explicit control flow
* complex branching
* complex graphs
* multi-agent architectures

---

## Functional API

Use functions/tasks/entrypoints to model workflows more directly.

Conceptually:

```python
from langgraph.func import entrypoint, task


@task
def step_1(x):
    return x * 2


@task
def step_2(x):
    return x + 10


@entrypoint()
def workflow(x):

    result1 = step_1(x).result()
    result2 = step_2(result1).result()

    return result2
```

The Functional API can be more natural when the workflow is fundamentally sequential code with durable execution requirements.

---

# 58. Graph API vs Functional API

| Feature             | Graph API | Functional API       |
| ------------------- | --------- | -------------------- |
| Explicit graph      | Excellent | Less explicit        |
| Nodes               | Yes       | Tasks                |
| Edges               | Explicit  | Control flow in code |
| Branching           | Excellent | Python control flow  |
| Loops               | Excellent | Python loops         |
| Visualization       | Excellent | Less graph-oriented  |
| Complex routing     | Excellent | Good                 |
| Simple workflows    | Good      | Excellent            |
| Multi-agent graphs  | Excellent | Good                 |
| Learning difficulty | Higher    | Lower initially      |

### Rule of thumb

Use **Graph API** when the graph itself is an important part of your architecture.

Use **Functional API** when normal Python control flow expresses the workflow more naturally.

---

# 59. Graph Compilation

Building:

```python
builder = StateGraph(State)
```

does not mean you have a runnable graph yet.

You generally:

```python
graph = builder.compile()
```

Compilation validates and prepares the graph for execution.

With persistence:

```python
graph = builder.compile(
    checkpointer=checkpointer
)
```

With caching:

```python
graph = builder.compile(
    checkpointer=checkpointer,
    cache=cache
)
```

---

# 60. invoke()

Use `invoke()` when you want the result after execution.

```python
result = graph.invoke(
    {"question": "What is RAG?"}
)

print(result)
```

Conceptually:

```text
Input
 ↓
Graph executes
 ↓
Final state
```

---

# 61. stream()

Use streaming when you want intermediate results.

```python
for chunk in graph.stream(
    {"question": "What is RAG?"}
):
    print(chunk)
```

Conceptually:

```text
Input
 ↓
Node 1 → chunk
 ↓
Node 2 → chunk
 ↓
Node 3 → chunk
 ↓
Final
```

---

# 62. Async Execution

For async applications:

```python
result = await graph.ainvoke(
    {"question": "Hello"}
)
```

And:

```python
async for chunk in graph.astream(
    {"question": "Hello"}
):
    print(chunk)
```

This is useful in:

* FastAPI
* async web applications
* high-concurrency systems
* streaming interfaces

---

# 63. Configuration

Configuration commonly contains runtime execution information such as the thread identifier.

Example:

```python
config = {
    "configurable": {
        "thread_id": "conversation-1"
    }
}
```

Then:

```python
graph.invoke(
    input_data,
    config=config
)
```

Do not confuse:

```text
config
```

with:

```text
state
```

State represents application data.

Config/runtime context represents execution configuration and dependencies.

---

# 64. Common Architecture

A production-grade LangGraph agent might look like:

```text
                        ┌──────────────┐
                        │    Input     │
                        └──────┬───────┘
                               ▼
                        ┌──────────────┐
                        │    Router    │
                        └──────┬───────┘
                               │
                ┌──────────────┼──────────────┐
                ▼              ▼              ▼
           Research         Execute        Support
                │              │              │
                └──────────────┼──────────────┘
                               ▼
                         ┌────────────┐
                         │  Evaluate  │
                         └─────┬──────┘
                               │
                         ┌─────┴─────┐
                         ▼           ▼
                       Pass        Retry
                         │           │
                         ▼           └──────→ Router
                       Human
                       Review
                         │
                         ▼
                       Final
```

With infrastructure:

```text
                ┌─────────────────────┐
                │      LangGraph      │
                │                     │
                │ State               │
                │ Nodes               │
                │ Edges               │
                │ Runtime             │
                │ Interrupts          │
                └─────────┬───────────┘
                          │
              ┌───────────┼────────────┐
              ▼           ▼            ▼
         Checkpointer    Store     Observability
              │           │            │
              ▼           ▼            ▼
          Persistence   Memory      LangSmith
```

---

# 65. Production Considerations

A production LangGraph system should consider:

## State design

Keep state:

* minimal
* explicit
* serializable
* meaningful

Avoid storing huge unnecessary objects.

---

## Idempotency

External side effects should be safe to retry.

Bad:

```python
send_money()
```

without safeguards.

Better:

```python
if not transaction_already_processed():
    execute_transaction()
```

---

## Retries

Retry transient failures.

Do not retry permanent failures blindly.

---

## Persistence

Use durable persistence for workflows that must survive:

* crashes
* restarts
* human delays
* long execution

---

## Human approval

Use interrupts for risky operations.

---

## Observability

Trace:

* node execution
* latency
* model calls
* tool calls
* failures
* state transitions
* retries

LangSmith is commonly used with LangGraph for tracing, debugging, and evaluation.

---

# 66. LangSmith

LangGraph handles orchestration.

LangSmith provides observability around the application.

Conceptually:

```text
LangGraph
    │
    ├── Node execution
    ├── LLM calls
    ├── Tool calls
    ├── State transitions
    └── Errors
            │
            ▼
        LangSmith
            │
            ├── Tracing
            ├── Debugging
            ├── Evaluation
            └── Monitoring
```

This becomes particularly important for complex multi-agent systems.

---

# 67. LangGraph Studio

LangGraph Studio provides an environment for inspecting and interacting with agent graphs during development.

A local development setup can use:

```bash
pip install --upgrade "langgraph-cli[inmem]"
```

The CLI can connect a local agent server to Studio for development and inspection.

---

# 68. Testing LangGraph

Do not test an agent only by asking:

> "Does the final answer look good?"

Test individual nodes.

Example:

```python
def test_retrieve():

    state = {
        "question": "What is RAG?"
    }

    result = retrieve(state)

    assert len(result["documents"]) > 0
```

Test routing:

```python
def test_router():

    state = {
        "score": 0.9
    }

    assert route(state) == "done"
```

Test full graph:

```python
result = graph.invoke(
    {
        "question": "What is RAG?"
    }
)

assert result["answer"]
```

---

# 69. Testing Strategy

A strong agent testing pyramid:

```text
                ┌───────────────┐
                │ End-to-End    │
                └───────┬───────┘
                        │
                 ┌──────┴──────┐
                 │ Graph Tests │
                 └──────┬──────┘
                        │
                ┌───────┴────────┐
                │ Node Tests     │
                └───────┬────────┘
                        │
               ┌────────┴─────────┐
               │ Unit Tests       │
               └──────────────────┘
```

Test:

* node correctness
* routing
* state updates
* tool errors
* retries
* interrupts
* persistence
* final outputs

---

# 70. Common Mistakes

## Mistake 1 — Putting everything into state

Bad:

```python
class State(TypedDict):
    everything: dict
```

Prefer explicit fields.

---

## Mistake 2 — Treating LangGraph as an LLM

LangGraph is not:

```text
LLM
```

It is:

```text
Orchestration Runtime
```

---

## Mistake 3 — Using an LLM for deterministic routing

If the decision is deterministic:

```python
if score > 0.8:
    ...
```

do not unnecessarily ask an LLM.

---

## Mistake 4 — No retry strategy

External systems fail.

Design for failure.

---

## Mistake 5 — No persistence for long-running workflows

If the workflow needs:

* human approval
* resume
* memory
* fault recovery

use checkpointing appropriately.

---

## Mistake 6 — Huge monolithic graph

Instead:

```text
Main Graph
 ├── Research Subgraph
 ├── Retrieval Subgraph
 ├── Analysis Subgraph
 └── Review Subgraph
```

---

## Mistake 7 — Ignoring idempotency

Retries can duplicate side effects.

Design external actions to be safely repeatable.

---

## Mistake 8 — No observability

A multi-agent system without traces is difficult to debug.

---

# 71. Important API Cheat Sheet

```python
from langgraph.graph import (
    StateGraph,
    START,
    END,
    MessagesState,
)
```

Build graph:

```python
builder = StateGraph(State)
```

Add node:

```python
builder.add_node("name", function)
```

Add edge:

```python
builder.add_edge("a", "b")
```

Start:

```python
builder.add_edge(START, "a")
```

End:

```python
builder.add_edge("a", END)
```

Conditional:

```python
builder.add_conditional_edges(
    "node",
    router,
    {
        "a": "node_a",
        "b": "node_b"
    }
)
```

Compile:

```python
graph = builder.compile()
```

Execute:

```python
graph.invoke(...)
```

Stream:

```python
graph.stream(...)
```

Async:

```python
graph.ainvoke(...)
```

Interrupt:

```python
from langgraph.types import interrupt
```

Resume:

```python
from langgraph.types import Command
```

Retry:

```python
from langgraph.types import RetryPolicy
```

Cache:

```python
from langgraph.types import CachePolicy
```

Persistence:

```python
from langgraph.checkpoint.memory import InMemorySaver
```

---

# 72. Important Imports

## Graph

```python
from langgraph.graph import (
    StateGraph,
    START,
    END,
    MessagesState,
)
```

## Control

```python
from langgraph.types import (
    Command,
    Send,
    RetryPolicy,
    CachePolicy,
)
```

## Interrupts

```python
from langgraph.types import interrupt
```

## Persistence

```python
from langgraph.checkpoint.memory import InMemorySaver
```

## Runtime

```python
from langgraph.runtime import Runtime
```

## Functional API

```python
from langgraph.func import (
    entrypoint,
    task,
)
```

---

# 73. Minimal Complete Example

```python
from typing_extensions import TypedDict

from langgraph.graph import (
    StateGraph,
    START,
    END,
)


class State(TypedDict):
    question: str
    answer: str


def generate(state: State):

    answer = f"Processed: {state['question']}"

    return {
        "answer": answer
    }


builder = StateGraph(State)

builder.add_node(
    "generate",
    generate
)

builder.add_edge(
    START,
    "generate"
)

builder.add_edge(
    "generate",
    END
)

graph = builder.compile()


result = graph.invoke(
    {
        "question": "What is LangGraph?"
    }
)

print(result)
```

---

# 74. Complete Conditional Example

```python
from typing_extensions import TypedDict

from langgraph.graph import (
    StateGraph,
    START,
    END,
)


class State(TypedDict):
    question: str
    answer: str
    score: float


def generate(state: State):

    return {
        "answer": "Generated answer"
    }


def evaluate(state: State):

    return {
        "score": 0.9
    }


def route(state: State):

    if state["score"] >= 0.8:
        return "good"

    return "retry"


builder = StateGraph(State)

builder.add_node(
    "generate",
    generate
)

builder.add_node(
    "evaluate",
    evaluate
)

builder.add_edge(
    START,
    "generate"
)

builder.add_edge(
    "generate",
    "evaluate"
)

builder.add_conditional_edges(
    "evaluate",
    route,
    {
        "good": END,
        "retry": "generate",
    }
)

graph = builder.compile()


result = graph.invoke(
    {
        "question": "Explain RAG",
        "answer": "",
        "score": 0
    }
)

print(result)
```

---

# 75. Complete Agent Architecture

A more realistic agent:

```text
                           START
                             │
                             ▼
                     ┌──────────────┐
                     │   Planner    │
                     └──────┬───────┘
                            │
                            ▼
                     ┌──────────────┐
                     │   Research   │
                     └──────┬───────┘
                            │
                            ▼
                     ┌──────────────┐
                     │   Retrieve   │
                     └──────┬───────┘
                            │
                            ▼
                     ┌──────────────┐
                     │    Agent     │◄──────────┐
                     └──────┬───────┘           │
                            │                   │
                     Tool required?             │
                      /         \                │
                    No           Yes             │
                    │             │              │
                    │             ▼              │
                    │       Tool Executor        │
                    │             │              │
                    │             └──────────────┘
                    │
                    ▼
               ┌──────────┐
               │ Evaluate │
               └────┬─────┘
                    │
              ┌─────┴─────┐
              ▼           ▼
           Approved      Reject
              │           │
              ▼           ▼
            Human       Retry
              │           │
              ▼           └──────────→ Agent
            Final
              │
              ▼
             END
```

This combines:

* planning
* retrieval
* tool calling
* loops
* evaluation
* human approval
* state
* persistence

---

# 76. LangGraph Architecture for RAG

For advanced RAG:

```text
                         Query
                           │
                           ▼
                    Query Analyzer
                           │
                           ▼
                    Query Rewriter
                           │
                           ▼
                       Retriever
                           │
                           ▼
                     Reranker
                           │
                           ▼
                    Context Judge
                       /       \
                     Good       Bad
                      │          │
                      │          ▼
                      │       Rewrite
                      │          │
                      │          └──────→ Retriever
                      ▼
                   Generator
                      │
                      ▼
                  Validator
                  /       \
                Good       Bad
                 │          │
                 ▼          ▼
                END       Regenerate
```

This is where LangGraph becomes much more useful than a simple chain.

---

# 77. LangGraph Design Principles

Remember these principles:

### 1. Model the workflow first

Do not start with APIs.

### 2. Keep state explicit

Know exactly what moves between nodes.

### 3. Separate deterministic and probabilistic logic

Use normal Python for deterministic rules.

Use LLMs where reasoning/generation is actually needed.

### 4. Design for failure

External services fail.

### 5. Make side effects idempotent

Retries can happen.

### 6. Use persistence deliberately

Not every workflow needs long-term state.

### 7. Use subgraphs for modularity

Do not create one giant graph.

### 8. Observe everything important

Tracing becomes essential as complexity increases.

### 9. Use humans for high-risk decisions

Human-in-the-loop is a feature, not a failure.

### 10. Keep state small and meaningful

State is the backbone of the workflow.

---

# 78. LangGraph Interview Questions

## Beginner

### What is LangGraph?

A low-level orchestration framework and runtime for building stateful, long-running AI workflows and agents.

### What are the three fundamental graph concepts?

```text
State
Nodes
Edges
```

### What is a node?

A function that performs work and returns state updates.

### What is an edge?

A transition that determines the next node.

### What is START?

The graph entry point.

### What is END?

The graph termination point.

---

# 79. Intermediate Interview Questions

### What is a reducer?

A reducer determines how updates to a state key are combined.

### Why use conditional edges?

To dynamically determine the next step based on state.

### What is Command?

A control primitive that can combine state updates with routing.

### What is Send?

A mechanism useful for dynamically distributing work across parallel branches.

### What is checkpointing?

Saving graph state at execution steps for persistence and recovery.

### What is a thread?

A persistent execution context identified by a thread ID.

### What are interrupts?

Dynamic pauses in graph execution that allow external input or human approval.

---

# 80. Advanced Interview Questions

### How would you design a production multi-agent system?

```text
Supervisor
    ↓
Specialized subagents
    ↓
Shared / isolated state
    ↓
Tools
    ↓
Evaluation
    ↓
Human approval where necessary
    ↓
Persistent execution
    ↓
Observability
```

### How would you prevent duplicate side effects?

Use:

* idempotency keys
* transactional operations
* state checks
* external workflow identifiers
* safe retry semantics

### How would you implement self-correction?

```text
Generate
 ↓
Evaluate
 ↓
Pass? ── Yes → END
 ↓
No
 ↓
Improve
 ↓
Generate
```

### How would you debug an agent that made the wrong decision?

Inspect:

```text
Thread
 ↓
Checkpoint history
 ↓
State
 ↓
Node outputs
 ↓
LLM calls
 ↓
Routing decision
```

Use time travel and tracing where appropriate.

---

# 81. LangGraph vs Traditional Chain

## Traditional Chain

```text
A → B → C → D
```

Good for deterministic workflows.

## LangGraph

```text
        ┌────── B ──────┐
        │               │
A ──────┤               ├──→ D
        │               │
        └──── C ────────┘
              ↑
              │
              └── retry
```

LangGraph becomes valuable when workflows have:

* branches
* loops
* state
* persistence
* agents
* human interaction
* dynamic routing

---

# 82. LangGraph vs LangChain Agent

```text
LangChain Agent
       │
       ▼
High-level agent abstraction
       │
       ▼
LangGraph Runtime
       │
       ├── State
       ├── Nodes
       ├── Edges
       ├── Persistence
       ├── Interrupts
       └── Execution
```

Use the higher-level abstraction when it fits.

Drop down to LangGraph when you need control.

---

# 83. Recommended Learning Order

For mastering LangGraph:

```text
01. Graph Mental Model
        ↓
02. State
        ↓
03. Nodes
        ↓
04. Edges
        ↓
05. Conditional Edges
        ↓
06. Loops
        ↓
07. LLM Integration
        ↓
08. Tool Calling
        ↓
09. Command
        ↓
10. Send
        ↓
11. Persistence
        ↓
12. Threads
        ↓
13. Memory
        ↓
14. Interrupts
        ↓
15. Streaming
        ↓
16. Subgraphs
        ↓
17. Multi-Agent Systems
        ↓
18. Retry / Error Handling
        ↓
19. Caching
        ↓
20. Time Travel
        ↓
21. Observability
        ↓
22. Production Deployment
```

---

# 84. Projects to Build

## Project 1 — Basic Workflow

```text
Input
 ↓
Process
 ↓
Output
```

Learn:

* State
* Nodes
* Edges

---

## Project 2 — RAG Agent

```text
Query
 ↓
Retrieve
 ↓
Generate
 ↓
Evaluate
```

Learn:

* State
* LLMs
* RAG
* conditional routing

---

## Project 3 — Self-Correcting RAG

```text
Query
 ↓
Retrieve
 ↓
Evaluate
 ├── Good → Generate
 └── Bad → Rewrite → Retrieve
```

Learn:

* loops
* evaluation
* routing

---

## Project 4 — Research Agent

```text
Planner
 ↓
Research
 ↓
Search
 ↓
Summarize
 ↓
Critic
 ↓
Final
```

Learn:

* tool calling
* loops
* state

---

## Project 5 — Human Approval Agent

```text
Agent
 ↓
Action
 ↓
Human Approval
 ├── Approve → Execute
 └── Reject → Revise
```

Learn:

* interrupts
* persistence
* Command

---

## Project 6 — Multi-Agent Research System

```text
              Supervisor
              /    |    \
             /     |     \
      Researcher  Coder  Critic
             \     |     /
              \    |    /
               Supervisor
                    │
                    ▼
                  Final
```

Learn:

* subgraphs
* supervisor routing
* multi-agent state
* parallel execution

---

# 85. Production-Grade Agent Stack

A strong modern AI engineering stack can look like:

```text
                    Frontend
                       │
                       ▼
                    FastAPI
                       │
                       ▼
                  LangGraph
                       │
          ┌────────────┼────────────┐
          ▼            ▼            ▼
        LLMs         Tools        RAG
          │            │            │
          ▼            ▼            ▼
       Groq/API     APIs/DB      Vector DB
                       │
                       ▼
                 PostgreSQL
                       │
                       ▼
                 Checkpointer
                       │
                       ▼
                  LangSmith
```

Additional infrastructure can include:

```text
Redis
PostgreSQL
pgvector
Docker
Prometheus
Grafana
MLflow
Evidently
```

depending on project requirements.

---

# 86. The Most Important Mental Model

If you remember only one thing:

```text
                    STATE
                      │
          ┌───────────┼───────────┐
          ▼           ▼           ▼
        NODE        NODE        NODE
          │           │           │
          └───────────┼───────────┘
                      ▼
                     EDGE
                      │
                Decision / Loop
                      │
                      ▼
                    NODE
```

LangGraph is fundamentally about **controlling how state moves through an AI workflow**.

---

# 87. Final Summary

LangGraph gives you explicit control over agent execution.

The core abstraction is:

```text
State + Nodes + Edges
```

Then add:

```text
Conditional Routing
        +
Loops
        +
Tools
        +
LLMs
        +
Command / Send
        +
Persistence
        +
Interrupts
        +
Streaming
        +
Subgraphs
        +
Retries
        +
Caching
        +
Time Travel
        +
Observability
```

This transforms a basic LLM call into a controllable agent runtime.

The key progression is:

```text
LLM
 ↓
Chain
 ↓
Agent
 ↓
Stateful Agent
 ↓
Graph
 ↓
Persistent Graph
 ↓
Human-in-the-Loop
 ↓
Multi-Agent System
 ↓
Production Agent Runtime
```

---

# 88. Quick Revision Sheet

```text
LangGraph
│
├── Core
│   ├── State
│   ├── Nodes
│   ├── Edges
│   ├── START
│   └── END
│
├── Routing
│   ├── Conditional Edges
│   ├── Command
│   └── Send
│
├── Execution
│   ├── invoke()
│   ├── stream()
│   ├── ainvoke()
│   └── astream()
│
├── State
│   ├── TypedDict
│   ├── MessagesState
│   └── Reducers
│
├── Runtime
│   ├── Context
│   ├── Store
│   ├── Stream Writer
│   └── Execution Info
│
├── Persistence
│   ├── Checkpoints
│   ├── Threads
│   ├── Memory
│   ├── Replay
│   └── Fork
│
├── Human-in-the-Loop
│   ├── interrupt()
│   └── Command(resume=...)
│
├── Reliability
│   ├── RetryPolicy
│   ├── Error Handling
│   ├── Durable Execution
│   └── Idempotency
│
├── Performance
│   ├── Streaming
│   ├── Parallelism
│   └── CachePolicy
│
├── Architecture
│   ├── Subgraphs
│   ├── Multi-Agent
│   ├── Supervisor
│   ├── Planner
│   └── RAG
│
└── Production
    ├── LangSmith
    ├── Testing
    ├── Observability
    ├── Persistence
    └── Deployment
```

---

# 89. Official Documentation Topics to Revisit

For deeper study, the most important official LangGraph documentation areas are:

* LangGraph Overview
* Graph API
* Functional API
* Persistence
* Interrupts
* Time Travel
* Subgraphs
* Streaming
* Thinking in LangGraph
* LangGraph Studio

These concepts correspond closely to the current LangGraph documentation and are the best progression from beginner graph construction to production agent orchestration.

---

# 90. One-Line Definition

> **LangGraph is a low-level orchestration runtime that lets you build stateful, controllable, persistent, and long-running AI agents by modeling their execution as a graph of state, nodes, and transitions.**