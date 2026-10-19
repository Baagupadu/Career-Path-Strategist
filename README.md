# Career Path Strategist

An AI-powered career guidance application developed as part of the Baagupadu Intern Sandbox Programme.

## Project Overview

Career Path Strategist is an AI-based conversational system designed to help users explore career paths, identify professional skills, and understand the competencies associated with different technology roles.

The application will use retrieval-augmented generation (RAG) to ground career recommendations in a reference knowledge base. It will also extract skills from conversations asynchronously and visualize them through a live career skill tree.

This is an engineering demonstration project. Its outputs are not intended to constitute professional career advice.

## Objectives

- Develop an AI-powered conversational career guidance system.
- Implement hybrid retrieval using semantic and keyword search.
- Generate responses grounded in a reference career knowledge base.
- Validate generated recommendations through a bounded evaluation loop.
- Extract professional skills asynchronously without blocking chat responses.
- Visualize extracted skills through an interactive, live-updating skill tree.
- Preserve conversation history and archived skills when users change career direction.
- Support local model serving and reproducible development environments.

## Technology Stack

| Layer | Technology |
|---|---|
| Frontend | Next.js, React, TypeScript |
| Backend | Python 3.11+, FastAPI |
| Database | PostgreSQL, pgvector |
| ORM | SQLAlchemy 2.x (async), asyncpg |
| Database migrations | Alembic |
| AI workflow orchestration | LangGraph, Pydantic |
| Retrieval | Vector similarity search, PostgreSQL full-text search, Reciprocal Rank Fusion |
| Local model serving | Ollama, quantized GGUF model |
| Containerization | Docker Compose |
| Testing | pytest |
| CI/CD | GitHub Actions |

## Planned Features

### 1. AI Career Guidance

A conversational AI assistant that discusses a user's professional background, skills, interests, and career ambitions.

### 2. Hybrid Knowledge Retrieval

Combines semantic vector search with keyword-based PostgreSQL full-text search to retrieve relevant information from the career reference corpus.

### 3. Response Evaluation

A separate evaluation stage checks whether generated career recommendations contradict the retrieved reference material.

Contradictory responses are rejected and regenerated through a bounded retry mechanism, with a maximum of three retries.

### 4. Asynchronous Skill Extraction

Extracts professional skills and their mentioned proficiency levels from conversations using background processing.

Supported proficiency labels include:

- Beginner
- Intermediate
- Advanced
- Unspecified

### 5. Live Career Skill Tree

A split-screen interface featuring conversational chat and an automatically updating visualization of extracted skills.

### 6. Career-Switch Management

Allows users to explore a new career direction while preserving previous conversations and maintaining restorable, archived skills from earlier career tracks.

### 7. Local Model Serving

Supports a configurable locally served language model using Ollama.

## Planned Repository Structure

```text
Career-Path-Strategist/
├── backend/           # FastAPI, retrieval and AI workflows
├── frontend/          # Next.js chat interface and skill tree
├── ml/                # Dataset curation and model configuration
├── data/              # Approved synthetic data and corpus references
├── docs/              # Architecture, ADRs and technical reports
├── tests/             # Unit and integration tests
├── .github/           # CI workflows and repository templates
├── .env.example       # Example configuration (no secrets)
├── docker-compose.yml # Local service orchestration
└── README.md
```

This represents the target repository structure. Directories and configuration files will be added during development.

## Development Roadmap

| Milestone | Duration | Focus |
|---|---|---|
| M1 | Weeks 1–2 | System design, database schema, migrations and ingestion |
| M2 | Weeks 3–4 | Hybrid retrieval, evaluation and LangGraph orchestration |
| M3 | Weeks 5–6 | Asynchronous extraction, streaming chat and live skill tree |
| M4 | Weeks 7–8 | Dataset curation, local model serving, testing and final review |

## Getting Started

The project is currently in its initial setup and architecture-design phase.

Development environment setup instructions, application startup commands and Docker Compose configuration will be documented as the implementation progresses.

**Prerequisites (planned):**

- Python 3.11+
- Node.js
- PostgreSQL with pgvector
- Docker and Docker Compose
- Ollama
- Git

## Development Workflow

All development follows an issue-based Git workflow.

1. Create or select a GitHub Issue with acceptance criteria.
2. Create a short-lived working branch using the naming convention `type/ISSUE-ID-short-description`.
3. Implement the change and add appropriate tests or documentation.
4. Commit changes using Conventional Commits.
5. Submit a pull request linked to the Issue.
6. Ensure required CI checks pass and obtain reviewer approval.
7. Squash merge the approved pull request into `main`.

Direct development commits to `main` are not part of the normal workflow.

## Testing and Quality Standards

The project targets:

- Automated unit and integration testing.
- At least 70% line coverage for backend core logic.
- Typed Python and TypeScript code.
- Linting and static analysis through CI.
- Deterministic tests using mocked or recorded LLM responses.
- Defined failure handling and bounded retries.
- Reproducible local execution using Docker Compose.

## Documentation

Technical documentation will be maintained under the `docs/` directory, including:

- Technical Design Document (`docs/ARCHITECTURE.md`)
- Database and workflow diagrams
- Architecture Decision Records (`docs/adr/`)
- Retrieval and evaluator test reports
- API documentation
- Operations runbook

## Data Protection

The project is intended to use only approved synthetic data and the reference materials supplied by the organization.

Credentials, real personal data, production data, and model weights must not be committed to the repository.

## Project Status

**Phase:** Initial repository setup and technical design

**Current milestone:** M1 — Design and Data Layer

**Programme duration:** 8 weeks

## Ownership

Developed as part of the Baagupadu Intern Sandbox Programme.

Project materials and deliverables are governed by the organization's confidentiality and intellectual property requirements.