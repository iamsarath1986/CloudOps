# Step 7 — The Learning Path (Phases 6–11)

← [Phases 0–5](06-learning-path-phases-00-05.md)

---

# Phase 6 — CI/CD and release engineering
*~3 weeks. Prerequisite: Phases 3, 4, 5.*

### Concepts to learn
The pipeline as a product with users, not a script. **Build once, promote by digest** — the single most important idea in this phase: the artifact tested in dev must be bit-identical to the one deployed to production, which means never rebuilding per environment. Pipeline stages and what belongs in each: fast feedback first (lint, types, unit tests), slow and expensive last. Caching and why a slow pipeline is routed around. Environments, protection rules and approvals. Secrets in CI, and why OIDC (Phase 3) means there are almost none. Deployment strategies — recreate, rolling, blue-green, canary — and their cost/risk trade-offs. **Rollback as a first-class capability**, measured in seconds, not hoped for. Migration ordering and the expand-contract pattern, because "deploy code before migration" and "migration breaks the still-running old version" are the two classic deployment outages. Plan-on-PR / apply-on-merge for infrastructure. Reusable workflows and composite actions — the mechanism the golden path is later built on.

### Hands-on labs
1. Build a pipeline that fails at each stage in turn: a lint failure, a type failure, a test failure, a scan failure, a smoke-test failure. Confirm each is caught where you expect and that the error message is actionable.
2. Deliberately rebuild the image in the prod job instead of promoting the digest. Introduce a floating dependency. Watch dev and prod diverge. Fix it by promoting the digest and never do it again.
3. Time a rollback. Deploy a broken version, roll back, and record the seconds. That number belongs in your README.
4. Implement expand-contract: add a nullable column, deploy code that writes both, backfill, deploy code that reads the new column, drop the old. Then try the naive version and break production on purpose.
5. Move Terraform from your laptop into the pipeline: plan as a PR comment, apply on merge with an environment approval for prod.

### Project implementation
Three reusable workflows in `.github/workflows/`: `build-and-push` (build, scan, SBOM, push by digest), `deploy` (deploy a digest to an environment, run a smoke test, roll back on failure), `tf-plan-apply` (plan on PR, apply on merge, with approval on prod). `cargo-api` and `carrier-sim` call them. Dev deploys automatically on merge; prod requires approval. A nightly `drift-detect` workflow runs `terraform plan` and opens an issue if drift exists.

### Production considerations
Enterprises add change management: a change record linked to the deployment, a change advisory board for high-risk changes, freeze windows, and a deployment calendar. The engineering answer to heavyweight change management is small, frequent, reversible changes with automated evidence — be able to argue that position.

### Failure exercises
Break OIDC trust and diagnose the authentication failure. Deploy a migration before its code. Let a deploy succeed while readiness probes fail, so the pipeline reports green and the service is down — then fix the pipeline so it cannot lie to you again. Exhaust a GitHub Actions concurrency limit and watch queueing.

### Verification
Explain build-once-promote and what specifically breaks without it. State your rollback time and how you measured it. Explain expand-contract on a whiteboard. Justify every gate in your pipeline — if you cannot say what a gate prevents, delete it.

### Deliverable
Three reusable workflows consumed by two repositories. Two environments with approval on prod. A measured rollback time in the README. `docs/runbooks/failed-deployment.md`. `docs/adr/ADR-005-deployment-strategy.md`.

---

# Phase 7 — Kubernetes
*~5 weeks — the longest phase, deliberately. Prerequisite: Phases 4, 5, 6.*

> **Learn on `kind` locally for free; operate on AKS.** Do not spend budget watching a pod start. Spend it on the things only a real cluster teaches: node pools, autoscaling, upgrades, spot eviction, cloud load balancers and cloud identity.

