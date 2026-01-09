# Architecture Notes

## Current approach
- Databricks Community Edition for compute and notebooks
- Dataset stored in a controlled workspace location (Volumes or DBFS based on availability)
- PySpark notebooks as the primary artifact, exported to this repo daily

## Target evolution (later in the challenge)
- Move from exploratory reads to a layered design:
  - Bronze: raw ingest
  - Silver: cleaned and conformed
  - Gold: aggregated outputs for analytics and downstream use
