# Step 9 — The Operations Playbook

The JD's role definition names six processes: *incident management, problem management, service level management, capacity management, availability management, and IT service continuity.* This file is where you acquire them. **It is the part of the roadmap that most candidates cannot replicate**, because it requires a system that is real enough to break and a habit of writing things down while under pressure.

Scenarios begin at Milestone 10 and continue through Milestone 15. Do not front-load them: an incident against a system with no observability is just confusion.

---

## 1. The loop

Every scenario runs these seven steps, in this order, with the artifact written **during** the incident rather than reconstructed afterwards.

| Step | The question | The discipline |
|---|---|---|
| **Detect** | How did I find out? | If the answer is "I caused it so I knew", the monitoring failed. Log that as a finding. Record the detection time and the source. |
| **Investigate** | What is the blast radius? | Scope before cause: which users, which endpoints, which environment, since when. Resist the first plausible explanation. |
| **Diagnose** | What is actually wrong? | Form a hypothesis, state what evidence would disprove it, then go get that evidence. Write down hypotheses you rejected — they are the most valuable part of the record. |
| **Mitigate** | How do I restore service *now*? | **Mitigation precedes understanding.** Roll back, scale out, fail over, disable the feature. A workaround inside the SLA is a successful outcome, which is exactly what the JD means by "where applicable, workarounds are provided to minimise business impact". |
| **Fix** | What is the permanent correction? | Often a separate change, days later, through the normal pipeline. Sometimes it belongs to problem management rather than to this incident. |
| **Verify** | How do I know it is genuinely better? | The same signal that detected it must return to normal. "It looks fine" is not verification. |
| **Document** | What would the next person need? | Timeline with timestamps, impact, cause, actions, follow-ups with owners, and what you would change about detection. |

**The two rules that make this real:** start a timer and a text file the moment you detect, and never edit the timeline to make yourself look better. The wrong turns are the evidence of judgement.

## 2. Severity matrix

Write this before the first drill. You cannot classify under pressure without a pre-agreed scale.

| Sev | Definition | Response | Target resolution | Comms |
|:--:|---|---|---|---|
| **1** | Service unavailable to all users, or data loss | Immediately, drop everything | 1 hour to mitigation | Every 30 min |
| **2** | Major function degraded, or SLO burning fast | Within 15 min | 4 hours to mitigation | Every hour |
| **3** | Minor function affected, workaround exists | Next working block | 2 days | On resolution |
| **4** | Cosmetic, or a risk with no current impact | Backlog | Next sprint | None |

## 3. Templates

**`docs/incidents/INC-0NN-short-name.md`**
```
# INC-0NN — <one-line symptom, not cause>
Severity: · Detected: <ts> by <source> · Mitigated: <ts> · Resolved: <ts>
Impact: <who, what, how long, how measured>

## Timeline
HH:MM  what I observed / did / concluded

## Hypotheses
| # | Hypothesis | Evidence for | Evidence against | Verdict |

## Cause
## Mitigation applied
## Permanent fix (or link to PRB)
## Follow-ups
| Action | Owner | Due |
## What would have detected this sooner?
```

**`docs/problems/PRB-0NN-short-name.md`**
```
# PRB-0NN — <underlying condition>
Related incidents: INC-00x, INC-01y, INC-02z
## Causal chain  (each link: because X, Y happened)
## Contributing factors
## Why it recurred
## Permanent fix and its cost
## Known error / workaround while the fix is pending
## Verification that it stopped recurring
```

## 4. The scenario catalogue

Sixteen scenarios. Each is injectable by `platform/scripts/inject-fault.sh` or by hand. **The "trap" column is the wrong turn most people take** — read it only after your first attempt.

