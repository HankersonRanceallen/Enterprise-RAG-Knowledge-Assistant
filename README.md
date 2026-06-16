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

## Key Features

- PDF document ingestion and chunking
- Vector search using Qdrant
- Retrieval-Augmented Generation (RAG)
- LangGraph workflow orchestration
- Ollama-powered local LLM inference
- Input and output guardrails
- MCP-compatible tool integration
- Background document processing with Redis + ARQ
- RAGAS evaluation metrics dashboard
- FastAPI production-ready backend
- Streamlit user interface
  
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

PDF Upload & Ingestion — async background processing, duplicate detection via SHA-256 hash
Semantic Search — sentence-transformers embeddings + Qdrant HNSW vector search
Local LLM — Ollama running Llama 3.1 8B, fully offline, no API costs
LangGraph Pipeline — stateful graph with conditional edges and full observability
MCP Tools — pluggable tool interface (calculator built-in, web search stub ready to wire)
Guardrails — input (prompt injection detection) and output (PII leak prevention) checks
JWT + API Key Auth — dual authentication, scope-based access control
Background Queue — ARQ + Redis, uploads return instantly, ingest runs asynchronously
RAGAS Evaluation — faithfulness, answer relevancy, context precision, context recall
Prometheus + Grafana — pre-built dashboard, alerts-ready metrics
Streamlit UI — dark-themed chat interface with source cards and eval dashboard


Tech Stack
LayerTechnologyLLM InferenceOllama (Llama 3.1 8B)Embeddingssentence-transformers (all-MiniLM-L6-v2)Vector DatabaseQdrantPipeline OrchestrationLangGraphBackend APIFastAPI + uvicornFrontend UIStreamlitBackground JobsARQ + RedisAuthpython-jose (JWT) + passlib (bcrypt)EvaluationRAGASMonitoringPrometheus + GrafanaPDF ParsingpdfplumberChunkingLangChain RecursiveCharacterTextSplitterDeploymentDocker Compose

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
enterprise-ai-assistant/
│
├── docker-compose.yml              # All 8 services wired together
├── scripts/
│   └── start.sh                    # One-command bootstrap
│
├── backend/
│   ├── Dockerfile
│   ├── requirements.txt
│   │
│   ├── api/
│   │   ├── main.py                 # FastAPI app, middleware, route registration
│   │   ├── auth.py                 # JWT + API key authentication
│   │   ├── config.py               # All settings via environment variables
│   │   ├── monitoring.py           # Prometheus metrics + middleware
│   │   ├── queue.py                # ARQ Redis connection pool
│   │   └── routes/
│   │       ├── auth_routes.py      # /auth/token, /auth/me
│   │       ├── documents.py        # /documents/upload, /documents/{id}/status
│   │       ├── query.py            # /query — main RAG endpoint
│   │       ├── evaluation.py       # /evaluate — RAGAS metrics
│   │       └── health.py           # /health — service status
│   │
│   ├── pipeline/
│   │   ├── graph.py                # LangGraph RAG workflow
│   │   ├── ingestor.py             # PDF → chunks → embeddings → Qdrant
│   │   └── vectorstore.py          # Qdrant async client operations
│   │
│   ├── worker/
│   │   └── tasks.py                # ARQ background ingest task
│   │
│   ├── tools/
│   │   └── mcp_tools.py            # MCP tool interface + calculator
│   │
│   ├── guardrails/
│   │   └── checker.py              # Input/output safety checks
│   │
│   └── evaluation/
│       └── metrics.py              # RAGAS evaluation runner
│
├── frontend/
│   ├── Dockerfile
│   ├── requirements.txt
│   └── app.py                      # Streamlit chat UI + eval dashboard
│
└── docker/
    ├── prometheus.yml               # Prometheus scrape config
    └── grafana/
        └── provisioning/
            ├── datasources/         # Auto-wires Prometheus datasource
            └── dashboards/          # Pre-built RAG metrics dashboard

How It Works
PDF Ingestion

User uploads PDF via the Streamlit UI
FastAPI saves file to disk and enqueues an ARQ job — returns in milliseconds
Background worker picks up the job from Redis
pdfplumber extracts text page by page
SHA-256 hash checked — duplicate PDFs are detected and skipped
RecursiveCharacterTextSplitter splits text into overlapping chunks that respect sentence boundaries
all-MiniLM-L6-v2 converts each chunk to a 384-dim embedding vector (runs locally on CPU)
Vectors and metadata (filename, page number, text) upserted to Qdrant
Status written to Redis — UI polls and displays "complete"

Query Pipeline (LangGraph)

guardrail_input — scans for prompt injection patterns; short-circuits to END if triggered
retrieve — embeds question, searches Qdrant via HNSW cosine similarity, filters by score threshold
mcp_tools — runs enabled tools to augment context (best-effort, never blocks pipeline)
generate — builds structured prompt with retrieved context, calls Ollama, handles timeouts gracefully
guardrail_output — scans response for PII before returning; blocks if found
Response returned with answer, source chunks with page numbers, latency, and guardrail status


API Reference
Authentication
Option 1 — API Key:
bashcurl -H "X-API-Key: dev-key-1" http://localhost:8000/api/v1/documents
Option 2 — JWT Token:
bashTOKEN=$(curl -s -X POST http://localhost:8000/api/v1/auth/token \
  -d "username=admin&password=adminpassword" | jq -r .access_token)

