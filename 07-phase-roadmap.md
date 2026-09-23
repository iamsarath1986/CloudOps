# Section 7 — Phase-by-Phase Learning Roadmap

This section is the map: 23 phases, each producing a working, operated piece of **Runway**. Every phase has exactly one milestone file under [`milestones/`](milestones/README.md). **The milestone files hold the detail:** concepts, labs, what to build, the failures to cause, completion criteria, resources and mentor checkpoints. This page explains the order.

---

## 1. How this order differs from the one you proposed, and why

Your draft listed 21 phases (0–20). The content is almost all kept. What changes is the **sequence**, because several of your phases depend on things that came *after* them in your list. There are also two new phases and two splits.

| # | Change | Your draft | This roadmap | Why |
|:-:|---|---|---|---|
| 1 | **Azure core moved much later** | Phase 5, right after AWS | **M13**, after EKS, observability and operations | You learn the second cloud best once the concepts are solid, so Azure teaches you the *differences*, not networking for a second time. Azure's strengths are enterprise identity (Entra ID), management-group governance, Arc and Azure OpenAI. Those line up with the later phases, so Azure arrives just before it is needed. See ADR-002 in [Section 5](05-production-project-proposal.md). |
| 2 | **AKS is not a copy of EKS** | Phase 11, straight after EKS | **M14**, paired with multi-cluster GitOps | A second cluster in a second cloud only teaches something new if it raises a new problem. Here the problem is how to deliver to two clusters without cluster credentials in CI. That is Argo CD (ADR-018). Repeating the EKS labs on AKS would be the duplication you asked me to avoid. |
| 3 | **Operations moved before security/governance** | SRE at Phase 14, after security | **M11–M12**, straight after observability | Incidents need telemetry, and they should first happen while you run only one cloud. The JD's operational lines (incidents, problems, *changes*, service levels, EMS) are where you are weakest today, so they come as early as the dependencies allow. |
| 4 | **Reliability split in two** | One SRE phase | **M11** operations (incident, problem, change, SLOs, EMS) + **M12** capacity, HA and DR | The JD names availability, capacity and continuity separately from incident handling. Load testing and a measured DR drill each need weeks of their own. |
| 5 | **FinOps moved before platform engineering** | Phase 17, after the IDP | **M16**, before golden paths | The golden path must *carry* cost allocation: tags, showback and budgets per workload. Cost has to be a design input before you standardise, not a report added afterwards. |
| 6 | **Security basics are never deferred** | Security as Phase 13 | IAM from **M04**, supply chain in **M06**, pod security in **M09**, then the full multi-cloud security phase at **M15** | Retrofitting security is exactly the habit the JD's "solutions meet architectural and security standards" line screens out. M15 is where you *govern* security across two clouds, not where it first appears. |
| 7 | **AI split in two, and kept last** | One AI phase (18) | **M19** AI infrastructure and workloads + **M20** agents and MCP | Running an AI dependency (quotas, fallback, cost, gateway) and building a bounded agent are different skills. Both need the platform underneath them, and the agent needs a history of real incidents to be measured against. |
| 8 | **A first container runtime before Kubernetes (new)** | Straight from Docker to Kubernetes | **M07** CI/CD targets **ECS Fargate**; **M09** migrates it to EKS | This gives you a real "why Kubernetes?" answer (ADR-008). It also makes you do a zero-downtime migration, which is change management and end-of-life replacement, both named in the JD. |
| 9 | **Kubernetes fundamentals are local and short** | Separate phase at full length | **M08**, 3 weeks on `kind`/`k3d`, free | Don't pay for a cloud cluster just to watch a pod start. Spend cloud money on what only a real cluster teaches: node pools, Karpenter, VPC CNI, cloud identity, load balancers and upgrades. |
| 10 | **Docker kept after Terraform, as you proposed** | Phase 7 | **M06** | This works because M04–M05 run `cargo-api` as a **systemd** service on EC2. You learn exactly what a container replaces before you replace it. |
| 11 | **Hybrid stays near the end** | Phase 19 | **M21** | Arc, SSM and VPN cost little but depend on both clouds' identity and governance being in place (M13, M15). It can move ahead of M19 if you need it to. |
| 12 | **A capstone replaces "final architecture"** | Phase 20 | **M22** | This phase ends with the brief's Step 20 test: you get a new requirement and design the whole system on paper, unaided. |

