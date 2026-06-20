# RAG from Scratch 🔍

A beginner-friendly implementation of Retrieval-Augmented Generation (RAG) built step-by-step using LangChain, FAISS, HuggingFace embeddings, and DeepSeek. Every file is structured to explain not only *what* the code does, but also *why* each step is needed in a real RAG pipeline.

---

## What is RAG and Why Does It Matter?

**The problem with plain LLMs:**
Large Language Models can generate impressive answers, but they have limitations. They may hallucinate, rely on outdated knowledge, or fail to answer questions about private documents such as research papers, internal company policies, product manuals, or project-specific notes.

**What RAG does:**
Retrieval-Augmented Generation solves this by giving the LLM access to external documents at query time. Instead of retraining the model, the system retrieves the most relevant chunks from your documents and sends them to the LLM as context. The LLM then generates an answer grounded in those retrieved chunks.

**Why it matters:**
RAG is one of the most practical architectures for building production-ready AI Q&A systems. It is cost-effective, easy to update, and more transparent because the system can show which document chunks were used to generate the answer.

This project demonstrates the core RAG workflow from scratch:

```text
Load documents → Chunk text → Generate embeddings → Store in FAISS → Retrieve relevant chunks → Generate answer with DeepSeek
```

---

## Architecture

```text
YOUR DOCUMENTS (PDF / TXT / DOCX)
         │
         ▼
  ┌─────────────┐
  │  1. LOAD    │  Read files from disk into LangChain Document objects
  └──────┬──────┘
         │
         ▼
  ┌─────────────┐
  │  2. CHUNK   │  Split large documents into smaller overlapping chunks
  └──────┬──────┘
         │
         ▼
  ┌─────────────┐
  │  3. EMBED   │  Convert each chunk into vector embeddings
  └──────┬──────┘
         │
         ▼
  ┌─────────────┐
  │  4. INDEX   │  Store vectors in FAISS for similarity search
  └──────┬──────┘
         │
         │                    USER QUESTION
         │                         │
         │                         ▼
         │                  ┌─────────────┐
         │                  │  5. EMBED   │  Convert question into a vector
         │                  └──────┬──────┘
         │                         │
         ▼                         ▼
  ┌─────────────────────────────────────┐
  │         FAISS SIMILARITY SEARCH     │  Retrieve top-k relevant chunks
  └──────────────────┬──────────────────┘
                     │
                     ▼
             RELEVANT DOCUMENT CHUNKS
                     │
                     ▼
  ┌─────────────────────────────────────┐
  │     6. GENERATE (DeepSeek + Prompt) │  Generate grounded answer
  └──────────────────┬──────────────────┘
                     │
                     ▼
              SOURCE-GROUNDED ANSWER ✅
```

---

## Tech Stack

| Component        | Library / Tool                     | Purpose                                       |
| ---------------- | ---------------------------------- | --------------------------------------------- |
| Document loading | `langchain-community` loaders      | Read PDF, TXT, and DOCX files                 |
| Text splitting   | `RecursiveCharacterTextSplitter`   | Split documents into overlapping chunks       |
| Embeddings       | `sentence-transformers`            | Convert text into vector embeddings           |
| Vector database  | `faiss-cpu`                        | Store and search document embeddings          |
| LLM              | DeepSeek via OpenAI-compatible API | Generate final answers from retrieved context |
| Env management   | `python-dotenv`                    | Load API keys from `.env`                     |
| Orchestration    | LangChain                          | Connect retrieval and generation steps        |

---

## Step-by-Step Setup

### 1. Clone the repository

```bash
git clone https://github.com/pranjalisr/Applied-RAG-Systems.git
cd Applied-RAG-Systems/01-rag-from-scratch
```

---

### 2. Create and activate a virtual environment

Use **Python 3.11**.

Python 3.14 may cause dependency issues with `faiss-cpu`.

```bash
python3.11 -m venv .venv
source .venv/bin/activate
```

Verify the Python version:

```bash
python --version
```

Expected output:

```text
Python 3.11.x
```

---

### 3. Install dependencies

Upgrade pip:

```bash
python -m pip install --upgrade pip
```

Install all dependencies:

```bash
python -m pip install -r requirements.txt
```

> First install may take a few minutes because `faiss-cpu` and `sentence-transformers` are large packages.

---

## Required `requirements.txt`

Make sure your `requirements.txt` includes these dependencies:

```txt
langchain==0.1.20
langchain-community==0.0.38
langchain-openai==0.1.6
openai==1.30.5
httpx==0.27.2
faiss-cpu==1.8.0
sentence-transformers
python-dotenv
pypdf
docx2txt
```

The pinned versions of `openai`, `httpx`, and `langchain-openai` help avoid this error:

```text
Client.__init__() got an unexpected keyword argument 'proxies'
```

---

## Configure DeepSeek API

This project uses **DeepSeek instead of OpenAI** for answer generation.

Create a `.env` file:

```bash
cp .env.example .env
```

Open `.env` and add:

```env
DEEPSEEK_API_KEY=your_deepseek_api_key_here
DEEPSEEK_BASE_URL=https://api.deepseek.com
DEEPSEEK_MODEL=deepseek-chat
```

Do not wrap the values in quotes.

---

## DeepSeek Configuration in Code

In `src/generator.py`, make sure the required imports are present at the top:

```python
import os
from dotenv import load_dotenv

from langchain_openai import ChatOpenAI
```

Use DeepSeek through LangChain’s OpenAI-compatible client:

```python
load_dotenv()

llm = ChatOpenAI(
    model=os.getenv("DEEPSEEK_MODEL", "deepseek-chat"),
    api_key=os.getenv("DEEPSEEK_API_KEY"),
    base_url=os.getenv("DEEPSEEK_BASE_URL", "https://api.deepseek.com"),
    temperature=0,
)
```

DeepSeek is used only for **answer generation**.
The embeddings are still generated locally using HuggingFace sentence-transformers.

---

## Add Your Documents

Drop any `.pdf`, `.txt`, or `.docx` files into:

```text
data/sample_docs/
```

Example:

```text
data/sample_docs/RAG.pdf
```

For testing, this project was run using the paper:

```text
Retrieval-Augmented Generation for Large Language Models: A Survey
```

This PDF is a good test document because it contains explanations of RAG, Naive RAG, Advanced RAG, Modular RAG, indexing, retrieval, generation, evaluation, and future directions.

---

## Supported File Types

| File type | Support | Notes                                       |
| --------- | ------- | ------------------------------------------- |
| `.pdf`    | ✅       | Each page can be loaded as document content |
| `.txt`    | ✅       | Simple text files are supported             |
| `.docx`   | ✅       | Word documents are supported                |
| `.csv`    | ❌       | Not supported in this version               |

---

## Rebuild FAISS Index After Changing Documents

If you add, remove, or replace documents, delete the existing FAISS index:

```bash
rm -rf faiss_index
```

Then run the project again. The FAISS index will be rebuilt automatically.

---

## Run the Project

### Interactive mode

```bash
python main.py --model deepseek-chat
```

### Single question mode

```bash
python main.py --model deepseek-chat --question "According to the uploaded RAG survey paper, what problems do LLMs face, and how does Retrieval-Augmented Generation help solve them?"
```

### Debug mode

```bash
python main.py --debug --model deepseek-chat --question "According to the uploaded RAG survey paper, explain how RAG works."
```

Debug mode shows the retrieved chunks and the final prompt sent to the LLM. This is useful for checking whether the system is retrieving the correct document content.

---

## Recommended Questions for Testing

Use questions that are directly supported by your uploaded document.

### RAG importance

```bash
python main.py --model deepseek-chat --question "According to the uploaded RAG survey paper, what problems do LLMs face, and how does Retrieval-Augmented Generation help solve them?"
```

### RAG pipeline

