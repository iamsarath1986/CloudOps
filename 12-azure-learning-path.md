# Section 12 — Azure Learning Path

Azure is your **second** cloud, introduced at [M13](milestones/M13-azure-core.md) (~month 14), after you already understand networking, identity, Terraform, Kubernetes and production operations on AWS. That changes how you learn it: **you are not relearning cloud, you are learning the differences** — and Azure's differences are exactly where an enterprise like the target employer lives: identity, governance, hybrid control and enterprise AI.

> JD evidence: *"Proficiency in managing and operating AWS and **Azure** cloud platforms"* · *"Azure AKS"* · *"unify control across datacentres, networks, public and private cloud environments"* · *"PowerShell"* · *"Microsoft Certified: Azure Solutions Architect Expert"*.

---

## 1. Why Azure second, and what that means

- Azure's centre of gravity is **Entra ID** and the **management-group / Azure Policy** control plane. Those are governance and enterprise-identity topics; they are best learned once you have something worth governing.
- **AKS** is more managed than EKS. Learning Kubernetes on EKS first means you notice what AKS does for you — and what it hides.
- **Azure OpenAI** behind a private endpoint with managed identity is the enterprise AI pattern; it arrives at [M19](milestones/M19-ai-infrastructure.md) with an Azure foundation already in place.
- **Azure Arc** is the most developed hybrid control story and is the direct answer to *"unify control across datacentres … public and private cloud"* ([M21](milestones/M21-hybrid-multicloud.md)).

The rule from ADR-002: **never duplicate an AWS exercise on Azure.** Every Azure lab either teaches a *difference* or builds something that belongs on Azure per ADR-015 (workload placement): `runway-portal` BFF, `ai-gateway`, `flightdeck`, and the Azure half of `hello-svc`.

---

## 2. Learn it by difference

Before M13, fill in this table yourself from the AWS knowledge you have; then use it as the M13 reading list. The full concept mapping is in [Appendix A1](appendix/A1-cloud-architecture-concepts.md).

| Area | On AWS you learned | On Azure the *difference* is | Why it matters |
|---|---|---|---|
| Hierarchy | Organization → OU → account | Tenant → management group → subscription → **resource group** → resource | Resource groups are a lifecycle unit AWS has no direct equivalent for; RBAC and Policy attach at every level and **inherit downward** |
| Identity model | IAM principals live *in* each account; Identity Center federates people in | **Entra ID is the tenant-wide directory**; users, groups, service principals, managed identities all live there; subscriptions *trust* the tenant | Identity is centralised first, then authorised per scope |
| Authorisation | Policy JSON, evaluated per request, explicit deny wins | **Azure RBAC**: role definition + principal + scope; additive; deny assignments rare; data-plane vs control-plane roles differ | "Owner can't read the Key Vault secret" surprises every AWS engineer |
| Guardrails | SCPs (preventive), Config (detective) | **Azure Policy** does both: `Audit`, `Deny`, `Modify`, `DeployIfNotExists`, initiatives, exemptions, compliance reporting | One engine for prevention, detection and remediation |
| Network | VPC is regional, subnets are per-AZ, SGs attach to ENIs | VNet is regional, **subnets span zones**; NSGs attach to subnets or NICs; routing is implicit until you add UDRs | Zone placement is a resource setting, not a subnet choice |
| Private PaaS access | Interface endpoints + private DNS handled mostly for you | **Private endpoints + private DNS zones + VNet links** you manage explicitly | **S24** — the most instructive DNS failure in the roadmap |
| Workload identity | Pod Identity / IRSA → IAM role | **AKS Workload Identity** → user-assigned managed identity + **federated identity credential** | Same OIDC mechanism, different object model |
| Secrets | Secrets Manager | **Key Vault**: secrets, keys and certificates; RBAC mode; soft delete and purge protection | Purge protection decides whether a deletion is recoverable |
| Observability | CloudWatch, Logs Insights | **Azure Monitor**, Log Analytics workspaces, **KQL**, diagnostic settings (nothing is logged until you send it) | Forgetting diagnostic settings = no evidence during an incident |
| Cost | Cost Explorer, Budgets, Anomaly Detection | Cost Management, Budgets, exports (FOCUS), scopes = management group/subscription/RG | Cost scope follows the hierarchy |
| Automation shell | Bash + AWS CLI | **PowerShell Az module** and `az` CLI | The JD names PowerShell; enterprise Azure runbooks are often PowerShell |

---

## 3. The progression

