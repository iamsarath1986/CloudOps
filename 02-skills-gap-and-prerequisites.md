# Section 2 — Skills Gap and Prerequisite Analysis

## 1. How to read this

Every skill is scored on one scale, defined by **what you can demonstrate**, never by what you have read:

| Level | Name | The test |
|:-:|---|---|
| **L0** | No knowledge | You couldn't explain what it's for |
| **L1** | Understand concepts | You can explain what it is, what problem it solves and roughly how it works, without notes |
| **L2** | Implement with guidance | You can make it work by following a guide or with someone reviewing. You couldn't yet do it again from scratch |
| **L3** | Implement independently | You can build it unaided for a normal case and debug your own mistakes |
| **L4** | Operate in production | You've run it over time: upgraded it, recovered it from failure, paid for it, changed it under pressure without breaking it |
| **L5** | Design, troubleshoot and teach others | You set the standard others follow, make the trade-offs across teams, and can teach it at a whiteboard |

**Now** is my read of where you are today. It's based on your repositories (Python, FastAPI, PostgreSQL, Docker, testing, migrations, RAG/LLM projects) and your own statement that you haven't operated production cloud infrastructure. **Target** is the level you should reach by month ~24 ([Section 22](22-timeline.md)). **Where** is where the gap closes.

**How levels move:** reading gets you to L1. Building gets you to L2–L3. Breaking something and fixing it gets you to solid L3. Running it across months gets you to L4. Designing it for others, defending the design and teaching it gets you to L5.

---

## 2. The gap table

### Foundations

| Skill | Now | Target | Gap | Where it closes | What moves you up a level |
|---|:-:|:-:|:-:|---|---|
| Linux administration & internals | L2 | L4 | 2 | M01, then every incident | Diagnose disk/inode exhaustion, signal handling, a bound port and DNS failures from a cold shell |
| Networking (TCP/IP, DNS, TLS, routing, CIDR) | L1 | L5 | **4** | M01, M04, M09, M13, M21 | Name the failing hop in a cross-cloud request; a CIDR plan for two clouds plus a datacentre that you defended |
| Python (services + automation) | L3 | L5 | 2 | M00, M02, M10, M19 | Tooling other people run: typed, tested, packaged, and handling throttling and pagination properly |
| Bash | L2 | L4 | 2 | M01, M02 | `net-triage.sh` and `inject-fault.sh` used in real drills |
| PowerShell | L0–L1 | L3 | 2–3 | M02, M13 | One maintained Az-module tool; can read and modify others' scripts |
| Git | L2 | L4 | 2 | M00 onward | Recovery from reflog, bisect, protected-branch strategy for many repos |
| API / backend design | L3 | L5 | 2 | M00, M17, M19 | OpenAPI contract tests; a breaking change shipped without breaking a consumer; an API other teams use (ai-gateway) |
| TypeScript / React / Node | L1 | L3–L4 | 2–3 | M18 | The portal's SPA and BFF deployed, observed and upgraded by you |
| Distributed-systems fundamentals | L1–L2 | L4 | 2–3 | M03, M11, M12 | Stability patterns implemented and justified with measurements |

### Cloud

| Skill | Now | Target | Gap | Where it closes | What moves you up a level |
|---|:-:|:-:|:-:|---|---|
| Cloud architecture (vendor-neutral) | L1 | L5 | **4** | M03 onward, M22 capstone | An SDD you wrote, costed and defended; ADRs with rejected options argued properly |
| AWS core (Organizations, IAM, VPC, EC2, ALB, RDS, S3, Route 53) | L0–L1 | L4–L5 | **4** | M04, M05, M09, M12 | A multi-account estate run for over a year; a bill you explained; a quota you hit |
| Azure core (MGs, subscriptions, Entra ID, RBAC, VNet, private endpoints, Policy) | L0–L1 | L4 | **3–4** | M13, M14, M15, M21 | A landing zone with inherited policy; private-DNS failures diagnosed |
| IAM & identity federation (both clouds) | L1 | L5 | **4** | M04, M07, M09, M13 | Zero static credentials anywhere, including cross-cloud; a failed deploy diagnosed from the policy-evaluation chain alone |
| Large-footprint patterns (landing zones, vending) | L0 | L4 | **4** | M04, M13, M17, M18 | Account vending through a PR; guardrails that new accounts inherit |
| Hybrid (Arc, SSM, VPN, hybrid DNS) | L0 | L3 | 3 | M21 | A local VM and a k3s cluster governed by the same policy as cloud resources |

