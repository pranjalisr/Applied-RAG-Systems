# Agentic RAG with Real-Time Tools

A DeepSeek-powered Agentic RAG system that can combine **internal document retrieval** with **real-time tool usage**.

This project goes beyond standard RAG. Instead of always searching only the vector database, the system decides which tool is needed for each query — local knowledge base search, stock information, Wikipedia lookup, web search, or weather data.

For this implementation, the demo uses:

* **DeepSeek** as the LLM
* **FAISS** as the local vector database
* **HuggingFace sentence-transformer embeddings** for local document indexing
* **NVIDIA Annual Report PDF** as the knowledge base document
* **Finance tool execution** with safe fallback handling when external providers fail or rate-limit requests

---

## What This Project Demonstrates

This project demonstrates an **agentic retrieval pipeline** where the system can answer questions that require more than one source of information.

Example query:

> What does NVIDIA's annual report say about AI/data center growth, and what is NVDA's current stock price?

The system handles this by:

1. Searching the local NVIDIA annual report PDF using FAISS-based RAG
2. Calling the finance tool for NVDA stock information
3. Passing both real tool outputs to DeepSeek
4. Generating a grounded final answer
5. Showing which tools were used

---

## Agentic RAG vs Standard RAG

| Standard RAG                        | Agentic RAG                                           |
| ----------------------------------- | ----------------------------------------------------- |
| Always searches the vector database | Decides which tool is needed                          |
| One retrieval path                  | Multiple tool paths possible                          |
| Works only with stored documents    | Can combine documents with external tools             |
| Best for static document Q&A        | Best for mixed questions needing live/contextual data |
| Usually one LLM call                | May involve multiple tool calls                       |
| Fixed pipeline                      | Dynamic query-driven pipeline                         |

**When to use Agentic RAG:**
Use Agentic RAG when the user asks questions that combine internal knowledge with external or changing information.

Example:

> Summarize NVIDIA's data center growth from the annual report and also check NVDA stock information.

**When to use Standard RAG:**
Use Standard RAG when all questions are only about static uploaded documents and you want a simpler, faster, lower-cost pipeline.

---

## Architecture

```text
User Question
      │
      ▼
┌──────────────────────┐
│ Query Router / Agent │
│ DeepSeek + Tool Logic│
└──────────┬───────────┘
           │
           ▼
┌────────────────────────────────────────────────────────────┐
│                         Tool Registry                      │
│                                                            │
│  ┌────────────────────────┐   ┌─────────────────────────┐ │
│  │ search_knowledge_base  │   │ get_stock_data          │ │
│  │ FAISS + PDF chunks     │   │ finance data / fallback │ │
│  └────────────────────────┘   └─────────────────────────┘ │
│                                                            │
│  ┌────────────────────────┐   ┌─────────────────────────┐ │
│  │ search_wikipedia       │   │ get_weather             │ │
│  │ Wikipedia API          │   │ OpenWeather / mock mode │ │
│  └────────────────────────┘   └─────────────────────────┘ │
│                                                            │
│  ┌────────────────────────┐                               │
│  │ web_search             │                               │
│  │ Tavily API / optional  │                               │
│  └────────────────────────┘                               │
└────────────────────────────────────────────────────────────┘
           │
           ▼
┌──────────────────────────────┐
│ Real Tool Outputs Collected  │
└──────────┬───────────────────┘
           │
           ▼
┌──────────────────────────────┐
│ DeepSeek Final Synthesis     │
└──────────┬───────────────────┘
           │
           ▼
Final Answer + Tools Used
```

---

## Why This Version Uses Deterministic Tool Execution

The original LangChain agent flow uses an older `initialize_agent()` ReAct-style parser. With OpenAI models, this can work well, but with DeepSeek through the OpenAI-compatible client, the model may sometimes write tool calls as plain text instead of executing them.

Example of the problem:

```text
Tool used: search_knowledge_base(...)
Tool used: get_stock_data(...)
```

That looks correct in the answer, but the tools may not actually run.

To make the system reliable, this implementation uses a deterministic tool execution layer:

