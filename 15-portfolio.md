# Step 16 — The Final Portfolio

## 1. What the repository must prove

Not *"I can build a cloud system."* Plenty of repositories claim that.

> **"I run a production platform. Here is what broke, how I found it, what it cost, how long recovery took, and how someone else uses it without me."**

Every item below exists to support that sentence. If something in the repository does not, delete it — a lean repository that is entirely evidence beats a large one where a reviewer must hunt.

## 2. Contents

### Root
| File | What good looks like |
|---|---|
| `README.md` | **The most important file.** See §3 |
| `Makefile` | `make up`, `make down`, `make test`, `make scan`, `make loadtest` |
| `docs/competency.md` | Your honest self-assessment matrix with dates |

### `docs/architecture/`
System diagram (Mermaid source, committed, plus rendered) · network plan with subnet justification · data flow and trust boundaries · tagging standard · `manual-build-log.md` from Milestone 3, kept deliberately as the argument for IaC.

### `docs/adr/`
At least eight, indexed. ADR-001 cloud choice · ADR-002 AKS vs Container Apps · ADR-003 base images · ADR-004 environment separation · ADR-005 deployment strategy · ADR-006 managed inference vs self-hosted · ADR-007 read-only agent design · ADR-008 portability assessment. **Each must argue the rejected option properly.** An ADR with a one-line "we rejected X because it is worse" is not an ADR.

### `platform/`
`modules/` — versioned, each with a README documenting its interface · `environments/dev|prod/` — thin root modules · `policy/` — definitions, initiative, assignments · `observability/` — dashboards, alert rules, SLO definitions as code · `loadtest/` — k6 scripts · `scripts/` — `net-triage.sh`, `inject-fault.sh`, the cost reporter, and one PowerShell tool.

### `.github/workflows/`
Reusable: `build-and-push`, `deploy`, `tf-plan-apply`, `drift-detect`. Plus the thin consumer workflows, so a reviewer can see how little a workload has to write.

### `workloads/`
`cargo-api` (Python) · `carrier-sim` · `runway-portal` (React + Node/TypeScript). Hardened Dockerfiles, Kubernetes manifests, OpenAPI contract. **Plus a link to the separate `hello-svc` repository** — the cross-repo onboarding proof.

### `flightdeck/`
MCP server with documented tool schemas · the bounded agent · `evals/` with replayed incidents and scored results · the test asserting no write-capable tool is registered.

### `docs/runbooks/`
One per alert, plus: incident response and severity matrix · pod failure triage (the six states) · network triage (the four TCP failure modes) · database restore · disaster recovery with measured timings · Terraform state recovery · AKS upgrade · secret rotation · workload onboarding.

**The test for a runbook:** someone who is not you, at 3 a.m., can follow it without asking a question. Prove it — hand one to someone and watch.

### `docs/incidents/` and `docs/problems/`
Ten-plus incident records, four-plus RCAs. **These are the differentiator.** Timelines with timestamps, hypotheses you rejected, and a "what would have detected this sooner" section. Do not sanitise the wrong turns — they are the evidence of judgement.

### `docs/capacity/`, `docs/cost/`, `docs/security/`, `docs/operations/`
Load test reports showing the bottleneck progression and a capacity plan · monthly cost notes, cost-per-service, and the full anomaly investigation with before/after numbers · threat model with a controls matrix including accepted risks, and the leaked-credential drill · monthly operational reviews.

### `docs/platform/`
`onboarding-guide.md` written for a stranger, and the golden-path design document with what it deliberately does not support.

## 3. The README

The single highest-leverage file. Most reviewers spend ninety seconds. Structure it so the ninety seconds land on evidence:

1. **One paragraph:** what Runway is and who it is for.
2. **The architecture diagram**, rendered inline.
3. **The numbers table** — the section that makes a reviewer keep reading:

| | |
|---|---|
| Rollback time | measured, in seconds |
| RTO / RPO | measured in a drill, not a target |
| Max sustained request rate | and the resource that limits it |
| Time to onboard a new service | measured with `hello-svc` |
| Monthly cost | and the dominant line item |
| Incidents worked | and mean time to mitigate |
| SLO attainment | over the last 30 days |
| Policy violations blocked | count |
| AI: time to first hypothesis | before and after Flightdeck |

4. **What I operate** — links to the runbooks, incidents and operational reviews. Put this *above* "what I built".
5. **What I built** — components, with links.
6. **What I deliberately did not build, and why** — the exclusions table from [Step 6 §4](05-architecture.md#4-deliberately-excluded-and-why). This section makes senior reviewers trust the rest of the document.
7. **How to run it** — `make up`, and an honest note about the cost.

## 4. Presenting it

**On a CV:** one line per claim, each with a number. *"Built and operated an Azure platform on AKS with Terraform-managed infrastructure; measured RTO of 18 minutes in DR drills; reduced new-service onboarding from 3 hours to 20 minutes; identified and resolved a 40% cost anomaly."*

**In a cover letter or application note:** lead with the operational half, because that is the half the JD's role definition is made of and the half other applicants will not have.

**In an interview:** offer the incident records first. *"Would it be more useful to walk through the architecture, or through an incident where my first two hypotheses were wrong?"* Almost every experienced interviewer picks the second, and it is the conversation you are best prepared for.

**What not to do:** do not demo the AI agent first. It is the most eye-catching component and the least relevant to the job's core. Lead with operations; let Flightdeck be the thing they discover at the end and remember afterwards.

## 5. The honesty rule

Every number in the repository must be one you produced, and every claim must be one you can reproduce live. An interviewer who finds one inflated number discounts everything else, and reasonably so.

Where something is simulated rather than real — the region-loss scenario you documented but did not execute, the WAF you ran for two days rather than permanently — **say so in the document itself**. "I did not test this because it would cost $900/month, and here is what I would do" is a credible engineer's sentence. A claimed multi-region failover that you cannot demonstrate is a disqualifying one.

→ Next: [Step 17 — Final career skill map](16-career-skill-map.md)
