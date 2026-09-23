# Section 25 — Final Production Readiness Checklist

There are two parts:

- **Part 1** is a production readiness review (PRR) for Runway, the kind a senior platform engineer runs before a service goes live or a platform is handed to other teams. Every item can be checked, and each one names the evidence path that proves it.
- **Part 2** is the final capability self-test: the requirement from the brief, and the decisions you must be able to make without the roadmap in front of you.

Run Part 1 in [M22](milestones/M22-hardening-capstone.md), and again in the final revision week (R9). Run Part 2 cold, timed, in R9. You're finished when Part 1 is fully ticked with evidence (or each open item is an accepted risk you've written down) and you can do Part 2 without notes.

**How to mark items:** ✅ done, with the evidence linked · ⚠️ accepted risk, with the reason written in `docs/security/threat-model.md` or the relevant ADR · ❌ not done. *An item with no evidence is ❌, whatever you remember doing.*

---

## Part 1 — Runway production readiness review

### 1. Architecture

- [ ] Current architecture diagram, from committed source, matches what is deployed → `docs/architecture/`
- [ ] Every component can name the requirement it serves or the failure it prevents → [Section 6](06-final-project-architecture.md), `docs/architecture/components.md`
- [ ] ADR-001 to ADR-027 exist, and each argues the rejected option properly → `docs/adr/`
- [ ] Current SDD for Runway in the enterprise template, including cost and timescale estimates → `docs/solution-designs/`
- [ ] Composite availability calculated and compared with measured availability, with the gap explained → `docs/continuity/availability.md`
- [ ] Single points of failure listed, each removed or accepted → `docs/continuity/spof.md`
- [ ] "What we deliberately did not build" table is current → `README.md`

### 2. Network

- [ ] CIDR plan with no overlap across AWS dev/prod, Azure and the datacentre (ADR-004) → `docs/architecture/network-plan.md`
- [ ] Workloads and data in private subnets only; nothing stateful has a public endpoint → Terraform + Config/Policy compliance report
- [ ] VPC endpoints for S3/ECR/STS declared in code; a conftest rule enforces them → `platform/policy/conftest/`
- [ ] Private endpoints + private DNS zones linked for every Azure PaaS dependency; a synthetic check asserts private resolution → `platform/observability/synthetics/`
- [ ] TLS everywhere; certificate expiry alerts at 30/14/7 days; automated renewal (ACM / cert-manager) → alert rules + `docs/runbooks/certificates.md`
- [ ] WAF rules tuned in detection mode before prevention; false-positive log reviewed → `docs/security/waf-tuning.md`
- [ ] Egress restricted to an allow-list; every outbound dependency documented → `docs/security/egress.md`
- [ ] `net-triage.sh` works from inside a pod in both clusters → `platform/scripts/net-triage.sh`

### 3. Identity and access

- [ ] **Zero static cloud credentials anywhere**: CI, pods, cross-cloud, portal → `docs/security/iam-model.md` + a secret-scan report
- [ ] GitHub → AWS and GitHub → Entra via OIDC with narrow `sub` conditions → `platform/live/*/iam/`
- [ ] Pod Identity (EKS), Workload Identity (AKS) and cross-cloud federation (ADR-017) all in code → `platform/modules/workload/`
- [ ] Workforce access through Entra ID → IAM Identity Center; no IAM users with console passwords → ADR-016 + account audit output
- [ ] Least-privilege roles derived empirically; no `*` actions on prod roles without a written justification → IAM Access Analyzer findings reviewed
- [ ] Break-glass access exists, is monitored, and was tested once → `docs/runbooks/break-glass.md`
- [ ] Separation of duties: nobody applies to prod from a laptop; prod environment approval required → GitHub environment settings screenshot + workflow

### 4. Infrastructure as code

- [ ] Nothing in `workloads-prod` / `runway-workloads` exists outside Terraform (drift report clean for 30 days) → `drift-detect` workflow history
- [ ] State: S3 with `use_lockfile` + versioning + encryption; Azure Blob with lease locking + soft delete; access restricted → ADR-005
- [ ] State recovery procedure executed at least once → `docs/runbooks/terraform-state-recovery.md`
- [ ] Modules versioned by tag, each with a README interface and a CHANGELOG → `platform/modules/*/`
- [ ] Gates: fmt, validate, tflint, `terraform test`, trivy/checkov, conftest, Infracost, all on PR → `.github/workflows/tf-plan-apply.yml`
- [ ] `prevent_destroy` / delete-protection on stateful resources; conftest blocks unreviewed deletes → `platform/policy/conftest/`
- [ ] `make down && make up` rebuilds dev from nothing, timed → README numbers table

