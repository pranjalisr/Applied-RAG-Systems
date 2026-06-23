# Legal AI Assistant

> ⚠️ **DISCLAIMER: This tool is for educational purposes only. It does NOT constitute legal advice. Always consult a qualified attorney before making any legal or business decisions.**

A Retrieval-Augmented Generation (RAG) pipeline that helps users understand legal contracts by parsing the document, extracting important clauses, identifying risk patterns, detecting possible internal conflicts, and answering natural-language questions grounded in the actual contract text.

This version uses **DeepSeek** as the LLM provider, **FAISS** as the local vector database, and **HuggingFace sentence-transformer embeddings** for semantic retrieval.

---

## What the Tool Does

* **Parses** PDF and DOCX contract files into structured text with section detection
* **Indexes** the document into a FAISS vector store for semantic search
* **Summarises** the contract: contract type, parties, effective date, duration, key obligations, and plain-English overview
* **Extracts key clauses** such as termination, confidentiality, limitation of liability, governing law, indemnification, and IP ownership
* **Analyses legal risks** by flagging HIGH / MEDIUM / LOW risk patterns with plain-English explanations
* **Detects potential conflicts** between clauses, such as mismatched renewal or termination terms
* **Answers legal questions** using a RAG-based Q&A chain grounded in retrieved contract sections
* **Supports interactive Q&A** so users can ask multiple follow-up questions after analysis

---

## Supported Document Types

| Format | Extension | Notes                                                                         |
| ------ | --------- | ----------------------------------------------------------------------------- |
| PDF    | `.pdf`    | Works best with text-based PDFs. Scanned PDFs require OCR before use.         |
| Word   | `.docx`   | Supports structured Word documents. Heading styles improve section detection. |

---

## Architecture

```text
┌─────────────────────────────────────────────────────────────────┐
│                        main.py (CLI)                            │
└──────────────┬──────────────────────────────────────────────────┘
               │
    ┌──────────▼──────────┐
    │  document_parser.py  │  PDF / DOCX → full_text + sections
    └──────────┬──────────┘
               │
    ┌──────────▼──────────┐
    │     indexer.py       │  text → chunks → HuggingFace embeddings → FAISS
    └──────────┬──────────┘
               │
       ┌───────┴────────┐
       │    DeepSeek    │  OpenAI-compatible API via langchain-openai
       └───────┬────────┘
               │
    ┌──────────▼──────────┐   ┌──────────────────────┐
    │    summarizer.py     │   │  clause_extractor.py  │
    └─────────────────────┘   └──────────┬───────────┘
                                          │
                               ┌──────────▼──────────┐
                               │   risk_analyzer.py   │
                               └──────────┬──────────┘
                                          │
                               ┌──────────▼──────────┐
                               │ conflict_detector.py │
                               └─────────────────────┘
    ┌─────────────────────┐
    │     qa_chain.py      │  FAISS retriever + legal Q&A prompt
    └─────────────────────┘
```

---

## Setup

### 1. Clone / navigate to the project

```bash
git clone https://github.com/pranjalisr/Applied-RAG-Systems.git
cd Applied-RAG-Systems/02-legal-ai-assistant
```

If the repository is already cloned, just navigate into the project:

```bash
cd 02-legal-ai-assistant
```

---

### 2. Create a virtual environment

Use **Python 3.11** for this project. Python 3.14 may cause dependency issues with `faiss-cpu`.

```bash
python3.11 -m venv venv
source venv/bin/activate
```

For Windows:

```bash
python -m venv venv
venv\Scripts\activate
```

