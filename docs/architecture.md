# Architecture Notes

## Current approach
- Databricks Community Edition for compute and notebooks
- Dataset stored in a controlled workspace location (Volumes)
- PySpark notebooks as the primary artifact, exported to this repo daily

## Medallion Architecture – Design Rationale (Day 6)

The Medallion architecture is implemented as a deliberate separation of concerns rather than a purely technical layering.

### Bronze Layer – System of Record
The Bronze layer preserves raw source data with minimal transformation. Its primary responsibility is **fidelity and traceability**, not correctness.  
Audit metadata such as batch identifiers and ingestion timestamps are introduced at this stage to support replay, lineage, and operational debugging.

No business rules are enforced here by design. Any correction at this layer would compromise the ability to reason about upstream data quality and source behavior.

### Silver Layer – System of Truth
The Silver layer represents the first point where **data contracts** are enforced.  
This includes schema normalization, data quality gates, and deterministic deduplication to ensure idempotent processing.

Deduplication is handled explicitly using stable business keys and ordering logic, rather than relying on implicit assumptions or append-only behavior.  
This layer is optimized for correctness and consistency, forming the foundation for all downstream analytics.

### Gold Layer – System of Insight
The Gold layer exposes **business-ready aggregates** derived from Silver data.  
Datasets at this level are shaped by access patterns rather than source structure, prioritizing query efficiency, semantic clarity, and consumer usability.

Gold outputs are intentionally decoupled from ingestion mechanics, allowing analytical use cases to evolve without destabilizing upstream processing.

### Storage and Transaction Semantics
Delta Lake underpins all layers, providing ACID guarantees, schema enforcement, and versioned storage.  
These capabilities are leveraged to support safe incremental processing, controlled backfills, and time-based debugging without introducing custom state management.

The architecture favors explicit transformations and deterministic behavior over implicit optimizations, ensuring predictable outcomes as data volume and complexity grow.