### 5. Containers and supply chain

- [ ] All images multi-stage, non-root, read-only root filesystem where possible, pinned base digests → `workloads/*/Dockerfile`
- [ ] Trivy scan with a documented triage policy (severity × exploitability × fix available) → `docs/security/vulnerability-policy.md`
- [ ] SBOM generated for every image; cosign-signed; Kyverno rejects unsigned images at admission → `platform/policy/kyverno/`
- [ ] Base image rebuild triggered by a CVE, exercised once → incident or change record
- [ ] Images deployed by digest, never by mutable tag → Argo CD manifests

### 6. Kubernetes

- [ ] Every workload has correct liveness/readiness/startup probes, requests/limits derived from measurement, PDBs and topology spread → `platform/gitops/`
- [ ] Namespaces with ResourceQuota and LimitRange; RBAC per team; default-deny NetworkPolicy with explicit allows → `platform/modules/k8s-baseline/`
- [ ] Pod Security Standards `restricted` enforced (or documented exceptions) → Kyverno/PSA config
- [ ] Karpenter (EKS) and the AKS autoscaler configured with limits and consolidation; spot only for interruption-tolerant workloads → ADR-010
- [ ] A cluster upgrade performed on both EKS and AKS with traffic flowing; deprecated APIs checked beforehand → `docs/runbooks/cluster-upgrade.md`
- [ ] The six pod failure states triage runbook, used in a real incident → `docs/runbooks/pod-failure-triage.md`
- [ ] Argo CD self-heal on; no manual changes survive; OutOfSync alerting → ADR-018

### 7. CI/CD and change

- [ ] Build once, promote by digest across dev → prod; no rebuilds per environment → `.github/workflows/`
- [ ] Automated test pyramid: unit, contract (OpenAPI), integration (testcontainers), smoke, k6 performance gate → pipeline run logs
- [ ] Deploy verification checks real health (readiness + SLI), not just "job succeeded" (the S10 fix) → `deploy` workflow
- [ ] Rollback measured in seconds and recorded → README numbers table
- [ ] Expand-contract migration pattern documented and used → `docs/runbooks/database-migrations.md`
- [ ] Change records generated automatically; standard/normal/emergency types in use; freeze windows in the calendar → `docs/changes/`
- [ ] Change failure rate tracked monthly; every failed change has a PIR → `docs/operations/`

### 8. Secrets

- [ ] All secrets in Secrets Manager / Key Vault, delivered by External Secrets Operator; none in Git, images, manifests or Terraform variables → ADR-011 + secret-scan
- [ ] Secrets that Terraform state must contain are identified, and state access restricted accordingly → ADR-005
- [ ] Rotation procedure executed for every secret, timed; schedule written down → `docs/runbooks/secret-rotation.md`
- [ ] Key Vault soft delete + purge protection; Secrets Manager recovery window → Terraform
- [ ] Secret scanning (push protection) on all repos → repository settings

### 9. Security

- [ ] STRIDE threat model with a controls matrix; every threat has a control or a written accepted risk → `docs/security/threat-model.md`
- [ ] Leaked-credential drill (S27) run end to end: contain, rotate, assess, prevent → `docs/incidents/`
- [ ] Security logging: CloudTrail (org trail to the `security` account), Azure Activity Log, K8s audit logs; retained and queryable → ADR-012
- [ ] AI-specific threats (OWASP Top 10 for LLM Applications) assessed → `docs/security/ai-threat-model.md`
- [ ] Vulnerability SLA per severity defined and currently met → `docs/security/vulnerability-policy.md`

### 10. Governance

