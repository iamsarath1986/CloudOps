# Milestone M03 — Cloud Architecture Fundamentals (Vendor-Neutral) and the First Solution Design

*Phase 03 · ~2 weeks · Months ~2.6–3.1 · Prerequisites: M01 · Unlocks: M04 (AWS core), M13 (Azure core)*

## Why this phase exists

The JD's first responsibility is to *"develop IT solutions that meet customer requirements … robust, appropriate, of high quality, conforming to corporate standards"*, and it asks you to *"develop solution architecture for assigned projects in accordance with established EA standards"* and *"ensure that the developed architecture/solution meets service-level requirements in terms of expected availability, capacity, continuity, security, performance, and reliability."* That is architecture work, and a *Senior* engineer is expected to do it.

This phase deliberately contains **no cloud console**. You learn the ideas that AWS and Azure both implement — regions, zones, fault domains, stateless tiers, queues, caches, RTO/RPO — so that when you meet a product name in M04 you are learning a *row* of the concept table, not a button. It is the thing that makes your knowledge transfer from AWS to Azure in M13 rather than starting again.

You also write **Runway's first Solution Design Document** and **ADR-002** (AWS first). Being able to defend a design on paper before building it is the Senior skill the JD describes.

## Concepts to learn

Read [Appendix A1](../appendix/A1-cloud-architecture-concepts.md) alongside this milestone; it is the concept-to-service map.

- **Failure domains:** host → rack/fault domain → availability zone → region → provider. What each protects against and what it costs. Why "multi-AZ" is the default for production and "multi-region" is a business decision.
- **Availability maths:** serial dependencies multiply (99.9% × 99.9% = 99.8%); redundancy adds nines only if failures are independent. MTBF, MTTR, and why MTTR is usually the cheaper lever.
- **Stateless vs stateful tiers:** why you push state into managed databases, caches and object stores so compute can be disposable, scaled horizontally and replaced.
- **Scaling:** vertical vs horizontal; what stops horizontal scaling (shared state, sticky sessions, database connections).
- **Load balancing:** L4 vs L7, health checks, connection draining, cross-zone balancing.
- **API gateways and reverse proxies:** authentication, rate limiting, routing — and when a load balancer is enough.
- **Caching:** cache-aside, TTLs, invalidation, stampede, what must never be cached.
- **Queues:** decoupling, buffering, backpressure, at-least-once delivery and why consumers must be idempotent, dead-letter queues, visibility timeouts.
- **Databases:** managed relational vs self-managed, read replicas vs standby replicas, connection pooling, backups vs replication (replication faithfully copies your deletion).
- **Object storage:** durability vs availability, storage classes, lifecycle rules.
- **Identity:** authentication vs authorisation; human vs workload identity; why short-lived credentials beat long-lived keys.
- **Secrets:** lifecycle — creation, distribution, rotation, revocation, audit.
- **Resilience patterns:** timeouts, retries with jitter, circuit breakers, bulkheads, load shedding, graceful degradation.
- **RTO and RPO** as business decisions that constrain architecture and cost; DR strategies: backup-restore, pilot light, warm standby, active-active.
- **The shared responsibility model**, concretely for a VM, a managed Kubernetes cluster and a managed database.
- **Well-Architected thinking:** operational excellence, security, reliability, performance efficiency, cost optimisation, sustainability — as review questions, not slogans.
- **Solution design practice:** requirements (functional, non-functional), constraints, assumptions, options, decision, risks, cost estimate, delivery estimate. How to present an estimate to a project manager honestly (ranges and assumptions, not single numbers).

## Architecture decisions in this phase

### ADR-002 — AWS first, Azure second
- **Problem:** The JD requires *"proficiency in managing and operating AWS and Azure"*. Learning both at once halves the depth of each.
- **Options:** (a) Azure first; (b) AWS first; (c) both in parallel.
- **Decision:** (b) AWS first, Azure from M13.
  1. AWS primitives are explicit and unforgiving — you build internet gateways, NAT, route tables, security groups, NACLs and IAM policy JSON yourself — so they teach networking and identity mechanically.
  2. EKS is the less-managed Kubernetes (VPC CNI IP consumption, Pod Identity/IRSA, the load balancer controller, Karpenter), so it teaches more operation.
  3. Two of the three certifications the JD names are AWS Professional.
  4. Azure's distinctive strengths — Entra ID as enterprise identity, management-group governance with Azure Policy, Arc for hybrid, Azure OpenAI — line up with the *later* phases (governance, platform, hybrid, AI), so you learn Azure where it is strongest.
- **Counter-arguments, stated honestly:** enterprises like the employer are often Microsoft-heavy; PowerShell is named; Entra ID is usually the identity source of truth. Mitigations: Entra becomes the workforce IdP for *both* clouds in M13, and you operate Azure for ~10 months before the end.
- **Trade-offs:** Azure proficiency arrives at month ~14; if a job search starts earlier, your Azure story is thinner. Option (c) was rejected because it produces two shallow skill sets and doubles cost.
- **The rule that follows:** never duplicate every exercise on both clouds. Azure phases teach the *differences* and the *enterprise control plane*.

## Hands-on labs

These are paper and whiteboard labs. Being able to do them without a console is the point.

