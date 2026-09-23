# Milestone M09 — Production Kubernetes on EKS (+ ECS → EKS migration)

*Phase 09 · ~5 weeks · Months ~9.1–10.2 · Prerequisites: [M05](M05-terraform.md), [M07](M07-cicd.md), [M08](M08-kubernetes-fundamentals.md) · Unlocks: [M10](M10-observability.md), [M13](M13-azure-core.md)*

> Kubernetes basics are free on a laptop. **This milestone buys the parts only a cloud teaches**: VPC-native pod networking, IAM for pods, cloud load balancers, node provisioning, managed upgrades, and the bill that comes with each. `make up` at the start of every session and `make down` at the end. The EKS control plane is billed hourly.

---

## Why this phase exists

The JD names **Amazon EKS** explicitly, asks for *"proficiency in managing and operating AWS"*, and expects you to *"manage end-of-life replacement of equipment and/or software"*. EKS is the less-managed of the two Kubernetes services. You own the CNI's IP budget, the IAM plumbing for pods, the load balancer controller, node provisioning and add-on versions. That is why it teaches more than AKS, and why it comes first ([ADR-002](M03-cloud-architecture.md)).

It also contains your first **production migration**. Runway's workloads have run on ECS Fargate since M07, and you will move them to EKS with users still connected. Being able to move a live service between runtimes without an outage is a core competence for a senior engineer. The JD's *"Own and manage changes"* is tested here for real for the first time.

## Concepts to learn

**EKS architecture: who owns what**
- AWS runs the control plane (API server and etcd across three AZs) in an AWS-owned account and connects it to your VPC through cross-account ENIs. You own the nodes, the add-ons, and every configuration choice.
- Cluster endpoint access: public, private, or both. Why production uses private (or public with a CIDR allow-list) and how `kubectl` then reaches it.
- **Access management:** EKS access entries and access policies (the current model) versus the legacy `aws-auth` ConfigMap. How an IAM principal becomes a Kubernetes identity. Break-glass access.
- Managed node groups versus self-managed versus Fargate profiles versus **Karpenter** versus **EKS Auto Mode**, and what each hides from you.
- Add-ons and their lifecycle: VPC CNI, CoreDNS, kube-proxy, EKS Pod Identity Agent, EBS CSI driver. Version compatibility with the control plane is *your* problem.

**Networking the AWS way**
- **Amazon VPC CNI:** pods get real VPC IPs from ENIs attached to nodes. Consequences: the per-instance max-pods limit, **subnet IP exhaustion**, prefix delegation, and secondary CIDRs (100.64.0.0/10) for pod ranges. Why you sized private subnets as `/19`s in ADR-004.
- Security groups for pods (concept), and NetworkPolicy enforcement via the VPC CNI's network policy agent.
- **AWS Load Balancer Controller:** Ingress/Gateway → ALB, `Service type=LoadBalancer` → NLB. `target-type: ip` versus `instance`. Target group health checks versus Kubernetes readiness: two health systems that must agree. `deregistration_delay` and pod `preStop` sleeps, which are needed for zero-downtime rollouts.
- Private subnets, NAT gateway and **VPC endpoints** (ECR api/dkr, S3 gateway, STS, Secrets Manager, SQS, CloudWatch Logs). What breaks when an endpoint or NAT route is missing, and what NAT data processing costs.

**Identity for pods**
- **IRSA:** the cluster's OIDC issuer, projected ServiceAccount tokens, `AssumeRoleWithWebIdentity`, and the trust-policy `sub`/`aud` conditions. You need to understand it deeply because **its OIDC issuer is what enables cross-cloud federation in M13**.
- **EKS Pod Identity:** an agent DaemonSet plus associations, no per-cluster OIDC provider, and simpler trust policies. The default for Runway's in-AWS access.
- IMDS hop limit and why pods should not inherit the node role.

