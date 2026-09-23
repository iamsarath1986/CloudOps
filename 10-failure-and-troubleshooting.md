# Section 10 — Failure and Troubleshooting Exercises: the Operations Playbook

The JD asks the Senior Technical Engineer to *"ensure that assigned incidents are resolved within agreed service levels and problems are defined and analysed in line with the Problem Management Process"*, to *"own and manage changes within the technology area"*, to make sure *"Level 1 and Level 2 incidents are configured within EMS"*, and to meet *"service-level requirements in terms of expected availability, capacity, continuity, security, performance, and reliability."*

This section is where you get those skills. It gives you the incident loop, the severity and tier model, the templates, 36 failure scenarios, five fully worked examples, and the problem, change and review practices around them. **Most candidates can't replicate this part of the roadmap.** It needs a system real enough to break, and the habit of writing things down while you're under pressure.

Scenarios start in the milestone listed against each one. The full operational discipline (alert-detected incidents, severity, records created automatically through `ems-bridge`) starts at [M11](milestones/M11-sre-operations.md). Before M10 you have no real observability, and an incident against a system you can't see is just confusion. Before M11, work scenarios as guided troubleshooting exercises. From M11 on, every scenario is a real incident.

---

## 1. The loop

Every major scenario is worked through these six stages, then documented. **The record is written during the incident, not reconstructed afterwards.**

| Stage | The question | The discipline |
|---|---|---|
| **Detection** | How did I find out, and when? | If the answer is "I caused it so I knew", monitoring failed. Record that as a finding. Note the detection time, the source (alert name, tier, severity), and the gap between start and detection. |
| **Investigation** | What is the blast radius? | Scope before cause: which users, endpoints, clusters, clouds and environments, and since when. Check recent changes first, because the change log is the highest-yield question in operations. Resist the first plausible explanation. |
| **Mitigation** | How do I restore service *now*? | **Mitigate before you understand.** Roll back, shift traffic, scale out, fail over, switch off the feature, or fall back to a degraded path. A workaround within the SLA is a successful outcome; the JD's "impact on the business is minimised" means exactly this. |
| **Root Cause** | Why did it happen? | Form a hypothesis, say what evidence would disprove it, then go and get that evidence. Record the hypotheses you rejected. Separate the *trigger* (what happened today) from the *cause* (the condition that allowed it) and *contributing factors*. "Human error" is never a root cause. |
| **Permanent Fix** | What stops this cause recurring? | Usually a separate, planned change days later, through the normal pipeline and change process. It often becomes a problem record rather than part of the incident. |
| **Prevention** | What stops this *class* of failure, and what detects it sooner? | Detection improvements, guardrails in the golden path, policy, tests. **This is where operations feeds platform engineering.** |
| *(Document)* | What would the next person need? | Timeline with timestamps, impact, all six stages, follow-up actions with owners and due dates, and the link to any problem or change record. |

**Two rules that make it real:**

1. The moment you detect something, start a timer and a text file. `ems-bridge` does this for you from M11: the alert opens the incident issue.
2. Never edit the timeline to make yourself look better. The wrong turns are the evidence of your judgement.

## 2. Severity matrix

Write this before the first drill in [M11](milestones/M11-sre-operations.md). You can't classify under pressure without a scale agreed in advance. It is recorded in ADR-013.

| Sev | Definition | Examples in Runway | Respond | Mitigate within | Update cadence |
|:--:|---|---|---|---|---|
| **Sev1** | Critical business function unavailable, data loss or exposure, or security breach | `cargo-api` down in prod; RDS data deleted; leaked credential (S27) | Immediately | 1 h | Every 30 min |
| **Sev2** | Major degradation, or an SLO burning fast (fast-burn alert) | p95 six times normal (S16); AI summaries failing without fallback; one cloud's cluster unreachable | 15 min | 4 h | Hourly |
| **Sev3** | Minor function affected, a workaround exists, or slow burn | AI running on the degraded fallback (S31 with fallback working); drift detected (S06) | Next working block | 2 working days | On resolution |
| **Sev4** | No current user impact; a risk or hygiene issue | Certificate expiring in 21 days; cost trending over budget (S29 early) | Backlog | Next planning cycle | None |

