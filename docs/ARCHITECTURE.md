# Career Path Strategist — Technical Design Document

**Version:** 1.0 (Draft)  
**Status:** Proposed architecture — pending review  
**Milestone:** M1 — Design and Data Layer  
**Related Issue:** #1

## 1. Introduction

Career Path Strategist is an AI-powered conversational application designed to help users explore technology career paths, identify professional skills, and understand career-related requirements.

The system uses Retrieval-Augmented Generation (RAG) to ground career recommendations in an approved reference knowledge base.

It also supports asynchronous skill extraction, a live career skill tree, career-track switching, and an evaluation loop that checks generated recommendations against retrieved evidence.

The application is designed for engineering demonstration and does not provide professional career advice.

## 2. System Objectives

The main objectives are:

1. Store sessions, messages, extracted skills, and vector memories in PostgreSQL.
2. Ingest and index the supplied career reference corpus.
3. Implement hybrid retrieval using semantic and keyword search.
4. Orchestrate AI processing with LangGraph and typed Pydantic state.
5. Evaluate generated recommendations for consistency with reference material.
6. Extract professional skills asynchronously without blocking chat responses.
7. Stream chatbot responses and update the career skill tree in real time.
8. Preserve conversation history and archive skills when users switch career tracks.
9. Support configurable, locally served language models through Ollama.
10. Ensure reliability, testability, reproducibility, and maintainability.

## 3. Technology Stack

| Component | Technology |
|---|---|
| Frontend | Next.js, React, TypeScript |
| Backend API | FastAPI, Python 3.11+ |
| Database | PostgreSQL |
| Vector search | pgvector |
| ORM | SQLAlchemy 2.x, asyncpg |
| Database migrations | Alembic |
| Workflow orchestration | LangGraph |
| State validation | Pydantic |
| Keyword retrieval | PostgreSQL full-text search |
| Model serving | Ollama |
| Streaming | Server-Sent Events (SSE) |
| Containerization | Docker Compose |
| Testing | pytest |
| CI | GitHub Actions |

## 4. High-Level Architecture

The system is divided into five major layers.

### 4.1 Frontend Layer

The Next.js frontend provides:

- A conversational chat interface.
- A live career skill tree.
- Streaming assistant responses.
- Session and career-track visualization.

### 4.2 Backend API Layer

The FastAPI backend provides:

- Session and message management.
- Conversation processing endpoints.
- Streaming responses.
- Skill retrieval endpoints.
- Career-track management.
- Input validation and error handling.

### 4.3 AI Orchestration Layer

LangGraph coordinates:

- Career Intent Router
- Query Formulation
- Hybrid Retrieval
- Strategist Generator
- Reality-Check Evaluator
- Career-Switch Handler

A separate asynchronous worker performs skill extraction.

### 4.4 Persistence and Retrieval Layer

PostgreSQL stores:

- Sessions
- Messages
- Career tracks
- Extracted skills
- Vector memories
- Reference document chunks and metadata

pgvector provides cosine-similarity search, while PostgreSQL full-text search provides keyword matching using `tsvector` and GIN indexing.

### 4.5 Model Serving Layer

Ollama provides configurable local language-model inference.

The model identifier is provided through environment configuration. Model weights are not committed to Git.

## 5. Database Design

The proposed relational schema contains the following entities.

### Sessions

Stores conversation-session metadata.

Proposed fields:

- `id` — UUID primary key
- `created_at` — timestamp
- `updated_at` — timestamp
- `active_track_id` — nullable reference to active career track

### Messages

Stores user and assistant messages.

Proposed fields:

- `id` — UUID primary key
- `session_id` — foreign key
- `role` — user or assistant
- `content` — message text
- `created_at` — timestamp

### CareerTracks

Stores separate career directions associated with a session.

Proposed fields:

- `id` — UUID primary key
- `session_id` — foreign key
- `name` — career-track name
- `version` — version number
- `status` — active or archived
- `created_at` — timestamp
- `archived_at` — nullable timestamp

### ExtractedSkills

Stores skills extracted from conversations.

Proposed fields:

- `id` — UUID primary key
- `session_id` — foreign key
- `track_id` — foreign key
- `source_message_id` — foreign key
- `skill` — extracted skill name
- `proficiency_mentioned` — beginner, intermediate, advanced, or unspecified
- `created_at` — timestamp

### VectorMemories

Stores vectorized reference passages.

Proposed fields:

- `id` — UUID primary key
- `content` — reference text
- `embedding` — vector
- `source` — source document identifier
- `page_number` — source page
- `chunk_index` — chunk position
- `search_vector` — PostgreSQL tsvector
- `created_at` — timestamp

Foreign keys will enforce referential integrity. Appropriate cascading deletes will be configured, while career-switch archiving will preserve historical data.

The complete proposed ERD will be maintained in `docs/diagrams/database-erd.md`.

## 6. Knowledge Base Ingestion

The ingestion pipeline processes the organization-supplied career reference corpus.

The pipeline will:

1. Extract text from the supplied reference document.
2. Preserve source and page metadata.
3. Split extracted text into chunks respecting paragraph and sentence boundaries.
4. Apply configurable chunk overlap.
5. Generate embeddings for each chunk.
6. Save chunks and embeddings in PostgreSQL.
7. Create the required vector and full-text indexes.

Chunk size, overlap and embedding-model selection will be tested and documented.

## 7. Hybrid Retrieval

The retrieval system combines two methods.

