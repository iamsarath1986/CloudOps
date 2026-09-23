# Appendix A3 — Portfolio and Interview

## 1. What the repository must prove

Not *"I can build a cloud system."* Plenty of repositories claim that.

> **"I design and operate a governed platform across AWS and Azure. Here is what broke, how I found it, what it cost, how long recovery took, what I changed so it doesn't recur, and how other teams use it without me."**

Everything in the Runway repository exists to support that sentence. If something doesn't, delete it. A lean repository that is entirely evidence beats a large one where the reviewer has to dig.

## 2. What must exist, and what good looks like

| Location | Contents | What good looks like |
|---|---|---|
| `README.md` | The front page | See §3. Most reviewers give it about ninety seconds |
| `docs/solution-designs/` | SDD for Runway v0 (M03), v1 (M12), final (M22) | NFRs mapped to design choices, a cost estimate, risks, and the options rejected |
| `docs/adr/` | ADR-001 … ADR-027 | Each argues the *rejected* option properly. A one-line "X is worse" isn't an ADR |
| `docs/architecture/` | Diagrams as code, network/CIDR plan, trust boundaries, tagging standard, `manual-build-log.md` | The manual build log is kept on purpose, as the argument for IaC |
| `platform/modules/` | Versioned modules, each with an interface README | A consumer can use one without reading its internals |
| `platform/live/` | Thin roots per cloud/account/env/component | All logic lives in modules |
| `platform/policy/` | SCPs, Azure Policy, Kyverno, conftest | Each policy has a test, and a record of a real deploy it blocked |
| `platform/gitops/` | Argo CD app-of-apps for both clusters | One Git change reaches both clouds |
| `platform/observability/` | Dashboards, alert rules and SLOs as code | Every alert has `tier`, `severity` and a runbook link |
| `.github/workflows/` | Reusable workflows + thin callers | A consumer workflow is about ten lines |
| `workloads/`, `flightdeck/` | Services with hardened Dockerfiles, OpenAPI contracts, tests | Plus a link to the separate `hello-svc` repo, the cross-repo onboarding proof |
| `catalog/` | `catalog-info.yaml` per component; `data/` catalogue entries | The portal renders it; every dataset has an owner, classification and RPO |
| `docs/runbooks/` | One per alert, plus DR, restore, upgrade, rotation, onboarding | A stranger can follow one at 3 a.m. without asking a question |
| `docs/incidents/`, `docs/problems/`, `docs/changes/` | 15+ incidents, 5+ RCAs, change records incl. a failed change + PIR | **The differentiator.** Wrong turns kept in. Timelines written during the incident |
| `docs/operations/` | Monthly operational reviews | A year of them outweighs any diagram |
| `docs/capacity/`, `docs/continuity/` | Load-test reports, capacity plan, DR drill records | Measured numbers only, twice where it says twice |
| `docs/cost/` | Monthly notes, FOCUS-normalised reports, the anomaly investigation | Before/after numbers, on your own money |
| `docs/security/` | Threat model + controls matrix, the leaked-credential drill, AI agent threat model | Accepted risks stated and defended |
| `docs/lifecycle/eol-register.md` | Runtimes, K8s versions, Postgres major, providers, base images | Dates, owners, plans, and one completed replacement |
| `docs/estimates/` | Estimate vs actual per milestone | Shows you can guide PMs on cost and timescales, and that your estimates improved |
| `docs/competency.md` | Dated L0–L5 self-assessments ([Section 24](../24-competency-matrix.md)) | Includes honest downgrades |

## 3. The README

Structure it so the first ninety seconds land on evidence:

1. **One paragraph:** what Runway is, who it's for, and the fact that it spans AWS and Azure under one identity, delivery, policy and telemetry model.
2. **The architecture diagram**, rendered inline.
3. **The numbers table.** This is the section that keeps a reviewer reading:

| Measure | Value (measured, with date and method) |
|---|---|
| Time to onboard a new service (golden path, `hello-svc`) | per cloud |
| Rollback time | seconds |
| Change failure rate / changes per month | from change records |
| RTO / RPO | measured in two drills, not a target |
| Pilot-light region rebuild time | minutes |
| Max sustained request rate | and the resource that limits it |
| SLO attainment (last 30 days) | per service |
| Incidents worked · mean time to mitigate | from records |
| Monthly cost (AWS / Azure) · cost per 1k requests · cost per AI summary | FOCUS report |
| Policy violations blocked | count, by enforcement point |
| Static credentials in the system | **0**, and how you know |
| AI: time to first hypothesis | before/after Flightdeck, with the cases where it was wrong |