- [ ] SCPs: region restriction, root-user denial, protection of the security account's logging → `platform/policy/scp/`
- [ ] Azure Policy initiative: required tags, allowed locations, no public IPs on workload subnets, private endpoints for PaaS → `platform/policy/azure/`
- [ ] Rollout audit-first → deny for each policy, recorded; exemption process with expiry dates → ADR-019
- [ ] Datacentre resources (Arc/SSM) governed by the same tag/policy set → [M21](milestones/M21-hybrid-multicloud.md) evidence
- [ ] EOL register current: K8s versions, Postgres major, runtimes, providers, base images, with dates and plans → `docs/lifecycle/eol-register.md`

### 11. Observability

- [ ] OTel traces across portal → cargo-api → SQS → worker → ai-gateway → model, with context propagated across the queue and across clouds → a trace screenshot in README
- [ ] Structured JSON logs with `trace_id`, `request_id` and `service`; PII redaction verified → `workloads/*/logging`
- [ ] Standard dashboards per service (user experience, system health, dependencies, cost/capacity) as code → `platform/observability/dashboards/`
- [ ] Native signals (CloudWatch, Azure Monitor) available as the DR fallback when the Grafana stack is down → `docs/runbooks/observability-fallback.md`
- [ ] Cardinality and ingest-cost controls in place; retention defined per signal → ADR-012

### 12. Alerting and event management

- [ ] Every alert is symptom-based, or a justified cause-level early warning → `platform/observability/alerts/`
- [ ] Every alert has `tier` (L1/L2), `severity`, `service`, `runbook_url`; CI lint enforces it → alert lint workflow
- [ ] `ems-bridge` creates and deduplicates incident records from alerts → `workloads/ems-bridge/`
- [ ] Absence-of-data alerts for every critical telemetry stream (S17) → alert rules
- [ ] Every alert reviewed for usefulness in the last monthly review; noisy alerts fixed or deleted → `docs/operations/review-*.md`

### 13. SLOs and service levels

- [ ] SLOs for `cargo-api` (availability + latency), `runway-portal`, `ai-gateway` (including degraded path) with multi-window burn-rate alerts → `platform/observability/slo/`
- [ ] Internal SLA/OLA written, looser than the SLO; severity matrix and response targets published → ADR-013
- [ ] Error budget used in at least one recorded decision → `docs/operations/`
- [ ] 30-day SLO attainment reported in the README → README numbers table

### 14. Capacity and performance

- [ ] Measured maximum sustainable rate and the resource that limits it → `docs/capacity/`
- [ ] Capacity plan: headroom policy, growth trigger, quotas that would need raising (vCPU, IPs, TPM/RPM) → `docs/capacity/capacity-plan.md`
- [ ] Every autoscaler maximum checked against downstream limits (DB connections, AI TPM) → capacity plan
- [ ] Graceful degradation: load shedding with 503 + `Retry-After`; timeouts and circuit breakers on every dependency → golden-path client wrapper

### 15. Backup, continuity and DR

- [ ] RTO/RPO per tier agreed and **measured** in drills (PITR and full rebuild), executed at least twice → `docs/continuity/`
- [ ] Automated `backup-verify` Lambda restores to scratch and asserts integrity weekly → `workloads/` or `platform/`
- [ ] Pilot-light rebuild in a second AWS region from Terraform, timed → ADR-014 + drill record
- [ ] Dependency-ordered DR runbook, executed by someone else or by you cold after a month → `docs/runbooks/disaster-recovery.md`
- [ ] What is *not* covered is written down (cross-cloud DR rejected, observability blast radius) → ADR-014

### 16. Cost

- [ ] Budgets and anomaly detection in both clouds; hard alarm at $120/month → Terraform
- [ ] Tagging taxonomy enforced by policy; >95% of spend allocated → FOCUS report
- [ ] Unit costs tracked: cost per 1k requests, cost per AI summary, cost per environment → `docs/cost/`
- [ ] Monthly cost notes for every month of the roadmap, plus one full anomaly investigation (S29) → `docs/cost/`
- [ ] Showback per team visible in the portal → `runway-portal`
- [ ] Commitment (Savings Plans / Reservations) analysis written, even though the answer at this scale is "no" → `docs/cost/commitments.md`

### 17. Documentation and catalogue

