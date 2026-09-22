# Milestone 11 — Capacity and Continuity

*Phases 10 and 11 · ~5.5 weeks · Prerequisite: Milestone 10*

> Two of the JD's named processes — Capacity Management and IT Service Continuity — delivered together because they share a question: *what happens when the system is pushed beyond what it was designed for, or loses a piece of itself?*

**Learning topics**

*Capacity:* open- vs closed-loop load models, and why a fixed-user test hides queueing · percentiles and histograms · Little's Law and the knee of the latency curve · finding the *saturating resource* (usually a connection pool, thread pool, CPU limit or downstream dependency — rarely what you expected) · bottleneck shifting · HPA stabilisation windows and cluster autoscaler lag · Azure quotas as hard ceilings · headroom policy · cost per request.

*Continuity:* RTO and RPO as business decisions that constrain architecture · backup vs replication vs snapshot, and why replication faithfully replicates your deletion · point-in-time restore · immutability and the ransomware consideration · **the untested backup is not a backup** · zones vs regions · failover, data loss on failover, and failback · dependency ordering during recovery · runbooks written for someone who is not you, at 3 a.m.

**Build**
k6 scripts in `platform/loadtest/` for the three main paths, with a standard report format. `docs/capacity/capacity-plan.md`: measured ceiling, the limiting resource, headroom policy, the growth rate that would breach it, and the Azure quotas that would need raising. `docs/runbooks/disaster-recovery.md` and `docs/runbooks/database-restore.md`, both with timings you measured.

**Cloud resources**
Automated PostgreSQL backups with defined retention · a `backup-verify` Container Apps job that restores weekly to a scratch instance and asserts a row count · blob versioning on the state container · tuned HPA and node pool autoscaling.

**DevOps work**
The backup verification job runs on a schedule and alerts on failure — because an unverified backup is a belief, not a control. Load tests runnable from the pipeline against dev.

**Operations exercise**
Load-test with an open model at increasing arrival rates. Plot p50/p95/p99 against rate and find the knee. Fix the saturating resource, re-run, find the *next* bottleneck. **Repeat three times and record the sequence** — that progression is the lesson. Then make `carrier-sim` slow rather than failing, watch it consume your worker capacity, and fix it with a timeout and a circuit breaker. Then: destroy the entire environment and rebuild it from code with data restored, timing every stage. **Do it twice** — the gap between the two runs is the value of the runbook.

**Security considerations**
Lose access to the Key Vault and find out what "unrecoverable" means in practice; then enable purge protection. Let a certificate expire during a rebuild and discover that DR and certificate management are coupled.

**Expected outcome**
A stated maximum sustainable request rate with the resource that limits it, and a stated RTO/RPO **measured, not hoped for**, in the README.

**Evidence of mastery**
State your system's ceiling and its limiting resource. Explain why p99 matters more than the average. Explain how a *slow* dependency causes a total outage and the two mechanisms that prevent it. Explain your rebuild order and why. Hand someone your DR runbook and have them succeed without asking you a question.
