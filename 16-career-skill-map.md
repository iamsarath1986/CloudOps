# Step 17 — Final Career Skill Map

## 1. The matrix

**JD column:** ● explicitly required · ◐ implied by a responsibility · ○ not in the JD, added for your career.
**Depth:** the level from [Step 15](14-progress-measurement.md) you should reach on the 9-month plan.

| Skill | JD requires? | Target depth | Project experience | Evidence of mastery |
|---|:--:|:--:|---|---|
| Azure platform operation | ● | L4 Design | M3–M15, a live subscription across months | A bill you explained, a quota you hit, a service health event you handled |
| Cloud networking | ◐ | L4 Design | M2, M3, M6, M8 | A private-DNS break diagnosed to the exact hop; a justified subnet plan |
| Linux | ◐ | L4 Design | M2, and every later investigation | Disk, inode, signal and process failures diagnosed from a shell |
| Terraform / IaC | ● | L4→L5 | M6 onward; every resource | A `moved` refactor without destruction; state recovered; drift reconciled |
| Docker | ● | L4 Design | M5, M8 | Hardened non-root images; a base-CVE rebuild; exit code 137 explained |
| Kubernetes / AKS | ● | L4 Design | M8, M11, M13 | Six pod failure states in five minutes; a live cluster upgrade |
| CI/CD | ● | L4 Design | M7, M13 | Build-once-promote; a measured rollback; a mandatory stage added without touching consumers |
| Git / version control | ● | L4 Design | M1 onward | Branch protection, reflog recovery, bisect, a repo strategy for three consumers |
| Observability | ● | L4 Design | M9, M10, M14 | Dashboards as code; every alert has a runbook; a latency regression found from traces |
| Troubleshooting | ● | L5 Operate | M10's full catalogue | Sixteen incidents with hypotheses you rejected recorded |
| Incident management | ● | L4→L5 | M10, M11, M12, M15 | Ten records, timelines written live, workarounds inside SLA |
| Problem management | ● | L4 Design | M10 | Four RCAs; three problems whose permanent fix went into the golden path |
| Capacity management | ● | L3→L4 | M11 | Three bottleneck iterations; a capacity plan with quotas identified |
| Availability / reliability | ● | L3→L4 | M9, M11 | Two SLOs with burn-rate alerts; node loss survived with a PDB |
| Service continuity / DR | ● | L4 Design | M11 | A timed full rebuild with measured RTO/RPO, executed twice |
| IAM / access control | ● | L4 Design | M4 onward | Zero stored credentials; a deploy failure diagnosed from the permission chain |
| Cloud security | ● | L3→L4 | M4, M8, M15 | Threat model with a controls matrix; the leaked-credential drill |
| Governance / policy | ● | L3→L4 | M12 | Policy that blocked a real deployment; a documented exception process |
| Cost management / FinOps | ● | L3→L4 | M3, M12, continuously | An anomaly you detected on your own money, with before/after numbers |
| Platform engineering | ● | L4 Design | M13 | A workload you did not write, onboarded cross-repo, timed |
| Developer self-service | ● | L4 Design | M13 | The portal's PR-generating onboarding flow, and why it is not a direct API call |
| Workload onboarding | ● | L4 Design | M13 | `docs/platform/onboarding-guide.md` a stranger followed successfully |
| Python | ● | L4 Design | M1, M10, M12, M14 | `cargo-api`, fault injection, the cost reporter, the MCP server |
| Bash | ● | L4 Design | M2 onward | `net-triage.sh` and `inject-fault.sh`, both used in real drills |
| PowerShell | ● | L2 Build | M3 | One working tool; read and modify fluency |
| TypeScript / React / Node | ● | L3 Troubleshoot | M13 | `runway-portal`, deployed and operated by you |
| APIs / backend services | ● | L4 Design | M1, M13, M14 | An OpenAPI contract; a breaking change shipped without breaking a consumer |
| Serverless | ● | L3 Troubleshoot | M11, M12 | Backup-verify and cost-report jobs, with the ADR on why not AKS |
| Documentation / procedures | ● | L5 Operate | Every milestone | Runbooks executed by someone else; eight ADRs; twelve operational reviews |
| Hybrid / Azure Arc | ◐ | L2→L3 | M12 | A local VM and k3s cluster under the same policy initiative |
| AI integration (LLM APIs) | ● *(desirable)* | L3→L4 | M14 | Managed identity, private endpoint, fallback, cost per request |
| AI agents / tool calling | ● *(desirable)* | L3→L4 | M14 | A bounded agent with step limits and an audited tool loop |
| MCP | ● *(desirable)* | L3 Troubleshoot | M14 | A read-only MCP server with eight tools over live telemetry |
| AI security | ● *(desirable)* | L3 Troubleshoot | M14 | Three independent read-only mechanisms; tool-result injection tested |
| AI observability | ○ | L3 Troubleshoot | M14 | Token, cost, latency, fallback and validation-failure panels |
| Cloud architecture | ◐ | L4 Design | Throughout | Eight ADRs with rejected options argued properly |
| GitOps (Argo/Flux) | ○ | L1 Explain | — | Can explain the reconciliation model and why you deferred it |
| Service mesh | ○ | L1 Explain | — | Can explain what it would give you and its operational cost |
| AWS / second cloud | ○ | L2 Build | M15 optional | One network module ported, with a written comparison |
| ML training / MLOps | ○ | L1 Explain | — | Can explain the boundary between operating AI and building it |

