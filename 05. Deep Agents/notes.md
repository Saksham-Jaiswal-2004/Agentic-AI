# Deep Agents

## 1. What Are Deep Agents?

A **Deep Agent** is a long-running, highly capable AI agent designed to handle **complex, multi-step tasks** by combining:

* Planning
* Tool usage
* Persistent filesystem/workspace
* Context management
* Memory
* Subagents
* Skills
* Long-running execution
* Human-in-the-loop controls
* Code execution
* Task decomposition

The key idea is:

> **A Deep Agent is not just an LLM with tools; it is an LLM operating inside a structured execution environment.**

Deep Agents are particularly useful for tasks such as:

* Large-scale research
* Software development
* Data analysis
* Complex document processing
* Multi-step investigation
* Long-running workflows
* Autonomous project execution
* Tasks requiring many files and intermediate artifacts

The modern Deep Agents implementation from LangChain is built on top of the LangChain/LangGraph agent infrastructure and provides a preassembled harness containing capabilities such as planning, filesystem tools, subagents, memory, summarization, and prompt caching.

---

# 2. Core Idea

A basic agent can be thought of as:

```text
LLM
 ↓
Tool
 ↓
Result
 ↓
LLM
 ↓
Tool
 ↓
Result
```

A Deep Agent expands this into:

```text
                    ┌─────────────────────┐
                    │    Deep Agent       │
                    │                     │
                    │      LLM            │
                    │       │             │
                    │   Planning          │
                    │       │             │
                    │   Task Management   │
                    │       │             │
                    │   Context Mgmt      │
                    └───────┬─────────────┘
                            │
          ┌─────────────────┼──────────────────┐
          │                 │                  │
          ▼                 ▼                  ▼
      Tools            Filesystem          Subagents
          │                 │                  │
          ▼                 ▼                  ▼
       APIs            Workspace          Specialists
                            │
                            ▼
                         Memory
```

The important difference is that the Deep Agent has an **environment in which it can work**.

---

# 3. Deep Agent vs Basic Agent

| Feature              | Basic Agent     | Deep Agent          |
| -------------------- | --------------- | ------------------- |
| LLM                  | Yes             | Yes                 |
| Tools                | Yes             | Yes                 |
| Tool calling         | Yes             | Yes                 |
| Planning             | Basic/optional  | Built-in capability |
| Filesystem           | Usually absent  | Core capability     |
| Persistent workspace | Usually absent  | Supported           |
| Subagents            | Optional/manual | Built-in capability |
| Skills               | Usually manual  | Built-in capability |
| Long-running tasks   | Limited         | Designed for it     |
| Context management   | Basic           | Advanced            |
| Summarization        | Manual/optional | Built-in            |
| Memory               | Optional        | Integrated          |
| Task decomposition   | Limited         | Strong              |
| Code execution       | Optional        | Common              |
| Human approval       | Optional        | Supported           |
| Complex research     | Possible        | Strong use case     |
| Large projects       | Difficult       | Designed for them   |

---

# 4. Deep Agent Architecture

A Deep Agent can be viewed as several layers.

```text
┌───────────────────────────────────────────┐
│               USER / TASK                 │
└─────────────────────┬─────────────────────┘
                      │
                      ▼
┌───────────────────────────────────────────┐
│              DEEP AGENT                   │
│                                           │
│  ┌─────────────┐    ┌─────────────────┐  │
│  │    Model    │    │    Planning     │  │
│  └─────────────┘    └─────────────────┘  │
│                                           │
│  ┌─────────────┐    ┌─────────────────┐  │
│  │   Context   │    │  Task Manager   │  │
│  │ Management  │    │                 │  │
│  └─────────────┘    └─────────────────┘  │
│                                           │
│  ┌─────────────┐    ┌─────────────────┐  │
│  │ Filesystem  │    │    Memory       │  │
│  └─────────────┘    └─────────────────┘  │
│                                           │
│  ┌─────────────┐    ┌─────────────────┐  │
│  │   Skills    │    │   Subagents     │  │
│  └─────────────┘    └─────────────────┘  │
└─────────────────────┬─────────────────────┘
                      │
                      ▼
              ┌──────────────┐
              │ Tools / APIs  │
              └──────────────┘
```

---

# 5. Main Components

A Deep Agent typically contains:

1. **Model**
2. **Prompt / Instructions**
3. **Planning system**
4. **Todo/task management**
5. **Tools**
6. **Filesystem**
7. **Memory**
8. **Skills**
9. **Subagents**
10. **Context management**
11. **Summarization**
12. **Execution environment**
13. **Human-in-the-loop**
14. **Persistence**
15. **Observability**

---

# 6. Model

The model is responsible for reasoning and deciding what actions should happen next.

Example:

```text
User:
"Analyze this software project and fix all authentication bugs."

        ↓

LLM

        ↓

Plan:
1. Inspect repository
2. Understand authentication
3. Find bugs
4. Modify code
5. Run tests
6. Fix failures
7. Verify final implementation
```

The model therefore acts as the **decision-making component**.

A Deep Agent does not require a single specific model. The underlying framework supports different model providers.

---

# 7. Instructions

A Deep Agent needs persistent instructions describing:

* Its role
* Rules
* Constraints
* Working methodology
* Available resources
* Safety requirements
* Output requirements

A common Deep Agents convention is:

```text
AGENTS.md
```

For example:

```text
AGENTS.md

# Agent Instructions

You are a senior software engineer.

Before modifying code:

1. Inspect the repository.
2. Understand the existing architecture.
3. Create a plan.
4. Make minimal changes.
5. Run tests.
6. Verify the result.
```

`AGENTS.md` can function as persistent agent instructions in the Deep Agents ecosystem.

