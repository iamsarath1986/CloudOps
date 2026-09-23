# Milestone M13 — Azure Core, Landing Zone and Enterprise Identity

*Phase 13 · ~4 weeks · Months ~14.1–14.9 · Prerequisites: [M03](M03-cloud-architecture.md) (vendor-neutral concepts), [M09](M09-eks.md) (a production cluster and IAM depth) · Unlocks: [M14](M14-aks-multicluster.md), [M21](M21-hybrid-multicloud.md)*

> **The rule for the second cloud: learn the differences, not the duplicates.** You will not rebuild M04 by hand in Azure. You already know what a VPC, a subnet, a route table and a role are. This milestone teaches where Azure's *model* differs, and puts Azure where it is strongest: enterprise identity, hierarchical governance and private PaaS access.

---

## Why this phase exists

The JD requires *"proficiency in managing and operating AWS **and** Azure"* and *"operating large public cloud footprints and onboarding applications using governed and automated DevOps practices."* Enterprises such as the one in the JD usually run Microsoft identity as the system of record. Entra ID is where people exist and AWS is one of the places they go. Making Entra the workforce IdP for *both* clouds is the most realistic multi-cloud control you can build.

It sits here because your concepts (M03), IAM depth (M04) and Kubernetes operations (M09) are all transferable now. Azure at month 14 is a translation exercise with new ideas, not a second curriculum. It comes before AKS (M14), because AKS depends on the VNet, identity and private DNS design you make here.

## Concepts to learn

**The resource model: what is genuinely different from AWS**
- Tenant (Entra ID) → management groups → subscriptions → resource groups → resources. The **resource group** has no clean AWS equivalent: it is a lifecycle and RBAC unit inside a subscription. Compare with the AWS account as the hard isolation boundary and the Organizations OU as the policy scope.
- Azure Resource Manager as the single control-plane API (ARM/Bicep/Terraform all call it). Resource providers and their registration. Regions, availability zones (logical-to-physical zone mapping differs per subscription) and region pairs.

**Identity: the biggest conceptual shift**
- **Entra ID is a directory first**: users, groups, app registrations, service principals (the tenant-local instance of an app) and managed identities (system- vs user-assigned). Contrast with AWS IAM, where roles are assumed and principals are account-scoped.
- **Azure RBAC**: role definition + principal + **scope**, inherited downward (MG → sub → RG → resource). Additive, plus deny assignments. Contrast with AWS policy evaluation (explicit deny wins, SCP ∩ identity ∩ resource policies ∩ boundaries). Data-plane versus control-plane roles (Key Vault, Storage) is a common confusion.
- **Entra roles (directory) vs Azure roles (resources).** Privileged Identity Management (just-in-time elevation), Conditional Access and access reviews, at concept level plus one exercise.
- **Workforce federation to AWS:** Entra ID as SAML IdP and SCIM provisioner for **AWS IAM Identity Center**. Groups in Entra → permission sets in AWS. One joiner/mover/leaver process for both clouds.
- **Workload identity federation:** federated identity credentials on a user-assigned managed identity or app registration, trusting an external OIDC issuer (GitHub Actions, the **EKS OIDC issuer**). Issuer, subject and audience must match exactly. This is the mirror image of M09's IRSA.

**Networking: the differences**
- VNets span all AZs in a region (subnets are regional, unlike AWS subnets which are zonal). NSGs attach to subnets or NICs and are stateful. There are no separate NACLs. UDRs (route tables). **There is no Internet Gateway object.** Outbound through NAT Gateway or default outbound access, which is being retired, so be explicit.
- Hub-spoke with VNet peering (non-transitive). Azure Firewall/NVA in the hub (conceptual here, because of cost).
- **Private endpoints + private DNS zones** (`privatelink.vaultcore.azure.net`, and so on): a PaaS service gets a NIC in your subnet, and name resolution must be redirected by linking the private DNS zone to the VNet. **This is the most common Azure networking incident.** Compare with AWS interface endpoints and private DNS.
- Service endpoints versus private endpoints, and why enterprises mandate the latter.

**Governance scaffolding (deepened in M15)**
- Management group hierarchy à la Cloud Adoption Framework (CAF) landing zones: `platform` (identity, management, connectivity) versus `landing-zones` (corp/online) versus `sandbox`. Subscription vending as the Azure equivalent of AWS account vending.
- Azure Policy basics (assignment at an MG, effects) enough to enforce tags and allowed regions now. Full policy-as-code comes in M15.
- Cost Management scopes and budgets. Resource tags do **not** inherit from resource groups by default, so policy must do it.

**Operations**
- Azure Monitor: Activity Log (control plane), resource logs via diagnostic settings, Log Analytics and **KQL**, metrics. Azure Resource Graph for inventory queries (KQL over resources). The Az CLI and **Az PowerShell**: the JD names PowerShell, and this is where you use it for real.

## Architecture decisions in this phase

