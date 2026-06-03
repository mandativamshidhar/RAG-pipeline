# Document Search & Q&A Pipeline

This repository provides a document ingestion and retrieval service that:
- ingests PDFs
- chunks text and computes embeddings
- stores vectors in FAISS for fast similarity search
- exposes a REST API for querying and evaluation

Quickstart

1. Create a Python virtual environment and install dependencies:

```powershell
python -m venv .venv
.\.venv\Scripts\Activate.ps1
pip install -r requirements.txt
```

2. Set any needed environment variables (e.g. `OPENAI_API_KEY` if using remote LLMs).

3. Run the FastAPI app locally:

```powershell
uvicorn app.main:app --reload --host 0.0.0.0 --port 8000
```

Available endpoints

- `POST /ingest` — JSON payload `{"pdf_folder":"<path>"}` to ingest PDFs from a folder.
- `POST /query` — JSON payload `{"question":"...","top_k":5}` to query the indexed documents.
- `python -m app.quality` — run a simple retrieval-quality script (MRR@K) against the index.

Notes

- Do not commit the `metadata.db` or `faiss.index` files; they are listed in `.gitignore`.
- Rebuild the index on a new machine by calling the ingest endpoint or running the benchmark with `--rebuild`.
- For production deployment, a `Dockerfile` and `gunicorn.conf.py` are included.

Performance tuning

- Chunking defaults: 200 words per chunk, 50-word overlap — tuned for lower retrieval latency and reasonable context size.
- FAISS index: HNSW (`IndexHNSWFlat`) wrapped in `IndexIDMap`, defaults tuned with `M=32` and `efSearch=128`.
- To reach sub-2s query latency on a 200-page PDF corpus, use a machine with at least 4 CPU cores and sufficient RAM. If latency is not met, try:
	- lowering `efSearch` (reduces latency, may reduce recall)
	- reduce `chunk_size` to reduce per-chunk embedding cost but watch index size
	- use a GPU and faiss-gpu for faster search

