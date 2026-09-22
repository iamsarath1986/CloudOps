# Step 4 — Primary Cloud Selection

## 1. Does the JD require a specific cloud?

**No.** The posting is deliberately provider-neutral. Every relevant phrase is generic:

> "managing and supporting **public cloud** environments" · "cloud services such as compute, storage, networking, serverless and container services" · "**Associate-level cloud**, DevOps or developer certification"

No AWS, Azure or GCP service is named anywhere. Neither is any cloud-specific tool. The only named technologies are cross-cloud: Terraform, Docker, Kubernetes, Git, Python, Bash, PowerShell, TypeScript, React, Node.

This is good news and a trap. Good news, because you are free to pick one and go deep. A trap, because "provider-neutral JD" tempts people into shallow multi-cloud. Do not. **One cloud, to production depth, beats three clouds at tutorial depth — and the interview panel will find the difference in four minutes.**

## 2. Recommendation: **Microsoft Azure**

Seven reasons, weighted by how much they should actually move you.

| # | Reason | Weight |
|---|---|:--:|
| 1 | **The JD describes a hybrid estate** — "unify control across datacentres, networks, public **and private** cloud environments." Azure's hybrid story (Arc, Stack, hybrid identity) is the most developed of the three and the one enterprises with an existing datacentre actually adopt. This is the single strongest signal in the posting. | ●●●●● |
| 2 | **PowerShell is named** alongside Bash. Nobody lists PowerShell in a pure-AWS shop. Combined with an airline's near-certain Active Directory / Microsoft 365 estate, this reads as a Microsoft-centric enterprise. | ●●●● |
| 3 | **Large regulated enterprises in EMEA skew Azure**, driven by existing Microsoft licensing, Entra ID as the identity source of truth, and enterprise agreements. Airlines, banks and telcos are the archetype. | ●●●● |
| 4 | **The UAE has strong Azure region presence** (UAE North / UAE Central), which matters for data residency conversations you may be asked about in interview. | ●●● |
| 5 | **Azure's AI story is the most enterprise-integrated** — Azure OpenAI / AI Foundry behind private endpoints, Entra ID auth, and Azure Monitor telemetry. That maps exactly onto the JD's "AI/GenAI, AI agents or MCP" desirable, in a way that is about *integration and governance* rather than modelling. | ●●● |
| 6 | **AKS, Terraform's AzureRM provider, Azure Policy and Azure Monitor are all first-class**, so nothing in this roadmap is compromised by the choice. | ●●● |
| 7 | **Certification ladder fits the JD's wording precisely** — AZ-104 is literally an "Associate-level cloud" certification, AZ-400 a "DevOps" one, AZ-204 a "developer" one. | ●● |

### The honest counter-arguments

- **AWS has more market share and more job postings globally.** True. If your goal were maximum job-count optionality, AWS wins. It is not — your goal is this class of role in this region, and for a hybrid enterprise platform team, Azure is the better-aimed bet.
- **GCP has the nicest Kubernetes experience.** Also true, and irrelevant: GKE quality does not transfer to an employer who is not on GCP.
- **You might end up somewhere that runs AWS.** Likely, eventually. Which is why Milestone 13 exists: a deliberate, two-week port of one Terraform module and one workload to AWS. The point is not to learn AWS; it is to prove to yourself and to an interviewer that your knowledge is conceptual rather than console-shaped.

**Decision: Azure primary. AWS as a late, explicitly-scoped translation exercise. GCP: conceptual awareness only.** Write this as `ADR-001` in your repository on day one — it is your first architecture decision record, and being able to defend a *rejected* option is worth more in interview than the choice itself.

## 3. Learn the concept first, then the product name

Every phase in this roadmap introduces the vendor-neutral idea before the Azure service. This is not pedantry — it is what makes you portable, and it is how you avoid the most common failure mode of cloud learners: knowing where a button is but not what it does.

