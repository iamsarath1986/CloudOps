# Milestone M05 — Terraform: From First Apply to Team Practice

*Phase 05 · ~5 weeks · Months ~4.7–6 · Prerequisites: M04 · Unlocks: M07 (CI/CD), M09 (EKS), and every resource from here on · Track: [Section 13](../13-terraform-learning-path.md)*

## Why this phase exists

The JD asks for *"strong automation experience using Infrastructure as Code (IaC), preferably Terraform"* and for onboarding applications *"using governed and automated DevOps practices"*. Terraform is the highest-leverage skill in this roadmap: every other cloud claim you make is verified by whether it is in code, and the golden path in M17 is Terraform modules consumed by other teams.

It comes after the hand-built AWS environment because you now know what every resource *does*. By the end of this phase, **nothing in the lab exists outside code**, the environment can be destroyed and rebuilt with one command, and Terraform runs the way a platform team runs it — structured, tested, scanned and state-safe. CI integration arrives in M07; this phase makes it ready for that.

## Concepts to learn

Progress in four stages. Do not skip a stage because the next one looks more interesting.

**Stage 1 — the core model**
- Declarative vs imperative; why declarative needs **state**; the plan/apply cycle and the three-way comparison (config, state, real world).
- **Providers**: what a provider is (a plugin translating HCL to API calls), `required_providers`, version constraints (`~>`), the dependency lock file (`.terraform.lock.hcl`) and why it is committed.
- **Resources and data sources**: managed vs read-only; resource addresses.
- **Variables** (types, validation blocks, `sensitive`), **locals**, **outputs** (and output `sensitive`).
- **Dependencies**: the resource graph, implicit dependencies via references, `depends_on` as a last resort, why parallelism works.
- `count` vs `for_each` — and why removing a middle element from a `count` list destroys and recreates everything after it.
- Expressions: `for` expressions, conditionals, `dynamic` blocks (sparingly), functions (`cidrsubnet` — you will use it for ADR-004).

**Stage 2 — state and safety**
- **What state contains**, including secrets in plaintext — so it must be encrypted, access-controlled and never committed.
- **Remote state** in S3 with **native S3 locking (`use_lockfile = true`, Terraform ≥ 1.10)**; DynamoDB-based locking is deprecated. Bucket versioning for recovery; KMS encryption; bucket policy restricting access to the pipeline role and break-glass.
- **Locking**: what happens with concurrent applies; `force-unlock` and when it is safe.
- Reading a plan critically: `+`, `-`, `~`, and **`-/+` (replace)** — spotting the change that destroys data. Attributes that force replacement.
- **Lifecycle**: `prevent_destroy`, `create_before_destroy`, `ignore_changes` (and its abuse), `replace_triggered_by`; preconditions and postconditions.
- **`import` blocks** for adopting existing resources; **`moved` blocks** for refactoring without destruction; `removed` blocks for forgetting without destroying; `terraform state mv/rm/list/show` for surgery.
- **Drift**: its causes (console changes, other automation, provider defaults), detection with `plan -detailed-exitcode` and `-refresh-only`, and the organisational question — reconcile or adopt?
- **State recovery**: restore a previous version from S3 versioning; `state pull/push` and why `push` is dangerous; the serial and lineage fields.

**Stage 3 — structure like a team**
- **Modules** as APIs: minimal inputs, opinionated defaults, stable outputs, validation, a README with an example. Non-negotiables (tags, encryption) are not variables.
- **Versioning** modules by Git tag; semver as a promise to consumers; changelog.
- **Environments**: separate root modules and separate state per environment/component (ADR-005) — not workspaces. Small states = small blast radius and faster plans; the cost is wiring between them (`terraform_remote_state` or data-source lookups, and why lookups by tag are often better than coupling to another state).
- **Multi-account providers**: provider aliases with `assume_role` into `workloads-dev`, `workloads-prod`, `shared-services`.
- **Secrets**: never as literal variables; read from Secrets Manager via data source only where unavoidable (it lands in state); prefer letting the service generate and store its own (e.g. RDS `manage_master_user_password`). Ephemeral values/resources in newer Terraform versions — know they exist and what problem they solve.

**Stage 4 — quality gates**
- `terraform fmt -check`, `validate`.
- **Linting** with `tflint` (+ the AWS ruleset): invalid instance types, deprecated syntax, naming.
- **Security scanning** with `trivy config` (or checkov): public buckets, unencrypted volumes, open security groups.
- **Testing** with `terraform test` (`.tftest.hcl`): plan-mode unit tests for module logic, apply-mode tests against the sandbox account for integration; mocks for providers where useful. Terratest as the Go alternative — know when you would pick it.
- **Policy as code** on the plan JSON with conftest/OPA (introduced here with two rules; fully developed in M15).
- **Cost** visibility with Infracost on a plan.
- CI integration (implemented in M07): plan on PR with the plan as a comment, apply on merge with approval, no applies from laptops.