Severity is about **impact**, not cause. You may change it during the incident. Record each change and the reason in the timeline.

## 3. Alert tiers and event management (L1/L2)

The JD says monitoring must be set up so *"Level 1 and Level 2 incidents are configured within EMS."* EMS is the employer's term for its Event Management System. Runway models the concept rather than any particular product:

| Tier | Who acts | What qualifies | Required on the alert |
|---|---|---|---|
| **L1** | Service desk / first-line operations, following a runbook | Symptom is clear, the action is safe and documented, and no engineering judgement is needed (restart a failed CronJob, confirm a known degraded mode, acknowledge and escalate) | `tier: L1`, `severity`, `runbook_url`, `service`, the **exact runbook step** to execute, and the escalation condition |
| **L2** | Platform / engineering on-call | Needs diagnosis, a change, or judgement: SLO fast burn, failed deploy, data risk, security events | `tier: L2`, `severity`, `runbook_url`, `service`, the dashboard link, and the likely owners |

The rules, enforced in CI from [M11](milestones/M11-sre-operations.md):

- No alert without `tier`, `severity`, `service` and `runbook_url`. A lint step in `platform/observability/` fails the PR otherwise.
- An L1 alert whose runbook needs a judgement call is really an L2 alert. Fix the classification.
- **`ems-bridge`** receives Alertmanager webhooks and opens (or updates) a GitHub issue labelled `incident`, `sev-N` and `tier-LN` with the alert payload and runbook link, and deduplicates on alert fingerprint. That's your event-management layer: every alert becomes a tracked record automatically, and nothing depends on you remembering.
- Every alert gets reviewed in the monthly operational review: did it fire, was it useful, was the tier right?

## 4. Templates

These live in [Appendix A2](appendix/A2-documentation-standards.md). In short:

**`docs/incidents/INC-NNN-symptom.md`**
```
# INC-NNN — <one-line symptom, not cause>
Severity: SevN (changes: …) · Tier: LN · Scenario: SNN · Cloud(s): AWS/Azure
Started: <ts> · Detected: <ts> by <alert/source> · Mitigated: <ts> · Resolved: <ts>
Impact: <who, what, how long, how measured — SLO/error budget consumed>

## Timeline            HH:MM  observed / did / concluded
## Detection           how, how late, what should have caught it sooner
## Investigation       blast radius, recent changes checked
## Hypotheses          | # | Hypothesis | Evidence for | Evidence against | Verdict |
## Mitigation          what restored service, and when
## Root cause          trigger · cause · contributing factors
## Permanent fix       change record CHG-NNN / problem record PRB-NNN
## Prevention          detection + guardrail + golden-path changes
## Follow-ups          | Action | Owner | Due | Status |
```

**`docs/problems/PRB-NNN-condition.md`**: related incidents · causal chain · contributing factors · why it recurred · known error and workaround · permanent fix and its cost · verification that it stopped recurring.

**`docs/changes/CHG-NNN-summary.md`**: generated by the `change-record` reusable workflow from the PR and deployment ([M11](milestones/M11-sre-operations.md)). It holds the type (standard / normal / emergency), risk score, blast radius, rollback plan, test evidence, approver, window, outcome, and a link to the PIR if the change failed.

**`docs/operations/PIR-NNN.md`** (post-implementation review): what the change was meant to do, what happened, why the pre-change checks didn't catch it, and what changes in the change process.

---

## 5. The scenario catalogue (S01–S36)

**How to use it:** inject the fault yourself, from [M11](milestones/M11-sre-operations.md) using `platform/scripts/inject-fault.sh <scenario>` in random mode so you aren't watching when it fires. Then work the loop. **Read "The trap" column only after your first attempt.** It's the wrong turn most people take.