**Compute and scaling**
- **Karpenter:** NodePools, EC2NodeClasses, bin-packing, consolidation, disruption budgets, spot interruption handling, and instance-type flexibility. Contrast with Cluster Autoscaler (ASG-based, node-group-shaped) and EKS Auto Mode (AWS-operated Karpenter plus add-ons).
- The interaction: HPA adds pods → pods go Pending → Karpenter adds nodes → the scheduler binds. Each hop has a latency. Know the total.
- Node pools by workload class: `system` (on-demand, tainted for critical add-ons), `platform` (observability in M10), `general` (spot-friendly, stateless).

**Storage and state**
- EBS CSI: gp3 StorageClass, `WaitForFirstConsumer`, and the fact that **an EBS volume lives in one AZ**, so a StatefulSet pod is pinned to a zone. The Redis StatefulSet teaches this.
- Why Postgres is RDS and not a StatefulSet: backups, PITR, patching, failover as a service.

**Lifecycle**
- The EKS version support calendar (standard versus extended support and its surcharge). The upgrade order: control plane → add-ons → nodes. Deprecated API removal. Tooling such as `kubent`/`pluto` and EKS upgrade insights.
- Blue/green node pools versus in-place node rolling. PodDisruptionBudgets as the contract that makes draining safe.

## Architecture decisions in this phase

| ADR | Problem | Options | Decision | Trade-offs |
|---|---|---|---|---|
| **ADR-008** Moving from ECS to EKS | Runway will need many workloads, standard tenancy, a portable golden path and a second cloud. ECS is AWS-only and lacks a tenancy/policy ecosystem | Stay on ECS · EKS · both | Migrate to EKS. **Keep ECS knowledge as the fallback option for simple single services** | More to operate (add-ons, upgrades, CNI). In exchange: portability to AKS, namespaces/RBAC/policy, the Helm/Argo ecosystem. Migration via **weighted ALB target groups** (ECS 90/10 → 0/100), with rollback by weight |
| **ADR-009** Traffic entry | Expose EKS workloads with TLS and WAF, and replace the retired ingress-nginx | ingress-nginx (retired) · AWS LB Controller + ALB · in-cluster Gateway API implementation behind an NLB | AWS LB Controller provisioning an ALB (ACM certificate, AWS WAF in timeboxed windows), `target-type: ip`. Gateway API resources where the controller supports them. Envoy Gateway is the portable pattern reused on AKS in M14 | Two different edge patterns across the clouds (a cost of portability, stated honestly). ALB is billed per LCU. The cut-over checklist from M08 lab 12 is the migration evidence |
| **ADR-010** Node autoscaling | Pending pods must get capacity quickly and cheaply, spot-safe | Cluster Autoscaler · Karpenter · EKS Auto Mode | Karpenter | You operate Karpenter itself (versioned, needs IAM, needs its own nodes). Auto Mode was rejected *for learning*: it hides exactly what M09 exists to teach. Note that it may be the right choice at work |
| **ADR-011** Secrets delivery | Pods need secrets without secrets in Git or manifests | Kubernetes Secrets from CI · Secrets Store CSI · **External Secrets Operator** | ESO with AWS Secrets Manager via Pod Identity. The same operator is reused with Key Vault on AKS in M14 | Secrets still land as Kubernetes Secrets (protect with RBAC and envelope encryption via KMS). A refresh interval means rotation is not instant |

## Hands-on labs