---

# 8. Planning

Planning is one of the most important characteristics of Deep Agents.

Instead of immediately attempting the final answer, the agent can create a plan.

Example:

```text
Goal:
Build a production-ready authentication system.

Plan:

[ ] Analyze requirements
[ ] Design authentication architecture
[ ] Design database schema
[ ] Implement registration
[ ] Implement login
[ ] Implement sessions
[ ] Implement refresh tokens
[ ] Implement authorization
[ ] Write tests
[ ] Run tests
[ ] Fix failures
[ ] Final verification
```

The agent then works through these tasks.

---

# 9. Todo / Task Management

A Deep Agent can maintain a structured task list.

Example:

```text
TODO

1. Research authentication architecture
2. Inspect current implementation
3. Identify security problems
4. Implement fixes
5. Run tests
6. Review implementation
```

The task list provides:

* Progress tracking
* Decomposition
* State
* Recovery
* Better focus
* Reduced cognitive load

Instead of thinking about the entire problem at once, the agent focuses on the current task.

---

# 10. Task Decomposition

Complex tasks are broken into smaller tasks.

For example:

```text
"Build an e-commerce application"

              ↓

        Main Objective
              │
      ┌───────┼────────┐
      ↓       ↓        ↓
   Backend  Frontend  Database
      │       │        │
      ↓       ↓        ↓
   Auth     UI       Schema
   API      Pages    Indexes
   Orders   Forms    Relations
   Payment  State    Migration
```

This is critical for Deep Agents because large tasks cannot reliably fit into a single reasoning step.

---

# 11. Filesystem

One of the defining characteristics of Deep Agents is the ability to work with a **filesystem-like environment**.

The agent can perform operations such as:

```text
ls
read
write
edit
search
```

Conceptually:

```text
/project
│
├── README.md
├── package.json
├── src/
│   ├── auth/
│   ├── users/
│   └── payments/
│
├── tests/
│
└── docs/
```

The agent can inspect and modify these files during execution.

---

# 12. Why Filesystem Matters

Without a filesystem:

```text
Task
 ↓
LLM
 ↓
Huge context
 ↓
Answer
```

With a filesystem:

```text
Task
 ↓
LLM
 ↓
Filesystem
 ↓
Read relevant file
 ↓
Modify file
 ↓
Save result
 ↓
Continue
```

The filesystem acts as **external working memory**.

This means the agent doesn't need to keep every intermediate result inside the model's context.

---

# 13. Filesystem as Working Memory

Suppose an agent performs a large research project.

It can create:

```text
/research
    ├── plan.md
    ├── source-1.md
    ├── source-2.md
    ├── source-3.md
    ├── analysis.md
    └── final-report.md
```

Instead of keeping all of this in the context window:

```text
LLM Context
    ↓
Small relevant portion
    ↓
Filesystem
```

This makes long-running tasks much more practical.

---

# 14. Backend Abstraction

Deep Agents separates the **filesystem interface** from the actual storage implementation.

Conceptually:

```text
Agent
  │
  ▼
Backend Interface
  │
  ├── Local Filesystem
  ├── Agent State
  └── Persistent Store
```

This means the agent can interact with files using the same interface even when the underlying storage changes.

Deep Agents provides backend abstractions including `FilesystemBackend`, `StoreBackend`, and `StateBackend`, each having different persistence scopes.

---

# 15. Deep Agent Backends

### FilesystemBackend

Uses the local/container filesystem.

```text
Agent
 ↓
FilesystemBackend
 ↓
Container filesystem
```

Useful for:

* Static project files
* Code repositories
* Read-only knowledge
* Temporary workspace

Data generally does not survive redeployment unless included in the image or otherwise persisted.

---

### StateBackend

Stores files in agent/thread state.

```text
Agent
 ↓
StateBackend
 ↓
Thread State
```

Useful for:

* Per-thread workspace
* Temporary artifacts
* Conversation-specific files

---

### StoreBackend

Uses persistent storage.

```text
Agent
 ↓
StoreBackend
 ↓
Persistent Store
```

Useful for:

* Long-term memory
* Persistent user information
* Cross-session information

---

# 16. Backend Comparison

| Backend           | Scope           |                 Persistent? | Typical Use          |
| ----------------- | --------------- | --------------------------: | -------------------- |
| FilesystemBackend | Container       |          No across redeploy | Project/static files |
| StateBackend      | Thread          | Yes within persisted thread | Working state        |
| StoreBackend      | Deployment/user |                         Yes | Long-term data       |

The exact persistence behavior depends on the deployment and storage configuration.

---

# 17. Memory

Deep Agents can use multiple forms of memory.

A useful conceptual division is:

```text
Memory
│
├── Short-term / Thread State
│
├── Long-term Memory
│
└── Filesystem Memory
```

---

# 18. Thread Memory

Thread memory contains information relevant to the current execution/conversation.

Example:

```text
Thread 123

User:
Build authentication.

Agent:
Created auth module.

User:
Now add refresh tokens.

Agent:
Understands the previous implementation.
```

The agent doesn't need to rediscover everything from scratch.

---

# 19. Long-Term Memory

Long-term memory stores information across sessions.

Example:

```text
User Profile

name: Alex
preferred_language: Python
coding_style: TypeScript
project: SecureExam
```

A later conversation can use this information.

The Store is designed for persistent information that survives across sessions/containers.

---

# 20. Filesystem Memory

Information can also be stored in files.

Example:

```text
/memories
    ├── AGENTS.md
    ├── preferences.md
    ├── project-context.md
    └── decisions.md
```

The agent can read these files when needed.

This is particularly useful for:

* Project knowledge
* Instructions
* Research notes
* Decisions
* User preferences
* Generated artifacts

---

# 21. Skills

A **Skill** is a reusable set of instructions and knowledge for a specific capability.

Example:

```text
skills/
│
├── python/
│   └── SKILL.md
│
├── security/
│   └── SKILL.md
│
├── research/
│   └── SKILL.md
│
└── database/
    └── SKILL.md
```

Each skill teaches the agent how to perform a particular type of work.

---

# 22. Why Skills Exist

Instead of putting everything inside one massive system prompt:

```text
SYSTEM PROMPT
----------------
Python rules
Security rules
Database rules
Research rules
React rules
DevOps rules
...
```

Use:

```text
AGENTS.md
     +
Skills
```

The agent can load detailed information when necessary.

This is called **progressive disclosure**.

Deep Agents' Skills middleware exposes skill descriptions initially and lets the agent read the detailed skill content when needed, reducing unnecessary context consumption.

---

# 23. Progressive Disclosure

The process looks like:

```text
Agent starts
     │
     ▼
Sees available skills
     │
     ▼
"Security skill exists"
     │
     ▼
Needs security knowledge
     │
     ▼
Reads SKILL.md
     │
     ▼
Uses instructions
```

Instead of:

```text
Load EVERYTHING
     ↓
Huge Context
     ↓
High Token Cost
```

---

# 24. Skill Structure

Example:

```text
skills/
└── security/
    ├── SKILL.md
    ├── checklist.md
    └── examples.md
```

`SKILL.md`:

```markdown
# Security Skill

When reviewing authentication:

1. Check password hashing.
2. Check session expiration.
3. Check token rotation.
4. Check authorization.
5. Check sensitive logging.
6. Check rate limiting.
```

The agent reads this skill when performing security-related work.

---

# 25. Subagents

A Deep Agent can delegate tasks to **specialized subagents**.

Example:

```text
                    Main Agent
                       │
          ┌────────────┼────────────┐
          ▼            ▼            ▼
      Researcher    Coder       Reviewer
          │            │            │
          ▼            ▼            ▼
       Research      Code        Review
```

The main agent acts as an **orchestrator**.

---

# 26. Why Subagents?

Large tasks often contain independent responsibilities.

Example:

```text
"Analyze this startup"

        ↓

Main Agent
   │
   ├── Market Research Agent
   ├── Competitor Agent
   ├── Financial Agent
   ├── Technical Agent
   └── Report Agent
```

Each subagent focuses on a narrower problem.

---

# 27. Subagent Isolation

Subagents can have their own:

* Instructions
* Tools
* Model
* Skills
* State

For example:

```text
Research Agent
    Model: powerful reasoning model
    Tools: web search
    Skills: research

Coding Agent
    Model: coding model
    Tools: filesystem + shell
    Skills: software engineering
```

Deep Agents' subagent mechanism supports specialized child agents with their own configuration and isolated execution context.

---

# 28. Main Agent vs Subagent

| Main Agent                  | Subagent                  |
| --------------------------- | ------------------------- |
| Orchestrates                | Performs specialized task |
| Maintains overall objective | Focuses on assigned task  |
| Delegates                   | Executes                  |
| Integrates results          | Returns results           |
| Manages workflow            | Usually narrower scope    |

---

# 29. Example Delegation

User:

```text
"Analyze and improve my backend."
```

Main Agent:

```text
1. Ask researcher to inspect architecture
2. Ask security agent to audit authentication
3. Ask coding agent to implement fixes
4. Ask testing agent to run tests
5. Review all results
```

Execution:

```text
                    Main Agent
                        │
        ┌───────────────┼──────────────┐
        ▼               ▼              ▼
    Researcher       Security       Tester
        │               │              │
        ▼               ▼              ▼
    Architecture     Audit          Tests
        │               │              │
        └───────────────┼──────────────┘
                        ▼
                   Main Agent
                        │
                        ▼
                  Final Result
```

---

# 30. Tools

Deep Agents can use tools to interact with external systems.

Examples:

```text
Web Search
Database
Filesystem
Shell
Python
Git
APIs
Browser
Cloud services
MCP tools
```

A tool generally follows:

```text
Agent
 ↓
Tool Call
 ↓
External System
 ↓
Tool Result
 ↓
Agent
```

---

# 31. Tool Selection

The agent decides:

```text
What do I need?
       │
       ▼
Which tool can provide it?
       │
       ▼
Call tool
       │
       ▼
Interpret result
       │
       ▼
Continue
```

For example:

```text
Need to inspect source code
        ↓
Filesystem

Need current information
        ↓
Web Search

Need database information
        ↓
Database Tool

Need computation
        ↓
Python
```

---

# 32. Execution Environment

A Deep Agent becomes much more powerful when it has an execution environment.

Conceptually:

```text
┌───────────────────────────────┐
│       Agent Environment       │
│                               │
│ Filesystem                    │
│ Shell                         │
│ Code Execution                │
│ Tools                         │
│ APIs                          │
│ Memory                        │
└───────────────────────────────┘
```

This allows the agent to **do work**, not merely describe what should be done.

---

# 33. Code Execution

For programming tasks, a Deep Agent may:

```text
Read code
 ↓
Modify code
 ↓
Run code
 ↓
Observe output
 ↓
Identify error
 ↓
Modify code
 ↓
Run tests
 ↓
Repeat
```

This creates a feedback loop:

```text
PLAN
 ↓
ACT
 ↓
OBSERVE
 ↓
CORRECT
 ↓
ACT
 ↓
OBSERVE
```

---

# 34. Context Management

Long tasks can exceed the model's context window.

Deep Agents therefore need mechanisms for controlling context.

