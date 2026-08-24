---
title: "OLTP vs OLAP: Transactional vs Analytical Workloads"
parent: "Foundations: Bridging from Legacy DW & ETL"
nav_order: 3
---

# OLTP vs OLAP: Transactional vs Analytical Workloads
{: .no_toc }

*Part 1: Theory & Foundations &middot; Foundations: Bridging from Legacy DW & ETL*

Without this distinction clearly in mind, an architect will eventually approve running heavy analytical reports straight against the production order-taking database — and then get paged when Black Friday checkout traffic grinds to a halt because a dashboard is holding row locks. [Data Architecture Tenets & Styles](02-tenets-and-styles/) established that monolithic, distributed, and cloud are different bets about *where compute and storage live*; this topic is about a different, equally fundamental split — the shape of the *workload itself* — and it's the reason a warehouse or lake exists as a separate system in the first place.

## Two workloads, two sets of demands

**OLTP (Online Transactional Processing)** is the workload behind the systems that run a business moment to moment: placing an order, updating an inventory count, posting a payment. It reads and writes small numbers of rows, very frequently, from many concurrent users, and it needs the ACID guarantees from the first topic in this group to keep those operations correct under concurrency. A normalized relational schema suits it well, because normalization (the next topic) keeps each write small and unambiguous.

**OLAP (Online Analytical Processing)** is the workload behind the systems that make sense of the business in aggregate: monthly revenue by region, cohort retention, a year-over-year trend line. It reads huge numbers of rows — often the whole table — but writes rarely, usually in scheduled batches, and cares more about scan speed across billions of rows than about split-second write latency for any single one.

| | OLTP | OLAP |
|---|---|---|
| **Purpose** | Run the business (orders, payments, inventory) | Understand the business (trends, reporting, analysis) |
| **Query pattern** | Many small reads/writes, high concurrency | Few large scans/aggregations, lower concurrency |
| **Schema shape** | Normalized (minimizes write anomalies) | Denormalized / dimensional (minimizes joins at scan time) |
| **Latency target** | Milliseconds, per transaction | Seconds, per query, over huge data volumes |
| **Consistency need** | Strong (ACID) — a lost update is a real-money bug | Often relaxed — a report a few minutes stale is usually fine |
| **Typical engine** | PostgreSQL, MySQL, SQL Server, Oracle | Snowflake, Redshift, BigQuery, Synapse |

## Why the split forces a separate system

These two workloads actively fight each other on the same hardware: an OLAP-style full-table scan competes for the same I/O and locks that an OLTP system needs to stay fast for the next incoming order, and an OLTP-optimized normalized schema forces an OLAP query into dozens of joins it shouldn't need. That conflict — not fashion — is why a **data warehouse** (and later, a **data lake** or **lakehouse**) exists as a physically separate system from the operational database: it lets you denormalize, pre-aggregate, and scan at scale without ever touching the system a customer's checkout depends on.

{: .important }
> Running analytical queries directly against a production OLTP database isn't just "not best practice" — it's a direct violation of the scalability and reliability tenets from the previous topic, and at scale it becomes an outage waiting for a busy reporting day to trigger it.

## Where this shows up next

Every subsequent topic in this group assumes this split. The data-modeling refresher next distinguishes normalized OLTP design from the denormalized shapes OLAP favors; the ETL/ELT topic exists specifically to move data *from* OLTP systems *into* OLAP-friendly ones; and the batch/NRT/RT topic is really about how fast that movement needs to happen. Recognizing which workload you're looking at — a system of record versus a system of insight — is often the first and fastest diagnostic an architect runs on an unfamiliar design.

<!-- prevnext:start -->

---

| [&larr; Previous: Data Architecture Tenets & Styles: Monolithic, Distributed, Cloud](02-tenets-and-styles/) | [Next: Data Modeling, Database Types & Normalization Refresher &rarr;](04-data-modeling-refresher/) |
|:---|---:|

<!-- prevnext:end -->
