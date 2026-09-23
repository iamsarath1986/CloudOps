# Milestone M12 — Capacity, High Availability and Disaster Recovery

*Phase 12 · ~4 weeks · Months ~12.8–13.6 · Prerequisites: [M11](M11-sre-operations.md) (and M05's state discipline, M09's cluster) · Unlocks: [M17](M17-golden-paths.md); feeds [M16](M16-finops.md) and [M22](M22-hardening-capstone.md)*

---

## Why this phase exists

The JD requires that *"the developed architecture/solution meets service-level requirements in terms of expected availability, capacity, continuity, security, performance, and reliability"*, and names **resilience** among the cloud best practices. Senior engineers are asked for numbers: *how much traffic can it take, what fails first, how long to recover, how much data do we lose.* This milestone produces those numbers by measuring them.

It follows M11 because every load test that finds a limit and every DR drill is run as an incident, with the records and roles you now have. Doing it before the platform phases (M17+) means the golden path inherits tested defaults: resource requests from measurements, PDBs, topology spread, backup policy.

## Concepts to learn

**Capacity and performance**
- **Open versus closed load models.** A closed model (fixed virtual users) hides queueing, because users slow down when the system does. An open model (fixed arrival rate) is what the internet does to you. k6's `constant-arrival-rate` / `ramping-arrival-rate` executors.
- **Little's Law** (`L = λW`): concurrency = throughput × latency. Use it to size connection pools, worker concurrency and pod counts *before* testing, then check the prediction.
- **Saturation and the knee.** Latency is flat until utilisation of the bottleneck approaches 1, then it bends sharply. Find the saturating resource (pool, CPU limit, DB, a dependency, NAT, SQS consumer concurrency). **Bottlenecks shift**: fix one and the next appears.
- Autoscaling dynamics: HPA sync period and stabilisation, pod start time, Karpenter node launch time, image pull time. **Scaling latency is a capacity parameter.** Pre-scaling for known events. Scaling on queue depth (KEDA as the option, and ADR-worthy if you adopt it).
- Quotas and limits as hard ceilings: EC2 vCPU quotas per family, EIP, ENIs per instance, RDS connections, SQS in-flight messages, ALB targets. Quota increases take days, which is why capacity plans must be forward-looking.
- Headroom policy (N+1, "survive the loss of one AZ at peak"), forecasting from business events, and **cost per request** as the metric that ties capacity to FinOps.

**Availability**
- Availability maths: serial versus parallel components, and the effect of an AZ loss on a 3-AZ design with `maxSkew: 1` topology spread. SPOF hunting on your own diagram.
- **Failure domains:** process → pod → node → AZ → region → cloud provider → account. What each of your controls protects against: PDB (voluntary disruption only), topology spread (node/AZ), Multi-AZ RDS (AZ, with a failover time of roughly 60–120 s), cross-region copy (region).
- Graceful degradation, load shedding, admission control and fast 503s versus queueing to death.

