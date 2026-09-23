# Milestone M16 — FinOps

*Phase 16 · ~3 weeks · Months ~17–18 · Prerequisites: [M15](M15-security-governance.md) (tag policy enforced), [M10](M10-observability.md) (metrics you can join cost to) · Unlocks: [M17](M17-golden-paths.md) (cost as a property of the golden path), [M19](M19-ai-infrastructure.md) (token economics)*

## Why this phase exists

The JD puts cost in the team's mission statement, not in a footnote: *"With a strong emphasis on automation, agility, and cost optimization"*. It also lists FinOps next to security, governance and resilience: *"Strong knowledge of cloud concepts and best practices, including: Security, Governance, Resilience, FinOps"*. And it asks you to guide project managers on delivering *"according to agreed costs and timescales"*.

You have been paying real bills in two clouds since M04, and the budget alerts have been live the whole time. This phase turns that habit into a discipline. It comes after M15 because cost allocation only works when tags are **enforced**, and until M15 they were only requested. It comes before the golden path so that cost visibility is built into the platform from the start rather than added to it later.

## Concepts to learn

**The FinOps lifecycle.** The FinOps Foundation describes three phases: *Inform* (visibility and allocation), *Optimise* (rates and usage) and *Operate* (continuous governance). The central idea is that engineers make the spending decisions, so engineers need the cost data, promptly and attributed to what they own.

**How the bill is actually built.** For each service class, learn what you are charged for and when the charge starts:
- **Compute:** per second or per hour, whether or not anything is running on it.
- **Managed control planes:** the EKS control plane is charged hourly; AKS offers a free tier.
- **Storage:** per GB-month, plus requests.
- **Data transfer:** usually free inbound. Outbound is charged, and so is cross-AZ traffic. Data transferred between clouds is the most expensive transfer you have.
- **NAT Gateway:** charged per hour **and per GB processed**. This is the classic surprise bill, and pulling ECR images through NAT instead of VPC endpoints makes it worse.
- **Load balancers:** per hour plus capacity units.
- **Log ingestion:** per GB ingested, the second classic surprise.
- **AI tokens:** per request, which means traffic and spend become the same curve.

**Allocation.**
- Your tagging taxonomy is the foundation: `service, environment, owner, cost-centre, data-classification, managed-by`.
- Untagged spend is a governance defect, not a rounding error.
- Shared costs (the EKS control plane, the observability stack, NAT, the platform node pool) must be split across teams by an explicit rule: proportional to usage, even split, or kept centrally as platform overhead. Write the rule down, because allocation is a policy decision, not a calculation.
- Showback (you show each team its cost) and chargeback (you bill it) are different things.

**Kubernetes cost.** A node's cost belongs to the pods scheduled on it, measured by their requests, not their usage. Idle capacity (requested but unused) and unallocated capacity (not requested at all) are two different kinds of waste, owned by different people. OpenCost calculates the allocation. Karpenter consolidation and bin-packing are cost levers. Spot capacity trades reliability for price.

**Multi-cloud normalisation.** AWS and Azure use different billing schemas. **FOCUS**, the FinOps Open Cost and Usage Specification, is the common schema both clouds can export. Learn its core columns (BilledCost, EffectiveCost, ServiceCategory, ResourceId, Tags, ChargePeriod) and why EffectiveCost, which spreads commitments over usage, differs from BilledCost.

**Optimisation levers, in the order to try them:**
1. Delete things nobody uses (idle resources, orphaned disks and IPs, old snapshots).
2. Schedule non-production to be off outside working hours.
3. Rightsize, using utilisation data from M10.
4. Move to a better architecture: VPC endpoints instead of NAT, Graviton/ARM, storage lifecycle tiers, log sampling and retention.
5. Buy commitments: Savings Plans, Reserved Instances, Azure reservations. These apply only to a steady baseline you understand, and they are almost never right for a lab. Know how to size them anyway, because enterprises always ask.

