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

## Analytics & Semantic Layer Design

The analytics layer is treated as a separate architectural concern from ingestion and transformation.

### Semantic Layer as a Contract

Gold views form a semantic contract between data producers and consumers.

- Metric definitions are centralized and versionable
- Dashboards depend on semantics, not raw events
- Changes to source data do not ripple into analytical consumers

This approach reduces cognitive load, simplifies governance, and makes analytical behavior predictable.

### Performance Strategy

Two classes of analytical assets are used:

- **Semantic views** for flexibility and correctness
- **Snapshot tables** for dashboard performance and stable latency

Snapshots are designed to be refreshed on a schedule and are intentionally derived from semantic views to preserve consistency.

### Data Quality as Analytics

Data quality is surfaced alongside business metrics.

Null rates, invalid values, and completeness trends are exposed as Gold datasets, reinforcing the idea that analytical insight includes confidence in the underlying data.

Principle:
Analytics is not just about insight.  
It is about trust at scale.

## Performance Optimization

- Inspected query plans before changing storage to confirm pruning, pushdown, and shuffle behavior.
- Partitioned Silver events by `event_date` and `event_type` to align with dominant analytical filters.
- Avoided high-cardinality partition keys (user_id, product_id) to prevent small-file sprawl and skew.
- Treated file compaction as mandatory; used OPTIMIZE/ZORDER when available, otherwise enforced controlled rewrites.
- Benchmarked representative workload queries to validate improvements and avoid placebo tuning.
- Used caching only for short-lived, iterative analytics and explicitly uncached to keep results honest.

Key takeaway:  
Performance is shaped by data layout and access patterns, not last-mile query tweaks.


## Statistical Analysis and ML Feature Layer

Day 11 introduces a bridge between analytics and ML by formalizing two Gold assets:

- **gold.daily_kpis**: stable-grain dataset for statistical testing and trend analysis
- **gold.ml_features_purchase_intent**: reusable feature table for model training

Key design choices:
- Statistical tests operate on aggregated KPIs to ensure comparable samples.
- Feature engineering is treated as a governed layer, not notebook logic.
- Features combine time signals, user behavior history, user lifetime aggregates, and product popularity signals.

Principle:
ML readiness comes from reproducible feature tables, not ad-hoc transformations.

## ML Experiment Architecture

ML experiments are treated as a governed workflow:

- Training datasets are materialized as Gold tables to make inputs reproducible.
- Evaluation uses time-based splits to reflect production behavior and prevent leakage.
- MLflow is used for tracking parameters, metrics, and model artifacts to support comparison and iteration.

Principle:
If experiments can’t be reproduced from versioned inputs, they’re not engineering artifacts.
