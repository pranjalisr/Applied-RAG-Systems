# 03 — Research Agent

> **"Like a research assistant who can look up papers, take notes, compare findings, and generate research insights — rather than just answering one question."**

This project is an **agentic research assistant** built on top of Retrieval-Augmented Generation (RAG). It reads research papers from PDFs, extracts structured metadata, builds a FAISS vector index, and uses an LLM-powered agent to search, summarize, compare, and generate answers grounded in the uploaded papers.

This version uses **DeepSeek** as the reasoning model instead of OpenAI.

---

## What is an AI Agent?

A regular LLM call is a simple:

```text
prompt → response
```

You give the model some input, and it gives one answer.

An **AI agent** is different. It has access to **tools** — functions it can call to search papers, summarize metadata, compare findings, or generate reports. The agent decides dynamically which tool to use based on the question.

Think of the difference between:

* 🤖 **Simple LLM**: You ask, “What does this paper say about RAG?” and the model answers from memory or from a fixed prompt.
* 🕵️ **Research Agent**: You ask the same question, and the agent searches the indexed PDFs, reads relevant chunks, checks paper metadata, and then synthesizes an answer from the uploaded documents.

---

## How This Differs from Simple RAG

| Simple RAG                             | Research Agent                                     |
| -------------------------------------- | -------------------------------------------------- |
| Embed documents into a vector database | Same                                               |
| User query retrieves nearest chunks    | Agent decides which tool to call                   |
| One retrieval step                     | Multi-step: search → summarize → compare → answer  |
| Fixed pipeline                         | Dynamic reasoning loop                             |
| Good for direct Q&A                    | Better for synthesis, comparison, and gap analysis |
| No tool planning                       | Agent plans actions using tools                    |

In simple RAG, the flow is fixed:

```text
retrieve → generate
```

In this project, the agent can reason step by step:

```text
search papers → inspect metadata → summarize paper → compare findings → generate final answer
```

---

## The ReAct Loop Explained

This project uses the **ReAct** pattern.

**ReAct = Reason + Act**

The agent alternates between thinking and using tools:

```text
Thought: I need to find papers that discuss RAG techniques.
Action: search_papers
Action Input: RAG techniques retrieval augmented generation

Observation: Found relevant chunks from "Retrieval-Augmented Generation for Large Language Models".

Thought: I should summarize the relevant paper to identify the major techniques.
Action: summarize_paper
Action Input: Retrieval-Augmented Generation for Large Language Models: A Survey

Observation: The paper discusses Naive RAG, Advanced RAG, Modular RAG, retrieval, generation, and augmentation.

Thought: I now have enough information.
Final Answer: The main RAG techniques are...
```

Each **Observation** becomes part of the agent’s context. The agent uses that context to decide the next step.

---

## Architecture

```text
┌─────────────────────────────────────────────────────┐
│                    main.py                          │
│  CLI entry point                                    │
│  --query / --interactive / --report                 │
└──────────────────────┬──────────────────────────────┘
                       │
                       ▼
┌─────────────────────────────────────────────────────┐
│                Research Agent                       │
│        ReAct loop + DeepSeek reasoning model        │
└───────────────┬───────────────┬─────────────────────┘
                │               │
                │               │
                ▼               ▼
     ┌────────────────┐   ┌──────────────────┐
     │ search_papers  │   │ summarize_paper  │
     │ FAISS semantic │   │ Paper metadata   │
     │ search tool    │   │ lookup tool      │
     └───────┬────────┘   └─────────┬────────┘
             │                      │
             ▼                      ▼
┌─────────────────────┐    ┌──────────────────────────┐
│    FAISS Index      │    │   PaperMetadata objects  │
│ Chunked PDF text +  │    │ title, authors, abstract │
│ metadata            │    │ methodology, findings    │
└─────────────────────┘    └──────────────────────────┘

Additional report flow:

paper metadata
      ↓
gap_analyzer.py
      ↓
LLM synthesis
      ↓
report_generator.py
      ↓
Markdown report
```

