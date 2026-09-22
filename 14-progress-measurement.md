# Step 15 — Progress Measurement

## 1. The mastery levels

A tutorial completed is not a skill acquired. These five levels are cumulative and each is defined by something you **do in front of someone**, not by something you feel.

| Level | Name | The test |
|:--:|---|---|
| **1** | **Explain** | You can describe what it is, what problem it solves, and how it works, to someone who does not know — without notes, without analogies that hide the mechanism |
| **2** | **Build** | You can make it work from scratch, unaided, for a straightforward case |
| **3** | **Troubleshoot** | When it misbehaves you can form a hypothesis, test it, and find the cause — including when your first two hypotheses are wrong |
| **4** | **Design** | You can choose between options, justify the choice on cost, reliability and operability, and say what you deliberately rejected |
| **5** | **Operate** | You have run it over time: upgraded it, recovered it from failure, paid for it, and changed it under pressure without breaking it |

**Level 3 is the honest bar for "I know this."** Levels 1 and 2 are what a weekend produces. Level 5 is what separates a candidate from a colleague, and it cannot be acquired quickly — only by keeping the environment alive across months, which is why the roadmap insists on it.

## 2. The competency matrix

| Technology | L1 Explain | L2 Build | L3 Troubleshoot | L4 Design | L5 Operate |
|---|---|---|---|---|---|
| **Cloud networking** | Describe VNet/subnet/NSG/route table/private endpoint and what each controls | Build a working multi-subnet topology with a private database | Trace a failing request through DNS, LB, ingress, NSG, route and endpoint, and name the failing hop | Design hub-and-spoke with egress control and hybrid DNS; justify each boundary | Change network config in a live environment without an outage; recover from a private-DNS break under time pressure |
| **IAM** | Explain authn vs authz, RBAC scope inheritance, managed identity vs service principal | Assign roles, create a managed identity, set up OIDC federation | Diagnose a failed deploy from the effective-permission chain alone | Design least-privilege roles for a platform with multiple consumer teams | Rotate credentials, tighten an over-broad role without breaking consumers, run the leaked-credential drill |
| **Terraform** | Explain state, the plan/apply cycle, providers, modules | Write modules with remote state and deploy two environments | Read a destructive plan before applying; recover corrupted state; reconcile drift | Design a module interface and versioning policy for consumers you do not control | Run Terraform through a pipeline across months; ship a breaking module version and migrate consumers |
| **Docker** | Explain namespaces, cgroups, layers, and why a container is not a VM | Write a multi-stage, non-root, healthchecked image | Diagnose a container that will not start, is OOMKilled, or ignores `SIGTERM` | Choose base images and a scanning/triage policy; decide what belongs in an image | Maintain images over time: rebuild on base CVEs, manage digest pinning, keep build times sane |
| **Kubernetes** | Explain the reconciliation loop, control plane, scheduling, probes | Deploy a workload with probes, limits, HPA, ingress and network policy | Diagnose all six pod failure states in under five minutes from `kubectl` alone | Design namespace tenancy, resource governance, node pool strategy and disruption budgets | Upgrade a cluster with traffic flowing; handle node loss and spot eviction; resolve a cluster incident |
| **CI/CD** | Explain build-once-promote, gates, environments, rollback | Build a pipeline that builds, scans, deploys and rolls back | Diagnose a failure at any stage, including a deploy that reports green while the service is down | Design promotion, approval and rollback strategy; justify every gate | Operate pipelines other repositories depend on; add a mandatory stage without touching consumer repos |
| **Observability** | Explain logs vs metrics vs traces, RED/USE, cardinality, SLI/SLO | Instrument a service end to end and build dashboards and alerts as code | Diagnose a latency regression from traces, confirm with metrics, detail with logs | Choose signals, thresholds and retention; design alerting that does not get muted | Run it across months: tune thresholds, control ingestion cost, prove every alert has been useful |
| **Python automation** | Explain when a script becomes a tool needing tests and packaging | Write cloud automation using SDKs with retries and pagination | Debug throttling, partial failures and pagination bugs in your own tooling | Design tooling interfaces other engineers use; decide what to automate and what to leave manual | Maintain tooling others depend on: versioning, error handling good enough for 3 a.m. |
| **Security** | Explain least privilege, defence in depth, the trust boundaries in your system | Implement network restriction, secret management, image scanning, pod security | Diagnose a WAF 403, a broken egress rule, a failed token exchange | Threat-model an architecture and map controls to threats, including accepted risks | Run the credential-leak drill; rotate everything; keep vulnerability triage current over months |
| **Platform engineering** | Explain golden paths, paved road vs guardrail, platform-as-product | Build a reusable module and a reusable workflow | Debug a consumer's onboarding failure through your own abstraction | Design a path's scope, defaults, escape hatches and versioning policy | Support consumers over time: migrate them across a breaking change, measure and improve onboarding time |
| **Cost / FinOps** | Explain the cost drivers of your architecture and the FinOps cycle | Set budgets, tags and alerts; produce a cost-per-service report | Investigate a spending anomaly to a resource, a change and a decision | Model the cost of a design before building it; choose between compute options on unit economics | Manage a real budget over months, with an anomaly you detected and fixed, and before/after numbers |
| **Incident / problem mgmt** | Explain the difference, and why mitigation precedes diagnosis | Write an incident record and an RCA to a template | Run an incident end to end under time pressure, including wrong hypotheses | Design a severity matrix, escalation policy and detection strategy | Ten-plus incidents worked, four-plus RCAs, and repeat incidents that stopped recurring because of your fix |

