---
title: Architecture Decision Records, Anti-Patterns & War Stories
parent: Architecting for AI & Closing the Loop
nav_order: 2
---

# Architecture Decision Records, Anti-Patterns & War Stories
{: .no_toc }

*Part 7: The Frontier & The Defense &middot; Architecting for AI & Closing the Loop*

An architect who makes the right call but never writes down why loses that decision the moment they change teams — the next person inherits a lakehouse with a vector database bolted on and no record of what constraint forced that shape, so they either repeat the analysis from scratch or, worse, "fix" something that was deliberate. [Architecting for AI: Feature Stores, Vector Databases, RAG & Governance Guardrails](01-architecting-for-ai/) just added real architectural weight to your platform — a feature store, a vector database, a governance layer extended over models; this topic is about how you make decisions like that defensible, catalog the ways they go wrong, and survive the moment someone senior asks you to justify them.

## The architecture decision record: deciding in the open

An **ADR** (architecture decision record) is a short, permanent document that captures one decision at the moment it's made: the context that forced it, the options considered, the choice, and the consequences accepted. It is not a design document and not a slide deck — it's closer to a commit message for an architectural choice, meant to be read years later by someone who wasn't in the room.

A useful ADR is short enough to actually get written:

```markdown
# ADR-014: Use a managed vector database instead of pgvector

## Status
Accepted

## Context
Our RAG pilot needs sub-100ms retrieval over 40M embeddings, growing
20%/quarter. Our Postgres warehouse team has no experience operating
HNSW indexes at this scale.

## Decision
Adopt a managed vector database (Pinecone) rather than pgvector
in our existing warehouse.

## Consequences
+ No new operational burden on the warehouse team
+ Meets latency SLO out of the box
- New vendor, new bill, new access-control surface to govern
- Revisit if embedding volume plateaus and cost crosses $X/month
```

The habit worth building is deciding, explicitly, whether the choice is a **two-way door** (cheap to reverse — swap a caching layer, add an index) or a **one-way door** (expensive or impossible to reverse — pick a table format, commit to a vector database that becomes the system of record for embeddings tied to a governance regime). One-way doors earn a full ADR and a slower process; two-way doors don't need the ceremony. Confusing the two in either direction — treating a reversible choice like it needs a steering-committee sign-off, or treating a one-way door like a quick config change — is itself an anti-pattern.

## The anti-pattern catalog

An **anti-pattern** is a solution that looks reasonable, gets adopted repeatedly, and reliably produces a bad outcome — worth naming so you recognize it in a design review before it ships, not after.

| Anti-pattern | What it looks like | Why it fails | The fix |
|---|---|---|---|
| Lambda-by-default | Batch + streaming layers built for every pipeline "to be safe" | Doubles pipeline logic and cost for workloads that never needed sub-second latency | Ask "should this be streaming at all" before reaching for Lambda or Kappa |
| The mesh that became a mess | Domain teams own data products with no shared platform or standards | Ten domains, ten incompatible access patterns, no one can join across them | Federated governance: shared platform, locally owned products |
| Schema-on-read as an excuse | Raw files dumped in a lake with no contract, "we'll figure out the schema later" | Every consumer writes its own brittle parser; nobody agrees on grain | Data contracts enforced at the producer, even in a lake |
| The $50k query anti-pattern | Analysts given ad-hoc access to a pay-per-byte-scanned warehouse with no guardrails | One unfiltered `SELECT *` across years of unpartitioned data becomes a budget-line incident | Partitioning, query cost limits, and workload isolation before self-serve access |
| Governance as a gate, not a guardrail | Every new table or model needs a committee sign-off before use | Teams route around governance entirely; shadow pipelines multiply | Policy as code, applied automatically, reviewed by exception |
| The feature store as a second warehouse | Feature pipelines rebuilt independently of existing batch transforms | Training-serving skew reappears because "shared definition" was never actually shared | One feature definition, two serving paths, enforced in code |

## War story: the $50k query and the cost scars

The catalog entry above has a real shape to it: an analyst, given broad query access to a newly migrated cloud warehouse, ran a exploratory join across an unpartitioned five-year table with no `WHERE` clause, on a platform billed by bytes scanned. The query returned in nine minutes. The bill for that single query showed up two days later. The architecture wasn't wrong to give analysts self-serve access — the mistake was self-serve access without partitioning discipline, query cost caps, or a workload-isolation boundary between ad-hoc exploration and production reporting. The fix wasn't "restrict access"; it was "make the expensive path structurally hard to take by accident."

{: .important }
> Cost guardrails belong in the platform, not in a training deck. If a $50k mistake is technically possible for a single unreviewed query, it is only a matter of time before someone runs it — enforce partition filters and cost limits at the query-engine level, don't rely on analysts remembering a best practice.

## War story: the CDC loop and the reliability scars

The second scar is quieter and takes longer to notice. A team wired **CDC** (change data capture) from an operational database into the lake, and separately wired a reverse-ETL job to push aggregated results back into that same operational database for an internal tool. Nobody diagrammed the two pipelines together. The reverse-ETL write triggered a change event, which the CDC pipeline picked up as a new "change," which flowed back downstream and, through a second reverse-ETL cycle, wrote again — a slow-motion feedback loop that inflated a few tables by small amounts every night until a quarterly report was visibly wrong. The failure wasn't in either pipeline; it was in the absence of lineage that showed the two connected into a cycle at all.

## Defending your architecture: the board review

Both war stories point at the same defense mechanism: an architect who can produce the ADR that explains a decision, point to the anti-pattern it was specifically designed to avoid, and show the lineage graph that would have caught a loop like the CDC story before it shipped, is defending a design — not an opinion. That's the posture the capstone in this group asks you to practice for real, end to end, against a board that will ask exactly these questions.

<!-- prevnext:start -->

---

| [&larr; Previous: Architecting for AI: Feature Stores, Vector Databases, RAG & Governance Guardrails](01-architecting-for-ai/) | [Next: Capstone: Designing and Defending an AI-Ready Platform to the Board &rarr;](03-capstone-designing-and-defending-ai-ready-platform/) |
|:---|---:|

<!-- prevnext:end -->
