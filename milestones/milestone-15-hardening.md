# Milestone 15 — Hardening and Close-Out

*Phases 16 and 17 · ~4.5 weeks · Prerequisite: everything*

**Learning topics**
* STRIDE threat modelling over your own architecture; trust boundaries as the organising idea
* Defence in depth applied concretely across identity, network, workload, data and supply chain
* Egress control, and why outbound is the direction that matters for exfiltration
* Secret rotation vs revocation; supply chain — pinned digests, SBOM, dependency scanning, provenance
* Kubernetes hardening: pod security standards, dropped capabilities, read-only root filesystem, no privileged containers
* WAF rule sets and the false-positive problem; detection vs prevention and how to budget attention between them
* Logging for security rather than debugging: who did what, when, from where
* *(Optional)* Cloud portability: where models genuinely differ — IAM policy documents vs RBAC assignments, security groups vs NSGs plus route tables, workload identity across providers

**Build**
`docs/security/threat-model.md` with a controls matrix mapping each threat to a control and its evidence — including the entries that say "accepted risk", which are the honest ones. A secret rotation procedure and schedule. *(Optional)* `platform/modules/network-aws/` — one module ported to AWS with a written comparison — and one Azure DevOps pipeline definition.

**Cloud resources**
Application Gateway + WAF in a timeboxed hardening window: run in detection mode against your load test, tune the false positives, then enable prevention. Egress restriction with an allow-list.

**DevOps work**
Dependency and image scanning as pipeline gates with a documented triage policy and a vulnerability SLA per severity. Then a final pass over the whole repository for the portfolio: README, architecture diagrams, ADR index, and the numbers.

**Operations exercise**
**The leaked credential drill:** assume a pipeline credential is public. Contain, rotate, assess exactly what was accessible, prevent recurrence, and write it up as an incident with a full timeline. Then rotate every secret in the system and time it — you will find the ones you forgot. Then break your own service with a WAF rule and diagnose a 403 that is not the application's fault.

**Security considerations**
Apply pod security standards and read-only root filesystems, break your own workloads, and fix them properly rather than by exempting them. Enable egress restriction and discover every outbound dependency you did not know you had.

**Expected outcome**
A system whose security posture you can walk through threat by threat, including the risks you knowingly accepted and why — and a repository that reads as the work of someone who *operates* a production platform rather than someone who once deployed one.

**Evidence of mastery**
Walk through your threat model and defend each accepted risk. Explain your egress posture. Explain what an attacker who compromised one pod could and could not reach, and why. *(Optional)* Explain the difference between Azure RBAC and AWS IAM at the model level rather than the syntax level, and say what you would and would not need to learn to be productive on AWS in a month.
