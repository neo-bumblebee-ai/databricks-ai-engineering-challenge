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
