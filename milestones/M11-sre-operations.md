# Milestone M11 — SRE and Production Operations (incident, problem, change, SLOs, EMS)

*Phase 11 · ~4 weeks · Months ~11.7–12.5 · Prerequisites: [M10](M10-observability.md) · Unlocks: [M12](M12-capacity-ha-dr.md), [M15](M15-security-governance.md), [M17](M17-golden-paths.md), [M19](M19-ai-infrastructure.md)*

> **Never cut this milestone.** It is half of what the JD describes, and the half almost no self-taught candidate can show evidence for.

---

## Why this phase exists

Read the JD's operational lines together:

- *"Ensure that assigned incidents are resolved within agreed service levels and problems are defined and analysed in line with the Problem Management Process."*
- *"Ensure that solutions to problems meet architectural and security standards … and that the impact on the business is minimised."*
- *"Own and manage changes within the technology area."*
- *"Collaborate with other IT Technology teams to implement ongoing monitoring … to ensure Level 1 and Level 2 incidents are configured within EMS."*
- *"Ensure that the developed architecture/solution meets service-level requirements in terms of expected availability, capacity, continuity, security, performance, and reliability."*

That covers five ITIL practices (incident, problem, change, service level and event management), plus SRE's measurement discipline. It sits here because you now have telemetry (M10) to detect with, and a real system (M09) to break. It sits before capacity and DR (M12), because those drills need the incident machinery to already exist.

## Concepts to learn

**Service levels**
- **SLI** (a measured ratio of good events to all events), **SLO** (a target over a window), **SLA** (a contract with consequences; usually looser than the SLO). Measurement windows: rolling 28/30 days versus calendar.
- **Error budget** = 1 − SLO. The **error budget policy**: what happens when it is exhausted (freeze risky changes, prioritise reliability work). The budget is a decision tool, not a report.
- **Multi-window, multi-burn-rate alerts** (for example 14.4× over 1 h and 5 m for paging, 6× over 6 h and 30 m, 1× over 3 d for tickets). Why this beats static thresholds in both precision and recall.
- Composite availability: serial dependencies multiply downwards. Why your SLO cannot exceed the product of your hard dependencies' SLAs unless you design for it.

**Event management ("EMS")**
- Event → alert → incident. Correlation and deduplication. Tiers: **L1** is an event the service desk can act on with a runbook (restart a stuck job, scale a known queue, follow a known workaround). **L2** needs engineering diagnosis. **EMS is the employer's term. You are modelling the concept**: routing, enrichment, auto-ticketing and runbook linkage. Knowing *which* alerts are safe to hand to L1 is a senior judgement.

**Incident management**
- Mitigate first, understand later. Severity (Sev1–Sev4) set by impact, not by cause. Roles even when solo: incident commander, operations, communications and scribe. Status updates on a cadence. Handover. Workarounds as a legitimate outcome inside the SLA ("where applicable, workarounds are provided to minimise business impact").
- The loop you record, in this order and written **during** the incident: **Detection → Investigation → Mitigation → Root Cause → Permanent Fix → Prevention**, plus a timestamped timeline and rejected hypotheses.

**Problem management**
- An incident is an event. A problem is the underlying condition. Known error records and workarounds. Trend analysis across incidents. Causal chains, contributing factors, and why "human error" is never a root cause. 5 Whys and its limits. The discipline of not closing the problem when the incident closes.

**Change management / enablement**
- Change types: **standard** (pre-approved, low risk, repeatable: a golden-path deploy), **normal** (risk-assessed and reviewed), **emergency** (expedited, reviewed afterwards). Risk scoring from blast radius, reversibility, time of day, novelty and test evidence. Change calendar and freeze windows. **Post-implementation review (PIR)** for failed changes.
- The engineering answer to heavyweight CAB processes: **small, frequent, reversible changes with automated evidence** (plan output, test results, SLO state, rollback proof attached to the change record). DORA metrics: deployment frequency, lead time, **change failure rate**, time to restore.

**SRE practice**
- Toil: its definition and how to budget it. On-call hygiene. Blameless postmortems. Game days. The **monthly operational review** as the forum where all of the above meets the business.

## Architecture decisions in this phase

**ADR-013 SLOs, alert tiers (L1/L2), event management and the change process.**