```bash
python main.py --model deepseek-chat --question "According to Figure 2 in the uploaded RAG survey paper, explain how the RAG process works for question answering, including indexing, retrieval, and generation."
```

### RAG with external knowledge

```bash
python main.py --model deepseek-chat --question "According to the uploaded RAG survey paper, explain how Retrieval-Augmented Generation improves LLM answers by using external knowledge, document chunks, embeddings, vector search, and generation."
```

### RAG evolution

```bash
python main.py --model deepseek-chat --question "According to the uploaded RAG survey paper, explain how RAG evolved from Naive RAG to Advanced RAG and Modular RAG."
```

---

## How to Verify the LLM Uses Your Documents

This is the most important test for a RAG system.

### Step 1: Add a test document

Create a file:

```text
data/sample_docs/test.txt
```

Add this content:

```text
The Zorbax Protocol was established in 2019 by Dr. Eleanor Voss.
The protocol requires three phases: initialization, calibration, and review.
```

### Step 2: Rebuild the FAISS index

```bash
rm -rf faiss_index
```

### Step 3: Ask a question from the document

```bash
python main.py --model deepseek-chat --question "Who established the Zorbax Protocol?"
```

Expected answer:

```text
Dr. Eleanor Voss established the Zorbax Protocol in 2019.
```

### Step 4: Ask something not present in the documents

```bash
python main.py --model deepseek-chat --question "What is the capital of Australia?"
```

Expected answer:

```text
I don't know based on the provided documents.
```

If the model answers using general knowledge, check the prompt template in `src/generator.py`.

---

## Retriever Configuration

The retriever controls how many chunks are fetched from FAISS.

Example top-k retrieval:

```python
retriever = vectorstore.as_retriever(
    search_type="similarity",
    search_kwargs={"k": 8}
)
```

Here, `k=8` means the retriever sends the top 8 relevant chunks to the LLM.

If the retrieved chunks are repetitive or mostly from the same page, you can use MMR retrieval for more diverse results:

```python
retriever = vectorstore.as_retriever(
    search_type="mmr",
    search_kwargs={
        "k": 8,
        "fetch_k": 25,
        "lambda_mult": 0.5
    }
)
```

---

## Prompt Template

A strict prompt is used to reduce hallucinations:

```python
RAG_PROMPT_TEMPLATE = """
You are a helpful assistant. Answer the question based ONLY on the following context.
If the answer is not in the context, say "I don't know based on the provided documents."
Do not use your general knowledge.
Do not add a "What is not covered" section unless the user specifically asks for limitations.
Give a clean, structured answer with headings and bullet points.

Context:
{context}

Question: {question}

Answer:
"""
```

This keeps the answer grounded in the retrieved document chunks.

---

## Beginner Tips

### What happens if chunk size is too large or too small?

| Setting                | Effect                                                           |
| ---------------------- | ---------------------------------------------------------------- |
| `chunk_size` too large | Fewer chunks, but retrieval may include too much irrelevant text |
| `chunk_size` too small | Many tiny chunks, but each chunk may lack useful context         |
| Balanced chunk size    | Retrieves meaningful passages without overwhelming the LLM       |

---

### What does top-k mean?

`k` is the number of chunks retrieved for each question.

| Value   | Effect                                          |
| ------- | ----------------------------------------------- |
| `k=1`   | Very narrow context                             |
| `k=3`   | Small but focused context                       |
| `k=8`   | Better coverage for longer documents            |
| `k=10+` | More context, but may include irrelevant chunks |

If the answer says:

```text
I don't know based on the provided documents.
```

but you know the answer exists in the PDF, try:

1. Asking a more specific question.
2. Increasing `k`.
3. Using MMR retrieval.
4. Running with `--debug`.

---

## Troubleshooting

### `ModuleNotFoundError: No module named 'dotenv'`

Install `python-dotenv`:

```bash
python -m pip install python-dotenv
```

Also add this to `requirements.txt`:

```txt
python-dotenv
```

