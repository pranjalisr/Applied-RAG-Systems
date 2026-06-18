# 🔍 Applied-RAG-Systems

A curated collection of **Retrieval-Augmented Generation (RAG)** system built in Python — ranging from a bare-bones implementation from scratch to production-style agentic and multimodal systems. Each project is self-contained and progressively more advanced.

---

## 📁 Project Structure

```
Applied-RAG-Systems/
├── 01-rag-from-scratch/       # Build RAG without any framework
├── 02-legal-ai-assistant/     # Domain-specific RAG for legal documents
├── 03-research-agent/         # Autonomous research agent with RAG
├── 04-multimodal-rag/         # RAG over text + images
└── 05-agentic-rag-realtime/   # Real-time agentic RAG pipeline
```

---

## 🚀 Projects

### 01 — RAG from Scratch

A ground-up implementation of Retrieval-Augmented Generation without relying on high-level frameworks like LangChain or LlamaIndex. This project focuses on understanding the core mechanics of RAG — document chunking, embedding, vector similarity search, and LLM-based generation — by building each component manually.

**Key concepts:** text chunking, embeddings, cosine similarity, in-context generation

---

### 02 — Legal AI Assistant

A domain-specific RAG system designed for querying and understanding legal documents. Upload legal texts, contracts, or case files and get precise, citation-grounded answers via a conversational interface.

**Key concepts:** domain-specific retrieval, PDF ingestion, document QA, answer grounding

---

### 03 — Research Agent

An autonomous research agent that combines RAG with tool-use and multi-step reasoning. Given a research question, the agent retrieves relevant information, synthesizes findings, and produces a structured output — mimicking how a human researcher would work.

**Key concepts:** agent loops, tool use, multi-step reasoning, summarization

---

### 04 — Multimodal RAG

Extends RAG beyond text to handle mixed-media documents containing both text and images. Enables queries that span visual and textual content, making it suitable for technical manuals, research papers with figures, or slide decks.

**Key concepts:** multimodal embeddings, image understanding, cross-modal retrieval

---

### 05 — Agentic RAG (Real-time)

A real-time agentic RAG pipeline where the system can autonomously plan retrieval steps, query live or dynamic data sources, and return up-to-date answers. Combines the reliability of RAG with the flexibility of agentic behavior.

**Key concepts:** agentic planning, real-time data, dynamic retrieval, streaming responses

---

## 🛠️ Tech Stack

| Component | Tools |
|---|---|
| Language | Python 3.10+ |
| Embeddings | OpenAI Embeddings / HuggingFace Sentence Transformers |
| Vector Store | FAISS / ChromaDB / Qdrant |
| LLM | OpenAI GPT / Gemini / local via Ollama |
| Frameworks | LangChain / LlamaIndex (where applicable) |
| UI | Streamlit (where applicable) |

---

## ⚙️ Getting Started

### Prerequisites

- Python 3.10 or higher
- An OpenAI API key (or equivalent LLM provider key)

### Installation

1. Clone the repository:
   ```bash
   git clone https://github.com/pranjalisr/Applied-RAG-Systems.git
   cd RAG-Projects
   ```

2. Navigate to any project folder:
   ```bash
   cd 01-rag-from-scratch
   ```

3. Install dependencies:
   ```bash
   pip install -r requirements.txt
   ```

4. Set up your environment variables:
   ```bash
   cp .env.example .env
   # Edit .env and add your API keys
   ```

5. Run the project:
   ```bash
   python main.py
   # or for Streamlit apps:
   streamlit run app.py
   ```

> Each project folder contains its own `README.md` with project-specific setup instructions.

---

## 🧠 What is RAG?

**Retrieval-Augmented Generation (RAG)** is a technique that enhances LLMs by grounding their responses in external, retrieved knowledge rather than relying solely on training data. The pipeline works in three stages:

```
Query → Retrieve relevant documents → Augment LLM prompt → Generate grounded response
```

This reduces hallucinations, enables the model to answer questions about private or recent data, and provides citation-level traceability.

---

## 📌 Roadmap

- [x] RAG from scratch
- [x] Legal AI assistant
- [x] Research agent
- [x] Multimodal RAG
- [x] Agentic RAG with real-time retrieval
- [ ] RAG evaluation framework (RAGAS integration)
- [ ] GraphRAG implementation
- [ ] Production deployment guide

---


## 📄 License

This project is licensed under the MIT License — see the [LICENSE](LICENSE) file for details.

---


