# Section 1 — JD Skills Analysis

**Source:** *Senior Technical Engineer – Public Cloud* (Group IT, Platform Engineering). Every quotation below is verbatim from the posting.

This section does one job: work out what the posting actually expects you to *do*, and how well. It doesn't list technologies to collect. The technologies matter only as evidence of the capability behind them.

---

## 1. Read the posting as three passages

**The team.** *"Join our Platform Engineering team, the backbone of our cloud modernization journey. We architect and engineer next-generation cloud platforms that unify control across datacentres, networks, public and private cloud environments. With a strong emphasis on automation, agility, and cost optimization, we enable scalable, resilient, and secure infrastructure services that power critical business functions across the Group."*

**The role.** *"As Senior Technical Engineer – Public Cloud, you will design, develop, and deliver secure, scalable, and reliable public cloud solutions aligned with business needs and enterprise standards. The role will act as a technical expert within Public Cloud, supporting cloud automation, workload onboarding, platform engineering, and the development of internal self-service capabilities."*

**The operating model.** *"Working closely with application, architecture, security, network, and DevOps teams, the role will drive the adoption of Infrastructure as Code, CI/CD, observability, governance, and FinOps practices while ensuring solutions meet required standards for availability, performance, security, reliability, and operational support."*

### Five things these passages tell you that the skills list doesn't

1. **You design as well as build.** The verbs are *design, develop, deliver*, *"Develop solution architecture for assigned projects in accordance with established EA standards"*, and *"Guide Project Managers and Technology Team Managers on solution definition, technical design"*. A mid-level engineer is handed a design. A senior engineer writes one, defends it and estimates what it costs.
2. **You "drive adoption", so the platform has customers.** IaC, CI/CD, observability, governance and FinOps are things you get *other teams* to adopt. That's platform engineering stated as a responsibility, backed by *"internal developer tools, self-service portals"* in the requirements.
3. **You run what you build.** *"Ensure that assigned incidents are resolved within agreed service levels … problems are defined and analysed in line with the Problem Management Process … Own and manage changes within the technology area."* Incident, problem and change management are written into the job, and so is configuring *"Level 1 and Level 2 incidents … within EMS"*.
4. **Two clouds, at operating depth.** *"Proficiency in managing and operating AWS and Azure cloud platforms"* and *"Experience operating large public cloud footprints"*. "Proficiency" plus "operating" means you run both. Knowing how to translate between them isn't enough.
5. **It's hybrid.** *"Unify control across datacentres, networks, public and private cloud environments."* A datacentre estate exists and isn't going away.

---

## 2. What changed compared with the previous (non-senior) posting

The earlier roadmap targeted *Technical Engineer – Public Cloud* in the same team. The senior posting is not the same job with more years attached.

| Dimension | Previous posting | Senior posting | Consequence for the roadmap |
|---|---|---|---|
| Cloud provider | Provider-neutral | **"AWS and Azure"**, with EKS and AKS named | Both clouds at operating depth. One-cloud depth plus a translation exercise no longer meets the bar |
| Scope of design | Implement and support | **"design, develop, and deliver"**, "solution architecture … EA standards", "technical expert" | Solution design documents, ADRs, cost estimates and guiding others become graded deliverables |
| Scale | Public cloud environments | **"large public cloud footprints"** | Multi-account and multi-subscription landing zones, account/subscription vending, governance at scale |
| Platform engineering | "workload onboarding, automation and self-service capabilities" | **"cloud-native platforms, internal developer tools, self-service portals"** | You build a portal, not just modules and templates |
| Development | "Development experience" | **"Strong full-stack development experience"** | React + Node/TypeScript at real depth, not token depth |
| Testing | Implied | **"Automated testing, Automated deployment"** | A test pyramid across application *and* infrastructure code |
| ITIL processes | Incident, problem, service level, capacity, availability, continuity | Adds **"Own and manage changes"** and **EMS L1/L2 configuration** | Change management and event management become explicit milestones |
| Documentation | Docs and procedures | Adds **"data catalogue"**, "detailed designs, configuration details … operations manuals, and deployment architecture" | A data catalogue and enterprise-grade document templates |
| Lifecycle | Maintenance | **"Manage end-of-life replacement of equipment and/or software"** | An EOL register and at least one real replacement (e.g. retired ingress-nginx, Kubernetes upgrades, an ECS→EKS migration) |
| Certifications | Associate-level, desirable | **Professional/Expert**: AWS SA Pro, AWS DevOps Pro, Azure Solutions Architect Expert | Associate exams become stepping stones. The target is Professional level |
| AI | Exposure advantageous | AI/GenAI, agents, MCP advantageous | Unchanged: still a differentiator, still not the core |

