---
title: "Table Formats: Delta vs Iceberg vs Hudi"
parent: Storage & Table Formats
nav_order: 2
---

# Table Formats: Delta vs Iceberg vs Hudi
{: .no_toc }

*Part 3: Designing the Data Layer &middot; Storage & Table Formats*

[Storage Foundations](01-storage-foundations/) covered how bytes sit in object storage and what format they're written in — but a directory full of Parquet files has no concept of a transaction. Two concurrent jobs can each list a partition, decide what to write, and land conflicting files with no referee; a job that crashes halfway through a write leaves partial files with nothing to signal "this write never completed." An architect who treats a lake as just files-in-a-bucket will eventually ship a pipeline that silently corrupts or duplicates data under concurrent writes, and won't have a log to explain what happened. **Table formats** exist to close that gap: they add a transactional metadata layer on top of the same object storage and file formats from the previous topic, so a folder of Parquet files starts behaving like an actual table.

## Why Table Formats Exist: ACID on Top of Object Storage

Early Hive-style "tables" on a data lake were really just a naming convention — a directory path mapped to a table name, with no atomic way to add, replace, or delete files across that directory as a unit. Adding rows meant writing new files and hoping no one else was reading the directory mid-write; updating or deleting rows usually meant rewriting entire partitions. **Delta Lake**, **Apache Iceberg**, and **Apache Hudi** all solve the same underlying problem — bringing **ACID** guarantees (the same ones from [ACID, BASE & the CAP Theorem](../01-foundations/01-acid-base-and-cap-theorem/)) to a storage layer that was never built with transactions in mind — but they do it with different metadata designs:

- **Delta Lake** (originated at Databricks) keeps an ordered transaction log (`_delta_log`) of JSON commits plus periodic Parquet checkpoints; every writer reads the log to know the table's current valid state before committing.
- **Apache Iceberg** (originated at Netflix, now an Apache project) tracks a tree of **snapshots** built from manifest files listing which data files belong to the table at each point in time, with every column tracked by a stable ID rather than a position.
- **Apache Hudi** (originated at Uber) organizes changes along a **timeline**, with two table layouts — Copy-on-Write (rewrite affected files immediately, optimized for read speed) and Merge-on-Read (log deltas separately, optimized for write speed, compacted later).

All three give you atomic commits, concurrent-writer isolation, row-level **upserts** and deletes, and a durable log of every change — capabilities a Hive-style table directory simply cannot offer.

## Delta vs Iceberg vs Hudi: The Decision Matrix

| | **Delta Lake** | **Apache Iceberg** | **Apache Hudi** |
|---|---|---|---|
| **Origin / governance** | Databricks; open-sourced, Linux Foundation | Netflix; Apache Software Foundation | Uber; Apache Software Foundation |
| **Metadata model** | Transaction log (JSON + Parquet checkpoints) | Snapshot tree of manifest files | Timeline of instants (Copy-on-Write / Merge-on-Read) |
| **Strongest native engine** | Spark / Databricks (deepest integration) | Engine-neutral by design | Spark and Flink (streaming-first) |
| **Multi-engine support** | Growing (Trino, Presto, Athena; UniForm bridges to Iceberg) | Broadest — Spark, Flink, Trino, Presto, Snowflake, BigQuery, Athena | Good, narrower than Iceberg |
| **Partition evolution** | Limited (liquid clustering as newer alternative) | Native — change partitioning without rewriting data | Supported, less central to the design |
| **Upsert / CDC fit** | Strong `MERGE INTO` support | Solid, engine-dependent performance | Purpose-built — indexed upserts at scale, its core use case |
| **Typical fit** | Databricks-centric lakehouses | Open, multi-engine lakehouses; portability a priority | High-frequency upsert/CDC ingestion pipelines |

{: .important }
> The table format you choose is harder to reverse than the file format underneath it, because every downstream consumer — every engine, every catalog integration, every streaming reader — ends up coupled to its transaction semantics, not just its files. Treat this as closer to a **one-way door** than the Parquet-vs-ORC choice one topic back, and weigh your team's dominant engine and catalog ecosystem at least as heavily as the feature checklist above.

Run the matrix against a concrete case and the decision usually falls out quickly. Take a retailer replacing a nightly Informatica load into an on-prem warehouse with a cloud lakehouse: point-of-sale data arrives as a continuous stream of row-level changes (inserts, price corrections, refunds) from hundreds of stores, and the team's processing engine is already Flink for the streaming leg and Spark for batch backfills. That combination — high-frequency upserts, a streaming-first engine, multi-engine batch access for BI afterward — points toward Hudi's Merge-on-Read layout for the raw ingest tier, with a periodic compaction job keeping read performance in check; a shop standardized entirely on Databricks with less upsert pressure would reasonably land on Delta instead, and a platform team prioritizing engine portability above all else would lean Iceberg. The matrix doesn't hand you an answer in isolation — it narrows the field once you know your actual write pattern and engine footprint.

## Schema Evolution & Time Travel as Architecture, Not Features

**Schema evolution** — adding, renaming, or widening a column without rewriting historical data — isn't a convenience feature; it's what lets an upstream producer change its data shape without an immediate, coordinated rewrite of every downstream consumer, which is the same producer/consumer tension the data-contracts topic later in this course addresses at the process level. **Time travel** — querying a table as it existed at a prior snapshot or commit — is equally architectural: it gives you reproducible inputs for a machine-learning training run, an audit trail for "what did finance see when they closed the books," and a rollback mechanism when a bad batch job overwrites good data, all without a separate backup system.

## The Open Catalog & Interop: The Open-Lakehouse Promise (and Its Limits)

The pitch behind all three formats is the **open lakehouse**: one physical copy of data, in open file and table formats, queryable by whichever engine a given team already uses — Spark for engineering, Snowflake or Trino for analysts, Flink for streaming — instead of copying data into each tool's proprietary storage. A **catalog** (AWS Glue Data Catalog, Hive Metastore, Unity Catalog, or an Iceberg REST catalog) is what makes this real: it's the shared directory of "which tables exist, and where's their current metadata" that every engine consults.

The promise has real limits, though. Catalog choice can reintroduce the lock-in the open file formats were meant to avoid — a Unity-Catalog-only deployment isn't meaningfully more portable than a proprietary warehouse, even though the underlying files are Parquet. Cross-format interop still often means a conversion or a compatibility shim (Delta's UniForm exposing Iceberg-compatible metadata, for instance) rather than one engine natively reading another format's native metadata everywhere. And feature parity isn't universal — a capability available on one engine's Iceberg support may lag on another's. The practical takeaway: pick a table format alongside your catalog and dominant engines as one joint decision, not in isolation, using the cost/control/complexity/lock-in framework from the [Mental Model for Architecture Choices](../03-architects-decision-framework/04-mental-model-for-architecture-choices/) topic that opened this Part.

<!-- prevnext:start -->

---

| [&larr; Previous: Storage Foundations: Object Storage, File Formats & Access Patterns](01-storage-foundations/) | [Next: Consistency in Practice: Read-After-Write on Object Storage &rarr;](03-consistency-models/) |
|:---|---:|

<!-- prevnext:end -->
