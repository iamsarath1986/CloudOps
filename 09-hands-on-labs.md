# Section 9 — Hands-On Labs

Labs are the small, disposable experiments that come before you build something into Runway. They are how you learn **how a thing behaves**, and in particular how it fails, before you depend on it. Project implementation is what Runway keeps. Labs are what you throw away.

This section explains how the lab system works and indexes the key labs across all 23 milestones. Each milestone file has the full lab list and step detail. This index lets you see the whole progression in one place and pick labs to repeat in revision weeks.

---

## 1. Principles

| # | Principle | Why it matters |
|:--:|---|---|
| 1 | **Every lab proves one thing.** Write down what it proves before you start: *"This lab proves that a security group drop looks like a timeout, not a refusal."* | If you can't say what a lab proves, it's a tutorial, not a lab. |
| 2 | **Predict, then observe.** Write your prediction first, run the lab, then compare the result with the prediction. | The gap between what you predicted and what happened is where the learning is. A lab that matched your prediction taught you less than one that surprised you. |
| 3 | **Break it before you depend on it.** Most labs have a failure half: produce the error on purpose and read it properly. | You'll meet every one of these errors again in production. Meet them first somewhere cheap and calm. |
| 4 | **Labs are throwaway; the project is not.** Labs run in a scratch area, and nothing in a lab gets promoted into Runway by copy-paste. | Rebuilding the pattern in the project, properly, is how the lab turns into skill. |
| 5 | **Concept before console.** Monday's reading comes before Tuesday's lab (see [Section 23](23-weekly-study-routine.md)). | Otherwise you learn where a button is, not what it does. |
| 6 | **Timebox.** A lab gets 60–90 minutes. If it overruns, write down what blocked you and stop. | A lab that eats a week is a sign the concept wasn't understood first. Go back to the reading. |
| 7 | **Repeat from memory in revision weeks.** Pick three past labs and redo them without notes. | A lab you can't reproduce is one you didn't learn. |

## 2. Scratch-environment rules

Labs never run against the Runway environments (`workloads-dev`, `workloads-prod`, `runway-workloads`). They run here:

| Where | Use for | Rule |
|---|---|---|
| **Your laptop** (Docker, `kind`/`k3d`, a local Linux VM, `terraform` with a local backend) | Everything that doesn't need a real cloud API: Linux, networking, containers, Kubernetes basics, Terraform language features | Default. If a lab *can* run locally, it *must* run locally. |
| **AWS `sandbox` account** (from [M04](milestones/M04-aws-core.md)) | AWS behaviour that only exists in AWS: IAM evaluation, VPC routing, NAT, quotas, service errors | It has its own budget alert and an SCP limiting regions and instance sizes. Nuke it weekly. |
| **Azure `rg-sandbox-*` resource groups** in `runway-workloads` (from [M13](milestones/M13-azure-core.md)) | Azure behaviour: RBAC scope inheritance, private DNS, policy effects | Tag `purpose=lab` and add an `expires-on` tag. From [M15](milestones/M15-security-governance.md) onwards, a policy flags anything past its expiry. |
| **A throwaway Git repo** | Git disasters, CI experiments, workflow syntax | Never use the real `runway` repo for force-push experiments. |

**Isolation rule:** a lab must never need credentials that can touch `workloads-prod`. If it does, it isn't a lab. It's a change, and the change process applies ([M11](milestones/M11-sre-operations.md)).

## 3. Cost guardrails

Labs are where surprise bills come from, because they're where you create things fast and forget them.

1. **Budget alerts exist before the first lab resource does.** Set AWS Budgets on the sandbox account at $10, $20 and $30, and a matching Azure budget on the sandbox resource groups.
2. **Tag everything:** `purpose=lab`, `milestone=MNN`, `expires-on=YYYY-MM-DD`, `owner=you`.
3. **Tear down in the same session.** A lab resource left running overnight counts as a finding. Write it into that week's cost note.
4. **Weekly sweep.** Run `aws-nuke` (or your own script from [M02](milestones/M02-automation.md)) against the sandbox account, and delete Azure `rg-sandbox-*` groups, every Sunday. From [M16](milestones/M16-finops.md) onwards, `idle-detector` reports anything the sweep missed.
5. **Know the expensive services.** NAT Gateway (hourly charge plus per-GB data processing), load balancers, managed Kubernetes control planes, Multi-AZ databases, VPN gateways, Route 53 Resolver endpoints, Azure Firewall, and GPU instances. These get **timeboxed windows** that are written down before you start, never "I'll delete it later".
6. **A lab that could cost more than $5 gets a one-line estimate first,** written into the lab report. You'll compare it with the actual cost afterwards. That's FinOps practice from month one.