ITIL processes: **Inc** incident · **Prb** problem · **Chg** change · **SLM** service level · **Cap** capacity · **Avl** availability · **Cont** continuity · **Sec** security event · **Cost** financial management.

| ID | Symptom (as seen) | How to inject | Should be detected by | The trap | Processes | Milestone |
|:--:|---|---|---|---|---|:--:|
| S01 | Service unreachable by name | Delete or alter the Route 53 record; break `/etc/resolv.conf` or CoreDNS config | Synthetic check on the FQDN; `NXDOMAIN` in client logs | Restarting the app. The service is fine; `dig` says so in one command | Inc, Avl | [M01](milestones/M01-linux-networking.md) |
| S02 | Requests hang, then time out | Remove the SG ingress rule to RDS; add a NACL deny | Dependency error-rate and latency alert | Assuming the database is down. *Timeout* means the packet was dropped; *refused* would mean it arrived | Inc | [M01](milestones/M01-linux-networking.md)/[M04](milestones/M04-aws-core.md) |
| S03 | Clients reject HTTPS | Issue a 1-day certificate; serve the wrong SAN; drop the intermediate | Certificate-expiry alert at 30/14/7 days (if missing, that's the finding) | Renewing and moving on without asking why nothing warned you 30 days out | Inc, Avl, Prb | [M01](milestones/M01-linux-networking.md) |
| S04 | Deploy fails with AccessDenied | Remove an action from the deploy role; tighten the OIDC `sub` condition | Pipeline failure notification | Reading it as a network problem, or granting `*` to "unblock". Read the denied action and the evaluation chain | Inc, Sec, Chg | [M04](milestones/M04-aws-core.md) |
| S05 | Terraform can't acquire lock / state corrupted | Kill an apply mid-run; hand-edit a state copy | Pipeline failure; `plan` error | `force-unlock` without confirming nobody else is applying; editing state by hand without a backup | Inc, Chg | [M05](milestones/M05-terraform.md) |
| S06 | Nightly plan shows unexpected changes | Change an SG or tag in the console | `drift-detect` workflow opens an issue | Blindly applying to reconcile. First ask *who changed it and why*; it may have been a real fix | Prb, Chg | [M05](milestones/M05-terraform.md) |
| S07 | Plan wants to replace the database | Change a forces-new attribute (engine, identifier, subnet group) | PR review, and a conftest rule on `delete` actions against protected types | Approving a 40-line plan without reading the `-/+` | Chg, Cont | [M05](milestones/M05-terraform.md) |
| S08 | New image crashes on start in prod only | Non-root user plus a read-only root filesystem where the app writes to `/app/tmp` | Rollout failure; CrashLoopBackOff alert | "Works on my machine" because local runs as root | Inc, Chg | [M06](milestones/M06-containers.md) |
| S09 | Containers restart, exit code 137 | Memory limit below working set | Restart-rate alert | Reading only `describe` and missing `Last State: OOMKilled` and the previous container's logs | Inc, Cap | [M06](milestones/M06-containers.md) |
| S10 | Pipeline green, users get 5xx | Deploy with readiness pointing at a dead dependency; smoke test that only checks `200` on `/` | Availability SLO burn, not the pipeline | Trusting the green tick. Fix the pipeline so it can't lie again | Inc, Chg | [M07](milestones/M07-cicd.md) |
| S11 | Errors spike just after a release | Drop or rename a column the old version still reads | Error-rate alert correlated with deploy marker | Rolling back the app while the schema stays changed. Roll back is not always possible; expand-contract is | Inc, Chg, Prb | [M07](milestones/M07-cicd.md) |
| S12 | Pods never become Ready | Bad tag / crash on boot / missing ConfigMap key / failing probe | Deployment availability alert | Stopping at `get pods`. `describe`, events, `logs --previous` | Inc | [M08](milestones/M08-kubernetes-fundamentals.md) |
| S13 | Pods stuck Pending | Oversized requests; a taint without toleration; NodePool limits; a tiny subnet causing VPC CNI IP exhaustion | Pending-pods alert; Karpenter events | Adding nodes by hand. Read the scheduler's event: it says exactly why | Inc, Cap | [M09](milestones/M09-eks.md) |
| S14 | Pod gets AccessDenied calling AWS | Delete the Pod Identity association; wrong service account name | App error logs; dependency alert | Putting access keys in a Secret "temporarily" | Inc, Sec | [M09](milestones/M09-eks.md) |
| S15 | Private workloads can't reach the internet or AWS APIs | Remove the NAT route; delete the STS/ECR VPC endpoint | ImagePullBackOff surge; SDK timeouts | Debugging the app. Timeouts on *every* outbound call point at routing | Inc, Avl | [M09](milestones/M09-eks.md) |
| S16 | p95 latency up 6×, errors flat | Add 400 ms to `carrier-sim` | p95 latency SLO burn-rate alert | Blaming the database because it's the usual suspect. The trace says otherwise | Inc, SLM | [M10](milestones/M10-observability.md) |
| S17 | Dashboards go quiet | Scale the OTel Collector to 0; exceed an ingest limit | Absence-of-data alert (which you must have built) | Reading silence as "healthy". Missing telemetry is the most dangerous signal | Inc, Prb | [M10](milestones/M10-observability.md) |
| S18 | Intermittent 500s under load | Pool size 5, RDS `max_connections` low, HPA scales pods up | Error-rate alert under load; pool-wait metric | Scaling pods out, which multiplies connections and makes it worse | Inc, Cap, Prb | [M11](milestones/M11-sre-operations.md) |
| S19 | Latency high, CPU looks fine | CPU limit 100m on a busy service | Latency SLO; throttling metric (if you chart it) | Averaged CPU usage looks low. Throttling is per 100 ms period | Cap, Inc | [M12](milestones/M12-capacity-ha-dr.md) |
| S20 | Everything degrades after one dependency blips | `carrier-sim` fails for 60 s while clients retry immediately with no budget | Multiple services' SLOs burn together | Treating the victim services as causes. Look for the retry amplification | Inc, Avl, Prb | [M11](milestones/M11-sre-operations.md) |
| S21 | Brief outage when a node disappears | Terminate a node; simulate a spot interruption | Availability dip; node-not-ready | Discovering there was no PDB or topology spread only when it matters | Avl | [M12](milestones/M12-capacity-ha-dr.md) |
| S22 | Customer data missing | Delete rows / drop a table in the prod-like environment | User report or data-integrity check | Restoring the whole database over live data; not knowing the restore time until you try | Inc, Cont | [M12](milestones/M12-capacity-ha-dr.md) |
| S23 | Environment or region unavailable | `make down` of prod-like; block the region via SCP | Synthetic checks from outside | Discovering the runbook is incomplete. That's the point: do it twice | Cont, Avl | [M12](milestones/M12-capacity-ha-dr.md) |
| S24 | AKS app can't reach a private PaaS service | Unlink the private DNS zone from the VNet | Dependency error alert | "The service is up": it is, but its name resolves to the public IP and the traffic is blocked | Inc | [M13](milestones/M13-azure-core.md) |
| S25 | Cross-cloud call fails with 401 / invalid token | Change the service account name, or the federated credential's issuer / subject / audience | ai-gateway / insights-worker error rate | Creating a client secret to "get it working" | Inc, Sec | [M13](milestones/M13-azure-core.md) |
| S26 | Argo CD OutOfSync/Degraded; prod differs from Git | `kubectl edit` in prod; wrong sync-wave; invalid manifest merged | Argo CD health alert | Fixing it by hand in the cluster, which Argo reverts or which leaves Git lying | Inc, Chg | [M14](milestones/M14-aks-multicluster.md) |
| S27 | A credential is found in a public place | Treat a pipeline or app credential as leaked | Secret-scanning alert, or none (planned drill) | Rotating and stopping. Containment, exposure assessment and prevention are three more steps | Sec, Inc, Prb | [M15](milestones/M15-security-governance.md) |
| S28 | All deploys suddenly blocked | Assign an Azure Policy / SCP / Kyverno rule as Deny without an audit period | Pipeline failures across repos | Disabling the policy globally instead of a scoped, expiring exemption | Chg, Inc | [M15](milestones/M15-security-governance.md) |
| S29 | Bill jumped 40% | Leave a node pool scaled up; pull images through NAT; uncap log ingest | Cost anomaly (rate of change) alert; budget | Reporting the total. The job is attributing it to a resource, a change and a decision | Cost, Cap, Prb | [M16](milestones/M16-finops.md) |
| S30 | A new team can't onboard via the golden path | Onboard a workload needing a PVC or a non-HTTP port | Onboarding failure / support request | Hacking platform code for one consumer. Decide: extend the path or document an escape hatch | Prb, Chg | [M17](milestones/M17-golden-paths.md) |
| S31 | AI feature slow and erroring | Lower the TPM quota / deployment capacity; burst jobs into SQS | AI error-rate and 429 panel; queue age | Naive immediate retry, turning a rate limit into a self-inflicted outage | Inc, Cap, SLM | [M19](milestones/M19-ai-infrastructure.md) |
| S32 | AI spend rate spikes | Disable the cache; add a prompt bloat bug; loop a job | Cost-per-hour derivative alert on token metering | Waiting for the monthly budget alert. The derivative shows it hours earlier | Cost, Inc, Prb | [M19](milestones/M19-ai-infrastructure.md) |
| S33 | Agent gives a confidently wrong or manipulated answer | Log line with "ignore previous instructions…"; an ambiguous incident | Eval set; human review of the agent's evidence | Trusting fluent output. Check that the cited evidence exists and supports the claim | Sec, Prb | [M20](milestones/M20-ai-agents-mcp.md) |
| S34 | "Datacentre" workloads unreachable from cloud | Stop one VPN tunnel / the strongSwan service; break the DNS forwarder | Synthetic cross-network check; tunnel metric | Debugging the app on the far side before checking the tunnel state and routes | Inc, Avl | [M21](milestones/M21-hybrid-multicloud.md) |
| S35 | A release caused an outage; rollback needed at 2 a.m. | Ship a change that passes tests but breaks a real dependency contract | SLO burn correlated with a deploy marker | Fixing forward under pressure without an emergency change record | Chg, Inc, Prb | [M11](milestones/M11-sre-operations.md) |
| S36 | Cluster upgrade blocked / workloads break afterwards | Keep a removed API version or an EOL add-on in manifests | Upgrade insights / pre-flight check; post-upgrade errors | Upgrading first and reading the deprecation guide afterwards | Chg, Avl | [M09](milestones/M09-eks.md)/[M14](milestones/M14-aks-multicluster.md) |

**Minimum you have to work, with full written records:** eight from M11, then at least two per milestone after that. By [M22](milestones/M22-hardening-capstone.md) you should have **25+ incident records, 6+ problem records, 3+ PIRs**, and at least one scenario from every row above worked at least once.

---

## 6. Worked examples

These show the level of reasoning expected. Timestamps are illustrative. Your own records use real ones.

### Example 1 — S16: p95 latency up 6×, errors flat

**Detection.** 14:02. The `cargo-api` latency SLO fast-burn alert fires (L2, Sev2), and `ems-bridge` opens INC-031. It's a symptom alert, not a CPU alert. The fault started at 13:58, so detection lag was 4 minutes.
**Investigation.** Error rate flat, p50 flat, p95/p99 up six times. A flat p50 with a blown tail means *some* requests are affected, which points at one path or dependency rather than global resource pressure. It's scoped to `GET /shipments/{id}/eta`, prod on EKS only. Recent changes: the last deploy was 6 hours ago (Argo CD history), and there are no Terraform applies today.
**Mitigation.** 14:11. The ETA path calls `carrier-sim` with no timeout. You set a 300 ms timeout with a cached-ETA fallback through a feature flag, a standard change pre-approved for flags. p95 recovers by 14:14. **Mitigated in 12 minutes, inside Sev2's 4-hour target.**
**Root Cause.** The Tempo waterfall for a slow request shows 412 ms in the carrier call and 11 ms everywhere else. The trigger was the dependency slowing down. The cause is that the client had no deadline. The contributing factor is that nothing alerted on dependency latency, only on our own. Rejected hypotheses: database (pool-wait flat), throttling (zero throttled periods), a deploy (none).
**Permanent Fix.** CHG-044: a circuit breaker and a timeout budget in the shared HTTP client.
**Prevention.** The shared client wrapper goes into the golden-path template ([M17](milestones/M17-golden-paths.md)), so every onboarded workload gets timeouts, retries with budgets, and a breaker by default. You also add a dependency-latency alert. *Finding: we had no alert on our dependencies, only on ourselves.*

### Example 2 — S25: cross-cloud call fails with 401

**Detection.** 09:40. The `insights-worker` error-rate alert fires (L2, Sev3, because summaries degrade to queued retries). SQS queue age is climbing.
**Investigation.** Every call from `insights-worker` (EKS) to `ai-gateway` (AKS) returns 401. Other ai-gateway callers on AKS are fine, so the problem is in the cross-cloud identity path. Recent changes: an Argo CD sync at 09:31 renamed the worker's service account from `insights-worker` to `insights` as part of a naming clean-up.
**Mitigation.** Roll back the rename by reverting the commit in Git; Argo syncs it. Calls succeed at 09:52. The queue drains, so there was no data loss, only delay. That's why the async design matters.
**Root Cause.** The chain is: the pod's projected service-account token is issued by the EKS OIDC issuer with `sub=system:serviceaccount:insights:insights`. The Entra federated identity credential trusts `sub=system:serviceaccount:insights:insights-worker`. Entra refuses the token exchange, the worker gets no Entra token, and ai-gateway returns 401. You confirm it by decoding the projected token (`jq` on the JWT payload) and comparing it with the federated credential's subject. The cause is that the federated credential and the service account name were defined in two places (the Azure Terraform root and the Kubernetes manifest) with nothing checking they agree.
**Permanent Fix.** The `workload` module derives the service account name *and* the federated credential subject from the same `workload.yaml` field ([M17](milestones/M17-golden-paths.md)).
**Prevention.** A conftest rule: any change to a ServiceAccount name that is referenced by a federated credential fails the PR. A synthetic token-exchange check runs every 5 minutes. Rejected shortcut: "create a client secret for now". Written down, because that's the trap.

### Example 3 — S18: intermittent 500s under load

**Detection.** The error-rate burn alert during a scheduled load window (Sev2, L2).
**Investigation.** 500s happen only above about 120 req/s. The HPA has scaled `cargo-api` from 3 to 9 pods. Logs show `QueuePool limit … timed out`. The RDS `DatabaseConnections` metric is flat at the instance maximum.
**Mitigation.** Cap the HPA at 5 replicas (more pods mean more connections), and shed load on the non-critical listing endpoint with 503s and `Retry-After`. Errors stop, and p95 is degraded but inside the SLO.
**Root Cause.** Pods × pool size × workers exceeded `max_connections`. The autoscaler turned a load increase into connection exhaustion. Scaling out was the wrong direction for the saturated resource.
**Permanent Fix.** A connection budget calculation in the capacity plan; RDS Proxy (or PgBouncer) evaluated with numbers; pool sizing derived from the budget; the HPA max derived from it too.
**Prevention.** A pool-wait-time alert (the cause-level early warning), and a capacity-plan rule in [M12](milestones/M12-capacity-ha-dr.md): *every autoscaler max must be checked against every downstream connection limit.*

### Example 4 — S31: AI feature slow and erroring (429s)

**Detection.** The ai-gateway 429-rate panel crosses its threshold, and the `insights-worker` SQS queue age alert fires (Sev3, because the non-AI fallback keeps the feature partly working). The fallback-rate panel shows 38% of requests being served degraded.
**Investigation.** A batch backfill enqueued 4,000 exception jobs at 10:05. Azure OpenAI returns 429 with `retry-after`. The Bedrock fallback is also throttling because its on-demand quota is low. Token metering shows one team, `cargo`, using 94% of throughput.
**Mitigation.** Pause the backfill's producer. Lower `insights-worker` concurrency from 20 to 4. The gateway's per-team rate limit for `cargo` batch traffic goes down to 30% of the deployment's TPM. Interactive calls recover.
**Root Cause.** The worker retried 429s immediately (a hard-coded 3 retries, no jitter, ignoring `retry-after`), so every throttled call became four calls. Tokens-per-minute is a shared capacity pool with no per-caller share, and batch and interactive traffic shared one deployment.
**Permanent Fix.** Retries honour `retry-after` and use exponential backoff with full jitter and a retry budget. Batch traffic gets a separate priority class in the gateway (queued, lower TPM share). Worker concurrency is derived from TPM ÷ average tokens per job.
**Prevention.** AI capacity goes into the capacity plan (TPM/RPM as quotas, like vCPU). There's a load test of the AI path. There's an alert on the fallback rate and on the cost-per-hour derivative (S32 is its sibling).

### Example 5 — S29: bill jumped 40%

**Detection.** The AWS Cost Anomaly Detection alert on day 12 (Sev4 at first, raised to Sev3 once projected past budget). The cumulative budget alert would only have fired on day 19.
**Investigation.** The FOCUS-normalised report in `finops-collector` grouped by `service` and `environment` tags shows one cost centre, one environment and two line items: `NatGateway-Bytes` and EC2 on the `platform` node pool.
**Mitigation.** Scale the platform node pool back down. Add the ECR and S3 gateway / interface VPC endpoints in dev.
**Root Cause.** Two causes. (1) A Karpenter NodePool change on the 9th removed the consolidation policy, so nodes stayed up. (2) After a VPC rebuild, the ECR and S3 endpoints weren't recreated, so every image pull and Loki/Tempo S3 write went through NAT data processing. The endpoints had been added by hand in a lab and never codified.
**Permanent Fix.** Both fixes in Terraform, plus a conftest rule that workload VPCs must declare the S3/ECR/STS endpoints.
**Prevention.** The anomaly alert on daily rate of change stays. `idle-detector` flags node pools above baseline for more than 12 hours. *Finding: cumulative budget alerts are lagging indicators.* This write-up, on your own money, is the strongest FinOps evidence you can own.

---

## 7. Game days

From M11 onwards, one weekend session a month is a **game day**:

1. `inject-fault.sh --random --from <milestone-range>` picks a scenario and a time within a 2-hour window. You don't look at which.
2. You work the incident with alerts, dashboards and runbooks only. The runbooks are the ones you wrote, so their gaps become visible.
3. Afterwards, score yourself: time to detect, time to mitigate, the correct root cause, record quality, and whether the runbook was followable.
4. Every runbook stumble is a defect. Fix it the same day.

From [M14](milestones/M14-aks-multicluster.md), include cross-cloud scenarios. From [M20](milestones/M20-ai-agents-mcp.md), run each game day twice: once alone, once with `flightdeck`. Compare time-to-first-hypothesis honestly, including the times the agent was wrong.

## 8. From incidents to problems

Incidents are events. A problem is the condition that produces them. Every fourth incident, stop and ask which of them share a cause. Problems you should expect to find in your own records:

| Problem | Incidents that reveal it | Permanent fix, and where it lands |
|---|---|---|
| **Change outside the pipeline** | S06, S24, S26, S29 | SCP / Azure Policy restricting console writes in prod; Argo self-heal; drift detection with an owner |
| **No deadlines on dependency calls** | S16, S20, S31 | Standard client wrapper with timeouts, retry budgets and a breaker: **in the golden path** |
| **Detecting only bad things, never missing good things** | S03, S10, S17 | Synthetic checks and absence-of-data alerts as standard dashboards / alerts in the `workload` module |
| **Identity defined in two places** | S04, S14, S25 | Derive identity bindings from one source (`workload.yaml`) |
| **Autoscaling blind to downstream limits** | S18, S31 | Capacity-plan rule plus concurrency derived from downstream quotas |

**This is where operations feeds platform engineering.** Saying that in an interview shows you understand why platform teams exist: their backlog is largely made of permanent fixes to recurring problems.

## 9. Change failures and PIRs

The JD asks you to *"own and manage changes within the technology area."* The change process is introduced in [M11](milestones/M11-sre-operations.md) and recorded in ADR-013:

| Type | Definition in Runway | Approval | Example |
|---|---|---|---|
| **Standard** | Pre-approved, low risk, repeatable, automated with a tested rollback | None beyond the pipeline gates | Deploy a new app version through the golden path; toggle a feature flag |
| **Normal** | Anything else. Risk-scored (blast radius × reversibility × novelty) | PR review, plus the prod environment approval; high risk means a window and a written rollback plan | Node pool change, DB parameter change, policy moving from Audit to Deny |
| **Emergency** | Needed now to restore service | Retrospective approval within 1 working day; record created during the incident | S35 rollback at 2 a.m.; the S25 revert |

**Every failed change gets a PIR.** A failed change is one that caused an incident, needed a rollback, or didn't achieve its intent. The PIR answers why the pre-change evidence didn't predict the failure, and changes the process: a new test, a new check, a new risk factor. Track the **change failure rate** (a DORA metric) monthly. Freeze windows are written in the change calendar, which the portal shows ([M18](milestones/M18-developer-platform.md)).

The engineering argument you should be able to make: *the answer to heavyweight change control is small, frequent, reversible changes with automatically collected evidence.* The `change-record` workflow is that argument, implemented.

## 10. SLIs, SLOs, SLAs and error budgets in practice

| Term | Runway example | Who owns it |
|---|---|---|
| **SLI**, the measurement | Proportion of `cargo-api` requests (excluding 4xx) served successfully within 300 ms, measured at the ALB/gateway | Engineering |
| **SLO**, the target | 99.5% over a rolling 30 days | Engineering, agreed with the "business" (you, writing it down) |
| **SLA**, the promise with consequences | An internal OLA with the "cargo team": 99.0% monthly, with the severity response times above | Service owner / service level management |
| **Error budget** | 0.5% of 30 days ≈ 3.6 hours of bad minutes | Spent on releases and experiments; when it's exhausted, reliability work comes first |

In practice:

- **Alert on burn rate, not on the SLI dipping.** Multi-window, multi-burn-rate alerting (per *The SRE Workbook*, "Alerting on SLOs"): a fast burn (about 2% of budget in 1 h) pages as L2 Sev2; a slow burn (about 10% in 3 days) opens a Sev3 ticket.
- **Use the budget in a real decision at least once.** For example, freeze feature deploys for a week after S20 used 60% of the budget, and record the decision in the monthly review.
- **SLOs from M11 onwards:** `cargo-api` availability and latency; `runway-portal` availability; from M19, `ai-gateway` availability *including the degraded path* and a separate fallback-rate objective. AI quality is not an availability SLO. It's measured by the eval set.
- The SLA is always looser than the SLO, which is always looser than what you actually achieve. If you can't explain why, you don't understand the three yet.

## 11. The monthly operational review

One page, every month from M11, in `docs/operations/review-YYYY-MM.md`:

Incidents by severity and tier · MTTD, MTTA and MTTR · SLO attainment and error budget consumed per service · open problems and their age · changes by type, and the change failure rate · PIRs · alerts that fired (useful / noise / misclassified tier) · top three sources of toil and what you automated · cost against budget, the unit cost trend, and any anomaly · capacity headroom against the plan · the EOL register items due within 90 days · what you'd change about detection.

Twelve of these make a stronger artefact than any diagram in the repository. They're the evidenced answer to *"ensure that the developed architecture/solution meets service-level requirements in terms of expected availability, capacity, continuity, security, performance, and reliability."*

→ Next: [Section 11 — AWS Learning Path](11-aws-learning-path.md)