### Delivery

| Skill | Now | Target | Gap | Where it closes | What moves you up a level |
|---|:-:|:-:|:-:|---|---|
| Terraform / IaC | L0–L1 | L5 | **4–5** | M05, then everything | Versioned modules consumed by other roots; state recovered; drift reconciled; a `moved` refactor with no destruction |
| CI/CD & release engineering | L2 | L5 | 3 | M07, M14, M17 | Build-once-promote; measured rollback; a mandatory stage added without touching consumer repos |
| Automated testing (app + infra) | L2–L3 | L4 | 1–2 | M00, M05, M07 | `terraform test`, contract tests, and a performance gate that has actually blocked a merge |
| Docker & supply chain | L2–L3 | L4 | 1–2 | M06, M15 | Signed, SBOM'd, scanned images, verified at admission |
| Kubernetes (core) | L1 | L5 | **4** | M08, M09, M14 | Six pod failure states diagnosed in under 5 min; a live upgrade with traffic flowing |
| EKS | L0 | L4 | **4** | M09, M12 | VPC CNI IP exhaustion diagnosed; Karpenter tuned; Pod Identity broken and fixed |
| AKS | L0 | L4 | **4** | M14 | Workload Identity, node pools and upgrades operated; GitOps across both clusters |
| GitOps (Argo CD) | L0 | L3–L4 | 3–4 | M14, M17 | Out-of-sync and drift incidents handled; app-of-apps structure you designed |

### Operations

| Skill | Now | Target | Gap | Where it closes | What moves you up a level |
|---|:-:|:-:|:-:|---|---|
| Monitoring, logging, tracing | L1 | L5 | **4** | M10 | Telemetry supplied by the platform to every workload; a latency regression found from traces |
| Incident management | L0–L1 | L4 | **3–4** | M11 onward | 15+ incident records with timelines written during the incident |
| Problem management | L0 | L4 | **4** | M11 onward | RCAs that link repeat incidents, with permanent fixes that stopped them |
| Change management | L0 | L4 | **4** | M07, M11 | Change records generated from PRs; a failed change with a post-implementation review; a measured change failure rate |
| Event management (L1/L2, "EMS") | L0 | L3–L4 | 3–4 | M11 | Alert taxonomy and routing; auto-created incident records |
| SLOs / error budgets | L0–L1 | L4 | 3–4 | M10, M11 | An error budget that changed a real release decision |
| Capacity & performance | L1 | L4 | 3 | M12 | A measured ceiling, the resource that limits it, and a capacity plan |
| HA & DR | L0–L1 | L4 | **3–4** | M12 | RTO/RPO measured in two drills; a pilot-light region rebuilt from code |
| Troubleshooting across layers | L2 | L5 | 3 | Every milestone; [Section 10](10-failure-and-troubleshooting.md) | 36 scenario IDs worked, with the rejected hypotheses recorded |
| Lifecycle / EOL | L0 | L4 | **4** | M09, M14, M21 | EOL register kept current; ingress-nginx replaced; two Kubernetes upgrades performed |

### Governance, cost, platform, AI

| Skill | Now | Target | Gap | Where it closes | What moves you up a level |
|---|:-:|:-:|:-:|---|---|
| Cloud security & threat modelling | L1 | L4 | 3 | M04, M06, M15, M22 | STRIDE model with a controls matrix; the leaked-credential drill |
| Governance & policy-as-code | L0 | L4 | **4** | M15 | Policy that blocked real deploys at four enforcement points; an exception process |
| FinOps | L0–L1 | L4 | **3–4** | M04 onward, M16 | A bill anomaly on your own money, found and fixed; FOCUS-normalised cost across both clouds |
| Platform engineering / golden paths | L0–L1 | L5 | **4** | M17 | A workload you didn't write onboarded to both clouds, touching no platform code, timed |
| Internal developer portal / self-service | L0 | L4 | **4** | M18 | Self-service requests that open PRs, pass policy and provision |
| Documentation & data catalogue | L2 | L5 | 3 | Every milestone; [A2](appendix/A2-documentation-standards.md) | Runbooks a stranger executed; a data catalogue the portal renders |
| Solution architecture & estimation | L1 | L4–L5 | 3–4 | M03, M12, M22 | SDDs with estimates, and estimates tracked against actuals across 23 milestones |
| AI infrastructure (gateway, quotas, fallback, cost) | L1–L2 | L4 | 2–3 | M19 | ai-gateway under SLOs with cross-cloud fallback; cost per AI request known |
| AI agents & MCP | L1–L2 | L3–L4 | 2 | M20 | A read-only MCP server with three independent read-only mechanisms; injection tested |