## 2. Final categorisation

### Deep expertise — design, troubleshoot and operate independently
Terraform and infrastructure as code · Azure networking · Kubernetes operation · CI/CD and release engineering · observability · incident and problem management · platform engineering and golden paths.

These are the seven you should be able to hold a forty-minute technical conversation about without preparation, and the seven a hiring manager will remember.

### Strong working knowledge — comfortable in production
Azure identity and RBAC · Key Vault and secret lifecycle · Docker and image supply chain · Azure Policy and governance · cost management and FinOps · Linux administration · Python automation and tooling · Bash · backup, restore and DR execution · load testing and capacity planning · API design · serverless compute · AI service integration.

### Supporting knowledge — understand and work with when required
TypeScript, React and Node for internal tooling · PowerShell (read and modify) · Azure Arc and hybrid concepts · AI agents and MCP · AI security and observability · threat modelling · KQL · Helm.

### Future specialisation — after the core roadmap
Pick **one**, not three, and only after twelve months of real platform work:

| Direction | Where it leads | Start with |
|---|---|---|
| **Kubernetes depth** | Cluster platform engineer, operators, multi-cluster | CKS, GitOps with Argo, writing a controller |
| **Cloud security** | Cloud security engineer, security architect | AZ-500, supply chain security, detection engineering |
| **SRE** | SRE, reliability architect | Error budgets with teeth, chaos engineering, advanced capacity modelling |
| **AI platform** | AI platform engineer, ML infrastructure | Self-hosted inference, GPU capacity, model serving, evaluation platforms |
| **Architecture** | Cloud architect, principal engineer | Landing zones at scale, enterprise integration, cost modelling programmes |

Your stated goal points at **AI platform**, and this roadmap positions you for it — but the honest sequencing is: be an excellent platform engineer first. AI platform roles hire people who can operate infrastructure and happen to understand models, far more often than the reverse.

## 3. The progression

```
Software / IT Engineer
    Python · APIs · PostgreSQL · Docker · testing            ← where you are today
        ↓  M1–M5: Linux, networking, cloud, identity, containers
Cloud Engineer
    provisions and secures cloud infrastructure
        ↓  M6–M9: IaC, CI/CD, Kubernetes, observability
DevOps / Platform Engineer
    owns the path from commit to production                  ← THE JD'S LEVEL
        ↓  M10–M13: operations, capacity, continuity, governance, golden paths
Senior Cloud / Platform Engineer
    owns a platform other engineers depend on, and its economics
        ↓  M14–M15 + real production years
AI Platform / Cloud Engineer
    operates AI systems with the same rigour as everything else
```

**You reach the JD's level around Milestone 9–10.** Milestones 10 through 13 are what make you the candidate they remember rather than the candidate who is qualified — because operations and self-service are the two things the posting asks for that almost nobody arrives with.

## 4. The point of all of it

The objective was never to collect technologies. It is the engineering judgement to run the full cycle on a real system:

**Design → Provision → Automate → Deploy → Secure → Observe → Troubleshoot → Scale → Recover → Optimize → Operate**

Every milestone in this roadmap exercises several of those verbs on the same system, on purpose. That is why it is one project rather than fourteen tutorials, and it is why the answer to *"why is it built this way?"* matters more at every step than *"which command do I run?"*

The technologies will change. Terraform may not be what you use in five years, and AKS may not be where you run things. The judgement transfers completely.

← [Back to the index](README.md)