---

## Tech Stack

| Component             | Tool / Library                                         |
| --------------------- | ------------------------------------------------------ |
| Language              | Python                                                 |
| LLM                   | DeepSeek                                               |
| LLM Client            | `langchain-openai` with DeepSeek OpenAI-compatible API |
| Agent Framework       | LangChain                                              |
| Vector Database       | FAISS                                                  |
| Embeddings            | SentenceTransformers                                   |
| PDF Parsing           | PyPDF                                                  |
| Environment Variables | python-dotenv                                          |
| Schema Validation     | Pydantic                                               |

---

## Project Structure

```text
03-research-agent/
├── data/
│   └── papers/
│       ├── AttentionIsAllYouNeed.pdf
│       ├── Pre-training of Deep Bidirectional Transformers for Language Understanding.pdf
│       └── Retrieval-Augmented Generation for Large Language Models.pdf
├── reports/
│   └── rag_research_report.md
├── src/
│   ├── paper_parser.py
│   ├── paper_indexer.py
│   ├── research_agent.py
│   ├── gap_analyzer.py
│   └── report_generator.py
├── .env
├── .env.example
├── requirements.txt
├── main.py
└── README.md
```

---

## Setup

### 1. Clone the Repository

```bash
git clone https://github.com/pranjalisr/Applied-RAG-Systems.git
cd Applied-RAG-Systems/03-research-agent
```

---

### 2. Create a Virtual Environment

```bash
python -m venv venv
```

Activate it:

```bash
source venv/bin/activate
```

For Windows:

```bash
venv\Scripts\activate
```

---

### 3. Install Dependencies

```bash
python -m pip install --upgrade pip
pip install -r requirements.txt
```

Recommended compatible versions:

```txt
langchain==0.1.20
langchain-community==0.0.38
langchain-openai==0.1.6
openai==1.30.1
httpx==0.27.2
faiss-cpu==1.8.0
sentence-transformers==2.7.0
pypdf==4.2.0
python-dotenv==1.0.1
pydantic==2.7.1
arxiv==2.1.0
```

If you see an error like this:

```text
Client.__init__() got an unexpected keyword argument 'proxies'
```

reinstall the compatible packages:

```bash
pip uninstall -y openai langchain langchain-community langchain-openai httpx

pip install \
  langchain==0.1.20 \
  langchain-community==0.0.38 \
  langchain-openai==0.1.6 \
  openai==1.30.1 \
  httpx==0.27.2
```

---

### 4. Configure DeepSeek

Create a `.env` file:

```bash
cp .env.example .env
```

Add your DeepSeek credentials:

```env
DEEPSEEK_API_KEY=your_deepseek_api_key_here
DEEPSEEK_BASE_URL=https://api.deepseek.com
DEEPSEEK_MODEL=deepseek-v4-flash

PAPERS_DIR=data/papers
```

If your model name gives an API error, try:

```env
DEEPSEEK_MODEL=deepseek-chat
```

---

## DeepSeek Integration

This project uses `ChatOpenAI` from `langchain-openai`, but points it to DeepSeek’s OpenAI-compatible API.

In `main.py`, the LLM is initialized like this:

```python
llm = ChatOpenAI(
    model=args.model,
    temperature=0,
    openai_api_key=os.environ["DEEPSEEK_API_KEY"],
    openai_api_base=os.getenv("DEEPSEEK_BASE_URL", "https://api.deepseek.com"),
)
```

The CLI model argument uses DeepSeek by default:

```python
parser.add_argument(
    "--model",
    default=os.getenv("DEEPSEEK_MODEL", "deepseek-v4-flash"),
    metavar="MODEL",
    help="DeepSeek model name",
)
```

---

## How to Add Papers

Place research PDFs inside:

```text
data/papers/
```

Example:

```text
data/papers/
├── AttentionIsAllYouNeed.pdf
├── Pre-training of Deep Bidirectional Transformers for Language Understanding.pdf
└── Retrieval-Augmented Generation for Large Language Models.pdf
```

The pipeline will:

1. Extract text from each PDF.
2. Ask the LLM to extract metadata:

   * title
   * authors
   * year
   * abstract
   * methodology
   * key findings
   * limitations
3. Split papers into chunks.
4. Generate embeddings.
5. Store chunks in a FAISS vector index.
6. Make the indexed content available to the research agent.

---

## Recommended PDFs for This Demo

For this project, use papers that are topically related. Since this is a RAG research agent, the best starter set is:

| Paper                                                                              | Why Use It                                     |
| ---------------------------------------------------------------------------------- | ---------------------------------------------- |
| `Retrieval-Augmented Generation for Large Language Models`                         | Main RAG survey paper; best for RAG techniques |
| `Attention Is All You Need`                                                        | Foundational Transformer paper                 |
| `BERT: Pre-training of Deep Bidirectional Transformers for Language Understanding` | Important NLP foundation paper                 |

For the strongest demo, ask questions mainly around the RAG survey paper, because the Transformer and BERT papers are background papers and do not directly discuss RAG techniques.

---

## Source of PDFs

You can download papers from **arXiv**, which provides free access to research papers.

Suggested sources:

```text
Retrieval-Augmented Generation for Large Language Models: A Survey
arXiv: https://arxiv.org/abs/2312.10997

Attention Is All You Need
arXiv: https://arxiv.org/abs/1706.03762

BERT: Pre-training of Deep Bidirectional Transformers for Language Understanding
arXiv: https://arxiv.org/abs/1810.04805
```

Download the PDF versions and place them in:

```text
data/papers/
```

---

## Tips for Choosing PDFs

* Use **3–10 related papers** for best results.
* Use PDFs with selectable text.
* Avoid scanned PDFs unless OCR has already been applied.
* Keep papers in the same research area for better comparison.
* For gap analysis, avoid mixing unrelated topics.

Good topic groups:

```text
RAG papers
Transformer papers
BERT / NLP fine-tuning papers
Medical AI papers
Multimodal RAG papers
Agentic RAG papers
```

Bad topic mix:

```text
One RAG paper + one chemistry paper + one finance paper
```

The agent can still run, but the comparison will be weak.

---

## Running the Agent

### Ask a Single Question

```bash
python main.py --papers-dir data/papers --query "What are the main RAG techniques discussed across these papers?"
```

Expected output:

```text
[main] Found 3 PDF file(s) in 'data/papers'.
[main] Using model: deepseek-v4-flash

[main] === Step 1/3: Parsing papers ===
[paper_parser] Parsed 3 paper(s).

[main] === Step 2/3: Indexing papers in FAISS ===
[paper_indexer] Embedding 279 total chunks...
[paper_indexer] Index saved to 'papers_faiss_index'.

[main] === Step 3/3: Building research agent ===
[main] Agent ready.

Final Answer:
...
```

---

### Better Grounded Demo Query

Because only the RAG survey paper directly discusses RAG, this query is cleaner:

```bash
python main.py --papers-dir data/papers --query "Using only the uploaded PDFs, identify which paper discusses RAG techniques and summarize the main techniques from that paper."
```

---

### Interactive Mode

```bash
python main.py --papers-dir data/papers --interactive
```

Example questions:

```text
What does the RAG survey paper say about Naive RAG?
How is Advanced RAG different from Modular RAG?
Which uploaded paper discusses retrieval techniques?
Summarize the BERT paper.
Compare the Transformer paper and BERT paper.
```

Type this to exit:

```text
exit
```

---

### Generate a Gap Analysis Report

```bash
mkdir -p reports

python main.py \
  --papers-dir data/papers \
  --topic "Retrieval-Augmented Generation and Transformer-based NLP" \
  --report \
  --output reports/rag_research_report.md
```