| | |
|---|---|
| **Problem** | Alerts need to route to the right responder with the right urgency, incidents need records without manual toil, and every production change needs evidence and a reversible path, without a CAB meeting for each deploy |
| **Options** | (a) Page on every alert, and use a manual ticket and change spreadsheet. (b) A commercial ITSM (ServiceNow-like), which is out of budget and hides the mechanics. (c) **Alertmanager routing on `severity` × `tier` labels, an `ems-bridge` webhook that opens GitHub issues as incident records, burn-rate SLO alerts, and change records generated from PR + deploy metadata with risk scoring** |
| **Decision** | (c). Standard changes = golden-path deploys that passed all gates. Normal changes need a change record with risk score, rollback plan and approval (GitHub environment protection). Emergency changes are allowed with a mandatory PIR within 3 working days |
| **Trade-offs** | You build and maintain a small bridge service, and GitHub issues are not a real ITSM (no SLA clocks by default, so you add them via labels and a scheduled checker). In return, every mechanic is visible and portable to whatever EMS/ITSM an employer uses |

## Hands-on labs

1. **Write the SLOs before the drills.** `cargo-api` availability 99.5% over 28 days (non-5xx at the ALB), latency 95% of `GET /shipments/*` < 300 ms, and eta-worker freshness 99% of jobs < 2 min. Justify each target from M10 data, not from ambition.
2. **Burn-rate maths.** For 99.5% over 28 days, compute the burn rate that exhausts 2% of the budget in 1 hour. Implement the multi-window rules. Inject 3% errors and check which alert fires and when. Inject 0.6% errors and check that the slow-burn ticket fires and the pager does not.
3. **Severity matrix and escalation.** Write them before the first drill (reuse and adapt the Sev1–Sev4 table in [§10](../10-failure-and-troubleshooting.md)). Include L1/L2 routing and response targets per tier.
4. **Classify your alerts L1/L2.** For every alert from M10, decide whether a non-engineer can resolve it with a runbook. If yes, write the L1 runbook to that standard (numbered steps, expected output, and an escalation condition). If you cannot write it, it is L2.
5. **ems-bridge.** A tiny service (Python) that receives Alertmanager webhooks, dedupes by fingerprint, opens or updates a GitHub issue (`incident`, `sev-N`, `tier-LN`, `service/x` labels) with the runbook link, graph links and a timeline stub, and closes it on resolve with the duration recorded.
6. **Change record automation.** Extend the `change-record` reusable workflow. On a prod deploy it writes `docs/changes/CHG-NNN.md` (or an issue) with the PR, digest, plan diff, test evidence, SLO state before, risk score, rollback command and approver. It blocks the deploy if the error budget for the service is exhausted and the change is not an emergency.
7. **Game day.** `platform/scripts/inject-fault.sh --random` picks one of eight faults while you are away from the keyboard. Come back and run the incident from the page you received.
8. **Problem from incidents.** Take three of your incident records that share a cause and write one PRB that explains them.

## Project implementation

- `platform/observability/slo/`: SLO definitions as code (Sloth/Pyrra-style or hand-written recording rules), with burn-rate alerts replacing M10's simple thresholds for user-facing services.
- Alertmanager config as code: routes by `severity` and `tier`. L1 goes to the "service desk" channel with the runbook inline. L2 goes to the engineering pager (ntfy/Slack free tier or email). Inhibition rules (node down inhibits pod alerts on that node). Silences only with an expiry and a reason.
- `workloads/ems-bridge/` deployed through the `runway-service` chart, with its own SLO (it is production tooling).
- `docs/operations/severity-matrix.md`, `docs/operations/escalation.md`, `docs/operations/error-budget-policy.md`, `docs/operations/change-policy.md` (standard/normal/emergency, risk scoring, freeze windows, PIR).
- Templates in `docs/incidents/`, `docs/problems/`, `docs/changes/` (from [A2](../appendix/A2-documentation-standards.md)).
- `platform/scripts/inject-fault.sh`: at least ten fault modes covering the scenarios below.
- `docs/operations/review-YYYY-MM.md`: the first **monthly operational review**, covering incidents by severity, MTTA/MTTR, SLO attainment and budget consumed, open problems and their age, changes deployed and change failure rate, top toil and what you automated, cost versus budget, and detection gaps.

## Engineering problems to hit deliberately

| Problem | How to cause it | What you should learn |
|---|---|---|
| Alert storm | Node failure fires 40 pod alerts | Inhibition and grouping. Page on symptoms once |
| Pager at 3 a.m. for nothing | Fast-burn threshold too sensitive | Precision matters as much as recall |
| Incident with no record | Fix it "quickly" without the bridge | Why automated records exist |
| Change without rollback | Deploy a migration with no down path | Reversibility is part of risk scoring |
| Budget exhausted, deploy blocked | Real burn + a feature PR | The policy must say what *is* allowed (fixes, emergency) |

