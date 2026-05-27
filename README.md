# 🏢 Enterprise Knowledge Retrieval System

A production-oriented RAG (Retrieval-Augmented Generation) pipeline for enterprise document Q&A. Upload any PDF, ask natural language questions, and get answers grounded strictly in your document — with retrieval quality scores, latency metrics, and observability built in.

**Live demo:** [huggingface.co/spaces/Faraz618/enterprise-rag-system](https://huggingface.co/spaces/Faraz618/enterprise-rag-system)

---

## What It Does

- Upload enterprise PDFs (reports, policies, manuals, compliance docs)
- Ask natural language questions
- Retrieve semantically relevant document sections using FAISS
- Generate grounded answers via Groq's free LLM API (Llama 3.1)
- Evaluate answer quality with three proxy metrics
- Track latency, token usage, and traces per query

---

## Architecture

```
PDF Upload → Text Extraction → Chunking → Embedding → FAISS Index
                                                            ↓
User Query → Embed Query → Cosine Similarity Search → Top-K Chunks
                                                            ↓
                                              Relevance Threshold Check
                                             ↙                        ↘
                                       Fallback                 Prompt Assembly
                                                                      ↓
                                                        Groq API (Llama-3.1-8b)
                                                                      ↓
                                               Grounded Answer + Eval Scores
                                                                      ↓
                                                  Langfuse Trace + Local Log
```

---

## Tech Stack

| Layer | Technology |
|---|---|
| LLM | Groq API — Llama-3.1-8b-instant (free) |
| Embeddings | sentence-transformers/all-MiniLM-L6-v2 (local, free) |
| Vector Store | FAISS IndexFlatIP (exact cosine search) |
| PDF Parsing | PyPDF |
| Evaluation | Custom cosine similarity metrics |
| Observability | Langfuse (optional) + local JSONL logs |
| UI | Gradio 5 |
| Deployment | Hugging Face Spaces |

---

## Evaluation Metrics

Three proxy metrics — no LLM judge, no API cost, runs in milliseconds:

| Metric | What It Measures | Method |
|---|---|---|
| Faithfulness | Is the answer grounded in retrieved context? | Cosine sim: answer sentences vs context chunks |
| Answer Relevance | Does the answer address the question? | Cosine sim: answer vs query embeddings |
| Context Precision | Are retrieved chunks relevant to the query? | Rank-weighted average of FAISS scores |

---

## Setup — Hugging Face Spaces

1. Fork this repo or create a new Space (SDK: Gradio)
2. Add all files from this repository
3. Go to **Settings → Repository secrets** and add:
   - `GROQ_API_KEY` — free at [console.groq.com](https://console.groq.com)
   - `LANGFUSE_PUBLIC_KEY` *(optional)*
   - `LANGFUSE_SECRET_KEY` *(optional)*
4. Space builds automatically — first start takes ~3 minutes (model download)

## Setup — Local

```bash
git clone https://github.com/YOUR_USERNAME/enterprise-rag-system
cd enterprise-rag-system
pip install -r requirements.txt
cp .env.example .env
# Edit .env and add your GROQ_API_KEY
python app.py
```

---

## Project Structure

```
enterprise-rag-system/
├── app.py                    # Gradio UI + pipeline orchestration
├── requirements.txt
├── .env.example
├── src/
│   ├── ingestion.py          # PDF text extraction
│   ├── chunking.py           # Configurable text splitting
│   ├── embeddings.py         # Sentence-transformers + FAISS
│   ├── retrieval.py          # Query embedding + top-k search
│   ├── generation.py         # Groq API answer generation
│   ├── evaluation.py         # Faithfulness / relevance / precision
│   ├── observability.py      # Langfuse tracing + local logs
│   ├── metrics.py            # Rolling metrics dashboard
│   └── utils.py              # Shared helpers
├── evals/
│   ├── test_questions.json   # Evaluation dataset
│   └── eval_results.json     # Example results
├── architecture/
│   └── architecture.md       # Mermaid diagram + design decisions
└── logs/                     # Auto-created JSONL trace logs
```

---

## Known Limitations

- **Scanned PDFs** — PyPDF cannot extract text from image-based PDFs. Add Tesseract OCR for production use.
- **FAISS in-memory** — Index resets on Space restart. For persistence, serialize with `faiss.write_index()`.
- **Single document per session** — Multi-document corpus requires metadata filtering layer.
- **Groq free tier** — 14,400 requests/day, 6,000 tokens/minute. Sufficient for demos and development.

---

## License

MIT
