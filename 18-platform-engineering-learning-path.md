# Section 18 — Platform Engineering Learning Path

*The target role sits in a Platform Engineering team. This track is the one that turns everything else — cloud, Terraform, pipelines, Kubernetes, observability, policy — into a product other engineers use without needing to know what you know.*

---

## 1. What the JD actually asks for

> *"Join our Platform Engineering team, the backbone of our cloud modernization journey. We architect and engineer next-generation cloud platforms..."*
> *"The role will act as a technical expert within Public Cloud, supporting cloud automation, workload onboarding, platform engineering, and the development of internal self-service capabilities."*
> *"Experience building: Cloud-native platforms · Internal developer tools · Self-service portals · Automation solutions"*
> *"Experience operating large public cloud footprints and onboarding applications using governed and automated DevOps practices."*

This is the most distinctive requirement in the posting and the one almost no candidate can evidence. Note the senior framing: you are not a *consumer* of a platform; you are expected to **build the internal tools and portals**, and to onboard applications at scale in a governed way.

**Depth expected:** Advanced on golden paths, reusable modules and workload onboarding; Production Proficiency on self-service portals and developer tooling.

## 2. The idea in one sentence

A platform team builds **a product whose users are other engineers**, and its value is measured by one number:

> **How long does it take an engineer who has never met you to get a new service running in production — correctly configured, observed, secured and governed — and how many humans did they have to wait for?**

Write that number in your README. Everything in this track exists to shrink it without loosening governance.

## 3. The transition you will live through

The brief asked for this progression. Runway takes you through every step, in order, on the same system — and each step exists because the previous one hurt.

| Step | What it looks like | Where in Runway | The pain that forces the next step |
|:-:|---|---|---|
| 1 | **Manual cloud operations** | [M04](milestones/M04-aws-core.md): VPC, EC2, RDS, ALB built by hand; `manual-build-log.md` | Forty steps nobody could repeat; drift the moment you finish |
| 2 | **Infrastructure automation** | [M02](milestones/M02-automation.md), M04: Bash/Python/boto3 scripts | Scripts are imperative: they do not know what already exists |
| 3 | **Infrastructure as Code** | [M05](milestones/M05-terraform.md): everything in Terraform, `make up`/`make down` | Applies from a laptop; no review; no record |
| 4 | **CI/CD** | [M07](milestones/M07-cicd.md): plan on PR, apply on merge, build once/promote | Every repository copies the pipeline and then diverges |
| 5 | **Standardised cloud patterns** | [M09](milestones/M09-eks.md)–[M15](milestones/M15-security-governance.md): `k8s-baseline`, network, account and cluster patterns, tagging, policies | Each new service still needs a platform engineer to wire it up |
| 6 | **Reusable Terraform modules** | [M17](milestones/M17-golden-paths.md): versioned modules with designed interfaces, including `workload` | Consumers must still understand Terraform and the platform's internals |
| 7 | **Golden paths** | M17: template repo + `workload.yaml` + reusable workflows + Argo CD + standard telemetry | Onboarding is fast but still needs someone to know where the template lives and which PR to open |
| 8 | **Internal developer platform** | [M18](milestones/M18-developer-platform.md): `runway-portal` — service catalogue, data catalogue, SLO/cost/change views | Requests still arrive as messages to a human |
| 9 | **Self-service infrastructure** | M18: request a service / environment / sandbox account → the portal opens a PR → policy + pipeline + Argo CD do the rest | The end state: no ticket, full governance |

**The senior judgement is knowing *when* each step is worth it.** Building a portal before you have a golden path is a UI over chaos. Building a golden path before you have operated the things it provisions is a template for mistakes.

## 4. The concepts, as you will need them

### Developer experience (DX)
The platform's users are engineers with deadlines. DX is the sum of: how long it takes to start, how clear the errors are, how often they need to ask a human, and how legible the system is when it breaks. Measure it (time to first deploy, onboarding survey, support requests per onboarding) or you are guessing.

### Platform as a product
Voluntary adoption is the honest test. A platform slower than doing it by hand gets routed around; a platform mandated by decree survives until the next reorganisation. Product practices apply: a roadmap, a changelog, office hours, deprecation notices, and the courage to say no. (*Team Topologies*: the platform team reduces the cognitive load of stream-aligned teams.)

### Platform APIs
The interfaces your users touch, in order of preference: a **declarative contract** (`workload.yaml`), **versioned modules**, **reusable workflows**, and only then **the portal** (which renders and submits the contract). The contract is the API; the portal is one client of it. Design it for intent, not implementation: *"I am an HTTP service on port 8080, I need Postgres and a queue, my owner is team-cargo, my data is `internal`, target `aws`"* — never `aws_eks_cluster`.

### Service catalogue
The inventory of what exists, who owns it, what it depends on, and where its docs, dashboards and runbooks are. In Runway: Backstage-format `catalog-info.yaml` per component (so migrating to Backstage later is cheap — ADR-022), rendered by the portal. A **data catalogue** alongside it (`catalog/data/*.yaml`): dataset, owner, classification, PII, retention, location, backup and RPO — the JD's "data catalogue for the technology area".

### Golden paths
Opinionated, supported, well-lit routes covering ~80% of cases, with **documented escape hatches** for the rest. Two ways to get it wrong: so narrow nobody fits, or so configurable it stops being a path. Both look like low adoption. Document what the path supports, what it does not, and how to get help outside it (S30).

### Reusable templates
Template repositories (Dockerfile, `workload.yaml`, CI caller, README, `catalog-info.yaml`), module scaffolds, dashboard and alert templates. Templates are copied once and then diverge — which is why the *logic* belongs in versioned modules and workflows, and templates stay thin.

