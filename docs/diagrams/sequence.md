# Sequence Diagrams

**Project:** Career Path Strategist  
**Related Issue:** #1  
**Status:** Proposed design

## 1. Chat and Response Evaluation

```mermaid
sequenceDiagram
    autonumber
    actor User
    participant UI as Next.js Frontend
    participant API as FastAPI
    participant DB as PostgreSQL
    participant Graph as LangGraph
    participant Search as Hybrid Retriever
    participant LLM as Local LLM

    User->>UI: Submit career question
    UI->>API: POST /sessions/{id}/messages
    API->>API: Validate input and acquire session lock
    API->>DB: Load session and conversation
    API->>Graph: Execute workflow
    Graph->>Graph: Classify career intent
    opt Retrieval required
        Graph->>Search: Formulate and execute query
        Search->>DB: Vector and keyword search
        DB-->>Search: Ranked reference chunks
        Search-->>Graph: Fused context
    end
    loop Until approved or retry limit
        Graph->>LLM: Generate candidate
        LLM-->>Graph: Candidate answer
        Graph->>LLM: Evaluate groundedness
        LLM-->>Graph: Verdict and critique
    end
    alt Response approved
        Graph-->>API: Approved response
        API->>DB: Persist approved message
        API-->>UI: Stream approved text via SSE
    else Retries exhausted or evaluation failed
        Graph-->>API: Controlled fallback
        API->>DB: Persist fallback message
        API-->>UI: Stream fallback via SSE
    end
    API->>DB: Queue skill extraction work
    API->>API: Release session lock
    UI-->>User: Display assistant response
```

The first implementation will evaluate candidate responses before making them visible. Therefore time-to-first-token includes evaluation time.

The session lock ensures messages in the same session are processed in a defined order.

## 2. Asynchronous Skill Extraction

```mermaid
sequenceDiagram
    autonumber
    participant API as FastAPI
    participant DB as PostgreSQL
    participant Worker as Background Worker
    participant LLM as Local LLM
    participant UI as Skill Tree

    API->>DB: Persist skill-extraction job
    API-->>UI: Complete chat response
    Worker->>DB: Claim pending job
    Worker->>LLM: Extract mentioned skills
    LLM-->>Worker: Structured JSON output
    Worker->>Worker: Validate skill and proficiency
    alt Valid output
        Worker->>DB: Persist skills for source track
        Worker->>DB: Mark job completed
    else Processing failure
        Worker->>DB: Record error and retry status
    end
    loop At configured interval
        UI->>API: GET active track skills
        API->>DB: Query persisted skills
        DB-->>API: Skill records
        API-->>UI: Current skill tree data
    end
```

Skill extraction is independent of the request-response path.

A durable job record is proposed to protect queued work from process restarts.

The extraction job stores the track identifier from the originating message, preventing delayed work from being incorrectly assigned to a newer career track.

## 3. Career-Switch Handling

```mermaid
sequenceDiagram
    autonumber
    actor User
    participant UI as Next.js Frontend
    participant API as FastAPI
    participant Graph as LangGraph
    participant DB as PostgreSQL

    User->>UI: Request career direction change
    UI->>API: Submit new message
    API->>API: Acquire session lock
    API->>Graph: Classify career-switch intent
    Graph-->>API: Career switch detected
    API->>DB: Begin transaction
    DB->>DB: Archive previous career track
    DB->>DB: Create or restore requested track
    DB->>DB: Update session active track
    DB->>DB: Preserve messages and historical skills
    API->>DB: Commit transaction
    API->>Graph: Continue with updated track
    Graph-->>API: Approved response
    API-->>UI: Stream response
    UI->>API: Fetch active track skills
    API-->>UI: Updated skill tree
    API->>API: Release session lock
```

## 4. Proposed API Contracts

| Method | Endpoint | Purpose |
|---|---|---|
| POST | `/sessions` | Create conversation session |
| GET | `/sessions/{id}` | Retrieve session information |
| POST | `/sessions/{id}/messages` | Process chat message and stream response |
| GET | `/sessions/{id}/messages` | Retrieve conversation history |
| GET | `/sessions/{id}/skills` | Retrieve active-track skills |
| GET | `/sessions/{id}/tracks` | Retrieve career-track history |
| POST | `/sessions/{id}/tracks/switch` | Switch or restore career direction |

These endpoints are proposed API contracts and may be refined during implementation.

## 5. Consistency and Concurrency

Operations modifying a session's active track must execute under session-level serialization.

Background extraction jobs retain their originating message and track identifiers.

Failed database transactions must roll back without leaving a partially changed career track.

## 6. Status

These diagrams describe the intended system interactions. They will be updated to match the implemented API and background-processing architecture.