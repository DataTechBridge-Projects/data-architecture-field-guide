---
title: "AWS Samples: Data Mesh Reference Architecture (DataZone, CDK & CloudFormation)"
parent: Hands-on Tutorials
nav_order: 10
---

# AWS Samples: Data Mesh Reference Architecture (DataZone, CDK & CloudFormation)
{: .no_toc }

*Part 8: Hands-on Tutorials &middot; Hands-on Tutorials*

**Read the full tutorial:** [AWS Samples: Data Mesh Reference Architecture (DataZone, CDK & CloudFormation)](https://github.com/aws-samples/data-mesh-datazone-cdk-cloudformation) &#8599;

*Link verified 2026-08-23.*

This is a deployable reference repo, not a blog walkthrough: clone it and CDK/CloudFormation stand up a multi-account **data mesh**, with DataZone as the central catalog and subscription workflow connecting producer and consumer domains.

It's the full-stack, infrastructure-as-code companion to [Data Mesh: Decentralized Domain Ownership](../02-architecture-patterns-deep-dive/05-data-mesh/) — worth deploying once to see domain ownership, a central catalog, and self-serve subscription enforced as actual AWS resources rather than a diagram.

```mermaid
flowchart LR
    subgraph Producer Domain Account
        A[Domain data in S3/Glue] --> B[Publish as data product]
    end
    B --> C[Amazon DataZone - central catalog]
    C --> D[Subscription request / approval]
    subgraph Consumer Domain Account
        D --> E[Granted access via Lake Formation]
        E --> F[Consume in Athena / Redshift]
    end
    G[CDK / CloudFormation] -.deploys.-> A
    G -.deploys.-> C
    G -.deploys.-> E
```

<!-- prevnext:start -->

---

| [&larr; Previous: Multimodal RAG with Amazon Bedrock Data Automation & Knowledge Bases](09-rag-bedrock-vector-store/) |  |
|:---|---:|

<!-- prevnext:end -->
