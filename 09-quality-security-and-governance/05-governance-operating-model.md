---
title: "The Governance Operating Model: Owners, Stewards, Custodians & the Council"
parent: Quality, Security & Governance
nav_order: 5
---

# The Governance Operating Model: Owners, Stewards, Custodians & the Council
{: .no_toc }

*Part 5: Running It Like a Platform &middot; Quality, Security & Governance*

A policy engine can enforce *that* access is controlled, but it cannot decide *whether* a given exception request should be approved, *who* is on the hook when a golden record turns out to be wrong, or *what* happens when two domains define "active customer" differently. Those are organizational questions, not technical ones. [Data Strategy & Roadmaps](04-data-strategy-and-roadmaps/) covered how to decide what to build next; this topic closes out the group by covering who is actually accountable for everything the previous four topics described, once it's built — because a RACI chart, not a product, is what answers "who gets paged" when governance breaks down.

## Governance is an org design problem before it's a tooling problem

It's tempting to treat governance as solved once [Security & Governance](03-security-and-governance/)'s policy-as-code and catalog tooling is in place — the access rules are versioned, the PII is tagged, the lineage is traceable. But tooling only enforces decisions that a human already made: which fields count as sensitive, which exception requests are reasonable, which domain's definition of "revenue" wins when two disagree. A **governance operating model** is the explicit answer to who makes those decisions, so the answer isn't "whoever's loudest in the Slack thread" or "whoever happens to still remember why the rule exists."

## The three roles: owner, steward, custodian

Most working operating models separate accountability into three distinct roles, and the most common failure is collapsing them into one person or, worse, leaving all three unassigned and assuming "the data team" covers it.

| Role | Accountable for | Typically held by |
|---|---|---|
| **Data owner** | Business meaning, value, and risk tolerance for a domain's data; approves classification and access exceptions | A business-side leader close to the domain (VP of Finance for financial data, Head of Support for ticket data) |
| **Data steward** | Day-to-day correctness: definitions, quality rules, survivorship logic for the golden record, fielding disputes about what a column means | A senior analyst or subject-matter expert embedded in the domain, not a central data team member |
| **Data custodian** | Technical implementation: the pipelines, access controls, and infrastructure that enforce what the owner and steward decide | The platform/data engineering team |

The **owner** decides *what should be true*; the **steward** maintains the day-to-day reality of it being true; the **custodian** builds and runs the systems that make it true automatically. A single person can hold more than one of these roles in a small organization, but the three accountabilities still have to be named separately — otherwise a quality incident gets routed to whichever of the three happens to answer the page, regardless of whether they actually have the authority to fix it.

## RACI for a data product: making the roles concrete

A role list is abstract until it's applied to specific recurring decisions. A **RACI** (Responsible, Accountable, Consulted, Informed) matrix forces that specificity:

| Decision or task | Responsible | Accountable | Consulted | Informed |
|---|---|---|---|---|
| Classify a new column as PII | Custodian (engineer) | Owner | Steward | Governance council |
| Approve a one-off access exception | Steward | Owner | — | Governance council |
| Fix a wrong match/merge rule in MDM | Custodian | Steward | Owner | — |
| Resolve a cross-domain definition conflict | Steward (both domains) | Governance council | Owners (both domains) | — |
| Set the organization-wide PII retention floor | Governance council | Governance council | Owners | All stewards |

{: .important }
> A RACI with two people marked **Accountable** for the same decision isn't a stricter version of governance — it's a stalemate waiting to happen the first time those two disagree. Exactly one accountable party per row, always; "Consulted" is where you put everyone else who has a legitimate stake.

## The governance council: setting the floor, not every decision

A **governance council** — typically owners and stewards from each major domain, plus a central governance/compliance function — exists to do two things a single domain can't do for itself: set the small number of non-negotiable, organization-wide policies (this is the "floor" from [Security & Governance](03-security-and-governance/)'s **federated governance** model — PII classification standards, minimum retention, encryption baselines), and arbitrate disputes that cross domain boundaries, like two teams' conflicting definitions of "active customer." What the council deliberately does *not* do is approve every individual access request or every domain's internal quality rule — that would recreate the central-bottleneck problem federated governance exists to avoid. The council's job is to meet rarely enough that it isn't a tax on daily work, but regularly enough that disputes don't fester for months before anyone with cross-domain authority looks at them.

## Escalation: what happens when domains disagree

A working operating model names an escalation path *before* the first real dispute, not during it:

| Tier | Who resolves it | Example |
|---|---|---|
| 1 | The two domains' stewards, directly | "Our `order_date` and your `order_date` mean slightly different things — let's align the definition" |
| 2 | Governance council | Stewards can't agree; the definitions conflict with how each domain's KPIs are calculated |
| 3 | Executive sponsor | The council itself is split, or the dispute has real budget or compliance consequences |

Most disputes should resolve at Tier 1 — if most disputes are reaching the council, that's a signal the stewards closest to the data don't have enough delegated authority, not that the council needs to meet more often.

## Sizing the operating model to the organization

A ten-person startup does not need a formal council with quarterly meetings — the owner, steward, and custodian roles can be three hats on two people, and "escalation" is a conversation down the hall. The operating model earns its formal structure at the point a [data mesh](../02-architecture-patterns-deep-dive/05-data-mesh/)-style organization has enough autonomous domains that informal alignment stops scaling — which is the same inflection point [Data Products & the Mesh Operating Model](../11-serving-reliability-and-mesh-operating-model/03-data-products-and-mesh-operating-model/) describes for the platform itself. The mistake in both directions is symmetric: over-formalizing governance in a small organization just adds process with no disputes to resolve, and under-formalizing it in a large, domain-autonomous one is how a mesh quietly degrades into the "free-for-all" federated governance was designed to prevent.

<!-- prevnext:start -->

---

| [&larr; Previous: Data Strategy & Roadmaps: Maturity Assessment, Gap Analysis & Business Alignment](04-data-strategy-and-roadmaps/) | [Next: Privacy & Industry Regulation: GDPR, CCPA, HIPAA & BFSI Compliance (Basel, PCI DSS, AML/KYC, SOX) &rarr;](06-privacy-and-industry-regulation/) |
|:---|---:|

<!-- prevnext:end -->