### Concepts to learn
The control plane (API server, scheduler, controller manager, etcd) and the node components (kubelet, kube-proxy, container runtime). The **reconciliation loop** — declare desired state, controllers converge toward it — which is the single idea that makes everything else predictable. Pods, ReplicaSets, Deployments, StatefulSets, DaemonSets, Jobs and CronJobs, and when each is right. Services (ClusterIP, NodePort, LoadBalancer), Ingress and ingress controllers. ConfigMaps and Secrets, and why a Kubernetes Secret is base64, not encryption. Probes: liveness, readiness, startup — and the classic self-inflicted outage of a liveness probe that restarts a healthy-but-slow pod. Requests and limits, QoS classes, and what CPU throttling actually does to latency. The scheduler: node selectors, affinity, taints and tolerations, topology spread. HPA, cluster autoscaler, and why they interact badly if requests are wrong. Rollouts, `maxSurge`/`maxUnavailable`, and PodDisruptionBudgets. Namespaces, ResourceQuotas and LimitRanges as multi-tenancy tools. RBAC in the cluster. NetworkPolicy as default-deny. Persistent volumes and storage classes. Helm and Kustomize — what each is for, and why you will use Helm for third-party charts and a thin templating layer for your own. AKS specifics: node pools, the free control plane tier, spot nodes, workload identity, the CSI secrets driver, and the version upgrade model.

### Hands-on labs
1. On `kind`: deploy `cargo-api`, then deliberately cause and diagnose **each** of `ImagePullBackOff`, `CrashLoopBackOff`, `OOMKilled`, `Pending` (unschedulable), `CreateContainerConfigError`, and a readiness probe that never passes. For each, write the three commands that identify it. This list *is* the Kubernetes interview.
2. Set a liveness probe with a timeout shorter than your startup time. Watch the restart loop. Fix it with a startup probe.
3. Set CPU limits at 100m on a busy service. Watch p99 latency explode from throttling while CPU usage looks fine. This is the most misunderstood Kubernetes performance problem.
4. Roll out a broken image with `maxUnavailable: 0` and watch the rollout stall rather than take the service down. Then `kubectl rollout undo` and time it.
5. Apply a default-deny NetworkPolicy and break your own service. Then allow exactly what is needed.
6. On AKS: drain a node and watch pods reschedule. Then run a cluster version upgrade with traffic flowing, using a PDB.
7. Evict a spot node and observe what your workload does. Decide what belongs on spot and what does not.

### Project implementation
AKS via a `platform/modules/aks` module: free-tier control plane, small system pool, spot user pool, workload identity enabled, Key Vault CSI driver, Managed Prometheus enabled. `ingress-nginx` with TLS from cert-manager. A namespace per workload with ResourceQuota and LimitRange. `cargo-api`, `carrier-sim` and the worker deployed with correct probes, requests/limits, PDBs and HPA. Secrets mounted from Key Vault via CSI — no secrets in manifests. Default-deny NetworkPolicy with explicit allows. The `deploy` workflow from Phase 6 now targets AKS and watches rollout status.

### Production considerations
Enterprises run private clusters with no public API endpoint, admission control (Gatekeeper/Kyverno) enforcing policy at the cluster level, multiple node pools per workload class, cluster upgrades on a maintenance calendar, and a platform team that owns the cluster while application teams own namespaces. That boundary — cluster owner vs namespace tenant — is exactly the boundary Phase 13 productises.

### Failure exercises
Every failure in Lab 1, reproduced on AKS rather than `kind`. Exhaust a ResourceQuota and watch deployments fail to schedule. Delete the ingress controller and diagnose the 502 from the outside in. Expire the TLS certificate. Let a node run out of disk and watch eviction. Misconfigure workload identity so the pod cannot reach Key Vault, then trace the token exchange.

### Verification
Given only `kubectl`, diagnose any of the six pod failure states in under five minutes and explain the cause. Explain what happens, step by step, between `kubectl apply` and a running pod. Explain why CPU limits can make a service slower. Explain requests vs limits and how they drive scheduling and autoscaling.

### Deliverable
An AKS cluster entirely in Terraform. All workloads running with probes, resource policy, PDBs, HPA and NetworkPolicy. `docs/runbooks/pod-failure-triage.md` — your six-state diagnostic guide. `docs/runbooks/aks-upgrade.md`, written from an upgrade you performed. `docs/adr/ADR-002-aks-vs-container-apps.md`.

---