- [ ] Every component has a `catalog-info.yaml`; every dataset has a data catalogue entry (owner, classification, PII, retention, backup, RPO) → `catalog/`
- [ ] Runbooks: one per alert, plus DR, restore, state recovery, cluster upgrade, secret rotation, onboarding, break-glass → `docs/runbooks/`
- [ ] At least one runbook executed successfully by someone else without asking you a question → note in the runbook
- [ ] Onboarding guide followed by a stranger (or by you cold) → `docs/platform/onboarding-guide.md`
- [ ] Estimates against actuals recorded for every milestone → `docs/estimates/`

### 18. Operations

- [ ] 25+ incident records, 6+ problem records, 3+ PIRs, all from real drills or incidents → `docs/incidents/`, `docs/problems/`, `docs/operations/`
- [ ] Monthly operational reviews from M11 onwards, none missing → `docs/operations/review-*.md`
- [ ] Game days run monthly from M11; runbook defects fixed the same day → game-day notes
- [ ] Toil list with at least three items automated away → monthly reviews

### 19. Platform and self-service

- [ ] Golden path: template repo + `workload.yaml` + `workload` module + reusable workflows + Argo CD + standard telemetry/SLOs/policy → [M17](milestones/M17-golden-paths.md)
- [ ] `hello-svc` onboarded to **both** clouds without touching platform code; time-to-first-deploy measured → README numbers table
- [ ] Portal requests (service, environment, sandbox account) all produce PRs; none mutate cloud directly → ADR-022
- [ ] A breaking module release shipped with a migration guide; consumers migrated → `platform/modules/workload/CHANGELOG.md`
- [ ] What the path does not support, and the escape hatches → `docs/platform/`

### 20. AI

- [ ] Azure OpenAI via private endpoint + managed identity; Bedrock via cross-cloud federation; no API keys anywhere → ADR-023
- [ ] Routing primary → fallback → non-AI degraded, exercised with the primary disabled; fallback rate on a dashboard → `workloads/ai-gateway/`
- [ ] Per-team quotas and rate limits; retries honour `retry-after` with backoff + jitter (S31 fixed) → ai-gateway config
- [ ] Token metering and cost attribution per team; cost-rate-of-change alert (S32) → `platform/observability/`
- [ ] Structured outputs validated; fail closed on validation failure → ai-gateway / insights-worker tests
- [ ] Eval set runs as a pipeline gate; prompt and model versions recorded on every request's telemetry → `flightdeck/evals/`, `workloads/insights-worker/`
- [ ] Flightdeck is read-only through three independent mechanisms: identity, tool surface (CI test), audit → ADR-025
- [ ] Prompt injection through tool results tested (S33); output is advisory and never feeds automation → `docs/security/ai-threat-model.md`
- [ ] Time-to-first-hypothesis measured with and without Flightdeck, including the cases where it was wrong → `flightdeck/evals/results.md`
- [ ] Managed vs self-hosted inference decided with measured numbers → ADR-024

---

## Part 2 — The final capability self-test

The brief's final capability test:

> *"We need to deploy a secure, highly available application with an AI component that serves thousands of users."*

**The exercise.** In R9, take a fresh sheet of paper and set a **3-hour timer**. Don't look at the roadmap or Runway. Write a Solution Design Document using the template in [Appendix A2](appendix/A2-documentation-standards.md) for a *new* system, not Runway. Examples: a crew rostering assistant, a customer baggage-status service, or a supplier-invoice classifier. Pick the cloud yourself and justify it. State your assumptions (users, requests per second, data classification, residency, budget) and design against them.

Then review it against Part 1, and have someone ask you "why?" at every box.

### The decisions you must make independently

For each one, you must be able to state the decision, at least one rejected option with the reason, and how you'd verify it in production.