---

## 3. Category-by-category analysis

**Depth scale:** **Foundation** (can explain it and use it with guidance) → **Working Knowledge** (use it independently on normal tasks) → **Production Proficiency** (operate it in production, debug it under pressure, own its failures) → **Advanced/Expert** (design with it across teams, set standards, teach it, make the trade-offs).

**Class:** **E** = explicitly named · **I** = implied by a responsibility · **A** = advantageous.

### 3.1 Cloud platforms

| Area | Class | Expected depth | JD evidence for that depth | What it means in practice |
|---|:-:|---|---|---|
| **Cloud engineering (general)** | E | **Advanced/Expert** | *"design, develop, and deliver secure, scalable, and reliable public cloud solutions"*; *"act as a technical expert within Public Cloud"* | Knowing the services isn't enough. You choose between them, justify the choice on cost, reliability and operability, and other people follow your standard |
| **AWS** | E | **Production Proficiency → Advanced** | *"Proficiency in managing and operating AWS and Azure"*; *"operating large public cloud footprints"*; *"Amazon EKS"*; AWS SA Pro and DevOps Pro listed | Multi-account Organizations, VPC design, IAM policy evaluation, EKS operation, RDS/S3/Route 53/ACM/ALB, cost and quota management, run over months |
| **Azure** | E | **Production Proficiency → Advanced** | Same line; *"Azure AKS"*; Azure Solutions Architect Expert listed | Management groups, subscriptions, Entra ID, Azure RBAC, Azure Policy, VNet/private endpoints/private DNS, AKS, Key Vault, Azure Monitor |
| **Large-footprint operation** | E | **Production Proficiency** | *"Experience operating large public cloud footprints and onboarding applications using governed and automated DevOps practices"* | Landing zones, account/subscription vending, inherited guardrails, tagging at scale, fleet-wide patching and upgrades. Practise the pattern at small scale and be honest about the scale |
| **Hybrid / datacentre** | E (team) | **Working Knowledge** | *"unify control across datacentres, networks, public and private cloud environments"*; *"hybrid cloud capabilities"* | Site-to-site VPN, hybrid DNS, Azure Arc, AWS Systems Manager hybrid nodes, one policy over cloud and datacentre resources |

### 3.2 Operating system, network and programming

| Area | Class | Expected depth | JD evidence | What it means in practice |
|---|:-:|---|---|---|
| **Linux** | I | **Production Proficiency** | Named nowhere. Assumed by *"Troubleshooting across application and cloud infrastructure layers"*, containers, Bash, and every node in every cluster | systemd, journald, processes and signals, permissions, filesystems and inodes, cgroups and namespaces, and `ss`/`dig`/`curl -v`/`tcpdump`/`strace` from a shell |
| **Networking** | I | **Production Proficiency → Advanced** | *"unify control across datacentres, networks"*; working with *"network … teams"*; troubleshooting *"across … infrastructure layers"* | CIDR planning across two clouds and a datacentre, routing, NAT, DNS (including private and split-horizon), TLS, L4 vs L7 load balancing, VPN, private endpoints. Most incidents live here |
| **Programming (general)** | E | **Production Proficiency** | *"Strong full-stack development experience"*; *"Ensure process and enhancement specifications are translated into effective programs with error-free code using enterprise development tools"* | You ship and maintain code other people depend on: tested, reviewed, versioned, observable |
| **Python** | E | **Production Proficiency** | Listed first under software development *and* under automation | Services (FastAPI), cloud SDK automation (boto3, Azure SDK) with retries, pagination and throttling handled, CLI tooling, tests |
| **TypeScript / JavaScript, React, Node.js** | E | **Working Knowledge → Production Proficiency** | *"Strong full-stack development … TypeScript/JavaScript … React … Node.js … Or equivalent"* | A real internal web application (the self-service portal) with a Node/TS backend-for-frontend, deployed and operated by you |
| **Bash** | E | **Production Proficiency** | *"Automation/scripting experience using: Python, Bash, PowerShell"* | Safe, idempotent scripts (`set -euo pipefail`, exit codes, traps) used in CI and during incidents |
| **PowerShell** | E | **Working Knowledge** | Same line; implied by an Azure/Microsoft enterprise estate | Read, modify and write Az-module automation; the object pipeline; one maintained module |
| **Backend / API development** | E | **Production Proficiency** | *"Experience designing and developing: REST APIs, Backend services, Integrations, Web-based applications"* | OpenAPI-first design, idempotency, pagination, versioning, timeouts and retries, auth (OAuth2/OIDC), contract tests, and a breaking change shipped without breaking a consumer |
| **Integrations** | E | **Working Knowledge → Production Proficiency** | *"Integrations"* listed with REST APIs | Queues, webhooks, third-party APIs, cross-cloud service-to-service identity |

