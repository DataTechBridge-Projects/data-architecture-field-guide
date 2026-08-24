---
title: "Cost as an Architectural Decision: Storage, Compute & Egress Economics"
parent: Cost & Performance Architecture
nav_order: 1
---

# Cost as an Architectural Decision: Storage, Compute & Egress Economics
{: .no_toc }

*Part 5: Running It Like a Platform &middot; Cost & Performance Architecture*

An architect can design a platform with airtight access control, clean lineage, and a well-enforced data contract, hand it over, and still get called into a budget review three months later to explain why the monthly cloud bill is four times what finance approved — because nobody treated cost as a design constraint on the same footing as consistency or latency. [Security & Governance](../09-quality-security-and-governance/03-security-and-governance/) settled who can touch the data and on what legal basis; trust and access are solved problems by the time you get here. This topic asks the question that governance doesn't answer at all: is the platform affordable to run at the scale you're about to put it at? A locked-down, well-governed platform that quietly bankrupts its own budget is still a failed architecture, and the failure is invisible until the invoice arrives.

## Cost Is a Design Decision, Not a Finance Afterthought

In the on-prem warehouse world you came from, cost was mostly a sunk, fixed decision: the company bought a Teradata appliance or a SQL Server cluster, and whatever you built ran on hardware that was already paid for. Query a fact table sloppily and the only price was a slower dashboard. In the cloud, storage and compute are unbundled and metered, which means almost every architectural choice you make — how you partition a table, how long you retain raw data, whether a **data lake** consumer reads from object storage directly or through a warehouse, whether a pipeline runs hourly or continuously — has a line item attached to it. That's a gift as much as a trap: it means cost is no longer someone else's problem discovered a year later, it's a lever you pull at design time, right alongside performance and reliability. Architects who treat cost as a post-launch FinOps cleanup exercise are the ones who get summoned to explain the invoice; architects who model it during design are the ones who get to explain why it's *lower* than expected.

## Storage Economics: Hot, Warm, Cold, and the Retention Tax

Cloud object storage is priced in tiers that mirror how often data is actually read. **Hot** storage costs the most per gigabyte but has no retrieval fee and low-latency access — it's for data queried constantly, like the current quarter's fact tables. **Warm** storage costs less per gigabyte but adds a small retrieval charge and slightly higher latency — a reasonable home for last year's data that's queried occasionally for trend reports. **Cold/archive** storage is dramatically cheaper per gigabyte but comes with real retrieval latency (minutes to hours) and a retrieval fee that can itself be substantial — appropriate only for data you're keeping for compliance and rarely, if ever, expect to read.

The trap architects walk into is what's informally called the **retention tax**: data that started in hot storage because it was actively used, then quietly stayed there long after its actual query rate dropped to zero, because nobody built a tiering policy or a deletion date into the architecture. A table queried daily during its first month and then almost never again, but never moved to a cheaper tier, can cost five times more to store over its life than it needed to. The fix isn't a one-time cleanup — it's an architectural default: every table or partition gets a stated retention and tiering policy at design time, ideally automated (lifecycle rules in S3/ADLS, TTLs on partitions), not left to whoever remembers to run a cleanup script.

## Compute & Query Economics: Why a Bigger Cluster Can Cost Less

The instinct carried over from on-prem thinking is that a smaller instance is always the cheaper choice, because a smaller instance has a lower hourly rate. In cloud, elastic compute, that instinct is often wrong. Most modern query engines and warehouses (Snowflake, BigQuery, Redshift Serverless, Databricks SQL) bill by compute-time consumed, not by a flat hourly server rate you keep whether you use it or not. A query that runs on a larger cluster and finishes in one-quarter the time can cost less in total compute-seconds than the same query crawling on a smaller cluster for four times as long — especially once auto-suspend means the larger cluster also isn't billed while idle. Sizing for "cheap per hour" instead of "cheap per query" is one of the most common cost mistakes architects carry over from a data-center mindset.

{: .important }
> The counterintuitive rule worth internalizing: **a bigger cluster that finishes faster can cost less than a smaller one that grinds**, once billing is metered by compute-time rather than a flat server rate. Before rightsizing anything down to save money, check whether the query is compute-bound and would actually finish faster — and cheaper — on more compute, not less.

## The Egress Trap: When Moving Data Costs More Than Storing It

Storage and compute inside a single cloud region are often cheap or even free to move between each other. **Egress** — data leaving a cloud provider's network, or crossing regions/availability zones within it — is billed per gigabyte moved, and it is one of the most consistently underestimated line items in a cost model. A hybrid or multi-cloud architecture that replicates a warehouse's output to a BI tool hosted elsewhere, or that lets a data-science team pull raw event data out to a laptop or a different cloud for model training, can rack up transfer charges that dwarf the storage cost of the data itself. The trap is architectural, not accidental: every time a design draws an arrow across a network boundary — cross-region replication for disaster recovery, a multi-cloud analytics tool, a partner receiving a nightly extract — that arrow has a per-gigabyte price tag that needs to show up in the cost model before the design is approved, not after the first bill.

| Cost Lever | What Drives the Cost | The Common Trap | The Architectural Fix |
|---|---|---|---|
| Storage tiering | Hot / warm / cold price-per-GB spread | The **retention tax**: hot data that outlives its query rate | Automated lifecycle policies tied to actual access patterns, not manual cleanup |
| Compute sizing | Billed compute-seconds, not server-hours | Assuming smaller = cheaper regardless of query duration | Size for cost-per-query; auto-suspend/serverless for bursty workloads |
| Data transfer (**egress**) | Per-GB charge on cross-region/cross-cloud movement | Multi-cloud or hybrid designs that repeatedly move the same data | Keep compute next to storage; replicate deliberately, not by default |
| Join vs. denormalization | Compute cost of scanning and shuffling normalized tables at query time | Treating normalization as "free" because it was free on a fixed on-prem box | Materialize wide/denormalized tables where read volume far exceeds write volume |
| Query patterns | Full scans vs. pruned partitions | Ad-hoc queries against unpartitioned raw tables | Partition and cluster by the columns workloads actually filter on |

## When Denormalization Is Cheaper Than Joins

A normalized schema minimizes storage and write-side redundancy, which was the right trade-off when storage was the expensive resource and compute was fixed. At cloud scale, the trade-off frequently flips: storing a wider, denormalized table costs a little more in storage (which is cheap) in exchange for eliminating repeated joins at query time (which consume metered compute, every single time a query runs). A fact table joined to five dimension tables on every BI dashboard refresh is paying that join cost hundreds or thousands of times a day; materializing the join once into a wide table — the same instinct behind the One Big Table pattern covered later in this guide — turns a recurring compute cost into a one-time transformation cost. The rule of thumb: normalize where writes are frequent and storage dominates the cost; denormalize where reads vastly outnumber writes and compute dominates it. Neither is universally "correct" — the cost model tells you which one your workload actually needs.

Running an actual cost-estimation exercise before committing to a design — sketching expected data volume, query frequency, and retention against a cloud provider's published pricing — catches most of these traps before they become a line item. It's a cheap hour spent in a design review against an expensive quarter spent explaining an invoice.

<!-- prevnext:start -->

---

| [&larr; Previous: Cost & Performance Architecture](./) | [Next: Performance Architecture: Tuning by Workload &rarr;](02-performance-tuning-by-workload/) |
|:---|---:|

<!-- prevnext:end -->