## 4. The lab report

Every lab gets a short report in `labs/MNN/NN-short-name.md`, which lives in your notes repo and not in `runway`. Five minutes, no more:

```
# Lab MNN-NN — <what it proves>
Date: · Timebox: 90 min · Actual: · Est. cost: · Actual cost:

Prediction:   what I expected to happen, and why
Observation:  what actually happened (paste the key error / output)
Explanation:  the mechanism, in my own words, without notes
Surprise:     the thing I did not expect (if none, say so — then ask whether the lab was too easy)
Carry-over:   what changes in the Runway project because of this
```

The **Explanation** line is what matters. If you can't write it without looking something up, the lab isn't finished.

---

## 5. Lab index by milestone

The key labs only, one line each. Full steps are in each milestone file.

### [M00 — Engineering foundations](milestones/M00-foundations.md)
- Wreck a scratch repo (commit to the wrong branch, amend a pushed commit, force-push) and recover with `reflog`. *Proves: Git history is recoverable if you understand refs.*
- `git bisect` a regression you planted across 20 commits. *Proves: bisecting is mechanical, not magic.*
- Commit a fake secret and scrub it with `filter-repo`. *Proves: removing it from history doesn't un-leak it, so rotation is the real fix.*
- Branch protection: try to push to `main` and watch it fail. *Proves: controls you haven't tested aren't controls.*

### [M01 — Linux and networking](milestones/M01-linux-networking.md)
- Subnet `10.0.0.0/16` by hand for the CIDR plan. *Proves: you can size address space before a cloud forces you to.*
- `dig +trace` a name. Change the TTL and watch it propagate. *Proves: DNS changes are only as fast as the TTL allows.*
- Reproduce four TLS failures: expired certificate, wrong hostname, untrusted issuer, missing intermediate. *Proves: you can read a certificate chain.*
- `tcpdump` a `curl` request and identify the SYN, SYN-ACK, ACK, ClientHello and FIN. *Proves: you know what's on the wire.*
- **Produce the four connection failures: refused, timeout, reset and hang.** *Proves: you can tell where a packet died from `curl -v` alone. This is the most useful diagnostic shortcut in the roadmap.*
- Fill a disk, then separately exhaust its inodes. *Proves: "disk full" has two meanings.*
- Send `SIGTERM` mid-request, then implement graceful shutdown. *Proves: signal handling decides whether a deploy drops requests.*
- systemd unit with `Restart=`, a dependency, and `journalctl` filtering. *Proves: you understand what a container runtime later replaces.*

### [M02 — Python, Bash and PowerShell automation](milestones/M02-automation.md)
- A `boto3` script that lists everything tagged `purpose=lab`, with pagination and retry on throttling. *Proves: SDK automation has to survive partial failure.*
- A Bash script with `set -euo pipefail`: break it three ways and read the exit codes. *Proves: you write scripts that fail loudly.*
- The same task in Bash, Python and PowerShell. *Proves: you can read and modify all three, and you know when each one fits.*
- Package a Python CLI with tests and version it. *Proves: a script turns into a tool once someone else depends on it.*

### [M03 — Cloud architecture fundamentals](milestones/M03-cloud-architecture.md)
- Composite availability: compute Runway's theoretical SLA from its component SLAs on paper. *Proves: availability multiplies downwards.*
- Little's Law on paper for `cargo-api` at 50 req/s and 200 ms. *Proves: you can size concurrency before load testing.*
- Stateless versus stateful: kill a local `cargo-api` replica holding in-memory session state and watch requests fail. *Proves: why state lives outside the process.*
- Write the first SDD for Runway v0 and argue for one rejected option. *Proves: you can design before you build.*