### 3.3 Delivery

| Area | Class | Expected depth | JD evidence | What it means in practice |
|---|:-:|---|---|---|
| **Git** | E | **Production Proficiency** | *"Strong experience with DevOps practices, including: Git"* | Trunk-based flow, protected branches, CODEOWNERS, reflog/bisect recovery, repo strategy for many consumers |
| **CI/CD** | E | **Advanced** | *"Strong experience with DevOps practices … CI/CD pipelines, Automated testing, Automated deployment"*; *"drive the adoption of … CI/CD"* | Build once and promote by digest, gated environments, OIDC (no stored credentials), measured rollback, and reusable pipelines other teams consume. "Drive adoption" makes this platform-grade |
| **Automated testing** | E | **Production Proficiency** | *"Automated testing"*; *"error-free code"* | Unit, contract, integration (testcontainers), smoke, performance gates, *and* Terraform tests and policy tests on plans |
| **Docker** | E | **Production Proficiency** | *"Understanding of containerisation … Docker"* | Hardened multi-stage non-root images, signal handling, SBOMs, signing, CVE triage. "Understanding" is the posting's floor, not your target: you can't operate EKS/AKS on understanding alone |
| **Kubernetes** | E | **Production Proficiency** | *"containerisation and orchestration technologies such as … Kubernetes"* | Scheduling, probes, requests/limits, HPA, node autoscaling, RBAC, NetworkPolicy, upgrades, and diagnosing every pod failure state quickly |
| **EKS** | E | **Production Proficiency** | *"Amazon EKS"* | VPC CNI and IP planning, EKS Pod Identity/IRSA, AWS Load Balancer Controller, Karpenter, managed add-ons, version upgrades |
| **AKS** | E | **Production Proficiency** | *"Azure AKS"* | Azure CNI choices, Workload Identity, node pools and autoscaling, private clusters, upgrade channels, Azure Monitor integration |
| **Infrastructure as Code** | E | **Advanced/Expert** | *"Strong automation experience using Infrastructure as Code"*; *"drive the adoption of Infrastructure as Code"* | Nothing exists outside code; drift is detected and reconciled; other teams build on your modules |
| **Terraform** | E | **Advanced/Expert** | *"preferably: Terraform"* | Module design as API design, state isolation and recovery, `import`/`moved`, provider upgrades, testing, policy-as-code on plans, versioned module releases |

### 3.4 Platform engineering

| Area | Class | Expected depth | JD evidence | What it means in practice |
|---|:-:|---|---|---|
| **Platform engineering** | E | **Advanced** | *"Join our Platform Engineering team"*; *"supporting cloud automation, workload onboarding, platform engineering"*; *"Experience building: Cloud-native platforms"* | Treat the platform as a product whose users are engineers. Golden paths, sensible defaults, escape hatches, versioning and deprecation |
| **Internal developer platforms / tools** | E | **Production Proficiency** | *"internal developer tools"*; *"the development of internal self-service capabilities"* | Template repositories, a workload contract, reusable workflows, a service catalogue, CLI or portal front doors |
| **Self-service infrastructure / portals** | E | **Production Proficiency** | *"Self-service portals"*; *"onboarding applications using governed and automated DevOps practices"* | A portal where a team requests a service, environment or sandbox account and gets it through review, policy and a pipeline, without filing a ticket |
| **Automation solutions** | E | **Advanced** | *"Automation solutions"*; *"strong emphasis on automation"* | Toil removed with measurable before/after numbers, and automation that is safe to run by other people |

### 3.5 Security and governance

| Area | Class | Expected depth | JD evidence | What it means in practice |
|---|:-:|---|---|---|
| **Cloud security** | E | **Production Proficiency** | *"Strong knowledge of cloud concepts and best practices, including: Security"*; *"secure … solutions"*; *"solutions to problems meet architectural and security standards"* | Threat modelling, defence in depth, private networking, egress control, supply-chain controls, secret lifecycle, a leaked-credential drill |
| **IAM** | E (via security) | **Advanced** | Implied by "secure", "governed", two clouds, and every automated pipeline | AWS IAM policy evaluation (SCPs, boundaries, session policies), Azure RBAC and Entra ID, workload identity federation, no static keys, least privilege derived empirically |
| **Governance** | E | **Production Proficiency → Advanced** | *"Governance"*; *"drive the adoption of … governance"*; *"governed and automated DevOps practices"*; *"conforming to corporate standards"* | SCPs, Azure Policy, admission control, policy-as-code on plans, audit-then-deny rollout, exception process, tagging taxonomy |