Your existing AI work (RAG, LangChain and LLM API projects) puts you *ahead* on AI concepts. That's exactly why AI waits until month ~21: it's the area where you least need the time.

---

## 3. The five gaps that matter most

Almost everything above comes down to five statements. Close these and you're a credible senior candidate. Leave any one open and an experienced panel will find it.

1. **You have never operated cloud infrastructure you owned, in either cloud.** You close this by keeping a real multi-account AWS estate and an Azure landing zone alive, paid for and changed across more than a year. Deleting them after a weekend doesn't count. Closed by M04–M14, and kept closed by every milestone after.
2. **You have never been accountable when something broke.** Incident, problem and change management are part of the role's definition. You close this with 15+ incident records, 5+ RCAs and change records with a real failed change, all from drills you ran against a system real enough to break. Closed by M11, and by every scenario in [Section 10](10-failure-and-troubleshooting.md).
3. **You have never designed for other people to adopt.** "Drive the adoption of", "internal developer tools", "self-service portals". Success is measured by someone else's time to first deploy, not by your own. Closed by M17–M18.
4. **You have no evidence of working across two clouds.** The JD says AWS *and* Azure. Being able to translate isn't enough. You need to have run workloads in both, joined by federated identity, one GitOps control plane and one telemetry pipeline. Closed by M13–M14, M21.
5. **You have never written a design someone else signed off.** Senior means SDDs, ADRs, cost estimates and EA-style standards. Closed by the SDD in M03, the design reviews in every milestone's mentor checkpoint, and the M22 capstone.

Your strengths (Python, API design, PostgreSQL, Docker, testing) give you a real head start. They're why this roadmap can spend its hours on infrastructure and operations rather than on learning to write a service.

---

## 4. Prerequisite chains

Some skills can't be learned properly until others are in place. Each chain below reads *left must be at least L3 before right is worth starting*. The full dependency map is [Section 4](04-learning-dependency-map.md).

| Chain | Why the order is forced |
|---|---|
| **Linux → containers → Kubernetes → EKS/AKS** | A container is a Linux process with namespaces and cgroups. Without that, OOMKilled, exit 137 and ignored SIGTERM are mysteries rather than diagnoses |
| **Networking → VPC design → EKS networking → cross-cloud connectivity** | VPC CNI IP exhaustion, private-endpoint DNS and VPN routing are all CIDR, routing and DNS problems in cloud clothing |
| **Identity concepts (OAuth2/OIDC) → AWS IAM → CI OIDC federation → Pod Identity/IRSA → cross-cloud federation** | Each step reuses the previous step's trust model. Cross-cloud federation is just OIDC trust between issuers you already understand |
| **Manual cloud build → Terraform → CI/CD → GitOps** | You can't debug Terraform for a thing you've never built by hand, and you can't design GitOps without knowing what push pipelines get wrong |
| **Observability → incident management → problem management → SLO-driven change** | An incident against a system with no telemetry is just confusion. RCAs need records. Error budgets need SLIs |
| **Terraform modules + CI/CD + policy + observability → golden paths → developer portal** | A golden path is these four packaged as a product. The portal is a front door onto the golden path, not a replacement for it |
| **Tagging (M04) → policy enforcement (M15) → cost allocation (M16) → showback in the portal (M18)** | Untagged cost can't be attributed, and unenforced tags don't exist |
| **Operations experience (M11) → AI ops assistant (M20)** | The agent automates the first five queries you always run. You only know which five after running fifteen incidents by hand |
| **AWS depth → Azure** | The concepts transfer. Azure is then learned as *differences*: identity-first control plane, management groups, private endpoints, policy |

---

## 5. What doesn't need closing

Be deliberate about what you *don't* study:

- **Model training, fine-tuning, ML pipelines.** Not in the JD, not your goal.
- **Service-mesh internals.** Explain what one would give you and what it costs to run. Don't install one.
- **GCP.** Two clouds at depth beats three at tutorial level.
- **Front-end craft beyond internal tooling.** The portal needs to be correct, accessible and maintainable, not beautiful.

→ Next: [Section 3 — Target Engineer Profile](03-target-engineer-profile.md)
