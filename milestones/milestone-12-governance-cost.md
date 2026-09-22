# Milestone 12 — Governance, Cost and Hybrid Control

*Phase 12 · ~3 weeks · Prerequisite: Milestones 6, 8*

> Covers three explicit JD phrases at once: *"security, governance and cost awareness"* and *"unify control across datacentres, networks, public and private cloud environments."*

**Learning topics**
* Management group hierarchy and policy inheritance; Azure Policy definitions, initiatives, assignments
* The effects — `Audit`, `Deny`, `DeployIfNotExists`, `Modify` — and why rollout is always audit-first
* Exemptions and the exception process; policy-as-code shifted left with `conftest`/OPA against the Terraform plan
* The Azure cost model; tagging as the foundation of allocation; budgets, alerts, anomaly detection
* Reserved instances and savings plans (know when they apply; they will not for you) · spot and its eviction trade-off · right-sizing from utilisation · idle-resource detection
* Unit economics: cost per request, per service, per environment; the FinOps cycle of inform → optimise → operate
* Azure Arc for servers and Kubernetes: what it projects and what it does not; hybrid identity and DNS as concepts; VPN vs ExpressRoute at the "what problem does it solve" level

**Build**
`platform/policy/` in Terraform: at least four policies — required tags, no public IPs on the workload subnet, allowed regions, HTTPS-only storage — as an initiative, rolled out audit-first then to deny. `docs/architecture/tagging-standard.md`. A cost-per-service report job reading the Cost Management API.

**Cloud resources**
Policy definitions, initiative and assignments · budgets with three thresholds and an action group · a Container Apps job for the monthly cost report writing to Blob · **Azure Arc agents on a local Linux VM and a local k3s cluster**, governed by the same policy initiative.

**DevOps work**
`conftest` in the Terraform pipeline so a policy violation fails the PR rather than the apply — then compare those two experiences deliberately.

**Operations exercise**
Manufacture a cost anomaly: leave a node pool scaled up over a weekend, or remove the Log Analytics cap and generate volume. **Detect it from your own alert**, investigate, and write it up as an incident with before/after numbers. This is the single most persuasive FinOps artifact you can own, because it is your own money.

**Security considerations**
Assign a `Deny` policy without an audit period, break your own pipeline, then fix it properly with an exemption that has an expiry — and document the exception process you just invented.

**Expected outcome**
Non-compliant resources cannot be created. Cost is attributable per service per environment. Two non-Azure resources are governed by the same policy set as your cloud resources.

**Evidence of mastery**
Explain `Audit` vs `Deny` vs `DeployIfNotExists` with a scenario for each. Show a policy that blocked a real deployment of yours. State your cost per environment per month and which component dominates. Explain what Arc gives you and, more importantly, what it does not.
