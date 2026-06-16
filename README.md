# Enterprise-RAG-Knowledge-Assistant
Enterprise AI Knowledge Assistant
A production-grade, fully local RAG (Retrieval-Augmented Generation) system that lets you upload PDF documents and ask questions about them using AI. No API keys, no cloud services — everything runs on your machine.

📋 Table of Contents

Overview

Architecture
Features
Tech Stack
Prerequisites
Quick Start
Project Structure
How It Works
API Reference
Configuration
Monitoring
Evaluation
Extending the System
Deployment & Scaling
Security
Contributing


Overview
This system allows enterprises to build a private, secure AI assistant over their own document libraries. Upload PDFs, ask natural language questions, and get answers grounded in your documents — with full source attribution showing exactly which page and document the answer came from.
Key design decisions:

Fully local — LLM runs via Ollama, embeddings via sentence-transformers, no data leaves your infrastructure
Production-ready — async job queue, JWT auth, Prometheus metrics, Grafana dashboards
Extensible — MCP tool interface for plugging in web search, calculators, databases, and more
Observable — every pipeline stage is instrumented and visualised

  
Architecture
## System Architecture

```text
┌───────────────────────────────────────────┐
│               Streamlit UI                │
│                 Port 8501                 │
└───────────────────┬───────────────────────┘
                    │
                    ▼
┌───────────────────────────────────────────┐
│              FastAPI Backend              │
│                 Port 8000                 │
└───────┬───────────────┬───────────────────┘
        │               │
        ▼               ▼
┌───────────────┐ ┌───────────────┐
│    Qdrant     │ │    Ollama     │
│   Vector DB   │ │  Llama 3.1    │
└───────▲───────┘ └───────────────┘
        │
        │
┌───────┴───────┐
│  ARQ Worker   │
│ PDF → Embed   │
└───────▲───────┘
        │
┌───────┴───────┐
│     Redis     │
│ Job Queue     │
└───────────────┘
```
## LangGraph Workflow

```text
[Question]
     │
     ▼
Input Guardrails
     │
     ├── Blocked ─────────► END
     │
     ▼
Retrieve from Qdrant
     │
     ▼
MCP Tools
     │
     ▼
Generate Response
     │
     ▼
Output Guardrails
     │
     ├── PII Detected ───► END
     │
     ▼
Final Response
```
Features

## Features

* **LangGraph RAG Pipeline** — Stateful workflow with conditional routing, retrieval, tool execution, generation, and guardrails.
* **Local LLM Inference** — Ollama running Llama 3.1 8B fully offline with zero API costs.
* **Semantic Search** — Sentence-transformers embeddings stored in Qdrant HNSW vector indexes.
* **Async PDF Ingestion** — Background processing with ARQ and Redis.
* **MCP Tool Integration** — Pluggable tool architecture with calculator support and web-search extensibility.
* **Guardrails** — Prompt injection detection and PII prevention before responses are returned.
* **JWT + API Key Authentication** — Dual authentication model with role-based access.
* **RAGAS Evaluation** — Faithfulness, answer relevancy, context precision, and context recall scoring.
* **Prometheus + Grafana Monitoring** — Real-time observability and alert-ready metrics.
* **Streamlit Frontend** — Interactive chat interface with source attribution and evaluation dashboard.
* **Duplicate Detection** — SHA-256 hashing prevents re-indexing identical documents.


Tech Stack
## Tech Stack

| Layer                  | Technology                               |
| ---------------------- | ---------------------------------------- |
| LLM Inference          | Ollama (Llama 3.1 8B)                    |
| Embeddings             | sentence-transformers (all-MiniLM-L6-v2) |
| Vector Database        | Qdrant                                   |
| Pipeline Orchestration | LangGraph                                |
| Backend API            | FastAPI + Uvicorn                        |
| Frontend UI            | Streamlit                                |
| Background Jobs        | ARQ + Redis                              |
| Authentication         | python-jose (JWT) + passlib (bcrypt)     |
| Evaluation             | RAGAS                                    |
| Monitoring             | Prometheus + Grafana                     |
| PDF Parsing            | pdfplumber                               |
| Chunking               | LangChain RecursiveCharacterTextSplitter |
| Deployment             | Docker Compose                           |


Prerequisites

Docker Desktop — docker.com/products/docker-desktop
8 GB RAM minimum — 16 GB recommended for Llama 3.1 8B
~6 GB free disk — for the model download
Git


Quick Start
1. Clone the repository
bashgit clone https://github.com/yourusername/enterprise-ai-assistant.git
cd enterprise-ai-assistant
2. Run the start script
bashchmod +x scripts/start.sh
./scripts/start.sh
This will:

Start Qdrant, Ollama, and Redis
Wait for Ollama to be ready
Download llama3.1:8b (~4.7 GB, one-time)
Start the FastAPI backend, worker, frontend, Prometheus, and Grafana

