# Step 3 — Competency Map and Skills Gap

## How to read this

Four levels, defined as **capabilities you can demonstrate**, never as material you have read.

| Level | The test |
|---|---|
| **Beginner** | You can follow a guide and get a working result. You could not do it again from scratch. |
| **Intermediate** | You can build it unaided for a known case, and you know the vocabulary well enough to search effectively. |
| **Advanced** | You can debug it when it misbehaves, explain *why* it behaves that way, and make design trade-offs. |
| **Production-ready** | You would put it in front of paying users, own the pager for it, and have already proven you can recover it from failure. |

The **Start** column is my read of where you are today, from your repository and your stated "no real cloud yet": ⬤ = you are at or above this level. The **Target** column is where this roadmap takes you.

Any skill whose target is *Production-ready* must have an **artifact** and a **failure you recovered from**. Reading does not move a level. Building moves you to Intermediate. Breaking and fixing moves you to Advanced. Operating over time moves you to Production-ready.

---

## Foundations

| Skill | Start | Target | Beginner | Intermediate | Advanced | Production-ready |
|---|:--:|:--:|---|---|---|---|
| **Cloud fundamentals** | ○ | Prod | Name the service categories; create a resource in the portal | Choose between compute options; understand regions, zones, SLAs, the shared responsibility model | Design a subscription/RG layout with naming, tagging and quota strategy; explain cost drivers of a design before building it | Operate a live subscription for months: quota exhaustion, service health events, provider deprecations, a bill you had to explain |
| **Linux** | ◐ | Prod | Navigate, edit files, manage packages | Manage services with systemd, read journald, permissions, users, cron, disk/inode | Diagnose from a cold start: high load, a full filesystem, a zombie process, a port already bound, a DNS resolution failure; read `/proc` | Harden and patch a host; debug a container's behaviour via cgroups and namespaces; run a kernel-level investigation with `ss`, `strace`, `tcpdump` |
| **Networking** | ○ | Prod | Explain IP, ports, DNS, HTTP at a high level | Subnet with CIDR by hand; configure NSGs and route tables; understand TLS certificates and chains | Trace one failing request across client → DNS → LB → ingress → service → pod → database and name the exact failing hop; explain asymmetric routing, SNAT exhaustion, MTU | Design a hub-spoke topology with private endpoints, egress control and hybrid DNS; recover from a misconfiguration under time pressure |
| **Python** | ⬤ | Prod | Scripts, functions, stdlib | Typed code, packaging, virtualenvs, tests, async | Build CLI tooling and cloud SDK automation others use; handle retries, pagination, throttling correctly | Publish internal tooling with tests, packaging, versioning and error handling that an on-call engineer trusts at 3 a.m. |
| **Bash / PowerShell** | ◐ | Adv / Int | Run commands, simple scripts | `set -euo pipefail`, arguments, exit codes, pipes; PowerShell cmdlet/pipeline model | Write idempotent, safe automation scripts with clear failure modes; read someone else's PowerShell and modify it | Bash: ship scripts used in CI and incident response. PowerShell: *Intermediate is the target* — read/modify, use Az module basics |
| **Git** | ◐ | Prod | commit, push, pull, branch | Rebase vs merge, resolve conflicts, PR workflow, `.gitignore` discipline | Bisect a regression, recover lost work from reflog, structure a repo for multiple consumers, design a branching model | Own repository strategy for a platform: protected branches, required checks, CODEOWNERS, release tagging, monorepo/polyrepo boundaries |
| **APIs** | ⬤ | Prod | Call an API; understand verbs and status codes | Design a REST resource model, validate input, document with OpenAPI, handle auth | Design for idempotency, pagination, versioning, timeouts, retry-safety and backward compatibility | Operate an API: SLOs, rate limits, contract tests, deprecation policy, and a breaking change you shipped without breaking a consumer |

## Infrastructure and delivery

