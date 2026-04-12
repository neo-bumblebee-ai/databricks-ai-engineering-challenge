<p align="center">
  <img src="screenshots/banner.png" alt="Databricks 14 Days of AI Challenge" width="100%"/>
</p>

<div align="center">

# Databricks 14-Day AI Engineering Challenge

[![Databricks](https://img.shields.io/badge/Databricks-FF3621?style=for-the-badge&logo=databricks&logoColor=white)](https://databricks.com/)
[![Python](https://img.shields.io/badge/Python-3776AB?style=for-the-badge&logo=python&logoColor=white)](https://www.python.org/)
[![Delta Lake](https://img.shields.io/badge/Delta_Lake-00ADD8?style=for-the-badge)]()
[![PySpark](https://img.shields.io/badge/PySpark-E25A1C?style=for-the-badge&logo=apachespark&logoColor=white)]()
[![MLflow](https://img.shields.io/badge/MLflow-0194E2?style=for-the-badge&logo=mlflow&logoColor=white)]()
[![Status](https://img.shields.io/badge/Status-Completed-brightgreen?style=for-the-badge)]()

**A production-grade data and AI engineering implementation on Databricks — built in 14 days, engineered to last.**

</div>

---

## What This Repository Is

This is not a tutorial walkthrough. This is a structured engineering effort carried out over 14 days through the [Databricks AI Challenge](https://www.linkedin.com/company/indian-data-club/), organized by Indian Data Club (IDC) in collaboration with Databricks and Codebasics.

Each day has been treated as a production sprint — with real architectural decisions, documented trade-offs, and implementation patterns that translate directly to enterprise data platforms. The goal was not to complete exercises, but to validate how production-grade systems are designed, governed, and operated on the Lakehouse.

If you are evaluating this repository as part of a technical review, I would recommend starting with [`docs/architecture.md`](docs/architecture.md) and [`docs/decisions.md`](docs/decisions.md). The notebooks are the execution trail.

---

## Architecture at a Glance

```
Raw Events (Oct–Nov 2019)
        │
        ▼
  ┌─────────────┐
  │   Bronze    │  ← Raw ingestion + audit metadata (batch_id, ingestion_ts)
  └──────┬──────┘
         │
         ▼
  ┌─────────────┐
  │   Silver    │  ← Schema enforcement, deterministic deduplication, quality gates
  └──────┬──────┘
         │
         ▼
  ┌─────────────┐
  │    Gold     │  ← Consumer-facing aggregates, ML features, analytical datasets
  └──────┬──────┘
         │
    ┌────┴─────┐
    │          │
  MLflow    Genie / Mosaic AI
(Experiments) (NL → SQL, Sentiment, Insights)
```

Each layer is an operational contract — not a cosmetic label. Failures are isolated by design so that ingestion instability never reaches analytical consumers.

---

## Key Engineering Capabilities Demonstrated

| Area | What Was Built |
|---|---|
| **Lakehouse Architecture** | Medallion pipeline (Bronze / Silver / Gold) as layered operational contracts |
| **Delta Lake** | ACID transactions, schema enforcement, MERGE upserts, time travel, idempotent ingestion |
| **Unity Catalog Governance** | Volume-backed storage, external Delta tables, catalog/schema ownership boundaries |
| **Distributed Processing** | PySpark window functions, joins, UDFs, distributed feature engineering |
| **ML Engineering** | Scikit-Learn + Spark ML pipelines, time-based train/test splits, hyperparameter tuning |
| **Experiment Tracking** | MLflow — parameters, metrics, model signatures, artifact logging |
| **AI-Powered Analytics** | Genie (NL → SQL), sentiment inference via Mosaic AI, KPI-driven narrative generation |
| **Pipeline Orchestration** | Databricks Jobs with parameterized notebooks, task dependencies, failure isolation |

---

## Dataset

**Source:** [eCommerce Events History — Cosmetics Shop (Kaggle)](https://www.kaggle.com/datasets/mkechinov/ecommerce-events-history-in-cosmetics-shop)

- ~28M user behavior events across October and November 2019
- Event types: `view`, `cart`, `purchase`
- Fields: `event_time`, `event_type`, `product_id`, `category_id`, `category_code`, `brand`, `price`, `user_id`, `user_session`
- Ingested via Databricks Unity Catalog Volumes and processed through the full medallion pipeline

---

## Prerequisites

| Requirement | Notes |
|---|---|
| Databricks Workspace | Community Edition or higher |
| Unity Catalog | Required for Volumes and governed table registration |
| Dataset | Upload Kaggle CSV files to a Unity Catalog Volume at your configured path |
| Python 3.x + PySpark | Provided natively by the Databricks runtime |
| MLflow | Included in Databricks runtime; file-based tracking configured for Community Edition |

> **Note for Community Edition users:** Certain platform features — such as `OPTIMIZE`, `ZORDER`, and managed Unity Catalog — may be restricted. All notebooks handle these constraints explicitly with graceful fallbacks.

---

## Repository Structure

```
├── notebooks/          # Day-wise implementation notebooks (day00 → day14)
├── docs/
│   ├── architecture.md # Platform design, layer contracts, and design principles
│   ├── data_model.md   # Dataset schema and field definitions
│   └── decisions.md    # Daily engineering decisions log (the why, not the what)
├── screenshots/        # Proof artifacts and challenge outputs (day-wise)
└── exports/            # Exported query results and analysis outputs
```

---

## How to Run

```
1. Upload dataset CSVs to a Unity Catalog Volume in your Databricks workspace
2. Open notebooks/day00_setup_validation.ipynb — validate environment and data access
3. Run notebooks sequentially as per notebooks/README.md
4. Bronze → Silver → Gold Delta tables are materialised progressively
5. MLflow experiments begin at Day 12, built on Gold feature tables from Day 11
```

Full notebook index with descriptions: [`notebooks/README.md`](notebooks/README.md)

---

## Day-by-Day Progress

| Day | Focus Area | Key Outcome |
|---|---|---|
| 0 | Environment setup | Workspace validated, dataset onboarded to Volumes |
| 1 | Platform basics | Core Databricks + PySpark patterns established |
| 2 | Ingestion & transforms | Schema normalised, Oct + Nov unioned, quality baseline set |
| 3 | Transformations deep dive | Window functions, joins, UDFs, derived features |
| 4 | Delta Lake introduction | CSV → Delta, schema enforcement verified |
| 5 | Delta Lake advanced | MERGE upserts, time travel, idempotent ingestion |
| 6 | Medallion architecture | Bronze/Silver/Gold pipeline with audit metadata and deduplication |
| 7 | Pipeline operationalisation | Databricks Jobs, parameterised notebooks, task dependencies |
| 8 | Unity Catalog governance | Volume-aligned catalog registration, external table design |
| 9 | Analytical datasets | Semantic layer, snapshot tables, data quality as first-class output |
| 10 | Performance optimisation | Query plan analysis, partitioning strategy, compaction |
| 11 | Statistical analysis + ML prep | Hypothesis testing, Gold feature table (`ml_features_purchase_intent`) |
| 12 | MLflow basics | Reproducible experiments, time-based splits, baseline model |
| 13 | Model comparison | Spark ML pipelines, LR/RF/GBT comparison, hyperparameter tuning |
| 14 | AI-powered analytics | Genie (NL→SQL), sentiment pipeline, KPI-driven narrative generation |

---

## Tech Stack

| Layer | Technologies |
|---|---|
| Platform | Databricks (Community Edition / Professional) |
| Processing | PySpark, Spark SQL |
| Storage | Delta Lake, Unity Catalog Volumes |
| Languages | Python 3.x, SQL |
| ML | Scikit-Learn, Spark ML (VectorAssembler, LR, RF, GBT) |
| Experiment Tracking | MLflow (file-backed) |
| AI / Analytics | Databricks Genie, Mosaic AI |
| Visualisation | Pandas, Matplotlib |

---

## Further Reading

- [Architecture & Design Principles](docs/architecture.md)
- [Engineering Decisions Log](docs/decisions.md)
- [Notebook Index](notebooks/README.md)
- [Screenshots & Proof Artifacts](screenshots/README.md)

---

## Acknowledgements

Sincere thanks to [Indian Data Club (IDC)](https://www.linkedin.com/company/indian-data-club/), Databricks, and Codebasics for designing a challenge format that demands implementation over passive learning.

The format worked because it created accountability — day by day, decision by decision — which is exactly how production engineering should be done.