### Guardrails and policy as code
The paved road (defaults that are right) plus guardrails (limits that are enforced). Policy at four points — PR, cloud API, admission, runtime ([Section 17 §4](17-security-governance-learning-path.md#4-policy-at-four-enforcement-points-adr-019)). **Guardrails are what let you grant autonomy safely.**

### Self-service provisioning
*Without waiting for a human* — not *without governance*. Runway's portal **opens a pull request**; it never calls a cloud API to mutate anything. Review, policy, audit, change records and rollback stay in the loop while the ticket disappears. Standard changes (a workload through the golden path) auto-merge after checks; normal changes (a new sandbox account) wait for a named approver. Be able to defend that design — interviewers will probe it.

### Standardised observability
Because the `workload` module creates dashboards, alerts and SLOs, observability is a **property of being on the platform** rather than a task. Because the reusable workflow injects OTel configuration and the log schema, every service is queryable together. Cross-service queries are most of what an operator does.

### Standardised CI/CD
Consumer pipelines are six to fifteen lines calling your versioned workflows. The test: **can you add a mandatory stage (e.g. SBOM attestation) for every team with one PR to one repo?**

### Module design as API design
Minimal required inputs; opinions in the defaults; outputs consumers genuinely need; non-negotiables (tags, network policy, limits, log format) are *not* variables; semantic versioning as a promise; a written deprecation policy. The hard part of platform work is not `v1` — it is releasing `v2` and migrating consumers you do not control (ADR-021).

### Multi-cloud golden path
`target: aws | azure` in `workload.yaml`. The same contract produces an EKS or AKS namespace, identity (Pod Identity vs Workload Identity), secrets wiring (ESO against Secrets Manager or Key Vault), DNS, gateway route, dashboards and alerts. `hello-svc` is onboarded to **both** — the proof that the contract is the abstraction and the cloud is an implementation detail.

## 5. What to measure

| Metric | Why | Where it comes from |
|---|---|---|
| **Time to first production deploy** for an unfamiliar service | The headline number | Timed `hello-svc` onboarding, on both clouds |
| Humans waited on per onboarding | Self-service means zero for standard changes | Portal + PR history |
| Manual steps remaining | Each is a defect in the path | Your honest M17 list |
| Platform-code touches per onboarding | Should be zero; above zero is a leaky abstraction | Git history |
| Module / workflow version spread | Your migration debt | Report across consumer repos |
| Change failure rate, golden path vs hand-rolled | Is the path actually safer? | Change records |
| Support requests per onboarding | DX quality | `docs/platform/support-log.md` |

## 6. Anti-patterns to avoid on purpose

| Anti-pattern | Why it fails |
|---|---|
| A ticket queue with extra steps | If a human must act on every request, it is not self-service |
| A portal that mutates infrastructure directly | Bypasses review, policy, audit and rollback |
| A wrapper so thin it adds nothing | Consumers must still understand Terraform — they will skip it |
| A wrapper so thick nobody can debug it | When production breaks, someone must see through the abstraction |
| Building the portal first | A UI over an undefined contract; the contract changes and the UI rots |
| Mandatory adoption by decree | Survives until the next reorganisation |
| No deprecation policy | Every version you ever shipped lives forever |
| Golden path without the operational defaults | Fast onboarding of services nobody can observe |

## 7. Checkpoints

| By | You can |
|---|---|
| M05 | Show a module whose README was written before its code |
| M07 | Show two repositories consuming the same reusable workflows |
| M17 | Onboard `hello-svc` from a separate repository through the golden path, touching no platform code, and state the elapsed time; release a breaking module version with a migration guide |
| M18 | Show a sandbox account and a new service requested through the portal, each producing a reviewed PR and a change record; show the data catalogue entry for every dataset |
| M22 | Show onboarding time trend across releases of the path and explain what moved it |

## 8. What you should be able to say in an interview

> *"I built a multi-cloud golden path. A developer copies a template repository, fills in a `workload.yaml` of about ten fields — including `target: aws` or `azure` — and opens a PR, either by hand or through our portal. That produces a namespace with quota, a workload identity federated to the service account, secrets wired through External Secrets, DNS, TLS, a gateway route, standard dashboards, burn-rate alerts with runbook links, a catalogue entry, and policy-conformant tags — delivered by Argo CD. Standard changes auto-merge after gates; anything touching shared platform waits for an approver, and every deploy writes a change record. I onboarded a service I had not written, from a separate repository, to both EKS and AKS without touching platform code, in N minutes. The path deliberately does not support stateful workloads or non-HTTP protocols; here is the escape hatch. When I shipped v2 of the workload module I wrote the migration guide and moved the consumers myself, because migrating consumers is the actual job."*

## 9. Resources

1. **CNCF Platforms White Paper** and **CNCF Platform Engineering Maturity Model** — read both in full (short). They give you the vocabulary employers use.
2. ***Platform Engineering*** (Fournier & Nowland, 2024) — ch. 1–2 (why platforms), the chapters on building platforms as products and on migrations/deprecation. The best current book on the organisational side.
3. ***Team Topologies*** (Skelton & Pais) — ch. 5 "The Four Fundamental Team Topologies" (platform teams) and ch. 7 "Team Interaction Modes" (X-as-a-Service).
4. ***Terraform: Up & Running*** 3rd ed (Brikman) — ch. 8 "Production-Grade Terraform Code" (module design, versioning, testing).
5. **Backstage docs** — "Software Catalog → Descriptor Format" and "Software Templates" — so your catalogue is compatible even though you build your own portal.

→ Next: [Section 19 — FinOps Learning Path](19-finops-learning-path.md)