**Unit economics.** The useful number is cost per unit of business value: cost per 1,000 `cargo-api` requests, cost per shipment tracked, cost per environment-day and, later, cost per AI summary. A rising bill with falling unit cost is healthy growth. A flat bill with rising unit cost is a problem.

**Autoscaling economics.** Scaling out costs money immediately, while scaling in is delayed by stabilisation windows and consolidation. HPA `maxReplicas` works as a spending limit as well as a capacity setting. A bad scale-out policy can double the bill without the SLO moving at all.

**Anomaly detection.** Cumulative budget alerts are *lagging* indicators. Alerts on the rate of change of daily spend are *leading* indicators. AWS Cost Anomaly Detection and Azure cost alerts do not replace your own alert on daily-spend derivatives.

**Forecasting for project managers.** Estimate the cost of a design before you build it (pricing calculators plus your own measured unit costs), then compare with actuals. The JD's "agreed costs and timescales" is exactly this estimate-versus-actual discipline, which your `docs/estimates/` has been practising since M00.

## Architecture decisions in this phase

**ADR-020 — Cost allocation model: tags, FOCUS, OpenCost, showback**

| | |
|---|---|
| **Problem** | Spend is in two clouds, in two different schemas. Part of it is shared (clusters, NAT, the observability stack) and part of it sits inside Kubernetes, where a node's bill does not say which team caused it. Teams cannot see or own their cost. |
| **Options** | (a) Native consoles only: Cost Explorer and Azure Cost Management, viewed separately. (b) A commercial multi-cloud tool (Kubecost enterprise, Vantage, CloudHealth). (c) FOCUS exports from both clouds, normalised by `finops-collector`, combined with OpenCost for Kubernetes allocation and your own showback view. |
| **Decision** | (c). FOCUS exports go to S3 and Blob. `finops-collector` (a CronJob on EKS that reads Azure through federated identity) normalises them into one dataset. OpenCost splits cluster cost by namespace and label. Shared platform cost is split proportionally to requested CPU and memory, with the observability stack held as a named platform-overhead line. The result is published as monthly showback per team in Grafana now, and in the portal in M18. |
| **Trade-offs** | You maintain a small pipeline and own its correctness, and you will find bugs in your own allocation maths. In return: one schema, no vendor lock-in, and you understand every number. An enterprise would probably buy (b) and still need someone who understands (c) to trust it. |

Also revisit **ADR-004** (network plan). Record the NAT decision: one NAT Gateway in dev versus one per AZ in prod, plus VPC endpoints for S3, ECR and STS. Include the measured cost of each option.

## Hands-on labs

1. **Price before building.** Use the AWS and Azure pricing calculators to model one month of Runway as it exists now. Put the estimate in `docs/estimates/M16.md`. At the end of the phase compare it with the actual bill and explain the gap line by line. *Proves: you can forecast.*
2. **Enable FOCUS exports** in both clouds: AWS Data Exports in FOCUS format to S3, and an Azure Cost Management export to Blob. Open one row from each and map the columns by hand. *Proves: you understand the schema before automating it.*
3. **Tag audit.** Query both clouds for untagged or mis-tagged spend. Write the percentage down. Fix the sources, which will be resources created outside the modules. *Proves: allocation quality is measurable.*
4. **Install OpenCost** on EKS and read cost by namespace. Change one workload's CPU request from 500m to 100m without changing its usage, and watch its allocated cost move. *Proves: requests drive cost, usage does not.*
5. **NAT experiment.** In dev, pull a large image 20 times through NAT, then add the ECR and S3 VPC endpoints and repeat. Compare NAT bytes processed using CloudWatch metrics. *Proves: architecture is a cost lever.*
6. **Schedule dev off.** A Lambda plus EventBridge schedule scales dev node groups to zero and stops RDS at 22:00. A Karpenter NodePool disruption budget handles weekends. Measure the saving over one week. *Proves: the cheapest resource is one that is off.*
7. **Commitment sizing (paper exercise).** From 30 days of usage, calculate what a Compute Savings Plan would cover and save, and why you are not buying one. *Proves: you can have the enterprise conversation.*
8. **Daily-spend derivative alert.** Build a Grafana alert on day-over-day spend change above 30%, alongside the cumulative budget. *Proves: you know leading from lagging.*