Every milestone file has the same shape. It opens with why the phase exists, then concepts, then architecture decisions written as Problem → Options → Decision → Trade-offs. After that come labs, project work, engineering problems, troubleshooting exercises, deliverables, completion criteria, mentor checkpoints, resources, and a table comparing your estimate with what it actually took.

---

## 2. The phase table

Weeks are build weeks at about 10 h/week. **R** is a revision week, when you build nothing new. The plan assumes 46 productive weeks a year. Month ≈ cumulative week ÷ 3.83. The full timeline is in [Section 22](22-timeline.md).

| Phase | Milestone | Weeks | Ends ≈ month | Cloud spend |
|---|---|:-:|:-:|---|
| M00 | [Engineering foundations and the boring workload](milestones/M00-foundations.md) | 3 | 1 | none |
| M01 | [Linux and networking](milestones/M01-linux-networking.md) | 5 | 2 | none (local VMs) |
| M02 | [Python, Bash and PowerShell automation](milestones/M02-automation.md) | 2 | 2.6 | none |
| M03 | [Cloud architecture fundamentals + first SDD](milestones/M03-cloud-architecture.md) | 2 | 3.1 | none (pricing calculators only) |
| R1 | Revision | 1 | 3.4 | |
| M04 | [AWS core infrastructure, identity-first, built by hand](milestones/M04-aws-core.md) | 5 | 4.7 | low, new-account credits |
| M05 | [Terraform: from first apply to team practice](milestones/M05-terraform.md) | 5 | 6 | low |
| M06 | [Docker, images and the supply chain](milestones/M06-containers.md) | 3 | 6.8 | very low (ECR) |
| M07 | [CI/CD and release engineering (ECS Fargate)](milestones/M07-cicd.md) | 4 | 7.8 | low–medium |
| R2 | Revision · Terraform Associate | 1 | 8.1 | |
| M08 | [Kubernetes fundamentals (local)](milestones/M08-kubernetes-fundamentals.md) | 3 | 8.9 | none |
| M09 | [Production Kubernetes on EKS (+ ECS→EKS migration)](milestones/M09-eks.md) | 5 | 10.2 | medium, session windows |
| R3 | Revision · AWS Solutions Architect – Associate | 1 | 10.4 | |
| M10 | [Observability](milestones/M10-observability.md) | 4 | 11.5 | medium |
| M11 | [SRE and production operations](milestones/M11-sre-operations.md) | 4 | 12.5 | medium, soak windows |
| M12 | [Capacity, high availability and disaster recovery](milestones/M12-capacity-ha-dr.md) | 4 | 13.6 | spikes (Multi-AZ, second region) |
| R4 | Revision · CKA | 1 | 13.8 | |
| M13 | [Azure core, landing zone and enterprise identity](milestones/M13-azure-core.md) | 4 | 14.9 | low–medium, Azure credits |
| M14 | [Production AKS and multi-cluster GitOps](milestones/M14-aks-multicluster.md) | 3 | 15.7 | medium (two clusters) |
| R5 | Revision · AZ-104 | 1 | 15.9 | |
| M15 | [Cloud security and governance (multi-cloud)](milestones/M15-security-governance.md) | 4 | 17 | medium (WAF window) |
| M16 | [FinOps](milestones/M16-finops.md) | 3 | 17.8 | low, and you will lower it |
| R6 | Revision | 1 | 18 | |
| M17 | [Platform engineering: patterns, modules, golden paths](milestones/M17-golden-paths.md) | 4 | 19.1 | medium |
| M18 | [Internal developer platform and self-service](milestones/M18-developer-platform.md) | 4 | 20.1 | medium |
| R7 | Revision · AZ-305 prep window opens | 1 | 20.4 | |
| M19 | [AI infrastructure and AI workloads](milestones/M19-ai-infrastructure.md) | 4 | 21.4 | medium + token budget |
| M20 | [AI agents and MCP for operations](milestones/M20-ai-agents-mcp.md) | 3 | 22.2 | low + token budget |
| R8 | Revision · AWS SA Professional or AZ-305 | 1 | 22.5 | |
| M21 | [Hybrid and multi-cloud architecture](milestones/M21-hybrid-multicloud.md) | 3 | 23.2 | spikes (VPN, resolver endpoints) |
| M22 | [Production hardening, final architecture and capstone](milestones/M22-hardening-capstone.md) | 3 | 24 | medium |
| R9 | Final review · readiness checklist | 1 | 24.3 | |