| # | Scenario | How to inject | Should be detected by | The trap | Processes | From |
|:--:|---|---|---|---|---|:--:|
| 1 | **API latency suddenly increased** | Add 400 ms to `carrier-sim` | p95 latency SLO burn-rate alert | Blaming the database because it is the usual suspect; the trace says otherwise | Incident, SLM | M10 |
| 2 | **Pods are repeatedly restarting** | Lower the memory limit below working set | Pod restart rate alert | Reading only `kubectl describe` and missing exit code 137 and the previous container's logs | Incident, Troubleshooting | M10 |
| 3 | **Deployment succeeds but health checks fail** | Point readiness at a dependency that is down | Deployment success + availability drop | Trusting the pipeline's green tick. Fix the pipeline so it cannot lie to you again | Incident, CI/CD | M10 |
| 4 | **Application cannot reach the database** | Remove the NSG rule, or break the private DNS zone link | Dependency error rate alert | Assuming the database is down. It is up; DNS resolves to the public IP and the packet dies silently | Incident, Troubleshooting | M10 |
| 5 | **An IAM policy prevents deployment** | Delete the pipeline's role assignment | Pipeline failure | Reading the error as a networking problem because the message is unhelpful | Incident, Security | M10 |
| 6 | **Terraform reports infrastructure drift** | Change a resource in the portal | Nightly drift-detect workflow opens an issue | Blindly applying to reconcile. First ask *who changed it and why* — the answer may be a person solving a real problem | Problem, Governance | M10 |
| 7 | **Certificate expired** | Issue a 1-day cert | TLS expiry alert (and if you have none, that is the finding) | Renewing it and moving on, rather than asking why nothing warned you 30 days out | Incident, Availability | M10 |
| 8 | **Cascading failure from a slow dependency** | Make `carrier-sim` slow, not failing | Worker queue depth + thread pool saturation | Looking only at the failing service. The cause is that *nothing timed out* | Incident, Problem, Availability | M11 |
| 9 | **Cloud spending increased unexpectedly** | Leave a node pool scaled up over a weekend, or uncap Log Analytics | Budget threshold alert | Reporting the total. The job is attributing it to a resource, a change, and a decision | Incident, Cost, Capacity | M12 |
| 10 | **Traffic exceeds capacity** | Drive k6 past the measured knee | Latency SLO burn rate | Scaling out the wrong tier. Find the saturating resource first | Capacity, Incident | M11 |
| 11 | **A node is lost** | Cordon and drain, or evict a spot node | Pod rescheduling, brief availability dip | Discovering you had no PodDisruptionBudget only when it matters | Availability | M11 |
| 12 | **Database outage and restore** | Stop the server; then delete data | Dependency alert | Restoring the whole database when you needed one table; not knowing until you tried | Continuity, Incident | M11 |
| 13 | **Full environment loss** | `terraform destroy` on dev, deliberately | n/a — a planned drill | Following the runbook and finding it incomplete. That is the point; do it twice | Continuity, DR | M11 |
| 14 | **A leaked credential** | Assume a pipeline credential is public | n/a — a planned drill | Rotating and stopping. Containment, exposure assessment and prevention are three more steps | Security incident | M15 |
| 15 | **Log ingestion stopped** | Exceed the daily cap | Absence-of-data alert — which you must have built | Not having an alert for *missing* telemetry. Silence is the most dangerous signal | Monitoring, Cost | M12 |
| 16 | **An AI service is returning high latency** | Exceed TPM quota, or force 429s | AI latency/error panel and budget alert | Retrying immediately and making the rate limit worse | Incident, AI ops | M14 |

## 5. Three worked examples

### Scenario 1 — API latency suddenly increased

**Detect** — 14:02, p95 burn-rate alert for the `cargo-api` latency SLO. Not a CPU alert: you alert on symptoms.
**Investigate** — Dashboard: error rate flat, p50 flat, p95 and p99 up 6×. Flat p50 with a blown tail means *some* requests are affected, which points at a specific path or dependency, not at global resource pressure. Scoped to one endpoint, started 13:58, all environments.
**Diagnose** — Trace waterfall on a slow request: 412 ms in the carrier call, 11 ms everywhere else. Hypothesis: the dependency is slow. Disproving evidence would be a trace with a fast carrier call and slow total; there is none. Rejected hypotheses, recorded: database (pool metrics flat), CPU throttling (throttle metric zero), a recent deploy (last deploy was 6 hours prior).
**Mitigate** — 14:11: the endpoint has no timeout on the carrier call. Apply a 300 ms timeout with a cached-fallback response. p95 recovers by 14:14.
**Fix** — Circuit breaker around the carrier client, plus a dependency-latency alert so next time detection is at the cause rather than at the symptom.
**Verify** — p95 back under the SLO threshold for 30 minutes; error budget burn stops.
**Document** — Timeline, hypotheses table, and the finding: *we had no alert on dependency latency, only on our own.* That finding is worth more than the fix.

