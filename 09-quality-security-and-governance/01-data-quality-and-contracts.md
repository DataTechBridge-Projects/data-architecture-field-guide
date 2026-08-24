---
title: "Data Quality & Data Contracts: How Much Quality Is Enough?"
parent: Quality, Security & Governance
nav_order: 1
---

# Data Quality & Data Contracts: How Much Quality Is Enough?
{: .no_toc }

*Part 5: Running It Like a Platform &middot; Quality, Security & Governance*

A pipeline that runs every night without failing is not the same thing as a pipeline you can trust — [DataOps, Orchestration & Metadata](../08-dataops-orchestration-and-metadata/) gets a job scheduled, retried, and traceable end to end, but none of that tells you whether the numbers it produced are actually correct, or whether a schema that "just worked" yesterday quietly broke a downstream report today. An architect who only optimizes for pipelines *running* is optimizing for the wrong failure mode — a page at 2 a.m. is expensive, but a wrong number that sails silently into a board deck is worse, because nobody even knows to distrust it. This topic is about making correctness an engineered property of the platform instead of a hope pinned on the last team that touched the data.

## Why trust is the product

In a legacy warehouse, "data quality" often meant a reconciliation report an analyst ran once a month, eyeballing row counts against a source system. That worked when a handful of known ETL jobs fed a single warehouse. It breaks down completely once dozens of teams each own a pipeline feeding a shared lakehouse: nobody's watching all of them, and bad data doesn't announce itself. The job succeeds, the load completes on schedule, and a currency field silently switches from dollars to cents mid-flight, or an upstream API starts omitting a field and a null rate climbs from 0.1% to 40% without a single error thrown anywhere in the DAG.

The real cost isn't the bug — it's the erosion that follows it. The first time a stakeholder catches a dashboard showing an obviously wrong number, every number that dashboard has ever shown becomes suspect, and every number it shows afterward gets manually double-checked before anyone acts on it. Rebuilding that trust takes far longer than fixing the underlying defect ever did. **Data quality**, treated as an architectural concern rather than an afterthought, is the discipline of engineering pipelines so that failures are caught, loud, and attributable — not eliminated, which is impossible, but bounded and made visible before a human downstream discovers them the hard way.

## Shift-left: catching it at the producer

Historically, quality checks lived at the very end of the pipeline — a validation query run against the finished tables a BI team was about to build a dashboard on, reconciliation after the fact. **Shift-left testing** moves those checks as far upstream, toward the producer, as possible: validate shape and values at the ingestion boundary before a bad batch ever lands in bronze, and run schema, null-rate, and uniqueness checks at every medallion transition — bronze to silver, silver to gold — instead of waiting until gold is the only place anything gets tested.

The earlier a broken batch is caught, the cheaper it is to fix. A check that fails in CI before a producer's change merges costs a rejected pull request and five minutes of the producer's time. The same defect, caught three layers downstream instead, costs an incident channel, a root-cause investigation across every model built on top of it, and a reprocessing job that has to figure out exactly which rows are now wrong. A data pipeline is software, and the same shift-left instinct that moved application testing out of a QA phase and into the commit itself applies here: quality gates belong at every boundary where one team hands data to another, not only at the boundary where a consumer happens to query it.

## Data contracts: the agreement a producer can't quietly break

A **data contract** formalizes the producer-consumer relationship as an explicit, versioned interface instead of tribal knowledge passed around in a Slack thread. It declares a dataset's schema, the semantics of each field (what a column actually means, and in what unit), a freshness guarantee, and a quality threshold — and, critically, it's *enforced*: a change that violates the contract fails a build instead of shipping quietly. [The dbt Paradigm](../07-transformation-and-modern-data-stack/02-dbt-paradigm-contracts-idempotency/) already covered one concrete mechanism for this inside the warehouse — `contract: enforced: true` on a dbt model plus schema tests. The architectural idea generalizes well beyond that one tool: any team whose system emits events or tables that another team consumes should publish a contract for it, the same way a backend team publishes an API spec instead of leaving another team to reverse-engineer the JSON by trial and error.

## Schema evolution and breaking changes

**Schema evolution** — a dataset's structure changing as the business it describes changes — is not the question; it happens to every dataset eventually. The question a contract has to answer is whether a given change is *additive* or *breaking*. Adding a new nullable column, or a new event type nothing downstream depends on yet, is additive: existing consumers keep working untouched. Renaming a column, changing a type, narrowing what a value is allowed to mean, or removing a field a downstream model already reads is a **breaking change**, and it needs a version bump plus a migration window — both the old and new shape served in parallel for a defined deprecation period — so consumers fail on their own schedule instead of at 3 a.m. because a producer shipped on a Tuesday.

## Quality SLAs: how much is enough, and where to enforce

Zero data-quality incidents is not an achievable target, and chasing it uniformly is itself a cost decision most organizations shouldn't make. A **quality SLA** states, per dataset, the bar it actually commits to — 99.9% of rows pass null checks, freshness within fifteen minutes, referential integrity holds on every key — sized to how consequential that dataset is. A table a finance team certifies for revenue recognition earns a tight SLA and heavy enforcement; a table backing an internal engagement dashboard doesn't need the same rigor, and pretending otherwise just slows everyone down for no real reduction in risk.

Where that enforcement actually happens matters as much as the SLA number itself:

| Enforcement point | What it catches | Typical mechanism | Cost of skipping it |
|---|---|---|---|
| Producer / ingestion boundary | Malformed events, wrong types, contract violations before they land | Schema validation on write, contract check in CI | Bad data is baked into bronze; every downstream layer now needs a replay |
| Bronze &rarr; Silver | Duplicates, null-rate drift, referential integrity | Automated tests (e.g. dbt/Great Expectations-style checks), row-count deltas | Garbage propagates into every silver and gold model built on top |
| Silver &rarr; Gold | Business-rule correctness, aggregation sanity | Anomaly detection, range checks, reconciliation to source totals | Stakeholder-facing numbers are wrong; this is where trust actually breaks |
| Gold &rarr; Consumption | Freshness, certification status | Catalog "certified" badge, freshness monitor and alert | A consumer builds a decision on stale or unvetted data without knowing it |

{: .important }
> A quality SLA's job isn't to prevent every bad row — that's not achievable. It's to guarantee that when one gets through, someone finds out before a stakeholder does. An SLA with no enforcement point that actually pages a human on breach is a document, not a control.

Getting a single dataset's shape and freshness right is necessary but not sufficient — it says nothing about whether *the same real-world entity* is represented consistently across every system that touches it, which is exactly the problem the next topic takes on.

<!-- prevnext:start -->

---

| [&larr; Previous: Quality, Security & Governance](./) | [Next: Master Data Management: Golden Records, Matching & Stewardship &rarr;](02-master-data-management/) |
|:---|---:|

<!-- prevnext:end -->