**84 build weeks + 9 revision weeks = 93 productive weeks ≈ 24 months.** This is not padded. It is the pace that gets you to *operating* and not just *having built*. [Section 22](22-timeline.md) gives a 20-month and a 30-month variant.

---

## 3. The phases, one at a time

Each entry answers five questions. **Purpose:** what capability the phase builds. **Why here:** its place in the dependency chain ([Section 4](04-learning-dependency-map.md)). **Adds to Runway:** the concrete increment. **Unlocks:** what can only start once it is done. **Exit:** the one-line test that says you're finished. The full completion criteria are in each milestone file.

### Foundation block (months 1–3)

**M00 — Engineering foundations and the boring workload** · 3 wks
- **Purpose:** Git as a recovery tool rather than a save button, ADRs, twelve-factor config, and a deliberately boring application.
- **Why here:** everything after this is code in a repository that someone reviews.
- **Adds:** the `runway` monorepo; `cargo-api` (FastAPI, Postgres, OpenAPI, pytest); `carrier-sim`; `Makefile`; branch protection; ADR-001.
- **Unlocks:** M01 and M06.
- **Exit:** `make dev` and `make test` are green, `main` rejects direct pushes, and you can recover lost work from `reflog` without looking it up.

**M01 — Linux and networking** · 5 wks
- **Purpose:** the diagnostic layer under every cloud incident: processes, signals, systemd, filesystems, cgroups, CIDR, routing, DNS, TLS, and TCP failure modes.
- **Why here:** every later incident starts with a request that failed somewhere between a name and a socket.
- **Adds:** health and readiness endpoints, graceful shutdown, structured logs, `net-triage.sh`, a network runbook, and the first draft of the CIDR plan.
- **Unlocks:** M02, M03 and M04.
- **Exit:** you can reproduce and name refused / timeout / reset / hang from `curl -v` alone, and subnet a `/16` on a whiteboard.

**M02 — Python, Bash and PowerShell automation** · 2 wks
- **Purpose:** turn scripts into tools: argument parsing, idempotency, exit codes, retries with jitter, pagination, tests and packaging. Enough PowerShell to read and change an enterprise script.
- **Why here:** you already write Python well. This phase points that skill at operations before a cloud API is involved.
- **Adds:** `platform/scripts/` with a Python CLI package, a hardened Bash template, and a PowerShell module skeleton.
- **Unlocks:** M04 (SDK automation) and M11 (fault injection).
- **Exit:** a tool with tests that another person runs from its README without asking you anything.

**M03 — Cloud architecture fundamentals** · 2 wks
- **Purpose:** vendor-neutral architecture: regions and zones, failure domains, stateless versus stateful design, load balancing, queues, caching, RTO/RPO, and composite availability.
- **Why here:** you should choose services from concepts, not concepts from services. You also write the first Solution Design Document here, before spending any money.
- **Adds:** `docs/solution-designs/SDD-001-runway-v1.md`, the architecture diagram v0, a pricing-calculator estimate, and ADR-002 (AWS first).
- **Unlocks:** M04, and later M13.
- **Exit:** you can defend every box in your v0 diagram against "what happens when this fails?"

### AWS block (months 4–8)

**M04 — AWS core infrastructure, identity-first, by hand** · 5 wks
- **Purpose:** Organizations, OUs and accounts; IAM Identity Center; IAM policy evaluation; VPC with public/private/data subnets across 3 AZs; EC2 running `cargo-api` under systemd behind an ALB; RDS; S3; Route 53; ACM; budgets.
- **Why here:** you need the mental model before the code. Terraform written by someone who has never clicked the thing is Terraform they cannot debug.
- **Adds:** a live, hand-built, TLS-fronted `cargo-api` in `workloads-dev`, `manual-build-log.md`, ADR-003 and ADR-004.
- **Unlocks:** M05.
- **Exit:** you can diagnose an AccessDenied (S04) from the policy evaluation chain, and a timeout (S02) from security groups and routes, without being told what you broke.

