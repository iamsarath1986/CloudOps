# Section 19 — FinOps Learning Path

*Cost is an engineering signal, not an accounting afterthought. On this roadmap you learn FinOps on your own money across two clouds — which is the only way anyone genuinely learns it.*

---

## 1. What the JD actually asks for

> *"With a strong emphasis on automation, agility, and cost optimization..."*
> *"Strong knowledge of cloud concepts and best practices, including: Security · Governance · Resilience · FinOps"*
> *"...drive the adoption of ... governance, and FinOps practices..."*
> *"Guide Project Managers ... These are to be delivered according to agreed costs and timescales..."*

"FinOps" is named explicitly, twice, alongside security and governance — not as "cost awareness" but as a *practice to drive*. And the senior responsibility of guiding project managers on cost means you must **estimate the cost of a design before it is built** and explain the variance afterwards.

**Depth expected:** Production Proficiency on allocation, visibility, anomaly investigation and optimisation; Working Knowledge on commitment strategy; Advanced on cost-aware architecture (unit economics, design-time estimation).

## 2. The FinOps cycle, applied to Runway

| Phase | Question | What you do in Runway |
|---|---|---|
| **Inform** | Who spent what, on what, why? | Tags enforced by policy; FOCUS exports from both clouds; OpenCost for Kubernetes; `finops-collector`; showback per team in the portal |
| **Optimise** | Where is spend not producing value? | Rightsizing, idle detection, storage lifecycle, NAT/egress reduction, autoscaling tuning, caching AI responses |
| **Operate** | How do we keep it that way? | Budgets and anomaly alerts, cost gates in PRs (Infracost), monthly review, estimates vs actuals per milestone |

## 3. The progression

| Stage | Theme | Milestones |
|:-:|---|---|
| 1 | Budgets before resources; estimate before build | [M03](milestones/M03-cloud-architecture.md), [M04](milestones/M04-aws-core.md) |
| 2 | Tagging taxonomy and cost in code review | [M05](milestones/M05-terraform.md), [M07](milestones/M07-cicd.md) |
| 3 | Container and Kubernetes cost | [M09](milestones/M09-eks.md), [M12](milestones/M12-capacity-ha-dr.md) |
| 4 | Observability cost | [M10](milestones/M10-observability.md) |
| 5 | Second cloud, one view | [M13](milestones/M13-azure-core.md), [M14](milestones/M14-aks-multicluster.md) |
| 6 | The FinOps practice | [M16](milestones/M16-finops.md) |
| 7 | Cost as a platform feature | [M17](milestones/M17-golden-paths.md), [M18](milestones/M18-developer-platform.md) |
| 8 | AI inference economics | [M19](milestones/M19-ai-infrastructure.md), [M20](milestones/M20-ai-agents-mcp.md) |
| 9 | Hybrid and final cost model | [M21](milestones/M21-hybrid-multicloud.md), [M22](milestones/M22-hardening-capstone.md) |

### Stage 1 — Budgets and estimates · M03–M04
- The pricing model of each service class: per-hour (EC2, EKS control plane, NAT Gateway, ALB), per-request (Lambda, SQS), per-GB stored (S3, EBS snapshots), per-GB processed/transferred (NAT, cross-AZ, internet egress), per-GB ingested (logs), per-token (AI).
- **Create the budget and alert before the second resource exists** — in both clouds, when you reach each.
- Model the cost of your first Solution Design Document with the AWS Pricing Calculator; compare to the bill at month end. The variance note is the lesson — and the start of your "estimate vs actual" habit that every milestone ends with.

### Stage 2 — Tags and cost in review · M05, M07
- Taxonomy: `service`, `environment`, `owner`, `cost-centre`, `data-classification`, `managed-by`. Applied by a Terraform tagging module; later enforced by policy (M15). AWS cost allocation tags must be *activated* — a classic "why is my report empty?".
- **Infracost** comments on Terraform PRs: the cost delta of a change is visible before merge.
- `make down` as a cost control; the scheduled `idle-detector` Lambda begins here in its simplest form.

