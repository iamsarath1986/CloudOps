# Milestone 3 — First Cloud Footprint, Built by Hand

*Phase 2 · ~3 weeks · Prerequisite: Milestone 2*

> **Everything here is rebuilt in Terraform in Milestone 6 and then deleted.** That is the design. Terraform written by someone who has never clicked the thing is Terraform they cannot debug. Use the free trial credit now, while destroy-discipline has not yet formed.

**Learning topics**
* Tenant → management group → subscription → resource group → resource, and which control attaches where
* Regions, availability zones, paired regions; what zone-redundancy costs and buys
* Resource providers, quotas and limits — a top-three cause of failed deployments
* The shared responsibility model, concretely, for a VM vs AKS vs a managed database
* Service SLAs and how composite SLAs multiply downward
* Naming and tagging conventions, decided now rather than retrofitted
* ARM as the API underneath the portal, the CLI and Terraform

**Build**
`cargo-api` deployed by hand to a single VM: Docker installed manually, TLS via Let's Encrypt, a public DNS record. Record every manual step in `docs/architecture/manual-build-log.md` as you go — that file is the specification for Milestone 6, and its length is the argument for IaC.

**Cloud resources**
Resource group · VNet with the subnets from your Milestone 2 plan · NSGs · VM with public IP · PostgreSQL Flexible Server reachable **only** from the app subnet · Blob Storage · Azure DNS zone · **a budget with 50/80/100% alerts, created before the second resource exists**.

**DevOps work**
Do one identical task three ways — portal, `az` CLI, Azure PowerShell. The PowerShell repetition is deliberate; the JD names it and you need reading fluency, not authoring fluency.

**Operations exercise**
Remove the NSG rule and diagnose the resulting timeout using Milestone 2's method, without looking at what you changed. Stop the database and watch `cargo-api` behave badly — hang, or retry forever. Fix it with a connection timeout. Trigger a quota error deliberately and read it properly.

**Security considerations**
The database has no public endpoint. The VM's NSG allows only 443 from the internet and 22 from your IP. Remove your own RBAC assignment on the resource group and experience an authorisation failure from the inside.

**Expected outcome**
A working public URL with valid TLS, backed by a private database. `docs/cost/month-01.md` puts your pricing-calculator estimate next to the actual bill and explains the difference — that gap is the lesson.

**Evidence of mastery**
Draw the Azure resource hierarchy and say what attaches at each level. Explain what you are responsible for in AKS that you are not in Container Apps. State your monthly cost, which line dominates, and why. Explain what a composite SLA does to a four-component system.