### Scenario 4 — Application cannot reach the database

**Detect** — Dependency error rate alert; `cargo-api` readiness failing, so the pod is out of the load balancer and users see 503.
**Investigate** — Scope: all pods, all requests to the DB, started at 09:14. The database is up: the portal shows it healthy and you can reach it from your laptop over the VPN.
**Diagnose** — `kubectl exec` into a pod and run Milestone 2's `net-triage.sh`. DNS resolves the database FQDN to a *public* IP, not the private endpoint address. Hypothesis: the private DNS zone link to the VNet was removed. Confirmed in the portal. The packet is then dropped by the firewall — hence a timeout rather than a refusal, which is the clue you learned to read in Milestone 2.
**Mitigate** — Recreate the zone link. Resolution recovers in under the TTL.
**Fix** — Add `prevent_destroy` to the link, and a synthetic check that asserts the FQDN resolves to a private address.
**Verify** — Readiness green, error rate zero, synthetic check passing.
**Document** — Note that this began as a Terraform drift event: someone (you) removed it out of band. This incident and scenario 6 share a cause, which makes them a **problem**, not two incidents.

### Scenario 9 — Cloud spending increased unexpectedly

**Detect** — 80% budget alert on day 19 of the month, when the run rate should have hit 80% around day 24.
**Investigate** — Cost analysis grouped by your tags. One service, one environment, one resource type accounts for the delta. Without the tagging standard from Milestone 12 this step is impossible — which is the lesson.
**Diagnose** — Cost by day shows a step change on the 16th. Correlate with `docs/` and Git history: an HPA `maxReplicas` change merged on the 15th, combined with a load test left running. Two causes, not one; write both down.
**Mitigate** — Scale down; kill the load test.
**Fix** — A cost anomaly alert on daily rate of change rather than only on cumulative threshold; a scheduled job that flags node pools scaled above baseline for more than 12 hours.
**Verify** — Daily run rate returns to baseline; project the month-end total and compare with the original forecast.
**Document** — Before/after numbers, and the observation that a *cumulative* budget alert is a lagging indicator. **This write-up, on your own money, is the most persuasive FinOps evidence you can own.**

## 6. From incidents to problems

Incidents are events; a problem is the condition that produces them. Every fourth incident, stop and ask: which of these share a cause?

Three problems you should expect to find in your own records:
- **Out-of-band changes** (scenarios 4, 6, 7) → the cause is that change can happen outside the pipeline. Permanent fix: policy denying manual changes, plus drift detection with an owner.
- **Missing timeouts** (scenarios 1, 8, 16) → the cause is that no dependency call has a deadline. Permanent fix: a standard client wrapper in the golden path, so every onboarded workload gets timeouts, retries and a breaker by default. **This is where operations feeds platform engineering**, and saying that in an interview shows you understand why platform teams exist.
- **Detection gaps** (scenarios 3, 7, 15) → the cause is alerting on the presence of bad things and never on the absence of good things. Permanent fix: synthetic checks and absence-of-data alerts.

## 7. The monthly operational review

One page, monthly, in `docs/operations/review-YYYY-MM.md`:

Incidents by severity · MTTA and MTTR · SLO status and error budget consumed · open problems and their age · top three sources of toil and what you automated · cost versus budget and any anomaly · changes deployed and change failure rate · what you would change about detection.

Twelve of these is a stronger artifact than any diagram in your repository. It is the direct, evidenced answer to *"support the IT production processes... governing incident management, problem management, service level management, capacity management, availability management, and IT service continuity."*

→ Next: [Step 10 — Platform engineering](10-platform-engineering.md)
