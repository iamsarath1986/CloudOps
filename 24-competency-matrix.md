# Section 24 — Competency Matrix

Finishing a tutorial doesn't mean you've acquired a skill. This matrix defines, for each discipline in the roadmap, **the evidence you need to move from one level to the next**. Evidence means something in Git, a measured number, or something you did in front of someone. How confident you feel doesn't count.

## 1. The levels

| Level | Name | The test |
|:--:|---|---|
| **L0** | No knowledge | You couldn't explain what it's for. |
| **L1** | Understand concepts | You can explain what it is, what problem it solves and how it works internally, without notes, to someone who doesn't know it. |
| **L2** | Implement with guidance | You can make it work by following documentation, examples or a mentor's steer. You couldn't yet do it again from a blank page. |
| **L3** | Implement independently | You can build it from scratch, unaided, for a realistic case, and debug your own mistakes. |
| **L4** | Operate in production | You've run it over time: changed it under load, upgraded it, recovered it from failure, paid for it, and diagnosed incidents in it, with written records. |
| **L5** | Design, troubleshoot and teach others | You can choose between designs and defend the rejected option, diagnose failures other people are stuck on, and teach it well enough that someone else succeeds using your material. |

**Two rules make the matrix honest:**

1. **Levels are cumulative, and every level needs an artefact and a story.** "Terraform L4" means you can point to the state recovery runbook you wrote *and* tell the story of the time you used it.
2. **Reading never moves you a level.** Building takes you to L2–L3. Breaking and fixing takes you to L3. Operating over months takes you to L4. Designing for others and teaching takes you to L5.

L4 needs *time*. You can't reach it in a weekend, which is why Runway stays alive across two years and the operations phases insist on soak windows. L5 on a solo project is honestly capped for some areas. The table says where.

---

## 2. The matrix — evidence required to reach each level

Paths refer to the Runway repository layout ([Section 5](05-production-project-proposal.md)).

### Foundations

| Skill | L1 | L2 | L3 | L4 | L5 |
|---|---|---|---|---|---|
| **Linux** | Explain processes, signals, systemd, permissions, filesystems and inodes, cgroups/namespaces | Follow a guide to write a systemd unit and read `journalctl` | Diagnose from a cold start, unaided: full disk, inode exhaustion, port already bound, zombie, a DNS failure; graceful shutdown implemented ([M01](milestones/M01-linux-networking.md)) | Debug containers and nodes via `/proc`, cgroups, `ss`, `strace`, `tcpdump` during real incidents; records in `docs/incidents/` | Write the node-troubleshooting runbook someone else uses successfully; explain container isolation at kernel level |
| **Networking** | Explain CIDR, routing, DNS resolution order, TCP handshake, TLS chain, L4 vs L7 | Build a VPC with public and private subnets following docs | Subnet a /16 on a whiteboard; produce and name the four connection failures; `net-triage.sh` finds the broken hop ([M01](milestones/M01-linux-networking.md)) | S01/S02/S15/S24 diagnosed from symptoms in live environments; a network change made in prod with no outage | ADR-004 CIDR plan spanning AWS, Azure and the datacentre; hybrid DNS and VPN ([M21](milestones/M21-hybrid-multicloud.md)); teach the triage order |
| **Python** | Explain typing, async, packaging, testing | Write a working cloud SDK script from examples | Tools with retries, pagination, throttling handling and tests; `cargo-api` built and tested ([M00](milestones/M00-foundations.md), [M02](milestones/M02-automation.md)) | `ems-bridge`, `finops-collector`, `ai-gateway` running in prod with SLOs, and fixed during incidents | Tooling other engineers depend on, with versioning, error messages good enough for 3 a.m., and a contribution guide |
| **Bash / PowerShell** | Explain `set -euo pipefail`, exit codes, pipes; the PowerShell object pipeline | Modify an existing script | `net-triage.sh`, `inject-fault.sh`; one PowerShell module against Az ([M02](milestones/M02-automation.md)) | Scripts used in CI and in real incidents, failing safely | *(L4 target. For PowerShell, L3 is enough for this JD)* |
| **Git** | Explain the object model, refs, rebase vs merge | Branch, PR and resolve conflicts with help | Recover from reflog, bisect a regression, branch protection and CODEOWNERS ([M00](milestones/M00-foundations.md)) | Repository strategy across the monorepo plus split repos over months; release tagging for modules | Design the branching and release model for a platform whose consumers you don't control (ADR-001, ADR-021) |