## Architecture decisions in this phase

### ADR-005 — Terraform layout, state backends and environment separation
- **Problem:** How should Terraform be organised so it survives two clouds, six AWS accounts, several environments and eventually consumer teams?
- **Options:** (a) one root per environment with everything in it; (b) workspaces per environment; (c) Terragrunt; (d) plain Terraform with `live/<cloud>/<account>/<env>/<component>` roots and versioned modules.
- **Decision:** (d). `platform/modules/` holds versioned modules (`aws-network`, `aws-rds`, `aws-alb-service`, `aws-account-baseline`, `tags`); `platform/live/aws/workloads-dev/dev/network`, `.../dev/data`, `.../dev/app` are thin roots that only pass variables. State for AWS lives in an S3 bucket in `shared-services` with native lockfile locking and versioning; Azure state will live in Azure Blob (M13) — state stays with its cloud.
- **Trade-offs:** more roots to wire and apply in order (captured in the `Makefile`); Terragrunt would remove boilerplate but hides mechanics you must learn first — revisit in M17. Workspaces rejected: same code and backend for all environments, easy to apply to the wrong one, no per-environment access control on state.

## Hands-on labs

1. **Codify the manual build.** Rebuild M04's environment entirely in Terraform in a fresh VPC, cut traffic over, then destroy the hand-built one. Diff `manual-build-log.md` against the code: that page is the argument for IaC.
2. **`count` vs `for_each`.** Create subnets with `count`, remove the middle one, read the plan. Refactor to `for_each` with `moved` blocks so the plan shows **no changes**.
3. **Locking.** Two terminals, two applies. Watch the lock. Kill one mid-apply, see the stale lock, then decide whether `force-unlock` is safe and why.
4. **Drift.** Change a security group rule in the console. Detect it with `plan -detailed-exitcode`. Reconcile it. Then do the opposite: create a resource by hand and `import` it with an `import` block.
5. **Refactor into a module** without destroying anything, using `moved`. This separates people who can maintain Terraform from people who can only write it.
6. **State recovery.** In a copy, corrupt `terraform.tfstate`; restore the previous version from S3 versioning; confirm `plan` is clean. Write the procedure while it is fresh.
7. **Destructive plan.** Change the RDS identifier (forces replacement). Read `-/+`, stop, add `prevent_destroy`, and write the review checklist item that would have caught it.
8. **Provider upgrade.** Bump the AWS provider a major version in a branch; read the upgrade guide; fix deprecations. Note which changes would have been dangerous.
9. **Tests.** Write `terraform test` files for `aws-network`: plan-mode assertions that subnets land in three AZs and CIDRs match ADR-004; one apply-mode test in `sandbox` that asserts the NAT route exists, then auto-destroys.
10. **Scanners.** Run `tflint` and `trivy config`; fix or justify every finding with an inline, reviewed exception.

## Project implementation

- `platform/modules/`: `tags`, `aws-account-baseline` (budgets, CloudTrail settings, default EBS encryption, S3 Block Public Access), `aws-network`, `aws-rds`, `aws-alb-service` (ALB, target group, listener, certificate, DNS record), `aws-ec2-service` (launch template + ASG running the systemd unit), `aws-sqs-queue` (queue + DLQ + alarms). Each with a README, example, variables with validation, outputs, and tests.
- `platform/live/aws/<account>/<env>/<component>/` roots for `shared-services` (state bucket, ECR placeholder) and `workloads-dev` (network, data, app); `workloads-prod` roots exist but are applied only in soak windows.
- The Organization, OUs, SCPs and Identity Center permission sets imported into `platform/live/aws/management/global/org/` with `import` blocks — the hand-built structure becomes code without recreation.
- Tag module applied everywhere; `managed-by=terraform`.
- `Makefile`: `make up ENV=dev` (applies roots in dependency order), `make down ENV=dev` (destroys in reverse, skipping state and org), `make plan`, `make tf-check` (fmt, validate, tflint, trivy, tests).
- `docs/runbooks/terraform-state-recovery.md`, `docs/runbooks/terraform-drift.md`.
- `docs/adr/ADR-005-terraform-layout-and-state.md`.
- `docs/platform/terraform-review-checklist.md` — what a reviewer must check in every plan (replacements, IAM widening, public exposure, deletion of stateful resources, cost delta).

## Engineering problems to hit deliberately