### 3.6 Reliability and operations

| Area | Class | Expected depth | JD evidence | What it means in practice |
|---|:-:|---|---|---|
| **Reliability** | E | **Production Proficiency** | *"Resilience"*; *"reliable public cloud solutions"*; *"meets service-level requirements in terms of expected availability, capacity, continuity, security, performance, and reliability"* | SLIs/SLOs that reflect user pain, error budgets used in decisions, stability patterns (timeouts, breakers, bulkheads) |
| **High availability** | E | **Production Proficiency** | *"expected availability"*; *"scalable, resilient"* | Multi-AZ design, finding every single point of failure, PDBs and topology spread, failover measured under traffic |
| **Disaster recovery / continuity** | E | **Production Proficiency** | *"continuity"* in the service-level list; *"Resilience"* | RTO/RPO tiers, backup vs replication, tested restores, rebuilding from code, a pilot-light region, timed drills |
| **Capacity** | E | **Working Knowledge → Production Proficiency** | *"expected … capacity"* | Load testing, finding the saturating resource, quotas, headroom policy, forecasting |
| **Monitoring** | E | **Production Proficiency** | *"Experience with: Monitoring"*; *"implement ongoing monitoring of the relevant technology to ensure Level 1 and Level 2 incidents are configured within EMS"* | Symptom-based alerts, an alert taxonomy with L1/L2 routing, every alert with a runbook, event-management integration |
| **Logging** | E | **Production Proficiency** | *"Logging"* | Structured logs with correlation IDs, a central store, retention and cost control, nothing sensitive logged |
| **Observability** | E | **Advanced** | *"Observability"*; *"drive the adoption of … observability"* | OpenTelemetry across services and clouds, standard telemetry supplied by the platform, answering questions nobody anticipated |
| **Incident management** | E | **Production Proficiency** | *"Ensure that assigned incidents are resolved within agreed service levels"* | Severity, mitigate before you diagnose, workarounds, communication, timelines written during the incident |
| **Problem management** | E | **Production Proficiency** | *"problems are defined and analysed in line with the Problem Management Process"*; *"the impact on the business is minimised"* | RCA with causal chains, known-error records, a permanent fix that stops the incidents recurring |
| **Change management** | E | **Production Proficiency** | *"Own and manage changes within the technology area"* | Standard/normal/emergency changes, risk assessment, change records linked to deployments, freeze windows, post-implementation review, change failure rate |
| **Troubleshooting** | E | **Advanced** | *"Troubleshooting across application and cloud infrastructure layers"* | Bisecting the *system*: from the client through DNS, edge, LB, gateway, pod, identity, database and a dependency in another cloud, naming the exact hop that failed |
| **Lifecycle / EOL** | E | **Working Knowledge → Production Proficiency** | *"Manage end-of-life replacement of equipment and/or software within the technology area"* | EOL register, Kubernetes and database major upgrades, deprecated-API migration, replacing retired components |

### 3.7 Cost, documentation, architecture

| Area | Class | Expected depth | JD evidence | What it means in practice |
|---|:-:|---|---|---|
| **FinOps** | E | **Production Proficiency** | *"FinOps"* (twice); *"strong emphasis on … cost optimization"*; *"delivered according to agreed costs"* | Tagging and allocation, budgets and anomaly detection in both clouds, unit economics, rightsizing, commitment concepts, Kubernetes cost allocation, estimating the cost of a design *before* it's built |
| **Technical documentation** | E | **Advanced** | *"Publish and maintain high-quality documentation and a data catalogue … detailed designs, configuration details, support procedures, operations manuals, and deployment architecture"*; *"clear and concise: Technical documentation, Solution designs, Operational runbooks"* | Templates used consistently, runbooks a stranger can follow at 3 a.m., and a data catalogue with ownership, classification and retention |
| **Solution architecture** | E | **Advanced** | *"Develop solution architecture for assigned projects in accordance with established EA standards"*; *"Ensure that the design is robust, appropriate, of high quality, conforming to corporate standards"*; *"Guide Project Managers … delivered according to agreed costs and timescales"* | Solution design documents, ADRs with rejected options argued properly, NFRs mapped to design choices, cost and effort estimates you then track against actuals |

### 3.8 AI (advantageous)

