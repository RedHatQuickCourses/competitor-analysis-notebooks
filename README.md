# Competitor Analysis Use-case Demonstration with LlamaStack

> **LlamaStack Version**: 0.3.0

These notebooks are part of the "competitor analysis" proof-of-concept focused on the Indian Banking industry. They demonstrate RAG, agentic AI, and custom tool calling capabilities using LlamaStack.

## Prerequisites

Before running these notebooks on Red Hat OpenShift AI, you need to:

1. Prepare the RHOAI environment - See [competitor-analysis setup guide](https://github.com/rsriniva/competitor-analysis)
2. Have documents ingested via the KFP pipeline
3. Have embeddings stored in Milvus vector store (`competitor-docs`)
4. Set up Tavily API key for web search (notebooks 5-7)

---

## Notebooks Overview

### 1. MaaS Test (`1-maas-test.ipynb`)

**Purpose**: Basic connectivity test for remote Model-as-a-Service (MaaS) inference endpoint.

**What it does**:
- Tests direct API calls to the Granite model via MaaS
- Verifies API authentication and connectivity
- Simple completion request without LlamaStack

**Dependencies**: `requests`

---

### 2. LlamaStack Basic Test (`2-llamastack-test-basic.ipynb`)

**Purpose**: Verify LlamaStack server connectivity and demonstrate basic API operations.

**What it does**:
- Connects to LlamaStack server
- Lists available models and vector stores
- Tests basic chat completion API
- Demonstrates key 0.3.0 API patterns

**Key APIs**:
```python
client.models.list()
client.vector_stores.list()
client.chat.completions.create()
```

**Dependencies**: `llama-stack-client==0.3.0`

---

### 3. Simple RAG (`3-simple-rag.ipynb`)

**Purpose**: Demonstrate Retrieval-Augmented Generation using LlamaStack's vector store APIs.

**What it does**:
- Connects to pre-ingested document vector store (Milvus)
- Performs semantic search on competitor banking documents
- Generates LLM responses augmented with retrieved context
- Shows source attribution for answers

**Key Features**:
- Semantic search with embeddings
- Context-aware LLM responses
- Document source tracking

**Key APIs**:
```python
client.vector_stores.list()
client.vector_stores.search()
client.chat.completions.create()
```

**Dependencies**: `llama-stack-client==0.3.0`, `rich`, `pandas`

---

### 4. Agentic RAG (`4-agentic-rag.ipynb`)

**Purpose**: Combine RAG with agentic AI for intelligent document Q&A.

**What it does**:
- Uses Docling for advanced PDF extraction with OCR
- Creates/registers vector stores in LlamaStack
- Implements two-phase document ingestion (PDF→Markdown→Vector DB)
- Uses Agent with `file_search` tool for RAG queries

**Key Features**:
- Docling-based document processing
- Chunked document ingestion (handles large PDFs)
- Agent-based RAG with automatic tool selection

**Key APIs**:
```python
client.vector_stores.create()
client.tool_runtime.rag_tool.insert()
Agent(client, model, tools=[{"type": "file_search", "vector_store_ids": [...]}])
```

**Dependencies**: `llama-stack-client==0.3.0`, `docling`, `rich`

---

### 5. Real-Time Search (`5-real-time-search.ipynb`)

**Purpose**: Demonstrate web search integration for real-time information queries.

**What it does**:
- Integrates Tavily web search API via LlamaStack
- Answers questions requiring current information (exchange rates, prices, news)
- Shows both Agent-based and Responses API approaches

**Key Features**:
- Real-time web search capability
- Tavily API integration
- Multi-turn conversations with web context

**Key APIs**:
```python
# Using Agent class
agent = Agent(client, model, tools=[{"type": "web_search"}])
agent.create_turn(messages=[...])

# Using Responses API
client.responses.create(model, input, tools=[{"type": "web_search"}])
```

**Dependencies**: `llama-stack-client==0.3.0`, Tavily API key

---

### 6. ReAct Agent (`6-smart-agent.ipynb`)

**Purpose**: Demonstrate custom tool calling with multi-tool agent workflows.

**What it does**:
- Creates custom Yahoo Finance tool for Indian bank stock prices
- Combines custom tools with web search
- Implements smart query routing based on query content
- Shows manual tool execution loop with Chat Completions API

**Key Features**:
- Custom client-side tool definition (`@client_tool` decorator)
- Yahoo Finance API integration (HDFC, ICICI, SBI stocks)
- Query routing: stocks → Yahoo Finance, other → web search
- OpenAI-compatible function calling

**Supported Stock Tickers**:
- `HDFCBANK.NS` - HDFC Bank
- `ICICIBANK.NS` - ICICI Bank
- `SBIN.NS` - State Bank of India

**Key APIs**:
```python
@client_tool
def indian_bank_stock(ticker: str, period: str = "current"):
    ...

client.chat.completions.create(model, messages, tools=[...], tool_choice="required")
```

**Dependencies**: `llama-stack-client==0.3.0`, `yfinance`

---

### 7. Multi-Tool Agent (`7-multi-tool-agent.ipynb`)

**Purpose**: Clean implementation of multi-tool agent with LLM-driven tool selection.

**What it does**:
- Defines multiple tools with OpenAI-compatible function schemas
- Lets LLM automatically choose the right tool based on query
- Executes tools client-side and returns results to LLM
- Provides clean, minimal code pattern for multi-tool agents

**Key Features**:
- OpenAI-compatible tool definitions
- Automatic tool selection (`tool_choice="auto"`)
- Client-side tool execution loop
- Clean separation of tool definition and execution

**Tools Available**:
| Tool | Purpose |
|------|---------|
| `indian_bank_stock` | Stock prices for HDFC, ICICI, SBI |
| `web_search` | Gold prices, exchange rates, news |

**Key Pattern**:
```python
TOOLS = [
    {"type": "function", "function": {"name": "indian_bank_stock", ...}},
    {"type": "function", "function": {"name": "web_search", ...}}
]

response = client.chat.completions.create(
    model=model_id,
    messages=messages,
    tools=TOOLS,
    tool_choice="auto"
)

# Check tool_calls and execute the selected tool
if response.choices[0].message.tool_calls:
    # Execute tool and return result to LLM
```

**Dependencies**: `llama-stack-client==0.3.0`, `yfinance`, `rich`

---

## API Quick Reference (LlamaStack 0.3.0)

### Chat Completions
```python
response = client.chat.completions.create(
    model="model-id",
    messages=[{"role": "user", "content": "..."}],
    tools=[...],           # Optional: tool definitions
    tool_choice="auto"     # Optional: auto, required, none
)
```

### Vector Stores
```python
# List
stores = client.vector_stores.list()

# Create
client.vector_stores.create(name="...", metadata={...})

# Search
results = client.vector_stores.search(vector_store_id="...", query="...")
```

### Responses API (Agentic)
```python
response = client.responses.create(
    model="model-id",
    input="user query",
    instructions="system prompt",
    tools=[{"type": "web_search"}]
)
print(response.output_text)
```

### Agent Class
```python
from llama_stack_client import Agent

agent = Agent(client, model="model-id", tools=[...])
session_id = agent.create_session("session-name")
response = agent.create_turn(messages=[...], session_id=session_id)
```

---

## Environment Variables

| Variable | Description | Used By |
|----------|-------------|---------|
| `VLLM_API_TOKEN` | MaaS API authentication token | Notebook 1 |
| `TAVILY_SEARCH_API_KEY` | Tavily web search API key | Notebooks 5, 6, 7 |

---

## Progression Path

1. **Start with 1 & 2** - Verify connectivity
2. **Try 3** - Understand basic RAG
3. **Try 4** - Learn document ingestion and agentic RAG
4. **Try 5** - Add real-time web search
5. **Try 6 & 7** - Build custom tools and multi-tool workflows