| Skill | Start | Target | Beginner | Intermediate | Advanced | Production-ready |
|---|:--:|:--:|---|---|---|---|
| **Docker** | ⬤ | Prod | Build and run an image | Multi-stage builds, `.dockerignore`, volumes, compose, networking | Non-root images, minimal bases, layer caching strategy, signal handling and graceful shutdown, CVE triage | Images in a registry with scanning, SBOM, pinned digests, reproducible builds, and a rebuild triggered by a base-image CVE |
| **Kubernetes** | ○ | Prod | Run `kubectl get pods` against a cluster someone made | Write Deployment/Service/Ingress/ConfigMap/Secret manifests; understand probes, requests/limits | Diagnose CrashLoopBackOff, ImagePullBackOff, OOMKilled, Pending-unschedulable, failed readiness; understand the scheduler, HPA behaviour, rollout mechanics | Operate a cluster: version upgrades, node pool changes, PDBs, resource governance, RBAC, network policy, and a cluster incident you resolved |
| **Terraform** | ○ | Prod | `init/plan/apply` an example | Variables, outputs, data sources, remote state, provider auth | Author reusable modules with sane interfaces; `for_each`, lifecycle, `moved`, `import`; read a plan critically and predict destroy/replace | Run Terraform as a team would: versioned modules, state locking and isolation per environment, drift detection in CI, a state surgery you performed safely |
| **Infrastructure as Code (as a discipline)** | ○ | Prod | Know the term | Codify what you built by hand | Immutability, idempotency, the blast radius of a plan, secrets that must never enter state, dependency ordering | Nothing in the environment exists outside code; drift is detected automatically and reconciled through a documented process |
| **CI/CD** | ◐ | Prod | A pipeline that runs tests | Build an image, push to a registry, deploy to one environment | Build once and promote by digest; gates, approvals, environment secrets, OIDC federation instead of long-lived keys; caching | Multi-environment promotion, measured rollback time, pipeline as a product other repos consume, and a failed deploy you rolled back under pressure |
| **Cloud architecture** | ○ | Adv | Draw boxes | Assemble a working multi-tier system | Justify every component against cost, reliability and operability; articulate what you deliberately left out | (Production-ready here belongs to an Architect role — Advanced is the correct target now) |

## Running it

| Skill | Start | Target | Beginner | Intermediate | Advanced | Production-ready |
|---|:--:|:--:|---|---|---|---|
| **Monitoring** | ○ | Prod | Look at a built-in dashboard | Create dashboards and metric alerts | Choose the right signals; alert on symptoms not causes; control cardinality and cost | Every alert you own has a runbook, a tested threshold, and a record of whether it has ever been useful |
| **Logging** | ◐ | Prod | Read logs in a console | Structured JSON logs with correlation IDs; centralise them | Write effective KQL; design retention and sampling; keep sensitive data out of logs | Centralised logging with cost control, access control, retention policy, and a real investigation solved purely from logs |
| **Metrics** | ○ | Prod | Read CPU graphs | Emit custom metrics; use RED and USE | Understand percentiles vs averages, histogram buckets, rate windows, cardinality explosion | Metrics that drive SLOs and autoscaling decisions you can defend with data |
| **Tracing** | ○ | Adv | Know what a trace is | Instrument one service with OpenTelemetry | Propagate context across service, queue and cluster boundaries; read a waterfall and find the real cost | Sampling strategy, cost control, and a latency incident diagnosed from traces |
| **Observability (as a discipline)** | ○ | Prod | Have dashboards | Have logs, metrics and traces | Answer *unanticipated* questions about production without shipping new code | You detect incidents before users report them, and your mean time to *understand* is measured |
| **Troubleshooting** | ◐ | Prod | Search the error message | Form a hypothesis and test it | Bisect the system, not the code; reason from the failure's blast radius; know which layer to suspect first | A documented record of incidents you diagnosed, including the ones where your first three hypotheses were wrong |
| **Incident management** | ○ | Prod | Know the word | Follow a severity scale and communicate status | Run the response: mitigate first, diagnose second; declare, delegate, communicate, hand over | Post-incident reviews you wrote; workarounds you shipped inside an SLA; a clean timeline someone else could audit |
| **Problem management** | ○ | Prod | Know incident ≠ problem | Write a root cause analysis | Distinguish cause, trigger and contributing factors; build a known-error record; propose a permanent fix with a cost | Repeat incidents that stopped recurring because of a change you made, traceable in writing |
| **Reliability engineering** | ○ | Adv | Know SLA/SLO/SLI | Define one meaningful SLI and measure it | Set an SLO that reflects user pain; build burn-rate alerts; use the error budget in a real decision | (Production-ready implies organisational authority — Advanced is the correct target) |
| **High availability** | ○ | Prod | Know "multi-AZ" | Deploy across availability zones with a load balancer | Find every single point of failure in your own diagram; understand quorum, failover time and split brain | You failed a component out on purpose during traffic and measured what users experienced |
| **Backup / DR** | ○ | Prod | Enable backups | Perform a restore | Define and measure RTO/RPO; know what your backup does *not* cover; test restores on a schedule | A timed, documented full-environment rebuild and data restore drill you ran more than once |
| **Capacity management** | ○ | Adv | Watch CPU | Run a load test | Find the saturating resource; apply Little's Law; forecast headroom; tune autoscaling with evidence | A capacity plan with numbers you produced, plus the quota increases it required |