| # | Area | Decisions you must make | Where you learned it |
|:--:|---|---|---|
| 1 | **Architecture** | Stateless tiers, where state lives, sync vs async boundaries, the AI call path, failure domains, composite SLA | [M03](milestones/M03-cloud-architecture.md), [Appendix A1](appendix/A1-cloud-architecture-concepts.md) |
| 2 | **Cloud services** | Which cloud and which services, and why managed vs self-run for each (DB, cache, queue, K8s vs serverless containers vs functions, model endpoint) | ADR-002, ADR-008, ADR-015, ADR-024 |
| 3 | **Network design** | CIDR plan, AZ layout, public/private/data subnets, egress (NAT vs endpoints), private endpoints and DNS, edge (DNS, CDN, WAF, LB), hybrid connectivity if required | [M04](milestones/M04-aws-core.md), [M13](milestones/M13-azure-core.md), [M21](milestones/M21-hybrid-multicloud.md) |
| 4 | **IAM model** | Account/subscription structure, workforce identity and SSO, CI federation, workload identity, cross-cloud federation, break-glass, separation of duties | ADR-003, ADR-016, ADR-017 |
| 5 | **Terraform structure** | Module boundaries, live-root layout, state per component and environment, versioning, gates, drift handling | ADR-005, [M05](milestones/M05-terraform.md) |
| 6 | **Container strategy** | Base images, build, scanning, SBOM, signing, registry, promotion by digest | ADR-006 |
| 7 | **Kubernetes architecture** | Cluster count and placement, node pools, autoscaling, tenancy (namespaces, quotas, RBAC, NetworkPolicy), traffic entry, upgrades, or why not Kubernetes at all | ADR-008/009/010, ADR-018 |
| 8 | **CI/CD** | Pipeline stages, test pyramid, environments and approvals, deployment strategy, rollback, push vs GitOps, change records | ADR-007, ADR-013, ADR-018 |
| 9 | **Secrets management** | Stores, delivery mechanism, rotation, what ends up in state, blast radius of each secret | ADR-011 |
| 10 | **Security controls** | Threat model, defence in depth per layer, WAF, egress, supply chain, AI-specific controls, audit logging | ADR-019, [M15](milestones/M15-security-governance.md), [M22](milestones/M22-hardening-capstone.md) |
| 11 | **Monitoring** | SLIs and SLOs, standard dashboards, telemetry pipeline, native vs central stack | ADR-012 |
| 12 | **Logging** | Schema, correlation IDs, PII redaction, retention, ingest cost control, security logs | [M10](milestones/M10-observability.md) |
| 13 | **Alerting** | Symptom vs cause alerts, burn-rate thresholds, L1/L2 tiers, severities, routing, runbook links, absence-of-data | ADR-013 |
| 14 | **Scaling** | Expected load via Little's Law, the likely saturating resource, autoscaler signals and limits against downstream limits, AI TPM/RPM capacity, quotas to raise in advance | [M12](milestones/M12-capacity-ha-dr.md), [M19](milestones/M19-ai-infrastructure.md) |
| 15 | **Reliability strategy** | Timeouts, retry budgets, circuit breakers, load shedding, graceful degradation, AI fallback chain, PDBs and topology spread | [M11](milestones/M11-sre-operations.md), ADR-023 |
| 16 | **Backup** | What is backed up, how often, immutability, verification, what is *not* covered | [M12](milestones/M12-capacity-ha-dr.md) |
| 17 | **Disaster recovery** | RTO/RPO tiers with business justification, DR pattern and its cost, the rebuild order, how and how often it's tested | ADR-014 |
| 18 | **Cost controls** | Cost model before building, tagging and allocation, budgets and anomaly alerts, unit costs, AI token budgets, commitment strategy | ADR-020, [M16](milestones/M16-finops.md) |
| 19 | **Operational procedures** | Severity matrix, incident/problem/change processes, on-call and escalation, runbooks, operational review cadence, EOL management, data catalogue | [Section 10](10-failure-and-troubleshooting.md), [Appendix A2](appendix/A2-documentation-standards.md) |
| 20 | **Delivery guidance** | Estimate the build in weeks and the run cost per month, name the risks, and propose a phased plan a project manager could track | `docs/estimates/`, SDD template |

### And then troubleshoot it

A design isn't finished until you can say how it fails. For your capstone design, write out:

- **The five most likely incidents** in its first three months, and for each: what detects it, what the L1 action is, and what L2 would investigate first.
- **One end-to-end request trace**, from user to DNS to edge to service to model and back, naming the failure point at each hop and the evidence that would show it.
- **One scenario from each column of the S01–S36 catalogue** ([Section 10](10-failure-and-troubleshooting.md)) mapped onto your new design: network, identity, Kubernetes, data, cost, AI.

**You're done when** you can do all of this in three hours without notes, defend every box against "why not X?", and point to a Runway artefact that proves you've actually done each thing at least once rather than only read about it. That combination is what makes someone a senior cloud platform engineer rather than someone who has studied cloud.

← [Back to the index](README.md)