**M05 — Terraform: from first apply to team practice** · 5 wks
- **Purpose:** state, locking (S3 native `use_lockfile`), modules, `for_each`, `lifecycle`, `moved`, `import`, drift, tests, linting, scanning, and the `live/` layout.
- **Why here:** you now know what the resources are, so you can codify them. Everything after this is only ever created by code.
- **Adds:** `platform/modules/aws-*`, `platform/live/aws/...`, `make up` / `make down`, ADR-005. The hand-built environment is destroyed.
- **Unlocks:** M07, and every later infrastructure phase.
- **Exit:** `make down && make up` rebuilds dev from nothing, and you can spot a destructive plan (S07) before applying it.

**M06 — Docker, images and the supply chain** · 3 wks
- **Purpose:** namespaces, cgroups, layers, multi-stage builds, non-root, PID 1 signals, digests, scanning, SBOM and signing.
- **Why here:** you have run `cargo-api` under systemd, so you know what the container is replacing. It only depends on M00, so it can move earlier if cloud spend has to pause.
- **Adds:** hardened Dockerfiles, images in ECR by digest, Trivy/Syft/cosign in the `Makefile`, ADR-006.
- **Unlocks:** M07 and M08.
- **Exit:** you can state your image's size, user, CVE count and triage decision, and diagnose S08 and S09 from exit codes and logs.

**M07 — CI/CD and release engineering (ECS Fargate target)** · 4 wks
- **Purpose:** build once and promote by digest, the automated test pyramid, OIDC to AWS, environments and approvals, rolling versus blue-green, measured rollback, expand-contract migrations, plan-on-PR / apply-on-merge, and the first change records.
- **Why here:** it needs an artefact (M06) and code-defined infrastructure (M05).
- **Adds:** reusable workflows `build-sign-push`, `deploy`, `tf-plan-apply`, `drift-detect`, `change-record`; `cargo-api` on ECS Fargate in dev and prod; ADR-007.
- **Unlocks:** M09 and every later delivery.
- **Exit:** a measured rollback time is in the README, and the pipeline can no longer report green while users get 5xx (S10).

### Kubernetes and operations block (months 9–14)

**M08 — Kubernetes fundamentals (local)** · 3 wks
- **Purpose:** the reconciliation loop, the main objects, probes, requests and limits, scheduling, RBAC, NetworkPolicy, Helm versus Kustomize. Also the ingress-nginx retirement as a live end-of-life example.
- **Why here:** it's free on `kind` and needs only containers.
- **Adds:** manifests and charts for the workloads; `pod-failure-triage.md` v1.
- **Unlocks:** M09.
- **Exit:** you can cause and diagnose all of S12 in under five minutes each, from `kubectl` alone.

**M09 — Production Kubernetes on EKS** · 5 wks
- **Purpose:** VPC CNI and IP planning, managed node groups plus Karpenter, Pod Identity and IRSA, the AWS Load Balancer Controller, Gateway API, External Secrets, PDBs, HPA, upgrades, and a zero-downtime ECS→EKS migration.
- **Why here:** it needs cloud networking (M04), IaC (M05), pipelines (M07) and Kubernetes concepts (M08).
- **Adds:** `aws-eks` module, cluster add-ons, all workloads on EKS, ECS decommissioned, ADR-008 to ADR-011, the EOL register started.
- **Unlocks:** M10, M13 and M14.
- **Exit:** a cluster version upgrade with traffic flowing and zero failed requests, plus S13–S15 diagnosed unaided.

**M10 — Observability** · 4 wks
- **Purpose:** OpenTelemetry, RED/USE, cardinality, percentiles, logs, metrics and traces across services and queues, symptom-based alerting, and dashboards as code. Uses AMP, Loki, Tempo and Grafana, with CloudWatch kept.
- **Why here:** you cannot operate what you cannot see. M11's incidents without it would just be confusion.
- **Adds:** the OTel Collector, the observability stack, four dashboards, a saved query library, alert rules with runbooks, ADR-012.
- **Unlocks:** M11 and M16. (M19 and M20 depend on it heavily.)
- **Exit:** you can find a latency regression from traces, confirm it with metrics and detail it with logs (S16), and an absence-of-data alert exists (S17).

