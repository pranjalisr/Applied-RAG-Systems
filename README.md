# 🔍 Applied-RAG-Systems

A curated collection of **Retrieval-Augmented Generation (RAG)** systems built in Python — starting from a simple RAG pipeline built from scratch and gradually moving toward domain-specific, agentic, multimodal, and real-time RAG applications.

This repository is designed as a practical learning path for understanding how modern RAG systems are built, improved, evaluated, and extended for real-world GenAI use cases.

---

## 📌 Overview

Retrieval-Augmented Generation, commonly called **RAG**, helps Large Language Models answer questions using external knowledge instead of relying only on their training data.

This repository contains five progressively advanced RAG projects:

1. **RAG from Scratch** — understand the core mechanics of RAG without frameworks.
2. **Legal AI Assistant** — apply RAG to contracts and legal documents.
3. **Research Agent** — combine retrieval with agentic reasoning.
4. **Multimodal RAG** — retrieve and answer from text, images, and tables.
5. **Agentic RAG with Real-Time Tools** — let an agent choose between internal documents and live tools.

Each project is self-contained and includes its own setup, dependencies, and README.

---

## 📁 Project Structure

```bash
Applied-RAG-Systems/
├── 01-rag-from-scratch/       # Build RAG without high-level frameworks
├── 02-legal-ai-assistant/     # Contract/legal document analysis with RAG
├── 03-research-agent/         # Agentic research assistant with tool use
├── 04-multimodal-rag/         # RAG over text, images, and tables
├── 05-agentic-rag-realtime/   # Agentic RAG with real-time tools
├── .gitignore
└── README.md
```

---

## 🚀 Projects

## 01 — RAG from Scratch

A bare-bones implementation of Retrieval-Augmented Generation built without relying on high-level RAG frameworks like LangChain or LlamaIndex.

This project focuses on understanding the internal mechanics of RAG by manually implementing the core pipeline: loading documents, splitting text into chunks, generating embeddings, performing similarity search, and passing retrieved context to an LLM for grounded answer generation.

### What It Does

- Loads and processes source documents
- Splits documents into smaller text chunks
- Converts chunks into embeddings
- Stores and searches embeddings using vector similarity
- Retrieves the most relevant chunks for a user query
- Sends retrieved context to an LLM
- Generates answers grounded in the retrieved content

### Key Concepts

- Document loading
- Text chunking
- Embeddings
- Cosine similarity
- Vector search
- Prompt construction
- Grounded generation

### Why This Project Matters

Before using advanced RAG frameworks, it is important to understand what happens internally. This project explains the foundation of RAG by building every major step manually.

---

## 02 — Legal AI Assistant

A domain-specific RAG system designed to help understand legal documents such as contracts, agreements, policies, and case files.

Instead of giving generic answers, the system retrieves relevant clauses from the uploaded legal document and generates responses grounded in the actual document text. It can summarize contracts, extract important clauses, flag risks, detect conflicting terms, and answer legal-document questions with citations.

> ⚠️ **Disclaimer:** This project is for educational purposes only and does not provide legal advice. Always consult a qualified legal professional before making legal or business decisions.

### What It Does

- Parses legal documents such as PDF and DOCX files
- Splits legal text into searchable chunks
- Indexes documents into a vector store
- Extracts important clauses such as:
  - Termination
  - Indemnification
  - Confidentiality
  - Limitation of liability
  - Governing law
  - Intellectual property ownership
- Flags potential risk areas in plain English
- Detects possible internal conflicts between clauses
- Answers user questions using retrieved legal context

### Key Concepts

- Domain-specific RAG
- Legal document parsing
- Clause extraction
- Risk analysis
- Conflict detection
- Citation-grounded question answering

### Why This Project Matters

Legal documents are long, dense, and easy to misread. This project shows how RAG can be adapted for a specialized domain where grounding, traceability, and careful language are important.

---

## 03 — Research Agent

An autonomous research assistant that combines Retrieval-Augmented Generation with agentic reasoning and tool use.

Unlike simple RAG, which performs one retrieval step and generates an answer, this project behaves more like a research workflow. It can break a question into smaller steps, retrieve relevant information, reason over it, and produce a structured research-style response.

### What It Does

- Accepts a research question from the user
- Searches relevant internal documents
- Uses tools when needed
- Performs multi-step reasoning
- Synthesizes retrieved information into a structured answer
- Produces research-style summaries and comparisons

### Key Concepts

- AI agents
- Agent loops
- Tool use
- Multi-step reasoning
- Retrieval planning
- Research synthesis
- Summarization