**Semantic retrieval**

Uses pgvector cosine similarity to retrieve passages with similar meanings.

**Keyword retrieval**

Uses PostgreSQL full-text search to retrieve passages matching specific job titles and technical terms.

**Rank fusion**

Reciprocal Rank Fusion (RRF) combines results from both retrieval methods.

A configurable top-k parameter controls the number of results returned.

Retrieval evaluation will use at least 20 labelled queries and report:

- Recall@5
- Mean Reciprocal Rank (MRR)

Results will be compared across semantic-only, keyword-only and hybrid retrieval.

## 8. LangGraph Workflow

The workflow maintains a typed state containing:

- Conversation messages
- Retrieved context
- Route decision
- Evaluation critique
- Retry count
- Active career track
- Execution status

### 8.1 Career Intent Router

Determines whether a message needs knowledge-base retrieval or conversational processing.

### 8.2 Query Formulation

Rewrites retrieval-oriented messages into search queries optimized for the career reference corpus.

### 8.3 Strategist Generator

Generates the assistant response using conversation history and available reference context.

### 8.4 Reality-Check Evaluator

Checks whether generated career recommendations contradict retrieved reference material.

If a contradiction is found:

1. Reject the candidate response.
2. Attach evaluation feedback to the workflow state.
3. Regenerate the answer using the critique.
4. Stop after a maximum of three retries.

If all retries fail, the system returns a controlled fallback message instead of unsupported recommendations.

Only approved responses will be presented as final recommendations. Streaming must be designed to avoid presenting rejected drafts as accepted answers.

### 8.5 Career-Switch Handler

Detects an intentional change of career direction.

When a switch is confirmed:

1. Archive the current career track.
2. Preserve its associated skills.
3. Create a new active career track.
4. Keep the complete conversation history.
5. Allow previous track data to be restored.

The full workflow diagram will be maintained in `docs/diagrams/workflow.md`.

## 9. Asynchronous Skill Extraction

Skill extraction runs outside the user-facing response path.

The extractor receives eligible conversation messages and produces validated JSON.

Expected format:

```json
{
  "skill": "Python",
  "proficiency_mentioned": "intermediate"
}
```

Supported proficiency values are:

- `beginner`
- `intermediate`
- `advanced`
- `unspecified`

The extractor persists validated results to PostgreSQL.

Its failures must not interrupt response generation.

A durable database-backed background-job approach is proposed for reliable processing and retry handling. The final worker design will be documented before implementation.

## 10. Frontend and Streaming

The frontend uses a split-screen layout.

**Left panel:** Conversational chat.

**Right panel:** Live career skill tree.

Server-Sent Events will stream approved response content to the frontend.

The skill tree will initially use lightweight polling to detect newly persisted skills without refreshing the page.

The polling interval will be configurable. This design choice will be documented in an ADR.

## 11. Concurrency and Failure Handling

Messages arriving rapidly within the same session must be processed in a predictable order.

A session-scoped locking or queue mechanism will serialize state-changing operations.

The backend will also define handling for:

- Database failures
- Retrieval failures
- Model-serving timeouts
- Evaluation retry exhaustion
- Background skill-extractor failures
- Streaming connection interruptions

All retries will have explicit limits.

## 12. Security and Data Protection

The application will:

- Validate API inputs using Pydantic.
- Treat user inputs and retrieved documents as untrusted.
- Avoid storing credentials in source control.
- Use environment variables for configuration.
- Store only synthetic conversational data.
- Avoid production data and real personal information.
- Exclude model weights from the repository.

Authentication and multi-tenancy are outside the initial project scope.

## 13. Testing Strategy

Planned automated tests include:

- Database relationship and cascading-delete tests.
- Ingestion and chunking tests.
- Retrieval fusion and ranking tests.
- LangGraph routing tests.
- Evaluation-loop retry and fallback tests.
- Skill extraction and validation tests.
- Career-switch restoration tests.
- Concurrent message-ordering tests.
- Background worker failure-isolation tests.

LLM responses will be mocked or replayed to keep tests deterministic.

The backend core logic target is at least 70% line coverage.

## 14. Observability and Performance

Each request will receive a correlation ID.

Structured logs will record:

- Workflow-node execution
- Retrieval operations
- Model inference
- Evaluation retries
- Background extractor status
- Errors and failure outcomes

Performance measurements will include time-to-first-token and end-to-end latency.

Specific latency targets must be agreed with the reviewer.

## 15. Deployment and Reproducibility

The application will use Docker Compose for local execution.

Planned services include:

- FastAPI backend
- Next.js frontend
- PostgreSQL with pgvector
- Local model-serving integration
- Background skill-extraction worker

A complete setup and runbook will be delivered during the final milestone.

## 16. Technical Design Status

This document describes the initial proposed architecture.

Database constraints, worker infrastructure, streaming behavior, and model choices will be refined during implementation and reviewer feedback.

The design is not considered final until the architecture review is completed and the implementation reflects the approved decisions.

## 17. Supporting Documents

- [Database ERD](diagrams/database-erd.md)
- [LangGraph Workflow](diagrams/workflow.md)
- [Sequence Diagrams](diagrams/sequence.md)
- [ADR 001 — Database Design](adr/001-database-design.md)
- [ADR 002 — Hybrid Retrieval](adr/002-hybrid-retrieval.md)
- [ADR 003 — Background Skill Extraction](adr/003-background-skill-extraction.md)