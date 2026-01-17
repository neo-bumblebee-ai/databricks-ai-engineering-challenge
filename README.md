<p align="center">
  <img src="assets/banner.png" alt="Databricks 14 Days of AI Challenge" width="100%"/>
</p>

<div align="center">

# Databricks 14 Days of AI Challenge

[![Databricks](https://img.shields.io/badge/Databricks-FF3621?style=for-the-badge&logo=databricks&logoColor=white)](https://databricks.com/)
[![Python](https://img.shields.io/badge/Python-3776AB?style=for-the-badge&logo=python&logoColor=white)](https://www.python.org/)
[![Delta Lake](https://img.shields.io/badge/Delta_Lake-00ADD8?style=for-the-badge)]()
[![Status](https://img.shields.io/badge/Status-In_Progress-yellow?style=for-the-badge)]()

</div>

---

## Overview

This repository documents my hands-on work through the **Databricks 14 Days of AI Challenge**, used intentionally as a structured environment to validate modern data engineering and AI architecture patterns.
This is an initiative organized by the [Indian Data Club (IDC)](https://www.linkedin.com/company/indian-data-club/) and sponsored by [Databricks](https://databricks.com/) and [Codebasics](https://codebasics.io/)

With nearly two decades of experience designing and operating enterprise-scale data platforms, this effort is not about learning tools in isolation. The objective is to evaluate Databricks as a unified platform under realistic conditions: governance constraints, storage semantics, catalog behavior, and operational trade-offs that surface in real production environments.

The challenge provides a fixed scope and timeline. Within that boundary, the focus here is on building solutions that are correct, governable, and evolvable rather than optimized for quick demos.

## Progress Log
- Day 0: Environment setup and dataset onboarding
- Day 1: Platform basics and initial PySpark exploration
- Day 2: Ingestion and basic transformations (schema normalization, Oct + Nov union, minimal data quality checks)
- Day 3: PySpark transformations deep dive (joins, window functions, derived features, UDFs)
- Day 4: Delta Lake introduction (CSV to Delta, tables, schema enforcement)
- Day 5: Delta Lake advanced (MERGE upserts, time travel, idempotent ingestion, optimization attempts)
- Day 6: Medallion architecture with Delta Lake (Bronze/Silver/Gold, audit metadata, deterministic deduplication)
- Day 7: Operationalized the pipeline using Databricks Jobs with parameterized notebooks, task dependencies, and failure isolation.
- Day 8: Validated Unity Catalog concepts by aligning Volumes, Delta storage, and catalog registration with controlled governance boundaries.