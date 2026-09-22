# Milestone 8 — Kubernetes

*Phase 7 · ~5 weeks, the longest milestone, deliberately · Prerequisite: Milestones 5, 6, 7*

> Learn on `kind` locally for free; **operate** on AKS. Do not spend budget watching a pod start — spend it on node pools, autoscaling, upgrades, spot eviction, cloud load balancers and cloud identity.

**Learning topics**
* Control plane and node components; the **reconciliation loop** as the idea that makes everything else predictable
* Workload objects and when each is right: Deployment, StatefulSet, DaemonSet, Job, CronJob
* Services, Ingress, ingress controllers; ConfigMaps and Secrets (base64, not encryption)
* Probes — and the self-inflicted outage of a liveness probe restarting a healthy-but-slow pod
* Requests, limits, QoS classes; what CPU throttling does to latency
* Scheduling: node selectors, affinity, taints, tolerations, topology spread
* HPA, cluster autoscaler, and why they interact badly when requests are wrong
* Rollouts, `maxSurge`/`maxUnavailable`, PodDisruptionBudgets
* Namespaces, ResourceQuota, LimitRange as multi-tenancy tools; cluster RBAC; default-deny NetworkPolicy
* Helm vs Kustomize; AKS specifics: node pools, free control plane tier, spot, workload identity, CSI secrets driver, version upgrades

**Build**
`platform/modules/aks`. `ingress-nginx` with TLS from cert-manager. A namespace per workload with quota and limit ranges. `cargo-api`, `carrier-sim` and the ETA worker deployed with correct probes, requests/limits, PDBs and HPA. Secrets mounted from Key Vault via the CSI driver — **no secrets in manifests**.

**Cloud resources**
AKS with free-tier control plane, a small system pool and a spot user pool, workload identity and Managed Prometheus enabled. An Azure Load Balancer fronting ingress. Destroyed between sessions.

**DevOps work**
The `deploy` workflow now targets AKS and watches rollout status, failing the job if the rollout does not converge. Manifests templated thinly; third-party components via Helm.

**Operations exercise**
Deliberately cause and diagnose **each** of `ImagePullBackOff`, `CrashLoopBackOff`, `OOMKilled`, `Pending`/unschedulable, `CreateContainerConfigError`, and a readiness probe that never passes. For each, write the three commands that identify it — **this list is the Kubernetes interview.** Then set CPU limits at 100m on a busy service and watch p99 explode from throttling while CPU usage looks fine. Then drain a node, then run a cluster version upgrade with traffic flowing and a PDB in place.

**Security considerations**
Default-deny NetworkPolicy with explicit allows — break your own service first, then allow exactly what is needed. Misconfigure workload identity so the pod cannot reach Key Vault, and trace the token exchange to find out why.

**Expected outcome**
An AKS cluster entirely in Terraform, carrying three workloads with resource governance, disruption budgets, autoscaling and network policy, upgradable without an outage.

**Evidence of mastery**
Given only `kubectl`, diagnose any of the six pod failure states in under five minutes and explain the cause. Explain step by step what happens between `kubectl apply` and a running pod. Explain why CPU limits can make a service slower. Explain requests vs limits and how they drive both scheduling and autoscaling.
