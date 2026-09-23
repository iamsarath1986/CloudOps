# Milestone M10 — Observability

*Phase 10 · ~4 weeks · Months ~10.6–11.5 · Prerequisites: [M09](M09-eks.md) · Unlocks: [M11](M11-sre-operations.md), [M14](M14-aks-multicluster.md), [M16](M16-finops.md)*

---

## Why this phase exists

The JD asks for *"experience with monitoring, logging, observability and troubleshooting across application and cloud infrastructure layers"* and requires you to *"collaborate with other IT Technology teams to implement ongoing monitoring of the relevant technology to ensure Level 1 and Level 2 incidents are configured within EMS"*. You cannot run incident management (M11), capacity work (M12) or FinOps (M16) on a system you cannot see. An incident against a system with no telemetry is just confusion.

It sits after EKS because there is now a real, multi-hop system: browser → CloudFront/ALB → pod → Redis/RDS/SQS → worker → carrier-sim. It sits before operations because every alert you build here becomes the entry point to a drill in M11.

## Concepts to learn

**The signals and what each is for**
- **Metrics** for trends, aggregation and alerting. **Logs** for detail about discrete events. **Traces** for causality across hops. **Profiles** get a mention, as the fourth signal. Know which question each one answers cheaply and which it cannot answer at all.
- **RED** (rate, errors, duration) for request-driven services, and **USE** (utilisation, saturation, errors) for resources: nodes, connection pools, queues, NAT, RDS.
- **Percentiles versus averages.** Histograms and bucket boundaries. Why you cannot average percentiles across pods, and why p99 is a real customer.
- **Cardinality.** Every unique label combination is a time series. A `shipment_id` label is a memorable bill, and in a managed Prometheus it is literally metered.

**How the pipeline works internally**
- **OpenTelemetry:** SDKs (auto- and manual instrumentation), the data model (resource attributes, spans, span events, metrics instruments), **W3C `traceparent` context propagation**, and propagation *through SQS message attributes*. The Collector: receivers → processors (batch, memory_limiter, k8sattributes, attributes/redaction, tail sampling) → exporters. Agent (DaemonSet) versus gateway (Deployment) topology.
- Prometheus model: pull scraping, remote-write, recording rules, PromQL `rate()` versus `irate()`, the `histogram_quantile()` pitfalls, and staleness.
- Loki's model: indexes labels, not content, so **label discipline equals cost discipline**. LogQL. Tempo's object-store backend, trace-ID lookup, and TraceQL.
- **Exemplars:** jumping from a latency spike on a metric to the exact trace.

**Using the signals**
- **SLIs from telemetry:** a good-events/total-events ratio for availability, and a latency threshold ratio for latency. SLOs and burn rate are introduced here and used seriously in M11.
- **Alert on symptoms, not causes.** Alert on "users are getting errors or slowness", never on "CPU is 80%". Cause-based signals belong on dashboards. Every alert is either actionable or deleted.
- **Absence of data is a signal.** `absent()` / "no data" alerting. A silent dashboard is the most dangerous state.
- Structured logging: JSON, a fixed schema (`ts, level, service, env, trace_id, span_id, request_id, msg, …`), and what must never be logged (tokens, PII, secrets). Redaction in the Collector as defence in depth.
- Retention, sampling and cost. Head versus tail sampling (keep all errors and slow traces, sample the rest).
- **Native cloud signals still matter:** CloudWatch for ALB access/target metrics, RDS (Performance Insights, connections), SQS age of oldest message, Lambda, VPC Flow Logs, and NAT gateway bytes. Grafana reads them as data sources. You must be able to use CloudWatch Logs Insights directly when the in-cluster stack itself is down.

## Architecture decisions in this phase

**ADR-012 Observability architecture.**