| The concept (learn this) | Azure | AWS | GCP |
|---|---|---|---|
| Account/billing boundary | Subscription | Account | Project |
| Org hierarchy & policy scope | Management Group | Organization / OU | Folder |
| Grouping & lifecycle unit | Resource Group | (tags / CloudFormation stack) | (labels) |
| Private network | VNet | VPC | VPC |
| Subnet firewall | NSG | Security Group + NACL | Firewall rules |
| Custom routing | Route Table / UDR | Route Table | Routes |
| Private service access | Private Endpoint / Private Link | VPC Endpoint / PrivateLink | Private Service Connect |
| L7 load balancer + WAF | Application Gateway + WAF | ALB + WAF | HTTPS LB + Cloud Armor |
| L4 load balancer | Azure Load Balancer | NLB | TCP/UDP LB |
| Managed DNS | Azure DNS / Private DNS Zone | Route 53 | Cloud DNS |
| VM / scale set | VM / VMSS | EC2 / ASG | Compute Engine / MIG |
| Managed Kubernetes | AKS | EKS | GKE |
| Container registry | ACR | ECR | Artifact Registry |
| Serverless container | Container Apps | App Runner / Fargate | Cloud Run |
| Function-as-a-service | Azure Functions | Lambda | Cloud Functions |
| Object storage | Blob Storage | S3 | Cloud Storage |
| Managed PostgreSQL | PostgreSQL Flexible Server | RDS / Aurora | Cloud SQL |
| Secret store | Key Vault | Secrets Manager / Parameter Store | Secret Manager |
| Workload identity (no secrets) | Managed Identity / Workload Identity | IAM Role for Service Account | Workload Identity Federation |
| Identity provider | Entra ID | IAM + Identity Center | Cloud Identity |
| Authorization model | Azure RBAC | IAM policies | Cloud IAM |
| Policy-as-code guardrails | Azure Policy | SCP + Config | Organization Policy |
| Metrics & logs platform | Azure Monitor / Log Analytics | CloudWatch | Cloud Monitoring/Logging |
| Managed Prometheus/Grafana | Azure Monitor Managed Prometheus + Managed Grafana | AMP + AMG | Managed Prometheus |
| Cost management | Cost Management + Budgets | Cost Explorer + Budgets | Billing + Budgets |
| Hybrid/on-prem projection | Azure Arc | Systems Manager / EKS Anywhere | Anthos |

Print this table. When you learn "private endpoint", learn the *row*, not the cell. That habit is the entire difference between six months of Azure experience and six months of transferable cloud experience.

## 4. Cost reality for a self-funded lab — your budget: **$30–60/month**

This constraint is not a compromise. It is the FinOps curriculum, applied to your own money, which is the only way anyone actually learns cost discipline.

**Rules of the lab:**

1. **Terraform apply at the start of a session, `terraform destroy` at the end.** Everything stateful that must survive lives in Blob Storage or a dump file, not in a running resource. Make this a single `make up` / `make down` from Milestone 4 onward; if tearing down is inconvenient, you will not do it.
2. **A budget with an action group that emails you at 50%, 80% and 100%** exists before the second resource is created. Milestone 1.
3. **Expensive components run in deliberate, timeboxed windows**, not continuously:

| Component | Approx. cost if always on | How this lab uses it |
|---|---|---|
| Azure Firewall | ~$900+/mo | **Never continuously.** One 2-day exercise, or replaced entirely by NSG + UDR reasoning. Not required by the JD. |
| Application Gateway v2 + WAF | ~$180–250/mo | Two timeboxed windows (Milestone 6 and Milestone 12 hardening), destroyed same day. `ingress-nginx` + a standard LB is the daily driver. |
| AKS nodes (2 × B2als_v2) | ~$60/mo | Destroyed between sessions. Free control-plane tier. Spot node pool for the non-system pool. |
| PostgreSQL Flexible Server B1ms | ~$15–25/mo | **Stop** the server between sessions (supported for up to 7 days), or destroy and restore from dump. |
| Log Analytics | Per-GB ingest | Daily ingestion cap set from day one. Blowing through it is a deliberate Milestone 10 exercise. |
| ACR Basic | ~$5/mo | Always on. Cheap and stateful. |
| Key Vault, Blob, Azure DNS, Budgets, Policy, Arc | < $5/mo combined | Always on. |
| Azure OpenAI (small model, low volume) | $2–10/mo | Token budget + alert before the first call. |

4. **Kubernetes *learning* happens on local `kind`/`k3d`, for free.** Kubernetes *operating* happens on AKS. Do not burn budget watching a pod start.
5. **The first 30 days include a free credit.** Spend it on Milestone 2's deliberately manual build, when destroy-discipline has not yet formed.
6. **Every month, write one line in `docs/cost/monthly-notes.md`**: what you spent, on what, and what surprised you. That file becomes portfolio evidence for the JD's "cost awareness" requirement.

**Expected steady state: $30–45/month** with discipline, spiking to $60–80 in the two Application Gateway months and the Arc/hybrid month. If you hit $100, you have a real incident — treat it as one, and write it up. That write-up is worth more in an interview than any dashboard screenshot.

→ Next: [Step 5 — The project](04-the-project.md)
