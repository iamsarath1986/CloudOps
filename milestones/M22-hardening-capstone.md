# Milestone M22 — Production Hardening, Final Architecture and Capstone

*Phase 22 · ~3 weeks (+ final revision week R9) · Months ~23–24 · Prerequisites: everything · Unlocks: the job, and the [production readiness checklist](../25-production-readiness-checklist.md)*

## Why this phase exists

The JD describes a senior engineer who can *"design, develop, and deliver secure, scalable, and reliable public cloud solutions"* and who can *"ensure that the developed architecture/solution meets service-level requirements in terms of expected availability, capacity, continuity, security, performance, and reliability"*. It also expects you to *"develop solution architecture for assigned projects in accordance with established EA standards"*.

This milestone tests those claims three ways:
1. **Harden** Runway as a whole system rather than one component at a time.
2. **Consolidate** the final architecture and its documentation, so that someone who is not you could operate it.
3. **Design something new from nothing**, on your own. The capstone is the brief's final capability test: given a fresh requirement, you design the architecture, choose the services, build the network, identity, Terraform and container strategy, and plan Kubernetes, CI/CD, secrets, security, monitoring, logging, alerting, scaling, reliability, backup, DR, cost and operations. Then you defend it.

## Concepts to learn

This phase is mostly *integration*, but four ideas carry it.

**Whole-system threat modelling.**
- Use STRIDE across trust boundaries that now cross two clouds, a datacentre, a CDN, two clusters, a portal and an AI agent.
- Record every threat as one of three outcomes: a named control with evidence, an accepted risk with a reason, or a gap with an owner.
- Include the supply chain (base images, actions, Terraform providers and modules, Helm charts, model versions) and the control plane itself: CI identities, Argo CD and the platform's GitHub App.

**Defence in depth, audited.**
- Assume one layer fails at a time and ask what still stops the attacker. Go through each layer in turn: identity, network, workload, data, supply chain and detection.
- Know the difference between *rotation* and *revocation*.
- Egress is where exfiltration happens. Detection covers what prevention misses.

**Production readiness as a review.**
- A production readiness review (PRR) asks, per service: SLOs defined and measured, alerts with runbooks, capacity tested, DR tested, security reviewed, cost owned, on-call ready, docs current, and EOL dates known.
- It is a checklist, and you apply it to your own platform honestly. The full checklist lives in [§25](../25-production-readiness-checklist.md).

**Solution architecture as a deliverable.**
- A solution design document (SDD), written to your template in [A2](../appendix/A2-documentation-standards.md), must cover:
  - context and requirements (functional and non-functional)
  - assumptions and constraints
  - options with trade-offs
  - the chosen architecture and its views (context, logical, deployment, network, identity, data flow)
  - NFR mapping (availability, capacity, continuity, security, performance)
  - operational model
  - cost estimate
  - delivery plan and timescale
  - risks
  - the ADRs that follow from it
- This is what *"guide Project Managers ... on solution definition, technical design ... delivered according to agreed costs and timescales"* looks like on paper.

## Architecture decisions in this phase

**ADR-027 — Retrospective: what I would change if I rebuilt Runway**

| | |
|---|---|
| **Problem** | Twenty-two phases of decisions were made with the knowledge you had at the time. Some were wrong, some were right for learning but wrong for production, and some you would keep exactly as they are. |
| **Options** | Leave the ADRs as they are, amend each one, or write one retrospective ADR that reviews them all. |
| **Decision** | Write one retrospective ADR. For **every** ADR from 001 to 026, record keep / change / would-not-do-at-work, with a sentence of evidence each. Add three things you would do differently from day one and three you would defend under pressure. Mark superseded ADRs `Superseded by ADR-027 §n`. |
| **Trade-offs** | It is honest, and it exposes your mistakes. That exposure is the point. Interviewers ask "what would you do differently?", and the senior answer is specific, evidence-based and includes what you would keep. |

The capstone produces its own ADRs, numbered inside the capstone SDD as `CAP-ADR-01…`, so they stay separate from Runway's register.

## Hands-on labs

1. **STRIDE, whole system.** Take the final architecture diagram and, for each trust boundary, list the threats, controls, evidence and accepted risks. Expect 40–60 rows. *Proves: you can reason about security as a system.*
2. **Rotate everything, timed.** Rotate every secret, key, certificate and federated trust in both clouds and on GitHub. Record how long each takes and which ones you forgot. You will forget at least one. *Proves: rotation is a capability, not an intention.*
3. **Egress lockdown.** Restrict egress from both clusters and the VPCs/VNets to an allow-list (security groups and NACLs, NetworkPolicy, and FQDN rules where affordable). Discover every outbound dependency you didn't know you had. *Proves: you know what your system talks to.*
4. **Pod hardening sweep.** Apply the `restricted` Pod Security Standard, read-only root filesystem, dropped capabilities and no service-account token automount where it isn't needed, across every golden-path workload, by changing the platform *once*. *Proves: the platform, not each team, carries security.*
5. **WAF tuning window.** Run AWS WAF on the ALB in count mode against a k6 replay, find the false positives, tune the rules, then switch to block, all inside a timeboxed window. *Proves: WAF is an operational practice.*
6. **Documentation stranger test.** Give three runbooks (DR, incident response and onboarding) to someone else, or to yourself after a week away. Every question they ask marks a defect. *Proves: docs work without you.*

