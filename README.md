# RAG Document Q&A Pipeline

> PDF ingestion → semantic chunking → FAISS vector search → GPT-3.5 generation, served via FastAPI REST API.

![Python](https://img.shields.io/badge/Python-3.10+-blue?logo=python&logoColor=white)
![FastAPI](https://img.shields.io/badge/FastAPI-0.100+-green?logo=fastapi&logoColor=white)
![LangChain](https://img.shields.io/badge/LangChain-latest-blueviolet)
![FAISS](https://img.shields.io/badge/FAISS-vector--search-orange)
![Docker](https://img.shields.io/badge/Docker-ready-2496ED?logo=docker&logoColor=white)
![License](https://img.shields.io/badge/License-MIT-lightgrey)

---

## Performance

| Metric | Result |
|---|---|
| Retrieval quality (MRR@5) | **0.81** |
| End-to-end query latency | **< 2s** on a 200-page corpus |
| Evaluation framework | RAGAS |
| Embedding model | `all-MiniLM-L6-v2` (Hugging Face, open source) |
| FAISS index type | `IndexHNSWFlat` + `IndexIDMap` (M=32, efSearch=128) |

---

## Architecture

```
PDF files
   │
   ▼
[PDF Loader] ──► [Text Chunker] ──► [Sentence Transformer Embeddings]
                  200 words/chunk        (Hugging Face, open source)
                  50-word overlap
                      │
                      ▼
                 [FAISS Index]
                 (HNSW, persisted)
                      │
                 [POST /query]
                      │
                      ▼
              [GPT-3.5-turbo] ──► JSON response
                      │
                 [POST /ingest]   [python -m app.quality]
```

---

## Tech Stack

| Layer | Tool |
|---|---|
| Ingestion & chunking | LangChain, PyMuPDF |
| Embeddings | `sentence-transformers/all-MiniLM-L6-v2` (open source) |
| Vector store | FAISS (`IndexHNSWFlat`) |
| LLM generation | OpenAI GPT-3.5-turbo |
| API server | FastAPI + Gunicorn |
| Containerisation | Docker |
| Evaluation | RAGAS (MRR@K) |

---

## Quickstart

### 1. Clone and install

```bash
git clone https://github.com/mandativamshidhar/RAG-pipeline.git
cd RAG-pipeline
python -m venv .venv
# Windows
.\.venv\Scripts\Activate.ps1
# Linux / macOS
source .venv/bin/activate

pip install -r requirements.txt
```

### 2. Set environment variables

```bash
export OPENAI_API_KEY=your_key_here
```

### 3. Run the API locally

```bash
uvicorn app.main:app --reload --host 0.0.0.0 --port 8000
```

### 4. Or run with Docker

```bash
docker build -t rag-pipeline .
docker run -e OPENAI_API_KEY=$OPENAI_API_KEY -p 8000:8000 rag-pipeline
```

---

## API Reference

### `POST /ingest`
Ingest all PDFs from a folder into the FAISS index.

```json
// Request
{ "pdf_folder": "./pdfs" }

// Response
{ "status": "ok", "chunks_indexed": 843 }
```

### `POST /query`
Query the indexed documents with a natural language question.

```json
// Request
{ "question": "What is the refund policy?", "top_k": 5 }

// Response
{
  "answer": "The refund policy states...",
  "sources": ["doc1.pdf (p.4)", "doc2.pdf (p.11)"]
}
```

### `python -m app.quality`
Run MRR@K retrieval quality evaluation against the current index.

```bash
python -m app.quality --k 5
# MRR@5: 0.81
```

---

## Project Structure

```
RAG-pipeline/
├── app/
│   ├── main.py          # FastAPI app + endpoints
│   ├── ingestor.py      # PDF loading, chunking, embedding
│   ├── retriever.py     # FAISS search logic
│   ├── generator.py     # GPT-3.5 generation
│   └── quality.py       # RAGAS / MRR evaluation
├── pdfs/                # Sample PDFs (gitignored in production)
├── Dockerfile
├── gunicorn.conf.py
├── requirements.txt
└── .gitignore           # faiss.index and metadata.db excluded
```

---

## Performance Tuning Notes

- **Chunk size**: 200 words / 50-word overlap — tuned for low latency with reasonable context window usage
- **HNSW parameters**: `M=32`, `efSearch=128` — balance between recall and query speed
- **Scaling latency**: Lower `efSearch` to trade recall for speed; switch to `faiss-gpu` for GPU-accelerated search
- **Minimum hardware**: 4 CPU cores + 4 GB RAM to sustain sub-2s latency on a 200-page corpus

---

## What I Learned

- HNSW approximate nearest-neighbour search achieves significantly lower query latency than flat L2 search at scale
- RAGAS evaluation revealed that chunk overlap has a larger effect on MRR than chunk size within the 150–300 word range
- Gunicorn with multiple workers is necessary to prevent FastAPI event-loop blocking under concurrent requests

---

## Author

**Vamshidhar Reddy Mandati**  
AI/ML Engineer · [LinkedIn](https://linkedin.com/in/vamshidhar-reddy-mandati) · [GitHub](https://github.com/mandativamshidhar)

---

## License

MIT