Important techniques include:

* Summarization
* External files
* Selective retrieval
* Skills
* Subagents
* Prompt caching
* Persistent state

---

# 35. Context Window Problem

Imagine:

```text
Turn 1     5K tokens
Turn 2     8K
Turn 3     10K
Turn 4     15K
Turn 5     20K
...
```

Eventually:

```text
Context
████████████████████████████
             ↑
        Too large
```

A Deep Agent must avoid putting every historical detail into every model call.

---

# 36. Summarization

The agent can compress old context.

Example:

Before:

```text
100 pages of conversation
```

After:

```text
Summary:
- Authentication implemented
- Refresh token rotation completed
- Device verification pending
- Tests failing in session module
```

The agent can continue from the summary.

Deep Agents provides summarization as part of its preassembled capabilities.

---

# 37. Externalizing Context

Instead of keeping everything in context:

```text
LLM Context
   │
   ├── Important instructions
   ├── Current task
   └── Relevant information
```

Everything else can be stored externally:

```text
Filesystem
    ├── research.md
    ├── analysis.md
    ├── logs.txt
    └── results.json
```

The agent retrieves only what it needs.

---

# 38. Prompt Caching

Repeated instructions and context can sometimes be cached.

Example:

```text
AGENTS.md
Large instructions
Stable project information
```

Rather than repeatedly processing identical content from scratch, prompt caching can improve efficiency where supported.

Deep Agents includes prompt caching among the capabilities assembled by its prebuilt harness.

---

# 39. Deep Agent Execution Loop

A simplified Deep Agent loop:

```text
                 ┌──────────────┐
                 │     Task     │
                 └──────┬───────┘
                        ▼
                 ┌──────────────┐
                 │    Plan      │
                 └──────┬───────┘
                        ▼
                 ┌──────────────┐
                 │ Choose Task  │
                 └──────┬───────┘
                        ▼
                 ┌──────────────┐
                 │ Choose Tool  │
                 └──────┬───────┘
                        ▼
                 ┌──────────────┐
                 │ Execute Tool │
                 └──────┬───────┘
                        ▼
                 ┌──────────────┐
                 │   Observe    │
                 └──────┬───────┘
                        ▼
                 ┌──────────────┐
                 │ Update State │
                 └──────┬───────┘
                        ▼
                 ┌──────────────┐
                 │ More Tasks?  │
                 └──────┬───────┘
                    Yes  │  No
                         │
              ┌──────────┘
              ▼
        Back to execution

                         No
                         │
                         ▼
                   Final Result
```

---

# 40. Deep Agent State

State can contain:

```text
State
│
├── Current task
├── Todo list
├── Conversation
├── Tool results
├── Files
├── Subagent results
├── User information
└── Intermediate artifacts
```

This allows execution to continue across multiple steps.

---

# 41. Durable Execution

Long-running tasks can fail.

Examples:

* API timeout
* Model failure
* Tool failure
* Network error
* Server restart
* Human approval delay

A robust Deep Agent should be able to resume rather than restart everything.

This is one reason Deep Agents are built on infrastructure such as LangGraph, which provides persistence and durable execution capabilities.

---

# 42. Human-in-the-Loop

Some actions should require human approval.

For example:

```text
Agent:
I want to delete production database.

        ↓

Human Approval

   Approve ❌
   Reject  ✅
```

Typical approval points:

* Sending emails
* Deleting data
* Deploying software
* Making financial transactions
* Modifying production systems
* Publishing content

---

# 43. Human-in-the-Loop Architecture

```text
Agent
  │
  ▼
Sensitive Action?
  │
 ┌┴──────────────┐
 │               │
No              Yes
 │               │
 ▼               ▼
Execute       Ask Human
                 │
          ┌──────┴──────┐
          ▼             ▼
       Approve        Reject
          │             │
          ▼             ▼
       Execute        Stop
```

---

# 44. Skills vs Tools vs Subagents

These concepts are often confused.

| Concept    | Purpose                                     |
| ---------- | ------------------------------------------- |
| Tool       | Performs an action                          |
| Skill      | Teaches how to perform a class of tasks     |
| Subagent   | Performs a delegated task                   |
| Filesystem | Stores/retrieves working information        |
| Memory     | Stores information across execution/session |
| Planner    | Determines what should happen               |
| Model      | Makes decisions and generates reasoning     |

Example:

```text
Security Skill
      │
      ▼
Security Subagent
      │
      ▼
Uses:
- filesystem tool
- database tool
- scanner tool
```

---

# 45. Deep Agent Project Structure

A typical Deep Agents CLI-style project can look like:

```text
my-agent/
│
├── deepagents.toml
├── AGENTS.md
├── .env
│
├── mcp.json
│
├── skills/
│   ├── research/
│   │   └── SKILL.md
│   │
│   └── coding/
│       └── SKILL.md
│
├── subagents/
│   ├── researcher/
│   │   ├── deepagents.toml
│   │   └── AGENTS.md
│   │
│   └── reviewer/
│       ├── deepagents.toml
│       └── AGENTS.md
│
└── user/
```

This convention is used by the Deep Agents CLI for project discovery.

---

# 46. `AGENTS.md`

This is the main instruction file.

Example:

```markdown
# Agent Instructions

You are a senior research assistant.

## Rules

1. Break complex tasks into smaller tasks.
2. Use the filesystem for intermediate work.
3. Verify important information.
4. Delegate specialized work when useful.
5. Do not modify production systems without approval.
6. Keep a record of important decisions.
```

---

# 47. `skills/`

Example:

```text
skills/
├── research/
│   ├── SKILL.md
│   └── sources.md
│
├── coding/
│   ├── SKILL.md
│   └── standards.md
│
└── security/
    ├── SKILL.md
    └── checklist.md
```

---

# 48. `subagents/`

Example:

```text
subagents/
│
├── researcher/
│   ├── deepagents.toml
│   ├── AGENTS.md
│   └── skills/
│
├── coder/
│   ├── deepagents.toml
│   └── AGENTS.md
│
└── reviewer/
    ├── deepagents.toml
    └── AGENTS.md
```

---

# 49. MCP Integration

Deep Agents can use **MCP servers** to access external tools.

Conceptually:

```text
Deep Agent
     │
     ▼
MCP
     │
 ┌───┼──────────┐
 ▼   ▼          ▼
DB  Search     APIs
```

Example:

```text
mcp.json

{
  "mcpServers": {
    "docs": {
      "url": "https://example.com/mcp"
    }
  }
}
```

The Deep Agents deployment-oriented CLI documentation currently supports HTTP and SSE MCP transports; local stdio processes are not supported in that deployment path.

---

# 50. Deep Agents CLI

The Deep Agents ecosystem provides a CLI-oriented development/deployment path.

Typical commands include:

```bash
deepagents init
deepagents dev
deepagents deploy
```

Conceptually:

```text
init
 ↓
Create project

dev
 ↓
Develop/test locally

deploy
 ↓
Build and deploy
```

The CLI discovers configuration from the project's conventional files and directories.

---

# 51. `deepagents.toml`

The deployment configuration can contain sections such as:

```toml
[agent]
name = "my-agent"
model = "provider:model"

[auth]
provider = "anonymous"

[frontend]
enabled = true

[sandbox]
provider = "langsmith"

[memories]
backend = "store"
```

The `[agent]` section is the core required configuration in the CLI deployment structure; other sections are optional depending on the features required.

---

# 52. Sandboxes

A sandbox gives the agent an isolated execution environment.

Example:

```text
Deep Agent
    │
    ▼
Sandbox
    │
    ├── Files
    ├── Code
    ├── Shell
    └── Runtime
```

This is particularly useful for:

* Code execution
* Testing
* Data processing
* File manipulation
* Potentially unsafe operations

A sandbox limits the blast radius of agent actions.

---

# 53. Why Sandboxing Matters

Without isolation:

```text
Agent
 ↓
Operating System
 ↓
Production Environment
```

Potentially dangerous.

With isolation:

```text
Agent
 ↓
Sandbox
 ↓
Temporary Environment
```

Much safer.

---

# 54. Deep Agent Deployment Architecture

A production setup can look like:

```text
                  User
                   │
                   ▼
              Frontend/API
                   │
                   ▼
              Agent Server
                   │
        ┌──────────┼───────────┐
        ▼          ▼           ▼
      Agent      Memory      Tools
        │          │           │
        ▼          ▼           ▼
    Subagents     Store       MCP
        │
        ▼
     Sandbox
        │
        ▼
   Files / Code
```

---

# 55. Memory Architecture

A production Deep Agent may use multiple storage mechanisms:

```text
                   Agent
                     │
        ┌────────────┼────────────┐
        ▼            ▼            ▼
   Checkpointer     Store     Filesystem
        │            │            │
        ▼            ▼            ▼
    Thread State  Long-term    Workspace
```

These solve different problems.

---

# 56. Checkpointer

A checkpointer stores execution/thread state.

Useful for:

* Resuming execution
* Conversation state
* Long-running workflows
* Human approval interruptions

---

# 57. Store

A Store is better suited to information that must persist beyond a particular thread.

Examples:

```text
User preferences
Project information
Long-term memories
Profiles
Persistent knowledge
```

---

# 58. Filesystem

The filesystem is best suited for:

```text
Temporary work
Source code
Documents
Research notes
Intermediate artifacts
Generated files
```

---

# 59. Deep Agent Context Architecture

A useful conceptual model is:

```text
             MODEL CONTEXT
                   │
        ┌──────────┼──────────┐
        ▼          ▼          ▼
   Instructions   Current    Relevant
                  Task       Information
                              │
                              ▼
                         Retrieval
                              │
                              ▼
                         Files/Memory
```

The goal is:

> **Put the right information into the context at the right time.**

Not:

> Put everything into the context.

---

# 60. Deep Agent as an Operating System

A useful analogy is:

```text
Traditional Computer

CPU
 ↓
Memory
 ↓
Disk
 ↓
Programs
 ↓
I/O
```

Deep Agent:

```text
LLM
 ↓
Context
 ↓
Filesystem
 ↓
Skills
 ↓
Tools
 ↓
Subagents
 ↓
Memory
```

The LLM is effectively the **decision-making processor**, while the surrounding infrastructure provides the environment needed for complex work.

---

# 61. Deep Agent as a Project Manager

Another useful analogy:

```text
User = Client

Main Agent = Project Manager

Subagents = Specialists

Skills = SOPs / Manuals

Tools = Software / Equipment

Filesystem = Project Folder

Memory = Organizational Knowledge

Todo List = Project Plan

Model = Decision Maker
```

This analogy explains why Deep Agents are effective for complex projects.

---

# 62. Example: Research Deep Agent

User:

```text
"Prepare a comprehensive report on the semiconductor industry."
```

Deep Agent:

```text
1. Create research plan
2. Search for sources
3. Delegate market research
4. Delegate technology research
5. Delegate competitor analysis
6. Save findings to files
7. Cross-check information
8. Summarize findings
9. Generate report
10. Review report
11. Produce final document
```

Filesystem:

```text
/research
├── plan.md
├── market.md
├── technology.md
├── competitors.md
├── sources.md
├── analysis.md
└── final-report.md
```