## Governance, cost, platform, AI

| Skill | Start | Target | Beginner | Intermediate | Advanced | Production-ready |
|---|:--:|:--:|---|---|---|---|
| **Security** | ◐ | Prod | Know not to commit secrets | TLS, network restriction, least privilege, image scanning | Threat-model your own system; defence in depth; egress control; supply chain; key rotation | A security incident drill (leaked credential) executed end to end: contain, rotate, assess exposure, prevent |
| **IAM** | ◐ | Prod | Create a user | Assign roles; understand scope inheritance | Design least-privilege custom roles; managed identity vs service principal vs secret; federated credentials (OIDC) | An IAM-caused deployment failure you diagnosed from the effective-permission chain, and a role you tightened without breaking anything |
| **Cloud governance** | ○ | Adv | Know policies exist | Apply a built-in policy | Author policy-as-code with audit-then-deny rollout; management group strategy; tagging taxonomy | Policy that has blocked real non-compliant deployments, with an exception process you documented |
| **Cost optimization / FinOps** | ○ | Prod | Look at the bill | Set budgets and alerts; tag for allocation | Attribute cost per service/environment; find waste; model the cost of a design *before* building it | A cost anomaly you detected, investigated and fixed, with before/after numbers — on your own money |
| **Platform engineering** | ○ | Prod | Know the term | Write one reusable Terraform module and one reusable workflow | Design a golden path with sane defaults, escape hatches and a versioning policy; treat internal engineers as users | Someone else onboarded a workload through your path without your help, and you have the time-to-first-deploy number |
| **Developer self-service** | ○ | Adv | Write documentation | Provide templates and scaffolding | Build a request-to-provision flow with policy guardrails and an approval boundary | (Production-ready requires real internal customers — Advanced is the honest ceiling for a solo project) |
| **AI application deployment** | ○ | Prod | Call an LLM API from a script | Deploy a service that calls a managed model, with secrets handled properly | Timeouts, retries with jitter, rate-limit handling, fallbacks, circuit breaking, structured output validation | An AI-backed feature running behind the same SLOs, budgets and alerts as everything else |
| **LLM APIs** | ◐ | Adv | Send a prompt | Use structured outputs and tool schemas | Control cost and latency per request; version prompts; handle context limits and partial failures | (Advanced is the right target — the JD asks for exposure, not specialism) |
| **AI agents** | ○ | Adv | Know what tool calling is | Build an agent with 2–3 read-only tools | Bound the agent: tool allowlists, argument validation, step limits, no-write guarantees, auditable transcripts | (Advanced) |
| **MCP** | ○ | Adv | Know the protocol exists | Run an MCP server and connect a client | Design safe tool surfaces over live systems; authentication and scoping; schema and error design | (Advanced) |
| **AI security** | ○ | Int | Know prompt injection exists | Keep secrets and data out of prompts; deny-by-default tool access | Treat all model output as untrusted input; sanitise tool arguments; defend against injection through retrieved content | (Intermediate is the correct target for this JD) |
| **AI observability** | ○ | Adv | Log the prompt | Track tokens, latency, error rate and cost per request | Trace an AI call as part of a distributed trace; measure quality with an eval set; alert on cost and refusal-rate regressions | (Advanced) |
| **AI infrastructure** | ○ | Int | Know GPUs are involved | Understand managed-model quotas, TPM/RPM limits and regional availability | Reason about when self-hosting is justified vs managed inference, and its cost and capacity implications | (Intermediate — you are operating AI, not building serving infrastructure) |

---

## The three gaps that matter most

Everything above reduces to three statements. If only these three change, you are qualified for this job.

1. **You have never operated cloud infrastructure you owned.** Closed by Milestones 1–5 and, critically, by keeping the environment alive and paid for across months rather than deleting it after a weekend.
2. **You have never been on the hook when something broke.** Closed by the operations playbook — a dozen injected incidents worked to written RCAs. This is the differentiator the JD asks for and almost no self-taught candidate has.
3. **You have built for yourself, never for other engineers.** Closed by Milestone 11, the golden path, where success is measured by someone else's time-to-first-deploy rather than by your own.

Your existing strengths — Python, API design, PostgreSQL, Docker, testing — are not a small head start. They are why this roadmap can spend its time on infrastructure instead of on learning to write a service.

→ Next: [Step 4 — Choosing the primary cloud](03-cloud-choice.md)
