# Section 5 — Production Project Proposal: **Runway**

## 1. The idea in one paragraph

**Runway is a small internal cloud platform that spans AWS and Azure, plus the workloads that run on it.** It provides one paved road from a Git repository to a running service that is observed, governed, cost-attributed and recoverable, on either cloud. The platform covers:

- landing zones on both clouds
- versioned Terraform modules and reusable CI/CD workflows
- two Kubernetes clusters (EKS and AKS) delivered by GitOps
- a shared observability stack and an event-management layer that routes L1 and L2 alerts
- policy guardrails and a cost model
- a self-service portal where an engineer requests a service, an environment or a sandbox account and gets a reviewed pull request instead of a ticket

A deliberately boring cargo-tracking application runs on it. A tiny third-party-style service proves that onboarding works on both clouds without touching platform code. Two AI components arrive last. The first is a shared **AI gateway**, the platform's governed, metered, fallback-capable access layer to managed models. The second is **Flightdeck**, a strictly read-only operations agent that queries the platform's telemetry through MCP tools to shorten a human's investigation. Neither can change infrastructure.

**The platform is the subject. The application stays boring.**

## 2. Why this project, and not the obvious alternatives

| Candidate | Why it is rejected |
|---|---|
| "Deploy a three-tier app to AWS with Terraform" | Covers maybe a third of the JD. It has no Azure, no onboarding, no self-service, no governance across footprints, no change or problem management and no FinOps. It is the project everyone builds, so it does not stand out. |
| "The same app, built twice: once on AWS, once on Azure" | Duplicates every exercise, which your brief explicitly rejects. You learn the second cloud as a vocabulary list rather than as a different control plane. |
| "Build an AI agent for DevOps" | Inverts the JD's priorities. AI is *advantageous*; operating AWS and Azure platforms is *required*. An AI-first portfolio reads as an AI candidate with weak infrastructure. |
| "Kubernetes home lab" | Free and useful, but there is no bill (so no FinOps), no cloud identity, no landing zone and no managed services. It evidences almost nothing on the cloud side. |
| "A multi-region, active-active global system" | Unachievable at 10 h/week, expensive, and the JD does not ask for it. Senior engineers are recognised by what they chose not to build. |
| **Runway** | The platform *is* the product, which matches the team you are applying to ("Platform Engineering team, the backbone of our cloud modernization journey"). Both clouds have a *reason* to exist in it. Operations are structural, not decorative. AI arrives last and is scoped to what the JD asks for. |

**The governing constraint:** `cargo-api` is finished in M00 and then barely extended. If you find yourself adding business features, you have drifted. Every hour spent on application logic is an hour not spent on what you are being hired for.

## 3. The components

Each one exists because it solves a specific engineering problem, and each one is a teaching device. Anything that could not say which problem it solves was deleted.

### 3.1 The platform: `runway-platform` (the real subject)

| Part | The problem it solves |
|---|---|
| **Landing zones**: AWS Organizations (OUs: `management`, `security`, `shared-services`, `workloads-dev`, `workloads-prod`, `sandbox`); Azure management groups `runway → platform / landing-zones / sandbox` over two subscriptions | "Operating large public cloud footprints" means many accounts and subscriptions under one set of guardrails. A small multi-account structure teaches the same blast-radius, billing and policy-inheritance lessons as a large one. |
| **Account/subscription vending** (Terraform, requested via portal PR) | Onboarding at scale starts with "give this team an isolated, governed place to work", not with a pod. |
| **Terraform modules + live roots** | Reproducible infrastructure with small states and small blast radii. Reviewed plans, drift detection and the ability to rebuild from nothing. |
| **Reusable CI/CD workflows** (`build-sign-push`, `deploy`, `tf-plan-apply`, `drift-detect`, `change-record`) | One improvement reaches every consumer. The test: *can you add a mandatory pipeline stage without touching any workload repo?* |
| **EKS (AWS) and AKS (Azure)**, add-ons delivered by **Argo CD** | The JD names both. Two clusters in two clouds are what make GitOps genuinely justified rather than fashionable (ADR-018). |
| **Observability stack**: OTel Collector, Amazon Managed Service for Prometheus, Loki, Tempo, Grafana; CloudWatch and Azure Monitor as native sources | One place to answer "what is broken, where, since when" across both clouds. Built as code. |
| **Event management (`ems-bridge`)** and L1/L2 alert tiers | Models the JD's "Level 1 and Level 2 incidents are configured within EMS". Alerts become incident records automatically, each with an owner tier and a runbook. |
| **Policy**: SCPs, Azure Policy, Kyverno, conftest | Guardrails at four enforcement points, so autonomy can be granted safely. |
| **FinOps**: tags, budgets, anomaly detection, FOCUS exports, OpenCost, `finops-collector` | Cost per service, per team and per environment across both clouds, on your own money. |
| **Golden path**: template repo, `workload.yaml`, `workload` module | Onboarding in hours without platform-team involvement. The requirement almost nobody can evidence. |
| **Catalogues**: service catalogue (Backstage-format `catalog-info.yaml`) and data catalogue (`catalog/data/*.yaml`) | The JD's "Publish and maintain … a data catalogue for the technology area". Who owns what, what data lives where, its classification, retention and recovery point. |
| **Docs**: ADRs, SDDs, runbooks, incidents, problems, changes, EOL register, estimates | The JD asks for detailed designs, support procedures, operations manuals and deployment architecture. Here they are part of the deliverable. |

