# Engineering Decisions Log

This file captures small decisions made daily that improve repeatability, clarity, and scale.

## Day 0
- Preferred Volumes (Unity Catalog) for dataset storage instead of ad-hoc uploads, to keep paths consistent and permissions explicit.

## Day 1
- Keep transformations modular and readable: load → validate → transform → aggregate.
- Always validate row count and schema before any transformation work.