| Stage | Milestones | ≈ Months | You leave with (L-scale) |
|---|---|:--:|---|
| 1 — Tenant, identity, landing zone, network | [M13](milestones/M13-azure-core.md) | 14–15 | L3: Entra ID as workforce IdP for both clouds; hub-spoke VNet; private endpoints working |
| 2 — AKS and multi-cluster | [M14](milestones/M14-aks-multicluster.md) | 15–16 | L3→L4: AKS in Terraform, workload identity, Argo CD managing EKS and AKS |
| 3 — Governance and security | [M15](milestones/M15-security-governance.md) | 16–17 | L4: Azure Policy initiatives audit→deny, Defender for Cloud basics, Key Vault lifecycle |
| 4 — Cost | [M16](milestones/M16-finops.md) | 17–18 | L4: FOCUS exports from both clouds normalised; Azure budgets |
| 5 — Platform and portal | [M17](milestones/M17-golden-paths.md), [M18](milestones/M18-developer-platform.md) | 19–20 | L4: golden path targets AKS; portal on AKS with Entra SSO |
| 6 — AI | [M19](milestones/M19-ai-infrastructure.md), [M20](milestones/M20-ai-agents-mcp.md) | 21–22 | L4: Azure OpenAI with private endpoint, managed identity, quota and cost telemetry |
| 7 — Hybrid | [M21](milestones/M21-hybrid-multicloud.md) | 23 | L3→L4: Arc-enabled servers and Kubernetes under the same policy |

---

## 4. Service areas and what you build

### Entra ID — first, and the most important ([M13](milestones/M13-azure-core.md))

