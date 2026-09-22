# Step 1 — Job Description Analysis

**Source:** Technical Engineer – Public Cloud (Group IT, Platform Engineering), Dubai — LinkedIn posting, retrieved 2026-09-19.

## 1. What the posting actually says

Three passages carry most of the signal. Read them before the table.

**The team.** *"Join our Platform Engineering team, the backbone of our cloud modernization journey. We architect and engineer next generation cloud platforms that unify control across datacentres, networks, public and private cloud environments. With a strong emphasis on automation, agility, and cost optimization, we enable scalable, resilient, and secure infrastructure services that power critical business functions."*

**The role.** *"...responsible for the technology solutions implementation and operational support of the relevant technology area, including configuration, testing, implementation, ongoing maintenance, and support. Support the IT production processes and efficient deployment of solutions governing incident management, problem management, service level management, capacity management, availability management, and IT service continuity."*

**The desirables.** *"Cloud certification or relevant technical certification is desirable... Exposure to AI/GenAI, AI agents or MCP integrations would be advantageous."*

### Three things this tells you that the skills list does not

1. **This is not a pure build role.** Six ITIL process names are written into the role definition. Roughly half the job is *running* a platform that already exists. Most cloud portfolios on the internet stop at "I deployed it." That is where this job starts.
2. **It is hybrid, not cloud-native-greenfield.** "Unify control across datacentres, networks, public **and private** cloud." There is an on-premises estate and it is not going away. Skills that only exist inside one public cloud's console are worth less here than skills that span both.
3. **The platform has customers.** "Cloud workload onboarding, automation and self-service capabilities" means internal development teams consume what you build. That is the differentiator between a Cloud Engineer and a Platform Engineer, and it is explicitly in the requirements.

### What the posting notably does *not* ask for

No named cloud provider. No service mesh. No GitOps tooling by name. No Kafka, no data engineering, no ML training, no model fine-tuning, no multi-cloud. No SRE error-budget language. Do not put these in the project because they are fashionable — several appear later in this roadmap only as clearly-labelled career extensions.

## 2. Requirement table

Classification column: **E** = explicit in the JD, **I** = implied by a responsibility, **X** = not in the JD, added for your longer-term career.

