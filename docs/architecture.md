# Architecture Notes

## Platform Context
- Databricks Community Edition used for compute, notebooks, and orchestration
- Data stored in Unity Catalog Volumes under a controlled workspace namespace
- PySpark notebooks treated as pipeline units, versioned daily in Git

This setup prioritizes reproducibility, clear ownership, and portability over convenience.

---

## Medallion Architecture – Intentional Design

The Medallion architecture is implemented as an operational contract, not a cosmetic layering.

Each layer exists to absorb a specific class of failure without leaking instability downstream.

---

### Bronze Layer – System of Record

The Bronze layer preserves source data with minimal intervention.

- Raw events are ingested with full fidelity
- Operational metadata (batch_id, ingestion_ts) is added for traceability
- No business rules or corrections are applied

Bronze exists to support replay, auditability, and forensic debugging.  
Correctness is intentionally deferred.

---

### Silver Layer – System of Truth

The Silver layer is where data contracts are enforced.

- Schemas are normalized and stabilized
- Deterministic deduplication is applied using business keys and ingestion order
- Data quality rules are explicit and repeatable

Silver is designed for idempotency and consistency.  
Every downstream metric assumes Silver is correct by construction.

---

### Gold Layer – System of Insight

The Gold layer exposes consumer-facing aggregates.

- Built exclusively from Silver data
- Shaped by access patterns, not source structure
- Optimized for analytical queries and semantic clarity

Gold datasets are insulated from ingestion mechanics so analytical use cases can evolve independently.

---

## Storage and Transaction Semantics

Delta Lake underpins all layers.

- ACID guarantees ensure safe concurrent writes
- Schema enforcement prevents silent drift
- Versioned storage enables time travel and controlled backfills

State is managed through data, not application logic.  
Determinism is favored over implicit optimizations.

---

## Orchestration as Architecture

Pipeline execution is treated as a first-class architectural concern.

- Databricks Jobs define layer dependencies (Bronze → Silver → Gold)
- Notebooks are parameterized and stateless
- Guardrails prevent accidental cross-layer writes

Failures are isolated by layer to avoid partial corruption.

---

## Governance and Catalog Design

Governance follows storage, not the other way around.

- Volumes define physical ownership and constrain catalog usage
- Delta tables are materialized before metastore registration
- Schemas represent ownership boundaries
- Views act as consumer contracts

External tables are preferred to decouple data lifecycle from metadata lifecycle and support replayability.

Principle: catalogs organize data; they do not own it.

---

## Design Principles

- Correctness over convenience  
- Reproducibility over speed  
- Explicit contracts over implicit assumptions  

This architecture is optimized for scale, replayability, and long-term maintainability, not short-lived experimentation.s