### Stage 3 — Kubernetes cost · M09, M12
- Cluster cost = control plane + nodes + load balancers + storage + data transfer + observability. The pod does not appear on the bill; the node does.
- **Requests drive cost**, not usage: over-requested pods force more nodes. Rightsizing requests from measured usage is the biggest lever.
- Karpenter consolidation, spot capacity and interruption handling; bin-packing; the economics of HPA (scale-out cost vs latency SLO) and of headroom policy.
- Cross-AZ traffic charges and topology-aware routing; the VPC CNI and IP address constraints.
- **NAT Gateway**: hourly + per-GB processed. Pulling images through NAT instead of a VPC endpoint is a real, common surprise bill.

### Stage 4 — Observability cost · M10
Metrics priced per sample ingested (AMP), logs per GB ingested and retained (CloudWatch, Log Analytics), traces per span stored. Cardinality control, sampling, retention tiers, and dropping at the Collector. The observability bill can exceed the workload bill — meet that on a small scale deliberately.

### Stage 5 — Two clouds, one view · M13–M14
Azure's cost model differs: resource-group scoping, Azure Budgets with action groups, Azure Advisor, Cost Management exports. **FOCUS** (FinOps Open Cost and Usage Specification) exports from both AWS (Data Exports) and Azure (Cost Management) give you one schema; `finops-collector` (CronJob on EKS, reading Azure via federated identity) normalises them into one dataset.

### Stage 6 — The practice · M16
The consolidation milestone: allocation model (ADR-020), showback per team, unit economics, anomaly detection (AWS Cost Anomaly Detection + your own rate-of-change alert), rightsizing and idle programmes, commitment analysis, and the four bill investigations in §5.

### Stage 7 — Cost as a platform feature · M17–M18
Every golden-path workload is tagged, allocated and shown back by construction. The portal shows cost per service this month, trend, and cost per 1k requests. Sandbox accounts are vended with a budget and an auto-expiry. Cost is a property of being on the platform.

### Stage 8 — AI economics · M19–M20
Per-token pricing makes a traffic spike and a bill spike the same event. `ai-gateway` meters tokens per request and attributes them to team and service; per-team token budgets; exact-match response cache and its hit rate as a cost metric; model routing by cost (smaller model first where quality allows); managed vs self-hosted break-even (ADR-024); provisioned throughput vs pay-as-you-go at the concept level; flightdeck's per-session token budget.

### Stage 9 — Hybrid and final model · M21–M22
Datacentre costs are real but hidden (hardware amortisation, power, people); VPN gateway hourly charges; hybrid DNS resolver endpoint charges. Final deliverable: a cost model for Runway at 10× and 100× load with the dominant line items and the first three optimisations you would make.

## 4. Optimisation levers, ranked by what usually pays

| Lever | Typical mechanism | Runway exercise |
|---|---|---|
| Delete what is idle | Scheduled detection + owner notification + expiry | `idle-detector` Lambda; unattached EBS, old snapshots, idle load balancers, stopped-but-billed resources |
| Rightsize | Requests/instances from measured p95 usage | Rightsize every workload's requests from a month of metrics |
| Stop paying for data movement | VPC endpoints, same-AZ routing, CDN for static | Gateway endpoints for S3, interface endpoints for ECR; CloudFront for the SPA |
| Storage lifecycle | Tiering, retention, snapshot policies | S3 lifecycle rules for Loki/Tempo and backups; log retention per tier |
| Autoscaling economics | Scale to demand; consolidation; spot for interruptible work | Karpenter consolidation; spot for workers; HPA tuned against the SLO |
| Commitments | Savings Plans, Reserved Instances, Azure reservations/savings plans | Analysis only — model what you would commit to at steady state and why *not* now |
| Architecture | Serverless for spiky/scheduled, cache, async | Lambda for scheduled jobs; Redis cache hit rate; SQS smoothing AI demand |