### 3.2 Workloads

| Component | Where | The problem it exists to teach |
|---|---|---|
| **`cargo-api`**: Python/FastAPI, OpenAPI contract, PostgreSQL (RDS), Redis cache, publishes SQS jobs | EKS | Every operationally interesting property in one boring service: stateful dependency (backup, restore, PITR, connection exhaustion), a read-heavy endpoint (caching, load testing, autoscaling), async work (queues, DLQs, backpressure), and schema migrations (expand–contract, release safety). |
| **`carrier-sim`**: a fake external carrier API with latency and failure toggles | EKS | A dependency you control, so you can make it slow, flaky or down on demand. Timeouts, retries, circuit breakers, cascading failure. |
| **`eta-worker`**: SQS consumer that recomputes ETAs | EKS | Queue semantics, visibility timeouts, poison messages, DLQs, scaling on queue depth. |
| **`runway-portal`**: React SPA on S3 + CloudFront; Node/TypeScript BFF on AKS with Entra ID SSO | Both | The JD's "strong full-stack development" and "self-service portals". The front door to the platform. The BFF holds credentials; the browser never does. |
| **`hello-svc`**: forty lines, own repository | EKS **and** AKS | A timed proof: onboard a workload you did not write, to *both* clouds, through the golden path, touching no platform code. |
| **`ai-gateway`** (M19): Python/FastAPI | AKS | The platform's shared, governed LLM access layer. Per-team quotas, token metering and cost attribution, caching, timeouts, a circuit breaker, and routing Azure OpenAI → Amazon Bedrock → non-AI fallback. |
| **`insights-worker`** (M19): SQS consumer | EKS | Produces structured summaries and classifications of shipment exceptions via `ai-gateway`. It is async because LLM latency and quotas vary. Cross-cloud call with federated identity. |
| **`flightdeck`** (M20): remote MCP server + bounded agent | AKS | Read-only operations assistant. Evidence for the JD's "AI agents" and "MCP integrations" in the most defensible form there is: bounded, audited and measured. |
| **`finops-collector`** (M16): CronJob | EKS | Normalises AWS and Azure FOCUS cost data and publishes showback. |
| **Lambda jobs**: `backup-verify`, `idle-detector` | AWS | Scheduled platform jobs where serverless is the correct choice: short, periodic, event-free. No pod needs to exist for them. |

### 3.3 Why both clouds have a job (ADR-015)

Workloads are not split across clouds for coverage. Each cloud hosts what it is best placed to host in an enterprise like the target employer:

- **AWS / EKS** hosts the transactional system: `cargo-api`, its workers, RDS, SQS and the shared observability stack. It is the less-managed Kubernetes and the more explicit networking model, which is why it comes first.
- **Azure / AKS** hosts the enterprise-identity-facing and AI tier: the portal BFF (next to Entra ID, the workforce IdP), `ai-gateway` (next to Azure OpenAI over a private endpoint) and `flightdeck`.
- The two meet in three places, all without a static credential: `insights-worker` → `ai-gateway`, `ai-gateway` → Bedrock (fallback), and AKS → AMP (metrics remote-write). That is the recurring lesson of the architecture: **federated identity across cloud boundaries, never keys** (ADR-017).

## 4. How the project maps to the job description