| Area | Class | Expected depth | JD evidence | What it means in practice |
|---|:-:|---|---|---|
| **AI / GenAI infrastructure** | A | **Working Knowledge** (JD) · **Production Proficiency** (your career goal) | *"Experience with the following would be advantageous: AI / Generative AI (GenAI)"* | Running an AI dependency as a workload: managed-model quotas, private endpoints, managed identity, rate limits, caching, queues, fallbacks, token cost, AI telemetry, eval gates |
| **AI agents** | A | **Working Knowledge** | *"AI agents"* | Tool calling, bounded loops (step, time and token limits), read-only by construction, auditable, human in the loop |
| **MCP integrations** | A | **Working Knowledge** | *"Model Context Protocol (MCP) integrations"* | A remote MCP server with authenticated, read-only tools over real platform telemetry, and defences against injection through tool results |

---

## 4. Prerequisites the JD assumes but never names

These are what a senior interviewer tests in the first ten minutes, because everything on the list above depends on them.

| Assumed skill | Why the JD depends on it | Where you build it |
|---|---|---|
| **Linux internals** (processes, signals, filesystems, cgroups, namespaces) | Every container is a Linux process; every node is a Linux host | [M01](milestones/M01-linux-networking.md) |
| **TCP/IP, DNS, TLS, HTTP** as a debugging order | "Troubleshooting across … infrastructure layers" is mostly this | [M01](milestones/M01-linux-networking.md) |
| **CIDR and routing** across multiple networks | Two clouds plus a datacentre means address plans that must not overlap | [M01](milestones/M01-linux-networking.md), [M04](milestones/M04-aws-core.md) |
| **Distributed-systems basics** (statelessness, idempotency, retries, back-pressure, consistency) | "Scalable, resilient" can't be designed without them | [M03](milestones/M03-cloud-architecture.md) |
| **Identity protocols** (OAuth2, OIDC, SAML, JWT claims) | Workload federation, SSO, the portal, cross-cloud calls | [M04](milestones/M04-aws-core.md), [M13](milestones/M13-azure-core.md) |
| **Databases in operation** (connections, migrations, backups, PITR) | Continuity and incident work centre on state | [M00](milestones/M00-foundations.md), [M12](milestones/M12-capacity-ha-dr.md) |
| **Queueing and performance maths** (percentiles, Little's Law, saturation) | "Capacity, performance" in the service-level list | [M12](milestones/M12-capacity-ha-dr.md) |
| **Threat modelling** | "Architectural and security standards" | [M15](milestones/M15-security-governance.md) |
| **Written communication** (ADRs, SDDs, runbooks, estimates) | Guiding PMs, EA standards, documentation duties | Every milestone; [A2](appendix/A2-documentation-standards.md) |
| **Service-management vocabulary** (ITIL 4: incident, problem, change, SLM) | The role's process language | [M11](milestones/M11-sre-operations.md) |
| **Estimation** (effort, cost, risk) | *"delivered according to agreed costs and timescales"* | The "Estimate vs actual" table in every milestone |

---

## 5. What the JD notably does *not* ask for

Don't add these because they're fashionable. Several appear later only as clearly labelled explanations.

- **No service mesh**, no Kafka or data engineering, no ML training or fine-tuning, no GPU fleet management.
- **No GCP.** Two clouds is the requirement; a third would dilute both.
- **No multi-region active-active.** "Resilience" and "continuity" are satisfied by tested, measured recovery.
- **No named GitOps tool, observability vendor or portal framework.** Every choice is yours to justify in an ADR. That's a senior test in itself.
- **No SRE job title.** SRE practices (SLOs, error budgets, toil) serve the ITIL processes the JD names; they don't replace them.
- **AI is advantageous, not required.** An AI-first portfolio would read as an AI candidate with thin infrastructure skills.

---

## 6. Priority summary

**P0: the roadmap fails if these aren't at Production Proficiency.**
AWS and Azure operation · Linux · networking · IAM on both clouds · Terraform · CI/CD with automated testing · Docker · Kubernetes on EKS and AKS · observability and troubleshooting · incident, problem and change management · Python and Bash · API/backend development · documentation and solution design.

**P1: Production Proficiency or strong Working Knowledge, with evidence.**
Platform engineering, golden paths and the self-service portal · governance and policy-as-code · FinOps · HA, DR and capacity · security hardening · TypeScript/React/Node · lifecycle/EOL · large-footprint landing zones · event management (L1/L2).

**P2: differentiators.**
AI infrastructure, the AI gateway, agents and MCP · hybrid (Arc, SSM, VPN, hybrid DNS) · PowerShell depth · Professional-level certifications.

**P3: explain only.**
Service mesh · Kubernetes operators · cross-cloud DR · GPU serving at scale · GCP.

→ Next: [Section 2 — Skills Gap and Prerequisite Analysis](02-skills-gap-and-prerequisites.md)
