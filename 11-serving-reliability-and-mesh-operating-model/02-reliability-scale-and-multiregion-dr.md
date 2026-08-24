---
title: "Reliability: SLAs/SLOs, Observability, Multi-Region DR & Tenancy"
parent: Serving, Reliability & the Mesh Operating Model
nav_order: 2
---

# Reliability: SLAs/SLOs, Observability, Multi-Region DR & Tenancy
{: .no_toc }

*Part 6: Delivering Value & Staying Up &middot; Serving, Reliability & the Mesh Operating Model*

The moment a semantic layer's metrics feed a reverse-ETL sync into Salesforce or a data API a partner's checkout page calls at 2am, you've quietly made a promise that data will be there, on time, and correct — and an architect who hasn't turned that promise into a measurable, monitored commitment finds out how serious it was during the incident review, not before. [The Serving Layer: BI, Semantic Layer, Reverse ETL & Data APIs](01-serving-layer-bi-semantic-reverse-etl-apis/) showed how many different consumers now depend on your platform directly; this topic is about keeping every one of those promises when something breaks — because at this point in the platform's life, something eventually will.

## SLAs, SLOs & error budgets for data

Borrowed straight from site-reliability engineering, three terms let an architect turn a vague promise into something operational. An **SLA** (service level agreement) is the external or contractual commitment — "gold-layer tables are refreshed by 7am" — often with a business consequence attached if it's missed. An **SLO** (service level objective) is the internal target you actually engineer and monitor against, usually stricter than the SLA to leave margin: 99% of days refreshed by 6:45am, say. The **SLI** (service level indicator) is the measured signal — the actual timestamp of each day's refresh completion — that tells you whether the SLO was met. The gap between "met" and "100%" is your **error budget**: if the SLO is 99%, you have a budget of roughly 3.65 bad days a year to spend on deploys, experiments, and acceptable risk before you owe anyone an apology.

```mermaid
flowchart TD
    A[Business commitment: SLA\ne.g. "gold tables fresh by 7am"] --> B[Internal target: SLO\ne.g. 99% of days on time]
    B --> C[Measured signal: SLI\nactual refresh timestamp per day]
    C --> D{Within SLO?}
    D -- Yes --> E[Error budget intact\nship changes normally]
    D -- No --> F[Error budget burned]
    F --> G{Budget exhausted\nfor the period?}
    G -- No --> E
    G -- Yes --> H[Freeze risky changes,\nopen an incident]
```

The value of this chain isn't the vocabulary — it's that it gives a data team the same permission SRE teams have long had: an error budget that isn't exhausted is *permission to take risk* (ship a migration, try a new orchestration pattern), and one that is exhausted is a forcing function to stop and stabilize, without an argument about whose fault the last outage was.

## Observability & incident response: detect, diagnose, recover

You can't hold an SLO you can't see. Data **observability** extends the metrics/logs/traces idea from application monitoring to freshness, volume, schema, and distribution: is this table's row count within its normal range today, did a column's null rate spike, did an upstream schema change silently break a downstream join? The incident lifecycle mirrors any other production system — **detect** (an anomaly monitor or a failed test fires), **diagnose** (lineage tells you which upstream job or source actually caused it, not just which table looks wrong), and **recover** (rerun, backfill, or roll back). The lineage graph from your metadata catalog is what turns diagnosis from a Slack thread of guesses into a five-minute lookup — another reason the metadata layer covered earlier in this guide isn't a nice-to-have.

## Scaling patterns: vertical, horizontal & elastic

Reliability under load is also a scaling decision. **Vertical scaling** — a bigger single warehouse node or instance — is the fastest lever to pull and the one most familiar from a legacy on-prem database, but it has a ceiling and a blast radius: one node, one point of failure. **Horizontal scaling** spreads work across more nodes or shards, trading some coordination complexity for a much higher ceiling and better fault isolation. **Elastic scaling** — the cloud-native default in serverless warehouses — adds and removes capacity automatically in response to load, which is what makes a spiky BI workload and a quiet overnight batch job affordable on the same platform without either starving the other.

## Multi-region & disaster recovery: RPO/RTO as architecture

Scaling handles load; disaster recovery handles loss — a region outage, a corrupted table, a bad deploy that deletes data. Two numbers turn "we have a DR plan" into something you can actually design against: **RPO** (recovery point objective), the maximum data loss you can tolerate, measured in time since the last good backup or replica; and **RTO** (recovery time objective), the maximum downtime you can tolerate before service is restored. Every DR tier trades cost against both:

| DR tier | Typical RPO | Typical RTO | Relative cost |
|---|---|---|---|
| Backup & restore | Hours | Hours to a day | Lowest |
| Pilot light (minimal standby infra, scaled up on failover) | Minutes to an hour | Tens of minutes | Low-medium |
| Warm standby (scaled-down live replica) | Seconds to minutes | Minutes | Medium-high |
| Multi-region active-active | Near zero | Near zero (automatic failover) | Highest |

{: .important }
> An RPO/RTO number nobody has actually tested with a real failover drill is not a guarantee — it's a guess with a business consequence attached. The gap between a documented DR tier and a rehearsed one is exactly where architects get burned: the standby exists, the replication job exists, and it turns out nobody checked whether the restore actually completes within the promised window until the day it mattered.

## Tenancy: single vs multi-tenant

The last reliability lever is **tenancy**: how many customers or business units share the same physical infrastructure. Single-tenant architecture gives each customer (or, internally, each sensitive domain) its own isolated stack — strongest blast-radius containment and the easiest story for a strict compliance requirement, at the highest infrastructure cost. Multi-tenant architecture pools infrastructure across customers with logical isolation (row-level security, separate schemas, or separate catalogs within shared compute) — far more cost-efficient at scale, but a bug in the isolation logic now risks leaking one tenant's data into another's query results. Most data platforms serving external customers, whether through data APIs or embedded analytics, end up somewhere in between: pooled compute with hard isolation at the storage and access-control layer, reserving full single-tenant isolation for the handful of customers whose contracts demand it.

Reliability, at this scale, stops being a single team's monitoring dashboard and becomes an organizational question: who owns the SLO for each table, who gets paged, who decides the DR tier is worth the cost. That question is exactly where the next topic picks up.

<!-- prevnext:start -->

---

| [&larr; Previous: The Serving Layer: BI, Semantic Layer, Reverse ETL & Data APIs](01-serving-layer-bi-semantic-reverse-etl-apis/) | [Next: Data Products & the Data Mesh Operating Model &rarr;](03-data-products-and-mesh-operating-model/) |
|:---|---:|

<!-- prevnext:end -->