Open the report:

```bash
cat reports/rag_research_report.md
```

or in VS Code:

```bash
code reports/rag_research_report.md
```

---

## Sample Queries

Use these to test the agent:

```text
"What are the main RAG techniques discussed across these papers?"

"Using only the uploaded PDFs, which paper discusses RAG techniques?"

"What is the difference between Naive RAG, Advanced RAG, and Modular RAG?"

"What retrieval methods are mentioned in the RAG survey?"

"Summarize Attention Is All You Need."

"Summarize the BERT paper."

"Compare Attention Is All You Need and BERT."

"What are the main research gaps in the RAG survey?"

"What are the key findings across all uploaded papers?"
```

---

## Example Output

For the query:

```bash
python main.py --papers-dir data/papers --query "What are the main RAG techniques discussed across these papers?"
```

Example final answer:

```text
The main RAG techniques discussed are primarily covered in the survey
"Retrieval-Augmented Generation for Large Language Models".

The techniques can be grouped into:

1. Naive RAG
   - Basic retrieve-then-generate pipeline.
   - Documents are indexed, relevant passages are retrieved, and the retrieved context is passed to the LLM.

2. Advanced RAG
   - Improves retrieval and generation quality.
   - Includes query rewriting, query expansion, chunk optimization, reranking, filtering, and fusion.

3. Modular RAG
   - Uses flexible interchangeable modules.
   - Can include search modules, memory modules, verification modules, and fusion modules.

Other techniques include sparse retrieval, dense retrieval, hybrid retrieval,
iterative retrieval, adaptive retrieval, and prompt engineering with retrieved context.
```

---

## How to Interpret the Gap Analysis

The gap analysis report usually contains:

| Section              | Meaning                                                    |
| -------------------- | ---------------------------------------------------------- |
| Common Themes        | Ideas that appear across multiple papers                   |
| Contradictions       | Places where papers disagree                               |
| Missing Experiments  | Experiments that could have been done but were not covered |
| Missing Populations  | Understudied groups, languages, domains, or datasets       |
| Methodological Gaps  | Missing methods or evaluation approaches                   |
| Suggested Next Steps | Research directions generated from the analysis            |

> ⚠️ Always verify the output manually. LLMs can generate plausible but incorrect research gaps. Treat the report as a first draft, not a final academic conclusion.

---

## Common Errors and Fixes

### 1. No PDF files found

Error:

```text
No PDF files found in data/papers
```

Fix:

```bash
mkdir -p data/papers
```

Then add PDFs:

```text
data/papers/paper_name.pdf
```

---

### 2. DeepSeek API key missing

Error:

```text
DEEPSEEK_API_KEY environment variable is not set
```

Fix `.env`:

```env
DEEPSEEK_API_KEY=your_deepseek_api_key_here
DEEPSEEK_BASE_URL=https://api.deepseek.com
DEEPSEEK_MODEL=deepseek-v4-flash
```

---

### 3. `proxies` keyword argument error

Error:

```text
Client.__init__() got an unexpected keyword argument 'proxies'
```

Fix:

```bash
pip uninstall -y openai langchain langchain-community langchain-openai httpx

pip install \
  langchain==0.1.20 \
  langchain-community==0.0.38 \
  langchain-openai==0.1.6 \
  openai==1.30.1 \
  httpx==0.27.2
```

---

### 4. Metadata validation warning

Example:

```text
Input should be a valid list
```

This happens when the LLM returns a string or `None` where the schema expects a list.

Fix in `src/paper_parser.py`:

```python
authors: list[str] | str = Field(
    default_factory=list,
    description="List of author names",
)

limitations: list[str] | str | None = Field(
    default=None,
    description="Limitations acknowledged by the authors",
)
```

---

### 5. LangChain deprecation warnings

Warnings:

```text
initialize_agent was deprecated
Chain.run was deprecated
```