| ADR | Problem | Options | Decision | Trade-offs |
|---|---|---|---|---|
| **ADR-015** Workload placement across AWS and Azure | Using two clouds only makes sense if each hosts what it is best at, otherwise it is duplicated cost | Duplicate everything in both · split by environment · **split by capability** | AWS/EKS: transactional workloads, data, shared observability. Azure/AKS: the enterprise-identity-facing and AI tier (runway-portal BFF, ai-gateway, flightdeck) next to Entra ID and Azure OpenAI. hello-svc on both via the golden path | Cross-cloud calls (latency, egress cost, identity federation) are now part of the design. Two edge patterns. You must operate both. Justified by the JD and by where each cloud is strongest |
| **ADR-016** Workforce identity | Two clouds, two user stores, two offboarding processes | Separate IAM Identity Center directory + Entra · **Entra ID as IdP for IAM Identity Center (SAML + SCIM)** · a third-party IdP | Entra as the single workforce IdP. Permission sets mapped from Entra groups | Migrating the Identity Center identity source is disruptive (a normal change, with a rollback plan and a break-glass IAM user kept sealed). Entra becomes a dependency of AWS console access |
| **ADR-017** Cross-cloud workload identity | EKS workloads must call Azure (finops-collector, insights-worker → ai-gateway), and AKS workloads must call AWS (Bedrock, AMP) | Static keys in secrets · a secret broker · **OIDC federation both ways** | Federation both ways: Entra federated credentials trusting the EKS issuer, and an AWS IAM OIDC provider trusting the AKS issuer | Issuer URLs must be stable (recreating a cluster changes them, so Terraform must update both sides). Token lifetimes and clock skew. Debugging spans two clouds' audit logs. **Zero static credentials is worth it** |

## Hands-on labs

1. **Translation sheet first.** Before touching Azure, fill in `docs/architecture/aws-azure-model-differences.md`: account vs subscription vs RG, IAM policy evaluation vs RBAC scope inheritance, zonal vs regional subnets, SG vs NSG, IGW vs none, interface endpoint vs private endpoint. Then verify every row with an experiment. *Proves: concept-first portability ([A1](../appendix/A1-cloud-architecture-concepts.md)).*
2. **Scope inheritance.** Assign Reader at the MG, Contributor at one RG, and a deny assignment/lock on a resource. Predict effective access, then test it with `az role assignment list --all` and actual calls. Compare with an AWS SCP + identity policy scenario. *Proves: two authorisation models.*
3. **Data-plane gotcha.** Give yourself Owner on a Key Vault (RBAC mode) and try to read a secret. You can't. Add `Key Vault Secrets User`. *Proves: control plane ≠ data plane.*
4. **Private endpoint the hard way.** Key Vault with public access disabled, a private endpoint in the spoke, and a private DNS zone **not** linked. `nslookup` resolves to a public IP and access is denied. Link the zone and the name resolves to 10.30.x.x. *Proves: S24's mechanics.*
5. **GitHub → Azure with no secret.** Federated credential for the `runway` repo's `environment:prod` subject. Break the subject (`ref:` vs `environment:`) and read `AADSTS70021`. *Proves: federation claims matter.*
6. **EKS pod → Entra token.** Create a user-assigned managed identity with a federated credential trusting the EKS OIDC issuer and `system:serviceaccount:finops:finops-collector`. From the pod, exchange its projected token for an Entra token (MSAL / `azure-identity` `WorkloadIdentityCredential` with env vars) and list the subscription's cost. *Proves: ADR-017 works.*
7. **PowerShell for real.** Write `Get-RunwayAzureInventory` (Az PowerShell + Resource Graph): every resource missing mandatory tags, and every public IP, with output as objects and CSV. Package it as a module with Pester tests in `platform/scripts/powershell/`. *Proves: the JD's PowerShell line with a tool you will reuse.*
8. **Identity source migration (normal change).** Switch IAM Identity Center's identity source from its built-in directory to Entra ID (SAML + SCIM). Pre-create the groups, map the permission sets, keep a break-glass path, and validate with a test user before cut-over. *Proves: change management on a real control plane.*

## Project implementation

- `platform/live/azure/…`: management group hierarchy (`runway` → `platform`, `landing-zones`, `sandbox`), realistically **2 subscriptions** (`runway-platform`, `runway-workloads`) with env separation by resource group. State this constraint in the README. Terraform state in Azure Blob (lease locking) per ADR-005.
- `platform/modules/azure-network`: hub VNet (shared services, future VPN gateway subnet for M21) and spoke VNet 10.30.0.0/16 (AKS nodes, private endpoints, AKS API server subnet), NSGs, UDRs, explicit NAT Gateway for egress, peering, private DNS zones linked to hub and spoke.
- `platform/modules/azure-keyvault`: RBAC mode, purge protection, private endpoint.
- Entra ID: groups `runway-platform-admins`, `runway-developers`, `runway-readers`, `runway-l1-servicedesk`, mapped to Azure RBAC and to AWS permission sets. PIM for `platform-admins` if your licence allows (otherwise documented as a gap).
- Federation: GitHub → Entra (Terraform workflows for Azure), EKS issuer → Entra (finops-collector identity), AWS IAM OIDC provider placeholders for the AKS issuer (completed in M14).
- Budgets + a Cost Management export (FOCUS format prepared for M16). Diagnostic settings → Log Analytics workspace (ingestion cap set!). Activity Log exported.
- Initial Azure Policy: required tags (Modify to inherit from RG), allowed locations, deny public IPs in landing zones (audit first).
- Grafana gets Azure Monitor as a data source.

