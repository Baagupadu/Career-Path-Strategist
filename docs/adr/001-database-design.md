# ADR 001 — PostgreSQL Database and Career-Track Versioning

**Status:** Proposed  
**Related Issue:** #1

## Context

Career Path Strategist must persist sessions, conversation messages, extracted skills and searchable reference passages.

The system must also support career switching without losing conversation history or previously extracted skills.

Data consistency and referential integrity are mandatory.

## Decision

Use PostgreSQL with pgvector as the primary database.

Use:

- SQLAlchemy 2.x asynchronous ORM.
- asyncpg for asynchronous database access.
- Alembic for versioned schema migrations.
- Foreign-key constraints and cascading deletes.
- A separate CareerTracks entity for career-direction versioning.
- Logical archival rather than deleting old career-track data.

Each session references its active career track.

Extracted skills are associated with the career track and message that produced them.

When a user switches careers, the existing track is archived and a new or previously archived track becomes active.

All track updates must be transactional.

## Alternatives Considered

### Alternative 1: MongoDB

Advantages:

- Flexible document structures.
- Convenient storage of conversational data.

Disadvantages:

- Different approach to relational consistency.
- Additional complexity for the required PostgreSQL vector and keyword retrieval stack.

### Alternative 2: Separate Vector Database

Advantages:

- Specialized vector retrieval capabilities.

Disadvantages:

- Another database service to maintain.
- Additional synchronization between relational and vector data.

### Alternative 3: Permanently Delete Old Skills

Advantages:

- Simpler storage model.

Disadvantages:

- Historical skills would be lost.
- Previous tracks could not be restored.

## Consequences

### Benefits

- Relational consistency.
- Integrated vector and keyword search.
- Restorable historical career tracks.
- Version-controlled database migrations.

### Trade-offs

- Additional foreign-key relationships.
- More complex active-track constraints.
- Transactional handling of career switches.
- Database migrations must be coordinated carefully.

## Implementation Notes

The schema will include session ownership constraints for career tracks and skills.

It will enforce at most one active career track per session using an appropriate database constraint.

The active-track reference and cascading relationships must be designed so session deletion and archival remain safe.

The exact embedding dimension will be decided after choosing the embedding model.

## Validation

The design will be verified through migration, foreign-key, cascade-deletion and career-track restoration tests.

## Review Status

Pending technical review and implementation validation.