These are not errors. The project still runs.

To hide warnings during demo:

```bash
PYTHONWARNINGS="ignore" python main.py --papers-dir data/papers --query "Using only the uploaded PDFs, identify which paper discusses RAG techniques and summarize the main techniques from that paper."
```

---

### 6. Agent format warning

Warning:

```text
Invalid Format: Missing 'Action:' after 'Thought:'
```

This can happen because DeepSeek sometimes answers directly instead of following the old ReAct format.

Fix in `src/research_agent.py` where `initialize_agent(...)` is called:

```python
agent = initialize_agent(
    tools=tools,
    llm=llm,
    agent=AgentType.ZERO_SHOT_REACT_DESCRIPTION,
    verbose=False,
    handle_parsing_errors=True,
    max_iterations=5,
    early_stopping_method="generate",
)
```

---

## Limitations

1. **The answer depends on uploaded PDFs**
   If only one paper discusses RAG, the answer will mostly come from that paper.

2. **LLMs can still hallucinate**
   The agent is grounded in retrieved chunks, but it may still make unsupported claims. Always verify important research claims against the source PDF.

3. **PDF extraction quality varies**
   Multi-column papers, scanned PDFs, equations, and figures may not parse perfectly.

4. **Metadata extraction can be inconsistent**
   LLMs may return `authors` as a string, list, or incomplete data. The schema has been made flexible to handle this.

5. **Best with related papers**
   The agent works best when all PDFs are from the same research area.

6. **Old LangChain agent API produces warnings**
   The current version works, but future versions should migrate from `initialize_agent` to newer constructors like `create_react_agent`.

---

## How to Extend

### Add a New Tool

1. Create a new file:

```text
src/tools/my_tool.py
```

2. Define a tool function:

```python
from langchain.tools import Tool

def my_tool_function(query: str) -> str:
    return "Tool result"

def create_my_tool() -> Tool:
    return Tool(
        name="my_tool",
        func=my_tool_function,
        description="Use this tool when the agent needs to..."
    )
```

3. Import it inside the research agent file.

4. Add it to the tools list passed to `initialize_agent`.

The agent can then choose to use the tool based on the tool description.

---

## Ideas for Future Tools

* **Citation Tool** — generate BibTeX entries from paper metadata.
* **Timeline Tool** — show how papers evolved over time.
* **Keyword Tool** — extract top recurring keywords across all PDFs.
* **Dataset Tool** — identify datasets used across papers.
* **Method Comparison Tool** — compare methodology across selected papers.
* **ArXiv Tool** — search arXiv for newer related papers.
* **PDF Section Tool** — retrieve specific sections like abstract, methodology, results, or limitations.

---

## Final Demo Command

For the cleanest demo, run:

```bash
PYTHONWARNINGS="ignore" python main.py \
  --papers-dir data/papers \
  --query "Using only the uploaded PDFs, identify which paper discusses RAG techniques and summarize the main techniques from that paper."
```

Expected result:

```text
The uploaded paper that directly discusses RAG techniques is
"Retrieval-Augmented Generation for Large Language Models".

The main techniques include:
- Naive RAG
- Advanced RAG
- Modular RAG
- Query rewriting
- Query expansion
- Sparse retrieval
- Dense retrieval
- Hybrid retrieval
- Reranking
- Filtering
- Fusion
- Iterative retrieval
- Adaptive retrieval
- Evaluation using retrieval and generation quality metrics
```

---

## Summary

This project demonstrates an **agentic RAG system for research papers**.

It can:

* Parse research PDFs.
* Extract structured metadata.
* Build a FAISS vector index.
* Search relevant paper chunks.
* Summarize individual papers.
* Compare findings across papers.
* Generate gap analysis reports.
* Use DeepSeek as the LLM reasoning engine.

Instead of being a fixed retrieve-and-answer chatbot, this system behaves more like a lightweight research assistant that can decide which tools to use while answering complex research questions.