# Phase 8 — Observability
*~3 weeks. Prerequisite: Phase 7.*

### Concepts to learn
The three signals and what each is *for*: logs for detail, metrics for aggregate trend and alerting, traces for causality across hops. Why traces lead in a distributed system. OpenTelemetry: spans, attributes, context propagation, exporters — and why choosing the standard protects your instrumentation from a backend change. Structured logging with correlation IDs. **RED** (rate, errors, duration) for request-driven services and **USE** (utilisation, saturation, errors) for resources — two checklists that cover most of what you need. Cardinality: why a user ID as a metric label creates millions of time series and a memorable bill. Percentiles versus averages, and why p99 is a real customer. SLIs, SLOs and error budgets. **Alerting on symptoms, not causes** — alert on "users are getting errors", never on "CPU is 80%", because cause-based alerts produce noise and get muted. Log sampling and retention as cost control. What must never be logged.

### Hands-on labs
1. Instrument `cargo-api` and `runway-portal` with OpenTelemetry. Fire one request and read the waterfall. Find the hop you were wrong about.
2. Break context propagation between the API and the worker. Watch the trace fragment into orphans. Fix it. This is the most common tracing bug in queue-based systems.
3. Add a high-cardinality label (shipment ID) to a metric. Watch the series count and the ingestion cost climb. Remove it, and write the rule you derived into the dashboard description.
4. Write ten KQL queries against Log Analytics that you will actually reuse: errors by endpoint over time, p95 latency by route, failed deployments, top talkers, unusual 5xx sources, and so on. Save them. Flightdeck will later call these exact queries.
5. Deliberately exceed your Log Analytics daily cap. Watch ingestion stop. Now you understand why log volume is an operational concern.
6. Author one dashboard as code in Grafana and commit the JSON. Never click a dashboard into existence again.

### Project implementation
OpenTelemetry across all workloads, exporting to Managed Prometheus and Log Analytics. Structured JSON logs carrying `trace_id` and `request_id`. Four dashboards, all as code in `platform/observability/`: **user experience** (rate, errors, latency by endpoint), **system health** (pod restarts, CPU throttling, memory, connection pool, queue depth), **dependencies** (database, carrier-sim, Azure API error rates and latency), **cost and capacity** (spend by tag, node utilisation, HPA activity). Two SLOs — `cargo-api` availability and p95 latency — with multi-window burn-rate alerts. Four to six alert rules, each symptom-based, each with a runbook link in its description.

### Production considerations
Enterprises centralise observability across hundreds of services, which makes cardinality and retention a budget line rather than a detail; they enforce a logging schema so cross-service queries work; and they treat "no alert without a runbook" as policy. Data residency and PII handling in telemetry are real constraints — a trace attribute containing a customer identifier is a data incident.

### Failure exercises
Delete a dashboard and rebuild it from code in under five minutes — if you cannot, it was not code. Send a sensitive field into a span attribute, find it in your telemetry backend, and implement redaction. Create a flapping alert and experience alert fatigue for a day, then fix the threshold and write down what you learned about hysteresis.

### Verification
Given a latency regression, show how you would find the cause using traces, then confirm it with metrics, then get the detail from logs — in that order. Explain the difference between an SLI, an SLO and an SLA. Justify every alert you have: what symptom, what runbook, has it ever fired usefully.

### Deliverable
`platform/observability/` containing dashboards, alert rules and SLO definitions as code. A saved KQL query library. `docs/runbooks/` with one runbook per alert. An OpenTelemetry trace screenshot in the README showing a full request path.

---

# Phase 9 — Reliability and operations
*~4 weeks. Prerequisite: Phase 8. **This is the phase that distinguishes you for this specific JD.***

### Concepts to learn
The ITIL processes named in the role definition, and what each actually means in engineering terms. **Incident management**: restore service first, understand later; severity classification; the incident commander role even when it is only you; communication cadence; workarounds as legitimate outcomes within an SLA. **Problem management**: an incident is an event, a problem is its underlying cause; known-error records; the discipline of not closing a problem when the incident closes. **Service level management**: SLA vs SLO vs SLI, measurement windows, and what an SLA breach costs. **Availability management**: availability maths, single points of failure, MTBF/MTTR, and why MTTR is usually the cheaper lever. **IT service continuity**: RTO and RPO as business decisions that constrain architecture. Root cause analysis: causal chains, contributing factors, and why "human error" is never a root cause. Blameless postmortems. Toil, and automation as its cure. On-call practice: escalation, handover, and the written timeline.