**M11 — SRE and production operations** · 4 wks · **never cut**
- **Purpose:** incident, problem and **change** management; SLIs, SLOs and error budgets; the L1/L2 alert tiers with the `ems-bridge` (the JD's EMS); severity matrix; PIRs; monthly operational review; DORA metrics.
- **Why here:** the telemetry exists and there is one cloud to reason about. This is the half of the JD most candidates cannot evidence.
- **Adds:** `docs/incidents/`, `docs/problems/`, `docs/changes/`, `inject-fault.sh`, `ems-bridge`, two SLOs with burn-rate alerts, ADR-013.
- **Unlocks:** M12, and all later operations.
- **Exit:** at least 8 incident records written *during* drills, 2 RCAs linking repeat incidents, and one failed change worked through to a PIR (S35).

**M12 — Capacity, high availability and disaster recovery** · 4 wks
- **Purpose:** open-model load testing, Little's Law, saturation, HPA and Karpenter tuning, Multi-AZ, PDBs and topology spread, backup versus replication, PITR, pilot light into a second region, measured RTO/RPO.
- **Why here:** you need SLOs to know what "enough capacity" means, and incident practice to run a drill properly.
- **Adds:** k6 scripts and reports, a capacity plan, the `backup-verify` Lambda, DR runbooks with measured timings, ADR-014.
- **Unlocks:** M15 and M17 (the golden path inherits these defaults).
- **Exit:** RTO and RPO measured in two executions of the DR drill, plus a stated max sustainable request rate and the resource that limits it.

### Second cloud and control block (months 15–18)

**M13 — Azure core, landing zone and enterprise identity** · 4 wks
- **Purpose:** Entra ID, management groups, subscriptions, Azure RBAC, hub-spoke VNet, private endpoints and private DNS, and Key Vault. Entra becomes the workforce IdP for AWS too, and pods get cross-cloud workload identity.
- **Why here:** the concepts are already solid, so Azure teaches the differences and the enterprise control plane.
- **Adds:** `azure-*` modules, the `live/azure/...` roots, Entra→IAM Identity Center federation, ADR-015 to ADR-017.
- **Unlocks:** M14 and M21.
- **Exit:** a pod on EKS calls an Entra-protected API with no stored secret, and you can diagnose S24 and S25 unaided.

**M14 — Production AKS and multi-cluster GitOps** · 3 wks
- **Purpose:** AKS specifics (networking modes, workload identity, node pools, upgrade channels), Envoy Gateway, and Argo CD delivering to both clusters.
- **Why here:** it is the first time Runway has two clusters, which is the problem GitOps solves.
- **Adds:** `azure-aks`, `platform/gitops/` app-of-apps, the portal BFF running on AKS, ADR-018.
- **Unlocks:** M15 and M17.
- **Exit:** one Git commit changes both clusters, CI holds no cluster credentials, and drift is reverted automatically (S26).

**M15 — Cloud security and governance (multi-cloud)** · 4 wks
- **Purpose:** threat modelling, SCPs, Azure Policy (audit before deny), conftest on plans, Kyverno at admission, image signature verification, egress control, WAF, rotation and the leaked-credential drill.
- **Why here:** there are now two clouds and two clusters to govern, and the control points are known.
- **Adds:** `platform/policy/`, the threat model and controls matrix, ADR-019.
- **Unlocks:** M16 and M17.
- **Exit:** a policy has blocked a real deployment of yours, and S27 is worked end to end: contain, rotate, assess, prevent.

**M16 — FinOps** · 3 wks
- **Purpose:** tagging enforced by policy, budgets and anomaly detection, FOCUS exports from both clouds, OpenCost, unit economics, rightsizing, commitment concepts.
- **Why here:** you need tags (M15) and telemetry (M10), and the golden path (M17) must carry cost allocation.
- **Adds:** `finops-collector`, the `idle-detector` Lambda, a cost dashboard, a monthly cost review, ADR-020.
- **Unlocks:** M17.
- **Exit:** a 40% bill jump (S29) is traced to a resource, a change and a decision, with before/after numbers.

### Platform block (months 19–20)

**M17 — Platform engineering: patterns, modules, golden paths** · 4 wks · **never cut**
- **Purpose:** platform as a product, module design as API design, the `workload.yaml` contract, reusable workflows, standard telemetry and SLOs, versioning and consumer migration.
- **Why here:** you can only standardise what you have already built, operated, secured and costed.
- **Adds:** the golden-path template repository, the `workload` module, `hello-svc` onboarded to *both* clouds, ADR-021.
- **Unlocks:** M18 and M19.
- **Exit:** `hello-svc` onboarded cross-repo to AWS and Azure without touching platform code, with the elapsed time measured.

**M18 — Internal developer platform and self-service** · 4 wks
- **Purpose:** service catalogue, data catalogue, and the portal (React SPA on S3+CloudFront, Node/TypeScript BFF on AKS). Request forms that **open PRs**. Account vending, showback, and the change calendar.
- **Why here:** the portal is a front end to the golden path, so the path has to exist first.
- **Adds:** `runway-portal`, the `catalog/` directory, the sandbox account vending flow, ADR-022.
- **Unlocks:** M19.
- **Exit:** a stranger requests a service and a sandbox account through the portal and gets them through review and pipeline, with no ticket and no direct cloud mutation.

### AI, hybrid and capstone block (months 21–24)

**M19 — AI infrastructure and AI workloads** · 4 wks
- **Purpose:** the `ai-gateway` (quotas, cache, metering, fallback Azure OpenAI → Bedrock → degraded), async AI through SQS, evals as a pipeline gate, and a self-hosted CPU model lab.
- **Why here:** an AI service is a workload. It goes through the golden path with SLOs, a budget and a runbook like everything else.
- **Adds:** `ai-gateway`, `insights-worker`, AI dashboards, ADR-023 and ADR-024.
- **Unlocks:** M20.
- **Exit:** during S31 the fallback engages and shows on the dashboard, and cost per AI summary is a measured number.

**M20 — AI agents and MCP for operations** · 3 wks
- **Purpose:** `flightdeck`, a remote read-only MCP server plus a bounded agent. Three independent read-only mechanisms, tool-result injection testing, and scoring against your own incident history.
- **Why here:** by now you have 20+ worked incidents, so you know which queries you always run first. They are the specification and the baseline.
- **Adds:** `flightdeck/`, its evals, ADR-025.
- **Unlocks:** M22.
- **Exit:** a measured time-to-first-hypothesis before and after, with the wrong answers reported honestly (S33).

**M21 — Hybrid and multi-cloud architecture** · 3 wks
- **Purpose:** a local "datacentre" (VM + k3s) connected through Arc and SSM, a timeboxed AWS↔Azure site-to-site VPN, hybrid DNS, and end-of-life migration planning.
- **Why here:** it needs both clouds' identity and governance (M13, M15). It can move ahead of M19 if you need it to.
- **Adds:** the hybrid runbooks, ADR-026, datacentre entries in the EOL register.
- **Unlocks:** M22.
- **Exit:** one Azure Policy initiative governs cloud and "datacentre" resources, and you can diagnose S34 unaided.

**M22 — Production hardening, final architecture and capstone** · 3 wks
- **Purpose:** a security and reliability review of the whole system, a final game day, the final SDD, ADR-027, and the Step 20 capstone: design a new system from a one-line requirement.
- **Why here:** it's the last phase because it reviews everything.
- **Adds:** the final architecture pack, the readiness checklist completed ([Section 25](25-production-readiness-checklist.md)), and the portfolio README numbers.
- **Unlocks:** the job search, and the specialisation you choose next.
- **Exit:** you pass the capstone design review and the game day with only runbooks, and every README number can be reproduced live.

---

## 4. Rules that apply to every phase

1. **A phase is finished when its completion criteria pass**, not when the code runs. Say the answers out loud. If you hedge, you are not finished.
2. **Every phase leaves something in Git**, whether built, operated, measured, secured or broken-and-fixed. Nothing is theoretical.
3. **The deliberate failures are the phase.** Skip the build polish before you skip a failure exercise.
4. **Write incident timelines while the incident is happening.** Reconstructed timelines are fiction.
5. **`make down` at the end of every session**, apart from planned soak windows. Your budget discipline and your IaC discipline are the same habit.
6. **Fill in the estimate-vs-actual table.** Estimating effort and cost accurately is the JD's "guide Project Managers… agreed costs and timescales", and you only get better at it by measuring your misses.

→ Next: [Section 8 — Detailed Milestones](milestones/README.md)
