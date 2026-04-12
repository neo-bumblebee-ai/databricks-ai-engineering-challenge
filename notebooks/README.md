# Notebooks index (Days 0–14)

This folder contains the execution trail for the 14-day challenge. Each notebook is designed to be runnable independently, but the recommended order is sequential (day00 → day14).

## Run order

1. **day00_setup_validation.ipynb**
   Workspace and dataset onboarding validation.

2. **day01_platform_basics.ipynb**
   Core Databricks + Spark basics.

3. **day02_ingestion_and_basic_transforms.ipynb**
   Typed ingestion, timestamp normalization, baseline transforms.

4. **day03_transformations_deep_dive.ipynb**
   Window functions, joins, feature-like transforms.

5. **day04_delta_lake_intro.ipynb**
   CSV → Delta, schema enforcement patterns.

6. **day05_delta_advanced.ipynb**
   MERGE/upsert patterns, time travel, idempotency concepts.

7. **day06_medallion_bronze_silver_gold.ipynb**
   Medallion pipeline as layered contracts.

8. **day07_bronze_layer.ipynb**
   Bronze ingestion pattern with audit metadata.

9. **day07_silver_layer.ipynb**
   Silver contract enforcement + deterministic deduplication.

10. **day07_gold_layer.ipynb**
    Gold aggregates for analytics and downstream consumption.

11. **day07_workflow_driver.ipynb**
    Orchestration driver (intended for Jobs / multi-task execution).

12. **day08_governance.sql.ipynb**
    Unity Catalog governance patterns.

13. **day09_analysis_datasets.ipynb**
    Analysis-grade datasets and metrics.

14. **day10_perfomance_optimization.ipynb**
    Layout, partitioning strategy, and performance checks.

15. **day11_statistical_analysis_ML_prep.ipynb**
    Stats and ML feature preparation.

16. **day12_AI_ML.ipynb**
    MLflow basics: tracking parameters, metrics, artifacts.

17. **day13_ml_model_comparison.ipynb**
    Comparing baseline models and feature variants.

18. **day14_ai_powered_analytics.ipynb**
    Genie / NL analytics integration patterns.
