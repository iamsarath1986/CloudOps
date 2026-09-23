# Section 3 — Target Engineer Profile

## 1. The adjacent roles, drawn by their real boundaries

Job titles blur between companies. Two things don't: **what the role is accountable for when something breaks at 3 a.m.**, and **who its customer is**. Use those two questions to tell the roles apart.

| Role | Primary customer | Accountable for | Typical artefact | Fails when |
|---|---|---|---|---|
| **Cloud Engineer** | Project and business teams | Cloud resources that exist, are configured correctly and stay healthy | A provisioned, secured landing zone | Resources are inconsistent, undocumented, or clicked into existence |
| **DevOps Engineer** | Development teams | A fast, safe, repeatable path from commit to production | Pipelines, build and deployment strategy | Deploys are manual, slow or unrecoverable |
| **Platform Engineer** | *Other engineers, as users of a product* | A reusable, opinionated, self-service path that many teams use | Golden path: modules + templates + portal + defaults | Teams route around the platform because it's slower than doing it by hand |
| **Site Reliability Engineer** | End users of the service | Reliability as a measured number; error budgets; toil reduction | SLOs, burn-rate alerts, automation that removes toil | Reliability is an opinion rather than a metric |
| **Cloud Architect** | Delivery programmes and leadership | Design decisions, standards and trade-offs across many systems | Reference architectures, SDDs, ADRs, cost models | Designs are unimplementable, or nobody owns them |
| **Cloud Security Engineer** | Risk owners, auditors | Controls that measurably reduce risk | Threat models, guardrails, detections | Controls exist on paper and not in code |
| **AI Engineer** | Product teams | An AI feature that's correct, fast, safe and affordable | Prompts, tool schemas, evals, retrieval | Impressive in a demo, unreliable in production |
| **AI Platform / AI Infrastructure Engineer** | AI engineers and product teams | Calling or serving models being secure, observable, rate-limited, budgeted and recoverable | AI gateway, quotas, serving infrastructure, AI telemetry, eval gates | Nobody can say what AI costs, who is using it, or what happens when the model is down |

Three distinctions you should be able to state in an interview:

- **DevOps vs Platform Engineering.** DevOps is a *practice* any team can adopt. Platform Engineering is a *team structure* that turns that practice into a product for other teams. *"Drive the adoption of Infrastructure as Code, CI/CD, observability, governance, and FinOps practices"* is platform language: you're accountable for other teams adopting these practices, not only for doing them yourself.
- **Senior Engineer vs Architect.** The JD asks you to *"develop solution architecture … in accordance with established EA standards"*. You **work within** enterprise architecture standards and write solution designs. You don't set enterprise strategy. That's the next rung up.
- **AI Engineer vs AI Platform Engineer.** The AI Engineer decides what the model is asked and whether the answer is good. The AI Platform Engineer makes sure calling the model is governed, observable, budgeted and survives outages. **This JD, and your stated direction, both point at the second.**

## 2. Where your target profile sits

> **A senior multi-cloud Platform Engineer (AWS + Azure) who designs, automates, operates and governs production platforms, runs them with genuine ITIL/SRE discipline, enables other teams through golden paths and self-service, and can operate AI workloads on that platform with the same rigour as everything else.**

No single title covers this, so here it is as a blend:

```
Platform Engineer              ████████████████████  35%
Cloud Engineer (AWS + Azure)   ██████████████        25%
SRE / Production Operations    ██████████            18%
Solution/Cloud Architecture    ██████                10%
Security & Governance / FinOps ████                   7%
AI Platform Engineer           ███                    5%
```

The AI share is small **on purpose**. It's what makes you memorable, not what makes you qualified. Over time it grows into your specialisation (§6).

**Why this blend and not "DevOps Engineer":** incident, problem and change management, EMS and the service-level list are service-management language. Large airlines, banks and telcos run this way. If you present as someone who does pipelines and Terraform with no operational story and no design documents, you'll come across as mid-level to this employer, whatever your technical skill.

**Why architecture appears at all:** the senior posting adds *design*, *solution architecture*, *EA standards* and *guiding PMs on cost and timescales*. You need to own designs at the scale of a solution: an SDD, its ADRs, its cost estimate and its NFRs. You don't need to own enterprise strategy.

## 3. Depth tiers: you won't learn everything equally

Each tier has a test, written as something you can do rather than something you feel.

### Tier 1 — Deep expertise (target L5)
*Test: you can design it, operate it under failure, and teach it at a whiteboard without notes.*

| Area | Why it earns depth |
|---|---|
| **Terraform and IaC at team scale** | Every other cloud claim is checked by whether it's in code. It's also the thing you "drive the adoption of" |
| **Cloud networking across AWS, Azure and the datacentre** | The most common source of real incidents, the most portable knowledge, and the core of "unify control across … networks" |
| **Kubernetes operation on EKS and AKS** | Named explicitly. *Operating* it, as opposed to installing it, is where most candidates are weak |
| **CI/CD, release engineering and change management** | "Automated deployment" plus "own and manage changes": one discipline seen from two sides |
| **Observability and cross-layer troubleshooting** | What the operational half of the job is made of |
| **Platform engineering and golden paths** | Both the team's name and the requirement most candidates can't evidence |
| **Identity and federation across clouds** | The thing that makes a multi-cloud platform either secure or a credentials sprawl |

