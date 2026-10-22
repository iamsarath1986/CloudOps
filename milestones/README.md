# Section 8 — Detailed Milestones

**Current execution plan (2026-10-06):** deploy the existing FinanceTracker project and learn concepts as each deployment step needs them. These milestones are reference material, not required assignments. Standalone exercises, deliberate failure drills, design homework, and mentor-question gates are superseded by the [current execution plan](../README.md#current-execution-plan--financetracker-deployment).

There are twenty-three milestones, one per phase, and each one must leave something working in the Runway repository. **Phases are the syllabus and milestones are the unit of delivery.** [Section 7](../07-phase-roadmap.md) explains why each phase sits where it does. The files below say what you learn, what you build, what you break, what you hand in and how you prove you are finished.

The **track documents** ([§11–§20](../11-aws-learning-path.md)) follow one discipline, such as Terraform or observability, through every milestone that touches it. Use a milestone file to find out what to do this week. Use a track to see how one skill develops over two years.

## The milestones

~10 focused hours a week. Months assume 46 productive weeks a year, with revision weeks included.

| # | Milestone | Weeks | ≈ Months | You end up with |
|:-:|---|:-:|:-:|---|
| M00 | [Engineering foundations and the boring workload](M00-foundations.md) | 3 | 0–1 | A protected repo, ADR-001, `cargo-api` + `carrier-sim` running locally with tests |
| M01 | [Linux and networking](M01-linux-networking.md) | 5 | 1–2 | `net-triage.sh`, a CIDR plan, and the four TCP failure modes recognisable on sight |
| M02 | [Python, Bash and PowerShell automation](M02-automation.md) | 2 | 2–2.6 | Tested, packaged ops tooling others could use at 3 a.m. |
| M03 | [Cloud architecture fundamentals](M03-cloud-architecture.md) | 2 | 2.6–3.1 | ADR-002 (AWS first), the first SDD, availability and RTO/RPO maths you can do on a whiteboard |
| *R1* | *Revision* | 1 | 3.4 | |
| M04 | [AWS core infrastructure, identity-first, by hand](M04-aws-core.md) | 5 | 3.4–4.7 | An AWS Organization, a hand-built VPC/EC2/ALB/RDS stack, budgets, zero long-lived keys |
| M05 | [Terraform: from first apply to team practice](M05-terraform.md) | 5 | 4.7–6 | Everything in code, remote state with native locking, `make up`/`make down`, drift detection |
| M06 | [Docker, images and the supply chain](M06-containers.md) | 3 | 6–6.8 | Hardened, scanned, signed images in ECR, referenced by digest |
| M07 | [CI/CD and release engineering](M07-cicd.md) | 4 | 6.8–7.8 | Reusable workflows, OIDC to AWS, build-once-promote to ECS Fargate, measured rollback |
| *R2* | *Revision · Terraform Associate* | 1 | 8.1 | |
| M08 | [Kubernetes fundamentals](M08-kubernetes-fundamentals.md) | 3 | 8.1–8.9 | Every pod failure state diagnosed on kind; the ingress-nginx EOL replacement |
| M09 | [Production Kubernetes on EKS](M09-eks.md) | 5 | 8.9–10.2 | EKS with Karpenter, Pod Identity, ESO and the LB Controller; ECS→EKS migration with zero downtime |
| *R3* | *Revision · AWS SA Associate* | 1 | 10.4 | |
| M10 | [Observability](M10-observability.md) | 4 | 10.4–11.5 | OTel → AMP/Loki/Tempo → Grafana, dashboards and alerts as code |
| M11 | [SRE and production operations](M11-sre-operations.md) | 4 | 11.5–12.5 | SLOs, L1/L2 event management, incident/problem/change records from real drills |
| M12 | [Capacity, high availability and disaster recovery](M12-capacity-ha-dr.md) | 4 | 12.5–13.6 | A measured capacity ceiling, a Multi-AZ window, and a measured RTO/RPO from a pilot-light rebuild |
| *R4* | *Revision · CKA* | 1 | 13.8 | |
| M13 | [Azure core, landing zone and enterprise identity](M13-azure-core.md) | 4 | 13.8–14.9 | Management groups, hub-spoke VNet, Entra → IAM Identity Center, cross-cloud federation |
| M14 | [Production AKS and multi-cluster GitOps](M14-aks-multicluster.md) | 3 | 14.9–15.7 | AKS alongside EKS, both reconciled by Argo CD |
| *R5* | *Revision · AZ-104* | 1 | 15.9 | |
| M15 | [Cloud security and governance](M15-security-governance.md) | 4 | 15.9–17 | Policy at four enforcement points; threat model; leaked-credential drill |
| M16 | [FinOps](M16-finops.md) | 3 | 17–17.8 | FOCUS-normalised multi-cloud cost, OpenCost, showback, an anomaly you attributed |
| *R6* | *Revision* | 1 | 18 | |
| M17 | [Platform engineering: golden paths](M17-golden-paths.md) | 4 | 18–19.1 | `workload.yaml` → a governed, observed service on either cloud, timed |
| M18 | [Internal developer platform and self-service](M18-developer-platform.md) | 4 | 19.1–20.1 | runway-portal, service and data catalogues, PR-based self-service, account vending |
| *R7* | *Revision · AZ-305 prep window opens* | 1 | 20.4 | |
| M19 | [AI infrastructure and AI workloads](M19-ai-infrastructure.md) | 4 | 20.4–21.4 | ai-gateway with cross-cloud fallback, async AI worker, AI SLOs and cost per request |
| M20 | [AI agents and MCP for operations](M20-ai-agents-mcp.md) | 3 | 21.4–22.2 | flightdeck: a read-only MCP server and bounded agent, evaluated against your own incidents |
| *R8* | *Revision · AWS SA Professional or AZ-305* | 1 | 22.5 | |
| M21 | [Hybrid and multi-cloud architecture](M21-hybrid-multicloud.md) | 3 | 22.5–23.2 | A "datacentre" under Arc/SSM, VPN and hybrid DNS windows, a 6 Rs migration assessment |
| M22 | [Production hardening, final architecture and capstone](M22-hardening-capstone.md) | 3 | 23.2–24 | A whole-system threat model, a final game day, and an independently designed capstone SDD |
| *R9* | *Final review · readiness checklist* | 1 | 24.3 | |

**Totals:** 84 build weeks and 9 revision weeks, 93 productive weeks in all. That is **about 24 months** at 8–12 hours a week. [Section 22](../22-timeline.md) gives the ~20-month fast track and the ~30-month comfortable version, and names exactly what each one cuts.

**Checkpoints on the way:**

| Month | Where you are |
|:-:|---|
| ~8 | AWS, Terraform and CI/CD. You could hold a junior cloud or DevOps role. |
| ~13 | You operate EKS in production using SRE practice. This is mid-level Cloud/DevOps Engineer. |
| ~18 | Both clouds, security, governance and FinOps. |
| ~24 | The senior-target profile. |

## Revision weeks

Every revision week follows the same pattern, and you build nothing new in it:

1. **Reproduce from memory.** Re-run three troubleshooting exercises from earlier milestones without notes. Anything you cannot reproduce, you did not really learn, so go back and redo it.
2. **Read your own documentation as a stranger would.** Follow one runbook literally. Every place you stumble is a defect, so fix it.
3. **Refresh the numbers.** Update rollback time, RTO/RPO, request ceiling, onboarding time, monthly cost and cost per unit. A stale number was never a real one.
4. **Score the [competency matrix](../24-competency-matrix.md)**, and mark yourself down whenever you are in doubt.
5. **Take at most one exam**, if one is due ([§21](../21-certification-strategy.md)).
6. **Rest.** Burning out in month five is the most common way a two-year plan fails.

## How every milestone file is structured

Each file uses the same sections in the same order, so you always know where to look:

| Section | What it holds |
|---|---|
| Why this phase exists | The JD lines it serves, quoted, and its place in the dependency chain |
| Concepts to learn | The theory, including how things work internally |
| Architecture decisions | **Problem → Options → Decision → Trade-offs** for the ADRs in this phase (global numbering 001–027) |
| Hands-on labs | Small throwaway experiments, each with what it proves |
| Project implementation | What is added to Runway, with paths |
| Engineering problems to hit deliberately | Problems you cause on purpose, how to cause them, and what they teach |
| Troubleshooting exercises | Realistic incidents described as the symptom you would see. Scenario IDs S01–S36 are catalogued in [§10](../10-failure-and-troubleshooting.md) |
| Deliverables | A checklist of concrete artefacts |
| Completion criteria | Measurable tests, never "followed the tutorial" |
| Mentor checkpoints | What to bring for review, the questions you will be asked, and the category of failure injected after you think you are done |
| Resources | Two to five items, with the exact sections to read |
| Estimate vs actual | Planned against actual hours and cost, which is practice for guiding project managers on "agreed costs and timescales" |

## The rules

1. **A milestone is finished when its completion criteria are met and you can answer its mentor questions out loud without notes.** Code that runs is not enough. If you hedge, you are not finished.
2. **Every milestone produces something in Git.** None of them is purely theory. Every concept ends up built, deployed, operated, measured, secured or troubleshot on the *same* system.
3. **The engineering problems and troubleshooting exercises are not optional.** Most of the learning happens there, and they produce the incident, problem and change records that make your portfolio unusual.
4. **Write it down while it is happening.** An incident timeline written after the fact is fiction. Major incidents use **Detection → Investigation → Mitigation → Root Cause → Permanent Fix → Prevention**.
5. **Design before you build.** Bring the interface, diagram or ADR draft to the mentor checkpoint *before* writing code. The mentor reviews your attempt and does not hand you a solution unless you ask for one.
6. **`make down` at the end of every session**, and `make hybrid-down` from M21 onwards. Your budget and your IaC discipline are the same habit.
7. **Never cut M11 or M17.** They are what the JD's operational and platform-engineering halves are made of. They are also the two you will be most tempted to skip, because neither produces an impressive screenshot. M18's portal UI and M21 are the safest reductions.

→ Next: [Section 9 — Hands-on labs](../09-hands-on-labs.md)
