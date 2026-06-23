# 04 — Multimodal RAG

A Retrieval-Augmented Generation system that understands **text, images, and tables** inside PDF documents.

> A text-only RAG can answer *"What does the paper say about Transformers?"*
> But a multimodal RAG pipeline can go further — it can extract text, detect tables, save images, build separate indexes, and route the query to the right type of content.

In this version, the project uses **DeepSeek instead of OpenAI** for text and table reasoning.

> Note: DeepSeek is used through its OpenAI-compatible API for text generation and table descriptions.
> Image extraction works, but image captioning is skipped using `--skip-images` because DeepSeek is not a direct GPT-4V vision replacement.

---

## What "Multimodal" Means

| Query                                         | Text-only RAG                     | This System                                                       |
| --------------------------------------------- | --------------------------------- | ----------------------------------------------------------------- |
| "Explain the Transformer architecture"        | ✅ Can answer                      | ✅ Can answer                                                      |
| "What BLEU scores are reported in the table?" | ⚠️ Only if table text is captured | ✅ Extracts tables and describes them                              |
| "Summarise the main idea of the paper"        | ✅ Can answer                      | ✅ Uses retrieved document chunks                                  |
| "What does the architecture diagram show?"    | ❌ Cannot answer                   | ⚠️ Image is extracted, but captioning is skipped in DeepSeek mode |
| "What are the main experimental results?"     | ⚠️ Partial                        | ✅ Uses text + table descriptions                                  |

---

## Architecture

```text
PDF Document
     │
     ▼
┌────────────────────┐
│  multimodal_parser │  ── extracts text / tables / images from PDF
└────────┬───────────┘
         │
    ┌────┴─────────────────────────────────┐
    ▼                ▼                     ▼
┌─────────┐   ┌────────────┐    ┌─────────────────┐
│  Text   │   │   Images   │    │     Tables      │
│ Blocks  │   │  PNG files │    │ structured rows │
└────┬────┘   └─────┬──────┘    └────────┬────────┘
     │              │                    │
     │              │               DeepSeek table
     │              │                description
     │              │                    │
     ▼              ▼                    ▼
┌──────────┐   Image captioning   ┌──────────────────┐
│  FAISS   │   skipped with       │      FAISS       │
│ Text Idx │   --skip-images      │    Table Idx     │
└────┬─────┘                      └────────┬─────────┘
     └──────────────────┬──────────────────┘
                        │
                 ┌──────┴──────┐
                 │ Query Router │  ── classifies query → TEXT / TABLE / IMAGE / ALL
                 └──────┬──────┘
                        │
                 ┌──────┴──────┐
                 │ Multi-Retriev│  ── fetches top-k from relevant indexes, merges results
                 └──────┬──────┘
                        │
                 ┌──────┴──────┐
                 │  Generator  │  ── DeepSeek builds final grounded answer
                 └─────────────┘
```

---

## Model Comparison

| Modality        | Model / Tool             | Why                                                                |
| --------------- | ------------------------ | ------------------------------------------------------------------ |
| Text embeddings | `all-MiniLM-L6-v2`       | Fast, free, local embedding model with good retrieval quality      |
| Text generation | DeepSeek                 | Used instead of OpenAI for final answer generation                 |
| Tables          | DeepSeek                 | Converts extracted table rows into searchable prose descriptions   |
| Images          | Skipped in DeepSeek mode | Images are extracted, but not captioned when using `--skip-images` |
| Vector store    | FAISS                    | Lightweight local vector database                                  |