### Cloud

| Skill | L1 | L2 | L3 | L4 | L5 |
|---|---|---|---|---|---|
| **AWS** | Explain Organizations, accounts, regions/AZs, the shared responsibility model, IAM evaluation, VPC primitives | Hand-build a VPC, EC2 and RDS following docs ([M04](milestones/M04-aws-core.md)) | Build multi-account, multi-AZ infrastructure in Terraform, unaided; debug AccessDenied from the evaluation chain | EKS, RDS and SQS operated across months: quota hit, upgrade, PITR restore, cost anomaly (S13/S22/S29) | Design an account structure / landing zone (ADR-003) and a DR tier (ADR-014); ready for SA Professional |
| **Azure** | Explain tenant / management group / subscription / RG, Entra objects, RBAC inheritance, private endpoints | Hand-build a hub-spoke VNet with a private endpoint following Learn docs ([M13](milestones/M13-azure-core.md)) | AKS, Key Vault, private DNS and Azure OpenAI in Terraform, unaided; S24 diagnosed | AKS and the AI tier operated for months with upgrades, policy rollouts and incidents | Design a governance hierarchy and workload placement (ADR-015/016); ready for AZ-305 |
| **IAM** (both clouds) | Explain authn vs authz, roles, policies, federation, workload identity | Set up OIDC from GitHub to AWS with a guide | Least privilege derived empirically; Pod Identity, Workload Identity and cross-cloud federation built ([M13](milestones/M13-azure-core.md)) | Zero static credentials across both clouds for months; S04/S14/S25 diagnosed; a role tightened without breakage | Explain three independent read-only mechanisms (ADR-025) and the full federation graph on a whiteboard |
| **Terraform** | Explain state, plan/apply, providers, the dependency graph | Write a root module from docs | Modules, remote state with `use_lockfile`, `moved`/`import`, tests and conftest ([M05](milestones/M05-terraform.md)) | Terraform run through pipelines for months: drift reconciled, state recovered, a provider major upgrade done | Design the module interface and versioning for consumers you don't control; migrate them across a breaking release ([M17](milestones/M17-golden-paths.md)) |

### Delivery and runtime

| Skill | L1 | L2 | L3 | L4 | L5 |
|---|---|---|---|---|---|
| **Docker** | Explain namespaces, cgroups, layers, the PID 1 trap | Write a working Dockerfile | Multi-stage, non-root, pinned, scanned, SBOM, signed ([M06](milestones/M06-containers.md)) | Images maintained over time: base CVE rebuild, digest pinning, S08/S09 diagnosed | ADR-006 base image and supply-chain policy, enforced by Kyverno |
| **Kubernetes** | Explain the reconciliation loop, scheduler, probes, requests/limits | Deploy with manifests from examples on kind | Six failure states diagnosed in under 5 minutes; NetworkPolicy, PDB, HPA ([M08](milestones/M08-kubernetes-fundamentals.md)) | EKS and AKS upgraded with traffic flowing; S13/S21/S36 handled; Karpenter tuned from data ([M09](milestones/M09-eks.md), [M14](milestones/M14-aks-multicluster.md)) | Design multi-cluster tenancy, node strategy and GitOps (ADR-018); CKA; teach pod triage |
| **CI/CD** | Explain build-once-promote, gates, environments, rollback | A pipeline from a template | Reusable workflows, OIDC, digest promotion, measured rollback ([M07](milestones/M07-cicd.md)) | Pipelines that others depend on, operated for months; change records generated automatically; S10/S11/S35 handled | Add a mandatory stage to every consumer without touching their repos; justify every gate |