3. Open the UI
ServiceURLCredentialsChat UI (Streamlit)http://localhost:8501API key: dev-key-1API Docs (Swagger)http://localhost:8000/docs—Qdrant Dashboardhttp://localhost:6333/dashboard—Prometheushttp://localhost:9090—Grafanahttp://localhost:3000admin / admin
4. Upload a PDF and start asking questions

Open http://localhost:8501
Enter dev-key-1 as the API key in the sidebar
Upload a PDF using the file uploader
Wait for ingest status to show "complete"
Type your question in the chat box


Project Structure

```text
enterprise-ai-assistant/
│
├── docker-compose.yml                 # All services wired together
│
├── scripts/
│   └── start.sh                       # One-command bootstrap
│
├── backend/
│   ├── Dockerfile
│   ├── requirements.txt
│   │
│   ├── api/
│   │   ├── main.py                    # FastAPI app, middleware, route registration
│   │   ├── auth.py                    # JWT + API key authentication
│   │   ├── config.py                  # Environment-based configuration
│   │   ├── monitoring.py              # Prometheus metrics + middleware
│   │   ├── queue.py                   # ARQ Redis connection pool
│   │   │
│   │   └── routes/
│   │       ├── auth_routes.py         # /auth/token, /auth/me
│   │       ├── documents.py           # Upload + ingest status endpoints
│   │       ├── query.py               # Main RAG endpoint
│   │       ├── evaluation.py          # RAGAS evaluation endpoint
│   │       └── health.py              # Service health checks
│   │
│   ├── pipeline/
│   │   ├── graph.py                   # LangGraph workflow
│   │   ├── ingestor.py                # PDF → chunks → embeddings
│   │   └── vectorstore.py             # Qdrant async operations
│   │
│   ├── worker/
│   │   └── tasks.py                   # ARQ background jobs
│   │
│   ├── tools/
│   │   └── mcp_tools.py               # MCP tools + calculator
│   │
│   ├── guardrails/
│   │   └── checker.py                 # Prompt injection & PII detection
│   │
│   └── evaluation/
│       └── metrics.py                 # RAGAS evaluation runner
│
├── frontend/
│   ├── Dockerfile
│   ├── requirements.txt
│   └── app.py                         # Streamlit chat UI + evaluation dashboard
│
└── docker/
    ├── prometheus.yml                 # Prometheus scrape configuration
    │
    └── grafana/
        └── provisioning/
            ├── datasources/           # Auto-configured Prometheus datasource
            └── dashboards/            # Pre-built RAG metrics dashboards
```
How It Works
PDF Ingestion

1. User uploads a PDF through the Streamlit interface.
2. FastAPI saves the file and enqueues an ARQ background job.
3. Redis stores the job and returns immediately.
4. An ARQ worker picks up the task.
5. `pdfplumber` extracts text page-by-page.
6. A SHA-256 hash is generated to detect duplicate documents.
7. `RecursiveCharacterTextSplitter` creates overlapping chunks.
8. `all-MiniLM-L6-v2` generates 384-dimensional embeddings.
9. Chunks and metadata are stored in Qdrant.
10. Status updates are written to Redis.
11. Streamlit polls the status endpoint and displays completion progress.

---

## Query Pipeline (LangGraph)

### 1. Input Guardrails

* Detect prompt injection attempts.
* Block malicious requests before retrieval.

### 2. Retrieval

* Embed user query.
* Perform HNSW cosine similarity search in Qdrant.
* Filter results using similarity threshold.

### 3. MCP Tools

* Execute optional tools.
* Calculator
* Web Search
* Custom MCP integrations

### 4. Generation

* Build structured context prompt.
* Send prompt to Ollama.
* Generate answer using Llama 3.1 8B.

### 5. Output Guardrails

* Scan response for PII.
* Block unsafe output if detected.

### Response Payload

- Generated answer
- Source chunks
- Page references
- Latency metrics
- Guardrail status

# API Reference

## Authentication

### API Key

```bash
curl -H "X-API-Key: dev-key-1" \
http://localhost:8000/api/v1/documents
```

### JWT Token

```bash
TOKEN=$(curl -s -X POST \
http://localhost:8000/api/v1/auth/token \
-d "username=admin&password=adminpassword" \
| jq -r .access_token)

curl -H "Authorization: Bearer $TOKEN" \
http://localhost:8000/api/v1/documents
```

---

## Endpoints

