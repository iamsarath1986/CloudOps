# Cloud / Platform / DevOps Roadmap — **Runway**

## Current execution plan — FinanceTracker deployment

Updated: 2026-10-06.

We will learn cloud and platform engineering by deploying and operating the existing **FinanceTracker** project at `/home/user/PycharmProjects/FinanceTracker`. It already runs locally in Docker. We will reuse its Vue frontend, FastAPI backend, PostgreSQL database, and production Docker Compose configuration rather than build the Runway sample application first.

This plan takes precedence over the original milestone sequence and mentor rules below:

1. Work toward a real deployment in small, concrete steps. Explain each concept when that step needs it, then guide the implementation and explain the result.
2. Do not assign understanding quizzes, standalone practice exercises, or deliberate failure drills. Check actual deployment behavior through builds, health checks, application use, and logs.
3. Do not require ADR drafts, design homework, or completion-criteria questions before deployment work. Record useful decisions and operating instructions as the project progresses.
4. Hosting target: **AWS**, confirmed on 2026-10-06. Start with the existing production Compose stack on one EC2 instance. Confirm account access, budget, domain, and whether existing data must be transferred before provisioning. Choose the region and instance size using the budget and application requirements.
5. Introduce networking, identity, Linux administration, DNS, HTTPS, secrets, persistent storage, backups, and logs as the deployment requires them. Add infrastructure as code, CI/CD, observability, and managed services as subsequent project increments. Revisit Kubernetes and multi-cloud when there is a concrete reason to introduce them.
6. Preserve application data. The original automatic `make down` rule does not apply to the deployed FinanceTracker service; teardown requires an explicit purpose and a data-preservation plan.

The documents below remain a reference curriculum and the original Runway proposal. Their application-building assignments, fixed dates, mandatory drills, and mentor checkpoints are not the active work plan.

---

*One production-grade, multi-cloud platform, built and operated over roughly two years alongside a full-time job. It closes the gap between where you are and the **Senior Technical Engineer – Public Cloud** role (Group IT, Platform Engineering). More importantly, it closes the gap between you and being able to design, run and troubleshoot real cloud platforms and the AI workloads on them.*

---

## The short version

**The job**, in the posting's own words: *"design, develop, and deliver secure, scalable, and reliable public cloud solutions"* on **AWS and Azure**, acting as *"a technical expert within Public Cloud, supporting cloud automation, workload onboarding, platform engineering, and the development of internal self-service capabilities"*, while driving *"Infrastructure as Code, CI/CD, observability, governance, and FinOps practices"*.

**What the senior version of the role changes:**

1. **Two clouds, operated at proficiency.** Not a translation exercise at the end. Both AWS and Azure are named, and so are EKS and AKS.
2. **Design authority.** Solution architecture to EA standards, guiding project managers on cost and timescales, and owning changes and end-of-life replacement. You are expected to decide, not just implement.
3. **Platform as a product.** Self-service portals, internal developer tools and governed onboarding of applications onto a *"large public cloud footprint"*.
4. **Operations.** Incidents resolved within service levels, problem management, changes you own, L1/L2 monitoring configured in the event management system (EMS), and documentation that includes a data catalogue.

**The project:** *Runway* is an internal platform spanning AWS (EKS) and Azure (AKS). It provides a golden path and a self-service portal, and runs a deliberately boring Python workload plus a shared AI gateway and a strictly read-only AI operations assistant (MCP). No static credentials exist anywhere in it, not even between clouds.

**The plan:** 23 milestones over **about 24 months** at 8–12 h/week (84 build weeks + 9 revision weeks). **AWS comes first, Azure second.** Around 36 deliberately injected failure scenarios, each worked to a written record. The budget is roughly $50–90/month, kept down by build/destroy discipline.

---

## The documents