### Hands-on labs
1. Write your severity matrix (Sev1–Sev4) with response and resolution targets, before the first incident. You cannot classify under pressure without a pre-agreed scale.
2. Write incident and RCA templates. Use them for every drill from now on.
3. Run your first full drill end to end with a timer: break something, let the alert find you, and work **Detect → Investigate → Diagnose → Mitigate → Fix → Verify → Document**. Write the timeline *during* the incident, not afterwards.
4. Run a game day: have a script inject one of five failures at random while you are not watching. Being genuinely surprised is the point.
5. Take three related incidents and write one RCA that explains all three. That is problem management.
6. Calculate the composite availability of your architecture from its components' SLAs. Compare with what you actually measured. Explain the gap.

### Project implementation
`docs/incidents/` and `docs/problems/` with templates and the records from every drill. A severity matrix and an escalation policy in `docs/runbooks/incident-response.md`. A fault-injection script in `platform/scripts/inject-fault.sh` covering at least eight failure modes. An operational review document written monthly: incidents this month, MTTR, SLO status, error budget consumed, top toil, and what you automated.

### Production considerations
Enterprises have a service desk, ticket SLAs, major incident procedures with a bridge call, formal problem records, and monthly service review meetings with the business. The engineering skill is the same; the overhead is communication. Being able to say "I resolved incidents within agreed service levels and provided workarounds to minimise business impact" *with evidence* is a near-verbatim quote from this JD.

### Failure exercises
Work the full scenario catalogue in [Step 9](09-operations-playbook.md) — twelve-plus scenarios, each end to end, each producing a written record.

### Verification
Explain the difference between incident and problem management with an example from your own records. Given an alert, describe your first five minutes. Explain why you mitigate before you diagnose. Present an RCA you wrote and defend the causal chain against questioning.

### Deliverable
At least ten incident records and four RCAs, all from drills you actually ran. A severity matrix. A fault injection script. `docs/runbooks/incident-response.md`. One monthly operational review.

---

# Phase 10 — Capacity and performance
*~2.5 weeks. Prerequisite: Phase 9.*

### Concepts to learn
Open-loop vs closed-loop load models, and why a fixed-user closed-loop test hides the queueing behaviour real traffic produces. Percentiles, histograms and why averages lie. Little's Law (`L = λW`) and using it to predict queueing. Saturation and the knee of the latency curve. Finding the *saturating resource* — usually a connection pool, a thread pool, a CPU limit or a downstream dependency, rarely the thing you expected. Bottleneck shifting: fix one and the next appears. HPA metrics and stabilisation windows; cluster autoscaler lag. Azure quotas and limits as hard capacity ceilings. Headroom policy and forecasting. Cost per request as the capacity metric that connects to FinOps.

### Hands-on labs
1. Load-test `cargo-api` with k6 using an open model at increasing arrival rates. Plot p50, p95, p99 against rate. Find the knee.
2. Identify the saturating resource at the knee. Fix it. Re-run. Find the *next* bottleneck. Repeat three times, and record the sequence — this progression is the lesson.
3. Reproduce connection pool exhaustion: set the pool to 5, drive 50 concurrent requests, and watch latency become queue time.
4. Set CPU limits too low and observe throttling as a latency problem with no CPU alarm firing.
5. Tune the HPA: watch it react too slowly, then too aggressively (flapping), then correctly. Understand stabilisation windows from behaviour, not documentation.
6. Make `carrier-sim` slow (not failing). Watch it consume your worker capacity. Implement a timeout and a circuit breaker and re-run.

