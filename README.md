# Cloud / Platform / DevOps Roadmap — **Runway**

*A single production-grade project that closes the gap between where you are and a Technical Engineer – Public Cloud role in Group IT (Platform Engineering) — and, more importantly, between where you are and being able to operate real cloud systems.*

---

## The short version

**The job**, in the posting's own words: *"unify control across datacentres, networks, public and private cloud environments"* with *"a strong emphasis on automation, agility, and cost optimization"*, plus *"incident management, problem management, service level management, capacity management, availability management, and IT service continuity."*

**Three things that shapes:**

1. It is **half build, half run.** Six ITIL processes are written into the role definition. Most cloud portfolios stop at "I deployed it"; this one starts there.
2. It is **hybrid**, not greenfield. There is a datacentre and it is not going away.
3. The platform **has customers** — *"cloud workload onboarding, automation and self-service capabilities"*. That is platform engineering, and it is the requirement almost no candidate can evidence.

**The project:** *Runway* — a small internal cloud platform on Azure, the two workloads that run on it, a third throwaway workload that proves onboarding generalises, and a strictly read-only AI operations assistant. The application layer stays deliberately boring; the platform is the subject.

**The plan:** 9 months at 8–12 h/week, on a $30–60/month Azure budget with build/destroy discipline. 15 milestones. Roughly 16 deliberately-injected incidents, each worked to a written record.

---

## The documents

| Step | File | What it settles |
|:--:|---|---|
| 1 | [Job analysis](00-job-analysis.md) | The full JD broken down, every requirement categorised and prioritised P0–P3, and the honest gap read |
| 2 | [Target profile](01-target-profile.md) | The seven adjacent roles drawn by their real boundaries, and where you should sit across them |
| 3 | [Skills gap](02-skills-gap.md) | 38 competencies at Beginner → Intermediate → Advanced → Production-ready, with your start and target |
| 4 | [Cloud choice](03-cloud-choice.md) | Why Azure, what the counter-arguments are, the concept→provider translation table, and the cost discipline |
| 5 | [The project](04-the-project.md) | Runway: the five components, why this project and not the obvious alternatives, what is out of scope |
| 6 | [Architecture](05-architecture.md) | The diagram, why each component exists, three request paths to know by heart, and what was deliberately excluded |
| 7 | [Learning path](06-learning-path-phases-00-05.md) | 18 phases. Concepts → labs → implementation → production reality → **failure exercises** → verification → deliverable. Also [6–11](07-learning-path-phases-06-11.md) and [12–17](08-learning-path-phases-12-17.md) |
| 8 | [Milestones](milestones/README.md) | 15 delivery units with learning topics, build, cloud resources, DevOps work, ops exercise, security, outcome and evidence |
| 9 | [Operations playbook](09-operations-playbook.md) | The Detect→Document loop, a severity matrix, templates, 16 scenarios, and three fully worked examples |
| 10 | [Platform engineering](10-platform-engineering.md) | Golden paths, module design as API design, reusable workflows, policy as the precondition for self-service |
| 11 | [AI in cloud operations](11-ai-in-cloud-operations.md) | Running an AI service as a workload; then agents, MCP, and read-only-by-construction |
| 12 | [Certifications](12-certifications.md) | Which two or three are worth it, when you will be ready, and how not to let them derail the build |
| 13–14 | [Timeline and weekly pattern](13-timeline-and-weekly-pattern.md) | 6-, 9- and 12-month versions with exactly what each drops; the dependency graph; the weekly rhythm; revision weeks |
| 15 | [Progress measurement](14-progress-measurement.md) | Explain → Build → Troubleshoot → Design → Operate, defined per technology, plus the twenty questions |
| 16 | [Portfolio](15-portfolio.md) | What the repository must contain, how to write the README, how to present it, and the honesty rule |
| 17 | [Career skill map](16-career-skill-map.md) | The final matrix, the four depth categories, and the progression to AI Platform Engineer |

---

## The five rules

1. **Understand before you type.** Mondays have no terminal. The roadmap is built around *why*, because the technologies will change and the judgement will not.
2. **Break it on purpose.** The failure exercises are not garnish — they are where most of the learning is, and they produce the incident records that make the portfolio unusual.
3. **Write it down while it is happening.** A timeline reconstructed afterwards is fiction. Friday is documentation day for exactly this reason.
4. **Measure, don't claim.** Every number in your README — rollback time, RTO, request ceiling, onboarding time, monthly cost — must be one you produced and can reproduce live.
5. **`make down` at the end of every session.** Your budget and your infrastructure-as-code discipline are the same habit.

## What makes this different from every other cloud roadmap

Most stop at Milestone 9. The last six — operations, capacity, continuity, governance and cost, the golden path, and a *bounded* AI assistant — are where the JD actually lives, and they are the ones you will be most tempted to skip because none of them produces an impressive screenshot.

Milestones 10 and 13 are the two you must not cut.

## Relationship to the other roadmap in this repository

`../roadmap/` is a separate, untouched **AI Engineer** track for OpsPilot: RAG, self-hosted inference, evaluation harnesses, diagnosis quality. Different career direction, different project. If the two ever meet, the honest way is to deploy OpsPilot as a fourth workload through Runway's golden path — a genuinely strong portfolio moment, and not part of this roadmap.