| Method | Endpoint                      | Auth | Description        |
| ------ | ----------------------------- | ---- | ------------------ |
| GET    | /api/v1/health                | No   | Service health     |
| POST   | /api/v1/auth/token            | No   | Generate JWT       |
| GET    | /api/v1/auth/me               | Yes  | Current user       |
| POST   | /api/v1/documents/upload      | Yes  | Upload PDF         |
| GET    | /api/v1/documents/{id}/status | Yes  | Poll ingest status |
| GET    | /api/v1/documents             | Yes  | List documents     |
| DELETE | /api/v1/documents/{id}        | Yes  | Delete document    |
| POST   | /api/v1/query                 | Yes  | Ask questions      |
| POST   | /api/v1/evaluate              | Yes  | Run RAGAS          |
| GET    | /api/v1/evaluate/demo         | Yes  | Demo metrics       |
| GET    | /api/v1/metrics               | No   | Prometheus metrics |

---

## Query Example

```bash
curl -X POST http://localhost:8000/api/v1/query \
-H "X-API-Key: dev-key-1" \
-H "Content-Type: application/json" \
-d '{
  "question": "What is the refund policy?",
  "top_k": 5,
  "doc_filter": null
}'
```

### Example Response

```json
{
  "answer": "Customers may request a full refund within 30 days of purchase.",
  "sources": [
    {
      "doc_id": "abc-123",
      "filename": "policy.pdf",
      "page": 3,
      "score": 0.87
    }
  ],
  "guardrail_triggered": false,
  "latency_ms": 4231.5
}
```

---

# Configuration

| Variable             | Default              |
| -------------------- | -------------------- |
| OLLAMA_MODEL         | llama3.1:8b          |
| OLLAMA_TEMPERATURE   | 0.1                  |
| OLLAMA_NUM_CTX       | 4096                 |
| CHUNK_SIZE           | 512                  |
| CHUNK_OVERLAP        | 64                   |
| TOP_K                | 5                    |
| SIMILARITY_THRESHOLD | 0.5                  |
| GUARDRAILS_ENABLED   | true                 |
| JWT_SECRET_KEY       | random               |
| API_KEYS             | dev-key-1, dev-key-2 |
| MAX_FILE_SIZE_MB     | 50                   |

---

# Monitoring

Grafana Dashboard

* Total Queries
* Active Queries
* Guardrail Triggers
* Query Latency (P50/P95)
* Retrieval Score Distribution
* Ingest Throughput

## Recommended Alerts

| Alert                        | Meaning                     |
| ---------------------------- | --------------------------- |
| Guardrail spike              | Potential probing           |
| P95 latency > 30s            | Ollama overloaded           |
| Median retrieval score < 0.5 | Retrieval quality degrading |
| Health check failure         | Service unavailable         |

---

# Evaluation

Run RAGAS evaluation from the Streamlit dashboard or API.

```bash
curl -X POST http://localhost:8000/api/v1/evaluate \
-H "X-API-Key: dev-key-1" \
-H "Content-Type: application/json"
```

## Metrics

| Metric            | Target |
| ----------------- | ------ |
| Faithfulness      | > 0.80 |
| Answer Relevancy  | > 0.80 |
| Context Precision | > 0.70 |
| Context Recall    | > 0.70 |

---

# Extending the System

## Change LLM

```bash
docker exec ollama ollama pull mistral:7b
```

Update:

```env
OLLAMA_MODEL=mistral:7b
```

Restart:

```bash
docker compose restart backend worker
```

## Add MCP Tool

Edit:

```text
backend/tools/mcp_tools.py
```

Implement:

```python
async def arun(self, query: str):
    ...
```

## Add Guardrail Rules

Edit:

```text
backend/guardrails/checker.py
```

Add patterns to:

* `_INPUT_PATTERNS`
* `_PII_PATTERNS`

---

# Deployment & Scaling

## Scale API

```bash
docker compose up -d --scale backend=3
```

## Scale Workers

```bash
docker compose up -d --scale worker=4
```

### Production Checklist

* Rotate JWT secret
* Replace development API keys
* Add HTTPS
* Store hashes in Redis
* Configure Qdrant snapshots
* Configure Grafana alerts
* Add rate limiting
* Upgrade to vLLM for GPU inference

---

# Security

* Local-first processing
* No document data leaves the environment
* JWT expires after 60 minutes
* Passwords hashed with bcrypt
* Input guardrails prevent prompt injection
* Output guardrails detect PII
* Guardrail activity monitored in Prometheus

> Default JWT secrets and API keys are for development only. Replace before production deployment.

---

# Contributing

```bash
git checkout -b feature/my-feature
```

```bash
./scripts/start.sh
```

Open a Pull Request.

## Contribution Ideas

* Streaming responses
* Conversation memory
* DOCX support
* Hybrid search
* Cross-encoder reranking
* Multi-tenant support
* Document versioning

---

# License

MIT License

See `LICENSE` for details.

---

# Acknowledgements

Built with:

* LangGraph
* Qdrant
* Ollama
* RAGAS
* FastAPI
* Redis
* Streamlit
* Prometheus
* Grafana


Acknowledgements
Built with LangGraph, Qdrant, Ollama, RAGAS, and Streamlit.