4. **What I operate:** links to runbooks, incidents, changes and operational reviews. Put this *above* "what I built".
5. **What I built:** components, with links.
6. **What I deliberately didn't build, and why:** service mesh, active-active, cross-cloud DR, Kafka, training. This section is what makes senior reviewers trust the rest.
7. **How to run it:** `make up`, what it costs per hour, and `make down`.

## 4. Presenting it

**On a CV:** one line per claim, each with a number. For example: *"Designed and operated a multi-account AWS and Azure platform (EKS, AKS, Argo CD, Terraform) with zero static credentials; reduced new-service onboarding to N minutes on either cloud via a golden path and self-service portal; measured RTO of N min across two DR drills; found and fixed a 40% cost anomaly."*

**In an application note:** lead with operations and platform, because those are the halves of the JD other applicants won't have.

**In an interview:** offer the incident records first. *"Would it be more useful to walk through the architecture, or through an incident where my first two hypotheses were wrong?"* Almost every experienced interviewer picks the second, and it's the conversation you're best prepared for.

**For design questions:** use the capstone method from [Section 25](../25-production-readiness-checklist.md): clarify requirements and NFRs → sketch → identity → network → runtime → data → delivery → observability → security → reliability → cost → operations → what you'd deliberately leave out.

**What not to do:** don't demo the AI agent first. It's the most eye-catching component and the least central to the job. Lead with operations and the golden path, and let Flightdeck be what they discover at the end and remember afterwards.

## 5. Twenty-five questions

If you can answer all twenty-five without notes, at a whiteboard, you're ready. Practise out loud in every revision week.

**Networking and identity**
1. The app on EKS can't reach RDS. What do you check, in what order, and what does each result rule out?
2. Explain connection refused vs connection timeout vs reset, and what each tells you about where the packet died.
3. Walk me through AWS IAM policy evaluation when an SCP, a permissions boundary and an identity policy all apply.
4. How does a pod on EKS get credentials to call Azure OpenAI, with no stored secret? Draw the token exchange.
5. Why did your AKS app resolve a private endpoint to a public IP, and how did you prove it?

**Infrastructure and delivery**
6. What is Terraform state, what are three ways it gets corrupted, and how did you recover from one?
7. How is your Terraform laid out, and why not workspaces?
8. Explain build-once-promote-by-digest and what specifically breaks without it.
9. Why did you move from push pipelines to Argo CD, and what did you give up?
10. Walk me through what happens between `kubectl apply` and a running pod.
11. Why can CPU *limits* make a service slower?
12. How did you replace a retired component (ingress-nginx) and how did you manage that change?

**Operations**
13. Your p95 latency went up six times while errors stayed flat. Walk me through the investigation.
14. The difference between an incident, a problem and a change, with an example of each from your own records.
15. Why do you mitigate before you diagnose? When wouldn't you?
16. How do your alerts decide between L1 and L2, and what happens after an alert fires?
17. What are your RTO and RPO, how did you measure them, and what was wrong the first time?
18. What's the maximum request rate your system sustains, and what limits it?

**Governance, cost, platform**
19. Show me a policy that blocked a real deployment. Where else could you have enforced it, and why there?
20. Your bill jumped 40%. How do you find out why, across two clouds?
21. What does your golden path deliberately *not* support, and what's the escape hatch?
22. How long does onboarding take on each cloud, and how do you know?

**AI**
23. How do you control cost, latency and failure behaviour of an AI dependency? What happens when Azure OpenAI returns 429?
24. How do you *guarantee* your agent can't change infrastructure? Name three independent mechanisms.

**Judgement**
25. What would you change if you rebuilt Runway from scratch, and what would you keep exactly as it is?

**Question 25 decides senior versus mid-level.** Have three specific changes and one thing you'd defend unchanged. ADR-027 is where you write that answer down.

## 6. The honesty rule

Every number in the repository must be one you produced, and every claim one you can reproduce live. If an interviewer finds one inflated number, they'll discount everything else, and they'd be right to.

Where something is simulated, say so in the document itself. That includes the VPN window that ran for two days, the Multi-AZ database enabled only for the HA drill, the "large footprint" that is six accounts, and "EMS" modelled with Alertmanager and a webhook. *"I ran the WAF for two days rather than permanently because it would have cost $N/month; here is what I measured"* is a credible engineer's sentence. A claimed capability you can't demonstrate is disqualifying.

← [Back to the index](../README.md)