| Topic | What to understand | What you build |
|---|---|---|
| Objects | Users, groups, app registrations vs **service principals** (the app's instance in a tenant), managed identities (system vs user-assigned) | Groups `platform-admins`, `developers`, `readers`, `sre-oncall` |
| Tokens | OAuth2/OIDC flows: auth code + PKCE (SPA), client credentials, on-behalf-of; token audience and scopes | `runway-portal` SSO; BFF validates tokens |
| Workforce federation | SAML + SCIM provisioning into AWS IAM Identity Center | **ADR-016**: Entra ID becomes the identity source for AWS — migrated as a planned change with rollback |
| Workload federation | Federated identity credentials: issuer, subject, audience | **ADR-017**: EKS pods → Entra (trusting the EKS OIDC issuer); GitHub Actions → Entra (**S25** when the subject is wrong) |
| Enterprise controls | Conditional Access, PIM (just-in-time elevation), access reviews, break-glass accounts | Know them by problem solved; enable PIM if your licence allows, otherwise document |

### Landing zone and hierarchy ([M13](milestones/M13-azure-core.md))

Management groups `runway` → `platform` / `landing-zones` / `sandbox`. Realistically two subscriptions — `runway-platform` (hub network, shared services) and `runway-workloads` (dev and prod separated by **resource group**). State this constraint honestly in ADR-015; enterprise subscription vending is taught conceptually from the Cloud Adoption Framework and exercised if your account allows extra subscriptions.

### Networking ([M13](milestones/M13-azure-core.md), [M21](milestones/M21-hybrid-multicloud.md))

- Hub-spoke: hub VNet (`10.30.0.0/18` range from ADR-004) peered to a workload spoke; NSGs on subnets; UDRs only where you can explain the route.
- **Private endpoints** for Key Vault, Azure OpenAI, Blob; **private DNS zones** linked to the VNets. Build it, then break the zone link (S24).
- Azure Standard Load Balancer in front of Envoy Gateway on AKS; NAT Gateway for deterministic egress.
- Timeboxed later: VPN Gateway for the AWS↔Azure tunnel and Azure DNS Private Resolver ([M21](milestones/M21-hybrid-multicloud.md)).

### AKS ([M14](milestones/M14-aks-multicluster.md))

| Topic | The AKS-specific point |
|---|---|
| Networking model | Azure CNI Overlay (pods not on VNet IPs) vs flat Azure CNI — contrast with EKS VPC CNI and the IP-exhaustion problem it avoids |
| Node pools | System vs user pools; spot user pool; node auto-provisioning (Karpenter-based) or cluster autoscaler |
| Identity | Entra ID integration for `kubectl` with Azure RBAC for Kubernetes; Workload Identity for pods |
| Upgrades | Auto-upgrade channels, node image upgrades, planned maintenance windows, surge settings; LTS awareness |
| Delivery | Argo CD from the EKS management plane, or per-cluster — decided in **ADR-018** |
| Traffic entry | Gateway API + Envoy Gateway + cert-manager (portable) vs Application Gateway for Containers (native) — compare with ADR-009's AWS choice |

### Key Vault ([M13](milestones/M13-azure-core.md), [M14](milestones/M14-aks-multicluster.md), [M15](milestones/M15-security-governance.md))

RBAC mode (not access policies), soft delete + **purge protection**, secret versions and expiry, private endpoint only. Delivered to pods by External Secrets Operator (ADR-011 — same pattern as Secrets Manager on EKS). Drill: let a secret expire (S-style exercise in M15) and delete a vault to learn what purge protection saves.

### Azure Policy ([M15](milestones/M15-security-governance.md))

Required tags, allowed regions, deny public IPs on workload subnets, deny public network access on PaaS, require private endpoints, `DeployIfNotExists` for diagnostic settings. **Audit first, then Deny** — S28 is what happens if you skip the audit period. The same initiative later governs Arc-connected machines and clusters ([M21](milestones/M21-hybrid-multicloud.md)).

### Azure Monitor and KQL ([M14](milestones/M14-aks-multicluster.md), [M10](milestones/M10-observability.md)'s stack as the primary)

The central stack (ADR-012) is OpenTelemetry → AMP / Loki / Tempo → Grafana. Azure Monitor is kept for what only Azure sees: AKS control-plane logs, Activity Log, Azure OpenAI metrics, Key Vault audit events. Learn **KQL** properly — ten saved queries you actually reuse — and add Log Analytics as a Grafana data source. Set a daily ingestion cap; diagnostic settings via Policy.

### Azure OpenAI / AI Foundry ([M19](milestones/M19-ai-infrastructure.md))

Deployments and model versions, **TPM/RPM quotas** per region and deployment type, content filtering, managed identity auth (no keys), private endpoint, diagnostic logs. `ai-gateway` calls it as the **primary** route; Bedrock is the cross-cloud fallback (ADR-023).

### Azure Arc ([M21](milestones/M21-hybrid-multicloud.md))

Arc-enabled servers for the "datacentre" VM and Arc-enabled Kubernetes for k3s: inventory, tags, Policy, (optionally) GitOps extension. Understand what Arc projects and what it does not (it does not move workloads; it extends the control plane).

---

## 5. PowerShell

PowerShell is named in the JD. The target is **L3 for reading and modifying, L2–L3 for writing**:

| When | What |
|---|---|
| [M02](milestones/M02-automation.md) | Object pipeline model, `Get-Member`, error handling (`-ErrorAction`, `try/catch`), modules; one small tool |
| [M13](milestones/M13-azure-core.md) | Every Azure landing-zone step done once with the **Az module** (`Connect-AzAccount -Identity`, `Get-AzResource`, `New-AzRoleAssignment`) |
| [M16](milestones/M16-finops.md) | `platform/scripts/RunwayOps` PowerShell module: tag-compliance and orphaned-resource report across subscriptions |
| [M21](milestones/M21-hybrid-multicloud.md) | Arc onboarding scripts for the datacentre VM |

---

## 6. Checkpoints

| After | Checkpoint (without notes) |
|---|---|
| M13 | Sign in to the AWS console via Entra ID; explain the SAML assertion and SCIM flow. Explain why "Owner" on a subscription cannot read a Key Vault secret in RBAC mode. Diagnose S24 from inside a pod in under 10 minutes. |
| M14 | One `workload.yaml` change deploys to AKS through Argo CD; upgrade AKS one minor version with traffic flowing; explain AKS vs EKS networking in one whiteboard diagram. |
| M15 | Show an Azure Policy that denied a real deployment and the exemption process with expiry. |
| M19 | Show that Azure OpenAI is unreachable from the public internet and that no API key exists anywhere. |
| M21 | A datacentre VM and k3s cluster appear in Azure, tagged and policy-evaluated by the same initiative as cloud resources. |

---

## 7. Certification readiness

- **AZ-104 (Azure Administrator)** at R5, **~month 16**, after M13–M14. Gap-find against the objective domain: typically Azure Files, VM backup, VNet peering details, storage redundancy options.
- **AZ-305 (Designing Azure Infrastructure Solutions)** → **Azure Solutions Architect Expert** (requires AZ-104), **~months 20–24**. AZ-305 is a design exam: identity/governance, data storage, business continuity, infrastructure. Your ADRs (014–017, 019, 023, 026) and solution designs are the preparation; study the Well-Architected and CAF material for services Runway does not use (SQL, Cosmos DB, Site Recovery).

See [Section 21](21-certification-strategy.md). Verify current exam codes before booking.

---

## 8. Resources

1. **Microsoft Cloud Adoption Framework — Ready: Azure landing zones** — "Design areas" (identity, resource organisation, network topology, governance). Read before M13.
2. **Azure Architecture Center — AKS baseline architecture** — read the whole reference architecture before M14; it is the closest public document to an enterprise AKS standard.
3. **Microsoft Entra documentation** — "Workload identity federation", "Managed identities overview", and the AWS IAM Identity Center SSO tutorial. **Azure RBAC docs** — "How Azure RBAC determines if a user has access".
4. **Azure Policy documentation** — "Policy definition structure", "Understand Policy effects", "Exemptions". **Kusto Query Language** — the KQL tutorial series in Azure Monitor docs.
5. **Microsoft Learn sandboxes** and ***Learn PowerShell in a Month of Lunches*** (ch. 1–13 for the pipeline and objects model).

→ Next: [Section 13 — Terraform Learning Path](13-terraform-learning-path.md)