---

# 63. Example: Coding Deep Agent

User:

```text
"Fix all authentication issues in my application."
```

Execution:

```text
Inspect repository
       ↓
Understand architecture
       ↓
Create plan
       ↓
Read authentication files
       ↓
Analyze vulnerabilities
       ↓
Implement changes
       ↓
Run tests
       ↓
Observe failures
       ↓
Fix failures
       ↓
Run tests again
       ↓
Review changes
       ↓
Final response
```

---

# 64. Example: Multi-Agent Deep Agent

Task:

```text
"Build a complete startup business plan."
```

Main Agent:

```text
                  Main Agent
                       │
       ┌───────────────┼───────────────┐
       ▼               ▼               ▼
Market Agent      Finance Agent    Tech Agent
       │               │               │
       ▼               ▼               ▼
Market Size       Revenue Model     Architecture
Competitors       Costs             Stack
Trends            Forecast          MVP
       │               │               │
       └───────────────┼───────────────┘
                       ▼
                 Main Agent
                       │
                       ▼
                 Final Plan
```

---

# 65. Deep Agent Failure Handling

A robust Deep Agent must assume that failures happen.

Possible failures:

```text
Tool failure
API failure
Invalid output
Bad reasoning
Missing information
File conflict
Code compilation error
Test failure
Timeout
Model failure
```

The agent should:

```text
Detect
 ↓
Diagnose
 ↓
Retry / Correct
 ↓
Verify
```

---

# 66. Retry Strategy

Example:

```text
API call
   ↓
Failure
   ↓
Check error
   ↓
Retry?
 ┌─┴─┐
Yes No
 │   │
 ▼   ▼
Retry  Escalate
```

Retries should not blindly repeat dangerous actions.

---

# 67. Verification

Deep Agents should not simply assume an action succeeded.

Bad:

```text
Agent:
"I fixed the code."
```

Better:

```text
Modify code
 ↓
Run tests
 ↓
Check results
 ↓
Inspect output
 ↓
Confirm fix
```

Verification is particularly important for:

* Code
* Research
* Security
* Data processing
* Production changes

---

# 68. Observability

Complex agents need visibility into what they are doing.

Useful information:

```text
Task started
Tool called
Tool result
Subagent started
Subagent completed
File changed
Error occurred
Retry
Human approval requested
Task completed
```

This helps developers debug agent behavior.

---

# 69. Deep Agent Security

Deep Agents can have significant capabilities, therefore security is critical.

Important controls:

### Tool permissions

Only provide necessary tools.

```text
Researcher
→ Search only

Coder
→ Filesystem + test runner

Deployment Agent
→ Deployment tools
```

---

### Filesystem permissions

Control:

```text
Read
Write
Delete
Execute
```

---

### Sandbox

Use isolated execution for risky operations.

---

### Human approval

Require approval for high-impact actions.

---

### Authentication

Secure access to the agent and its tools.

---

### Secrets

Never expose:

```text
API keys
Passwords
Tokens
Private keys
```

to the model unnecessarily.

---

# 70. Principle of Least Privilege

A Deep Agent should have only the permissions required for its task.

Bad:

```text
Agent
 └── Full production access
```

Better:

```text
Research Agent
 └── Search access

Coding Agent
 └── Development repository

Deployment Agent
 └── Deployment API
     + approval requirement
```

---

# 71. Deep Agent vs Workflow

A deterministic workflow:

```text
A → B → C → D
```

A Deep Agent:

```text
A
 ↓
Decide
 ├── B
 ├── C
 └── D
      ↓
Observe
      ↓
Decide again
```

The Deep Agent can adapt its execution based on results.

---

# 72. Deep Agent vs Simple Tool-Calling Agent

### Simple Tool-Calling Agent

```text
Prompt
 ↓
LLM
 ↓
Tool
 ↓
Result
```

### Deep Agent

```text
Prompt
 ↓
Planning
 ↓
Task Management
 ↓
Context Management
 ↓
Tools
 ↓
Filesystem
 ↓
Memory
 ↓
Subagents
 ↓
Verification
 ↓
Long-running execution
```

---

# 73. Deep Agent vs Multi-Agent System

They are related but not identical.

A Deep Agent **can use subagents**, but subagents are only one part of the architecture.

```text
Deep Agent
│
├── Planning
├── Filesystem
├── Memory
├── Skills
├── Tools
├── Context Management
└── Subagents
```

A multi-agent system may focus primarily on:

```text
Agent A
   ↓
Agent B
   ↓
Agent C
```

Deep Agents focus more broadly on **building a capable long-running execution environment**.

---

# 74. Deep Agent Design Principles

## Principle 1 — Externalize Work

Don't keep everything inside model context.

Use:

```text
Files
Memory
State
```

---

## Principle 2 — Decompose Complex Tasks

Break:

```text
Huge Task
```

into:

```text
Small Tasks
```

---

## Principle 3 — Delegate

Use subagents for specialized work.

---

## Principle 4 — Load Knowledge on Demand

Use skills and progressive disclosure.

---

## Principle 5 — Verify

Never blindly trust tool results.

---

## Principle 6 — Persist

Save useful state and artifacts.

---

## Principle 7 — Isolate

Use sandboxing and permissions.

---

## Principle 8 — Keep Context Relevant

Only provide the model with what it needs.

---

# 75. Typical Deep Agent Lifecycle

```text
1. Receive objective
        ↓
2. Understand requirements
        ↓
3. Create plan
        ↓
4. Create tasks
        ↓
5. Inspect environment
        ↓
6. Load relevant skills
        ↓
7. Execute tools
        ↓
8. Write intermediate artifacts
        ↓
9. Delegate to subagents
        ↓
10. Observe results
        ↓
11. Update plan
        ↓
12. Summarize context when necessary
        ↓
13. Verify work
        ↓
14. Request human approval if required
        ↓
15. Complete objective
        ↓
16. Save important state
        ↓
17. Return final result
```