### Running it

| Skill | L1 | L2 | L3 | L4 | L5 |
|---|---|---|---|---|---|
| **Observability** | Explain logs vs metrics vs traces, RED/USE, cardinality, percentiles | Instrument one service from the OTel docs | Full OTel pipeline, dashboards and alerts as code ([M10](milestones/M10-observability.md)) | Months of operation: thresholds tuned, ingest cost controlled, every alert reviewed for usefulness; S16/S17 | ADR-012 defended, including its blast-radius weakness; a standard telemetry contract in the golden path |
| **SRE** (incident, problem, change, SLOs) | Explain incident vs problem vs change; SLI/SLO/SLA; why mitigation comes before diagnosis | Write an incident record from the template | Run an incident end to end under time pressure; burn-rate alerts built ([M11](milestones/M11-sre-operations.md)) | 25+ incidents, 6+ problems, 3+ PIRs; monthly reviews; error budget used in a real decision | Design the severity, tier and change process (ADR-013); a runbook someone else executes cold |
| **Capacity, HA and DR** | Explain Little's Law, the knee, RTO/RPO, backup vs replication | Run k6 from examples; restore a snapshot | Find the saturating resource three times; timed PITR ([M12](milestones/M12-capacity-ha-dr.md)) | Capacity plan kept current; DR drill executed twice with measured RTO/RPO | ADR-014 DR tiers with costs; rejected cross-cloud DR defended |

### Governance and platform

| Skill | L1 | L2 | L3 | L4 | L5 |
|---|---|---|---|---|---|
| **Security** | Explain least privilege, defence in depth, trust boundaries, the supply chain | Enable scanning / WAF following docs | Threat model with a controls matrix; signing plus admission; egress restriction ([M15](milestones/M15-security-governance.md), [M22](milestones/M22-hardening-capstone.md)) | Leaked-credential drill (S27) run; vulnerability triage kept current for months | Defend each accepted risk in the threat model; review someone else's design |
| **Governance** | Explain SCP, Azure Policy effects, admission control, audit-first rollout | Assign a built-in policy | Custom policies at four enforcement points (ADR-019) | Policy blocked real deploys; exemptions with expiry; S28 recovered | Design the enforcement model for autonomy with guardrails, across both clouds |
| **FinOps** | Explain cost drivers, tagging, allocation, commitments, the FinOps lifecycle | Set budgets and tags | FOCUS normalisation, OpenCost, unit cost per 1k requests ([M16](milestones/M16-finops.md)) | Months of your own bill managed; S29 investigated with before/after numbers | Model the cost of a design before building it; showback by team in the portal |
| **Platform Engineering** | Explain golden paths, platform as product, paved road vs guardrail | Write a reusable module and workflow | Golden path end to end ([M17](milestones/M17-golden-paths.md)); portal PR-based provisioning ([M18](milestones/M18-developer-platform.md)) | `hello-svc` onboarded to both clouds with a timed result; consumers migrated across a breaking release | *Honest ceiling for a solo project: L4.* L5 needs real internal customers. Say so in interview |
| **System Design** | Explain regions/AZs, statelessness, caching, queues, consistency, SLA maths | Draw a design from a reference architecture | SDDs for Runway v0 and later phases ([M03](milestones/M03-cloud-architecture.md)) | Runway evolved through 27 ADRs without a rewrite | Capstone design done cold in 3 hours ([Section 25](25-production-readiness-checklist.md)); ADR-027 retrospective |
| **Documentation** | Explain ADR / SDD / runbook / PIR purposes | Fill a template | Runbooks, ADRs, catalogue entries written as you go | Runbooks followed successfully by someone else; data catalogue and EOL register kept current | Documentation standards ([Appendix A2](appendix/A2-documentation-standards.md)) adopted by someone else |
| **AI Infrastructure** | Explain managed inference, TPM/RPM quotas, structured output, MCP, tool calling, prompt injection | Call a model with managed identity from a guide | ai-gateway with routing, fallback, metering and an eval gate; read-only MCP agent ([M19](milestones/M19-ai-infrastructure.md), [M20](milestones/M20-ai-agents-mcp.md)) | AI tier under SLOs and budgets for months; S31/S32/S33 handled | ADR-023/024/025 defended; explain three read-only mechanisms; *L4 is the target for this JD* |