### Tier 2 — Strong production proficiency (target L4)
*Test: you use it confidently in production and debug your own mistakes, but check the docs for edge cases.*

AWS and Azure service operation beyond the core · incident, problem and event management · SLOs and error budgets · HA/DR and capacity · security hardening and threat modelling · policy-as-code and governance · FinOps and cost allocation · Docker and supply chain · Argo CD · Python automation · Bash · API/backend design · solution design documents and estimation · lifecycle/EOL management · the developer portal (React + Node/TS) · AI gateway operation.

### Tier 3 — Working knowledge (target L3)
*Test: you can build it and explain the trade-offs, and you'd want a review before running it in production.*

PowerShell · Azure Arc and AWS SSM hybrid · site-to-site VPN and hybrid DNS · AI agents and MCP · self-hosted inference · Kyverno policy authoring beyond the basics · OpenCost · Backstage (evaluated, not adopted).

### Tier 4 — Conceptual (target L1–L2)
*Test: you can explain what it is, when you'd reach for it, and why you haven't.*

Service mesh · Kubernetes operators and CRD authoring · multi-region active-active · cross-cloud DR · GPU serving at scale · Control Tower internals · compliance frameworks (ISO 27001, PCI DSS) at awareness level · GCP mapping.

## 4. What you are explicitly *not* becoming

Say this once, because the usual way these roadmaps fail is quiet scope creep:

- **Not a data scientist or ML engineer.** No training loops, fine-tuning or model architecture.
- **Not a product developer.** React and Node are here to build *internal platform tooling*, which is the context the JD implies.
- **Not a dedicated security engineer.** You'll implement controls competently and think in threat models. You won't run a red team.
- **Not an enterprise architect.** You'll write solution designs within standards. You won't set the standards for a group of companies.
- **Not a certification collector.** Three Professional/Expert certifications *validate* the project. They don't replace it ([Section 21](21-certification-strategy.md)).

## 5. The progression ladder, with month checkpoints

Months assume the 24-month plan in [Section 22](22-timeline.md) at ~10 h/week.

```
Software / IT Engineer                                    ← today
  Python · APIs · PostgreSQL · Docker · testing · some LLM work
      ↓  M00–M05: Linux, networking, automation, cloud architecture, AWS by hand, Terraform
Cloud Engineer (AWS)                                      ← month ~6
  provisions and secures AWS infrastructure, entirely in code
      ↓  M06–M09: containers, CI/CD, Kubernetes, EKS
Cloud / DevOps Engineer                                   ← month ~10   (junior–mid cloud roles are realistic)
  owns the path from commit to a production EKS cluster
      ↓  M10–M12: observability, SRE and ITIL operations, capacity/HA/DR
Mid-level Cloud / DevOps / SRE Engineer                   ← month ~13-14
  operates production with measured SLOs, incident/problem/change records, tested DR
      ↓  M13–M16: Azure, AKS + GitOps, security & governance, FinOps
Multi-cloud Cloud / Platform Engineer                     ← month ~18
  runs AWS and Azure under one identity, policy, delivery and cost model
      ↓  M17–M18: golden paths, developer portal, account vending
Senior Platform Engineer — THE JD'S LEVEL of capability   ← month ~20
  other teams build on what you designed
      ↓  M19–M22: AI infrastructure, agents/MCP, hybrid, hardening, capstone
Senior Platform Engineer who operates AI workloads        ← month ~24
      ↓  real production years
AI Platform Engineer / Principal Platform Engineer / Cloud Architect
```

**Be honest about tenure.** The JD asks for 5+ years in a related IT engineering role. You have the years; what you lack is cloud-operating evidence. This roadmap builds capability and a record of evidence. It doesn't replace production tenure, and in an interview you should say that plainly rather than blur it. A repository full of real incident records, drills and designs is the strongest honest substitute that exists.

## 6. Where it leads afterwards

Pick **one** specialisation, and only after a year of real platform work:

| Direction | Where it leads | Start with |
|---|---|---|
| **AI platform** (your stated goal) | AI platform / AI infrastructure engineer | Self-hosted inference at scale, GPU capacity and scheduling, model serving, evaluation platforms |
| **Kubernetes depth** | Cluster platform engineer | CKS, writing a controller, multi-cluster fleet management |
| **Architecture** | Cloud architect, principal engineer | Landing zones at scale, enterprise integration, cost-modelling programmes |
| **SRE** | Reliability lead | Error budgets as policy, chaos engineering, advanced capacity modelling |
| **Cloud security** | Cloud security engineer | Detection engineering, supply-chain security, identity threat detection |

AI platform roles hire people who can operate infrastructure and happen to understand models far more often than the reverse. So be an excellent platform engineer first. This roadmap puts you in position for the AI move without spending your first year on it.

→ Next: [Section 4 — Learning Dependency Map](04-learning-dependency-map.md)
