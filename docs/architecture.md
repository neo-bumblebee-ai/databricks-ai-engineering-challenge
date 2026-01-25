# Architecture Notes

## Platform Context
- Databricks Community Edition used for compute, notebooks, and orchestration
- Data stored in Unity Catalog Volumes under a controlled workspace namespace
- PySpark notebooks treated as pipeline units, versioned daily in Git

This setup prioritizes reproducibility, clear ownership, and portability over convenience.

---

## What Gets Built

### Layers and primary assets
- **Bronze:** raw event ingestion + audit metadata (`batch_id`, `ingestion_ts`)
- **Silver:** normalized schema + deterministic deduplication + quality gates
- **Gold:** consumer-facing analytical tables and views
  - `gold.daily_kpis`
  - `gold.ml_features_purchase_intent`
  - `ml_training_product_day` (Gold training dataset used for modeling)

The intent is to keep downstream consumption stable even as ingestion logic evolves.

---

## Medallion Architecture as an Operational Contract

The Medallion architecture is implemented as an operational contract, not a cosmetic layering.

Each layer exists to absorb a specific class of failure without leaking instability downstream.

---

### Bronze Layer: System of Record
Bronze preserves source data with minimal intervention.

- Raw events ingested with full fidelity
- Operational metadata (`batch_id`, `ingestion_ts`) added for traceability
- No business rules or corrections applied

Bronze exists to support replay, auditability, and forensic debugging.
Correctness is intentionally deferred.

---

### Silver Layer: System of Truth
Silver is where data contracts are enforced.

- Schemas normalized and stabilized
- Deterministic deduplication applied using business keys and ingestion order
- Data quality rules explicit and repeatable

Silver is designed for idempotency and consistency.
Every downstream metric assumes Silver is correct by construction.

---

### Gold Layer: System of Insight
Gold exposes consumer-facing aggregates and analysis-grade datasets.

- Built exclusively from Silver data
- Shaped by access patterns, not source structure
- Optimized for analytical queries and semantic clarity

Gold is insulated from ingestion mechanics so analytics can evolve without destabilizing consumers.

---

## Storage and Transaction Semantics

Delta Lake underpins all layers.

- ACID guarantees for safe concurrent writes
- Schema enforcement to prevent silent drift
- Versioned storage for time travel and controlled backfills

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

This architecture is optimized for scale, replayability, and long-term maintainability, not short-lived experimentation.

---

## Analytics and Semantic Layer Design

The analytics layer is treated as a separate architectural concern from ingestion and transformation.

### Semantic Layer as a Contract
Gold views form a semantic contract between producers and consumers.

- Metric definitions centralized and versionable
- Dashboards depend on semantics, not raw events
- Changes to source data do not ripple into analytical consumers

This reduces cognitive load, simplifies governance, and makes analytical behavior predictable.

### Performance Strategy
Two classes of analytical assets are used:

- **Semantic views** for flexibility and correctness
- **Snapshot tables** for dashboard performance and stable latency

Snapshots are refreshed on a schedule and intentionally derived from semantic views to preserve consistency.

### Data Quality as Analytics
Data quality is surfaced alongside business metrics.

Null rates, invalid values, and completeness trends are exposed as Gold datasets, reinforcing that analytical insight includes confidence in the underlying data.

Principle:
Analytics is not just about insight.
It is about trust at scale.

---

## Performance Optimization
- Inspect query plans before changing storage to confirm pruning, pushdown, and shuffle behavior
- Partition Silver events by `event_date` and `event_type` to align with dominant analytical filters
- Avoid high-cardinality partition keys (`user_id`, `product_id`) to prevent small-file sprawl and skew
- Treat file compaction as mandatory; use OPTIMIZE/ZORDER when available, otherwise enforce controlled rewrites
- Benchmark representative workload queries to validate improvements and avoid placebo tuning
- Use caching only for short-lived, iterative analytics and explicitly uncache to keep results honest

Key takeaway:
Performance is shaped by data layout and access patterns, not last-mile query tweaks.

---

## Statistical Analysis and ML Feature Layer

Day 11 introduces a bridge between analytics and ML by formalizing two Gold assets:

- `gold.daily_kpis`: stable-grain dataset for statistical testing and trend analysis
- `gold.ml_features_purchase_intent`: reusable feature table for model training

Key design choices:
- Statistical tests operate on aggregated KPIs to ensure comparable samples
- Feature engineering is treated as a governed layer, not notebook logic
- Features combine time signals, user behavior history, user lifetime aggregates, and product popularity signals

Principle:
ML readiness comes from reproducible feature tables, not ad-hoc transformations.

---

## ML Experiment Architecture

ML experiments are treated as a governed workflow:

- Training datasets materialized as Gold tables to make inputs reproducible
- Evaluation uses time-based splits to reflect production behavior and reduce leakage
- MLflow tracks parameters, metrics, and artifacts to support comparison and iteration

Principle:
If experiments can’t be reproduced from versioned inputs, they’re not engineering artifacts.

---

## Model Comparison and Feature Engineering

### Flow
Bronze → Silver → Gold (`ml_training_product_day`)
→ Spark ML Pipelines
→ MLflow experiment tracking

### ML Layer
- VectorAssembler
- LR / RF / GBT regressors
- Train/Test split (80/20)
- Hyperparameter tuning

### Tracking
- MLflow (file-backed)
- Stored metrics, parameters, artifacts, signatures

Principles:
Reproducible. Explainable. Production-oriented.

---

## AI-Powered Analytics: Genie and Mosaic AI

### Flow
Gold analytics tables
→ Genie (NL → SQL)
→ Spark / SQL analytics
→ GenAI inference (NLP)
→ MLflow artifact and insight tracking

### Components
- Analytics layer: Gold views and KPI tables
- Query layer: SQL + Genie abstraction
- AI layer: NLP inference (sentiment)
- Insight layer: heuristic + AI-assisted summaries
- Tracking layer: MLflow (file-backed, Volume storage)

Design principles:
- AI augments analytics, not replaces it
- Interpretability over black-box generation
- Traceable insights via artifacts