Verify the Python version:

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
python -m pip install --upgrade pip
python -m pip install -r requirements.txt
```

If you face the `Client.__init__() got an unexpected keyword argument 'proxies'` error, install the compatible `httpx` version:

```bash
python -m pip install "httpx==0.27.2"
```

Recommended `requirements.txt`:

```txt
langchain==0.1.20
langchain-community==0.0.38
langchain-openai==0.1.6
faiss-cpu==1.8.0
sentence-transformers==2.7.0
pypdf==4.2.0
python-docx==1.1.2
openai==1.30.1
python-dotenv==1.0.1
pydantic==2.7.1
rich==13.7.1
httpx==0.27.2
```

---

### 4. Configure environment variables

Create a `.env` file:

```bash
cp .env.example .env
```

Update `.env` with your DeepSeek API key:

```env
DEEPSEEK_API_KEY=your_deepseek_api_key_here
DEEPSEEK_BASE_URL=https://api.deepseek.com
DEEPSEEK_MODEL=deepseek-chat
```

This project uses `langchain-openai` because DeepSeek supports an OpenAI-compatible API format.

---

### 5. Add a contract file

Create the data folder if it does not already exist:

```bash
mkdir -p data/sample_contracts
```

For testing, this project uses a public SEC contract PDF:

```text
master_services_agreement.pdf
```

Place it here:

```text
data/sample_contracts/master_services_agreement.pdf
```

The sample document used for testing is a **Master Services Agreement** between Chartis International LLC and MMR Information Systems, Inc., sourced from the public SEC archive.

Source PDF:

```text
https://www.sec.gov/Archives/edgar/data/1285701/000113626110000125/exhibit10-28.pdf
```

Download using:

```bash
curl -L "https://www.sec.gov/Archives/edgar/data/1285701/000113626110000125/exhibit10-28.pdf" \
-o data/sample_contracts/master_services_agreement.pdf
```

---

## How to Run

### Full analysis

Runs parsing, indexing, executive summary, clause extraction, risk analysis, conflict detection, and Q&A if a question is provided.

```bash
python main.py --file data/sample_contracts/master_services_agreement.pdf --model deepseek-chat
```

---

### Ask a single question and exit

```bash
python main.py --file data/sample_contracts/master_services_agreement.pdf --model deepseek-chat --question "Who are the parties?"
```

Another example:

```bash
python main.py --file data/sample_contracts/master_services_agreement.pdf --model deepseek-chat --question "What are the termination rights?"
```

---

### Skip risk analysis and conflict detection

Use this when you want a faster test run.

```bash
python main.py --file data/sample_contracts/master_services_agreement.pdf --model deepseek-chat --skip-risks --skip-conflicts --question "Who are the parties?"
```

---

### Interactive Q&A mode

This allows you to ask multiple questions after the document has been parsed and indexed.

```bash
python main.py --file data/sample_contracts/master_services_agreement.pdf --model deepseek-chat --interactive
```

To exit interactive mode, type:

```text
exit
```

---

## Sample Questions to Ask

```text
Who are the parties to this agreement?
```

```text
What is the purpose of this Master Services Agreement?
```

```text
What services does MMR provide?
```

```text
What are the key obligations of Chartis?
```

```text
What are the key obligations of MMR?
```

```text
What are the termination rights?
```

```text
Can either party terminate this agreement for material breach?
```

```text
What happens if a party fails to cure a breach?
```

```text
What are the notice requirements for termination?
```

```text
Does the agreement automatically renew?
```

```text
What confidentiality obligations are included?
```

```text
Do confidentiality obligations survive termination?
```

```text
What does the agreement say about limitation of liability?
```

```text
Are there any liability caps?
```

```text
What are the biggest risks in this agreement?
```

```text
Are there any conflicting clauses?
```

```text
Which clauses should be clarified before signing?
```

```text
Which clauses should a lawyer review carefully?
```

---

## Output Sections Explained

| Section                   | What it shows                                                                            |
| ------------------------- | ---------------------------------------------------------------------------------------- |
| **Legal Notice**          | Educational-use disclaimer and warning that the tool is not legal advice                 |
| **Parsing Document**      | Reads the PDF/DOCX and detects pages and sections                                        |
| **Building Vector Index** | Splits the contract into chunks and stores embeddings in FAISS                           |
| **Executive Summary**     | Contract type, parties, effective date, duration, obligations, and plain-English summary |
| **Key Clause Extraction** | Extracted legal clauses with type, section reference, and plain-English explanation      |
| **Risk Analysis**         | HIGH / MEDIUM / LOW risk findings with explanation and suggested fairer alternatives     |
| **Conflict Detection**    | Potential contradictions between clauses, such as inconsistent notice or renewal terms   |
| **Q&A**                   | Natural-language answer grounded in retrieved chunks from the contract                   |

---

## Example Output

For the question:

```text
What are the termination rights?
```

The system identifies termination rights such as:

```text
1. Termination for Material Breach
2. Termination for Change in Law
3. Termination by Non-Renewal
4. Termination upon Termination of All Local Agreements
```

It also highlights possible risks such as:

```text
- Auto-renewal traps
- Vague cure periods
- Redacted notice periods
- Possible one-sided termination language
```

And detects possible conflicts such as:

```text
- Termination notice period mismatch
- Auto-renewal vs termination inconsistency
```

---

## Important Fixes Applied

During local setup, the following issues were fixed:

### Python version issue

`faiss-cpu==1.8.0` does not install properly with Python 3.14. The project should use Python 3.11.

```bash
python3.11 -m venv venv
source venv/bin/activate
```

---

### DeepSeek integration

The original OpenAI configuration was replaced with DeepSeek settings.

Instead of:

```env
OPENAI_API_KEY=sk-...
OPENAI_MODEL=gpt-4
```

Use:

```env
DEEPSEEK_API_KEY=your_deepseek_api_key_here
DEEPSEEK_BASE_URL=https://api.deepseek.com
DEEPSEEK_MODEL=deepseek-chat
```

The LLM client uses:

```python
llm = ChatOpenAI(
    model=args.model,
    temperature=0,
    openai_api_key=api_key,
    openai_api_base=base_url,
)
```

---

### `httpx` compatibility issue

If this error appears:

```text
Client.__init__() got an unexpected keyword argument 'proxies'
```

Fix it with:

```bash
python -m pip install "httpx==0.27.2"
```

Also keep this in `requirements.txt`:

```txt
httpx==0.27.2
```

---

### Clause JSON parsing fix

DeepSeek may return JSON wrapped in markdown like:

````text
```json
[
  {
    "clause_type": "termination"
  }
]
````