## Project implementation

- `workloads/finops-collector/`: a Python CronJob on EKS.
  - Reads AWS FOCUS data from S3 using Pod Identity.
  - Reads Azure FOCUS data from Blob using an Entra federated credential that trusts the EKS OIDC issuer. This is the ADR-017 pattern, reused.
  - Normalises both into Parquet in S3 and publishes daily per-service metrics (`runway_cost_daily{service,environment,cloud,cost_centre}`) to AMP.
- OpenCost installed through GitOps (`platform/gitops/`), with Prometheus as its source.
- `platform/observability/dashboards/finops.json`:
  - Spend by cloud, service, environment and team.
  - Unallocated and untagged spend.
  - Idle versus requested versus used capacity.
  - Unit cost (cost per 1k requests, joined with the M10 request-rate metrics).
  - NAT bytes processed.
  - Log ingestion GB per day.
- Alerts:
  - Budgets at 50/80/100% in both clouds (already exist, now reviewed).
  - AWS Cost Anomaly Detection monitor per linked account.
  - Your own derivative alert.
  - An untagged-spend alert above 5%.
  - Tier `L2`, routed through `ems-bridge`.
- Lambda `idle-detector`: runs nightly and flags these as issues labelled `finops`:
  - Unattached EBS volumes and unused Elastic IPs.
  - Load balancers with zero requests over 7 days.
  - Snapshots older than the retention policy.
  - Node pools scaled above baseline for more than 12 hours.
- `docs/cost/allocation-policy.md`: the shared-cost rule, the showback format, and the owner of each unallocated line.
- `docs/cost/unit-economics.md`: current unit costs with the queries that produce them.
- The Infracost PR comment from M05 now also shows the monthly delta against your `docs/cost/budget.md`.

## Engineering problems to hit deliberately

| Problem | How to cause it | What you should learn |
|---|---|---|
| NAT data-processing bill | Remove the ECR/S3 endpoints; run a node-churn day with Karpenter | Why cost incidents hide in per-GB line items, and how to find them in CUR/FOCUS by usage type |
| Log explosion | Set one service to DEBUG in prod over a weekend | Ingestion is billed per GB; retention and sampling are cost controls |
| Orphaned resources | `terraform state rm` a disk and destroy the rest; delete a cluster without deleting its load balancer | Resources outside state are invisible to IaC and visible to the bill |
| Allocation that lies | Give a workload huge requests and tiny usage | Showback charges requests; teams game or complain — both are signals |
| Scale-out with no SLO benefit | Raise HPA `maxReplicas` and a low CPU target; run k6 | `maxReplicas` is a spending limit; tie scaling to SLO, not CPU vanity |
| Cross-AZ traffic surprise | Pin a chatty client and server to different AZs | Topology-aware routing is a cost lever as well as a latency one |

## Troubleshooting exercises

Write each as a full incident: **Detection → Investigation → Mitigation → Root Cause → Permanent Fix → Prevention**, with a timeline written during the work.

