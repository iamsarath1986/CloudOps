# Section 16 — Observability & SRE Learning Path

*The spine of "running it": from structured logs on a laptop to SLO-driven operations across two clouds, with incident, problem and change management done to a standard an enterprise service desk would recognise. This is the half of the job most self-taught candidates cannot evidence.*

---

## 1. What the JD actually asks for

> *"Experience with: Monitoring · Logging · Observability · Troubleshooting across application and cloud infrastructure layers"*
> *"Ensure that the developed architecture/solution meets service-level requirements in terms of expected availability, capacity, continuity, security, performance, and reliability."*
> *"Collaborate with other IT Technology teams to implement ongoing monitoring of the relevant technology to ensure Level 1 and Level 2 incidents are configured within EMS."*
> *"Ensure that assigned incidents are resolved within agreed service levels and problems are defined and analysed in line with the Problem Management Process... Own and manage changes within the technology area."*

Three things to notice:

1. **"Across application and cloud infrastructure layers."** Not "can read a dashboard". You must follow a symptom from a browser through CDN, load balancer, gateway, pod, database and cloud control plane — on two clouds.
2. **"L1 and L2 incidents configured within EMS."** Monitoring here is an *integration* job: alerts must land in an enterprise Event Management System, classified so that a service desk (L1) can act on some and engineering (L2) gets the rest. You will model this in Runway with alert tiers and an `ems-bridge` (EMS is the employer's term; you are modelling the concept).
3. **Six service-management processes** — incident, problem, change, service level, capacity, availability/continuity — are written into the role. Senior means you *design* these for your technology area, not just follow them.

**Depth expected:** Production Proficiency on monitoring/logging/troubleshooting; Advanced on SLO design, alert design and incident/problem practice.

## 2. The progression

| Stage | Theme | Milestones | You leave able to |
|:-:|---|---|---|
| 1 | Signals on one host | [M00](milestones/M00-foundations.md), [M01](milestones/M01-linux-networking.md) | Emit structured JSON logs with request IDs; debug from `journalctl`, `ss`, `curl -v`, `dig` |
| 2 | Cloud-native signals | [M04](milestones/M04-aws-core.md), [M05](milestones/M05-terraform.md) | Read CloudWatch metrics/logs, ALB access logs, VPC flow logs; create alarms as code |
| 3 | Container & cluster signals | [M06](milestones/M06-containers.md), [M08](milestones/M08-kubernetes-fundamentals.md), [M09](milestones/M09-eks.md) | Read exit codes, events, kubelet and control-plane logs; diagnose the six pod failure states |
| 4 | The observability platform | [M10](milestones/M10-observability.md) | Run OTel + AMP + Loki + Tempo + Grafana as code; answer questions you did not anticipate |
| 5 | SRE and service management | [M11](milestones/M11-sre-operations.md) | Run SLOs, error budgets, L1/L2 alert routing, incidents, problems, changes, monthly reviews |
| 6 | Capacity, availability, continuity | [M12](milestones/M12-capacity-ha-dr.md) | Find the saturating resource; survive node/AZ loss; restore data and rebuild an environment with measured RTO/RPO |
| 7 | Second cloud, one pane | [M13](milestones/M13-azure-core.md), [M14](milestones/M14-aks-multicluster.md) | Bring AKS and Azure Monitor into the same SLOs and dashboards; trace a cross-cloud call |
| 8 | Standardised telemetry | [M17](milestones/M17-golden-paths.md) | Make dashboards, alerts and SLOs a side effect of onboarding |
| 9 | AI and agent observability | [M19](milestones/M19-ai-infrastructure.md), [M20](milestones/M20-ai-agents-mcp.md) | Monitor tokens, cost, TTFT, fallback rate; measure time-to-first-hypothesis with flightdeck |

## 3. Concept depth, stage by stage

### Signals (Stages 1–4)
- **Logs** for detail, **metrics** for aggregate trend and alerting, **traces** for causality across hops. Know what each is bad at: logs are expensive at volume, metrics lose individual requests, traces are sampled.
- **OpenTelemetry**: SDK → Collector → backend. Spans, attributes, resource attributes (`service.name`, `deployment.environment`, `cloud.provider`), context propagation (W3C `traceparent`), and propagation *through SQS* — the classic place traces fragment. The Collector as the point where you redact, sample, enrich and route. Choosing the standard protects your instrumentation when the backend changes (ADR-012).
- **RED** (rate, errors, duration) for request-driven services; **USE** (utilisation, saturation, errors) for resources. Two checklists that cover most dashboards.
- **Cardinality**: a label per shipment ID creates a time series per shipment and a memorable AMP bill. Rule: labels are bounded enumerations.
- **Percentiles not averages**; histograms and bucket choice; why p99 is a real customer; why you cannot average percentiles across pods.
- **Native vs portable**: CloudWatch and Azure Monitor see the cloud control plane and managed services; your OTel stack sees your code. You need both, and you need to know which one survives when the other is down.

### SLOs and alerting (Stage 5)
- **SLI** = a ratio of good events to valid events, measured where the user feels it (ALB/gateway, not the pod). **SLO** = target over a window. **SLA** = a contract with consequences, always looser than the SLO. **Error budget** = 1 − SLO, spent by incidents *and* by risky changes.
- **Multi-window, multi-burn-rate alerts** (e.g. 14.4× over 1 h and 5 m; 6× over 6 h and 30 m) — page on budget burn, not on thresholds.
- **Alert on symptoms, not causes.** "Users are getting errors" pages; "CPU is 80%" goes to a dashboard, if anywhere. Cause-based alerts produce noise, noise produces muting, muting produces outages.
- **Absence-of-data alerts** — silence is the most dangerous signal (S17).
- **Every alert has a runbook link, an owner, a tier and a severity.** No exceptions — enforced by a CI check on the alert rules.

### Event management and L1/L2 (Stage 5)

| Alert tier | Who acts | Requirement for the alert to be L1 | Example in Runway |
|---|---|---|---|
| **L1** | Service desk / operations bridge | A deterministic runbook step a non-engineer can execute and verify | Certificate expires in 14 days → raise renewal ticket; disk 85% on the datacentre VM → run cleanup job |
| **L2** | Platform/application engineering | Diagnosis requires judgement | SLO fast burn; dependency latency; failed deployment rollback |

Alertmanager routes on `tier` and `severity` labels; the `ems-bridge` webhook opens an incident record (GitHub issue labelled `incident`, severity pre-filled, runbook linked) — so the incident *record* starts at detection time, not when you remember to write it. Senior skill: **moving alerts from L2 to L1** by making their runbooks deterministic. That is toil reduction with an organisational payoff.

### Service management processes (Stages 5–6)

| Process | Engineering meaning | Runway artefact |
|---|---|---|
| Incident | Restore service first, understand later; severity; comms cadence; workarounds are legitimate outcomes inside the SLA | `docs/incidents/INC-NNN.md`, written *during* |
| Problem | The underlying condition behind one or more incidents; known-error records; not closed when the incident closes | `docs/problems/PRB-NNN.md` |
| Change | Standard/normal/emergency; risk; rollback plan; PIR on failure; change failure rate | `docs/changes/CHG-NNN.md` (see [Section 15 §5](15-cicd-learning-path.md#5-change-management-the-engineering-version)) |
| Service level | SLIs/SLOs per service, reviewed monthly | `platform/observability/slos/` + monthly review |
| Capacity | Measured ceiling, saturating resource, headroom policy, quota forecast | `docs/capacity/capacity-plan.md` |
| Availability | Composite availability maths, SPOF register, MTTR as the cheaper lever | `docs/architecture/availability.md` |
| Continuity | RTO/RPO tiers, tested restores, rebuild runbooks | `docs/continuity/` |

The incident write-up format used throughout the roadmap: **Detection → Investigation → Mitigation → Root Cause → Permanent Fix → Prevention**, with a live timeline and a rejected-hypotheses table. Templates: [Appendix A2](appendix/A2-documentation-standards.md). Scenario catalogue: [Section 10](10-failure-and-troubleshooting.md).

### Capacity, availability, continuity (Stage 6)
- Open vs closed load models; **Little's Law** (`L = λW`); the knee of the latency curve; bottleneck shifting; connection pools as the usual first saturation point (S18).
- CPU throttling as a latency problem with no CPU alarm (S19).
- Availability maths: serial dependencies multiply downwards; redundancy only helps if failures are independent; AZ vs region; PDBs and topology spread (S21).
- RTO/RPO as business decisions; **replication is not a backup**; an untested backup is a belief; the rebuild is a runbook, timed twice (S22, S23; ADR-014).

### Toil
Manual, repetitive, automatable work that scales with the service. Track the top three sources in each monthly review and automate one. The `ems-bridge`, `backup-verify` Lambda and the `idle-detector` all exist because a manual task showed up in a review first.

## 4. The monthly operational review

One page in `docs/operations/review-YYYY-MM.md`, from [M11](milestones/M11-sre-operations.md) onward:

Incidents by severity and tier · MTTA/MTTR · SLO attainment and error budget consumed per service · open problems and their age · changes deployed, change failure rate and failed-change PIRs · top three toil sources and what was automated · capacity headroom and quota status · cost versus budget and anomalies · EOL register items due in the next 90 days · what you would change about detection.

Twelve of these is stronger evidence for this JD than any architecture diagram.

## 5. Checkpoints

| By | You can |
|---|---|
| M01 | Explain connection refused vs timeout vs reset vs hang, and what each says about where the packet died |
| M09 | Diagnose any of the six pod failure states in under five minutes from `kubectl` alone |
| M10 | Delete a dashboard and rebuild it from code in under five minutes; find a latency regression via traces → metrics → logs, in that order |
| M11 | Show ten incident records, four problem records, a failed-change PIR, and your L1/L2 alert inventory |
| M12 | State your measured request ceiling and its limiting resource; your measured RTO/RPO from two drills |
| M14 | Trace one request from the EKS `insights-worker` to the AKS `ai-gateway` and back in one waterfall |
| M20 | Show time-to-first-hypothesis with and without flightdeck across replayed incidents |

## 6. Resources

1. ***The Site Reliability Workbook*** (free at sre.google) — ch. 2 "Implementing SLOs", ch. 5 "Alerting on SLOs" (the burn-rate maths), ch. 9 "Incident Response". Then *Site Reliability Engineering* ch. 6 "Monitoring Distributed Systems" and ch. 15 "Postmortem Culture".
2. ***Observability Engineering*** (Majors, Fong-Jones, Miranda) — Part I (ch. 1–6) for the "unknown unknowns" framing; ch. 12 on SLO-based alerting.
3. **OpenTelemetry docs** — "Concepts" (signals, context propagation) and "Collector → Configuration"; **Prometheus docs** — "Practices: Naming", "Instrumentation" and "Alerting".
4. ***Release It!*** 2nd ed (Nygard) — ch. 4 "Stability Antipatterns" and ch. 5 "Stability Patterns". Read before M11/M12: timeouts, circuit breakers, bulkheads, and why retry storms happen.
5. **AWS Well-Architected — Reliability Pillar** (sections on workload architecture, change management and failure management) for the cloud-specific view of the same ideas.

→ Next: [Section 17 — Security & Governance Learning Path](17-security-governance-learning-path.md)