## Project implementation

**Hardening:**
- Egress allow-lists in place.
- The `restricted` PSS is the golden-path default.
- Kyverno enforces signatures and provenance for all images, Helm charts and Argo CD sources.
- GitHub Actions are pinned by SHA.
- Terraform providers have hash lock files, and modules are pinned.
- Rotation schedules are automated where possible and calendared where not.
- WAF is tuned, with the configuration kept as code.
- CloudTrail, Azure Activity Log, Kubernetes audit logs and flightdeck audit logs are centralised, with detections for:
  - console sign-in to prod
  - IAM or role-assignment changes outside the pipeline
  - Kyverno policy exceptions
  - disabled logging

**`docs/security/threat-model.md`** is rewritten for the whole system, with a controls matrix and accepted risks. `docs/security/rotation.md` holds the schedule and measured timings.

**Final architecture:**
- `docs/architecture/runway-final.md` covers the context, container and deployment views (Mermaid), three request paths from memory (user, deployment, incident investigation), the multi-cloud placement rationale and the exclusions table.
- `docs/solution-designs/SDD-000-runway.md` is the platform's own SDD, updated to its final state.

**Operations close-out:**
- A PRR is run against every service, with results in `docs/operations/prr-<service>.md`.
- The EOL register is fully current.
- The final monthly operational review covers the whole 24 months: incidents, MTTR trend, SLO attainment, change failure rate, cost trend and unit cost trend.

**Repository as evidence:**
- The `runway/README.md` numbers table is refreshed with measured values (see [A3](../appendix/A3-portfolio-and-interview.md)).
- There is a "What I operate" section, and a "What I deliberately did not build" section.

**Final game day (the last big exercise):**
- A four-hour, pre-announced but unscripted game day.
- A helper, or `inject-fault.sh --random --count 4`, injects four faults drawn from S01–S36, overlapping in time, across both clouds and the datacentre. At least one is a security scenario and at least one is a cost scenario.
- You run it as incident commander and responder: EMS auto-creates the records, you use flightdeck as an advisor only, you follow the change process for any fix, and you write the timelines live.
- Afterwards: a postmortem, problem records where a cause recurs, and an updated readiness checklist.

**Capstone design exercise (Step 20 of the brief).**

The requirement, handed to you fresh without having seen it before:

> "We need to deploy a secure, highly available application with an AI component that serves thousands of users."

The mentor adds constraints only when you ask the right clarifying questions:
- user geography
- data classification
- the AI feature's purpose and tolerance for errors
- peak concurrency
- RTO and RPO
- budget
- compliance
- team size and skills
- existing enterprise estate (identity provider, network, cloud agreements)

Produce, **on your own and in a timebox of 10 hours**, `docs/solution-designs/SDD-CAP-capstone.md` covering every item below. Each decision gets Problem → Options → Decision → Trade-offs:

| Area | What you must decide and justify |
|---|---|
| Requirements | Clarifying questions asked; NFRs quantified (availability target, p95 latency, peak RPS, RTO/RPO, data residency) |
| Architecture | Context and deployment views; stateless tiers; where state lives |
| Cloud services | Which cloud(s) and why; which managed services vs self-run |
| Network | VPC/VNet design, CIDR, public/private tiers, egress, private endpoints, DNS, CDN/WAF, TLS |
| IAM | Workforce and workload identity, least-privilege model, no static credentials, break-glass |
| Terraform | Module and state layout, environments, pipeline, drift, policy checks |
| Containers | Build, base images, signing, SBOM, registry |
| Kubernetes | Cluster topology, node pools, autoscaling, tenancy, ingress/Gateway, PDBs, upgrades |
| CI/CD | Test pyramid, build-once-promote, GitOps or push, progressive delivery, rollback time target |
| Secrets | Store, delivery, rotation |
| Security controls | Threat model summary, top 10 threats and controls, supply chain, WAF, detection |
| AI component | Managed vs self-hosted, gateway, quotas, caching, fallback, eval gate, data handling, OWASP LLM mapping, cost per request |
| Monitoring, logging, alerting | SLIs/SLOs, alert tiers, dashboards, telemetry pipeline and its cost |
| Scaling | Capacity model (Little's Law), load-test plan, autoscaling policy, quota plan (including AI TPM) |
| Reliability | Failure-mode analysis, timeouts, retries, circuit breakers, degradation modes |
| Backup and DR | Tiering, backup and restore, DR pattern, tested RTO/RPO plan |
| Cost controls | Monthly estimate with the calculator, unit cost, budgets, tagging, anomaly detection |
| Operational procedures | On-call, incident, problem and change process, runbooks list, EOL plan, data catalogue entries |
| Delivery | Phased delivery plan with effort and timescale — the PM-facing page |

Then **defend it** in a 45-minute mock design review with the mentor, who plays EA, security, the PM and the on-call lead. Finish by building a thin slice (one service plus its AI call, on the golden path) to prove that the SDD can actually be built.

## Engineering problems to hit deliberately

| Problem | How to cause it | What you should learn |
|---|---|---|
| Forgotten secret | Rotation lab | Inventory of credentials is itself a control; add it to the data catalogue |
| Egress breakage | Lock down egress | Undeclared dependencies are risk; declare them in `catalog-info.yaml` |
| Hardening breaks workloads | `restricted` PSS + read-only FS | Fix in the platform once, via the golden path — not per team |
| WAF false positive | Block mode without tuning | A 403 that is not the app's fault — diagnose from WAF logs |
| Overlapping incidents | Game day | Prioritise by impact, delegate (even to your future self via notes), communicate |
| Design without questions | Start the capstone without clarifying | Requirements you assumed become risks you own |

## Troubleshooting exercises

1. **S27, "A credential was found in a public place."** Assume a GitHub App private key or a federated-trust misconfiguration has leaked. The response has four parts: contain, rotate, assess what was reachable (CloudTrail, Activity Log, audit logs), and prevent. Rotating alone is failing the exercise. Write it up as Detection → Investigation → Mitigation → Root Cause → Permanent Fix → Prevention.
2. **The final game day.** Four overlapping faults across the whole estate. Deliverables: live timelines, one postmortem covering the day, problem records where causes recur, and change records for every fix.
3. **"Something is calling home."** An egress-deny log shows an unexpected outbound connection from a pod. Find which workload, which dependency and why, and decide whether it is a supply-chain issue or a missing declaration.

## Deliverables

- [ ] `docs/adr/ADR-027-retrospective.md`
- [ ] `docs/security/threat-model.md` (whole system, controls matrix, accepted risks), `docs/security/rotation.md`
- [ ] Hardening in code: egress allow-lists, PSS restricted default, signed-everything admission, pinned actions/providers, tuned WAF, security detections
- [ ] `docs/architecture/runway-final.md`, `docs/solution-designs/SDD-000-runway.md` (final)
- [ ] `docs/operations/prr-*.md` for every service; EOL register current; final operational review
- [ ] Game-day postmortem + incident/problem/change records; S27 incident record
- [ ] `docs/solution-designs/SDD-CAP-capstone.md` + its `CAP-ADR`s + the thin-slice implementation + design review notes
- [ ] `runway/README.md` numbers table refreshed; [§25 checklist](../25-production-readiness-checklist.md) completed honestly
- [ ] Competency matrix ([§24](../24-competency-matrix.md)) final self-assessment with evidence links

## Completion criteria

- Every threat in the model is a control with evidence, an accepted risk with a reason, or a gap with an owner. None is blank.
- Full credential rotation is **measured**, and you can state the time. Nothing was left unrotated in the second run.
- The game day ended with all four faults mitigated, live timelines, and at least one problem record. The postmortem names what you would change about detection.
- The capstone SDD covers every row of the table above. You answered the mentor's review questions without having to go back and redesign, and the thin slice runs on the golden path.
- A person who is not you executed the DR runbook and the onboarding guide without asking a question.
- You can answer all of [A3's interview questions](../appendix/A3-portfolio-and-interview.md) out loud, including "what would you change?" with at least three specifics and one thing you would keep.

## Mentor checkpoints

- **Before hardening:** bring the STRIDE table. I will pick three rows and ask you to show me the evidence live.
- **Capstone:** I give you the requirement without warning. You get one round of clarifying questions, so choose them well. The review is adversarial on purpose. Expect "why not serverless?", "why Kubernetes at all?", "what if the AI provider is down for a day?", "what does this cost at 10× traffic?" and "who gets paged at 3 a.m., and what do they read first?"
- **Game day:** I play the business stakeholder asking for updates every 20 minutes.
- **The failure I will inject after you think you are done:** a fault in the one place you have not tested since you built it. The category is a recovery path that decayed.

## Resources

- **AWS Well-Architected Framework** (all six pillars) and the **Azure Well-Architected Framework** (the five pillars plus the assessment tool). Run both reviews against Runway, and record the high-risk findings.
- ***Threat Modeling: Designing for Security*** (Shostack): the chapters on STRIDE and on tracking threats to resolution. Alternatively, the relevant chapters of Anderson's *Security Engineering*.
- **Google SRE Book:** "Production Readiness Reviews" (the chapter on the evolving SRE engagement model) and "Postmortem Culture".
- **Azure Architecture Center**, "Baseline highly available zone-redundant web application", and **AWS Prescriptive Guidance** / reference architectures. Compare their choices with your capstone SDD *after* you have written it, not before.
- **[§25 Production readiness checklist](../25-production-readiness-checklist.md)** and **[A2 Documentation standards](../appendix/A2-documentation-standards.md)**, both from this roadmap.

## Estimate vs actual

| | Planned | Actual | Variance note |
|---|---|---|---|
| Hours | 30 (+10 for R9) | | |
| Cloud cost this phase | $90–150 (WAF + game-day windows) | | |