## 5. Investigate the unexpectedly high bill — exercises

Each is run as an incident (S29, S32) with the Detection → Investigation → Mitigation → Root Cause → Permanent Fix → Prevention write-up and before/after numbers. **Attribute the cost to a resource, a change and a decision — never just report the total.**

| # | Setup (inject it, then forget about it for a few days) | Detection you must have built | What the investigation should find |
|:-:|---|---|---|
| 1 | Private-subnet workloads pull images and call S3 through the NAT Gateway; run a load test | Daily rate-of-change alert, not only the monthly budget | NAT "bytes processed" dominating; fix with S3 gateway + ECR interface endpoints; quantify the trade-off of interface endpoint hourly cost |
| 2 | Raise HPA `maxReplicas` and leave a k6 test running over a weekend; Karpenter scales out | Cost anomaly alert; node count panel | Two causes, not one: a config change and a forgotten test; permanent fix: nodes-above-baseline alert, load tests with TTLs |
| 3 | Add a high-cardinality label and debug-level logging to one service | Observability cost panel; AMP/CloudWatch ingest | Metrics series explosion and log GB/day; fix at the Collector; set per-service ingest budgets |
| 4 | Detach volumes, leave snapshots and an unused ALB after an ECS→EKS migration | `idle-detector` report | "Migration debris" — the resources nobody owns after a change; fix with ownership tags and post-change clean-up in the change template |
| 5 | Disable the ai-gateway response cache and ship a prompt that inlines full log dumps | Tokens/minute and cost-rate alert per team | Prompt bloat × cache miss; fix with context limits, cache, and per-team token budgets (S32) |
| 6 | Cross-AZ chatter: pin a chatty client and server in different AZs | Data transfer line item | Inter-AZ transfer; topology-aware routing or co-location, with the availability trade-off written down |

## 6. Unit economics

The senior move is to talk about cost **per unit of value**, not per month:
- cost per 1,000 `cargo-api` requests (by environment)
- cost per shipment-exception summary (tokens + queue + compute)
- cost per onboarded service per month (platform overhead ÷ services)
- cost per flightdeck investigation

Put these on the cost dashboard, and use them in solution designs: *"At 10× volume this design costs roughly X per month, dominated by Y; here is the first optimisation."* That sentence is what the JD means by guiding project managers on agreed costs.

## 7. Checkpoints

| By | You can |
|---|---|
| M04 | Show a budget that existed before your second resource, and your first estimate-vs-actual |
| M09 | Explain your cluster's monthly cost by component and why requests, not usage, drive it |
| M16 | Show cost per service per environment across both clouds from one FOCUS-based dataset; complete four bill investigations with before/after numbers |
| M19 | State cost per AI summary and the cache hit rate that keeps it there |
| M22 | Present a 10×/100× cost model with dominant line items |

## 8. Resources

1. ***Cloud FinOps*** 2nd ed (Storment & Fuller) — Part I (ch. 1–5) for the model; ch. on allocation/tagging, rightsizing, and commitment-based discounts.
2. **FinOps Foundation Framework** (finops.org) — "Principles", "Phases", and the "Allocation" and "Anomaly Management" capabilities; the **FOCUS specification** overview and column dictionary (skim).
3. **AWS Well-Architected — Cost Optimization Pillar**, plus AWS docs on "Cost allocation tags" and "Data transfer" pricing concepts; **Azure Well-Architected — Cost Optimization**.
4. **OpenCost docs** — "Cost allocation" specification (how node cost is split to pods).
5. **Karpenter docs** — "Disruption → Consolidation" and "Spot" best practices.

→ Next: [Section 20 — AI Infrastructure Learning Path](20-ai-infrastructure-learning-path.md)