```text
User query
   ↓
Python checks what tools are needed
   ↓
Runs the actual tools
   ↓
Collects real observations
   ↓
Sends tool outputs to DeepSeek
   ↓
DeepSeek writes final grounded answer
```

This makes the project more stable for demos and avoids fake tool usage.

---

## Tool Decision Logic

The system checks the user query and routes it to the right tools.

Example query:

```text
What does NVIDIA's annual report say about AI/data center growth,
and what is NVDA's current stock price?
```

The system detects:

| Query Signal            | Tool Used               |
| ----------------------- | ----------------------- |
| `annual report`         | `search_knowledge_base` |
| `AI/data center growth` | `search_knowledge_base` |
| `NVDA`                  | `get_stock_data`        |
| `stock price`           | `get_stock_data`        |

Then it produces a final answer using both outputs.

---

## Tools

### 1. `search_knowledge_base`

Searches the local FAISS vector index built from files inside:

```text
data/knowledge_base/
```

Supported file types:

```text
.pdf
.txt
```

For this demo, the knowledge base uses NVIDIA's annual report.

---

### 2. `get_stock_data`

Fetches stock/financial information for a ticker such as:

```text
NVDA
AAPL
MSFT
GOOGL
```

The original finance tool uses `yfinance`. However, free finance endpoints can sometimes fail or rate-limit repeated requests.

To avoid hallucinating stock prices, this implementation includes safe handling:

```text
If finance data is available:
    return the stock information

If external finance provider fails:
    clearly say the finance tool ran but live price was unavailable
```

This is safer than inventing a stock price.

---

### 3. `search_wikipedia`

Uses Wikipedia for general factual background.

Example:

```text
What does Wikipedia say about transformer models?
```

---

### 4. `web_search`

Uses Tavily for live web search if a `TAVILY_API_KEY` is provided.

If no Tavily key is present, the tool is shown as unavailable or mock mode.

---

### 5. `get_weather`

Uses OpenWeatherMap if an API key is provided.

If no weather key is present, the tool runs in mock mode.

---

## Data Source Used for Demo

For the knowledge base, use NVIDIA's annual report PDF.

Recommended document:

```text
NVIDIA Fiscal 2025 Annual Report
```

Source:

```text
https://s201.q4cdn.com/141608511/files/doc_financials/2025/annual/NVIDIA-2025-Annual-Report.pdf
```

Place it here:

```text
data/knowledge_base/nvidia-2025-annual-report.pdf
```

Why this PDF works well:

* It contains strong discussion of AI and data center growth
* It includes NVIDIA's revenue and business segment information
* It pairs naturally with the finance tool for NVDA stock-related questions
* It gives a clean demo of internal document search + external tool usage

---

## Setup

### 1. Clone the repository

```bash
git clone https://github.com/pranjalisr/Applied-RAG-Systems.git
cd Applied-RAG-Systems/05-agentic-rag-realtime
```

---

### 2. Create a virtual environment

Use Python 3.11.

```bash
python3.11 -m venv .venv
source .venv/bin/activate
```

Check Python version:

```bash
python --version
```

Expected:

```text
Python 3.11.x
```

---

### 3. Install dependencies

```bash
pip install --upgrade pip
pip install -r requirements.txt
```

If PDF loading fails, install `pypdf` manually:

```bash
pip install pypdf
```

---

### 4. Create `.env`

Create a `.env` file:

```bash
cp .env.example .env
```

Or create it manually:

```env
DEEPSEEK_API_KEY=your_deepseek_api_key_here
DEEPSEEK_MODEL=deepseek-v4-flash
DEEPSEEK_BASE_URL=https://api.deepseek.com

TAVILY_API_KEY=
OPENWEATHERMAP_API_KEY=

KNOWLEDGE_BASE_DIR=data/knowledge_base
```

Only the DeepSeek key is required for the main demo.

Tavily and OpenWeatherMap are optional.

---

## API Key Guide

