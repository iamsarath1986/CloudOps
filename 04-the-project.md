# Step 5 — The Project: **Runway**

## 1. The idea in one paragraph

**Runway is a small internal cloud platform on Azure, and the workloads that run on it.** It provides one paved road from a Git repository to a running, observed, governed service: reusable Terraform modules, reusable CI/CD workflows, a Kubernetes runtime, a shared observability stack, policy guardrails, budgets, and a self-service portal where an engineer can request a new service and get infrastructure without filing a ticket. Two real workloads run on it — one Python API, one TypeScript web application — and a third throwaway service exists only to prove that onboarding a workload the platform team did not write takes under a day. Attached to the platform is **Flightdeck**, a strictly read-only AI operations assistant that helps a human investigate an incident faster by querying the platform's own telemetry through MCP tools. It cannot change anything.

## 2. Why this project, and not the obvious alternatives

| Candidate project | Why it is rejected |
|---|---|
| "Deploy a three-tier app to Azure with Terraform" | Covers maybe 40% of the JD. Nothing about onboarding, self-service, governance, cost, or the six ITIL processes in the role definition. This is the project everyone builds, which is exactly why it does not differentiate. |
| "Build an AI agent for DevOps" | Inverts the priority. The JD lists AI as *desirable* and cloud platform operation as *required*. An AI-first project reads as an AI candidate with weak infrastructure. |
| "Multi-cloud landing zone" | Impressive-sounding, unachievable at 10 h/week, and produces shallow knowledge in three clouds. The JD does not ask for it. |
| "Kubernetes home lab" | Free and useful, but it evidences none of the cloud, governance, cost or onboarding requirements, and there is no bill to teach you FinOps. |
| **Runway** | The platform *is* the product, which matches the team you are applying to. The application layer is deliberately small so it never competes for your time. Operations are structural rather than decorative. AI arrives last, scoped to what the JD actually asks for. |

**The governing constraint: the application must stay boring.** Every hour spent on business logic is an hour not spent on the thing you are being hired for. `cargo-api` should be finished in two weekends and then never significantly extended. If you find yourself adding features to it, you have drifted.

## 3. The five components

### Component 1 — `runway-platform` *(the real subject of this roadmap)*
The platform itself, entirely as code.

- Versioned Terraform modules: `network`, `aks`, `postgres`, `keyvault`, `observability`, `workload` (the per-service module every onboarded app instantiates)
- Reusable CI/CD workflows other repositories call: `build-and-push`, `deploy-to-aks`, `terraform-plan-apply`
- Azure Policy definitions and assignments, tagging taxonomy, budgets
- Observability as code: dashboards, alert rules, SLO definitions
- Runbooks, ADRs, incident and RCA records

### Component 2 — `cargo-api` *(workload 1 — Python)*
An internal service that tracks cargo shipments. Deliberately unambitious, but chosen so it has every operationally interesting property:

- A REST API (FastAPI) with an OpenAPI contract → API and backend requirements
- PostgreSQL with migrations → data, backup, restore, DR drills
- A background job that recomputes ETAs → async work, queues, capacity
- A call to an external "carrier" API (which you also run, so you can make it fail) → dependency failure, timeouts, retries, circuit breaking
- A cache → cache stampede and invalidation drills
- A read-heavy endpoint → load testing, autoscaling, percentile latency

### Component 3 — `runway-portal` *(workload 2 — TypeScript/React/Node)*
The platform's self-service front door, and simultaneously proof that the golden path handles a non-Python workload. This is where the JD's explicit React/Node/TypeScript requirement is satisfied, in the context where it is actually plausible: internal tooling.

- **React SPA + Node/TypeScript BFF** (the BFF holds the Azure credentials; the browser never does)
- **Onboard a service** — a form that renders a workload module invocation and a CI workflow, then opens a pull request against `runway-platform`. Provisioning happens through review and pipeline, never through a button that calls the Azure API directly.
- **Read-only views**: service catalogue with owners and tags; environment and deployment status; SLO status; cost per service this month; open incidents.

### Component 4 — `hello-svc` *(workload 3 — the proof)*
Forty lines of Go or Node in its own repository. Its only purpose is a timed exercise: **onboard it through the golden path without modifying any platform code, and record the elapsed time.** That number — time to first production deploy for an unfamiliar workload — is the single most convincing metric in your portfolio for the JD's onboarding requirement.

### Component 5 — `flightdeck` *(the AI component — last, and deliberately constrained)*
A read-only operations assistant for incident investigation.

- An **MCP server** exposing tools over the platform's own telemetry: `get_service_health`, `get_recent_logs`, `get_deployment_status`, `get_pod_status`, `get_metrics`, `get_active_alerts`, `get_runbook`, `get_recent_changes`
- An **agent** (Azure OpenAI) that an engineer talks to during an incident, which calls those tools and produces a summary, a hypothesis list, and a citation of the evidence for each
- **Hard constraints, enforced in code and tested:** no write tool exists; the Azure identity the MCP server uses holds `Reader` only; every tool call is logged with arguments and results; the agent's output is advisory text that a human acts on
- Treated like any other workload: it goes through the golden path, has SLOs, dashboards, a budget and a runbook

