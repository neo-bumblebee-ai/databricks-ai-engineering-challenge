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

