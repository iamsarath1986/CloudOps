# Step 8 — Milestones

Fifteen milestones, one per learning phase. **Phases are the syllabus; milestones are the delivery unit.** The phase tells you what to learn and what to break; the milestone tells you what must exist and what you must be able to explain when it is finished.

| # | Milestone | Phase | Weeks | You end up with |
|---|---|:--:|:--:|---|
| 1 | [Foundations and the workload](milestone-01-foundations.md) | 0 | 2 | A repo, an ADR, and a boring service that runs locally |
| 2 | [Linux and the network](milestone-02-linux-networking.md) | 1 | 3 | The diagnostic skills every later incident depends on |
| 3 | [First cloud footprint, by hand](milestone-03-first-cloud.md) | 2 | 3 | A live, paid-for Azure environment you built by clicking |
| 4 | [Identity, access and secrets](milestone-04-identity.md) | 3 | 2.5 | Zero stored credentials anywhere |
| 5 | [Containers and the registry](milestone-05-containers.md) | 4 | 2 | Hardened, scanned images in ACR, referenced by digest |
| 6 | [Everything as code](milestone-06-iac.md) | 5 | 4 | `make up` / `make down`; nothing exists outside Terraform |
| 7 | [Continuous delivery](milestone-07-cicd.md) | 6 | 3 | Build once, promote by digest, measured rollback |
| 8 | [Kubernetes](milestone-08-kubernetes.md) | 7 | 5 | An AKS cluster you can diagnose and upgrade |
| 9 | [Observability](milestone-09-observability.md) | 8 | 3 | Dashboards, SLOs and alerts that each have a runbook |
| 10 | [Reliability and operations](milestone-10-operations.md) | 9 | 4 | Ten incident records and four RCAs from real drills |
| 11 | [Capacity and continuity](milestone-11-capacity-continuity.md) | 10, 11 | 5.5 | A measured ceiling, a capacity plan, and a tested DR runbook |
| 12 | [Governance, cost and hybrid control](milestone-12-governance-cost.md) | 12 | 3 | Policy that blocks real deploys; a cost anomaly you fixed; Arc |
| 13 | [The golden path](milestone-13-golden-path.md) | 13 | 4 | A workload you did not write, onboarded in under a day |
| 14 | [AI operations](milestone-14-ai-operations.md) | 14, 15 | 5.5 | An AI service under SLOs and a read-only MCP agent |
| 15 | [Hardening and close-out](milestone-15-hardening.md) | 16, 17 | 4.5 | A threat model, a security drill, and a portfolio that reads well |

**≈54 weeks at 8–12 focused hours per week.** See [Step 13](../13-timeline-and-weekly-pattern.md) for the 6-, 9- and 12-month variants and exactly what each one drops.

## The rules

1. **A milestone is finished when you can do the "Evidence of mastery" section without notes** — not when the code runs. Say the answers out loud. If you hedge, you are not finished.
2. **Every milestone produces something in Git.** No milestone is theoretical; every concept ends up built, deployed, operated, measured, secured or troubleshot.
3. **The failure exercises are not optional.** They are where most of the learning is, and they are what produce the incident records that make your portfolio unusual.
4. **Write it down while it is happening.** Incident timelines written after the fact are fiction. The habit of writing during is itself an operational skill.
5. **Destroy the environment at the end of every session.** `make down`. Your budget and your IaC discipline are the same habit.