1. **Cluster from Terraform.** Build `platform/modules/aws-eks`: private and public endpoint with a CIDR allow-list, access entries, add-ons pinned by version, and KMS envelope encryption for Secrets. `terraform destroy` it, then rebuild it, timing both. *Proves: the cluster is cattle.*
2. **Access entries.** Map a `platform-admins` IAM Identity Center role to cluster-admin and a `developers` role to namespace-scoped edit. Try `kubectl delete ns kube-system` as a developer. *Proves: IAM → Kubernetes RBAC mapping.*
3. **IP exhaustion on purpose.** Put a node group in a tiny `/27` subnet, scale the pods, and watch them sit `Pending` or `ContainerCreating` with CNI errors. Fix it with prefix delegation or proper subnets. *Proves: VPC CNI economics.*
4. **IRSA by hand, then Pod Identity.** Wire IRSA for `eta-worker` → SQS manually: OIDC provider, trust policy, annotation. Read the projected token (`jwt.io` locally). Break the `sub` condition and read the error. Then replace it with a Pod Identity association and compare the trust policies. *Proves: pod → IAM, both mechanisms.*
5. **ALB zero-downtime.** Deploy with the LB controller, run k6 at a constant rate, and roll the Deployment. Count the 5xx. Add `preStop` sleep + `terminationGracePeriodSeconds` + readiness gate, tune the deregistration delay, and repeat until errors reach zero. *Proves: two health systems must agree.*
6. **Karpenter.** Create a NodePool with spot + on-demand. Scale to 30 replicas and time Pending → Running. Scale down and watch consolidation. Simulate a spot interruption through the interruption queue. *Proves: autoscaling latency and disruption.*
7. **Egress without NAT.** Remove the NAT route in dev and keep only the VPC endpoints. Find out what still works (ECR pulls, STS, SQS) and what does not (carrier-sim's external call, GitHub). *Proves: the egress map of your system.*
8. **Upgrade.** Put a deprecated API (an old `autoscaling/v2beta2` HPA manifest in a chart) into Git. Run `pluto`/`kubent` and EKS upgrade insights. Upgrade the control plane one minor version, then the add-ons, then the nodes (Karpenter drift), with k6 running and PDBs in place. *Proves: the upgrade order and API removal.*
9. **EBS and zones.** Force the Redis StatefulSet's node away in a different AZ. The pod stays Pending because of volume node affinity. *Proves: stateful + zonal storage.*

## Project implementation

- `platform/live/aws/workloads-dev/dev/eks` and `…/workloads-prod/prod/eks` using `modules/aws-eks`. Three node classes via Karpenter: `system` (on-demand, tainted), `platform`, `general` (spot-first). A free-tier-friendly footprint in dev.
- Add-ons as code: VPC CNI (prefix delegation, network policy enabled), CoreDNS, kube-proxy, Pod Identity Agent, EBS CSI, AWS Load Balancer Controller, Karpenter, External Secrets Operator, cert-manager (for internal TLS), metrics-server. All pinned. Record them in `docs/lifecycle/eol-register.md`.
- `cargo-api`, `carrier-sim` and `eta-worker` deployed with the `runway-service` chart. Postgres is **RDS** (from M05), reached over the data subnet with a security group reference. Redis is an in-cluster StatefulSet on gp3. `eta-worker` uses real **SQS + DLQ** via Pod Identity.
- ALB with an ACM certificate on `cargo.<your-domain>`, Route 53 alias, and WAF in timeboxed windows.
- The **ECS → EKS migration**: `docs/changes/CHG-0xx-ecs-to-eks.md` (normal change: risk, plan, validation, rollback by weight, communication), executed with weighted target groups, then ECS decommissioned and its Terraform removed. That completes the first EOL entry.
- The `deploy` reusable workflow now runs `helm upgrade --atomic --wait` against EKS via GitHub OIDC → an IAM role with an access entry. The job fails if the rollout does not converge. (Push-based until M14.)
- Namespace per workload, with quota, LimitRange, default-deny NetworkPolicy and Pod Security Admission `restricted`.

## Engineering problems to hit deliberately

| Problem | How to cause it | What you should learn |
|---|---|---|
| Pods can't get IPs | Undersized subnets / max-pods | The VPC CNI trades simplicity for IP consumption |
| 502s during every deploy | No `preStop`, deregistration delay > grace period | ALB and kubelet shut down on different clocks |
| AccessDenied from a pod | Wrong Pod Identity association / IRSA `sub` | Read the assumed-role ARN in CloudTrail, not the app log |
| Image pulls hang in private subnets | Missing ECR/S3 endpoints and no NAT | ECR pulls layers from S3 |
| Karpenter launches nothing | NodePool requirements unsatisfiable / IAM / subnet tags | Karpenter logs and NodeClaim status are the evidence |
| Upgrade breaks a chart | API removed in the target version | Scan before you upgrade, not after |
| Bill up after a week | NAT processing + idle nodes left up | Preview of S29 (M16) |

## Troubleshooting exercises

- **S13, "Pods stuck Pending"**, injected blind, one of: cluster IP exhaustion, a taint with no toleration, ResourceQuota exhausted, or a Karpenter NodePool that cannot satisfy an instance requirement. Target: name the cause from `describe`, events, NodeClaims and Karpenter logs in under 10 minutes.
- **S14, "Pod gets AccessDenied calling AWS".** `eta-worker` stops draining SQS. The queue depth alarm (CloudWatch for now) is your detection. Full write-up.
- **S15, "Private workloads cannot reach the internet / AWS APIs".** Someone "cleaned up" a route table. Some calls fail (external) and some keep working (via VPC endpoints). The mixed signal is the lesson. Full write-up.
- **S36, "Cluster upgrade blocked / workloads break after upgrade".** Found during lab 8. Write it up as a failed change, with PIR notes.
- **S03 on the ALB:** the ACM certificate is valid but Route 53 points to an old ALB after a rebuild. The symptom is a TLS hostname error.

## Deliverables

- [ ] `platform/modules/aws-eks`, `platform/modules/k8s-baseline` (add-ons, namespaces, policies), live roots for dev and prod
- [ ] Runway workloads on EKS with RDS, SQS, Redis, ALB/TLS, ESO-delivered secrets, Pod Identity, no static credentials
- [ ] ADR-008, ADR-009, ADR-010, ADR-011
- [ ] `docs/changes/CHG-0xx-ecs-to-eks.md` with measured error rate during cut-over
- [ ] `docs/runbooks/eks-upgrade.md` written from the upgrade you performed; `docs/runbooks/pod-failure-triage.md` extended with EKS-specific causes
- [ ] `docs/lifecycle/eol-register.md` rows: EKS version, each add-on, ingress-nginx (retired → replaced), ECS (decommissioned)
- [ ] Incident records for S13, S14, S15, S36

## Completion criteria

- `make up` (network + EKS + add-ons + workloads) completes in **under 30 minutes** and `make down` leaves zero billable EKS/EC2/ALB/NAT resources (proved with a tag-based resource query).
- A rolling deploy under constant k6 load produces **0 failed requests**. You show the k6 summary.
- The ECS → EKS cut-over happened with measured error rate < 0.1% and a rehearsed rollback path.
- A control plane + add-on + node upgrade completed with traffic flowing and the error rate recorded.
- You diagnose S13's four variants blind, each in under 10 minutes.
- You explain IRSA and Pod Identity token flows on a whiteboard, including the trust policy conditions.

## Mentor checkpoints

- **Before you build:** the node class design, subnet sizing maths (pods per node × nodes × headroom), and the migration plan with rollback. Expect challenges on the rollback trigger. When exactly do you flip the weights back?
- **During:** bring the lab 5 k6 output before and after the `preStop` fix.
- **Questions I will ask:** Why does a pod need a VPC IP here but not on AKS overlay (preview of M14)? What is in the ServiceAccount token, and who validates it? Why is RDS not a StatefulSet? What does EKS Auto Mode take away from you?
- **The failure I will inject:** something that works in dev and fails only in the prod account. Think about what is different between accounts, not between manifests.

## Resources

1. **Amazon EKS User Guide:** *Networking (Amazon VPC CNI, prefix delegation, IP address management)*, *Grant access to Kubernetes APIs (access entries)*, *EKS Pod Identity*, *IAM roles for service accounts*, *Update existing cluster to new Kubernetes version*, *Kubernetes version lifecycle*.
2. **EKS Best Practices** (in the EKS docs): *Networking*, *Reliability (applications, data plane)*, *Cluster Autoscaling → Karpenter*, *Security → IAM*.
3. **Karpenter docs:** *Concepts → NodePools, NodeClasses, Disruption*; *Troubleshooting*.
4. **AWS Load Balancer Controller docs:** *Ingress/Service annotations*, *Pod readiness gate*, and the *Deploy* guide for IAM.
5. ***Production Kubernetes* (Rosso et al.):** ch. *Pod Networking*, *Multitenancy*, *Autoscaling*.

## Estimate vs actual

| | Hours | Cost |
|---|---|---|
| Planned | 50 | ~$60–90 (control plane + nodes + ALB + NAT in session windows) |
| Actual | | |
| Variance note | | |
