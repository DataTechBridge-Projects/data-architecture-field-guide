---
title: "The dbt Paradigm: Transformation as Code, Data Contracts & Idempotent Reprocessing"
parent: Transformation & the Modern Data Stack
nav_order: 2
---

# The dbt Paradigm: Transformation as Code, Data Contracts & Idempotent Reprocessing
{: .no_toc }

*Part 4: Moving & Shaping Data &middot; Transformation & the Modern Data Stack*

A dbt project without contracts or a reprocessing discipline turns "just rerun the failed job" into a coin flip — sometimes it repairs the table, sometimes it silently doubles every row that already loaded successfully. This is the paradigm that lets an architect actually trust reprocessing enough to design recovery procedures around it, instead of treating every failed run as an incident that needs a human to manually diff tables before anyone touches "rerun." The [previous topic](01-etl-vs-elt-and-medallion-in-practice/) settled *where* transformation logic runs (ELT, pushed into the warehouse's own compute) and *how* it's organized (bronze, silver, gold); this topic is about how those bronze-to-silver-to-gold transforms actually get written, enforced, and safely rerun without corrupting what's already landed.

## The dbt paradigm: transformation as code

In the Informatica or Talend world, a transformation lived inside a proprietary repository, edited through a GUI mapping canvas, and deployed by an admin with access to that tool. Two engineers couldn't easily diff two versions of the same mapping, and "code review" meant screen-sharing. dbt (data build tool) replaces that with **transformation-as-code**: every bronze-to-silver or silver-to-gold step is a `.sql` file — a *model* — checked into the same git repository as everything else you own. That single change unlocks the rest of the modern software toolchain for something that used to be GUI-locked: pull requests, diffs, code review, and a **CI/CD** pipeline that compiles and tests every model before it's allowed to merge.

Two side effects matter as much as the code-as-files idea itself. First, models reference each other with a `ref()` function instead of a hardcoded table name, so dbt can compile the full dependency graph automatically — this is where your transformation **lineage** comes from, and it's often the backbone a **data catalog** ingests rather than something engineers maintain by hand. Second, because the transform logic is just SQL text, the orchestration layer (the next topic's subject) only needs to know "run dbt," not the internals of every individual mapping — the DAG of models lives inside the project, not inside a separate orchestration tool's configuration.

## Data contracts as code

A **data contract** is an explicit, enforced agreement about a table's shape — its column names, types, nullability, and key uniqueness — between the team that produces it and every team or system that consumes it downstream. In a classic warehouse, a producing team could rename or drop a column in a view and the first anyone heard about it was a broken downstream report, days later. dbt lets you make that agreement machine-checked instead of tribal knowledge: a model's YAML file declares `contract: enforced: true` plus column-level types and constraints, and dbt refuses to build the model at all if the compiled SQL doesn't match the declared shape. Schema tests (`unique`, `not_null`, `relationships`, `accepted_values`) extend the same idea to data *values*, not just structure, and run as part of the same CI check.

The architectural decision isn't "use contracts everywhere" — it's *where* to spend that rigor:

| Layer | Typical contract posture | Why |
|---|---|---|
| Bronze | None | Raw and append-only by design; enforcing shape here defeats the point of an unmodified audit trail |
| Silver | Selective | Enforce on the join keys and types that many downstream silver/gold models depend on |
| Gold | Enforced | This is what BI tools, finance, and ML features consume directly — a silent shape change here is the expensive kind of incident |

A broken contract failing your CI pipeline at merge time is a cheap, boring event. The same break reaching a gold table un-caught is the kind a stakeholder discovers in a dashboard, and then asks you to explain.

## Idempotency and reprocessing in transforms

**Idempotency** means running the same transform twice — or twenty times, after twenty retries — on the same input produces exactly the same result as running it once: no duplicate rows, no double-counted aggregate. It sounds like a property you'd get for free from "just SQL," but incremental models make it easy to lose. An incremental model that only ever `insert`s new rows since the last run will, on a retried or replayed batch, insert that batch's rows a second time — the job reports success, the row count just quietly grows.

dbt gives you two idempotent-by-construction shapes to choose from instead. A `full-refresh` model truncates and rebuilds from source every run, so reprocessing is trivially idempotent — it just costs full-table compute every time. An incremental model with a declared `unique_key` and a `merge` (upsert) strategy is idempotent for cheap: a retried batch overwrites the same keys instead of appending beside them.

{% raw %}
```sql
-- models/gold/fct_orders.sql — illustrative dbt model, not a runnable script
{{ config(
    materialized='incremental',
    unique_key='order_id',
    incremental_strategy='merge'
) }}

select
    order_id,
    customer_id,
    order_amount,
    order_ts
from {{ ref('silver_orders') }}
{% if is_incremental() %}
where order_ts > (select max(order_ts) from {{ this }})
{% endif %}
```
{% endraw %}

```yaml
# models/gold/fct_orders.yml — illustrative dbt contract definition, not a runnable file
models:
  - name: fct_orders
    config:
      contract:
        enforced: true
    columns:
      - name: order_id
        data_type: string
        constraints:
          - type: not_null
          - type: unique
      - name: order_amount
        data_type: numeric
        constraints:
          - type: not_null
```

The `merge` strategy keyed on `order_id` is what makes a retry of this model safe: a batch that partially loaded before failing gets overwritten on rerun, not appended a second time next to itself.

An incremental model that only appends will double-count every retried batch without ever throwing an error — the job succeeds, the numbers are just wrong, and nobody notices until finance asks why revenue jumped 9% overnight with no corresponding order volume to explain it. Idempotent design, not optimism, is what makes "just rerun it" a safe instruction instead of the start of an incident.
{: .important}

Put the three pieces together and you have the actual operating discipline behind ELT and medallion: transformation-as-code makes the bronze-to-gold logic reviewable and testable, contracts make each layer's shape a checked guarantee instead of an assumption, and idempotency makes reprocessing — the thing that *will* happen after every pipeline failure — a routine operation instead of a forensic one. Orchestrating when those idempotent, contract-checked models actually run is where the next topic picks up.

<!-- prevnext:start -->

---

| [&larr; Previous: ETL vs ELT & the Medallion Pattern in Practice](01-etl-vs-elt-and-medallion-in-practice/) | [Next: DataOps, Orchestration & Metadata &rarr;](../08-dataops-orchestration-and-metadata/) |
|:---|---:|

<!-- prevnext:end -->
