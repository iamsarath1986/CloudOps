# Milestone 6 — Everything as Code

*Phase 5 · ~4 weeks · Prerequisite: Milestones 3, 4, 5 · **The highest-leverage milestone in the roadmap***

**Learning topics**
* Declarative vs imperative, and why declarative requires state
* What Terraform state contains (including secrets), why it is encrypted, and why concurrent applies need locking
* Providers and version pinning; the resource graph; implicit vs explicit dependencies
* `count` vs `for_each` and why `for_each` is almost always correct
* `lifecycle`: `prevent_destroy`, `ignore_changes`, `create_before_destroy`
* Reading a plan critically — spotting a *replace* that will destroy data
* `terraform import` and `moved` blocks: refactoring without destruction
* Module design as API design: inputs, defaults, outputs, versioning as a contract
* Environment separation via directories and separate state, not workspaces — and being able to say why
* **Drift**: causes, detection, and the organisational choice between reconcile and adopt

**Build**
`platform/modules/`: `network`, `postgres`, `keyvault`, `storage`, and a first `workload` module. `platform/environments/dev/` and `prod/` as thin root modules that only pass variables — **all logic lives in modules**. A tagging module applied everywhere: `service`, `environment`, `owner`, `cost-centre`, `managed-by=terraform`.

**Cloud resources**
Everything from Milestone 3, recreated from code — then the hand-built originals destroyed. Plus Blob Storage for remote state with locking, and separate state files per environment.

**DevOps work**
`make up` and `make down` bring the whole environment up for a session and tear it down after. **Your budget and your IaC discipline are the same habit** — if teardown is inconvenient, you will not do it.

**Operations exercise**
Change a resource in the portal, `plan`, read the drift, reconcile it. Then the reverse: `import` an out-of-band resource. Then refactor a resource into a module using `moved`, without destroying it — this is the skill separating people who can *maintain* Terraform from people who can only write it. Then corrupt a copy of your state and recover it from blob versioning, writing the procedure down while it is fresh.

**Security considerations**
State contains secrets: the storage account is private, versioned, and access-controlled. Add `prevent_destroy` to the database after you have written a plan that would have destroyed it and caught yourself in review.

**Expected outcome**
Nothing in the subscription exists outside Terraform. `terraform destroy` followed by `terraform apply` reproduces the environment. `docs/runbooks/terraform-state-recovery.md` and `docs/adr/ADR-004-environment-separation.md` exist.

**Evidence of mastery**
Explain what state is and three distinct ways it corrupts. Explain why `for_each` is safer than `count` when removing a middle element. Given a plan output, identify the destructive changes before applying. Defend directory-based environment separation against workspaces.