---

## 3. Target levels across the timeline

Month checkpoints match [Section 22](22-timeline.md). **Bold** marks where the JD needs you by the end.

| Skill | Month 8 (after M07) | Month 13 (after M12) | Month 18 (after M16) | Month 24 (after M22) |
|---|:--:|:--:|:--:|:--:|
| Linux | 3 | 4 | 4 | **4** |
| Networking | 3 | 4 | 4 | **5** |
| Python | 3 | 4 | 4 | **4** |
| Bash / PowerShell | 3 / 2 | 3 / 2 | 4 / 3 | **4 / 3** |
| Git | 3 | 4 | 4 | **4** |
| AWS | 3 | 4 | 4 | **5** |
| Azure | 0 | 1 | 3 | **4** |
| IAM | 3 | 4 | 4 | **5** |
| Terraform | 3 | 4 | 4 | **5** |
| Docker | 3 | 4 | 4 | **4** |
| Kubernetes | 1 | 4 | 4 | **5** |
| CI/CD | 3 | 4 | 4 | **5** |
| Observability | 1 | 4 | 4 | **4** |
| SRE | 1 | 4 | 4 | **5** |
| Capacity / HA / DR | 1 | 3 | 4 | **4** |
| Security | 2 | 3 | 4 | **4** |
| Governance | 1 | 2 | 4 | **4** |
| FinOps | 2 | 2 | 4 | **4** |
| Platform Engineering | 1 | 2 | 3 | **4** |
| System Design | 2 | 3 | 4 | **5** |
| Documentation | 3 | 4 | 4 | **5** |
| AI Infrastructure | 1 | 1 | 1 | **4** |

How to read it:

- **Month 13** is roughly a mid-level Cloud/DevOps engineer on AWS: L4 in the core skills.
- **Month 24** is the senior-target profile: L4 everywhere the JD requires, L5 in networking, IAM, Terraform, Kubernetes, CI/CD, SRE and system design.
- Azure is deliberately 0 until M13. That's ADR-002 working as designed, not a gap.
- AI reaching L4 only at month 24 is also deliberate. It's the last thing added and it sits on everything else.

## 4. The self-assessment protocol

Every revision week (R1–R9):

1. **Score each row honestly** in `docs/competency.md`, with the date and a one-line evidence link per level claimed.
2. **Score down when in doubt.** The purpose is to find gaps while there's still time to close them. A matrix that only ever goes up is lying.
3. **Prove one claim live.** Pick one skill at your claimed level and demonstrate it without notes: a lab from [Section 9](09-hands-on-labs.md), a scenario from [Section 10](10-failure-and-troubleshooting.md), or a whiteboard explanation. If you hedge, drop the level.
4. **Compare with the target table.** Anything two or more levels behind target goes into next month's plan. If you're behind on a never-cut milestone (M11, M17), extend the timeline rather than skipping.
5. **Get an outside check once a quarter.** Mentor review, a peer, or explaining it to a colleague. Teaching reveals L5 gaps faster than anything else.

Showing an interviewer a matrix you've honestly *downgraded* yourself on, with dates, is an unusually strong signal of engineering maturity.

→ Next: [Section 25 — Final Production Readiness Checklist](25-production-readiness-checklist.md)