**Alternative for full image support:** Use [LLaVA via Ollama](https://ollama.com/library/llava) later for local image captioning, then remove `--skip-images`.

---

## Setup

### 1. Clone and enter the project

```bash
git clone https://github.com/pranjalisr/Applied-RAG-Systems.git
cd Applied-RAG-Systems/04-multimodal-rag
```

---

### 2. Create and activate a virtual environment

Use Python 3.11.

```bash
python3.11 -m venv venv
source venv/bin/activate
```

For Windows:

```bash
python -m venv venv
venv\Scripts\activate
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

First upgrade pip:

```bash
python -m pip install --upgrade pip
```

Then install dependencies:

```bash
python -m pip install -r requirements.txt
```

If you face package version issues, install these compatible versions:

```bash
python -m pip uninstall -y openai httpx langchain-openai langchain langchain-community
python -m pip install "openai==1.55.3" "httpx==0.27.2" "langchain==0.1.20" "langchain-community==0.0.38" "langchain-openai==0.1.7"
```

Install the remaining packages:

```bash
python -m pip install faiss-cpu sentence-transformers python-dotenv pdfplumber pillow pandas "unstructured[pdf]"
```

Verify the important versions:

```bash
python -c "import openai, httpx, langchain_openai; print(openai.__version__, httpx.__version__)"
```

Expected:

```text
1.55.3 0.27.2
```

---

### 4. Configure environment variables

Create a `.env` file:

```bash
cp .env.example .env
```

Update `.env`:

```env
DEEPSEEK_API_KEY=your_deepseek_api_key_here
DEEPSEEK_BASE_URL=https://api.deepseek.com
DEEPSEEK_MODEL=deepseek-chat

IMAGES_OUTPUT_DIR=data/extracted/images
TABLES_OUTPUT_DIR=data/extracted/tables
```

---

### 5. Update `main.py` for DeepSeek

In `main.py`, keep the imports:

```python
from langchain_openai import ChatOpenAI
from openai import OpenAI
import os
```

Initialize the LLM like this:

```python
llm = ChatOpenAI(
    model=os.getenv("DEEPSEEK_MODEL", "deepseek-chat"),
    api_key=os.getenv("DEEPSEEK_API_KEY"),
    base_url=os.getenv("DEEPSEEK_BASE_URL", "https://api.deepseek.com"),
    temperature=0,
)
```

If the project also creates an OpenAI client, update it like this:

```python
openai_client = OpenAI(
    api_key=os.getenv("DEEPSEEK_API_KEY"),
    base_url=os.getenv("DEEPSEEK_BASE_URL", "https://api.deepseek.com"),
)
```

This allows the project to use DeepSeek through the same OpenAI-compatible interface.

---

## PDF Used for Testing

For testing this project, use:

```text
Attention Is All You Need
```

Source:

```text
https://arxiv.org/pdf/1706.03762
```

Download it into the project:

```bash
mkdir -p data/sample_docs
curl -L "https://arxiv.org/pdf/1706.03762" -o data/sample_docs/attention_is_all_you_need.pdf
```

This PDF is a good test document because it contains:

* normal research text
* architecture explanation
* images/figures
* tables
* experimental results

---

## Usage

### Ask a single question

```bash
python main.py \
  --file data/sample_docs/attention_is_all_you_need.pdf \
  --query "What is the Transformer architecture?" \
  --skip-images
```

---

### Ask a table-related question

```bash
python main.py \
  --file data/sample_docs/attention_is_all_you_need.pdf \
  --query "What BLEU scores are reported in the paper?" \
  --skip-images
```

---

### Skip image captioning during development

```bash
python main.py \
  --file data/sample_docs/attention_is_all_you_need.pdf \
  --query "Summarise the key findings" \
  --skip-images
```

This is the recommended mode when using DeepSeek.

---

### Skip both images and tables

```bash
python main.py \
  --file data/sample_docs/attention_is_all_you_need.pdf \
  --query "What is the main idea of the paper?" \
  --skip-images \
  --skip-tables
```

This runs the project in the fastest text-only mode.

---

### Interactive Q&A loop

```bash
python main.py \
  --file data/sample_docs/attention_is_all_you_need.pdf \
  --interactive \
  --skip-images
```

Example questions:

```text
What is self-attention?
```

```text
What does the paper say about multi-head attention?
```

```text
What are the main experimental results?
```

To exit:

```text
quit
```

---

## Full CLI Reference

```text
--file           Path to PDF document
--query          Question to answer
--model          Text generation model
--vision-model   Vision model for image captioning
--skip-images    Skip image captioning and image indexing
--skip-tables    Skip table description generation and table indexing
--interactive    Start interactive Q&A loop after indexing
```

For DeepSeek mode, the most reliable command is:

```bash
python main.py \
  --file data/sample_docs/attention_is_all_you_need.pdf \
  --query "What is the Transformer architecture?" \
  --skip-images
```

---

## Why `--skip-images` Is Used

The original multimodal pipeline supports image captioning using a vision model such as GPT-4V.

However, this version uses DeepSeek, and DeepSeek is being used here for text-based reasoning through an OpenAI-compatible API. Since it is not being used as a GPT-4V replacement, image captioning is skipped.

So the current working mode is:

```text
Text extraction       ✅
Table extraction      ✅
Image extraction      ✅
Text indexing         ✅
Table indexing        ✅
DeepSeek generation   ✅
Image captioning      ❌ skipped with --skip-images
```

This still demonstrates the core multimodal RAG pipeline because the system parses different document content types and performs routed retrieval over text and tables.

---

## Successful Local Run

The project was successfully tested with:

```bash
python main.py \
  --file data/sample_docs/attention_is_all_you_need.pdf \
  --query "What is the Transformer architecture?" \
  --skip-images
```

The pipeline completed:

```text
[main] Parsing document: data/sample_docs/attention_is_all_you_need.pdf
[parser] 'attention_is_all_you_need': 15 text blocks, 3 images, 10 tables extracted.
[main] Found 15 text blocks, 3 images, 10 tables.

[main] Indexing 15 text blocks ...
[text_indexer] Indexed 15 text chunks -> 'text_faiss_index'

[main] --skip-images set: skipping image captioning and indexing.

[main] Processing 10 table(s) ...
[table_processor] Describing table 1/10 ...
[table_processor] Describing table 2/10 ...
[table_processor] Describing table 3/10 ...
[table_processor] Describing table 4/10 ...
[table_processor] Describing table 5/10 ...
[table_processor] Describing table 6/10 ...
[table_processor] Describing table 7/10 ...
[table_processor] Describing table 8/10 ...
[table_processor] Describing table 9/10 ...
[table_processor] Describing table 10/10 ...

[main] Indexing 10 table description(s) ...
[table_indexer] Indexed 10 table descriptions -> 'table_faiss_index'

[main] Query: What is the Transformer architecture?
[main] Router selected modalities: ['TEXT']
[main] Retrieved 3 result(s) after merge/de-dup.
```

Example answer:

```text
The Transformer is a neural network architecture based solely on attention mechanisms,
replacing recurrence and convolutions entirely. It consists of an encoder and a decoder,
each composed of a stack of identical layers.

This answer is based on the provided text.
```

---

## Cost Considerations

Using DeepSeek for text and table reasoning is usually cheaper than using GPT-4-level models for every step.

The expensive part in a full multimodal system is usually image captioning with a vision model.

Cost-saving tips:

1. Use `--skip-images` while developing
2. Use DeepSeek for text/table reasoning
3. Cache table descriptions and indexes
4. Add local LLaVA later for free image captioning
5. Reuse FAISS indexes instead of rebuilding them every run

---

## What This Can vs Cannot Answer

### CAN answer in the current DeepSeek setup

* "What is the Transformer architecture?"
* "What is self-attention?"
* "What does the paper say about multi-head attention?"
* "What BLEU scores are reported in the paper?"
* "What are the main experimental results?"
* "Summarise the main idea of the paper."

### CANNOT fully answer in the current DeepSeek setup

* "What does the diagram visually show?"
* "Describe the architecture figure."
* "What is shown in the image on page 3?"

These require image captioning, which is skipped in this version.

---

## Comparison with Project 1: Basic RAG

| Feature             | Project 1: Basic RAG  | Project 4: Multimodal RAG                |
| ------------------- | --------------------- | ---------------------------------------- |
| Content types       | Text only             | Text + Tables + Extracted Images         |
| Indexes             | 1 FAISS index         | Separate indexes for text and tables     |
| Embedding model     | all-MiniLM-L6-v2      | all-MiniLM-L6-v2                         |
| LLM calls           | Final generation only | Table description + routing + generation |
| Query routing       | Not required          | Router selects relevant modality         |
| Image extraction    | ❌                     | ✅                                        |
| Image understanding | ❌                     | ⚠️ skipped in DeepSeek mode              |
| Table understanding | ❌                     | ✅                                        |
| Cost                | Low                   | Low to medium with DeepSeek              |
| Complexity          | Low                   | Higher                                   |

---

## New Concepts Introduced

* Multimodal document parsing
* Text, image, and table extraction from PDFs
* Separate FAISS indexes per modality
* LLM-generated table descriptions
* Query routing by modality
* Cross-modality retrieval
* DeepSeek integration through OpenAI-compatible API
* Development mode using `--skip-images`

---

## Project Structure

```text
04-multimodal-rag/
├── README.md
├── requirements.txt
├── .env.example
├── main.py
│
├── data/
│   ├── sample_docs/
│   │   └── attention_is_all_you_need.pdf
│   │
│   └── extracted/
│       ├── images/
│       └── tables/
│
├── src/
│   ├── multimodal_parser.py
│   ├── text_indexer.py
│   ├── image_processor.py
│   ├── image_indexer.py
│   ├── table_processor.py
│   ├── table_indexer.py
│   ├── query_router.py
│   ├── multi_retriever.py
│   └── generator.py
│
├── text_faiss_index/
└── table_faiss_index/
```

---

## Common Issues and Fixes

### 1. `Client.__init__() got an unexpected keyword argument 'proxies'`

This happens because of incompatible versions of `openai`, `httpx`, and `langchain-openai`.

Fix:

```bash
python -m pip uninstall -y openai httpx langchain-openai langchain langchain-community
python -m pip install "openai==1.55.3" "httpx==0.27.2" "langchain==0.1.20" "langchain-community==0.0.38" "langchain-openai==0.1.7"
```

Check:

```bash
python -c "import openai, httpx; print(openai.__version__, httpx.__version__)"
```

Expected:

```text
1.55.3 0.27.2
```

---

### 2. `unstructured-client requires httpx>=0.28.1`

You may see this warning:

```text
unstructured-client requires httpx>=0.28.1, but you have httpx 0.27.2
```

For this setup, keep:

```text
httpx==0.27.2
```

Do not upgrade to `0.28.1`, because the `proxies` error may return.

If needed, downgrade `unstructured-client`:

```bash
python -m pip uninstall -y unstructured-client unstructured
python -m pip install "unstructured[pdf]==0.13.7" "unstructured-client<0.25"
python -m pip install --force-reinstall "httpx==0.27.2"
```

---

### 3. DeepSeek API key not found

If you see:

```text
DEEPSEEK_API_KEY is not set
```

Check that `.env` exists:

```bash
ls -a
```

Then open `.env` and confirm:

```env
DEEPSEEK_API_KEY=your_deepseek_api_key_here
DEEPSEEK_BASE_URL=https://api.deepseek.com
DEEPSEEK_MODEL=deepseek-chat
```

---

### 4. PDF file not found

If you see:

```text
No such file or directory: data/sample_docs/attention_is_all_you_need.pdf
```

Download the PDF again:

```bash
mkdir -p data/sample_docs
curl -L "https://arxiv.org/pdf/1706.03762" -o data/sample_docs/attention_is_all_you_need.pdf
```

---

## Future Improvements

* Add LLaVA/Ollama for local image captioning
* Remove the need for `--skip-images`
* Add page-level citations in final answers
* Cache generated table descriptions
* Cache image captions
* Add support for multiple PDFs
* Add Streamlit frontend
* Add evaluation questions for each modality
* Add persistent FAISS loading instead of rebuilding every run

---

## Final Demo Command

Use this for the demo:

```bash
python main.py \
  --file data/sample_docs/attention_is_all_you_need.pdf \
  --query "Summarise the main idea of the paper and mention any table-based results." \
  --skip-images
```

This demonstrates:

```text
PDF parsing
Text extraction
Table extraction
Image extraction
FAISS indexing
Query routing
DeepSeek generation
Grounded RAG answer
```

---

## Summary

This project shows how a PDF can be converted into a multimodal-style RAG system. It extracts text, tables, and images, builds searchable indexes, routes each query to the right source of information, and generates grounded answers using DeepSeek.

In the current DeepSeek setup, the project supports strong text and table-based document reasoning. Image captioning is intentionally skipped with `--skip-images`, but the architecture is ready to support full visual reasoning later using a local model like LLaVA.
