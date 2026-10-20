# ADR 003 — Asynchronous Skill Extraction and Live Skill Updates

**Status:** Proposed  
**Related Issue:** #1

## Context

Career Path Strategist must extract skills from conversations without delaying the user's chat response.

It must also update the frontend when extracted skills are stored.

Background-processing failures must not affect the chat workflow.

Rapid messages and career switches introduce ordering and consistency challenges.

## Decision

Use a durable, database-backed background-job approach for skill extraction.

A separate worker will process pending extraction jobs.

Use polling as the initial frontend skill-update mechanism.

## Processing Flow

1. The chat workflow processes and persists a message.
2. A background extraction job is recorded.
3. The response proceeds without waiting for skill extraction.
4. The worker claims a pending job.
5. The worker calls the configured model.
6. The model produces structured skill data.
7. Pydantic validates the output.
8. Validated skills are persisted under the originating career track.
9. The worker marks the job completed.
10. Frontend polling detects updated skills.

## Structured Output

Each extracted skill must follow this schema:

```json
{
  "skill": "Python",
  "proficiency_mentioned": "intermediate"
}
```

The proficiency field must contain one of:

- beginner
- intermediate
- advanced
- unspecified

Invalid model output will be rejected or retried according to a bounded retry policy.

## Career-Track Consistency

Each extraction job stores its source message and original career-track identifier.

If a user changes career direction while an extraction job is running, the result remains associated with the original track.

This prevents old skills from appearing incorrectly in the new active track.

## Failure Handling

Worker failures must not terminate the user-facing chat request.

The proposed job system will support:

- Pending, processing, completed and failed statuses.
- Bounded retries.
- Recorded error information.
- Safe recovery of abandoned processing jobs.
- Idempotent persistence of extracted skills.

## Alternatives Considered

### FastAPI BackgroundTasks

Advantages:

- Simple integration.
- Minimal infrastructure.

Disadvantages:

- In-process background execution is not durable across service restarts.
- Less suitable for reliable retry and recovery requirements.

### External Queue and Worker

Advantages:

- Mature job-processing and retry capabilities.
- Supports more complex workloads.

Disadvantages:

- Additional infrastructure and configuration.

### Database-Backed Job Queue

Advantages:

- Reuses PostgreSQL.
- Can persist work across process restarts.
- Reduces initial infrastructure complexity.

Disadvantages:

- Requires careful worker coordination.
- Requires explicit retry, locking and recovery logic.

The database-backed approach is proposed for the initial sandbox implementation.

## Live Skill Tree Updates

The initial frontend will poll the backend periodically for active-track skills.

### Alternatives

**Polling**

- Simple to implement.
- Easy to test.
- Introduces a configurable update delay.

**Server-Sent Events**

- Supports server-to-client event streaming.
- Requires persistent streaming connection management.

**WebSockets**

- Supports bidirectional real-time messaging.
- Adds connection-management complexity.

Polling is selected initially because the skill tree only needs to receive updates after database changes.

## Testing

Tests will cover:

- Background execution.
- Valid and invalid extracted JSON.
- Worker failure isolation.
- Retry exhaustion.
- Duplicate-job handling.
- Rapid session messages.
- Career switching during extraction.
- Skill-tree updates.

## Consequences

The design adds a worker process and background-job persistence.

In exchange, it provides better reliability and separates skill extraction latency from chat latency.

## Review Status

Pending design approval and implementation.