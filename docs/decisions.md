# Engineering Decisions Log

This file captures small decisions made daily that improve repeatability, clarity, and scale.

## Day 0
- Preferred Volumes (Unity Catalog) for dataset storage instead of ad-hoc uploads, to keep paths consistent and permissions explicit.

## Day 1
- Keep transformations modular and readable: load → validate → transform → aggregate.
- Always validate row count and schema before any transformation work.
- Focused on validating core PySpark operations on the Oct 2019 dataset.
- Deliberately kept transformations simple to establish a clean baseline before ingestion and cleaning in Day 2.

## Day 2
- Standardized event_time into event_ts and normalized price type for consistent downstream processing.
- Combined Oct and Nov datasets with minimal cleaning rules (null/negative price filtering).

## Day 3
- Chose PySpark transformations over Pandas for all non-trivial operations to keep execution distributed and scalable.
- Used window functions for user-level sequencing and ranking to avoid expensive self-joins.
- Modeled derived features (revenue, purchase flags, conversion rates) explicitly instead of embedding logic inside aggregations.
- Demonstrated joins using event-derived dimensions rather than external lookup tables to stay within dataset scope.
- Used UDFs sparingly and only for cases not covered by native Spark functions, acknowledging performance trade-offs.

## Day 4
- Converted raw CSV data to Delta format to enable ACID guarantees and schema enforcement.
- Created Delta tables using both PySpark and SQL to validate interoperability.
- Verified schema enforcement behavior and documented duplicate insert handling.

## Day 5
- Implemented incremental Delta MERGE for upsert semantics instead of append-only ingestion.
- Encountered and resolved Delta MERGE ambiguity caused by multiple source rows matching the same target keys.
- Introduced deterministic batch deduplication using window functions prior to MERGE to guarantee idempotency.
- Used Delta time travel to validate version history and confirm MERGE operations.
- Attempted OPTIMIZE and VACUUM with graceful handling of environment-level restrictions.

## Day 6
- Implemented a Medallion architecture using Delta Lake with explicit Bronze, Silver, and Gold layers.
- Bronze layer captures raw data with audit metadata (batch_id, ingestion_ts) to support traceability and replay.
- Silver layer applies schema standardization, data quality gates, and deterministic deduplication using window functions to ensure idempotent processing.
- Gold layer produces business-ready aggregates (product and category performance) optimized for analytical consumption.
- Chose overwrite-based builds for clarity and determinism, with MERGE-based incremental patterns demonstrated separately.

## Day 7 – Orchestration & Job Design Decisions

**Why Databricks Jobs instead of chaining notebooks manually**
- Jobs enforce stateless execution, eliminating hidden dependencies from interactive sessions.
- Each task runs in a clean process, exposing configuration and schema issues early.

**Why parameters via widgets**
- Enables the same notebook to run across environments and layers.
- Prevents hardcoded paths, which are a common source of production incidents.
- Makes Bronze → Silver → Gold orchestration explicit and auditable.

**Why strict guardrails between layers**
- Explicit checks prevent accidental writes from Silver into Bronze paths.
- This avoids irreversible corruption of raw data and preserves replayability.

**Why deterministic deduplication**
- Business events can arrive late or duplicated.
- Deduplication based on natural keys + ingestion timestamp ensures idempotent processing.

**Why schema enforcement over flexibility**
- Delta Lake schema enforcement is treated as a contract, not a nuisance.
- Schema drift is surfaced immediately rather than propagating silently downstream.

**Key takeaway**
- Jobs expose architectural weaknesses early.
- Interactive notebooks hide them.

## Day 8 – Governance Decisions

- Validated physical data access before applying governance constructs.
- Wrote Delta tables to storage first, then registered them in the metastore.
- Aligned catalog usage with Volume ownership to avoid storage–metadata conflicts.
- Preferred external Delta tables to decouple data lifecycle from metadata.
- Used managed tables only as a fallback when external registration is restricted.

Key point: governance must follow storage, not the other way around.