---

# 76. Minimal Conceptual Implementation

A simplified Deep Agent can be thought of as:

```python
agent = DeepAgent(
    model=model,
    tools=tools,
    filesystem=filesystem,
    memory=memory,
    skills=skills,
    subagents=subagents
)
```

Then:

```python
result = agent.run(
    "Analyze the project and fix the authentication system."
)
```

Internally:

```text
run()
 │
 ├── plan()
 │
 ├── create_tasks()
 │
 ├── execute()
 │     ├── tools
 │     ├── filesystem
 │     └── subagents
 │
 ├── observe()
 │
 ├── update_plan()
 │
 ├── verify()
 │
 └── return()
```

---

# 77. LangChain Deep Agents

The current LangChain implementation exposes a prebuilt Deep Agent through:

```python
create_deep_agent(...)
```

The Deep Agent is built on top of the standard agent infrastructure and assembles common capabilities such as:

* Filesystem
* Planning
* Subagents
* Memory
* Summarization
* Prompt caching

This makes it suitable for long-running coding and research tasks.

---

# 78. Conceptual Python Example

```python
from deepagents import create_deep_agent

agent = create_deep_agent(
    model="...",
    tools=[
        search_tool,
        database_tool,
    ],
)

result = agent.invoke({
    "messages": [
        {
            "role": "user",
            "content": "Analyze this project and prepare a report."
        }
    ]
})
```

The exact API and configuration should be checked against the version of the Deep Agents package being used.

---

# 79. Custom Deep Agent Architecture

For more control, the underlying agent infrastructure can be assembled manually.

Conceptually:

```python
agent = create_agent(
    model=model,
    tools=tools,
    middleware=[
        FilesystemMiddleware(...),
        SummarizationMiddleware(...),
        MemoryMiddleware(...),
        SkillsMiddleware(...),
        SubAgentMiddleware(...),
    ],
)
```

This illustrates an important idea:

> **Deep Agents are largely a preassembled collection of agent capabilities and middleware rather than a completely separate category of model.**

The official documentation shows this relationship between `create_agent` and the prebuilt `create_deep_agent` stack.

---

# 80. Deep Agents and LangGraph

The relationship can be visualized as:

```text
LangGraph
   │
   ▼
Agent Infrastructure
   │
   ▼
LangChain Agents
   │
   ▼
Deep Agents
   │
   ├── Planning
   ├── Filesystem
   ├── Memory
   ├── Skills
   ├── Subagents
   └── Context Management
```

LangGraph provides lower-level infrastructure for long-running, stateful execution, persistence, streaming, and human-in-the-loop workflows.

---

# 81. Deep Agent Context Engineering

One of the most important ideas is **context engineering**.

The objective is:

```text
Maximum useful information
            +
Minimum irrelevant information
```

A Deep Agent may combine:

```text
System instructions
        +
Current task
        +
Relevant memory
        +
Relevant skill
        +
Relevant files
        +
Relevant tool output
```

rather than dumping an entire knowledge base into the context.

---

# 82. Information Flow

```text
                 User Task
                     │
                     ▼
                  Planner
                     │
          ┌──────────┼──────────┐
          ▼          ▼          ▼
       Memory      Skills     Files
          │          │          │
          └──────────┼──────────┘
                     ▼
                   Model
                     │
          ┌──────────┼──────────┐
          ▼          ▼          ▼
        Tools    Subagents   Files
          │          │          │
          └──────────┼──────────┘
                     ▼
                  Results
                     │
                     ▼
                 Verification
                     │
                     ▼
                  Final Task
```

---

# 83. Deep Agent Advantages

### 1. Handles complex tasks

Can break large objectives into manageable tasks.

### 2. Long-running execution

Can continue working through many steps.

### 3. External workspace

Filesystem prevents excessive context growth.

### 4. Delegation

Subagents allow specialization.

### 5. Reusable knowledge

Skills avoid repeating instructions.

### 6. Persistent memory

Information can survive beyond a single interaction.

### 7. Better context management

Information can be loaded when required.

### 8. Tool integration

Can interact with real systems.

### 9. Verification

Can execute and inspect results.

### 10. Human control

High-impact actions can require approval.

---

# 84. Deep Agent Limitations

Deep Agents are powerful but introduce complexity.

### 1. Higher cost

Many model/tool calls can become expensive.

### 2. Latency

Complex tasks may require many steps.

### 3. Non-determinism

The same task may execute differently.

### 4. Failure propagation

A wrong decision early in execution can affect later steps.

### 5. Security risk

More capabilities mean a larger attack surface.

### 6. Debugging difficulty

It is harder to understand why an autonomous system made a decision.

### 7. State complexity

Memory, files, threads, and subagents can become difficult to manage.

### 8. Tool reliability

The overall system is only as reliable as its tools and execution environment.

---

# 85. When to Use Deep Agents

Deep Agents are especially useful when the task has:

```text
Complexity       → High
Steps            → Many
Duration         → Long
Tools            → Multiple
Files             → Many
Specialization   → Required
Memory           → Useful
Verification     → Required
```

Examples:

* Software engineering
* Large research projects
* Data analysis
* Document generation
* Security audits
* Complex investigations
* Multi-stage automation
* Long-running coding tasks

---

# 86. When NOT to Use Deep Agents

Do not use a Deep Agent for every problem.

For:

```text
"Convert 5 USD to INR"
```

