---
title: "A Mental Model for Architecture Choices: Table Formats, Cloud Providers, Hybrid Cloud & Build vs Buy"
parent: The Architect's Decision Framework
nav_order: 4
---

# A Mental Model for Architecture Choices: Table Formats, Cloud Providers, Hybrid Cloud & Build vs Buy
{: .no_toc }

*Part 2: The Architecture Landscape &middot; The Architect's Decision Framework*

The six-box reference architecture from [The Reference Architecture](03-reference-architecture/) tells you what decisions exist at each layer; it doesn't tell you how to actually make one when every option in front of you looks reasonable on a slide. This closing topic supplies that missing piece: one reusable mental model — six dimensions you can run any architecture choice through — applied to four decisions an architect faces again and again: which table format, which cloud provider, whether to go hybrid, and whether to build, buy, or compose. Everything after this topic in the course is, in one way or another, an instance of running a specific decision through this same model.

## The six dimensions

Any architecture choice worth deliberating over can be scored against the same six questions, regardless of whether the choice is a table format, a vendor, or an org-design pattern:

- **Cost**: not just sticker price — total cost of ownership across compute, storage, and egress, plus the engineering hours it takes to run the thing, whether that cost is front-loaded (build), ongoing (buy a subscription), or a mix (compose).
- **Control**: how much say you have over roadmap, data location, upgrade timing, and failure recovery. A vendor's SaaS gives you almost none of this; a from-scratch build gives you all of it — at the cost of owning all of it, too.
- **Complexity**: how many new moving parts, failure modes, and areas of required expertise this choice adds on top of what the team already operates.
- **Lock-in**: how expensive it is to leave once you're committed — proprietary APIs and formats, egress fees, staff who've specialized in one vendor's tooling, and the sheer volume of data that would need migrating.
- **Team skill**: what the team already knows how to operate at 2 a.m. versus what it would need to learn from scratch, and how long that learning curve realistically takes.
- **Reversibility**: essentially the two-way-door/one-way-door test from [Deciding Under Uncertainty](01-deciding-under-uncertainty/), applied specifically to this choice — can you walk it back at a bounded cost, or is this decision going to outlive the person who made it?

No single dimension decides a choice on its own — a cheap, low-control option that locks you in for a decade is not automatically better than an expensive, high-control one your team can unwind in a quarter. The value of the model is forcing all six onto the table at once instead of defaulting to whichever one is loudest in the room, usually cost.

{: .important }
> Lock-in and reversibility are the two dimensions architects most often skip, because cost and complexity are the ones a budget conversation forces you to state out loud. A choice that looks cheap and simple today but is a one-way door in eighteen months hasn't actually been fully costed — it's been costed for right now.

## Applying the model: table formats

**Delta Lake**, **Apache Iceberg**, and **Apache Hudi** all solve the same ACID-on-object-storage problem, covered in depth in [Table Formats: Delta vs Iceberg vs Hudi](../04-storage-and-table-formats/02-table-formats-delta-iceberg-hudi/) — but the six dimensions surface why the "right" one depends on what your team already runs, not on a feature checklist:

| Dimension | Delta Lake | Apache Iceberg | Apache Hudi |
|---|---|---|---|
| Cost | Low if already on Databricks; higher elsewhere | Comparable across engines by design | Higher operational cost tuning Copy-on-Write vs. Merge-on-Read |
| Control | Strong on Databricks; weaker off-platform | High — engine-neutral metadata | High for upsert-heavy pipelines specifically |
| Complexity | Low if Databricks-centric | Moderate — broadest engine matrix to learn | Higher — two table layouts to understand and choose between |
| Lock-in | Real toward the Databricks ecosystem | Lowest of the three by design intent | Moderate — tied to its own timeline model |
| Team skill needed | Spark/Databricks fluency | General multi-engine SQL/Spark fluency | Streaming-first (Flink/Spark) fluency |
| Reversibility | One-way door — hard to migrate off later | Closer to reversible, but still a real migration | One-way door for high-volume upsert pipelines |

## Applying the model: cloud providers

The **AWS** and **Azure** service maps from [AWS, Azure & Hybrid/Multi-Cloud Tooling for Data Professionals](../01-foundations/07-aws-azure-hybrid-tooling/) map onto the same six dimensions differently depending on what your team and your existing estate already look like:

| Dimension | AWS | Azure |
|---|---|---|
| Cost | Granular, usage-based pricing across many services | Bundled workspaces (Synapse) can simplify billing, at less granularity |
| Control | Fine-grained service composition | Deeper first-party integration, less à la carte |
| Complexity | Wide service catalog — more choices, more to learn | Fewer moving parts if staying inside Synapse/Databricks |
| Lock-in | High once pipelines are built around Glue/Redshift-specific features | High once workloads are built around Synapse-specific pooling |
| Team skill needed | Strongest fit for teams already SQL/Python-fluent from open tooling | Strongest fit for teams with an existing Microsoft/.NET stack |
| Reversibility | Genuinely portable at the storage layer (S3 + open formats); compute layer less so | Same pattern — ADLS Gen2 with open formats is portable; Synapse pools are not |

Neither column is categorically better — a team already deep in the Microsoft ecosystem pays a real, avoidable tax adopting AWS for no reason beyond novelty, and the reverse holds just as strongly.

## Applying the model: hybrid and multi-cloud

Running on-premises alongside one cloud (**hybrid**), or across two or more cloud providers deliberately rather than as a migration target (**multi-cloud**), trades every dimension against the single-cloud baseline in a fairly predictable direction:

| Dimension | Single cloud | Hybrid | Multi-cloud |
|---|---|---|---|
| Cost | Lowest — one provider's pricing and discounts | Higher — sunk on-prem cost plus cloud cost | Highest — duplicated tooling and egress across providers |
| Control | Bounded by one vendor's roadmap | High for on-prem workloads | Highest — no single vendor dependency |
| Complexity | Lowest — one IAM model, one monitoring stack | Higher — two operational worlds to run | Highest — N operational worlds, N sets of on-call runbooks |
| Lock-in | Real, but at least concentrated in one place | Lower cloud lock-in, but stuck with legacy on-prem debt | Lowest vendor lock-in, at the cost of never being deeply optimized on any one |
| Team skill needed | One platform's worth of expertise | On-prem expertise plus one cloud's | Multiple clouds' worth, genuinely rare to find in one team |
| Reversibility | Migrating off is a real project but a single one | Two migration projects, not one | Ongoing by design — nothing to "reverse," but never fully committed either |

Hybrid and multi-cloud are rarely chosen for their own sake — a regulator requiring data to stay on infrastructure the company already owns, or a merger that inherited two clouds, is a more common trigger than a deliberate hedge — but every one of these costs should be named and accepted explicitly, not discovered in the first cross-cloud egress bill.

## Applying the model: build vs. buy vs. compose

The same three options from [Deciding Under Uncertainty](01-deciding-under-uncertainty/) run cleanly through all six dimensions, which is a useful way to check a build-vs-buy-vs-compose instinct before it goes into a room with a CFO, a CISO, and a CTO in it:

| Dimension | Build | Buy | Compose |
|---|---|---|---|
| Cost | High upfront engineering cost, low marginal cost at scale | Predictable subscription cost, scales with usage/seats | Mixed — several smaller costs, easy to underestimate the sum |
| Control | Full control over every behavior | Minimal — bounded by the vendor's roadmap | Moderate — full control over the glue, none over each component internally |
| Complexity | High — you own every failure mode | Lowest — the vendor owns operating it | Higher than buy — you own the integration surface between components |
| Lock-in | None to a vendor, but high switching cost for your own custom system | Real — proprietary APIs, data formats, contract terms | Lower per-component, but real at the integration-glue level |
| Team skill needed | Deep engineering capacity across the whole stack | Minimal — mostly configuration and integration skill | Broad — familiarity with several tools instead of deep mastery of one |
| Reversibility | One-way door once the system is load-bearing | Two-way door if the vendor's data export is genuinely clean | Component-by-component — usually the most reversible of the three, one piece at a time |

Read as a table, compose's real advantage isn't that it's cheap or simple — it usually isn't, on either count — it's that it's the most reversible of the three at the component level, which is exactly why it fits a landscape where requirements, vendors, and even architecture patterns keep moving, the theme this whole group has been building toward.

<!-- prevnext:start -->

---

| [&larr; Previous: The Reference Architecture: Source to Ingest to Store to Transform to Serve to Consume](03-reference-architecture/) | [Next: Storage & Table Formats &rarr;](../04-storage-and-table-formats/) |
|:---|---:|

<!-- prevnext:end -->
