# Agentic AI

My notes and hands-on code from working through the **Agentic AI One Shot Course**, covering the modern LLM application stack end to end: LangChain, LangGraph, RAG (with and without vectors), Deep Agents, Guardrails, LLM Evaluation, and LLM Gateways.

Each module pairs a long-form `notes.md` (roughly 2,500 to 4,200 lines each) with runnable Jupyter notebooks or Python scripts. Most examples run on **Groq** (`llama-3.3-70b-versatile`, `openai/gpt-oss-120b`), so you can follow along on the free tier.

**Course video:** [Agentic AI One Shot Course](https://www.youtube.com/watch?v=rV3HJ4LEZ7k&t=382s)

---

## Modules

| # | Module | What's inside | Code |
|---|--------|---------------|------|
| 01 | [LangChain](01.%20LangChain/) | Agents, model integration, tools, messages, structured output, middlewares, guardrails | 7 notebooks + LLM gateway notebook |
| 02 | [LangGraph](02.%20LangGraph/) | State graphs, ReAct agents, memory, streaming, human-in-the-loop, MCP | 2 notebooks + MCP client/servers |
| 03 | [RAG](03.%20RAG/) | Ingestion, chunking, embeddings, ChromaDB and FAISS vector stores, retrieval pipelines, RAG evaluation | 3 notebooks + modular `src/` pipeline |
| 04 | [Vectorless RAG](04.%20Vectorless%20RAG/) | Reasoning-based retrieval with PageIndex tree search, so no embeddings and no chunking | 1 notebook |
| 05 | [Deep Agents](05.%20Deep%20Agents/) | Planning, todos, a virtual filesystem, subagents, and custom system prompts with `deepagents` | 1 notebook |
| 06 | [Guardrails](06.%20Guardrails/) | Input, output, tool, RAG, and PII guardrails, risk levels, and HITL patterns | Notes (code is in `01. LangChain/LangChain/07. GuardRails.ipynb`) |
| 07 | [LLM Evaluation](07.%20LLM%20Evaluation/) | Offline and online evals, datasets, LLM-as-a-judge, rubrics, and judge biases | Notes (code is in `03. RAG/rag_evaluation.ipynb`) |
| 08 | [LLM Gateways](08.%20LLM%20Gateways/) | Provider abstraction, routing and load-balancing strategies, fallbacks, caching, cost tracking, observability, and gateway-level guardrails | Notes (code is in `01. LangChain/llm_gateways.ipynb`) |

---

## Highlights

### 01 · LangChain
- `create_agent` with custom tools, plus the tool execution loop
- One interface over **OpenAI (`gpt-4.1`), Gemini (`gemini-2.5-flash-lite`), and Groq** through `init_chat_model` and the provider classes, with streaming and batching
- `SystemMessage`, `HumanMessage`, `AIMessage`, and `ToolMessage`
- Structured output with **Pydantic**, **TypedDict**, and **dataclasses**, including nested schemas
- **Middlewares:** summarization (token-size and fraction triggers) and Human-in-the-Loop (approve, reject, edit)
- **Guardrails:** deterministic and model-based checks, built-in `PIIMiddleware`, custom `before_agent` and `after_agent` hooks, layered guardrails, and a healthcare chatbot use case

### 01 · LLM Gateways (`llm_gateways.ipynb`)
- A unified `completion()` API across OpenAI, Gemini, and Groq with **LiteLLM**
- Automatic fallbacks, cost tracking with `completion_cost`, and in-memory response caching
- Smart routing and load balancing with `Router`: `simple-shuffle`, `least-busy`, `latency-based-routing`, and `cost-based-routing`
- Observability through LiteLLM success and failure callbacks
- LangChain integration via `ChatLiteLLM` and `.with_fallbacks()`
- An end-to-end smart router for a chatbot: a cheap classifier picks a model chain per query type
- Gateway-level guardrails in pure Python callbacks: PII redaction (email, phone, SSN, Aadhaar, PAN, card, IP), prompt-injection blocking, and forbidden topics
- Production best practices and a comparison of popular LLM gateways

### 02 · LangGraph
- A chatbot built with the Graph API: `StateGraph`, nodes, edges, and `add_messages` reducers
- Tool-calling chatbot using `ToolNode` and `tools_condition`, with Tavily web search
- **ReAct agent** architecture (act, observe, reason)
- Persistent memory with `MemorySaver` checkpointers, plus `stream()` and `astream()` modes
- **Human-in-the-loop** with `interrupt` and `Command`
- **MCP demo:** a `MultiServerMCPClient` agent that connects to a `stdio` math server and a `streamable-http` weather server

### 03 · RAG
- **Notebooks:** document loaders (`data.ipynb`), then PDF processing, `RecursiveCharacterTextSplitter`, `all-MiniLM-L6-v2` embeddings, a ChromaDB vector store, a retriever, and simple, enhanced, and streaming RAG pipelines (`pdf_loader.ipynb`)
- **Modular pipeline (`src/`):**
  - `data_loader.py` loads PDF, TXT, CSV, Excel, Word, and JSON files
  - `embedding.py` handles chunking and SentenceTransformer embeddings
  - `vectorstore.py` builds, saves, loads, and queries a FAISS index
  - `search.py` retrieves context and summarizes it with Groq
- **Evaluation (`rag_evaluation.ipynb`):** chatbot and RAG evaluation with LangSmith datasets and experiments, using LLM-as-a-judge evaluators for **correctness, relevance, groundedness, and retrieval relevance**

### 04 · Vectorless RAG
- Uploads a PDF ("Attention Is All You Need") to **PageIndex** and builds a hierarchical tree index
- An **LLM tree search** reasons over section titles and summaries to choose nodes, instead of using embedding similarity
- An end-to-end pipeline: tree search, then node retrieval, then a grounded answer
- The LLM calls go to Groq (`openai/gpt-oss-120b`) through the OpenAI SDK's Groq-compatible `base_url`

### 05 · Deep Agents
- Compares a basic agent with `create_deep_agent` on a research task
- Built-in `write_todos` planning, a virtual filesystem, and subagent delegation
- Custom system prompts and an internet search tool via Tavily

### 06 to 08 · Guardrails, Evaluation, Gateways (notes)
- **Guardrails:** a layered defense model, a tool permission model, least privilege, action risk levels 0 to 4, and handling prompt injection in retrieved documents
- **Evaluation:** model, prompt, component, application, and agent-level evals, pairwise vs. pointwise judging, and rubric design
- **Gateways:** why gateways exist, provider abstraction, routing strategies, fallbacks and retries, caching, rate limiting, and observability (the hands-on code is described under *01 · LLM Gateways* above)

---

## Repository Structure

```text
Agentic-AI/
├── 01. LangChain/
│   ├── LangChain/              # 01. Intro … 07. GuardRails (notebooks)
│   ├── llm_gateways.ipynb      # LiteLLM gateway demos
│   ├── notes.md
│   └── pyproject.toml
├── 02. LangGraph/
│   ├── 01. Basic ChatBot/
│   ├── 02. Human Assistance/
│   ├── 03. MCP Demo/           # client.py, mathserver.py, weather.py, test_groq.py
│   ├── notes.md
│   └── pyproject.toml
├── 03. RAG/
│   ├── data/                   # pdf_files/ (attention, agriculture), text_files/
│   ├── notebook/               # data.ipynb, pdf_loader.ipynb
│   ├── src/                    # data_loader, embedding, vectorstore, search
│   ├── app.py
│   ├── rag_evaluation.ipynb
│   ├── notes.md
│   └── pyproject.toml
├── 04. Vectorless RAG/
│   ├── data/attention.pdf
│   ├── vectorless_rag.ipynb
│   └── notes.md
├── 05. Deep Agents/
│   ├── deep_agents_basics.ipynb
│   ├── notes.md
│   └── pyproject.toml
├── 06. Guardrails/notes.md
├── 07. LLM Evaluation/notes.md
└── 08. LLM Gateways/notes.md
```

---

## Tech Stack

- **Frameworks:** LangChain, LangGraph, LangSmith, Deep Agents, LiteLLM, MCP (`FastMCP`, `langchain-mcp-adapters`)
- **LLM providers:** Groq, OpenAI, Google Gemini
- **Retrieval:** ChromaDB, FAISS, Sentence Transformers, HuggingFace Embeddings, PageIndex
- **Document loading:** PyPDF, PyMuPDF, BeautifulSoup
- **Structured output:** Pydantic, TypedDict, dataclasses
- **Tools:** Tavily Search
- **Tooling:** Python 3.14, [uv](https://docs.astral.sh/uv/), Jupyter

---

## Getting Started

Modules 01, 02, 03, and 05 are each their own **uv** project with its own `pyproject.toml` and `uv.lock`. Module 04 (Vectorless RAG) has no project file (see below), and modules 06 to 08 are notes only.

### 1. Clone

```bash
git clone https://github.com/Saksham-Jaiswal-2004/Agentic-AI.git
cd Agentic-AI
```

### 2. Install a module's dependencies

```bash
cd "02. LangGraph"
uv sync            # creates .venv from uv.lock
```

If you're not using uv, install from `requirements.txt` instead:

```bash
python -m venv .venv
.venv\Scripts\activate        # Windows
# source .venv/bin/activate   # macOS/Linux
pip install -r requirements.txt
```

> `pyproject.toml` is the source of truth. Some `requirements.txt` files list only the core packages. For example, `litellm` and `langchain-litellm` (01) and `langchain-huggingface` and `langsmith` (03) appear only in `pyproject.toml`, so prefer `uv sync`.

**Jupyter kernel:** 02 (LangGraph) and 05 (Deep Agents) don't declare `ipykernel`. Add it before running their notebooks:

```bash
uv add --dev ipykernel     # or: pip install ipykernel
```

**Vectorless RAG (04):** there is no project file, so create an environment by hand:

```bash
cd "04. Vectorless RAG"
python -m venv .venv
.venv\Scripts\activate
pip install pageindex openai python-dotenv ipykernel
```

### 3. Configure API keys

Create a `.env` file in the module folder. It is gitignored, so it won't be committed. Add only the keys the module you're running needs:

```env
GROQ_API_KEY=...          # used almost everywhere
OPENAI_API_KEY=...        # model integration, gateways
GOOGLE_API_KEY=...        # Gemini (LangChain)
GEMINI_API_KEY=...        # Gemini via LiteLLM
TAVILY_API_KEY=...        # web search (LangGraph, Deep Agents)
LANGSMITH_API_KEY=...     # tracing and evaluation
LANGSMITH_TRACING=true
PAGEINDEX_API_KEY=...     # Vectorless RAG
```

### 4. Run

- **Notebooks:** open them in VS Code or Jupyter and select the module's `.venv` kernel.
- **MCP demo:**
  ```bash
  cd "02. LangGraph/03. MCP Demo"
  python weather.py      # terminal 1: HTTP MCP server on :8000
  python client.py       # terminal 2: launches the math server over stdio and queries both
  python test_groq.py    # optional: checks your GROQ_API_KEY and lists available Groq models
  ```
  Run these from inside `03. MCP Demo` with the module's venv activated. `client.py` starts `mathserver.py` with the `python` on your PATH, using a relative path.
- **RAG pipeline:**
  ```bash
  cd "03. RAG"
  python src/search.py   # builds the FAISS index into faiss_store/ on first run (or loads it), then answers a sample query
  ```
  The index is gitignored, so the first run embeds everything in `data/`.

---

## Progress

- [x] LangChain
- [x] LangGraph
- [x] RAG
- [x] Vectorless RAG
- [x] Deep Agents
- [x] Guardrails
- [x] LLM Evaluation
- [x] LLM Gateways

---

## Course Timeline

| Timestamp | Topic |
|-----------|-------|
| 00:00:00 | Introduction |
| 00:02:31 | LangChain |
| 02:35:12 | LangGraph |
| 05:02:29 | RAG |
| 07:10:43 | Vectorless RAG |
| 08:02:11 | Deep Agents |
| 08:45:43 | Guardrails |
| 09:22:55 | LLM Evaluation |
| 10:30:25 | LLM Gateways |

---

## References

- [Agentic AI One Shot Course](https://www.youtube.com/watch?v=rV3HJ4LEZ7k&t=382s)
- [LangChain docs](https://docs.langchain.com/) · [LangGraph docs](https://langchain-ai.github.io/langgraph/) · [LangSmith](https://docs.smith.langchain.com/)
- [Deep Agents](https://github.com/langchain-ai/deepagents) · [LiteLLM](https://docs.litellm.ai/) · [PageIndex](https://pageindex.ai/) · [Model Context Protocol](https://modelcontextprotocol.io/)

---

## License

For educational and personal learning purposes.