| | |
|---|---|
| **Problem** | Runway needs one place to see metrics, logs and traces across workloads and, from M14, **two clusters in two clouds**, with history that survives `make down`, on a small budget |
| **Options** | (a) Cloud-native only: CloudWatch Container Insights + X-Ray now, Azure Monitor later. (b) Self-hosted Prometheus/Loki/Tempo/Grafana with local disks. (c) **OTel everywhere + Amazon Managed Service for Prometheus (metrics) + Loki & Tempo on S3 + Grafana OSS**, native signals as data sources. (d) SaaS (Grafana Cloud free tier) |
| **Decision** | (c). OTel keeps instrumentation vendor-neutral. AMP stores metrics outside the cluster, so SLO history survives teardown. S3-backed Loki/Tempo are cheap and survive teardown. Grafana is stateless, with dashboards as code |
| **Trade-offs** | You operate Loki/Tempo/Grafana/Collector (versions, sizing, and their own alerts). **The stack sits inside the AWS blast radius**: if EKS or the region is down, your observability is down. Mitigation: CloudWatch/Azure Monitor remain the fallback, documented in the DR runbook (M12). (d) is the documented budget fallback. (a) was rejected because it would give two siloed tools after M13 |

## Hands-on labs

1. **Instrument and read a waterfall.** Auto-instrument `cargo-api` (FastAPI, SQLAlchemy, Redis, httpx), fire one request, and read the trace. Find the hop you guessed wrong. *Proves: traces show causality.*
2. **Break propagation across SQS.** Publish a job without injecting `traceparent` into the message attributes and watch the worker start orphan traces. Fix it with manual propagation. *Proves: async boundaries are where tracing breaks.*
3. **Cardinality bomb.** Add `shipment_id` as a metric label, generate traffic, and watch active series in AMP (and the cost estimate) climb. Remove it and write the rule into `platform/observability/README.md`. *Proves: labels are a budget.*
4. **Percentile lies.** Average the per-pod p95s and compare with `histogram_quantile` over the summed buckets. Then change the bucket boundaries and watch the "p99" move. *Proves: histograms are a design decision.*
5. **Query library.** Write ten PromQL/LogQL/TraceQL queries you will reuse: error ratio by route, p95 by route, 5xx by upstream, pod restarts, CPU throttling ratio, SQS age of oldest message, DB connection saturation, slowest spans by service, logs for a trace ID, deploy markers. Save them in `platform/observability/queries/`. **flightdeck calls exactly these in M20.**
6. **Redaction.** Put an email address into a span attribute and a log line. Find it in Tempo and Loki. Add a Collector redaction processor and a logging filter. Prove it is gone. *Proves: telemetry is data subject to governance.*
7. **Silence.** Scale the Collector to zero. Dashboards go flat and no alert fires. Build the absence alert, repeat, and see it fire. *Proves: detection gaps.*
8. **Dashboard as code.** Delete a dashboard in the UI and restore it from Git in under five minutes. If you cannot, it was not code.
9. **Native fallback.** With Grafana off, answer "which target is returning 502s?" using only CloudWatch ALB metrics and Logs Insights. *Proves: you are not dependent on your own stack.*

## Project implementation

- OTel SDK in every Runway workload. Resource attributes `service.name`, `service.version` (image digest), `deployment.environment`, `runway.owner`. Log schema enforced by a shared logging config.
- `platform/modules/observability`: AMP workspace, IAM for remote-write (Pod Identity), S3 buckets with lifecycle rules for Loki/Tempo, Grafana OSS on the `platform` Karpenter node pool with Entra-ready OIDC login stubbed to IAM Identity Center for now. Also an OTel Collector agent DaemonSet + gateway Deployment with memory_limiter, k8sattributes, redaction, and tail sampling (keep 100% of errors and traces over 500 ms, sample 10% of the rest).
- `platform/observability/`: dashboards as code (Grafana JSON or Jsonnet/Grafonnet):
  - **User experience:** RED per route, SLI panels.
  - **Workload health:** restarts, throttling, memory versus limit, pool saturation, queue depth/age.
  - **Dependencies:** RDS, Redis, SQS, carrier-sim, AWS API errors.
  - **Platform:** nodes, Karpenter, add-on health, Collector/Loki/Tempo self-metrics.
  - **Cost & capacity:** placeholder panels, filled in by M12 and M16.
- **Deploy markers:** the `deploy` workflow writes a Grafana annotation with the digest and PR, so "did something just change?" is visible on every graph.
- Alert rules as code: availability and latency SLI alerts for `cargo-api` (simple thresholds for now; burn-rate alerts in M11), SQS age, absence-of-data, TLS certificate expiry < 21 days, and pod restart rate. Each alert carries labels `severity`, `tier` (placeholder for L1/L2 in M11) and `runbook_url`. **No alert without a runbook.**
- Native: VPC Flow Logs to S3 (queried with Athena on demand), ALB access logs, RDS Performance Insights (free tier), and CloudWatch as a Grafana data source.