| Service                | Required? | Purpose                                            |
| ---------------------- | --------- | -------------------------------------------------- |
| DeepSeek               | Yes       | Main LLM for final reasoning and answer generation |
| Tavily                 | Optional  | Live web search                                    |
| OpenWeatherMap         | Optional  | Weather data                                       |
| yfinance               | No key    | Stock data, but may be rate-limited                |
| Wikipedia              | No key    | Wikipedia summaries                                |
| FAISS                  | No key    | Local vector search                                |
| HuggingFace embeddings | No key    | Local text embeddings                              |

---

## Add the NVIDIA Annual Report

Create the knowledge base folder:

```bash
mkdir -p data/knowledge_base
```

Download the NVIDIA annual report:

```bash
curl -L "https://s201.q4cdn.com/141608511/files/doc_financials/2025/annual/NVIDIA-2025-Annual-Report.pdf" \
  -o data/knowledge_base/nvidia-2025-annual-report.pdf
```

---

## Build or Refresh the FAISS Index

The first run automatically builds the FAISS index.

If you change or replace documents, delete the old index:

```bash
rm -rf data/knowledge_base/.faiss_index
```

Then run the project again.

---

## Run the Project

### Single Query Mode

```bash
python main.py --query "What does NVIDIA's annual report say about AI/data center growth, and what is NVDA's current stock price?"
```

---

### Interactive Mode

```bash
python main.py --interactive
```

Then ask questions like:

```text
What does NVIDIA say about data center growth?
```

```text
Summarize NVIDIA's AI strategy from the annual report.
```

```text
Use the knowledge base and finance tool to answer: how is NVIDIA positioned in AI and what is NVDA stock status?
```

---

### Disable Memory

```bash
python main.py --interactive --no-memory
```

---

### Hide Verbose Trace

```bash
python main.py --query "Summarize NVIDIA's AI growth from the annual report" --no-verbose
```

---

## Example Output

Example query:

```bash
python main.py --query "What does NVIDIA's annual report say about AI/data center growth, and what is NVDA's current stock price?"
```

Example output:

```text
ANSWER

1. NVIDIA Annual Report Summary

According to the knowledge base, NVIDIA's fiscal 2025 annual report highlights extraordinary growth driven by AI and data center demand.

- Revenue surged 114% year-over-year to $130.5 billion.
- Data Center growth was led by demand for Hopper architecture.
- NVIDIA describes itself as a full-stack AI company spanning chips, systems, software, and AI models.
- Ethernet for AI and Spectrum-X were also highlighted as important contributors.

2. Current NVDA Stock Information

The finance tool for NVDA was executed, but the external finance provider did not return a usable live price in this run. The system reports this clearly instead of inventing a stock price.

3. Short Conclusion

NVIDIA's annual report shows major AI and data center growth, while the finance tool confirms that stock retrieval was attempted. If live finance data is unavailable, the system safely reports that limitation.

Tools Used: search_knowledge_base, get_stock_data
```

---

## Example Multi-Tool Queries

### Finance + RAG

```text
What does NVIDIA's annual report say about AI growth, and what is NVDA's stock status?
```

Expected tools:

```text
search_knowledge_base
get_stock_data
```

---

### RAG Only

```text
Summarize NVIDIA's data center growth from the annual report.
```

Expected tool:

```text
search_knowledge_base
```

---

### Wikipedia + RAG

```text
What is a GPU according to Wikipedia, and how does NVIDIA discuss GPUs in the annual report?
```

Expected tools:

```text
search_wikipedia
search_knowledge_base
```

---

### Weather + RAG

```text
What is the weather in Tokyo, and do our internal event guidelines mention weather risks?
```

Expected tools:

```text
get_weather
search_knowledge_base
```

---

### Web Search + RAG

```text
Search the latest AI infrastructure news and compare it with NVIDIA's annual report.
```

Expected tools:

```text
web_search
search_knowledge_base
```

Requires:

```env
TAVILY_API_KEY=your_tavily_key_here
```

---

## Project Structure