### [M04 — AWS core infrastructure](milestones/M04-aws-core.md)
- Budgets and alerts before the second resource. *Proves: the habit.*
- Derive least privilege empirically: start with an empty policy and add one action per `AccessDenied`. *Proves: least privilege is built, not guessed.*
- Read the IAM policy evaluation logic: create an explicit deny that beats an allow, and an SCP that beats both. *Proves: you understand the evaluation order.*
- Build a VPC by hand: internet gateway, NAT, route tables, public and private subnets. Remove one route and diagnose it. *Proves: routing is explicit.*
- Security group versus NACL: stateful versus stateless, shown with a return-traffic failure. *Proves: why NACLs surprise people.*
- Trigger a service quota error and read it. *Proves: quotas are a capacity ceiling.*

### [M05 — Terraform](milestones/M05-terraform.md)
- Rebuild the hand-built M04 environment in Terraform, then diff the code against the manual build log. *Proves: the case for IaC, in one page.*
- S3 backend with `use_lockfile`: run two concurrent applies. *Proves: locking works, and you know how to `force-unlock` safely.*
- Console drift, then `plan`, then reconcile. Also `import` an out-of-band resource. *Proves: state is a belief about reality.*
- `moved` refactor into a module without destruction. *Proves: you can maintain Terraform, not just write it.*
- Corrupt a copy of the state file and recover it from S3 versioning. *Proves: state recovery is a procedure.*
- `count` versus `for_each`: remove a middle element. *Proves: why `for_each` is usually right.*
- `terraform test` plus a failing conftest policy on the plan JSON. *Proves: IaC can be tested before it's applied.*

### [M06 — Docker and containers](milestones/M06-containers.md)
- A naive image versus multi-stage and slim versus distroless. Record size, build time and CVE count. *Proves: image choices are measurable.*
- Reorder the Dockerfile for caching and time a one-line rebuild. *Proves: layer order is build latency.*
- The PID 1 signal trap: shell-form versus exec-form entrypoint. *Proves: why containers get SIGKILLed.*
- Set a memory limit below the working set and see exit code 137. *Proves: you'll recognise OOMKilled instantly.*
- Generate an SBOM and find a transitive dependency you didn't know about. *Proves: you ship more than you wrote.*
- Pull by a mutable tag after it has moved. *Proves: why digests exist.*

### [M07 — CI/CD and release engineering](milestones/M07-cicd.md)
- A pipeline that fails at each stage in turn: lint, types, unit, contract, scan, smoke. *Proves: every gate catches what it claims to.*
- Rebuild per environment instead of promoting by digest, and watch dev and prod diverge. *Proves: why build-once exists.*
- Time a rollback on ECS Fargate. *Proves: rollback is a number, not a hope.*
- Expand-contract migration, then the naive version breaking prod on purpose. *Proves: how schema changes cause most deploy outages.*
- Break the OIDC trust condition and read the failure. *Proves: you can debug federation.*

### [M08 — Kubernetes fundamentals](milestones/M08-kubernetes-fundamentals.md)
- **Cause and diagnose each of ImagePullBackOff, CrashLoopBackOff, OOMKilled, Pending, CreateContainerConfigError and a readiness probe that never passes.** *Proves: the six-state triage, which is the Kubernetes interview.*
- A liveness probe shorter than startup time, fixed with a startup probe. *Proves: probes can cause outages.*
- A rollout with `maxUnavailable: 0` and a broken image, then `rollout undo`. *Proves: safe rollouts stall instead of taking the service down.*
- A default-deny NetworkPolicy that breaks your own service, then explicit allows. *Proves: you understand pod-to-pod traffic.*
- An ingress-nginx lab, then the same routing with Gateway API. *Proves: you can replace a retired component (the EOL exercise).*