1. **S29 — "The bill jumped 40%."** On day 12 your derivative alert fires. Before looking, have a helper, or `inject-fault.sh --cost`, plant one of: NAT data processing, a scaled-up Karpenter NodePool, DEBUG logging, or a forgotten load test. Attribute the extra spend to a resource, a change (Git, Argo CD history, the change record) and a decision. Reporting only the total is failing the exercise.
2. **"Untagged spend is 18% this month."** Find where it comes from. You will probably discover resources created by controllers (load balancers from the LB Controller, disks from CSI) that did not inherit tags. The permanent fix belongs in the module or controller configuration, not in a clean-up script.
3. **"Team X disputes its showback."** Their dashboard says they cost twice what they believe. Investigate: requests versus usage, and shared-cost allocation. Decide whether the model or the team is wrong, and write the answer in `docs/cost/allocation-policy.md`.
4. **Problem record.** Link S29 to earlier cost-related incidents (for example the M10 log-ingestion incident, S17) in a `PRB-` record: *cost changes can merge without cost visibility*. The permanent fix is the Infracost gate plus budget-delta review in the change process (ADR-013).

## Deliverables

- [ ] `docs/adr/ADR-020-cost-allocation-model.md`
- [ ] `workloads/finops-collector/` with tests, deployed through GitOps
- [ ] OpenCost running; `platform/observability/dashboards/finops.json`
- [ ] Derivative, anomaly, untagged-spend and budget alerts, each with a runbook in `docs/runbooks/cost-*.md`
- [ ] Lambda `idle-detector` and a dev off-hours schedule
- [ ] `docs/cost/allocation-policy.md`, `docs/cost/unit-economics.md`, `docs/cost/monthly-notes.md` (continued)
- [ ] `docs/incidents/INC-0xx-cost-anomaly.md` (S29) and one `PRB-` record
- [ ] `docs/estimates/M16.md` with calculator estimate vs actual and an explanation of the gap
- [ ] ADR-004 amended with the measured NAT/VPC-endpoint decision

## Completion criteria

- Given any day's spend, you can attribute **≥95%** of it to a service and owner in under 10 minutes.
- Untagged or unallocated spend is **below 5%**, and an alert fires if it rises above that.
- You can state, from the dashboard: monthly cost per environment, the dominant line item in each cloud, cost per 1k `cargo-api` requests, and the observability stack's share.
- You attributed the planted S29 anomaly to a resource, a change and a decision, and your derivative alert detected it **at least 3 days before** the 80% budget alert would have.
- The dev off-hours schedule shows a measured weekly saving, with the number written down.
- You can explain, without notes: requests-based allocation, BilledCost versus EffectiveCost, why a Savings Plan is wrong for this lab, and the NAT-versus-endpoint trade-off, including its cost.

## Mentor checkpoints

- **Before building:** bring your allocation policy draft. I will ask how you split the observability stack and why, what happens to a team whose namespace is empty for a month, and who owns unallocated cost.
- **During:** bring one day of FOCUS data from each cloud. We will reconcile them by hand before you trust `finops-collector`.
- **Review questions:** "Your bill went up and your unit cost went down. Is that good?" "What would you tell a PM who asks what a new service will cost per month?" "Which of your savings would survive a production SLO?"
- **The failure I will inject after you think you are done:** a cost increase that none of your alerts will catch in its current form. The category is a slow, steady increase in a per-GB line item.

## Resources

- **FinOps Foundation Framework** (finops.org/framework): read *Principles*, *Phases*, and the *Capabilities* for Allocation, Anomaly Management, Rate Optimisation and Unit Economics.
- **FOCUS specification** (focus.finops.org): the column definitions for BilledCost, EffectiveCost, ServiceCategory and Tags, plus the AWS and Azure FOCUS export docs.
- ***Cloud FinOps*, 2nd ed. (Storment & Fuller):** Part II (Inform: allocation, tags, showback), the chapters on rate optimisation and commitments, and the unit-economics chapter.
- **AWS Well-Architected Framework: Cost Optimization pillar**, specifically "Expenditure and usage awareness" and "Cost-effective resources". Also the Azure WAF Cost Optimization design principles.
- **OpenCost documentation:** the specification ("How cost is allocated") and installation with an existing Prometheus.

## Estimate vs actual

| | Planned | Actual | Variance note |
|---|---|---|---|
| Hours | 30 | | |
| Cloud cost this phase | $60–90 | | |
