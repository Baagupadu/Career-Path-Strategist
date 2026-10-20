# ADR 002 — Hybrid Retrieval Using pgvector, Full-Text Search and RRF

**Status:** Proposed  
**Related Issue:** #1

## Context

The career reference corpus contains job titles, technical terms and descriptions of professional skills.

Semantic retrieval can identify conceptually related passages but may fail to prioritize exact terms.

Keyword retrieval can identify exact terminology but may miss relevant passages using alternative wording.

The project requires comparison of retrieval strategies.

## Decision

Use hybrid retrieval combining:

1. Semantic similarity search using pgvector cosine distance.
2. PostgreSQL full-text search using tsvector and GIN indexes.
3. Reciprocal Rank Fusion (RRF) to combine ranked results.

The final number of returned passages will be controlled through a configurable top-k parameter.

## Retrieval Process

1. Receive a career-related question.
2. Generate a retrieval-optimized search query.
3. Generate its embedding.
4. Execute semantic vector retrieval.
5. Execute keyword-based full-text retrieval.
6. Merge duplicate passages using their stable chunk identifiers.
7. Calculate Reciprocal Rank Fusion scores.
8. Sort the fused results.
9. Return the top-k passages with source and page metadata.

## Reciprocal Rank Fusion

The proposed scoring formula is:

RRF(d) = sum(1 / (k + rank(d)))

Where:

- d is a retrieved passage.
- rank(d) is its position in a retrieval result list.
- k is a configurable fusion constant.

An initial k value of 60 is proposed as an experiment, not a proven optimum.

## Alternatives Considered

### Semantic-only Retrieval

Advantages:

- Captures similarity in meaning.
- Handles paraphrased questions.

Disadvantages:

- May not prioritize exact technical terminology.

### Keyword-only Retrieval

Advantages:

- Effective for exact indexed terms.
- Straightforward ranking and inspection.

Disadvantages:

- Can miss semantically relevant passages.

### Weighted Score Fusion

Advantages:

- Supports explicit weighting of retrieval methods.

Disadvantages:

- Requires normalization and calibration across different score scales.

### Reciprocal Rank Fusion

Advantages:

- Combines rankings without directly comparing incompatible similarity scores.
- Simple to implement and evaluate.

Disadvantages:

- Ignores the magnitude of original retrieval scores.
- Quality depends on candidate selection and ranking.

## Evaluation Strategy

Create at least 20 labelled queries using the approved reference corpus.

The evaluation set will include:

- Exact job-title queries.
- Exact tool-name queries.
- General career-path questions.
- Paraphrased career requirements.

Report:

- Recall@5.
- Mean Reciprocal Rank (MRR).

Compare:

- Semantic-only retrieval.
- Keyword-only retrieval.
- Hybrid retrieval.

The evaluation dataset, ranking configuration and results will be documented during Milestone 2.

No improvement claim will be made until the measurements are available.

## Consequences

The hybrid approach requires maintaining embeddings, tsvector values and their associated indexes.

The team must measure retrieval relevance, latency and ingestion consistency.

## Review Status

Pending evaluation results and reviewer approval.