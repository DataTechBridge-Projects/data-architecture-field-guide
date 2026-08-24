---
title: AWS, Azure & Hybrid/Multi-Cloud Tooling for Data Professionals
parent: "Foundations: Bridging from Legacy DW & ETL"
nav_order: 7
---

# AWS, Azure & Hybrid/Multi-Cloud Tooling for Data Professionals
{: .no_toc }

*Part 1: Theory & Foundations &middot; Foundations: Bridging from Legacy DW & ETL*

An architect who only knows one cloud vendor's service names will still design the *shape* of a system correctly but will burn weeks re-learning the *implementation* every time a client, employer, or acquisition runs on the other one — and will completely misjudge the cost and complexity of a genuinely hybrid estate, which is now the default rather than the exception. [Batch, Near-Real-Time & Real-Time Processing](06-batch-realtime-and-robust-pipelines/) established the latency tiers a pipeline can run at; this topic is about *where* that pipeline actually runs — the concrete AWS and Azure services that implement ingestion, storage, transformation, and serving, and what changes when they have to work together across a cloud boundary instead of inside just one.

## AWS for data professionals

AWS's data services map cleanly onto the layers this course keeps returning to. **Amazon S3** is the default **object storage** layer underneath most AWS data lakes — cheap, durable, and the landing zone every other service reads from and writes to. **AWS Glue** is the managed **ETL**/**ELT** and cataloging layer: it crawls S3 to infer schema into the Glue Data Catalog and runs Spark-based transformation jobs without you managing a cluster. **Amazon Redshift** is AWS's cloud data warehouse, the OLAP engine for structured, high-concurrency BI workloads. **Amazon Kinesis** is the streaming ingestion service — the AWS equivalent of a managed Kafka for **RT**/**NRT** event pipelines. **Amazon EMR** is managed Hadoop/Spark for large-scale batch and ML workloads that need more control than Glue's serverless model offers. **AWS Lambda** rounds this out as the serverless compute layer for lightweight, event-triggered transformation — a file lands in S3, a Lambda function fires. Together these give an architect a full **medallion**-capable stack without provisioning a single server.

## Azure for data professionals

Azure's naming is different but the layers underneath are the same ones an architect already recognizes. **Azure Data Lake Storage (ADLS Gen2)** is Azure's object storage foundation, built on Blob Storage with a hierarchical namespace that makes directory-style operations cheaper than flat blob storage alone. **Azure Synapse Analytics** unifies data warehousing and big-data analytics in one workspace — it's simultaneously Azure's answer to Redshift and, through its Spark pools, to parts of what EMR does on AWS. **Azure Data Factory (ADF)** is the orchestration and **ELT** pipeline-authoring tool — the closest analog a Talend or Informatica veteran will find to their old drag-and-drop pipeline canvas, now pointed at cloud sources and sinks instead of on-prem ones. **Azure Databricks** is a first-party, deeply integrated Spark platform for large-scale transformation and machine learning, effectively Azure's version of EMR but with a stronger notebook-first workflow. **Azure Stream Analytics** handles streaming ingestion and windowed aggregation, Azure's answer to Kinesis.

| Layer | AWS | Azure |
|---|---|---|
| Object storage | S3 | ADLS Gen2 |
| Cataloging / serverless ETL | Glue | Synapse (Spark pools) / Purview |
| Pipeline orchestration | Glue Workflows / Step Functions | Data Factory |
| Big-data compute (Spark/Hadoop) | EMR | Databricks |
| Cloud warehouse (OLAP) | Redshift | Synapse (dedicated SQL pools) |
| Streaming ingestion | Kinesis | Stream Analytics |
| Event-driven compute | Lambda | Azure Functions |

{: .important }
> Don't memorize this table as a set of interchangeable parts — the services on each side differ enough in pricing model, scaling behavior, and operational ceiling that swapping one in for the other is a real design decision, not a find-and-replace. Redshift's node-based pricing and Synapse's dedicated-pool model, for instance, produce very different cost curves for the same workload.

## Hybrid and multi-cloud architectures

A **hybrid architecture** keeps some workloads on-premises — often for regulatory, latency, or sunk-cost reasons — while extending others into one cloud; a **multi-cloud architecture** deliberately runs workloads across two or more cloud providers (commonly AWS, Azure, and GCP together) rather than treating a second vendor as a migration target. Neither is chosen for its own sake — an architect ends up here because of a merger that inherited two different clouds, a regulator that requires certain data to stay in-country on infrastructure the company already owns, or a deliberate hedge against a single vendor's pricing and outage risk.

The cost of that flexibility is real and shows up in four places an architect has to design for explicitly, not discover in production:

- **Latency**: a query or pipeline stage that crosses a cloud or on-prem boundary pays network latency that an intra-region call never does — fine for a nightly batch job, potentially disqualifying for an RT pipeline.
- **Data transfer / egress cost**: moving data *out* of a cloud provider is metered and can dominate a hybrid architecture's bill if the data-gravity direction isn't planned deliberately — the general rule is to move compute to the data, not the reverse, wherever possible.
- **Governance and consistency**: access control, lineage, and cataloging tools don't automatically span providers — a **data catalog** or **RBAC** policy built for one cloud needs a deliberate federation strategy (or a third-party tool layered on top) to mean anything on the other.
- **Operational duplication**: every additional platform is another set of IAM models, monitoring dashboards, and on-call runbooks a team has to know, which is real headcount and cognitive cost even when the technology itself works.

None of this makes hybrid or multi-cloud wrong — a regulated bank with data-residency requirements, or an enterprise that acquired a company running on the other major cloud, often has no better option — but it does mean an architect should treat "just run it on both" as a deliberate, costed trade-off, following the same build-vs-buy-vs-compose reasoning this course develops further once the decision framework is introduced. Knowing both vendors' service maps, and the tax that crossing between them adds, is what turns a cloud migration or a multi-region design from a guess into an estimate.

<!-- prevnext:start -->

---

| [&larr; Previous: Batch, Near-Real-Time & Real-Time Processing: Building Robust Pipelines](06-batch-realtime-and-robust-pipelines/) | [Next: Migrating a Legacy Warehouse to the Cloud: Patterns & Pitfalls &rarr;](08-migrating-legacy-warehouse-to-cloud/) |
|:---|---:|

<!-- prevnext:end -->