### [M09 — Production Kubernetes on EKS](milestones/M09-eks.md)
- Exhaust VPC CNI IPs on a small subnet and watch pods stay Pending. *Proves: EKS pods consume VPC addresses.*
- Pod Identity versus IRSA: break each and trace the credential chain. *Proves: you know where a pod's AWS identity comes from.*
- Karpenter: an unschedulable pod triggers a new node. Then consolidation. *Proves: node autoscaling reacts to requests, not to CPU usage.*
- Drain a node with and without a PDB. *Proves: disruption budgets matter only when you need them.*
- A zero-downtime ECS→EKS cutover using weighted target groups. *Proves: a migration is a change with a rollback plan.*
- A cluster version upgrade with traffic flowing, and a deprecated API caught before it's removed. *Proves: EOL is managed.*

### [M10 — Observability](milestones/M10-observability.md)
- Instrument with OTel and read one waterfall. Find the hop you were wrong about. *Proves: traces lead in distributed systems.*
- Break context propagation across SQS and watch the orphaned spans. *Proves: async systems need explicit propagation.*
- Add a high-cardinality label and watch AMP series and cost climb. *Proves: cardinality is a cost problem.*
- Rebuild a deleted dashboard from code in under 5 minutes. *Proves: dashboards as code are real.*
- Stop the collector and see whether anything alerts. *Proves: absence-of-data alerting (S17).*

### [M11 — SRE and production operations](milestones/M11-sre-operations.md)
- Write the severity matrix and alert tiers before the first drill. *Proves: you can classify under pressure.*
- Error budget maths for 99.5% over 30 days, plus multi-window burn-rate thresholds. *Proves: SLOs are arithmetic.*
- `ems-bridge`: an alert creates an incident record automatically. *Proves: event management is wiring plus discipline.*
- A game day with random fault injection while you're not watching. *Proves: detection works when you're genuinely surprised.*
- A failed change leading to an emergency change and a PIR. *Proves: change management is an engineering loop, not paperwork.*

### [M12 — Capacity, high availability and disaster recovery](milestones/M12-capacity-ha-dr.md)
- Open-model k6 at rising arrival rates. Plot p50/p95/p99 and find the knee. *Proves: capacity has a shape.*
- Three bottleneck iterations. *Proves: fixing one bottleneck reveals the next.*
- Connection pool of 5 against 50 concurrent requests. *Proves: latency becomes queueing time (S18).*
- CPU limits that are too low: throttling with no CPU alarm. *Proves: S19.*
- PITR restore of the database, timed. *Proves: your real data RPO and RTO.*
- Pilot-light rebuild in a second region from Terraform, timed twice. *Proves: the runbook improves only through execution.*

### [M13 — Azure core and enterprise identity](milestones/M13-azure-core.md)
- RBAC scope inheritance from management group down to resource, compared with IAM evaluation. *Proves: the models differ, and you can say how.*
- Private endpoint without the private DNS zone link, compared with the linked version. *Proves: S24's mechanism.*
- Federate an EKS pod to Entra ID and break the subject claim. *Proves: cross-cloud identity has no static keys (S25).*
- Switch IAM Identity Center's identity source to Entra ID in a sandbox first. *Proves: identity migrations need rehearsal.*

### [M14 — Production AKS and multi-cluster GitOps](milestones/M14-aks-multicluster.md)
- AKS Workload Identity reading Key Vault through External Secrets. *Proves: one pattern, two clouds.*
- Argo CD: make a manual `kubectl edit` and watch self-heal revert it. *Proves: pull-based reconciliation beats drift.*
- A bad sync-wave ordering (app before CRD). *Proves: S26.*
- An AKS version upgrade with PDBs, compared with the EKS upgrade. *Proves: upgrades transfer across clouds.*

### [M15 — Cloud security and governance](milestones/M15-security-governance.md)
- A tag policy in Audit, reach compliance, then Deny. The same in an SCP and in Kyverno. *Proves: the audit-first rollout.*
- A Deny without an audit period breaks your own pipeline. *Proves: S28, and why exemptions have expiry dates.*
- cosign-sign an image and have Kyverno reject an unsigned one. *Proves: supply-chain enforcement at admission.*
- STRIDE on Runway: each threat gets a control or an accepted risk. *Proves: you can threat-model your own design.*
- The leaked-credential drill. *Proves: S27 is contain, rotate, assess and prevent, not just "rotate".*