| Job Requirement | Class | Category | Skill / Technology | What I need to learn | Practical experience needed | Priority |
|---|:--:|---|---|---|---|:--:|
| "Good hands-on experience managing and supporting public cloud environments" | E | Cloud | Azure (primary), subscription/RG/tenant model | Resource hierarchy, regions/zones, quotas, SLAs, the shared responsibility model | Run a live subscription for months: provision, patch, pay for, and break it | **P0** |
| "cloud services such as compute, storage, networking, serverless and container services" | E | Cloud | VMs/VMSS, Blob + disks, VNet, Functions/Container Apps, AKS/ACI | When each compute model is correct and what it costs; storage tiers and redundancy; serverless cold start and concurrency | Deploy the same workload three ways (VM, container app, AKS) and justify the choice with numbers | **P0** |
| "Understanding of containerisation technologies such as Docker and Kubernetes" | E | Containers | Docker, Kubernetes, AKS | Images/layers, non-root, healthchecks, signals; pods, deployments, services, ingress, probes, requests/limits, HPA, RBAC | Operate a cluster: rollouts, rollbacks, a CrashLoopBackOff you diagnose from first principles, an OOMKill you fix | **P0** |
| "Infrastructure as Code and automation tools such as Terraform, Python, Bash and/or PowerShell" | E | IaC | Terraform, Bash, Python, PowerShell | State, providers, modules, variables/outputs, `for_each`, lifecycle, drift, import, workspaces vs directories | Every cloud resource in the project created by `terraform apply`; a deliberate drift incident reconciled | **P0** |
| "Experience with Git, DevOps practices and CI/CD pipelines" | E | CI/CD | Git, GitHub Actions (+ Azure DevOps port) | Branching, PR review, trunk-based flow; build-once/promote-by-digest, environments, approvals, secrets, OIDC federation | A pipeline that builds, scans, gates, deploys to two environments, and can roll back in a measured time | **P0** |
| "monitoring, logging, observability and basic application/cloud troubleshooting" | E | Observability | Azure Monitor, Log Analytics/KQL, Managed Prometheus + Grafana, OpenTelemetry | Logs vs metrics vs traces; RED/USE; cardinality; symptom-based alerting; SLI/SLO | Dashboards and alerts you authored; incidents you detected *from the alert*, not from someone telling you | **P0** |
| "Development experience using technologies such as Python, TypeScript/JavaScript and APIs, React, Node.js" | E | Programming | Python, TypeScript, Node, React | Typed Python, async, packaging; TS fundamentals; a Node BFF; enough React to build a real internal UI | One Python service and one TypeScript/React+Node app, both containerised and deployed by you | **P0** |
| "Experience developing or integrating APIs, backend services and web applications" | E | Backend | HTTP, REST, OpenAPI, auth | Status codes, idempotency, pagination, versioning, timeouts/retries, OAuth2/OIDC token flow | An API with an OpenAPI contract that another component you wrote consumes | **P0** |
| "Experience supporting cloud workload onboarding, automation and self-service capabilities" | E | Platform Eng | Golden paths, reusable Terraform modules, reusable CI workflows, scaffolding | What a paved road is; module design and versioning; template repos; environment config; policy guardrails | Onboard a workload you did not write, through your own path, in under a day, touching no platform code | **P0** |
| "cloud concepts and best practices, including security, governance and cost awareness" | E | Security / Gov / FinOps | Entra ID, RBAC, Key Vault, Azure Policy, Budgets, tagging | Least privilege, managed identity vs secrets, policy-as-code, tag taxonomy, cost allocation and anomaly detection | A policy that blocks a real deployment of yours; a cost spike you investigate and fix | **P0** |
| "Ability to create clear and concise technical documentation and operational procedures" | E | Documentation | ADRs, runbooks, RCA format, onboarding guides | How to write a runbook someone can follow at 3 a.m.; ADR structure; blameless postmortems | A runbook another person executed successfully without asking you questions | **P0** |
| "assigned incidents are resolved within the agreed Service Levels... workarounds are provided" | E | Incident Mgmt | Severity, triage, comms, workarounds, SLA clocks | Detect → investigate → diagnose → mitigate → fix → verify → document; mitigate before you understand | 12+ injected incidents worked end to end with written timelines | **P0** |
| "problems are defined and analysed in line with the Problem Management Process" | E | Problem Mgmt | RCA, known error, problem records | The difference between incident and problem; 5-whys and causal chains; permanent fix vs workaround | RCA documents that link repeat incidents to one underlying cause, plus the fix that closed it | **P0** |
| "technical input to assist with Capacity Management" | E | Capacity | Load testing, quotas, autoscaling, forecasting | Little's Law, saturation, headroom, percentiles; Azure quota model; HPA/cluster autoscaler behaviour | A load test that finds your real bottleneck and a capacity plan derived from it | **P1** |
| "...IT Service Continuity, and Availability" | E | DR / HA | Backup, restore, RTO/RPO, zones/regions, failover | Availability maths, single points of failure, backup vs replication, tested-vs-assumed recovery | A timed rebuild of the environment from Terraform + a database restore, with measured RTO/RPO | **P1** |
| "solutions meet the architectural and security standards" | E | Security | Network security, secrets, identity, image scanning, WAF | Defence in depth, private endpoints, egress control, CVE triage, supply-chain basics | A threat model for your platform and controls you can point at for each risk | **P1** |
| "unify control across datacentres... public **and private** cloud" | E | Hybrid | Azure Arc, hybrid identity, hybrid networking | Arc-enabled servers/Kubernetes, VPN/ExpressRoute concepts, hybrid DNS | A local k3s cluster and a local VM projected into Azure via Arc and governed by the same policy | **P1** |
| "serverless" | E | Cloud | Azure Functions / Container Apps Jobs | Event triggers, cold start, concurrency, when serverless is wrong | The scheduled cost report and drift detector implemented as serverless jobs | **P1** |
| "Cloud certification... Associate-level cloud, DevOps or developer certification" | E | Certification | AZ-104, AZ-204, AZ-400, CKA, Terraform Associate | Exam scope only after the hands-on work exists | Pass AZ-104 + HashiCorp Terraform Associate; CKA if Kubernetes becomes your depth | **P2** |
| "Exposure to AI/GenAI, AI agents or MCP integrations" | E | AI | Azure OpenAI/AI Foundry, tool calling, MCP | Structured outputs, validation, timeouts/retries, token cost, prompt versioning, evaluation, guardrails | A read-only MCP server over your own platform telemetry and an agent that uses it during a real drill | **P2** (JD) / **P1** (your stated career goal) |
| "Collaborate with cross-functional teams... technical consultancy and guidance" | I | Soft/Arch | Architecture communication | Diagrams that survive questioning; trade-off articulation; written decision records | ADRs where you rejected an option and can defend why | **P1** |
| "configuration, testing, implementation, ongoing maintenance" | I | Ops | Patching, upgrades, lifecycle | AKS version upgrades, node image patching, Terraform provider upgrades, certificate rotation | Perform a cluster upgrade and a certificate rotation without an outage | **P1** |
| "service level management" | I | SLM | SLI/SLO/SLA, error budgets | Defining an SLI that reflects user pain; measurement windows | Two SLOs measured continuously with a burn-rate alert | **P1** |
| Linux (assumed everywhere, named nowhere) | I | OS | Linux administration | systemd, journald, processes, permissions, networking tools, disk/inode, cgroups | Every debugging exercise done from a shell, not a console UI | **P0** |
| Networking (assumed by "networks", "networking") | I | Network | TCP/IP, DNS, TLS, LB, NSG/UDR, private endpoints | Subnetting, routing, name resolution order, TLS handshake and cert chains, L4 vs L7 | Diagnose a broken request through six network layers and name the exact hop that failed | **P0** |
| GitOps / Argo CD / Flux | X | CD | Declarative cluster delivery | Reconciliation model, drift in clusters | Only after plain pipelines are second nature — a late, optional extension | **P3** |
| Service mesh (Istio/Linkerd) | X | Network | mTLS, traffic shifting | What problems it solves and its operational cost | Conceptual familiarity only. Do not install one | **P3** |
| AWS or GCP | X | Cloud | Second provider | Mapping concepts, not relearning them | One Terraform module ported to AWS at the very end, purely to prove transferability | **P3** |
| Kubernetes operators / CRDs | X | Containers | Extending the API | Controller/reconcile loop concept | Conceptual only, unless you go deep on Kubernetes later | **P3** |
| ML training, MLOps pipelines, fine-tuning | X | AI | Model lifecycle | Not required by this JD and not your stated goal | Skip. Operating AI *applications* ≠ training models | **P3** |

