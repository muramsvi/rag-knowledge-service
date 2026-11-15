# rag-knowledge-service
Production-grade Retrieval-Augmented Generation (RAG) microservice built with FastAPI, featuring structured observability, async ingestion, and comprehensive test coverage.
# rag-knowledge-service

[![CI Status](https://shields.io)]()
[![Python 3.12+](https://shields.io)]()
[![Code Style: Ruff](https://shields.io)]()
[![Type Checked: Mypy](https://shields.io)]()

A production-grade **Retrieval-Augmented Generation (RAG)** microservice built in Python with **FastAPI**. 

Designed with enterprise backend engineering principles in mind, this service goes beyond simple AI scripting by emphasizing **clean layered architecture, strict static typing, structured observability, asynchronous processing, and automated testing**.

---

## Architecture Overview

The service separates HTTP transport, business logic, and data persistence into distinct layers—mirroring standard enterprise microservice patterns to ensure high maintainability and testability.

```text
+-------------------+       +---------------------------------------+       +--------------------+

|                   |       |         rag-knowledge-service         |       |                    |
|   Client / User   +------>+  FastAPI Router (Validation & Auth)   |       |   Vector Database  |
|                   |       |                   |                   |       |  (Qdrant / Chroma) |
+-------------------+       |                   v                   |       |                    |

                            |  RAG Service (Chunking & Orchestration)+------>+  Semantic Search   |
                            |                   |                   |       +--------------------+
                            |                   v                   |
                            |  LLM Client Adapter (Retry & Timeout) +------>+  External LLM API  |
                            |                                       |       | (OpenAI/Anthropic) |
                            +---------------------------------------+       +--------------------+
                                                |
                                                v
                            +---------------------------------------+

                            |  Observability: Structlog & /metrics  |
                            +---------------------------------------+
```

### Core Engineering Pillars
* **Clean, Modular Design:** Decoupled `api`, `services`, and `repositories` layers using dependency injection and Pydantic schemas for end-to-end data validation.
* **Observability First:** Structured JSON logging (`structlog`) with correlation/request IDs, `/health` readiness probes, and Prometheus-compatible `/metrics` tracking request latency and token usage.
* **Resilient External Integrations:** Configurable timeouts, exponential backoff retries, and clean provider abstractions for vector stores and LLM APIs.
* **Asynchronous & Scalable:** Non-blocking I/O (`async`/`await`) for external API calls and background task processing for document ingestion.
* **Strict Code Quality:** Enforced static type checking (`mypy`), fast linting/formatting (`ruff`), and high test coverage (`pytest`) with mocked external dependencies.

---

## Project Structure

```text
rag-knowledge-service/
├── app/
│   ├── api/                 # FastAPI route handlers & dependency injection
│   ├── core/                # App configuration, structured logging, middleware
│   ├── models/              # Pydantic request/response DTOs & domain models
│   ├── services/            # Document chunking, embedding & RAG orchestration
│   └── repositories/        # Vector DB client & retrieval abstraction
├── tests/
│   ├── unit/                # Isolated unit tests with mocked LLM/DB clients
│   └── integration/         # End-to-end API tests via FastAPI TestClient
├── .github/workflows/       # CI pipeline (lint, typecheck, test)
├── Dockerfile               # Multi-stage production container build
├── docker-compose.yml       # Local orchestration (API + Vector DB)
├── pyproject.toml           # Dependency management & tool configuration
└── README.md
```

---

## Getting Started

### Prerequisites
* **Docker** & **Docker Compose** (Recommended for one-command setup)
* **Python 3.12+** (For local development without Docker)
* An API key for your chosen LLM provider (e.g., `OPENAI_API_KEY`)

### 1. Environment Configuration
Clone the repository and create a `.env` file from the example template:

```bash
git clone https://github.com<your-username>/rag-knowledge-service.git
cd rag-knowledge-service
cp .env.example .env
```

Update `.env` with your credentials:
```env
APP_ENV=development
LOG_LEVEL=INFO
LLM_API_KEY=your_api_key_here
VECTOR_DB_URL=http://qdrant:6333
```

### 2. Run with Docker Compose
Spin up both the FastAPI microservice and the vector database:

```bash
docker-compose up --build -d
```

The API will be available at `http://localhost:8000`. Interactive OpenAPI (Swagger) documentation is automatically generated at `http://localhost:8000/docs`.

---

## API Usage

### Health & Readiness Check
Verify service status and downstream vector database connectivity:
```bash
curl -X GET http://localhost:8000/health
```
**Response:**
```json
{
  "status": "healthy",
  "version": "1.0.0",
  "dependencies": {
    "vector_db": "connected"
  }
}
```

### Ingest a Document
Chunk, embed, and store a document into the knowledge base:
```bash
curl -X POST http://localhost:8000/api/v1/documents \
  -H "Content-Type: application/json" \
  -d '{
    "title": "Deployment Runbook",
    "content": "All production deployments require passing CI checks and a canary rollout phase.",
    "metadata": {"department": "platform-engineering"}
  }'
```

### Query the Knowledge Base
Ask a question and receive a grounded answer with source citations:
```bash
curl -X POST http://localhost:8000/api/v1/query \
  -H "Content-Type: application/json" \
  -d '{
    "question": "What is required before a production deployment?",
    "top_k": 3
  }'
```
**Response:**
```json
{
  "answer": "Production deployments require passing CI checks and undergoing a canary rollout phase.",
  "sources": [
    {
      "title": "Deployment Runbook",
      "score": 0.91,
      "snippet": "All production deployments require passing CI checks and a canary rollout phase."
    }
  ],
  "latency_ms": 412
}
```

---

## Testing & Code Quality

This project enforces strict quality gates to ensure maintainability in production environments.

```bash
# Run unit and integration tests with coverage report
pytest --cov=app tests/

# Run static type checking
mypy app/

# Run linter and formatting checks
ruff check .
```

---

## Future Production Enhancements
* [ ] **Hybrid Search:** Combine dense vector embeddings with sparse keyword search (BM25) for improved technical term retrieval.
* [ ] **Semantic Caching:** Integrate Redis to cache frequent queries and reduce LLM latency and token costs.
* [ ] **OpenTelemetry Tracing:** Export distributed traces to Jaeger or Datadog for granular latency breakdowns across chunking, retrieval, and generation steps.