**Continuity / DR**
- **RTO and RPO as business decisions** that constrain architecture and cost. Service tiers (for example, Tier 1: RTO 1 h / RPO 15 min, Tier 3: RTO 24 h / RPO 24 h).
- AWS DR strategies: **backup & restore → pilot light → warm standby → multi-site active/active**. The cost and complexity curve.
- Backup versus snapshot versus replication. **Replication is not backup**: it faithfully replicates your `DELETE`. PITR. Immutability (AWS Backup Vault Lock) and ransomware. Cross-account backup copies.
- **The untested backup is a belief.** Restore is the capability, and backup is only its precondition.
- Recovery dependency order: DNS/certs → network → identity → state (Terraform state, secrets) → data → cluster → add-ons → workloads → observability. Failback, the part everyone forgets.
- Where your observability and your Terraform state live during a regional failure (ADR-012's admitted blast radius).

## Architecture decisions in this phase

**ADR-014 DR strategy: backup-restore + pilot light; cross-cloud DR rejected.**

| | |
|---|---|
| **Problem** | Runway needs defined RTO/RPO per tier and a *tested* way to meet them on a learner's budget |
| **Options** | (a) Backup & restore only, same region. (b) **Backup & restore with cross-region copies + pilot light** (network, IAM, ECR replication, Terraform ready in region B; data restored on demand). (c) Warm standby EKS in region B. (d) Multi-region active/active. (e) Cross-cloud DR to Azure |
| **Decision** | (b). Tier-1 (`cargo-api` + RDS): target RTO ≤ 2 h, RPO ≤ 15 min in-region (PITR), RPO ≤ 24 h cross-region (snapshot copy). Measured, then targets adjusted to what you actually achieved |
| **Trade-offs** | Regional RTO is dominated by EKS + add-on bootstrap time (measure it). (c) and (d) are priced and described but not built, honestly labelled. (e) is rejected: data gravity, doubled operational surface, and a cross-cloud data path whose cost and consistency problems outweigh the benefit for this workload. Say this in the interview; it is a senior answer |

Also record (in the capacity plan, no ADR number) the decision on **connection management**: in-app pool sizing versus RDS Proxy versus PgBouncer sidecar, informed by S18 from M11.

## Hands-on labs

1. **Predict, then measure.** Use Little's Law to predict the throughput at which cargo-api's 20-connection pool saturates at 30 ms query time. Run k6 with an open model, ramping arrival rate. Plot p50/p95/p99 and error rate against arrival rate. Compare with your prediction.
2. **Bottleneck iteration ×3.** At the knee, identify the saturating resource using USE on each component. Fix it (pool size, Redis caching of the read-heavy route, worker concurrency, HPA target). Re-run. Record the sequence. **The progression is the lesson.**
3. **Scaling latency.** Drive a step from 20% to 150% of capacity. Time from overload → HPA decision → pods Pending → Karpenter node Ready → pods Ready. Tune it (smaller images, pre-pulled images, `minReplicas`, HPA `behavior`). Decide what headroom that latency requires.
4. **Throttling without an alarm (S19).** Re-enable CPU limits on cargo-api, run load, and watch p99 rise while CPU utilisation looks moderate. Detect it with the throttling ratio panel, not CPU %.
5. **Quota wall.** Lower the Karpenter NodePool `limits` (and/or request an instance family you have zero quota for) during a scale-out. Read what fails where.
6. **AZ failure.** Run 3 replicas with topology spread across 3 AZs, and a Multi-AZ RDS **in a timeboxed window**. Simulate AZ loss: cordon and drain all nodes in one AZ, and trigger RDS reboot with failover. Measure user-visible errors and duration.
7. **Point-in-time restore.** Delete a table's rows, PITR to a new instance 5 minutes earlier, and recover *only those rows* back. Then do a whole-DB restore. Time both.
8. **Full rebuild.** `make down` everything in dev, including RDS (final snapshot), then rebuild from nothing: Terraform, cluster, add-ons, restore, workloads, DNS. Time every stage. **Do it twice.** The difference between the two times is the value of the runbook.
9. **Pilot light in region B (timeboxed).** Cross-region snapshot copy + ECR replication, and Terraform for network/IAM pre-applied in region B. Declare "region A lost", restore into B, and switch Route 53 (health-checked failover record). Measure the RTO. Then **fail back**.
10. **Backup verification job.** A Lambda (`backup-verify`) that weekly restores the latest snapshot to a scratch instance, runs row-count and checksum assertions, publishes a metric and tears down. Alert if it fails or does not run.

## Project implementation

- `platform/loadtest/`: k6 scripts for three paths (read-heavy tracking, write shipment + enqueue, mixed), open model, thresholds as pass/fail. The optional `loadtest` workflow runs a short k6 smoke with thresholds as a **performance gate** in the pipeline.
- `docs/capacity/`: at least three load test reports (environment, traffic model, rates, percentiles, saturating resource, change made) and **`capacity-plan.md`**. It covers: measured ceiling per path and the limiting resource, the scaling latency, headroom policy, quotas that would need raising and their lead time, the forecast trigger ("at X req/s we need Y"), and cost per 1k requests at 30/60/90% utilisation.
- HPA targets, resource requests and pool sizes changed **to the measured values**, with the evidence linked in commit messages.
- Topology spread + PDBs everywhere in `runway-service` defaults. Multi-AZ RDS as a flag, on in prod windows only (cost noted).
- AWS Backup plan (cross-account to `security` or a dedicated backup vault, cross-region copy), `backup-verify` Lambda, Route 53 health checks and a failover record for `cargo.<domain>` (pilot-light pattern).
- `docs/continuity/`: `service-tiers.md` (RTO/RPO per service), `disaster-recovery.md` (dependency-ordered runbook with **measured** timings from two runs), `database-restore.md`, `region-failover.md` (with what was simulated versus executed, labelled honestly), and a BIA-lite (what the business loses per hour of outage, even if invented for a cargo tracker; the method is what counts).
- README numbers: max sustained rate + limiting resource, RTO/RPO measured, AZ-loss impact.

## Engineering problems to hit deliberately

| Problem | How to cause it | What you should learn |
|---|---|---|
| Scaling out makes it worse | Add pods when the DB is the bottleneck | Find the saturating resource before scaling |
| Throttling-induced latency | CPU limits under load | CPU % hides throttling |
| PDB blocks a drain forever | `minAvailable` = replicas | PDBs are a contract with the platform, so size them |
| Restore works, app doesn't | Secrets/users not in the snapshot, or DNS points to the old endpoint | Data restore ≠ service restore |
| Failover DNS never flips | Health check on the wrong path/port | Test the failover mechanism, not just the standby |
| Backup "succeeded" but is unusable | Restore to the wrong engine version / missing KMS grant | Why verification jobs exist |

## Troubleshooting exercises

Full write-up (Detection → Investigation → Mitigation → Root Cause → Permanent Fix → Prevention) for each:

- **S19, "Latency high, CPU looks fine".** Injected blind by re-adding a CPU limit through a "harmless" chart change.
- **S21, "Brief outage when a node disappears".** Karpenter spot interruption plus a workload whose replicas all sat on one node (topology spread was overridden). The trap is blaming spot. The cause is placement.
- **S22, "Customer data missing".** Someone ran a bad `DELETE`. Mitigate (stop writes? feature flag?), then recover rows via PITR side-restore. Record the RPO actually achieved.
- **S23, "Environment/region unavailable".** Planned drill of lab 9, executed as a Sev1 with comms cadence, timeline and a PIR on the runbook's defects.
- **Traffic exceeds capacity.** k6 beyond the knee as a surprise game day. Decide whether to shed load or scale out, and justify it with the capacity plan.
- **Problem record:** "Our recovery depends on components that live in the failure domain" (observability, state bucket, CI runners). This is PRB-00x with the permanent-fix options and their costs.

## Deliverables

- [ ] k6 scripts + ≥3 load test reports showing the bottleneck progression
- [ ] `docs/capacity/capacity-plan.md` with measured ceiling, scaling latency, quotas, cost per 1k requests
- [ ] Resource/HPA/pool settings derived from measurements
- [ ] AWS Backup plan, `backup-verify` Lambda + alert, Route 53 failover record
- [ ] `docs/continuity/*` with **two** timed executions of the DR runbook
- [ ] ADR-014
- [ ] Incident records S19, S21, S22, S23; one PRB

## Completion criteria

- You state the maximum sustainable request rate for each path, the resource that limits it, and the next bottleneck after that. You show the reports.
- The capacity plan predicts a scale event (for example "2× traffic") and a test confirms the prediction within ±20%.
- Full dev rebuild with data restore: **second run at least 30% faster than the first**, both timings in the runbook.
- In-region RPO measured ≤ 15 min (PITR) and the regional RTO measured (whatever it is), with the dominant stage named.
- A simulated AZ loss produces a user-visible error rate you have measured and can explain.
- `backup-verify` has run successfully at least 3 times and fired its alert once when you broke it.
- Someone else (or you, literally following it a month later) executes `database-restore.md` without asking a question.

## Mentor checkpoints

- **Before you build:** your Little's Law prediction and your RTO/RPO tiers with the cost of each. I will ask you to justify the tier-1 targets in business terms.
- **During:** load test report #1 and #3 side by side. What moved, and why.
- **Questions I will ask:** Why is replication not a backup? What is your RTO dominated by, and what would it cost to halve it? What does a PDB protect against, and what does it not? Where do your Terraform state and your dashboards live during a regional outage?
- **The failure I will inject:** during your DR drill, a dependency your runbook assumes is available will not be.

## Resources

1. **AWS Well-Architected, Reliability Pillar:** *Foundations (service quotas)*, *Workload architecture*, *Failure management → Plan for disaster recovery* and the whitepaper *Disaster Recovery of Workloads on AWS* (the four strategies).
2. **Grafana k6 docs:** *Using k6 → Scenarios → Executors* (arrival-rate executors) and *Thresholds*.
3. **Google *SRE* book:** ch. 21 *Handling Overload*, ch. 22 *Addressing Cascading Failures*, ch. 26 *Data Integrity*.
4. ***Designing Data-Intensive Applications* (Kleppmann):** ch. 5 *Replication* (why replication ≠ backup, failover pitfalls) and ch. 8 selectively.
5. **Amazon RDS User Guide:** *Backing up and restoring*, *Multi-AZ deployments*; **AWS Backup:** *Cross-Region and cross-account backup*.

## Estimate vs actual

| | Hours | Cost |
|---|---|---|
| Planned | 40 | ~$90–150 (Multi-AZ window, region-B pilot light window, load tests) |
| Actual | | |
| Variance note | | |