| JD requirement (quoted) | Where it is built | Evidence produced |
|---|---|---|
| "Proficiency in managing and operating AWS and Azure cloud platforms" | M04–M16, M21, continuously | Two live footprints operated for over a year, two bills explained, incident records on both |
| "operating large public cloud footprints and onboarding applications using governed and automated DevOps practices" | M04, M13, M15, M17, M18 | Multi-account/subscription landing zones, account vending by PR, `hello-svc` onboarded to both clouds, timed |
| "Security, Governance, Resilience, FinOps" | M12, M15, M16 | Threat model and controls matrix, policies that blocked real deploys, measured RTO/RPO, a cost anomaly with before/after numbers |
| "Develop solution architecture for assigned projects in accordance with established EA standards" | M03, M12, M22 | Solution Design Documents to a template, 27 ADRs with rejected options argued properly |
| "meets service-level requirements in terms of expected availability, capacity, continuity, security, performance, and reliability" | M10–M12 | SLOs with burn-rate alerts, capacity plan with a measured ceiling, DR drills |
| "ensure Level 1 and Level 2 incidents are configured within EMS" | M11 | Alert taxonomy, L1/L2 routing, `ems-bridge` auto-creating incident records |
| "incidents are resolved within agreed service levels … Problem Management Process" | M11 onward | 20+ incident records, 5+ problem records linking repeat incidents to one cause |
| "Own and manage changes within the technology area" | M07, M11, M13 | Change records from PR + deployment, a freeze window, a failed change with a PIR, change failure rate tracked |
| "documentation and a data catalogue … detailed designs, configuration details, support procedures, operations manuals, and deployment architecture" | Every milestone; M18 | `docs/`, `catalog/`, the data catalogue rendered in the portal |
| "Manage end-of-life replacement of equipment and/or software" | M09, M14, M21 | EOL register, ingress-nginx replacement, EKS/AKS version upgrades, ECS→EKS migration |
| "Guide Project Managers … according to agreed costs and timescales" | Every milestone | `docs/estimates/`: estimate versus actual for hours and cost, per phase |
| "Strong full-stack development … Python, TypeScript/JavaScript, APIs, React, Node.js" | M00, M18, M19 | `cargo-api`, `runway-portal`, `ai-gateway`: built, deployed and operated by you |
| "REST APIs, Backend services, Integrations, Web-based applications" | M00, M18, M19 | OpenAPI contracts, contract tests, cross-cloud integrations |
| "Docker, Kubernetes, Amazon EKS, Azure AKS" | M06, M08, M09, M14 | Hardened signed images, two production-style clusters upgraded in place |
| "Infrastructure as Code … Terraform … Python, Bash, PowerShell" | M02, M05 onward | Versioned modules, tests, drift detection, state recovery; `net-triage.sh`, Python tooling, one PowerShell module |
| "Git, CI/CD pipelines, Automated testing, Automated deployment" | M07, M14 | Reusable workflows, a test pyramid including Terraform tests and a k6 gate, measured rollback, GitOps |
| "Cloud-native platforms, Internal developer tools, Self-service portals, Automation solutions" | M17, M18 | Golden path, portal, vending, templates |
| "Monitoring, Logging, Observability, Troubleshooting across application and cloud infrastructure layers" | M10 onward | Dashboards and alerts as code, 36 scenarios diagnosed across layers |
| "AWS SA Professional / AWS DevOps Professional / Azure Solutions Architect Expert" | §21 | Taken after the matching milestones; the project is the preparation |
| "AI / GenAI, AI agents, MCP integrations" *(advantageous)* | M19, M20 | AI gateway with fallback and cost control, a read-only MCP agent with evals and an injection test |
| "unify control across datacentres, networks, public and private cloud" *(team description)* | M21 | A local "datacentre" governed through Arc and SSM by the same policy, with cross-cloud VPN in a timeboxed window |

Every JD line has somewhere it gets built, and nothing in the project exists without a JD line or a failure it prevents. Each component had to pass that test.

## 5. Evolution: how Runway grows

The project begins as one service on one VM and grows. Each stage is a working system you can demonstrate and a stable point to return to.

| Version | Milestones | ≈Month | What Runway is at this point |
|---|---|:-:|---|
| **v0: local** | M00–M03 | 1–3 | `cargo-api` + `carrier-sim` + Postgres under docker-compose. Network plan, first SDD, ADR-001/002. No cloud yet. |
| **v1: hand-built on AWS** | M04 | 4–5 | Organizations with OUs, IAM Identity Center, a VPC, EC2 running `cargo-api` under systemd behind an ALB, RDS, Route 53, ACM TLS. Budgets. Everything clicked, and everything written down. |
| **v2: codified and containerised** | M05–M07 | 5–8 | Everything in Terraform with small states, then hardened, signed images, then CI/CD with OIDC deploying to **ECS Fargate** across dev and prod accounts. Build once, promote by digest, measured rollback, change records. |
| **v3: Kubernetes in production** | M08–M12 | 8–14 | Zero-downtime migration from ECS to **EKS**, with Karpenter, the LB controller, Pod Identity and ESO. Then full observability, SLOs, L1/L2 event management, 20+ worked incidents, a capacity plan and DR drills with measured RTO/RPO. *This is a complete, operable single-cloud platform.* |
| **v4: multi-cloud** | M13–M16 | 14–18 | Azure landing zone, Entra ID as workforce IdP federated to AWS, AKS, Argo CD delivering to both clusters, cross-cloud workload identity. Security and governance across both clouds. FinOps across both bills. |
| **v5: a platform with customers** | M17–M18 | 19–20 | Golden path, `workload.yaml`, account vending, portal with catalogues and PR-based self-service. `hello-svc` onboarded to both clouds, timed. |
| **v6: AI-capable, hybrid, hardened** | M19–M22 | 21–24 | AI gateway with cross-cloud fallback, `insights-worker`, Flightdeck over MCP, a local "datacentre" under Arc and SSM, a threat model, hardening, and the capstone design exercise. |