### Project implementation
k6 scripts in `platform/loadtest/` for the three main paths. A load test report format: environment, traffic model, arrival rates, p50/p95/p99, error rate, saturating resource, what changed. A capacity plan in `docs/capacity/capacity-plan.md`: current measured ceiling, the resource that limits it, headroom policy, the growth rate that would breach it, and the Azure quotas that would need raising. Tuned HPA and resource requests derived from the tests, not guessed.

### Production considerations
Enterprises forecast from business events — a sale, a schedule change, a marketing campaign — and translate them into capacity requests weeks ahead, including quota increases that take days to approve. "Provide technical input to assist with Capacity Management" means exactly this translation.

### Failure exercises
Drive the system past its knee in production-like conditions and watch it degrade. Decide whether it degrades gracefully (shedding load, returning 503 quickly) or badly (queueing until everything times out). Implement admission control and re-run. Exhaust a subscription quota during a scale-out and diagnose it.

### Verification
State your system's maximum sustainable request rate and the resource that limits it. Explain why p99 matters more than the average. Explain how a slow dependency causes a total outage, and the two mechanisms that prevent it.

### Deliverable
k6 scripts and at least three load test reports showing the bottleneck progression. `docs/capacity/capacity-plan.md`. Tuned HPA and resource settings with the measurements that justify them.

---

# Phase 11 — Continuity: backup, restore and disaster recovery
*~3 weeks. Prerequisite: Phases 5, 7, 9. Moved earlier than your draft, so you have time to act on what it reveals.*

### Concepts to learn
RTO and RPO as business decisions that then constrain architecture and cost. Backup versus replication versus snapshot — and that replication is not a backup, because it faithfully replicates your deletion. Point-in-time restore. Backup retention, immutability and the ransomware consideration. **The untested backup is not a backup** — restore is the capability, backup is only the precondition. Availability zones versus regions, and what each protects against. Failover time, data loss on failover, and failback (the part everyone forgets). Dependency ordering during recovery: what must come up before what. Runbooks written for someone who is not you, at 3 a.m., under stress. Business continuity versus disaster recovery.

### Hands-on labs
1. Restore your database to a point in time five minutes ago. Measure how long it took. That is your real RPO and RTO for data.
2. Destroy the entire environment (`terraform destroy`) and rebuild it from code with data restored. Time every stage. Do it twice: the second time will be dramatically faster and the gap is the value of the runbook.
3. Delete a row you care about and recover it *without* restoring the whole database. Discover whether you can.
4. Lose Terraform state and recover from blob versioning.
5. Simulate region loss: document exactly what you would do, what you would lose, and how long it would take. Be honest that you are not testing it for real, and say why (cost) — that honesty is more credible than a claimed multi-region setup.
6. Hand your DR runbook to someone else (or to yourself in a month) and watch where they get stuck. Every stumble is a runbook defect.

### Project implementation
Automated backups with defined retention. A `backup-verify` job that restores to a scratch instance weekly and asserts a row count — because an unverified backup is a belief, not a control. `docs/runbooks/disaster-recovery.md` with the dependency-ordered rebuild procedure and measured timings. `docs/runbooks/database-restore.md`. An RTO/RPO statement in the README with the numbers you measured, not targets you hope for.

### Production considerations
Enterprises run DR tests on a schedule with auditors watching, maintain a business impact analysis that assigns RTO/RPO per service tier, and hold cross-region capacity reservations. The failure mode they are guarding against is exactly the one you will discover in Lab 2: a documented DR plan nobody has executed.

### Failure exercises
Delete the production database and recover it, timed. Corrupt a backup and discover it during the restore (this is why verification jobs exist). Let a certificate expire during the rebuild and find out that DR and certificate management are coupled. Lose access to the Key Vault and understand what "unrecoverable" means in practice — then enable purge protection.

### Verification
State your RTO and RPO with the measurement that produced them. Explain the order in which your system must be rebuilt and why. Explain why replication is not a backup with a concrete scenario. Hand someone your DR runbook and have them succeed without asking you a question.

### Deliverable
`docs/runbooks/disaster-recovery.md` with measured timings from at least two executions. An automated backup verification job. Measured RTO/RPO in the README. `docs/incidents/` entry for the drill, written as if it were real.

→ Next: [Phases 12–17](08-learning-path-phases-12-17.md)