### Why This Project Matters

Real research questions often need more than one retrieval step. This project demonstrates how RAG can be extended into an agentic workflow where the system plans, searches, reasons, and writes a more complete answer.

---

## 04 — Multimodal RAG

A Retrieval-Augmented Generation system that works with more than plain text.

Traditional RAG pipelines often fail when important information is stored inside images, charts, diagrams, screenshots, or tables. This project extends RAG to handle mixed-content documents by processing text, images, and tables separately, then routing the user query to the most relevant content type.

### What It Does

- Extracts text from PDF documents
- Detects and processes images or diagrams
- Extracts and summarizes tables
- Creates searchable representations for multiple modalities
- Routes questions to the right content type
- Answers using text, image descriptions, or table summaries
- Supports technical PDFs, research papers, manuals, and reports

### Key Concepts

- Multimodal retrieval
- Image understanding
- Table extraction
- Cross-modal search
- PDF intelligence
- Modality-aware routing
- RAG over mixed documents

### Why This Project Matters

Most real-world documents are not text-only. They contain charts, tables, diagrams, and screenshots. This project shows how RAG can be expanded to understand documents more completely.

---

## 05 — Agentic RAG with Real-Time Tools

An advanced RAG system where an LLM agent decides which tool to use based on the user’s question.

Instead of always searching the internal vector database, the agent can choose between internal document retrieval, live web search, Wikipedia lookup, stock price tools, weather tools, or other real-time data sources. This makes the system useful for questions that combine private knowledge with fresh external information.

### What It Does

- Accepts a user question
- Decides which tool is most appropriate
- Searches internal documents when the question depends on stored knowledge
- Uses live tools when the question needs current data
- Combines retrieved context with real-time outputs
- Generates a final answer using dynamic multi-step reasoning

### Key Concepts

- Agentic RAG
- Tool routing
- Real-time data retrieval
- Dynamic planning
- Live web search
- Wikipedia lookup
- Weather tools
- Stock/data tools
- Multi-tool reasoning

### Why This Project Matters

Standard RAG works well for static documents, but real users often ask questions that need both internal knowledge and live information. This project demonstrates how RAG can be combined with real-time tools to build more flexible GenAI systems.

---

## 🧠 Learning Progression

This repository is structured as a step-by-step RAG learning path.

| Project | Focus | Difficulty |
|---|---|---|
| 01 — RAG from Scratch | Understand core RAG internals | Beginner |
| 02 — Legal AI Assistant | Apply RAG to a specialized domain | Intermediate |
| 03 — Research Agent | Add agentic reasoning and tool use | Intermediate |
| 04 — Multimodal RAG | Retrieve from text, images, and tables | Advanced |
| 05 — Agentic RAG Realtime | Combine RAG with live tools | Advanced |

---

## 🧩 Standard RAG vs Advanced RAG Systems

| Type | How It Works | Limitation Solved |
|---|---|---|
| Standard RAG | Retrieves relevant chunks and generates an answer | Reduces hallucination |
| Domain-Specific RAG | Uses specialized parsing, prompts, and retrieval logic | Improves accuracy for domain documents |
| Agentic RAG | Lets the LLM decide the next action or tool | Handles multi-step questions |
| Multimodal RAG | Retrieves from text, images, and tables | Works with real-world mixed documents |
| Real-Time RAG | Uses live tools and external APIs | Answers questions that need fresh data |

---

## 🛠️ Tech Stack

| Component | Tools Used |
|---|---|
| Language | Python |
| Embeddings | OpenAI Embeddings, HuggingFace Sentence Transformers |
| Vector Stores | FAISS, ChromaDB |
| LLMs | OpenAI, Gemini, DeepSeek, local models via Ollama |
| RAG / Agent Frameworks | LangChain, LangGraph where applicable |
| Document Processing | PyPDF, PDFPlumber, Unstructured, Python utilities |
| UI | Streamlit where applicable |
| Real-Time Tools | Web search, Wikipedia, weather, stock/data APIs |

---

## ⚙️ Getting Started

### Prerequisites

Make sure you have the following installed:

- Python 3.10 or higher
- Git
- pip
- A virtual environment tool such as `venv`
- API keys for the LLM provider you want to use

---

## 1. Clone the Repository

```bash
git clone https://github.com/pranjalisr/Applied-RAG-Systems.git
cd Applied-RAG-Systems
```

---

## 2. Create a Virtual Environment

```bash
python -m venv venv
```

Activate it:

```bash
# macOS / Linux
source venv/bin/activate
```