## Engineering problems to hit deliberately

| Problem | How to cause it | What you should learn |
|---|---|---|
| Orphan traces | No propagation over SQS | Context must be carried explicitly across queues |
| Series explosion | High-cardinality label | Cost and query performance collapse together |
| PII in telemetry | Log the request body | Observability data is regulated data |
| Flapping alert | Threshold on a noisy gauge, no `for:` | Hysteresis and symptom-based design |
| Silent failure | Collector down | Absence alerts; meta-monitoring |
| Loki query times out | Querying without label selectors over 7 days | Label-indexed stores need label-first queries |

## Troubleshooting exercises

- **S16, "p95 latency up 6×, errors flat".** Inject 400 ms into carrier-sim. Detection must come *from your alert*, not from you knowing. Full write-up: traces → metrics → logs, in that order. Record every rejected hypothesis (database, throttling, recent deploy). Finding: "we had no dependency-latency alert".
- **S17, "Dashboards go quiet".** Two variants: the Collector crash-loops on memory, or AMP remote-write returns 403 after an IAM change. Full write-up. The finding is always about detection.
- **"Some requests are slow, only from one AZ".** Karpenter placed the Redis pod in another AZ from most callers. Find it from span attributes (`k8s.node.name`, zone) plus node labels.
- **"5xx on the ALB, the app says it served 200".** Target deregistration during a deploy (a regression of M09 lab 5). Prove it from ALB logs + deploy marker + pod lifecycle events.

## Deliverables

- [ ] OTel instrumentation in all workloads; the log schema documented in `platform/observability/README.md`
- [ ] `platform/modules/observability` + Collector, Loki, Tempo, Grafana, AMP as code
- [ ] Five dashboards and all alert rules as code; the query library (`platform/observability/queries/`)
- [ ] One runbook per alert in `docs/runbooks/` (linked from `runbook_url`)
- [ ] ADR-012
- [ ] Incident records for S16 and S17
- [ ] README screenshot: one full trace from ALB to RDS and SQS to worker

## Completion criteria

- Given a latency regression injected blind, you locate the responsible hop **in under 10 minutes** using traces first, then confirm with metrics and detail with logs.
- `make down && make up` loses **no** metric history, traces or logs within retention (AMP/S3), and dashboards come back identical from Git.
- Every alert has a runbook link, a symptom it detects, and a record of having been triggered in a drill.
- The absence-of-data alert fires within 10 minutes of the Collector stopping.
- You can answer the "which target is 502ing" question with Grafana switched off.
- The active series count for Runway is known, justified, and below a written budget (for example, < 50k series).

## Mentor checkpoints

- **Before you build:** your SLI definitions and alert list on one page. I will delete every alert that does not map to user pain or a runbook. Be ready to defend them.
- **During:** the S16 incident record, with the timeline written *during* the drill.
- **Questions I will ask:** Why alert on symptoms? How do you average p95 across pods correctly? What happens to observability during a region outage? What must never be in a span attribute?
- **The failure I will inject:** something that makes the telemetry *lie* rather than go silent. Think about clocks, sampling and missing labels.

## Resources

1. **OpenTelemetry docs:** *Concepts → Signals, Context propagation*; *Collector → Configuration, Deployment patterns*; *Python instrumentation*.
2. ***Observability Engineering* (Majors, Fong-Jones, Miranda):** ch. 1–7 (structured events, tracing), ch. 12–13 (SLOs, alerting on SLOs).
3. **Prometheus docs:** *Practices → Naming, Histograms and summaries, Alerting*; *Querying → Functions (rate, histogram_quantile)*.
4. **Google *SRE* book:** ch. 6 *Monitoring Distributed Systems* (the four golden signals). Free at sre.google.
5. **Grafana Loki docs:** *Get started → Labels (best practices, cardinality)*.

## Estimate vs actual

| | Hours | Cost |
|---|---|---|
| Planned | 40 | ~$40–70 (AMP samples, S3, platform node pool in session windows) |
| Actual | | |
| Variance note | | |
