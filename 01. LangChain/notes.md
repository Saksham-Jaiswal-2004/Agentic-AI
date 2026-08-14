# 🦜🔗 LangChain — Complete Notes

> **LangChain** is a framework for building applications powered by Large Language Models (LLMs), especially applications that require **structured prompts, tool usage, retrieval, memory, agents, workflows, and integration with external systems**.

---

## 📚 Table of Contents

1. [What is LangChain?](#1-what-is-langchain)
2. [Why LangChain?](#2-why-langchain)
3. [LangChain Architecture](#3-langchain-architecture)
4. [Installation](#4-installation)
5. [LLMs and Chat Models](#5-llms-and-chat-models)
6. [Messages](#6-messages)
7. [Prompts](#7-prompts)
8. [Output Parsers](#8-output-parsers)
9. [LCEL](#9-lcel)
10. [Runnables](#10-runnables)
11. [Structured Output](#11-structured-output)
12. [Tools](#12-tools)
13. [Tool Calling](#13-tool-calling)
14. [Agents](#14-agents)
15. [Agent Execution](#15-agent-execution)
16. [Retrieval](#16-retrieval)
17. [Document Loaders](#17-document-loaders)
18. [Text Splitters](#18-text-splitters)
19. [Embeddings](#19-embeddings)
20. [Vector Stores](#20-vector-stores)
21. [Retrievers](#21-retrievers)
22. [RAG](#22-rag)
23. [Conversation History](#23-conversation-history)
24. [Short-Term Memory](#24-short-term-memory)
25. [Long-Term Memory](#25-long-term-memory)
26. [Callbacks](#26-callbacks)
27. [Streaming](#27-streaming)
28. [Async Execution](#28-async-execution)
29. [Middleware](#29-middleware)
30. [LangChain Agents vs LangGraph](#30-langchain-agents-vs-langgraph)
31. [LangSmith](#31-langsmith)
32. [Evaluation](#32-evaluation)
33. [Production Architecture](#33-production-architecture)
34. [Common Design Patterns](#34-common-design-patterns)
35. [Common Mistakes](#35-common-mistakes)
36. [Best Practices](#36-best-practices)
37. [LangChain + Groq](#37-langchain--groq)
38. [Mini Projects](#38-mini-projects)
39. [Important APIs](#39-important-apis)
40. [Learning Roadmap](#40-learning-roadmap)

---

# 1. What is LangChain?

LangChain is an open-source framework for developing applications around LLMs.

A raw LLM can:

```text
Input → LLM → Output
```

A LangChain application can look like:

```text
User
  ↓
Prompt
  ↓
LLM
  ↓
Tool Selection
  ↓
External API / Database / Retriever
  ↓
LLM
  ↓
Structured Response
```

This makes LangChain useful for:

* AI assistants
* RAG applications
* Agents
* Tool-using systems
* Document analysis
* Question answering
* Autonomous workflows
* AI automation
* Data extraction
* Multi-step reasoning systems

---

# 2. Why LangChain?

Without a framework, developers have to manually manage:

* prompt construction
* message formatting
* model APIs
* tool schemas
* structured output
* retrieval
* document processing
* conversation state
* streaming
* retries
* observability
* agent loops

LangChain provides abstractions around these components.

### Traditional LLM Application

```text
Application
    ↓
OpenAI/Groq/Anthropic API
    ↓
LLM
```

### LangChain Application

```text
Application
    ↓
LangChain
 ┌──┴────┬────────┬─────────┐
 ↓       ↓        ↓         ↓
Models  Prompts  Tools   Retrieval
                 ↓
               APIs
```

---

# 3. LangChain Architecture

Modern LangChain can be understood through several major layers.

```text
                    LangChain Application
                           │
             ┌─────────────┴─────────────┐
             │                           │
          Models                       Agents
             │                           │
       ┌─────┴─────┐               ┌─────┴─────┐
       │           │               │           │
      LLMs     Chat Models       Tools      Middleware
       │                           │
       └────────────┬──────────────┘
                    │
                 Runnables
                    │
              Retrieval / RAG
                    │
        ┌───────────┼────────────┐
        ↓           ↓            ↓
   Documents    Embeddings   Vector Stores
```

LangChain also integrates closely with:

* LangGraph
* LangSmith
* LangServe
* community integrations
* provider-specific packages

---

# 4. Installation

## Basic Installation

```bash
pip install langchain
```

## Common Integrations

```bash
pip install langchain-core
pip install langchain-community
pip install langchain-text-splitters
```

For Groq:

```bash
pip install langchain-groq
```

For OpenAI:

```bash
pip install langchain-openai
```

For Hugging Face:

```bash
pip install langchain-huggingface
```

For vector databases, install the appropriate integration.

Example:

```bash
pip install langchain-pinecone
```

---

# 5. LLMs and Chat Models

LangChain distinguishes between traditional LLM-style models and chat models.

## LLM

Conceptually:

```text
Prompt → Completion
```

Example:

```python
from langchain_openai import OpenAI

llm = OpenAI()

response = llm.invoke(
    "Explain machine learning"
)

print(response)
```

---

## Chat Model

Chat models operate on messages.

```text
System Message
      ↓
Human Message
      ↓
AI Message
      ↓
Human Message
```

Example:

```python
from langchain_openai import ChatOpenAI

model = ChatOpenAI(
    model="gpt-4o-mini"
)

response = model.invoke(
    "Explain machine learning"
)

print(response.content)
```

---

# 6. Messages

Chat-based applications use different message types.

## HumanMessage

Represents user input.

```python
from langchain_core.messages import HumanMessage

message = HumanMessage(
    content="Explain RAG"
)
```

---

## SystemMessage

Defines model behavior.

```python
from langchain_core.messages import SystemMessage

message = SystemMessage(
    content="You are an expert AI engineer."
)
```

---

## AIMessage

Represents the model's response.

```python
from langchain_core.messages import AIMessage

message = AIMessage(
    content="RAG stands for Retrieval-Augmented Generation."
)
```

---

## ToolMessage

Represents the result returned by a tool.

```text
AIMessage
   ↓
Tool Call
   ↓
ToolMessage
   ↓
AIMessage
```

This structure is fundamental to tool-using agents.

---

# 7. Prompts

Prompts define how information is presented to the model.

## Basic Prompt

```python
from langchain_core.prompts import PromptTemplate

prompt = PromptTemplate.from_template(
    "Explain {topic} in simple terms."
)

result = prompt.invoke({
    "topic": "RAG"
})
```

---

# ChatPromptTemplate

For chat models:

```python
from langchain_core.prompts import ChatPromptTemplate

prompt = ChatPromptTemplate.from_messages([
    (
        "system",
        "You are an expert AI engineer."
    ),
    (
        "human",
        "Explain {topic}"
    )
])
```

Invoke:

```python
result = prompt.invoke({
    "topic": "LangGraph"
})
```

---

# Prompt Variables

```python
prompt = ChatPromptTemplate.from_messages([
    (
        "system",
        "You are a {role}."
    ),
    (
        "human",
        "Explain {topic}"
    )
])

prompt.invoke({
    "role": "teacher",
    "topic": "agents"
})
```

---

# 8. Output Parsers

Models usually return text.

Sometimes we need structured results.

Traditional LangChain applications can use output parsers.

Example:

```python
from langchain_core.output_parsers import StrOutputParser

parser = StrOutputParser()

result = parser.invoke(
    some_ai_message
)
```

The parser extracts:

```text
AIMessage
    ↓
String
```

Modern LangChain applications should generally prefer **native structured output from the model** when available.

---

# 9. LCEL

## LangChain Expression Language

LCEL allows components to be composed using the pipe operator:

```python
chain = prompt | model | parser
```

Conceptually:

```text
Prompt
  ↓
Model
  ↓
Parser
```

---

## Example

```python
from langchain_core.prompts import ChatPromptTemplate
from langchain_core.output_parsers import StrOutputParser

prompt = ChatPromptTemplate.from_template(
    "Explain {topic}"
)

chain = prompt | model | StrOutputParser()

response = chain.invoke({
    "topic": "Agentic AI"
})

print(response)
```

---

# Why LCEL?

LCEL provides:

* composability
* streaming support
* async execution
* batching
* observability
* parallel execution
* clean pipelines

---

# 10. Runnables

A **Runnable** is a central abstraction in LangChain.

Many LangChain components implement the Runnable interface.

Common methods include:

```python
.invoke()
.batch()
.stream()
.ainvoke()
.abatch()
.astream()
```

---

## invoke()

Used for a single input.

```python
result = chain.invoke(input)
```

---

## batch()

Used for multiple inputs.

```python
results = chain.batch([
    {"topic": "RAG"},
    {"topic": "Agents"},
    {"topic": "LangGraph"}
])
```

---

## stream()

Streams output progressively.

```python
for chunk in chain.stream(input):
    print(chunk)
```

---

## ainvoke()

Asynchronous execution.

```python
result = await chain.ainvoke(input)
```

---

# Runnable Composition

```python
chain = prompt | model | parser
```

Each component receives the previous component's output.

```text
Input
 ↓
Prompt
 ↓
Model
 ↓
Parser
 ↓
Output
```

---

# RunnableParallel

Run multiple operations simultaneously.

Conceptually:

```text
             ┌→ Chain A
Input ───────┼→ Chain B
             └→ Chain C
```

Useful when independent computations can happen concurrently.

---

# RunnablePassthrough

Pass input unchanged.

```python
from langchain_core.runnables import RunnablePassthrough
```

Useful in RAG pipelines.

---

# 11. Structured Output

Structured output is extremely important for production AI systems.

Instead of:

```text
"John is 25 and lives in Kolkata."
```

we want:

```json
{
    "name": "John",
    "age": 25,
    "city": "Kolkata"
}
```

---

## Pydantic Schema

```python
from pydantic import BaseModel, Field

class ContactInfo(BaseModel):
    name: str = Field(
        description="Person's name"
    )

    email: str = Field(
        description="Person's email"
    )
```

Then configure the model for structured output.

```python
structured_model = model.with_structured_output(
    ContactInfo
)
```

Invoke:

```python
result = structured_model.invoke(
    "John's email is john@example.com"
)
```

The result is validated against the schema.

---

# Why Structured Output Matters

It enables reliable integration with:

* databases
* APIs
* workflows
* frontend applications
* validation systems
* downstream agents

---

# 12. Tools

A tool gives an LLM the ability to interact with the external world.

Examples:

```text
Calculator
Weather API
Database
Search Engine
File System
Code Interpreter
CRM
Email
Calendar
```

Without tools:

```text
LLM → Text
```

With tools:

```text
LLM
 ↓
Tool Decision
 ↓
External System
 ↓
Tool Result
 ↓
LLM
```

---

# Creating a Tool

```python
from langchain.tools import tool

@tool
def add_numbers(a: int, b: int) -> int:
    """Add two numbers."""
    return a + b
```

The docstring is important because the model can use it to understand the tool.

---

# Tool Schema

The function signature defines the input schema.

```python
@tool
def search_product(
    product_name: str,
    category: str
):
    """Search for a product."""
```

The model can infer:

```json
{
    "product_name": "...",
    "category": "..."
}
```

---

# 13. Tool Calling

Tool calling allows a model to request execution of a function.

Flow:

```text
User
 ↓
LLM
 ↓
Tool Call
 ↓
Tool Execution
 ↓
Tool Result
 ↓
LLM
 ↓
Final Answer
```

Example:

```python
model_with_tools = model.bind_tools([
    add_numbers
])
```

Then:

```python
response = model_with_tools.invoke(
    "What is 10 + 20?"
)
```

The model may produce a tool call rather than directly answering.

---

# Tool Calling vs Function Calling

Modern terminology generally uses:

> **Tool calling**

The model determines:

* which tool to call
* when to call it
* arguments to pass

The application executes the actual function.

---

# 14. Agents

An agent is an LLM-powered system that can decide what actions to take.

A basic chain is deterministic:

```text
Prompt → Model → Output
```

An agent is dynamic:

```text
             ┌→ Tool A
             │
User → Agent ┼→ Tool B
             │
             └→ Tool C
                  ↓
              Observation
                  ↓
                Agent
                  ↓
              Final Answer
```

---

# Agent Components

An agent typically contains:

```text
Model
Tools
Instructions
State
Execution Loop
```

Modern LangChain provides agent abstractions that can manage this loop.

---

# Creating an Agent

A modern LangChain pattern is:

```python
from langchain.agents import create_agent

agent = create_agent(
    model=model,
    tools=[add_numbers]
)
```

Invoke:

```python
result = agent.invoke({
    "messages": [
        {
            "role": "user",
            "content": "What is 20 + 30?"
        }
    ]
})
```

---

# Agent Loop

Conceptually:

```text
        ┌──────────────┐
        │     User     │
        └──────┬───────┘
               ↓
        ┌──────────────┐
        │     Agent    │
        └──────┬───────┘
               ↓
        Need a tool?
          /       \
        Yes        No
         ↓          ↓
      Tool       Answer
         ↓
      Result
         ↓
       Agent
         ↑
         └────────
```

This loop is what makes an agent different from a simple chain.

---

# 15. Agent Execution

An agent may perform several steps.

Example:

```text
User:
"What is the weather in Kolkata and should I carry an umbrella?"
```

Agent:

```text
1. Identify weather requirement
2. Call weather tool
3. Receive weather data
4. Interpret result
5. Generate recommendation
```

This is dynamic execution.

---

# Agent vs Chain

| Feature        | Chain              | Agent           |
| -------------- | ------------------ | --------------- |
| Execution      | Fixed              | Dynamic         |
| Tool selection | Usually predefined | Model decides   |
| Complexity     | Lower              | Higher          |
| Predictability | High               | Lower           |
| Cost           | Usually lower      | Usually higher  |
| Best for       | Pipelines          | Decision-making |

---

# 16. Retrieval

Retrieval means finding relevant information from an external knowledge source.

Instead of asking the LLM to rely only on its training:

```text
User Question
      ↓
Retrieve relevant information
      ↓
LLM
      ↓
Answer
```

Retrieval is the foundation of RAG.

---

# 17. Document Loaders

Document loaders convert external data into LangChain documents.

Common sources:

* PDF
* HTML
* CSV
* JSON
* Markdown
* text
* web pages
* databases
* cloud storage

---

## Document

A document generally contains:

```python
Document(
    page_content="...",
    metadata={
        "source": "...",
        "page": 1
    }
)
```

Metadata is extremely important for:

* citations
* filtering
* debugging
* access control
* source attribution

---

# 18. Text Splitters

Large documents need to be divided into smaller chunks.

Why?

Because LLM context windows are limited and retrieval works better with focused chunks.

```text
Large Document
      ↓
Chunking
      ↓
Chunk 1
Chunk 2
Chunk 3
...
Chunk N
```

---

# Chunk Size

Example:

```text
chunk_size = 1000
```

This means approximately 1000 characters/tokens depending on the splitter.

---

# Chunk Overlap

Example:

```text
chunk_size = 1000
chunk_overlap = 200
```

Overlap helps preserve context across boundaries.

```text
Chunk 1:
AAAAAAAAAAAAAAAA

Chunk 2:
          AAAAAAAABBBBBBBBBB
                  ↑
                overlap
```

---

# Important Consideration

There is no universally correct:

```text
chunk_size
chunk_overlap
```

Optimal values depend on:

* document structure
* question types
* embedding model
* retrieval strategy
* context window
* evaluation results

---

# 19. Embeddings

Embeddings convert text into vectors.

```text
Text
 ↓
Embedding Model
 ↓
[0.12, -0.42, 0.83, ...]
```

Semantically similar text should have nearby vectors.

---

# Semantic Similarity

Example:

```text
"How does a neural network learn?"
```

may be close to:

```text
"Explain neural network training."
```

even though the words differ.

---

# Embedding Pipeline

```text
Document
 ↓
Chunk
 ↓
Embedding Model
 ↓
Vector
 ↓
Vector Database
```

---

# 20. Vector Stores

Vector stores store embeddings and allow similarity search.

Popular options include:

* FAISS
* Chroma
* Pinecone
* Weaviate
* Qdrant
* Milvus
* pgvector

Architecture:

```text
                Vector DB
                   │
        ┌──────────┼──────────┐
        ↓          ↓          ↓
     Vector 1   Vector 2   Vector 3
```

---

# 21. Retrievers

A retriever is an abstraction that returns relevant documents.

```text
Query
 ↓
Retriever
 ↓
Relevant Documents
```

Example:

```python
retriever = vectorstore.as_retriever()

docs = retriever.invoke(
    "What is RAG?"
)
```

---

# Retriever vs Vector Store

A vector store is responsible for storing/searching vectors.

A retriever provides a standardized retrieval interface.

```text
Vector Store
     ↓
Retriever
     ↓
Application
```

---

# 22. RAG

## Retrieval-Augmented Generation

RAG combines:

```text
Retrieval
+
Generation
```

Architecture:

```text
                 ┌───────────────┐
                 │ Knowledge Base│
                 └───────┬───────┘
                         ↓
                    Retriever
                         ↓
User Question ───────→ Context
                         ↓
                       Prompt
                         ↓
                        LLM
                         ↓
                      Answer
```

---

# RAG Pipeline

## Step 1 — Load

```text
PDF / Web / DB / Files
```

↓

## Step 2 — Split

```text
Documents → Chunks
```

↓

## Step 3 — Embed

```text
Chunks → Vectors
```

↓

## Step 4 — Store

```text
Vectors → Vector DB
```

↓

## Step 5 — Retrieve

```text
Question → Relevant Chunks
```

↓

## Step 6 — Generate

```text
Question + Context → LLM
```

---

# Basic RAG Concept

```python
docs = retriever.invoke(question)

context = "\n\n".join(
    doc.page_content
    for doc in docs
)

prompt = f"""
Answer using the following context.

Context:
{context}

Question:
{question}
"""

response = model.invoke(prompt)
```

---

# Advanced RAG

Production RAG often contains:

```text
Query
 ↓
Query Transformation
 ↓
Hybrid Retrieval
 ↓
Metadata Filtering
 ↓
Reranking
 ↓
Context Compression
 ↓
Prompt Construction
 ↓
LLM
 ↓
Citation Validation
```

---

# 23. Conversation History

LLMs are stateless by default.

If you send:

```text
User: My name is Saksham.
```

and later:

```text
User: What is my name?
```

the model does not automatically know the first message unless the application provides that history.

---

# Message History

Conceptually:

```text
Conversation
 ├── HumanMessage
 ├── AIMessage
 ├── HumanMessage
 └── AIMessage
```

The application can send relevant messages back to the model.

---

# 24. Short-Term Memory

Short-term memory means maintaining state during a conversation or workflow.

Example:

```text
Conversation Session
       ↓
Message History
       ↓
Current Agent State
```

In modern agent architectures, state management is especially important.

For complex workflows, **LangGraph** is generally the better abstraction for explicit durable state and long-running workflows.

---

# 25. Long-Term Memory

Long-term memory persists information across sessions.

Example:

```text
Session 1
   ↓
User preferences
   ↓
Memory Store
   ↓
Session 2
   ↓
Retrieve relevant memory
```

Possible storage:

* PostgreSQL
* Redis
* Vector databases
* dedicated memory systems

---

# Memory Types

### Semantic Memory

Facts:

```text
User prefers Python.
```

### Episodic Memory

Past events:

```text
User previously built a RAG application.
```

### Procedural Memory

Instructions or behavior:

```text
Always return answers in Markdown.
```

---

# 26. Callbacks

Callbacks allow applications to observe execution.

Useful for:

* logging
* tracing
* metrics
* debugging
* monitoring
* token tracking

Conceptually:

```text
Chain Start
    ↓
Model Start
    ↓
Tool Start
    ↓
Tool End
    ↓
Model End
    ↓
Chain End
```

Callbacks can observe these events.

---

# 27. Streaming

Instead of waiting for the complete answer:

```text
"LangChain is..."
```

the application receives chunks progressively.

```text
Lang
LangChain
LangChain is
LangChain is a
...
```

Example:

```python
for chunk in chain.stream(
    {"topic": "LangChain"}
):
    print(chunk, end="")
```

Streaming improves perceived latency.

---

# Agent Streaming

Agent applications may stream:

```text
Agent decision
↓
Tool call
↓
Tool result
↓
Agent response
↓
Final answer
```

This is particularly useful for UI applications.

---

# 28. Async Execution

Production applications often need asynchronous execution.

```python
response = await chain.ainvoke(
    input
)
```

Useful when handling:

* web requests
* multiple users
* external APIs
* parallel operations
* streaming

---

# Async Batch

```python
results = await chain.abatch([
    input1,
    input2,
    input3
])
```

---

# 29. Middleware

Middleware provides a mechanism for controlling or extending agent behavior around model/tool execution.

Possible uses include:

* guardrails
* logging
* retries
* dynamic prompts
* human approval
* model fallback
* rate limiting
* security checks
* tool restrictions

Conceptually:

```text
Request
   ↓
Middleware
   ↓
Agent
   ↓
Middleware
   ↓
Response
```

---

# 30. LangChain Agents vs LangGraph

This distinction is extremely important.

## LangChain

Best for:

```text
Models
Prompts
Tools
Structured Output
Retrieval
Simple Agents
Integrations
```

## LangGraph

Best for:

```text
Stateful Workflows
Complex Agents
Multi-Agent Systems
Human-in-the-loop
Durable Execution
Long-running Agents
Explicit Control Flow
```

---

# Relationship

Think of the ecosystem like:

```text
               AI Application
                     │
          ┌──────────┴──────────┐
          │                     │
      LangChain              LangGraph
          │                     │
  Components & Agents      Orchestration
          │                     │
          └──────────┬──────────┘
                     ↓
                  LangSmith
```

LangChain provides many building blocks.

LangGraph provides explicit orchestration for more complex agentic systems.

---

# When to use LangChain?

Use LangChain when you need:

* model integrations
* prompts
* structured output
* tools
* retrievers
* RAG
* simple agents
* reusable AI components

---

# When to use LangGraph?

Use LangGraph when you need:

* branching workflows
* loops
* state machines
* multi-agent systems
* human approval
* persistent execution
* complex agent orchestration

---

# 31. LangSmith

LangSmith is an observability and evaluation platform for LLM applications.

It can help developers inspect:

```text
User Request
 ↓
Prompt
 ↓
LLM
 ↓
Tool
 ↓
Retriever
 ↓
LLM
 ↓
Final Answer
```

---

# Why Observability Matters

Without tracing:

```text
Agent failed.
```

You don't know why.

With tracing:

```text
Agent
 ├── Prompt
 ├── Model Call
 ├── Tool Call
 ├── Tool Result
 ├── Retrieval
 ├── Second Model Call
 └── Final Output
```

You can identify the failure.

---

# 32. Evaluation

AI systems cannot be evaluated only with:

```text
"It seems good."
```

You need measurable evaluation.

---

# RAG Evaluation

Important metrics include:

### Retrieval Recall

Did retrieval find the relevant information?

### Context Precision

Was retrieved information actually relevant?

### Faithfulness

Is the answer supported by retrieved context?

### Answer Relevance

Does the answer actually answer the question?

---

# Agent Evaluation

Possible metrics:

```text
Task Success Rate
Tool Selection Accuracy
Tool Argument Accuracy
Number of Steps
Latency
Token Usage
Cost
Failure Rate
Hallucination Rate
```

---

# Evaluation Dataset

Example:

```json
{
    "question": "What is the refund policy?",
    "expected_answer": "Refunds are available within 30 days."
}
```

Run the agent against many examples.

```text
Dataset
   ↓
Agent
   ↓
Outputs
   ↓
Evaluator
   ↓
Metrics
```

---

# 33. Production Architecture

A production LangChain application might look like:

```text
                         ┌───────────────┐
                         │    Frontend   │
                         └───────┬───────┘
                                 ↓
                         ┌───────────────┐
                         │   API Layer   │
                         └───────┬───────┘
                                 ↓
                    ┌────────────────────────┐
                    │    LangChain Agent     │
                    └───────────┬────────────┘
                                │
              ┌─────────────────┼──────────────────┐
              ↓                 ↓                  ↓
            Tools            Retriever           Model
              ↓                 ↓                  ↓
          External APIs      Vector DB          LLM Provider
                                │
                                ↓
                         Knowledge Base
```

Supporting infrastructure:

```text
PostgreSQL
Redis
Vector DB
Object Storage
Observability
Evaluation
Authentication
Rate Limiting
```

---

# Production Principles

A production AI system should include:

* authentication
* authorization
* input validation
* structured outputs
* retries
* timeouts
* rate limits
* logging
* tracing
* evaluation
* monitoring
* fallback models
* caching
* cost monitoring
* prompt versioning

---

# 34. Common Design Patterns

## Pattern 1 — Simple Chain

```text
Input
 ↓
Prompt
 ↓
LLM
 ↓
Output
```

Use for deterministic tasks.

---

# Pattern 2 — Structured Extraction

```text
Input
 ↓
LLM
 ↓
Pydantic Schema
 ↓
Validated Object
```

Use for information extraction.

---

# Pattern 3 — Tool Calling

```text
User
 ↓
LLM
 ↓
Tool
 ↓
Result
 ↓
LLM
```

Use when the model needs external capabilities.

---

# Pattern 4 — RAG

```text
Question
 ↓
Retriever
 ↓
Context
 ↓
LLM
 ↓
Answer
```

Use for private/domain knowledge.

---

# Pattern 5 — Agent

```text
User
 ↓
Agent
 ↓
Decision
 ↓
Tool
 ↓
Observation
 ↓
Decision
 ↓
Answer
```

Use when the next action cannot be predetermined.

---

# Pattern 6 — Router

```text
                  ┌→ Technical Agent
User → Router ────┼→ Financial Agent
                  └→ Support Agent
```

The router determines which specialized workflow handles the request.

---

# Pattern 7 — Multi-Agent System

```text
                Supervisor
                    │
        ┌───────────┼───────────┐
        ↓           ↓           ↓
    Researcher   Coder      Reviewer
        │           │           │
        └───────────┼───────────┘
                    ↓
                  Output
```

For complex systems, LangGraph is often better suited for explicit orchestration.

---

# 35. Common Mistakes

## Mistake 1 — Using an Agent for Everything

Not every problem needs an agent.

If the workflow is:

```text
Input → Transform → Output
```

use a chain.

---

# Mistake 2 — Excessive Tools

Giving an agent 50 tools can make tool selection difficult.

Prefer:

```text
Small
Relevant
Well-described
Well-scoped
```

tools.

---

# Mistake 3 — Poor Tool Descriptions

Bad:

```python
@tool
def search(x):
    ...
```

Better:

```python
@tool
def search_customer_orders(
    customer_id: str
):
    """Retrieve all orders associated with a customer ID."""
```

---

# Mistake 4 — Ignoring Structured Output

Parsing free-form text manually is fragile.

Prefer:

```text
Pydantic
JSON Schema
Native structured output
```

when possible.

---

# Mistake 5 — No Evaluation

A demo working once does not mean the system is reliable.

Build an evaluation dataset early.

---

# Mistake 6 — No Observability

If an agent fails:

```text
"What happened?"
```

Tracing should answer that question.

---

# Mistake 7 — Huge Prompts

More instructions do not automatically produce better results.

Prefer:

```text
Clear
Specific
Minimal
Tested
Versioned
```

prompts.

---

# Mistake 8 — Treating RAG as Just Vector Search

Production RAG often requires:

```text
Chunking
+
Metadata
+
Retrieval
+
Reranking
+
Context selection
+
Citation
+
Evaluation
```

---

# 36. Best Practices

## 1. Keep Components Modular

Separate:

```text
models/
prompts/
tools/
retrievers/
agents/
evaluation/
```

---

## 2. Type Everything

Use:

```python
Pydantic
TypedDict
Python type hints
```

where appropriate.

---

## 3. Keep Tool Interfaces Small

Tools should perform one meaningful operation.

---

## 4. Validate External Inputs

Never blindly trust:

```text
User input
Tool output
Retrieved documents
Model output
```

---

## 5. Add Timeouts

External APIs can fail or hang.

---

## 6. Implement Retries Carefully

Retries should be:

```text
bounded
exponential
selective
```

Do not blindly retry every failure.

---

## 7. Track Cost

Track:

```text
Input tokens
Output tokens
Total tokens
Latency
Model
Number of calls
```

---

## 8. Evaluate Before Optimizing

Use data to determine whether changes actually improve:

```text
accuracy
latency
cost
reliability
```

---

# 37. LangChain + Groq

For free/low-cost experimentation, Groq can be used as a model provider through LangChain.

Install:

```bash
pip install langchain-groq
```

Set the API key:

```python
import os

os.environ["GROQ_API_KEY"] = os.getenv(
    "GROQ_API_KEY"
)
```

Create the model:

```python
from langchain_groq import ChatGroq

model = ChatGroq(
    model="YOUR_GROQ_MODEL"
)
```

Then use it like other LangChain chat models:

```python
response = model.invoke(
    "Explain Agentic AI."
)

print(response.content)
```

---

# Groq + Structured Output

```python
structured_model = model.with_structured_output(
    ContactInfo
)

result = structured_model.invoke(
    "John's email is john@example.com"
)
```

---

# Groq + Tools

```python
model_with_tools = model.bind_tools([
    add_numbers
])
```

The exact capabilities depend on the selected Groq model.

Always verify provider/model support for:

* tool calling
* structured output
* streaming
* JSON mode
* reasoning capabilities

---

# 38. Mini Projects

## Project 1 — AI Text Summarizer

Learn:

```text
Prompt
Model
Runnable
Output Parser
```

Architecture:

```text
Text
 ↓
Prompt
 ↓
LLM
 ↓
Summary
```

---

# Project 2 — Structured Resume Parser

Learn:

```text
Pydantic
Structured Output
LLM
```

Input:

```text
Resume PDF
```

Output:

```json
{
    "name": "...",
    "skills": [],
    "experience": [],
    "education": []
}
```

---

# Project 3 — Document RAG

Learn:

```text
Document Loader
Text Splitter
Embeddings
Vector Store
Retriever
LLM
```

Architecture:

```text
PDF
 ↓
Chunks
 ↓
Embeddings
 ↓
Vector DB
 ↓
Retriever
 ↓
LLM
```

---

# Project 4 — Tool-Using Assistant

Tools:

```text
Calculator
Weather
Search
Database
```

Learn:

```text
Tools
Tool Calling
Agents
```

---

# Project 5 — Research Agent

Architecture:

```text
User
 ↓
Research Agent
 ↓
Search Tool
 ↓
Retrieve Sources
 ↓
Summarize
 ↓
Verify
 ↓
Final Report
```

---

# Project 6 — RAG + Agent

Combine:

```text
Agent
+
Tools
+
Retriever
+
LLM
```

Example:

```text
User
 ↓
Agent
 ├── Search Tool
 ├── Database Tool
 └── Knowledge Base Retriever
          ↓
        Answer
```

---

# Project 7 — Production AI Support Agent

Include:

```text
Authentication
RAG
Tools
Agent
Memory
Structured Output
Tracing
Evaluation
Guardrails
```

This is much more portfolio-worthy than a simple chatbot.

---

# 39. Important APIs

## Model

```python
model.invoke()
model.stream()
model.ainvoke()
model.astream()
```

---

## Runnable

```python
runnable.invoke()
runnable.batch()
runnable.stream()
runnable.ainvoke()
runnable.abatch()
runnable.astream()
```

---

## Prompt

```python
prompt.invoke()
```

---

## Retriever

```python
retriever.invoke()
```

---

## Agent

```python
agent.invoke()
agent.stream()
agent.ainvoke()
agent.astream()
```

---

## Structured Output

```python
model.with_structured_output(...)
```

---

## Tools

```python
@tool
def my_tool(...):
    ...
```

---

## Tool Binding

```python
model.bind_tools([...])
```

---

# 40. Learning Roadmap

Recommended progression:

```text
                    LangChain
                        │
                        ↓
                 Basic LLM Calls
                        │
                        ↓
                     Prompts
                        │
                        ↓
                     Messages
                        │
                        ↓
                    Runnables
                        │
                        ↓
                       LCEL
                        │
                        ↓
               Structured Output
                        │
                        ↓
                      Tools
                        │
                        ↓
                  Tool Calling
                        │
                        ↓
                      Agents
                        │
              ┌─────────┴─────────┐
              ↓                   ↓
             RAG              LangGraph
              ↓                   ↓
         Advanced RAG       Agent Workflows
              │                   │
              └─────────┬─────────┘
                        ↓
                   LangSmith
                        ↓
                    Evaluation
                        ↓
                 Production AI
```

---

# 🧠 Core Concepts to Remember

If you remember only the following, remember these:

### 1. Model

```text
Produces intelligence
```

### 2. Prompt

```text
Defines what the model should do
```

### 3. Runnable

```text
Composable execution unit
```

### 4. Structured Output

```text
Makes model responses machine-readable
```

### 5. Tool

```text
Gives the model external capabilities
```

### 6. Agent

```text
Lets the model decide which actions to take
```

### 7. Retriever

```text
Finds relevant information
```

### 8. RAG

```text
Retrieval + Generation
```

### 9. Memory / State

```text
Maintains information across interactions or workflow steps
```

### 10. LangGraph

```text
Orchestrates complex stateful agent workflows
```

### 11. LangSmith

```text
Observability + evaluation
```

---

# 🔥 LangChain Mental Model

The simplest mental model is:

```text
                    ┌──────────────┐
                    │     MODEL    │
                    └──────┬───────┘
                           │
              ┌────────────┼────────────┐
              ↓            ↓            ↓
           Prompts       Tools      Retrieval
              │            │            │
              └────────────┼────────────┘
                           ↓
                       Runnables
                           ↓
                         Agent
                           ↓
                      Application
```

---

# 🆚 Chain vs RAG vs Agent vs LangGraph

| System      | Main Idea                             | Control Flow          |
| ----------- | ------------------------------------- | --------------------- |
| Chain       | Fixed pipeline                        | Deterministic         |
| RAG         | Retrieve knowledge                    | Mostly deterministic  |
| Agent       | Model chooses actions                 | Dynamic               |
| LangGraph   | Explicit agent/workflow orchestration | Stateful + controlled |
| Multi-Agent | Multiple specialized agents           | Collaborative         |

---

# 🏗️ Recommended Agentic AI Stack

For a serious Agentic AI project:

```text
Frontend
│
├── Next.js
└── React
        │
        ↓
Backend
│
├── FastAPI
├── LangChain
└── LangGraph
        │
        ├── LLM
        │
        ├── Tools
        │
        ├── RAG
        │   ├── Embeddings
        │   ├── Vector DB
        │   └── Retriever
        │
        └── State
             │
             ↓
        PostgreSQL / Redis
             │
             ↓
        Observability
             │
             ├── LangSmith
             ├── Prometheus
             └── Grafana
```

---

# 🎯 What LangChain Should Mean in Your Agentic AI Journey

LangChain should **not** be treated as:

> "A library that calls ChatGPT."

Instead, understand it as a toolkit for constructing:

```text
LLM Applications
       ↓
Composable Components
       ↓
Tools + Retrieval + Structured Data
       ↓
Agents
       ↓
Stateful Agentic Workflows
```

The important engineering skill is not memorizing LangChain classes.

The important skill is knowing:

> **When to use a model, chain, retriever, tool, agent, or workflow — and how to compose them into a reliable production system.**

---

# 📌 Final Cheat Sheet

```text
LangChain
│
├── Models
│   ├── Chat Models
│   └── LLMs
│
├── Messages
│   ├── Human
│   ├── AI
│   ├── System
│   └── Tool
│
├── Prompts
│   ├── PromptTemplate
│   └── ChatPromptTemplate
│
├── Runnables
│   ├── invoke
│   ├── batch
│   ├── stream
│   └── async
│
├── Structured Output
│   └── Pydantic
│
├── Tools
│   └── @tool
│
├── Tool Calling
│   └── bind_tools
│
├── Agents
│   └── create_agent
│
├── Retrieval
│   ├── Loaders
│   ├── Splitters
│   ├── Embeddings
│   ├── Vector Stores
│   └── Retrievers
│
├── RAG
│   ├── Retrieve
│   ├── Context
│   └── Generate
│
├── State / Memory
│
├── Middleware
│
├── Streaming
│
├── Async
│
├── Observability
│   └── LangSmith
│
└── Advanced Orchestration
    └── LangGraph
```

---

# 🚀 Final Takeaway

LangChain provides the **building blocks** for modern LLM applications.

A progression from simple to advanced looks like:

```text
LLM
 ↓
Prompt
 ↓
Chain
 ↓
Structured Output
 ↓
Tool
 ↓
Tool Calling
 ↓
Agent
 ↓
RAG + Agent
 ↓
Stateful Agent
 ↓
LangGraph Workflow
 ↓
Multi-Agent System
 ↓
Evaluated + Observable Production AI
```

For an **Agentic AI engineering portfolio**, the goal should therefore be to move beyond:

```python
model.invoke("Hello")
```

and progressively build systems demonstrating:

```text
✓ Structured outputs
✓ Tool calling
✓ RAG
✓ Agents
✓ State management
✓ LangGraph orchestration
✓ Human-in-the-loop
✓ Evaluation
✓ Observability
✓ Production deployment
```

That progression demonstrates actual **AI engineering ability**, rather than simply knowing how to call an LLM API.