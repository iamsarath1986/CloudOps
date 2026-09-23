# Milestone M14 — Production AKS and Multi-Cluster GitOps

*Phase 14 · ~3 weeks · Months ~15.1–15.7 · Prerequisites: [M13](M13-azure-core.md), [M10](M10-observability.md) (and everything from [M09](M09-eks.md)) · Unlocks: [M15](M15-security-governance.md), [M17](M17-golden-paths.md)*

> You already operate Kubernetes. This milestone is about **what changes when the platform is two clusters in two clouds**: a different managed-Kubernetes model, one delivery mechanism for both, one observability view, and one set of add-ons, versioned and upgraded as a fleet.

---

## Why this phase exists

The JD names **Azure AKS** next to EKS and asks for *"onboarding applications using governed and automated DevOps practices"* across *"large public cloud footprints"*. A second cluster turns every manual step into a problem: two sets of credentials in CI, two places to drift, two upgrade calendars. That is the forcing function for GitOps and for treating clusters as a **fleet**, which is how platform teams actually work.

It sits after M13, because AKS lives in the VNet, identity and private DNS design you just built. It sits before security and governance (M15) and golden paths (M17), because both need a consistent multi-cluster baseline to enforce and to build on.

## Concepts to learn

**AKS versus EKS: the model differences**
- **Control plane:** Free versus Standard tier (uptime SLA) versus Premium (long-term support). Private cluster via **API Server VNet Integration** versus private endpoint. Contrast with EKS endpoint access.
- **Networking:** **Azure CNI Overlay** (pods from a private overlay CIDR, node IPs from the VNet) versus Azure CNI with pod subnet versus the legacy kubenet (being retired). **Cilium dataplane** (Azure CNI powered by Cilium) for NetworkPolicy and eBPF. Contrast with the VPC CNI consuming VPC IPs. Why IP exhaustion is less likely here, and what you give up (pods are not directly routable from the VNet).
- **Identity:** Entra ID integration for `kubectl` (Azure RBAC for Kubernetes authorisation versus Kubernetes RBAC with Entra groups), `kubelogin`, and local accounts disabled. **AKS Workload Identity** (OIDC issuer + federated credential + ServiceAccount annotation) is the counterpart to Pod Identity/IRSA.
- **Node pools:** system versus user pools, VM size per pool, spot pools, **cluster autoscaler** versus **node auto-provisioning** (Karpenter-based). Availability zones per pool.
- **Upgrades:** auto-upgrade channels (patch/stable/rapid/node-image), planned maintenance windows, `maxSurge` for node pools, node image upgrades separate from Kubernetes version upgrades, and the Kubernetes version support calendar and LTS.
- **Egress:** outbound type (`loadBalancer`, `managedNATGateway`, `userDefinedRouting`), and why enterprises force UDR to a hub firewall.
- Add-ons versus self-managed: Azure Monitor/Container Insights, Key Vault CSI provider, and Application Gateway for Containers (Azure's Gateway API implementation). Decide what to adopt managed and what to run yourself **identically across both clouds**.

**Multi-cluster delivery and fleet management**
- **GitOps:** desired state in Git, a controller in each cluster pulling and reconciling, drift continuously corrected, and history in Git. Push (CI holds cluster credentials) versus pull (the cluster holds read access to Git). What changes for secrets, promotion and rollback.
- **Argo CD internals:** Application, ApplicationSet (cluster and git generators), AppProject (RBAC/tenancy boundary), sync waves and hooks, health assessment, sync policies (automated, prune, self-heal), diffing and `ignoreDifferences`. The hub-and-spoke topology (one Argo CD managing both clusters) versus one Argo CD per cluster.
- **Promotion** between environments with GitOps: image digest updates via PR (Argo CD Image Updater or a CI job opening a PR), never by mutating the cluster.
- **Fleet add-on management:** a versioned add-on catalogue (ESO, cert-manager, Envoy Gateway, OTel Collector, Kyverno from M15, metrics-server), cluster-specific values, and upgrade waves (dev → prod, AKS → EKS or vice versa).
- **Helm at fleet scale:** umbrella charts versus per-add-on Applications, value hierarchies, and `helm template` + diff in PRs.

**Observability across clouds**
- OTel Collector on AKS remote-writing to AMP via **AWS IAM OIDC provider trusting the AKS issuer** (completing ADR-017), and logs/traces to Loki/Tempo across the internet over TLS with authentication. Cross-cloud egress cost and latency as telemetry costs.
- Container Insights / Azure Monitor for AKS control plane logs. Why you keep both, and what each can see that the other cannot.

## Architecture decisions in this phase

**ADR-018 Push pipelines vs GitOps (Argo CD) for multi-cluster delivery.**

| | |
|---|---|
| **Problem** | With two clusters in two clouds, push-based `helm upgrade` from CI needs credentials for every cluster in CI, cannot detect drift between deploys, and multiplies pipeline logic per cluster |
| **Options** | (a) Keep push (GitHub Actions + OIDC to both clouds). (b) **Argo CD, hub-and-spoke**, installed on EKS, managing EKS and AKS. (c) Argo CD per cluster. (d) Flux per cluster |
| **Decision** | (b) Argo CD running on EKS's `platform` pool, with AKS registered as a remote cluster via a dedicated ServiceAccount (credentials in Secrets Manager via ESO). ApplicationSets generate one Application per workload × cluster from `platform/gitops/`. CI builds, signs, and **opens a PR** bumping the digest. Merge = deploy |
| **Trade-offs** | The hub is a single point of control. If EKS is down, AKS keeps running but cannot be changed via GitOps, so the break-glass `kubectl` path is documented. Argo CD needs network reachability to AKS's API (authorised IP ranges). (c) avoids the hub SPOF at the cost of two Argo CDs to operate, and is noted as the scale-out path. (a) was rejected because of credential sprawl and no drift reconciliation. Flux is a legitimate alternative; Argo chosen for the UI and ApplicationSets |

Also documented (no new ADR number): **AKS network model = Azure CNI Overlay with Cilium**; **AKS traffic entry = Envoy Gateway behind Azure Standard LB**, cert-manager with DNS-01 on Route 53 (reusing ADR-009's portable pattern); **secrets = ESO with Key Vault** (ADR-011 extended).

## Hands-on labs

1. **AKS from Terraform** (`platform/modules/azure-aks`): private API via VNet integration, Entra-integrated auth with local accounts disabled, Azure CNI Overlay + Cilium, system pool (zonal, on-demand) + spot user pool, OIDC issuer + Workload Identity, maintenance window, auto-upgrade `patch` channel, egress via NAT Gateway. Destroy and rebuild. Time both. *Proves: the second cluster is also cattle.*
2. **The same failure, a different cloud.** Re-run M08's S12 injection set on AKS. Note what looks different (events, node names, CSI errors) and what is identical. *Proves: Kubernetes knowledge transfers, and cloud glue does not.*
3. **Workload Identity.** runway-portal's BFF reads a Key Vault secret via ESO + Workload Identity, and ai-gateway (stub) gets an Entra token. Break the ServiceAccount annotation and then the federated credential subject. Compare the errors with M09's IRSA failures. *Proves: symmetric patterns, different error surfaces.*
4. **AKS → AWS.** Register the AKS OIDC issuer as an IAM OIDC provider and create a role trusting `system:serviceaccount:observability:otel-collector`. The Collector remote-writes to AMP with no keys. *Proves: ADR-017 in both directions.*
5. **Argo CD bootstrap.** Install Argo CD on EKS with Helm (the only manual `helm install`, recorded as a runbook). Register AKS. App-of-apps for add-ons. ApplicationSet for workloads. Turn on self-heal, then `kubectl edit` a replica count in prod and watch it revert. *Proves: continuous reconciliation.*
6. **Sync waves.** Deploy a workload whose Deployment needs a Secret from ESO and a CRD from an add-on. Order it wrong, then fix it with sync waves and health checks. *Proves: ordering is part of delivery.*
7. **Rollback, GitOps-style.** Deploy a bad digest via PR. Roll back by reverting the PR and time it. Compare with M07's measured rollback time, and with `argocd app rollback` (and why that leaves Git and the cluster disagreeing). *Proves: Git is the source of truth, including for rollback.*
8. **Fleet upgrade.** Upgrade the AKS control plane and node pools one minor version (surge, PDBs, k6 running), then upgrade one add-on (cert-manager) across both clusters in waves via Git. *Proves: lifecycle as a fleet.*
9. **Cross-cloud call path.** insights-worker stub on EKS calls an echo endpoint on AKS through Envoy Gateway with an Entra token obtained via federation. Measure the latency added by crossing clouds, and the egress bytes. *Proves: what ADR-015 costs.*

## Project implementation

- AKS `runway-az-prod` (and a `dev` variant sharing the module) in the spoke VNet. Karpenter-equivalent node auto-provisioning or cluster autoscaler per ADR note. Envoy Gateway + cert-manager. ESO → Key Vault. OTel Collector → AMP/Loki/Tempo. Pod Security Admission `restricted`. Default-deny Cilium NetworkPolicy.
- `platform/gitops/`: `clusters/{eks-dev,eks-prod,aks-dev,aks-prod}/` (cluster values), `addons/` (versioned catalogue with ApplicationSets), `workloads/` (ApplicationSet per workload, generated per target cluster), AppProjects per team (tenancy boundaries reused in M17).
- runway-portal BFF deployed on AKS (the SPA stays on S3 + CloudFront). hello-svc placeholders on both clusters.
- The `deploy` reusable workflow is rewritten: build → scan → sign → push → **PR to `platform/gitops`** with the new digest (dev auto-merge, prod via environment approval + change record from M11). Push-based `helm upgrade` is retired and recorded in the EOL register.
- Grafana: a cluster selector on every dashboard. Azure Monitor data source panels for the AKS control plane. SLO rules labelled by cluster/cloud.
- `docs/runbooks/aks-upgrade.md`, `docs/runbooks/argocd-break-glass.md`, `docs/runbooks/fleet-addon-upgrade.md`.

## Engineering problems to hit deliberately

| Problem | How to cause it | What you should learn |
|---|---|---|
| Argo shows Synced but the app is broken | No health check for a CRD / readiness not modelled | Synced ≠ Healthy. Teach Argo what healthy means |
| Self-heal fights a human | Manual hot-fix in prod | Emergency changes must go through Git too, or be paused explicitly |
| Prune deletes something important | Resource removed from Git by accident | Prune policies, `PrunePropagationPolicy`, protected resources |
| AKS pods can't pull from ECR | Cross-cloud registry auth | Registry strategy per cloud (ACR replication versus cross-cloud pull), an ADR-worthy note |
| Upgrade stalls | PDB `minAvailable` = replicas, surge 0 | Same lesson as EKS, a different knob |
| Cross-cloud latency surprise | Synchronous call chain across clouds | Placement decisions have latency budgets |

## Troubleshooting exercises

- **S26, "Argo CD shows OutOfSync/Degraded; prod differs from Git".** Three variants injected blind: a manual `kubectl` change (self-heal off in that project), a sync-wave ordering bug (Deployment before its ExternalSecret), and a manifest that renders differently per cluster because of a values typo. Full write-up.
- **S36, "Cluster upgrade blocked / workloads break after upgrade"**, on AKS. A node pool upgrade stalls on a PDB, and an add-on version incompatible with the new Kubernetes minor. Write it up as a change with PIR.
- **S25 revisited:** AKS was recreated, its issuer changed, and the Collector's AMP remote-write fails. The detection must be the absence-of-data alert from M10. If it did not fire, that is the finding.
- **S24 in AKS:** the BFF cannot reach Key Vault after someone added a new private DNS zone link in the wrong VNet. Diagnose it from a debug pod.
- **"Deploys to AKS are stuck, EKS is fine"**: Argo CD cannot reach the AKS API after authorised IP ranges changed. Mitigation through the break-glass path, and the permanent fix in Terraform.

## Deliverables

- [ ] `platform/modules/azure-aks` + live roots; AKS private, Entra-integrated, Workload Identity, CNI Overlay + Cilium
- [ ] Argo CD hub on EKS managing four cluster targets; `platform/gitops/` with add-on catalogue, ApplicationSets, AppProjects
- [ ] Deploy workflow = PR-to-GitOps with change records; push deploys retired (EOL register)
- [ ] Cross-cloud observability via federation (no static keys)
- [ ] ADR-018; runbooks: AKS upgrade, Argo break-glass, fleet add-on upgrade
- [ ] Incident records S26, S36 (AKS), S25 variant

## Completion criteria

- A digest change merged to `platform/gitops` reaches **both** clusters' dev with no human action, and prod after approval. The median time from merge to Healthy is measured.
- A manual change in prod is reverted by self-heal within the reconciliation interval, and a Git-revert rollback is measured and compared with M07's number.
- **No cluster credentials exist in CI.** You prove it by listing GitHub secrets and OIDC trusts.
- You explain VPC CNI versus Azure CNI Overlay, Pod Identity versus AKS Workload Identity, and ALB controller versus Envoy Gateway **as trade-offs**, not as trivia.
- Every dashboard answers "which cluster/cloud?" and the SLOs are computed per cluster.
- AKS minor upgrade + one fleet add-on upgrade completed with traffic flowing and errors measured.

## Mentor checkpoints

- **Before you build:** the ADR-018 draft, including what happens when the hub is down. I will ask you to run a deploy with EKS "unavailable".
- **During:** your `platform/gitops/` layout. Expect questions on how a new cluster or a new team is added, because that answer is M17's foundation.
- **Questions I will ask:** Push versus pull, and who holds which credentials? What does "Synced" not tell you? Why Overlay here but VPC IPs on EKS? How do you roll back in GitOps without lying to Git?
- **The failure I will inject:** a change that is correct in Git and wrong in exactly one of the four clusters.

## Resources

1. **AKS docs:** *Concepts → Networking (Azure CNI Overlay, Azure CNI powered by Cilium)*, *Use Microsoft Entra Workload ID with AKS*, *Upgrade options and auto-upgrade*, *Planned maintenance*; **Azure Architecture Center:** *AKS baseline architecture* (read in full).
2. **Argo CD docs:** *Core concepts*, *Operator manual → Declarative setup, Cluster management*, *User guide → Sync waves and hooks, Auto sync, ApplicationSet*.
3. ***Production Kubernetes* (Rosso et al.):** ch. *Multitenancy*, *Platform Abstractions*, *Continuous Delivery / GitOps*.
4. **OpenGitOps principles** (opengitops.dev): the four principles, as the definition to argue from.

## Estimate vs actual

| | Hours | Cost |
|---|---|---|
| Planned | 30 | ~$40–70 Azure (AKS nodes, NAT GW, LB in windows) + AWS unchanged |
| Actual | | |
| Variance note | | |