## Troubleshooting exercises

Each one must produce a full record (Detection → Investigation → Mitigation → Root Cause → Permanent Fix → Prevention) created by `ems-bridge`, with the timeline filled in live.

- **S18, "Intermittent 500s under load".** The SQLAlchemy pool is 5 and RDS `max_connections` is small. k6 at moderate load. The trap is scaling pods, which *adds* connections and makes it worse. Permanent fix direction: pool sizing maths, and RDS Proxy or PgBouncer as an option (ADR note).
- **S20, "Whole system degrades after one dependency blips".** carrier-sim fails for 30 s. Naive retries in cargo-api and the worker amplify the load 9×, and recovery takes minutes after carrier-sim is healthy again. Permanent fix: timeouts, retries with jittered backoff and a retry budget, and a circuit breaker (*Release It!* stability patterns).
- **S35, "Release caused an outage; rollback needed at 2 a.m."** A normal change passes the gates and then breaks a rarely used route. Declare, roll back (measured), and raise an **emergency change** for the hot-fix. Write the **PIR**: why the gates did not catch it, and what gate you add.
- **S16 revisited** as a game-day surprise, now with burn-rate detection. Compare detection time with M10.
- **Problem record:** combine S16, S20 and one of your M09 incidents into **PRB-001 "Dependency calls have no deadline"**. The permanent fix, a standard resilient HTTP client with timeouts, retry budget and breaker, is earmarked for the golden path in M17. **This is where operations feeds platform engineering.**

## Deliverables

- [ ] SLOs as code with burn-rate alerts for `cargo-api` and `eta-worker`
- [ ] Alertmanager routing (L1/L2 × severity), inhibition, silence policy
- [ ] `workloads/ems-bridge/` with its own SLO and runbook
- [ ] Automated change records + error-budget gate in the deploy workflow
- [ ] Severity matrix, escalation, error budget policy, change policy
- [ ] At least **eight incident records**, **two problem records** (PRB-001 plus one more), **one PIR**
- [ ] `inject-fault.sh` with ten or more modes
- [ ] First monthly operational review
- [ ] ADR-013

## Completion criteria

- In a blind game day, the incident record is opened **automatically** within 2 minutes of detection, and you reach mitigation for a Sev2 within your own matrix's target.
- For each SLO, you show a drill where the fast-burn alert fired and one where only the slow-burn ticket fired, and explain why that is correct.
- Every alert is classified L1 or L2. **An L1 runbook is executed successfully by someone else** (or by you a month later, strictly literally) without questions.
- Change failure rate, deployment frequency and MTTR are computed from your own records for the last month.
- You explain the difference between an incident and a problem using your own PRB-001, and defend its causal chain under questioning.
- One real deploy was blocked by the error-budget gate, and you can show the record.

## Mentor checkpoints

- **Before you build:** SLO targets with the data behind them, and your L1/L2 classification. Expect me to push alerts from L1 to L2 where the runbook asks for judgement.
- **During:** the S18 record. I will read the rejected-hypotheses table first.
- **Questions I will ask:** Why mitigate before you diagnose? What does your error budget policy let you ship when the budget is gone? What makes a change "standard"? What is the difference between a PIR and a postmortem? What do you hand to L1, and why?
- **The failure I will inject:** two overlapping incidents, one of which is caused by your own mitigation of the other.

## Resources

1. **Google *The Site Reliability Workbook*** (free at sre.google): ch. 2 *Implementing SLOs*, ch. 5 *Alerting on SLOs* (the burn-rate tables), ch. 9 *Incident Response*, ch. 10 *Postmortem Culture*.
2. **Google *Site Reliability Engineering*:** ch. 3 *Embracing Risk*, ch. 4 *Service Level Objectives*, ch. 5 *Eliminating Toil*, ch. 14 *Managing Incidents*.
3. ***Release It!*, 2nd ed. (Nygard):** Part I *Stability antipatterns* and *Stability patterns* (timeouts, circuit breaker, bulkheads).
4. **ITIL 4 practice guides (overview level):** *Incident management*, *Problem management*, *Change enablement*, *Service level management*, *Monitoring and event management*. Or the ITIL 4 Foundation syllabus if you take the optional exam.
5. **DORA** (dora.dev): the four key metrics definitions.

## Estimate vs actual

| | Hours | Cost |
|---|---|---|
| Planned | 40 | ~$40–70 (includes one 48–72 h soak window) |
| Actual | | |
| Variance note | | |