a Deep Agent is unnecessary.

For:

```text
"Summarize this paragraph"
```

a simple LLM call is sufficient.

For:

```text
"Calculate 25 × 37"
```

a calculator is sufficient.

The more useful Deep Agents become as:

```text
Task Complexity ↑
Task Duration ↑
Tool Count ↑
State Requirements ↑
```

---

# 87. Deep Agent Mental Model

Remember this:

```text
                 DEEP AGENT
                     │
        ┌────────────┼────────────┐
        │            │            │
      THINK        WORK         REMEMBER
        │            │            │
     Planning      Tools        Memory
     Reasoning     Files        State
     Decisions     Code         Store
        │            │            │
        └────────────┼────────────┘
                     │
                 DELEGATE
                     │
                 Subagents
                     │
                     ▼
                 VERIFY
                     │
                     ▼
                  FINISH
```

---

# 88. Most Important Terms

| Term                   | Meaning                                               |
| ---------------------- | ----------------------------------------------------- |
| Deep Agent             | Long-running agent with a rich execution environment  |
| Harness                | Components surrounding the model that shape execution |
| Planner                | Creates and updates execution plan                    |
| Todo                   | Tracks individual tasks                               |
| Tool                   | Performs an external action                           |
| Filesystem             | External workspace                                    |
| Backend                | Storage implementation behind filesystem operations   |
| Memory                 | Persistent information                                |
| Skill                  | Reusable instructions/knowledge                       |
| Subagent               | Specialized delegated agent                           |
| Middleware             | Components that add agent capabilities                |
| Summarization          | Compresses accumulated context                        |
| Checkpointer           | Persists execution/thread state                       |
| Store                  | Persistent long-term storage                          |
| Sandbox                | Isolated execution environment                        |
| MCP                    | Protocol for connecting external tools                |
| Human-in-the-loop      | Human approval/control mechanism                      |
| Durable execution      | Ability to survive interruptions/failures             |
| Progressive disclosure | Loading detailed knowledge only when required         |

---

# 89. One-Line Definitions for Exams/Interviews

### Deep Agent

> A Deep Agent is a long-running AI agent equipped with planning, tools, filesystem access, memory, subagents, context management, and other capabilities required to execute complex multi-step tasks.

### Filesystem

> The filesystem acts as an external workspace where an agent can store, retrieve, and manipulate information and intermediate artifacts.

### Skill

> A Skill is a reusable set of instructions and knowledge that teaches an agent how to perform a particular type of task.

### Subagent

> A Subagent is a specialized agent that receives a delegated task from a parent agent and returns its result.

### Backend

> A Backend provides the storage implementation behind the Deep Agent's filesystem-like interface.

### Checkpointer

> A Checkpointer persists execution or thread state so a long-running agent can resume its work.

### Store

> A Store provides persistent information that can survive beyond an individual thread or execution.

### Sandbox

> A Sandbox provides an isolated environment in which the agent can safely execute code and perform potentially risky operations.

### Progressive Disclosure

> Progressive disclosure means exposing only the information initially needed and loading detailed knowledge, such as skills or files, when required.

### Context Management

> Context management is the process of keeping the model's context relevant and manageable during long-running execution.

---

# 90. Final Architecture to Remember

The complete Deep Agent can be remembered as:

```text
                         USER
                          │
                          ▼
                    ┌───────────┐
                    │Deep Agent │
                    └─────┬─────┘
                          │
             ┌────────────┼────────────┐
             │            │            │
             ▼            ▼            ▼
         Planning      Context       Memory
             │        Management        │
             │            │             │
             └────────────┼─────────────┘
                          │
             ┌────────────┼────────────┐
             ▼            ▼            ▼
         Filesystem     Skills      Subagents
             │            │            │
             └────────────┼────────────┘
                          │
                    ┌─────▼─────┐
                    │   Tools   │
                    └─────┬─────┘
                          │
             ┌────────────┼────────────┐
             ▼            ▼            ▼
            APIs        Code         MCP
             │            │            │
             └────────────┼────────────┘
                          ▼
                    Execution
                          │
                          ▼
                    Verification
                          │
                    ┌─────▼─────┐
                    │   Human   │
                    │ Approval  │
                    └─────┬─────┘
                          │
                          ▼
                    Final Result
```

---

# 91. The Core Formula

The simplest way to remember Deep Agents is:

```text
Deep Agent
=
LLM
+
Planning
+
Tools
+
Filesystem
+
Memory
+
Skills
+
Subagents
+
Context Management
+
Persistence
+
Execution Environment
+
Verification
```

Or even more simply:

```text
                    DEEP AGENT
                         =
              Intelligence + Workspace
                         +
                 Memory + Tools
                         +
              Planning + Delegation
                         +
                 Long-running State
```

---

# 92. Final Takeaway

The defining idea behind Deep Agents is **not simply giving an LLM more tools**.

It is creating an environment where the model can:

```text
Understand
   ↓
Plan
   ↓
Decompose
   ↓
Work
   ↓
Store information
   ↓
Delegate
   ↓
Use tools
   ↓
Observe
   ↓
Correct
   ↓
Verify
   ↓
Remember
   ↓
Continue
   ↓
Complete
```

Therefore, a Deep Agent can be viewed as:

> **An LLM surrounded by the infrastructure required to perform complex, long-running, stateful work.**

The most important components to remember are:

```text
1. Planning
2. Filesystem
3. Memory
4. Skills
5. Subagents
6. Tools
7. Context Management
8. Summarization
9. Persistence
10. Sandboxing
11. Human-in-the-loop
12. Verification
```

Together, these transform a basic tool-calling agent into a **long-running execution system capable of handling complex projects**.