curl -H "Authorization: Bearer $TOKEN" http://localhost:8000/api/v1/documents
Endpoints
MethodEndpointAuthDescriptionGET/api/v1/healthNoneService status checkPOST/api/v1/auth/tokenNoneGet JWT tokenGET/api/v1/auth/meRequiredCurrent user infoPOST/api/v1/documents/uploadRequiredUpload PDF (async)GET/api/v1/documents/{id}/statusRequiredPoll ingest statusGET/api/v1/documentsRequiredList all documentsDELETE/api/v1/documents/{id}RequiredDelete documentPOST/api/v1/queryRequiredAsk a questionPOST/api/v1/evaluateRequiredRun RAGAS evaluationGET/api/v1/evaluate/demoRequiredDemo metricsGET/api/v1/metricsNonePrometheus metrics
Query example
bashcurl -X POST http://localhost:8000/api/v1/query \
  -H "X-API-Key: dev-key-1" \
  -H "Content-Type: application/json" \
  -d '{
    "question": "What is the refund policy?",
    "top_k": 5,
    "doc_filter": null
  }'
json{
  "answer": "Customers may request a full refund within 30 days of purchase...",
  "sources": [
    {
      "doc_id": "abc-123",
      "filename": "policy.pdf",
      "page": 3,
      "text": "Refund policy states customers may...",
      "score": 0.87
    }
  ],
  "guardrail_triggered": false,
  "latency_ms": 4231.5
}

Configuration
VariableDefaultDescriptionOLLAMA_MODELllama3.1:8bAny model available via OllamaOLLAMA_TEMPERATURE0.1Lower = more factual responsesOLLAMA_NUM_CTX4096Context window size in tokensCHUNK_SIZE512Characters per chunkCHUNK_OVERLAP64Overlap between chunksTOP_K5Chunks retrieved per querySIMILARITY_THRESHOLD0.5Minimum similarity score (0-1)GUARDRAILS_ENABLEDtrueToggle output guardrailsJWT_SECRET_KEYrandomChange this in productionAPI_KEYSdev-key-1,dev-key-2Change these in productionMAX_FILE_SIZE_MB50Upload size limit

Monitoring
Grafana dashboard auto-provisions at http://localhost:3000 with these panels:

Total Queries — cumulative request count
Active Queries — live gauge of concurrent requests
Guardrail Triggers — input vs output stage breakdown
Query Latency — p50 and p95 over time
Retrieval Score Distribution — median and p10 similarity scores
Ingest Throughput — chunks indexed per minute

Recommended alerts
ConditionMeaningInput guardrail rate spikesSystem is being probedQuery p95 latency > 30sOllama is overloadedRetrieval score median < 0.5Retrieval quality degradingBackend health check failsPipeline is down

Evaluation
Run RAGAS evaluation from the UI's Evaluation Dashboard tab, or via the API:
bashcurl -X POST http://localhost:8000/api/v1/evaluate \
  -H "X-API-Key: dev-key-1" \
  -H "Content-Type: application/json" \
  -d '{
    "samples": [{
      "question": "What is the refund policy?",
      "ground_truth": "Full refund within 30 days.",
      "answer": "Customers can get a refund within 30 days.",
      "contexts": ["Our policy allows refunds within 30 days..."]
    }]
  }'
MetricWhat it measuresTargetFaithfulnessAnswer grounded in context?> 0.8Answer RelevancyAnswer addresses the question?> 0.8Context PrecisionRetrieved chunks were useful?> 0.7Context RecallContext covers ground truth?> 0.7

Extending the System
Add a different LLM
bashdocker exec ollama ollama pull mistral:7b
# Update OLLAMA_MODEL=mistral:7b in docker-compose.yml
docker compose restart backend worker
Wire a real MCP tool
Edit backend/tools/mcp_tools.py and implement arun() in WebSearchTool:
pythonasync def arun(self, query: str) -> str:
    from mcp import ClientSession, StdioServerParameters
    from mcp.client.stdio import stdio_client
    server = StdioServerParameters(command="uvx", args=["mcp-server-brave-search"])
    async with stdio_client(server) as (r, w):
        async with ClientSession(r, w) as session:
            await session.initialize()
            result = await session.call_tool("brave_web_search", {"query": query})
            return result.content[0].text
Add a guardrail rule
Edit backend/guardrails/checker.py — add a pattern to _INPUT_PATTERNS or _PII_PATTERNS.
Enable GPU support
Uncomment the deploy block under ollama in docker-compose.yml (requires NVIDIA drivers).

Deployment & Scaling
Horizontal scaling
bash# Scale API workers (stateless — scale freely)
docker compose up -d --scale backend=3

# Scale ingest workers
docker compose up -d --scale worker=4
Production checklist

 Rotate JWT_SECRET_KEY to a cryptographically random value
 Replace API_KEYS with a database-backed key management system
 Add HTTPS termination (Nginx or Caddy)
 Move duplicate hash store from memory to Redis
 Configure Qdrant snapshots for backups
 Set up Grafana alerting
 Add request rate limiting per API key
 Replace Ollama with vLLM for high-throughput GPU inference


Security

All processing is fully local — no document data sent to external APIs
JWT tokens expire after 60 minutes
Passwords hashed with bcrypt
Input guardrails block prompt injection before hitting the LLM
Output guardrails scan for PII before returning responses
Guardrail trigger rate monitored in Prometheus


⚠️ The default JWT_SECRET_KEY and API_KEYS in docker-compose.yml are for local development only. Change them before any production deployment.


Contributing

Fork the repository
Create a feature branch: git checkout -b feature/your-feature
Make your changes
Verify services start: ./scripts/start.sh
Open a pull request

Ideas for contributions

Streaming responses from Ollama
Conversation memory across turns
DOCX, TXT, and Markdown file support
Hybrid search (vector + BM25 keyword)
Cross-encoder reranking after initial retrieval
Multi-tenant collection isolation
Document versioning and update detection


License
MIT License — see LICENSE for details.

Acknowledgements
Built with LangGraph, Qdrant, Ollama, RAGAS, and Streamlit.