| § | File | What it settles |
|:--:|---|---|
| 1 | [JD skills analysis](01-jd-skills-analysis.md) | Every requirement categorised, with its expected depth and the evidence in the JD for that depth; the prerequisites the JD assumes but never names |
| 2 | [Skills gap and prerequisites](02-skills-gap-and-prerequisites.md) | Where you are and where you need to be on the L0–L5 scale, and the gaps that matter most |
| 3 | [Target engineer profile](03-target-engineer-profile.md) | Adjacent roles, the blend you are aiming for, depth tiers, what you are *not* becoming |
| 4 | [Learning dependency map](04-learning-dependency-map.md) | What must come before what, and why |
| 5 | [Production project proposal](05-production-project-proposal.md) | Runway: its components, why this project, JD→project mapping, how it evolves |
| 6 | [Final project architecture](06-final-project-architecture.md) | Diagrams, Problem → Options → Decision → Trade-offs, request paths, what was excluded |
| 7 | [Phase-by-phase roadmap](07-phase-roadmap.md) | The 23 phases: purpose, placement, what each unlocks |
| 8 | [Detailed milestones](milestones/README.md) | M00–M22: concepts, labs, build, deliberate problems, troubleshooting, deliverables, completion criteria, resources |
| 9 | [Hands-on labs](09-hands-on-labs.md) | The lab system and an index of every key lab |
| 10 | [Failure and troubleshooting](10-failure-and-troubleshooting.md) | The operations playbook: the incident loop, severity, templates, scenarios S01–S36, worked examples |
| 11 | [AWS learning path](11-aws-learning-path.md) | AWS across the whole roadmap |
| 12 | [Azure learning path](12-azure-learning-path.md) | Azure as the second cloud, taught through its differences from AWS |
| 13 | [Terraform learning path](13-terraform-learning-path.md) | From first apply to the way a platform team runs Terraform |
| 14 | [Docker & Kubernetes learning path](14-docker-kubernetes-learning-path.md) | Container internals → EKS → AKS → multi-cluster |
| 15 | [CI/CD learning path](15-cicd-learning-path.md) | Release engineering, testing, GitOps, change records |
| 16 | [Observability & SRE learning path](16-observability-sre-learning-path.md) | Signals, SLOs, alerting, ITIL processes in engineering terms |
| 17 | [Security & governance learning path](17-security-governance-learning-path.md) | Identity, supply chain, policy as code, landing zones, threat models |
| 18 | [Platform engineering learning path](18-platform-engineering-learning-path.md) | Manual ops → golden paths → internal developer platform |
| 19 | [FinOps learning path](19-finops-learning-path.md) | Allocation, optimisation, Kubernetes and AI cost, bill investigations |
| 20 | [AI infrastructure learning path](20-ai-infrastructure-learning-path.md) | Operating AI workloads, agents and MCP, and how SRE practice changes for them |
| 21 | [Certification strategy](21-certification-strategy.md) | Which certifications, when, and why they come after the build work |
| 22 | [12–24 month timeline](22-timeline.md) | Monthly milestones, how the project progresses, the variants, what to cut when behind |
| 23 | [Weekly study routine](23-weekly-study-routine.md) | The weekly rhythm, weekend sessions, revision weeks, mentor sessions |
| 24 | [Competency matrix](24-competency-matrix.md) | L0–L5 per skill, the evidence needed to move up, target levels by month |
| 25 | [Production readiness checklist](25-production-readiness-checklist.md) | The final review, and the "design it yourself" capability test |

**Appendices:** [A1 — Cloud architecture concepts & AWS↔Azure mapping](appendix/A1-cloud-architecture-concepts.md) · [A2 — Documentation standards & templates](appendix/A2-documentation-standards.md) · [A3 — Portfolio and interview](appendix/A3-portfolio-and-interview.md)

---

## How we work: mentor mode

This roadmap is built to be followed *with* a mentor, not read like a textbook. For every major task:

1. I explain the concept, why we need it, how it works internally, the architecture, the alternatives and the trade-offs.
2. You get the implementation task and **attempt it yourself**. You only get complete solutions if you explicitly ask for them.
3. I review your work, point out problems, and help you improve it.
4. Then I inject a failure, you troubleshoot it, and we compare your approach with how a production engineer would handle it.

Every milestone ends with a **Mentor checkpoints** section listing what to bring for review and the kind of failure that will follow.

## The six rules

1. **Understand before you type.** Mondays have no terminal. The technologies will change; the judgement will not.
2. **Break it on purpose.** The failure exercises are where most of the learning happens, and they produce the incident records that make the portfolio unusual.
3. **Write it down while it is happening.** A timeline reconstructed afterwards is fiction.
4. **Measure, don't claim.** Every number (rollback time, RTO/RPO, request ceiling, onboarding time, cost per request, monthly spend) must be one you produced and can reproduce live.
5. **One cloud's concept at a time.** Learn the idea, build it on AWS, then learn Azure through its differences. Never do every exercise twice.
6. **`make down` at the end of every session.** Your budget discipline and your infrastructure-as-code discipline are the same habit.

## What makes this different from other cloud roadmaps

Most stop once Kubernetes is running. The part of this JD that separates senior candidates comes after that: operations and change ownership (M11), capacity and continuity (M12), governance and FinOps (M15–M16), the golden path and self-service portal (M17–M18), and a *bounded* AI layer (M19–M20). None of these produces an impressive screenshot, so they are the ones you will be most tempted to skip.

**M11 and M17 are the two you must not cut.**

## Relationship to the other roadmap in this repository

`../roadmap/` is a separate, untouched **AI Engineer** track for OpsPilot: retrieval-augmented generation (RAG), self-hosted inference, evaluation harnesses and diagnosis quality. It is a different career direction and a different project. If the two ever meet, the honest way is to onboard OpsPilot as another workload through Runway's golden path. That would be a genuinely strong portfolio moment, but it is not part of this roadmap.