## 3. Target levels by the end of each timeline

| Technology | 6-month | 9-month | 12-month |
|---|:--:|:--:|:--:|
| Cloud networking | 3 | 4 | 5 |
| IAM | 3 | 4 | 4 |
| Terraform | 4 | 4 | 5 |
| Docker | 3 | 4 | 4 |
| Kubernetes | 3 | 4 | 5 |
| CI/CD | 3 | 4 | 5 |
| Observability | 3 | 4 | 5 |
| Python automation | 3 | 4 | 4 |
| Security | 2 | 3 | 4 |
| Platform engineering | 3 | 4 | 4 |
| Cost / FinOps | 3 | 3 | 4 |
| Incident / problem mgmt | 3 | 4 | 5 |
| AI integration / agents / MCP | 1 | 3 | 4 |

**Level 4 across the board is comfortably above the bar for this JD.** The 9-month column is the target; the 12-month column is what makes the *next* role easy.

## 4. The self-assessment protocol

Every revision week, score yourself. Two rules make it honest:

1. **A level requires an artifact and a story.** "Terraform Level 4" means you can point at a module interface you designed *and* tell the story of the trade-off you made. No artifact, no level.
2. **Score down when in doubt.** The purpose is to find gaps while you still have time, not to feel good. A matrix that only ever goes up is a matrix that is lying.

Keep it in `docs/competency.md` with the date of each assessment, so the progression itself is visible. Showing an interviewer a competency matrix you have honestly downgraded yourself on is an unusually strong signal.

## 5. The twenty questions

If you can answer all twenty without notes at the end of the roadmap, you are ready. Test yourself out loud, at a whiteboard, at every revision week.

1. Walk me through what happens between `kubectl apply` and a running pod.
2. The app cannot reach the database. What do you check, in what order, and what does each result rule out?
3. What is Terraform state, and what are three ways it gets corrupted?
4. Explain build-once-promote-by-digest and what specifically breaks without it.
5. Why can CPU *limits* make a service slower?
6. Difference between an SLI, an SLO and an SLA — with an example from your own system.
7. Why do you mitigate before you diagnose?
8. Explain managed identity vs service principal vs OIDC federated credential.
9. Your p95 latency doubled. Walk me through the investigation.
10. What is your RTO and RPO, and how did you measure them?
11. What is the maximum request rate your system sustains, and what limits it?
12. Show me a policy that blocked a real deployment of yours.
13. Your cloud bill jumped 40%. How do you find out why?
14. What does your golden path deliberately *not* support, and why?
15. How long does it take someone to onboard a new service, and how do you know?
16. Explain connection refused vs connection timeout and what each says about where the packet died.
17. How do you guarantee your AI agent cannot change infrastructure? Name three independent mechanisms.
18. Explain prompt injection through tool results and your mitigation.
19. Walk me through your threat model and defend one accepted risk.
20. What would you do differently if you rebuilt this platform from scratch?

**Question 20 is the one that decides senior versus mid-level.** Have a real answer, with at least three specifics and one thing you would keep exactly as it is.

→ Next: [Step 16 — Final portfolio](15-portfolio.md)