**Why an AI assistant is genuinely justified here rather than bolted on:** by the time you build it, you will have run a dozen incidents manually and will know precisely which five queries you always run first. Automating *those* is a real engineering decision with a measurable before/after (time to first hypothesis). Building it before you have that experience produces a demo. This is why Flightdeck is Milestone 12, not Milestone 2.

## 4. How the project maps to the job description

| JD requirement | Where it is built | Evidence produced |
|---|---|---|
| Managing/supporting public cloud environments | Milestones 2–12, continuously | A live subscription, a bill, an uptime record, an incident log |
| Compute, storage, networking, serverless, containers | M2, M4, M6, M9, M10 | Terraform for all five categories; an ADR comparing VM vs Container Apps vs AKS |
| Docker / Kubernetes | M1, M6 | Hardened images, AKS operation, cluster upgrade, five pod-failure diagnoses |
| Terraform / IaC / automation | M4 onward | Versioned modules, drift detection in CI, a state recovery |
| Git, DevOps, CI/CD | M5 onward | Reusable workflows consumed by three repos; measured rollback time |
| Monitoring, logging, observability, troubleshooting | M7, M8 | Dashboards, SLOs, alerts with runbooks, 12+ worked incidents |
| Python, TypeScript, APIs, React, Node | `cargo-api`, `runway-portal` | Two deployed applications, both operated by you |
| Cloud workload onboarding, automation, self-service | M11 | The golden path + the `hello-svc` onboarding time |
| Security, governance, cost awareness | M3, M10 | Policy that blocked a real deploy; a cost anomaly you fixed; a threat model |
| Incident / problem / capacity / availability / continuity | M8, M9, and the ops playbook | Incident records, RCAs, a capacity plan, a measured DR drill |
| Documentation and operational procedures | Every milestone | Runbooks someone else executed; ADRs; an onboarding guide |
| AI/GenAI, agents, MCP | M12 | Flightdeck, its tool schemas, its eval results, its guardrail tests |
| Hybrid / private cloud (from the team description) | M10 | A local k3s cluster and VM governed through Azure Arc by the same policy |

Every JD line has a home. Nothing in the project exists without one — that is the test each component had to pass.

## 5. Explicitly out of scope

Written down so that scope creep has to argue with a document rather than with your mood at 11 p.m.

- No service mesh. No Argo CD or Flux until the optional extension.
- No multi-region active-active. One region, with a documented and *tested* region-loss recovery procedure — which is what real mid-size platforms actually have.
- No Kafka, no data pipeline, no warehouse.
- No model training, fine-tuning or GPU serving. Managed inference only.
- No microservice sprawl. Three workloads is the point; a fourth adds nothing.
- No mobile app, no public users, no authentication beyond Entra ID SSO for the portal.
- No custom Kubernetes operator.

## 6. Repository layout

Start as a monorepo — it is far easier to show a reviewer. Split `hello-svc` out into its own repository at Milestone 11, because cross-repository reusable-workflow consumption is the thing you need to prove.

```
runway/
├── README.md                     # architecture, what works, what you operate, the numbers
├── docs/
│   ├── adr/                      # ADR-001 cloud choice, ADR-002 K8s vs Container Apps, ...
│   ├── architecture/             # diagrams (source + rendered), data flow, trust boundaries
│   ├── runbooks/                 # one per alert, plus DR and onboarding procedures
│   ├── incidents/                # INC-001..., timelines, written during the drill
│   ├── problems/                 # RCA-001..., linked to the incidents they explain
│   ├── capacity/                 # load test reports, capacity plan
│   ├── cost/                     # monthly notes, cost-per-service, the anomaly write-up
│   └── security/                 # threat model, controls matrix, the credential-leak drill
├── platform/
│   ├── modules/                  # network, aks, postgres, keyvault, observability, workload
│   ├── environments/{dev,prod}/  # thin root modules; all logic lives in modules
│   ├── policy/                   # Azure Policy definitions + assignments
│   ├── observability/            # dashboards, alert rules, SLO definitions, as code
│   └── scripts/                  # Bash + Python + one PowerShell tool
├── .github/workflows/            # reusable: build-and-push, deploy-to-aks, tf-plan-apply, drift
├── workloads/
│   ├── cargo-api/                # Python/FastAPI + Postgres + worker
│   ├── carrier-sim/              # the failable external dependency
│   └── runway-portal/            # React SPA + Node/TypeScript BFF
└── flightdeck/                   # MCP server (read-only tools) + agent + evals
```

## 7. Relationship to your existing OpsPilot roadmap

They are different projects and should stay that way. OpsPilot is an **AI product**: RAG over runbooks, self-hosted inference, evaluation harnesses, diagnosis quality. Runway is a **platform**: the thing an AI product would be deployed *onto*. Flightdeck overlaps with OpsPilot only superficially — it does no retrieval, no self-hosting and no quality modelling; it is eight read-only tools and a bounded agent loop.

If you later want them to meet, the honest way is: deploy OpsPilot as a fourth workload through Runway's golden path. That would be a genuinely strong portfolio moment. It is not part of this roadmap.

→ Next: [Step 6 — Architecture](05-architecture.md)