## 3. Priority summary

**P0 — the roadmap is a failure if these are not production-ready.**
Azure fundamentals and daily operation · Linux · networking · Docker · Kubernetes/AKS · Terraform · Git + CI/CD · observability and troubleshooting · incident and problem management · IAM/secrets/policy basics · cost awareness · Python + Bash · one TypeScript/React/Node app · API design · documentation and runbooks · workload onboarding and self-service.

**P1 — strong working knowledge, demonstrated at least once with evidence.**
Serverless · capacity management and load testing · availability and DR with measured RTO/RPO · security hardening and threat modelling · hybrid/Arc · PowerShell · SLI/SLO · lifecycle and upgrades · architecture communication.

**P2 — desirable, and where you differentiate.**
AZ-104 and Terraform Associate · AI/GenAI integration · AI agents and MCP · FinOps depth · policy-as-code depth.

**P3 — deliberately deferred.**
GitOps tooling · service mesh · second cloud · operators · anything ML-training-shaped.

## 4. The honest gap read

You meet the "5+ years IT Engineering" bar and the degree requirement. Your repository already shows Python, FastAPI, PostgreSQL, Docker, testing and migrations — that covers a real slice of the *development* requirements, which many infrastructure candidates fail.

What is missing is everything with "cloud" or "operate" in it. You stated you have not run production cloud infrastructure. That is the entire gap, and it is the entire roadmap: **P0 items 1, 2, 3, 4, 5, 6, 9, 10, 11, 12, 13 in the table above.** The good news is that the gap is contiguous — one project closes almost all of it, because in a real platform these skills are not separable.

→ Next: [Step 2 — Target engineering profile](01-target-profile.md)