---

### `No matching distribution found for faiss-cpu==1.8.0`

This usually happens when using Python 3.14.

Fix:

```bash
deactivate
rm -rf .venv
python3.11 -m venv .venv
source .venv/bin/activate
python -m pip install -r requirements.txt
```

---

### `Client.__init__() got an unexpected keyword argument 'proxies'`

This is caused by incompatible package versions.

Fix:

```bash
python -m pip uninstall -y openai httpx langchain-openai
python -m pip install openai==1.30.5 httpx==0.27.2 langchain-openai==0.1.6
```

---

### DeepSeek API key not found

Check that `.env` contains:

```env
DEEPSEEK_API_KEY=your_deepseek_api_key_here
DEEPSEEK_BASE_URL=https://api.deepseek.com
DEEPSEEK_MODEL=deepseek-chat
```

Also make sure `load_dotenv()` is called before reading environment variables.

---

### Answer is too short or says “not covered”

This usually means the retrieved chunks did not contain enough relevant information.

Try:

```bash
python main.py --debug --model deepseek-chat --question "Your question here"
```

Then check:

1. Which pages were retrieved.
2. Whether the retrieved chunks actually contain the answer.
3. Whether the question is too broad.
4. Whether `k` should be increased.
5. Whether MMR retrieval should be used.

---

### Stale or wrong answers after changing documents

Delete the FAISS index and rebuild:

```bash
rm -rf faiss_index
python main.py --model deepseek-chat
```

---

## Example Output

Example command:

```bash
python main.py --model deepseek-chat --question "According to the uploaded RAG survey paper, explain how Retrieval-Augmented Generation improves LLM answers by using external knowledge, document chunks, embeddings, vector search, and generation."
```

Example output:

```text
PIPELINE READY — Let's ask some questions!

Question:
According to the uploaded RAG survey paper, explain how Retrieval-Augmented Generation improves LLM answers...

Answer:
Retrieval-Augmented Generation improves LLM answers by incorporating knowledge from external databases. Documents are split into chunks, encoded into vector embeddings, stored in a vector database, and retrieved based on semantic similarity. The retrieved chunks are combined with the user question and passed to the LLM to generate a grounded answer.

Sources used:
- data/sample_docs/RAG.pdf, page 2
- data/sample_docs/RAG.pdf, page 3
- data/sample_docs/RAG.pdf, page 4
```

---

## Project Structure

```text
01-rag-from-scratch/
├── README.md                 ← Project documentation
├── requirements.txt          ← Python dependencies
├── .env.example              ← Template for API keys
├── .env                      ← Local environment variables
├── main.py                   ← Entry point for the pipeline
├── faiss_index/              ← Cached FAISS vector index
├── data/
│   └── sample_docs/          ← Add your PDF, TXT, or DOCX files here
└── src/
    ├── __init__.py           ← Makes src a Python package
    ├── document_loader.py    ← Step 1: Load documents
    ├── chunker.py            ← Step 2: Split documents into chunks
    ├── embedder.py           ← Step 3: Generate embeddings
    ├── vector_store.py       ← Step 4: Store vectors in FAISS
    ├── retriever.py          ← Step 5: Retrieve relevant chunks
    └── generator.py          ← Step 6: Generate answer with DeepSeek
```

---

## Final Notes

This project shows the complete foundation of a RAG-based document question-answering system.

The main idea is simple:

```text
Do not rely only on the LLM's internal memory.
Retrieve relevant external knowledge first.
Then generate an answer grounded in that retrieved context.
```

This version uses:

```text
HuggingFace embeddings for local vector creation
FAISS for vector search
DeepSeek for answer generation
LangChain for orchestration
```

It is a practical starting point for building document Q&A systems, research assistants, internal knowledge bots, and production GenAI applications.

* Note: This version uses DeepSeek through an OpenAI-compatible API endpoint instead of OpenAI. The embedding model remains HuggingFace-based, while DeepSeek is used only for answer generation. 