**If you stop at v3 you still have a coherent, defensible project.** Each later version adds scope without invalidating what came before.

## 6. Explicitly out of scope

This is written down so that scope creep has to argue with a document rather than with your mood at 11 p.m.

- **No service mesh.** You will be able to explain what Istio or Linkerd would give you (mTLS, traffic shifting, L7 telemetry) and why three services do not justify the operational cost.
- **No Kafka, data pipelines or warehouse.** SQS teaches the queueing lessons for far less.
- **No model training, fine-tuning or permanent GPU serving.** You operate AI; you do not build models. One timeboxed self-hosted inference lab is the limit.
- **No multi-region active-active, and no cross-cloud DR.** You will have a single-region primary with a *tested* backup-restore and pilot-light recovery, which is what most enterprises actually run (ADR-014).
- **No custom Kubernetes operators.** You will understand the reconcile loop; you will not write a controller.
- **No GCP.** Two clouds deeply beat three shallowly.
- **No microservice sprawl, no public users, no mobile app.** Three business workloads is the point.

## 7. Repository layout

Start as a monorepo, because it is far easier to show a reviewer (ADR-001). Split `hello-svc` into its own repository and publish `templates/golden-path/` as a template repository in M17, because cross-repository consumption of your workflows and modules is what proves the path is real.

```
runway/
├── README.md                       # what it is, the numbers table, what I operate, what I built, what I didn't
├── Makefile                        # up, down, test, scan, loadtest, inject
├── catalog/                        # catalog-info.yaml per component; data/ entries (data catalogue)
├── docs/
│   ├── adr/  architecture/  solution-designs/  runbooks/  incidents/  problems/  changes/
│   ├── capacity/  continuity/  cost/  security/  operations/  lifecycle/  platform/  estimates/
├── platform/
│   ├── modules/                    # aws-network, aws-eks, aws-rds, aws-account, azure-network, azure-aks, azure-keyvault,
│   │                               # azure-openai, k8s-baseline, observability, workload, ...
│   ├── live/aws/<account>/<env>/<component>/   live/azure/<subscription>/<env>/<component>/
│   ├── policy/                     # SCPs, Azure Policy, Kyverno, conftest (OPA)
│   ├── gitops/                     # Argo CD app-of-apps, cluster add-ons (M14+)
│   ├── observability/              # dashboards, alert rules, SLOs as code
│   ├── loadtest/                   # k6
│   └── scripts/                    # net-triage.sh, inject-fault.sh, Python tools, one PowerShell module
├── .github/workflows/              # reusable workflows + thin callers
├── workloads/ cargo-api/ carrier-sim/ eta-worker/ insights-worker/ runway-portal/ ai-gateway/ ems-bridge/ finops-collector/
├── flightdeck/                     # MCP server (read-only tools), agent, evals
└── templates/golden-path/          # template repo source (published as its own repo in M17)
```

Documentation conventions for every folder under `docs/` are in [Appendix A2](appendix/A2-documentation-standards.md).

## 8. Budget, in one paragraph

Two clouds cost more than one. Plan for **roughly $50–90/month on average**:

- near zero through M00–M03
- spikes to around $120–150 in the EKS, HA/DR and VPN windows
- budget alerts in both clouds before the second resource exists, and a hard alarm at $120

`make up` and `make down` every session is the discipline that makes this affordable, and it is also your IaC discipline. Operations phases include a few 48–72-hour "soak windows" where prod stays up so that real time passes over it. The known surprise bills (NAT Gateway data processing, log ingestion, a forgotten node pool) are exercises in [M16](milestones/M16-finops.md), not accidents. If your ceiling must stay near $60, the fallbacks are in [Section 22](22-timeline.md): Grafana Cloud's free tier, more local-cluster work, shorter windows, and skipping Multi-AZ and VPN windows. Each has a named cost to your learning.

→ Next: [Section 6 — Final Project Architecture](06-final-project-architecture.md)