1. **Composite availability.** Compute the availability of: ALB → 2 app instances in 2 AZs → single-AZ database; then with a Multi-AZ database. Explain where the nines went.
2. **Find the single points of failure** in three diagrams (a classic VM + DB, a Kubernetes app with one NAT gateway, a queue-based system with one consumer). Name the failure, the blast radius and the fix, with its cost.
3. **Queue semantics.** Simulate at-least-once delivery with `carrier-sim` and a local Redis list: kill a consumer mid-message. Prove duplicate processing, then make the consumer idempotent.
4. **Cache stampede.** Expire a hot key under load (local `hey` or `k6` against `cargo-api` with a naive cache). Watch the database spike. Fix with request coalescing or TTL jitter.
5. **RTO/RPO worksheet.** For each Runway data store (shipments DB, object storage, Terraform state, logs), write the RTO/RPO you would *want*, the strategy that achieves it and the cost class. This becomes an input to M12.
6. **Estimate a delivery.** Estimate effort and monthly cost for "Runway v1 on AWS" (M04–M09) as a range with stated assumptions. You will compare with actuals — this practises the JD's "guide Project Managers … according to agreed costs and timescales".

## Project implementation

- `docs/solution-designs/SDD-001-runway-v1.md` — the first Solution Design Document using the template in [Appendix A2](../appendix/A2-documentation-standards.md): context, requirements (incl. NFRs: availability target, RTO/RPO, latency, security, cost ceiling), constraints (budget, 10 h/week, lab not production), options, the chosen logical architecture (vendor-neutral: edge → L7 LB → stateless API tier → queue + worker → relational DB + cache + object storage), risks, cost estimate, delivery estimate.
- `docs/architecture/logical-architecture.md` with a Mermaid diagram and three request paths (read, write, async ETA recompute) traced hop by hop.
- `docs/adr/ADR-002-aws-first-azure-second.md`.
- `docs/estimates/runway-v1-estimate.md` — ranges and assumptions.
- `cargo-api`: add an in-process cache-aside layer against a local Redis for `GET /shipments/{id}` and an idempotent consumer for the ETA recompute queue (local Redis list for now; SQS in M04). Keep it small.

## Engineering problems to hit deliberately

| Problem | How to cause it | What you should learn |
|---|---|---|
| Hidden single point of failure | One NAT, one consumer, one DB | Redundancy is per tier, not per diagram |
| Duplicate processing | Kill consumer mid-message | At-least-once → idempotent consumers |
| Cache stampede | Hot key expires under load | Caches can *cause* database outages |
| Composite SLA illusion | Multiply component SLAs | Serial dependencies lose nines |
| Estimate as a single number | Give "3 weeks" | Estimates are ranges with assumptions |

## Troubleshooting exercises

- **"After the nightly ETA job, some shipments show two ETA-change notifications."** Cause: at-least-once delivery plus a non-idempotent consumer. Diagnose from logs with `request_id`/message IDs.
- **"Every hour on the hour the database CPU spikes and the API slows down."** Cause: synchronised TTL expiry of hot keys. Prove it from timing correlation before you touch the cache code.

## Deliverables

- [ ] `docs/solution-designs/SDD-001-runway-v1.md`
- [ ] `docs/architecture/logical-architecture.md` with diagram and three request paths
- [ ] `docs/adr/ADR-002-aws-first-azure-second.md` — rejected options argued at greater length than the decision
- [ ] `docs/continuity/rto-rpo-worksheet.md`
- [ ] `docs/estimates/runway-v1-estimate.md`
- [ ] Idempotent consumer and cache-aside in `cargo-api`, with tests for duplicate delivery

## Completion criteria

- Draw Runway's logical architecture from memory and trace a write request through every hop, naming the failure mode at each.
- Compute composite availability for two variants and explain the difference in one sentence.
- Explain backup vs replication with a concrete data-loss scenario.
- Explain at-least-once delivery and demonstrate your consumer is idempotent with a test.
- Present SDD-001 in ten minutes to someone (or a recording) and answer "what did you reject and why?" for three decisions.

## Mentor checkpoints

- **Design review (the important one):** bring SDD-001 before writing ADR-002. Expect: "Which NFR drives the most cost?", "What happens when the database is unavailable for five minutes?", "Why is this not over-engineered for a lab?"
- **Estimate review:** the mentor will challenge assumptions, not numbers. Keep the estimate — you will compare it with actuals at M09.
- **The failure I'll inject after you think you're done:** a change in requirements (a tighter RPO) that invalidates one of your decisions. You will update the SDD, not start again.

## Resources

- AWS Well-Architected Framework — read the Reliability pillar ("Design principles", "Workload architecture", "Failure management") and the Cost Optimization pillar's design principles.
- Azure Well-Architected Framework — the Reliability checklist (compare the vocabulary with AWS's; the ideas are the same).
- Michael Nygard, *Release It!* (2nd ed.) — ch. 4 (Stability Antipatterns) and ch. 5 (Stability Patterns).
- Martin Kleppmann, *Designing Data-Intensive Applications* — ch. 1 (reliability, scalability, maintainability) and ch. 5 (replication).

## Estimate vs actual

| | Hours | Cost |
|---|---|---|
| Planned | 20 | $0 |
| Actual | | |
| Variance note | | |