````

To fix parsing, the clause extractor cleans the model response before calling `json.loads()`.

The cleaner extracts only the JSON array from the LLM response and removes markdown fences.

---

## Project Structure

```text
02-legal-ai-assistant/
├── README.md
├── requirements.txt
├── .env.example
├── .env
├── main.py
├── data/
│   └── sample_contracts/
│       └── master_services_agreement.pdf
├── legal_index_master_services_agreement/
│   └── FAISS index files generated after running the project
├── src/
│   ├── __init__.py
│   ├── document_parser.py
│   ├── indexer.py
│   ├── summarizer.py
│   ├── clause_extractor.py
│   ├── risk_analyzer.py
│   ├── conflict_detector.py
│   └── qa_chain.py
└── prompts/
    ├── summary_prompt.txt
    ├── clause_prompt.txt
    └── risk_prompt.txt
````

---

## Dependencies

| Package                 | Purpose                                                        |
| ----------------------- | -------------------------------------------------------------- |
| `langchain`             | Core orchestration for LLM-based workflows                     |
| `langchain-community`   | Community integrations such as FAISS vector store support      |
| `langchain-openai`      | OpenAI-compatible chat model wrapper used with DeepSeek        |
| `faiss-cpu`             | Local vector similarity search                                 |
| `sentence-transformers` | HuggingFace embedding model for local embeddings               |
| `pypdf`                 | PDF text extraction                                            |
| `python-docx`           | DOCX parsing                                                   |
| `openai`                | OpenAI-compatible API client used by DeepSeek                  |
| `python-dotenv`         | Loads API keys and model config from `.env`                    |
| `pydantic`              | Data validation                                                |
| `rich`                  | Formatted terminal output                                      |
| `httpx`                 | HTTP client dependency required by OpenAI-compatible API calls |

---

## Limitations

> These are not bugs — they are natural limitations of the technology.

1. **This tool is not a lawyer.**
   It can help identify clauses, risks, and possible conflicts, but it cannot provide legal advice.

2. **The model may miss risks or produce false positives.**
   Some contract risks require jurisdiction-specific legal expertise.

3. **PDF extraction can be imperfect.**
   Tables, footnotes, scanned pages, and complex formatting may reduce extraction quality.

4. **Scanned PDFs require OCR.**
   If the PDF is image-based, `pypdf` may extract little or no text.

5. **Long contracts may be truncated for summary and clause extraction.**
   Very long documents may require chunk-level or map-reduce style summarisation for better coverage.

6. **Embeddings are not legal-domain specific.**
   The default `all-MiniLM-L6-v2` embedding model works well for general semantic search but may miss highly technical legal meaning.

7. **RAG answers depend on retrieval quality.**
   If the relevant clause is not retrieved, the final answer may be incomplete.

8. **Always verify findings manually.**
   Every clause, risk, and conflict should be reviewed against the original contract by a qualified attorney.

---

## Demo Commands

Fast test:

```bash
python main.py --file data/sample_contracts/master_services_agreement.pdf --model deepseek-chat --skip-risks --skip-conflicts --question "Who are the parties?"
```

Full analysis:

```bash
python main.py --file data/sample_contracts/master_services_agreement.pdf --model deepseek-chat --question "What are the termination rights?"
```

Interactive mode:

```bash
python main.py --file data/sample_contracts/master_services_agreement.pdf --model deepseek-chat --interactive
```

---

## Final Status

The project runs locally with:

```text
DeepSeek + LangChain + FAISS + HuggingFace Embeddings + PDF RAG
```

Successfully tested on:

```text
data/sample_contracts/master_services_agreement.pdf
```

Working features:

```text
✅ PDF parsing
✅ FAISS indexing
✅ Executive summary
✅ Clause extraction
✅ Risk analysis
✅ Conflict detection
✅ RAG-based Q&A
✅ Interactive Q&A
```
