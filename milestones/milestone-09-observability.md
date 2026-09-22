# Milestone 9 — Observability

*Phase 8 · ~3 weeks · Prerequisite: Milestone 8*

**Learning topics**
* What each signal is *for*: logs for detail, metrics for trend and alerting, traces for causality — and why traces lead in a distributed system
* OpenTelemetry: spans, attributes, context propagation, exporters; why the standard protects you from a backend change
* Structured logging with correlation IDs
* **RED** for request-driven services, **USE** for resources
* Cardinality — why a shipment ID as a metric label creates millions of series and a memorable bill
* Percentiles vs averages; p99 is a real customer
* SLIs, SLOs, error budgets, burn-rate alerting
* **Alerting on symptoms, not causes** — cause-based alerts produce noise and get muted
* Log sampling, retention and ingestion caps as cost control; what must never be logged

**Build**
OpenTelemetry across all workloads. Structured JSON logs carrying `trace_id` and `request_id`. Four dashboards, **as code** in `platform/observability/`: user experience, system health, dependencies, cost and capacity. Two SLOs — `cargo-api` availability and p95 latency — with multi-window burn-rate alerts. Four to six symptom-based alert rules, each with a runbook link in its description.

**Cloud resources**
Log Analytics workspace with a daily ingestion cap · Managed Prometheus · Managed Grafana · alert rules and action groups. All in Terraform.

**DevOps work**
Dashboards and alert rules committed as code and deployed by the pipeline. **Never click a dashboard into existence again.** A saved KQL query library — ten queries you will actually reuse, which Flightdeck later calls verbatim in Milestone 14.

**Operations exercise**
Break context propagation between API and worker; watch the trace fragment into orphans; fix it. Add a high-cardinality label and watch series count and cost climb; remove it and write the rule into the dashboard description. Exceed the daily ingestion cap deliberately and watch logging stop. Delete a dashboard and rebuild it from code in five minutes — if you cannot, it was not code.

**Security considerations**
Send a sensitive field into a span attribute, find it in your telemetry backend, and implement redaction. A trace attribute containing customer data is a data incident, not a detail.

**Expected outcome**
You can answer an unanticipated question about production without shipping new code, and every alert you own has a runbook.

**Evidence of mastery**
Given a latency regression, show how you find the cause with traces, confirm with metrics, then get detail from logs — in that order. Explain SLI vs SLO vs SLA. Justify every alert: what symptom, what runbook, has it ever fired usefully.