```bash
# Windows
venv\Scripts\activate
```

---

## 3. Navigate to a Project

Example:

```bash
cd 01-rag-from-scratch
```

---

## 4. Install Dependencies

```bash
pip install -r requirements.txt
```

---

## 5. Configure Environment Variables

Create a `.env` file inside the selected project folder:

```bash
cp .env.example .env
```

Then add the required API keys.

Example:

```env
OPENAI_API_KEY=your_openai_api_key
GOOGLE_API_KEY=your_gemini_api_key
DEEPSEEK_API_KEY=your_deepseek_api_key
SERPAPI_API_KEY=your_serpapi_key
```

Not every project needs every key. Check the individual project README for the exact environment variables required.

---

## 6. Run the Project

For normal Python projects:

```bash
python main.py
```

For Streamlit apps:

```bash
streamlit run app.py
```

For agent-based projects, the command may look like:

```bash
python agent.py
```

Each project folder contains its own README with exact setup and run instructions.

---

## 🧠 What is RAG?

**Retrieval-Augmented Generation**, or **RAG**, is a technique that improves LLM responses by giving the model relevant external context before it answers.

Instead of asking the LLM to rely only on what it already knows, RAG first searches your documents or data source, retrieves useful context, and then asks the LLM to answer using that context.

The basic flow looks like this:

```text
User Query
   ↓
Retrieve Relevant Documents
   ↓
Add Retrieved Context to Prompt
   ↓
Generate Grounded Answer
```

This makes LLM applications more reliable because the answers are based on actual source material.

---

## ✅ Why RAG Matters

RAG is useful because it helps solve some of the biggest limitations of LLMs:

- Reduces hallucinations
- Allows LLMs to answer from private documents
- Supports recent or frequently updated data
- Adds traceability through retrieved sources
- Makes GenAI systems more useful for real-world business and research workflows
- Helps build domain-specific AI assistants
- Improves trust by grounding outputs in retrieved evidence

---

## 📌 Use Cases Covered

This repository demonstrates RAG patterns for:

- Document question answering
- Legal document analysis
- Contract clause extraction
- Risk detection in agreements
- Research summarization
- Multi-step agent reasoning
- PDF understanding
- Image-aware retrieval
- Table-aware retrieval
- Real-time information retrieval
- Tool-using AI agents

---

## 📊 Project Comparison

| Project | Data Type | Retrieval Type | Agentic | Real-Time | Multimodal |
|---|---|---|---|---|---|
| RAG from Scratch | Text | Vector similarity | No | No | No |
| Legal AI Assistant | Legal documents | Domain-specific vector search | No | No | No |
| Research Agent | Text / research docs | Multi-step retrieval | Yes | Optional | No |
| Multimodal RAG | Text, images, tables | Cross-modal retrieval | Partially | No | Yes |
| Agentic RAG Realtime | Documents + live data | Tool-routed retrieval | Yes | Yes | Optional |

---

## 🧪 What I Learned

Through these projects, this repository explores:

- How embeddings represent document meaning
- Why chunk size and chunk overlap affect retrieval quality
- How vector databases power semantic search
- How prompt construction changes answer quality
- Why domain-specific RAG needs better parsing and grounding
- How agents decide which tools to call
- How multimodal systems handle non-text information
- How real-time tools make RAG more useful for dynamic questions
- How to build practical GenAI systems beyond simple chatbot demos

---

## 🗺️ Roadmap

- [x] RAG from scratch
- [x] Legal AI assistant
- [x] Research agent
- [x] Multimodal RAG
- [x] Agentic RAG with real-time tools
- [ ] Add RAG evaluation using RAGAS
- [ ] Add GraphRAG implementation
- [ ] Add production deployment guide
- [ ] Add Docker setup for all projects
- [ ] Add CI workflow for testing examples
- [ ] Add comparison notebook for retrieval quality

---

## 🤝 Contributions

This repository is mainly built as a learning and portfolio project, but suggestions, improvements, and issue reports are welcome.

You can contribute by:

- Improving documentation
- Adding new RAG examples
- Fixing bugs
- Adding evaluation scripts
- Improving UI or deployment setup
- Adding tests for retrieval and generation quality

---

## 📄 License

This project is licensed under the MIT License.

---

## ⭐ Final Note

RAG is not just about connecting a vector database to an LLM.

A good RAG system needs thoughtful chunking, clean document processing, strong retrieval, grounded prompting, evaluation, and sometimes agentic decision-making.

This repository explores that journey step by step — from the simplest version of RAG to more practical and production-style systems.