## Engineering problems to hit deliberately

| Problem | How to cause it | What you should learn |
|---|---|---|
| "Owner but can't read secret" | RBAC-mode Key Vault, no data-plane role | Control vs data plane roles |
| PaaS resolves to public IP | Private DNS zone not linked / wrong zone name | Private endpoint = NIC + DNS |
| Federated login fails | Subject/issuer/audience mismatch | Claims are matched exactly |
| Role assignment "doesn't work" | Propagation delay / assigned at the wrong scope | Scope inheritance and eventual consistency |
| Egress breaks unexpectedly | Relying on default outbound access | Make egress explicit (NAT GW / firewall) |
| Tags missing on resources | Assumed RG tag inheritance | Policy `Modify` / `inherit tag` effects |

## Troubleshooting exercises

- **S24, "AKS app cannot reach a private PaaS service".** Here with a VM or a container in the spoke, before AKS exists: Key Vault access denied from the workload subnet. Full write-up. The trap is loosening the firewall, when the real cause is DNS.
- **S25, "Cross-cloud call fails with 401/invalid token".** The EKS cluster was recreated in a rebuild, so its OIDC issuer URL changed and the Entra federated credential still trusts the old one. finops-collector fails silently until your absence-of-data alert fires. Full write-up. The permanent fix is Terraform wiring the issuer output into the Azure federated credential, plus a synthetic token-exchange check.
- **S04 in Azure clothing:** the Terraform pipeline fails with `AuthorizationFailed` after a role assignment was moved from subscription to RG scope. Diagnose it from the Activity Log.
- **"Engineers can't log into AWS after the IdP switch."** A SCIM group sync scoping issue during lab 8. Mitigate via break-glass, and fix the forward path. Write it up as a failed-change PIR.

## Deliverables

- [ ] `docs/architecture/aws-azure-model-differences.md` (verified by experiment)
- [ ] Azure landing zone in Terraform: MGs, subscriptions, hub-spoke, private DNS, Key Vault, budgets, diagnostic settings, initial policies
- [ ] Entra groups + Azure RBAC + IAM Identity Center federation (SAML + SCIM)
- [ ] Federation: GitHub → Entra, EKS → Entra working, AKS → AWS prepared
- [ ] `platform/scripts/powershell/RunwayInventory` module + Pester tests
- [ ] ADR-015, ADR-016, ADR-017; `docs/changes/CHG-0xx-identity-source-entra.md`
- [ ] Incident records S24, S25; one PIR

## Completion criteria

- `terraform apply` from empty builds the Azure landing zone in **under 20 minutes**, and destroy leaves only the state storage and budgets.
- **Zero static credentials** across GitHub → AWS, GitHub → Azure and EKS → Azure. You prove it by listing every identity and its credential type.
- You explain AWS policy evaluation and Azure RBAC inheritance side by side, with a scenario where the same intent needs different constructs.
- A new Entra user added to `runway-developers` gets the correct Azure RBAC **and** AWS permission set with no manual AWS step, and is removed from both by one removal.
- You diagnose a blind S24 in under 10 minutes using `nslookup`/`dig` from inside the VNet.
- Your PowerShell module runs in CI with passing Pester tests and produces a useful report on your own estate.

## Mentor checkpoints

- **Before you build:** ADR-015 draft. I will argue for "just put everything on EKS". Defend the split in business and engineering terms, or change it.
- **During:** the translation sheet. I will pick three rows and ask you to prove them live.
- **Questions I will ask:** Why is a resource group not an account? What does a private endpoint actually create? What happens to your federated credentials when you rebuild a cluster? Who can grant themselves Owner in your tenant?
- **The failure I will inject:** an identity problem that looks like a network problem.

## Resources

1. **Microsoft Cloud Adoption Framework:** *Ready → Azure landing zone → Design areas* (resource organisation, identity and access, network topology) and *Management group and subscription organisation*.
2. **Azure docs:** *Azure RBAC → How Azure RBAC works / Understand scope*; *Private Link → Azure Private Endpoint DNS configuration*; *Workload identity federation (Entra)*.
3. **AWS IAM Identity Center User Guide:** *Configure SAML and SCIM with Microsoft Entra ID*.
4. ***Learn PowerShell in a Month of Lunches*:** chapters on the pipeline, objects, modules, and scripting and tests. Plus the Pester docs *Quick start*.
5. **Microsoft Learn sandbox paths:** *AZ-104: Manage identities and governance* and *Configure and manage virtual networks*. Use them as a gap-finder, not a course.

## Estimate vs actual

| | Hours | Cost |
|---|---|---|
| Planned | 40 | ~$20–40 Azure (mostly Key Vault, private endpoints, NAT GW in windows, Log Analytics) + unchanged AWS |
| Actual | | |
| Variance note | | |