```text
05-agentic-rag-realtime/
├── main.py
├── requirements.txt
├── .env.example
├── .env
├── data/
│   └── knowledge_base/
│       ├── nvidia-2025-annual-report.pdf
│       └── .faiss_index/
├── src/
│   ├── agent.py
│   ├── knowledge_indexer.py
│   ├── response_formatter.py
│   ├── tool_registry.py
│   └── tools/
│       ├── finance_tool.py
│       ├── rag_tool.py
│       ├── weather_tool.py
│       ├── web_search_tool.py
│       └── wiki_tool.py
```

---

## Important Files

### `main.py`

Main entry point.

Responsible for:

* Loading environment variables
* Checking tool availability
* Building or loading FAISS index
* Creating the tool registry
* Connecting to DeepSeek
* Running single-query or interactive mode
* Calling tools and formatting the final answer

---

### `src/knowledge_indexer.py`

Builds the FAISS index from files in:

```text
data/knowledge_base/
```

Main responsibilities:

* Load PDF and text files
* Split documents into chunks
* Generate embeddings
* Store vectors in FAISS
* Reload existing FAISS index on later runs

---

### `src/tool_registry.py`

Collects all available tools into a single registry.

Tools include:

```text
search_knowledge_base
get_stock_data
search_wikipedia
web_search
get_weather
```

---

### `src/tools/rag_tool.py`

Wraps the FAISS retriever as a LangChain-compatible tool.

Used for questions about:

* Uploaded PDFs
* Annual reports
* Internal documents
* Knowledge base content

---

### `src/tools/finance_tool.py`

Provides stock information through finance data sources.

Since free stock APIs may fail or rate-limit, the project includes safe fallback behavior.

---

### `src/response_formatter.py`

Formats the final answer and displays the tools used.

Example:

```text
Tools Used: search_knowledge_base, get_stock_data
```

---

## DeepSeek Configuration

This project uses DeepSeek through LangChain's OpenAI-compatible client.

Example configuration:

```python
from langchain_openai import ChatOpenAI

llm = ChatOpenAI(
    model=config["deepseek_model"],
    api_key=config["deepseek_api_key"],
    base_url=config["deepseek_base_url"],
    temperature=0,
)
```

Example `.env`:

```env
DEEPSEEK_API_KEY=your_deepseek_api_key_here
DEEPSEEK_MODEL=deepseek-v4-flash
DEEPSEEK_BASE_URL=https://api.deepseek.com
```

---

## Why Temperature is Set to 0

The model uses:

```python
temperature=0
```

This makes the output more deterministic.

That is useful for:

* Tool-based workflows
* Demo recording
* Debugging
* Preventing random format changes
* Reducing hallucinated tool behavior

---

## Handling Finance API Failures

During repeated local testing, finance APIs may return errors such as:

```text
429 Too Many Requests
404 Not Found
No valid price returned
```

Instead of hallucinating a stock price, the system returns a safe response:

```text
The get_stock_data tool was executed, but the external provider did not return usable live data.
```

This is intentional.

It shows the system is safe because it does not invent unavailable real-time values.

---

## Troubleshooting

| Problem                                                          | Cause                                         | Fix                                                               |
| ---------------------------------------------------------------- | --------------------------------------------- | ----------------------------------------------------------------- |
| `DEEPSEEK_API_KEY is required`                                   | Missing key in `.env`                         | Add your DeepSeek API key                                         |
| `Client.__init__() got an unexpected keyword argument 'proxies'` | Package version mismatch                      | Pin compatible `httpx`, `openai`, and `langchain-openai` versions |
| PDF not loading                                                  | Missing PDF parser                            | Install `pypdf`                                                   |
| FAISS reload error                                               | Old or corrupted FAISS index                  | Delete `data/knowledge_base/.faiss_index`                         |
| First run is slow                                                | Embedding model downloading                   | Wait once; later runs are faster                                  |
| Stock price unavailable                                          | Finance provider blocked/rate-limited request | Safe fallback will report unavailable                             |
| Web search unavailable                                           | Missing Tavily key                            | Add `TAVILY_API_KEY`                                              |
| Weather runs in mock mode                                        | Missing OpenWeatherMap key                    | Add `OPENWEATHERMAP_API_KEY`                                      |
| Tools Used shows `none`                                          | Agent only wrote fake tool text               | Use deterministic tool execution layer                            |
| DeepSeek writes tool calls as text                               | Old LangChain ReAct parser issue              | Execute tools in Python before final LLM synthesis                |

