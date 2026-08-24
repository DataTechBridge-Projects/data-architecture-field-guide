---
title: "Beyond Star Schemas: One Big Table & Wide Tables"
parent: Dimensional Modeling for the Cloud Era
nav_order: 4
---

# Beyond Star Schemas: One Big Table & Wide Tables
{: .no_toc }

*Part 3: Designing the Data Layer &middot; Dimensional Modeling for the Cloud Era*

An architect who only knows how to build star schemas will over-engineer half the marts they're asked for — plenty of cloud-era BI and reporting workloads are better served by skipping the joins entirely, and knowing when is the difference between a dashboard that returns in two seconds and one that returns in twenty. [Slowly Changing Dimensions & Conformed Dimensions Across the Enterprise](03-scd-and-conformed-dimensions/) assumed you were keeping the star intact and just managing change within it; this topic asks a more basic question — whether a star is even the right target once storage is nearly free and every dimension join has to be paid for in compute at query time.

## One Big Table: the cloud-warehouse default for a reason

A **one big table (OBT)** design pre-joins a fact table with all (or most) of its **dimension** tables into a single wide, denormalized table, resolved once at write time instead of on every read. Instead of a BI tool issuing `fact_sales JOIN dim_customer JOIN dim_product JOIN dim_date` on every query, it selects from one table that already has `customer_segment`, `product_category`, and `fiscal_quarter` sitting as plain columns next to the measures.

This inverts the instinct a legacy-warehouse background trained into you: normalize to avoid repeating `customer_segment` on every row. On a columnar cloud warehouse, that repetition is nearly free — columnar storage compresses a low-cardinality column like `segment` extremely well, and cloud storage costs cents per gigabyte per month. What isn't free is compute: every `JOIN` a query performs is bytes scanned and shuffled, billed per query on-demand or drawn down from a fixed-size warehouse's concurrency budget. OBT trades storage you're barely paying for against join compute you're paying for on every single query — which is why it's become the default shape for BI extracts, semantic-layer sources, and dashboards that get hit thousands of times a day.

## Wide tables and the economics of denormalization

A **wide table** generalizes the same idea beyond a single fact/dimension bundle: flatten an entire domain's related entities — order, order lines, customer, product, promotion — into one table with dozens or hundreds of columns, often built specifically to feed one high-traffic dashboard or one downstream tool (a semantic layer, a reverse-ETL sync, an ML feature pipeline) that can't or shouldn't do its own joins.

The trade-off is real, not free: a wide table duplicates data heavily, so a single upstream correction (a mislabeled product category) now has to be rewritten across every row that denormalized it, instead of updated once in a dimension table. Wide tables also erode the governance benefit of **conformed dimensions** — if five different wide tables each freeze in their own copy of "customer segment" at build time, they can drift out of sync with the dimension of record unless the pipeline that builds them is rerun on every dimension change. Treat OBT and wide tables as a *derived, rebuildable* layer sitting downstream of a conformed dimensional model — never as the system of record itself.

## Choosing your modeling paradigm

| Criterion | Star Schema | One Big Table (OBT) | Wide Table |
|---|---|---|---|
| Query simplicity for BI tools | Requires joins; tool must know the model | Excellent — single `SELECT`, no joins | Excellent — single `SELECT`, no joins |
| Storage cost | Lowest (no duplication) | Moderate (dimension attributes repeated per fact row) | Highest (many entities flattened together) |
| Compute cost per query | Higher — join cost paid on every query | Lower — join cost paid once at build time | Lowest for the target consumer, but wider scans |
| Update / maintenance cost | Low — change a dimension once | Moderate — rebuild on dimension change | High — rebuild is larger and more frequent |
| Governance / conformance | Strong — single conformed dimension enforced by design | Weaker — must be rebuilt from conformed source to stay correct | Weakest — easy to drift if not pipeline-managed |
| Best fit | Ad-hoc analysis, many fact tables sharing dimensions, evolving schemas | High-traffic BI dashboards, semantic-layer sources | Single high-value consumer (ML feature set, reverse-ETL sync, one dashboard) |

{: .important }
> OBT and wide tables are a *serving-layer* optimization, not a replacement for dimensional modeling underneath. Build and conform your **fact**/**dimension** model first — it's what keeps the wide table honest — then materialize OBT or wide-table views from it for the specific workloads that need join-free reads. Skipping straight to wide tables without a conformed model underneath just relocates the "five versions of customer segment" problem instead of solving it.

Picking a paradigm per workload, rather than a single model for the whole warehouse, is the practical synthesis of everything this group has covered: a design philosophy sets the enterprise-level structure, facts/dimensions/grain make that structure precise, SCD and conformance keep it trustworthy over time, and OBT/wide tables let you spend storage — which is cheap — to buy back the query compute that isn't.

<!-- prevnext:start -->

---

| [&larr; Previous: Slowly Changing Dimensions & Conformed Dimensions Across the Enterprise](03-scd-and-conformed-dimensions/) | [Next: Ingestion & Streaming Decisions &rarr;](../06-ingestion-and-streaming-decisions/) |
|:---|---:|

<!-- prevnext:end -->