### [M16 — FinOps](milestones/M16-finops.md)
- Deliberately create a NAT data-processing bill by pulling images through NAT, then add VPC endpoints. *Proves: data transfer is a design choice.*
- OpenCost allocation compared with the actual node bill. *Proves: shared cost needs an allocation rule.*
- FOCUS exports from both clouds, normalised into one report. *Proves: multi-cloud cost needs a common schema.*
- Leave a node pool scaled up over a weekend and detect it with your anomaly alert. *Proves: S29.*

### [M17 — Golden paths](milestones/M17-golden-paths.md)
- List every manual onboarding step honestly. *Proves: the length of that list is the specification.*
- Design the `workload.yaml` interface on paper before implementing it. *Proves: module design is API design.*
- Release a breaking module version without a migration guide and feel the pain. *Proves: migrating consumers is the real work.*
- Onboard a workload with a persistent volume. *Proves: where the abstraction leaks (S30).*

### [M18 — Internal developer platform](milestones/M18-developer-platform.md)
- A portal form that generates a PR, compared with a button that calls the cloud API. *Proves: why review, audit and rollback stay in the loop.*
- Account vending: request a sandbox account through the portal. *Proves: self-service with guardrails.*
- Time `hello-svc` onboarding to both clouds. *Proves: the golden path is cloud-agnostic.*

### [M19 — AI infrastructure and AI workloads](milestones/M19-ai-infrastructure.md)
- Call Azure OpenAI with managed identity and no key, through a private endpoint. *Proves: AI access is an identity and network problem first.*
- Exceed the TPM quota, then compare naive retry with backoff plus jitter. *Proves: S31.*
- Structured output validation failure: repair once, then fail closed. *Proves: model output is untrusted input.*
- Primary to Bedrock to degraded fallback with the primary disabled. *Proves: the fallback path works and shows up on the dashboard.*
- Self-host a small model on CPU and compare latency, throughput and cost per request with managed inference. *Proves: ADR-024 is made with numbers.*
- An eval set as a pipeline gate blocks a prompt regression. *Proves: prompts are deployable config.*

### [M20 — AI agents and MCP](milestones/M20-ai-agents-mcp.md)
- A two-tool MCP server over Streamable HTTP, and a client handshake. *Proves: you understand the protocol, not just a framework.*
- A bad tool description leads to misuse. Rewrite it. *Proves: tool descriptions are prompts.*
- An injection string in a log line. *Proves: S33, and that read-only by construction holds.*
- Remove the step limit and watch the token burn. *Proves: bounds must be in code.*
- Attempt a write through the agent's identity and have the cloud refuse it. *Proves: the guarantee is architectural.*

### [M21 — Hybrid and multi-cloud](milestones/M21-hybrid-multicloud.md)
- Arc-connect the local VM and k3s cluster and apply the same Azure Policy. *Proves: one control plane over the "datacentre".*
- An SSM hybrid activation on the same VM. *Proves: the AWS equivalent, and where it differs.*
- A site-to-site VPN between AWS and Azure (timeboxed). Break one tunnel. *Proves: S34 and tunnel redundancy.*
- Hybrid DNS forwarding in both directions. *Proves: name resolution across three networks.*

### [M22 — Hardening and capstone](milestones/M22-hardening-capstone.md)
- Rotate every secret in the system and time it. Find the ones you forgot. *Proves: rotation is a practised procedure.*
- Egress restriction: discover every outbound dependency. *Proves: you know what your system talks to.*
- The capstone design exercise from [Section 25](25-production-readiness-checklist.md#part-2--the-final-capability-self-test), done cold in 3 hours. *Proves: you can design without the roadmap.*

---

## 6. Which labs to repeat

In each revision week, pick three labs you did at least eight weeks earlier. If you only have time for a few over the whole roadmap, these carry the most weight in interviews and incidents:

1. The four connection failures (M01)
2. IAM evaluation order, including an SCP (M04)
3. State recovery and `moved` (M05)
4. The six pod failure states (M08)
5. Pod Identity / federation break-and-trace (M09, M13)
6. The PITR restore, timed (M12)
7. The leaked-credential drill (M15)
8. The 429 retry-storm comparison (M19)

→ Next: [Section 10 — Failure and Troubleshooting Exercises](10-failure-and-troubleshooting.md)