---

## Package Version Fix

If you get:

```text
Client.__init__() got an unexpected keyword argument 'proxies'
```

Run:

```bash
pip uninstall -y openai langchain-openai httpx
pip install "httpx==0.27.2" "openai==1.30.1" "langchain-openai==0.1.6"
```

Then verify:

```bash
pip show httpx openai langchain-openai
```

Expected:

```text
httpx==0.27.2
openai==1.30.1
langchain-openai==0.1.6
```

---

## Verify That Tools Are Actually Running

A correct run should show:

```text
Tools Used: search_knowledge_base, get_stock_data
```

If you see:

```text
Tools Used: none
```

then the model probably wrote fake tool calls instead of executing tools.

This project fixes that by executing tools directly in Python and passing real observations to DeepSeek.

---

## How to Add a Custom Tool

Adding a custom tool requires three steps:

1. Write the tool function
2. Wrap it as a LangChain `Tool`
3. Register it in the tool registry

---

### Step 1 — Create a tool file

Create:

```text
src/tools/my_tool.py
```

Example:

```python
from langchain.tools import Tool


def my_custom_function(input_str: str) -> str:
    try:
        return f"Processed input: {input_str}"
    except Exception as exc:
        return f"My tool failed: {exc}"


def create_my_tool() -> Tool:
    return Tool(
        name="my_custom_tool",
        func=my_custom_function,
        description=(
            "Use this tool when the user asks about custom project-specific data. "
            "Input should be a short natural-language query."
        ),
    )
```

---

### Step 2 — Register it

Open:

```text
src/tool_registry.py
```

Import your tool:

```python
from src.tools.my_tool import create_my_tool
```

Add it to the tools list:

```python
tools.append(create_my_tool())
```

---

### Step 3 — Add routing logic

In `main.py`, inside the deterministic tool execution function, add a condition:

```python
if "custom keyword" in q:
    result = tool_map["my_custom_tool"].run(query)

    observations.append(
        f"""Tool: my_custom_tool
Input: {query}
Observation:
{result}"""
    )

    tools_used.append("my_custom_tool")
```

---

## Demo Script

Use this flow in your demo video:

### Step 1 — Show the data folder

Show:

```text
data/knowledge_base/nvidia-2025-annual-report.pdf
```

Explain:

> This PDF is the internal knowledge base document. The system chunks it, embeds it, and stores it in FAISS.

---

### Step 2 — Run the command

```bash
python main.py --query "What does NVIDIA's annual report say about AI/data center growth, and what is NVDA's current stock price?"
```

---

### Step 3 — Explain the setup logs

Point out:

```text
DeepSeek connected
RAG Tool ready
Finance Tool ready
FAISS index loaded
```

---

### Step 4 — Explain the answer

Point out:

```text
NVIDIA Annual Report Summary
Current NVDA Stock Information
Tools Used: search_knowledge_base, get_stock_data
```

---

### Step 5 — Explain safety

Say:

> The system does not hallucinate a stock price when the external finance provider fails. It clearly says that the finance tool ran, but live data was unavailable.

---

## Limitations

* Free finance APIs can fail, rate-limit, or return delayed data
* Web search requires Tavily API key
* Weather requires OpenWeatherMap API key
* PDF quality affects retrieval quality
* Scanned PDFs may require OCR before indexing
* DeepSeek can summarize tool outputs well, but tool execution is handled in Python for reliability
* This project is designed for educational and demo purposes, not production trading or financial advice

---

## Final Summary

This project demonstrates a practical Agentic RAG system powered by DeepSeek.

It can:

* Search internal documents using FAISS
* Use real tools based on the user query
* Combine PDF knowledge with finance/tool outputs
* Avoid hallucinating unavailable live data
* Produce a final answer grounded in executed tool results

The core idea is simple:

```text
RAG gives the system memory.
Tools give the system access to the outside world.
DeepSeek turns both into a clear final answer.
```
