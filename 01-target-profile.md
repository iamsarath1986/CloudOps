# Step 2 — The Engineer You Should Become

## 1. The seven roles, drawn by their actual boundaries

Titles blur between companies. What does not blur is **what the role is accountable for when something goes wrong at 3 a.m.**, and **who the customer is.** Use those two questions to separate them.

| Role | Primary customer | Accountable for | Typical artifact | Fails when |
|---|---|---|---|---|
| **Cloud Engineer** | The business / project teams | Cloud resources existing, configured correctly, and staying healthy | A provisioned, secured landing zone | Resources are inconsistent, undocumented, or clicked into existence |
| **DevOps Engineer** | Development teams | The path from commit to production being fast, safe, repeatable | A pipeline, a build, a deployment strategy | Deploys are manual, slow, or unrecoverable |
| **Platform Engineer** | *Other engineers, as users of a product* | A reusable, opinionated, self-service path that many teams travel | A golden path: modules + templates + portal + defaults | Teams route around the platform because it is slower than doing it by hand |
| **Site Reliability Engineer** | End users of the service | Reliability as a measured number; error budgets; toil reduction | SLOs, burn-rate alerts, automation that deletes repetitive work | Reliability is an opinion rather than a metric |
| **Cloud Architect** | Executives and delivery programmes | Design decisions, standards, and trade-offs across many systems | Reference architectures, ADRs, standards, cost models | Designs are unimplementable or unowned |
| **AI Engineer** | Product teams | An AI-powered feature being correct, fast, safe and affordable | Prompts, tool schemas, evals, guardrails, retrieval | The feature is impressive in a demo and unreliable in production |
| **MLOps / AI Platform Engineer** | Data scientists and AI engineers | The lifecycle of models and AI services in production | Model registry, serving infra, GPU capacity, AI observability, eval gates | Models cannot be deployed, versioned, rolled back or measured |

Two clarifications that matter for your decisions later:

- **DevOps vs Platform Engineering.** DevOps is a *practice* (short feedback loops, shared ownership) that any team can adopt. Platform Engineering is an *organisational structure* — a team that productises that practice for others. The JD sits squarely on the platform side: "workload onboarding, automation and self-service capabilities" is product language.
- **AI Engineer vs AI Platform Engineer.** The AI Engineer decides what the model is asked and whether the answer is good. The AI Platform Engineer makes sure calling it is secure, observable, rate-limited, budgeted and recoverable. **This JD wants the second one, at exposure level.** The JD says "AI/GenAI, AI agents or MCP integrations" — integration, not modelling.

## 2. Where your target profile sits

> **Platform Engineer on an Azure-centric hybrid cloud, with genuine operational (ITIL) competence, who can safely run AI-powered services as workloads on that platform.**

Expressed as a blend, because no single title captures it:

```
Platform Engineer        ████████████████████  45%
Cloud Engineer           ██████████████        30%
SRE / Operations         ████████              15%
AI Platform Engineer     ████                  10%
```

Notice what is *not* in the blend: Cloud Architect (that is where this role leads in 3–5 years, not where you enter), AI Engineer (prompt/eval craft is useful context, not your product), and Data/ML (not on this path at all).

**Why this blend and not "DevOps Engineer":** the JD's operational half — incident, problem, capacity, availability, continuity — is not DevOps vocabulary. It is service-management vocabulary. Large airlines, banks and telcos run this way. If you present yourself as a pipeline-and-Terraform person with no operational story, you will read as junior to this specific employer regardless of your skill.

## 3. Depth tiers — you are not going to learn everything equally

Three tiers. The rule for each is written as a test, not a feeling.

### Tier 1 — Deep expertise
*Test: you can design it, operate it under failure, and teach it to someone else without notes.*

| Area | Why it earns depth |
|---|---|
| **Terraform and infrastructure as code** | Every other cloud skill you claim is verified by whether it is in code. This is the highest-leverage single skill on the list. |
| **Azure networking** (VNet, subnets, NSG, UDR, DNS, private endpoints, load balancing, TLS) | The most common source of real incidents and the hardest thing to learn from a tutorial. It is also the most portable — the model is the same on every cloud. |
| **Kubernetes operation** (not installation) | The JD names it, the industry runs on it, and *operating* it is where almost everyone is weak. |
| **CI/CD and release engineering** | "Efficient deployment of solutions" is the literal role definition. |
| **Observability and troubleshooting** | The skill the operational half of the job is actually made of. |
| **Incident and problem management** | Six ITIL processes are in the role definition. Nearly no self-taught candidate has this. It is your cheapest differentiator. |
| **Platform/golden-path design** | The explicit requirement most candidates cannot evidence at all. |

### Tier 2 — Strong working proficiency
*Test: you use it confidently in production and can debug your own mistakes, but you would check documentation for the edge cases.*

Azure identity and RBAC · Key Vault and secret lifecycle · Azure Policy · Cost management and FinOps · Docker image hardening · Linux administration · Python automation and tooling · Bash · Backup, restore and DR execution · Load testing and capacity planning · API design · Azure Monitor/KQL · Azure Container Apps and Functions · TypeScript/Node/React (enough to build and operate a real internal UI) · Azure Arc · LLM API integration, structured output, guardrails, cost and latency control.

### Tier 3 — Conceptual familiarity
*Test: you can explain what it is, what problem it solves, when you would reach for it, and why you have not.*

PowerShell (read and modify; write when needed) · AWS and GCP service mapping · GitOps tooling (Argo CD/Flux) · Service mesh · Kubernetes operators and CRDs · ExpressRoute and enterprise WAN · Vector databases and RAG · Model fine-tuning and training infrastructure · eBPF and advanced Linux tracing · Compliance frameworks (ISO 27001, PCI-DSS) at awareness level.

## 4. What you are explicitly *not* becoming

Say this out loud once, because the most common way these roadmaps fail is silent scope creep:

- Not a data scientist or ML researcher. No training loops, no fine-tuning, no model architecture.
- Not a full-stack product developer. React and Node exist here to build *internal platform tooling*, which is exactly the context the JD implies.
- Not a security specialist. You will implement controls competently and think in threat models; you will not run a red team.
- Not a multi-cloud expert. One cloud, deeply. A second one is a two-week translation exercise at the end, not a second curriculum.

## 5. The progression this profile sits on

```
Software / IT Engineer            ← you are here, with real Python + API + Docker depth
        ↓  add: cloud, Linux, networking, IaC
Cloud Engineer                    ← Milestones 1–5 of this roadmap
        ↓  add: pipelines, Kubernetes, observability
DevOps / Platform Engineer        ← Milestones 6–10; this is the JD's level
        ↓  add: golden paths, governance, FinOps, operational ownership
Senior Cloud / Platform Engineer  ← Milestone 11, plus real production years
        ↓  add: AI workload operation, agents, AI observability and cost
AI Platform / Cloud Engineer      ← Milestone 12 and beyond
```

The JD sits at the third rung. This roadmap takes you through the third rung and gives you the evidence for the fourth and fifth, which is what makes you interesting rather than merely qualified.

→ Next: [Step 3 — Skills gap analysis](02-skills-gap.md)
