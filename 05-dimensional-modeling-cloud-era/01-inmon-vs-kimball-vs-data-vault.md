---
title: "Warehouse Design Philosophies: Inmon vs Kimball vs Data Vault"
parent: Dimensional Modeling for the Cloud Era
nav_order: 1
---

# Warehouse Design Philosophies: Inmon vs Kimball vs Data Vault
{: .no_toc }

*Part 3: Designing the Data Layer &middot; Dimensional Modeling for the Cloud Era*

Picking **Inmon**, **Kimball**, or **Data Vault** as your warehouse's design philosophy isn't a stylistic preference you can defer — it decides whether onboarding a new subject area takes your team a sprint or a quarter, and whether an auditor can trace a number back to its source system five years from now. Get it wrong and you don't find out until the mismatch is expensive to unwind: a Kimball shop bolting on audit trails after the regulator asks, or an Inmon shop watching BI requests queue up behind a normalization layer nobody outside the data team can query. [Table Formats: Delta vs Iceberg vs Hudi](../04-storage-and-table-formats/02-table-formats-delta-iceberg-hudi/) settled the *where* — which open table format sits under your data and how it gets ACID guarantees on object storage. This topic is the *how it's shaped* question sitting directly on top of that choice: the same Iceberg or Delta table can hold a normalized Inmon-style entity, a Kimball **fact** table, or a Data Vault satellite, and the table format itself has no opinion on which one you pick.

## The question each school actually answers

All three philosophies are answers to the same underlying question — asked from different starting points: *what does the warehouse optimize for first, structure or speed-to-insight or auditability?*

**Inmon (the Corporate Information Factory)** answers "structure first." Bill Inmon's model builds a single, heavily normalized (typically 3NF) enterprise data warehouse as the integrated system of record, with subject-oriented, non-volatile, time-variant data pulled in from every source system. Departmental **dimension**al data marts are spun off *from* that normalized core afterward, on demand. It's top-down: model the enterprise once, correctly, then let consumption-layer marts follow.

**Kimball (the dimensional bus architecture)** answers "speed-to-insight first." Ralph Kimball's model skips the giant normalized core and builds directly for the business questions users ask, using **star** (and occasionally **snowflake**) schemas — a **fact** table of measurable events surrounded by **dimension** tables of descriptive context. Instead of one big normalized model, Kimball ties independently-built marts together with **conformed dimensions**: shared dimensions (a single `dim_customer`, a single `dim_date`) reused across every fact table so metrics stay comparable enterprise-wide. It's bottom-up: model one business process, ship it, repeat.

**Data Vault** answers "auditability and change tolerance first." It splits every entity into three table types — **hubs** (business keys), **links** (relationships/transactions between hubs), and **satellites** (descriptive attributes with full history, insert-only, never updated in place). Nothing is ever overwritten or reshaped when a source system changes; you just add satellites. That makes Data Vault the most resilient to upstream schema churn and the easiest to prove to an auditor, but it's rarely queried directly — most Data Vault shops build a Kimball-style **information mart** on top of the raw vault for BI consumption, making it less a competitor to Kimball than a staging philosophy that feeds one.

## Comparison

| | Inmon (CIF) | Kimball (Dimensional Bus) | Data Vault |
|---|---|---|---|
| Primary structure | Normalized (3NF) enterprise core | Star/snowflake **fact**/**dimension** marts | Hubs, links, satellites (insert-only) |
| Build direction | Top-down: enterprise model first, marts derived | Bottom-up: one business process at a time | Bottom-up: one source system at a time |
| Time to first delivery | Slow — value arrives after the core is modeled | Fast — a mart ships in weeks | Fast to load, slow to expose (needs a mart on top) |
| Auditability / history | Moderate | Handled per-dimension via **SCD** types | Excellent — nothing is overwritten, full history by design |
| Resilience to source schema change | Moderate — core model absorbs some change | Fragile if grain/conformance wasn't planned | Very high — new satellites, no rework |
| Query ergonomics for BI tools | Poor without a mart layer | Excellent — this is what star schemas are for | Poor — requires a derived mart |
| Best fit | Large enterprise integrating many systems that needs one governed version of truth | Teams optimizing for fast, trustworthy BI delivery | Regulated industries with frequent source change and audit requirements |

## Which one your constraints actually point to

In practice, few cloud-era shops adopt any of these purely. The decision usually comes down to three questions: How often do source systems change shape under you? How fast does the business need the first dashboard? And who has to sign off on data lineage — a BI stakeholder, or a compliance officer?

- Small team, BI-first, need value in weeks → **Kimball**, straight to star schemas.
- Regulated industry (finance, healthcare, insurance) with frequent upstream changes and audit obligations → **Data Vault** for the raw/history layer, with a Kimball mart layered on top for consumption.
- Large enterprise consolidating dozens of source systems into one governed, subject-oriented model before anyone builds a mart → **Inmon**.

{: .important }
> The most common cloud-era pattern isn't "pick one" — it's Data Vault (or a lighter insert-only raw layer) for ingestion and history, feeding Kimball-style star schemas for the semantic/BI layer. Treat Inmon vs Kimball vs Data Vault as answering *different* layers of the same warehouse, not a single either/or choice for the whole platform.

The next topic assumes you've landed on some flavor of dimensional mart — whether it's your only layer (pure Kimball) or the layer sitting on top of a vault or normalized core — and goes into the mechanics that make that mart trustworthy: facts, dimensions, and the grain decision that has to come before either.

<!-- prevnext:start -->

---

| [&larr; Previous: Dimensional Modeling for the Cloud Era](./) | [Next: Facts, Dimensions & Grain: The Foundation of Dimensional Modeling &rarr;](02-facts-dimensions-grain/) |
|:---|---:|

<!-- prevnext:end -->