| Problem | How to cause it | What you should learn |
|---|---|---|
| Stale state lock | Kill an apply mid-run | Locks protect state; force-unlock only when you *know* nothing is running |
| Corrupted state | Hand-edit state in a copy | Versioned backends make state recoverable; `push` can make it worse |
| Console drift | Edit an SG by hand | Drift has an owner and a reason; ask before reconciling |
| Accidental replacement | Change a forces-new attribute | Read every `-/+`; `prevent_destroy` on stateful resources |
| `count` index shift | Remove a middle element | `for_each` with stable keys |
| Secret in state | Pass a DB password as a variable | State is sensitive; prefer service-managed secrets |
| Provider breaking change | Major-version bump | Pin, read upgrade guides, upgrade deliberately |
| Cross-root coupling | Hard-code another root's output | Loose coupling via data sources and tags |

## Troubleshooting exercises

- **S05 — "Terraform says it cannot acquire the state lock, and nobody is running it."** Decide whether it is safe to unlock, prove it, act. Then: "state is corrupted after an interrupted apply" — recover from versioning. Full incident write-up: `docs/incidents/INC-003-terraform-state.md`.
- **S06 — "The nightly plan shows changes nobody made."** (Run the drift check by hand for now; M07 automates it.) Find *who* changed it in CloudTrail and *why* before deciding reconcile vs adopt.
- **S07 — "The PR plan wants to replace the database."** Explain which attribute forced it and the safe way to make the change (e.g. snapshot, blue/green, or accept a different attribute).
- **"`make up` works on Tuesday and fails on Saturday."** Hidden cause: an unpinned provider or module minor version changed behaviour. Find it from the lock file history.

## Deliverables

- [ ] Every M04 resource recreated in Terraform; hand-built resources deleted; Org/SCP/Identity Center imported
- [ ] `platform/modules/*` with READMEs, validations, outputs and `terraform test` coverage
- [ ] `platform/live/aws/...` roots with remote state (S3, `use_lockfile`, versioning, KMS)
- [ ] `make up` / `make down` / `make tf-check`
- [ ] Two conftest rules on plan JSON (required tags; no `0.0.0.0/0` ingress except on the ALB)
- [ ] `docs/runbooks/terraform-state-recovery.md`, `docs/runbooks/terraform-drift.md`, `docs/platform/terraform-review-checklist.md`
- [ ] ADR-005
- [ ] `docs/incidents/INC-003-terraform-state.md`

## Completion criteria

- `make down && make up ENV=dev` rebuilds the full dev environment in under 25 minutes, and the API passes the smoke test afterwards (measured, twice).
- `terraform plan` on every root shows no changes immediately after apply (no perpetual diffs).
- Given five plan excerpts, identify every destructive change in under five minutes.
- Refactor a resource into a module and between roots with zero replacements, using `moved`/`import`/`removed`.
- Recover a deliberately corrupted state from S3 versioning following your own runbook, without improvising.
- Explain: what is in state and why it is sensitive; three ways state gets corrupted; why `for_each` is safer than `count`; why you rejected workspaces; how native S3 locking works and what replaced DynamoDB locking.
- `tflint`, `trivy config` and `terraform test` all pass; every suppression has a written justification.

## Mentor checkpoints

- **Design review:** bring the module interfaces (README first, no code) and ADR-005. Expect: "Which inputs are really implementation details?", "What happens when prod needs a different NAT strategy from dev?"
- **Code review:** the `aws-network` module and its tests. The mentor checks for leaky inputs, missing validation, outputs consumers should not depend on, and untested logic.
- **The failure I'll inject after you think you're done:** a state problem that appears mid-apply in a shared root. You will need to decide under time pressure what is safe.

## Resources

- Yevgeniy Brikman, *Terraform: Up & Running* (3rd ed.) — ch. 3 (state), ch. 4 (modules), ch. 5 (tips and tricks: loops, conditionals, zero-downtime), ch. 8 (production-grade code), ch. 9 (testing).
- Terraform docs — "State" (purpose, locking, remote state, sensitive data), "Backend type: s3" (the `use_lockfile` section), "Refactoring" (`moved`), "Import" (`import` blocks), "Tests" (`terraform test`), "Lifecycle meta-argument".
- AWS provider "Upgrade Guide" for your major version — read before bumping.
- terraform-best-practices.com (Anton Babenko) — "Key concepts" and "Code structure".
- HashiCorp tutorial "Manage resources in Terraform state" (state `mv`/`rm`/`import` hands-on).

## Estimate vs actual

| | Hours | Cost |
|---|---|---|
| Planned | 50 | $20–35 |
| Actual | | |
| Variance note | | |
