# Section 13 — Terraform Learning Path

Terraform is the highest-leverage single skill in this roadmap. Every other cloud skill you claim is verified by one question: **is it in code?** This file takes you from a first `apply` to running Terraform the way a platform engineering team maintains it — versioned modules consumed by other teams, tested, scanned, policy-checked, drift-detected, and recoverable when state goes wrong.

> JD evidence: *"Strong automation experience using Infrastructure as Code (IaC), preferably **Terraform**"* · *"drive the adoption of Infrastructure as Code, CI/CD …"* · *"onboarding applications using governed and automated DevOps practices"* · *"building … automation solutions"*. For a *Senior* role, "strong" means you design module interfaces other people consume, not only write resources.

---

## 1. The six stages

| Stage | Name | Milestones | ≈ Months | Level at exit |
|---|---|---|:--:|:--:|
| 1 | First contact: the plan/apply model | [M05](milestones/M05-terraform.md) wk 1 | 5 | L2 |
| 2 | State, remote backends and locking | [M05](milestones/M05-terraform.md) wk 2 | 5 | L2→L3 |
| 3 | Modules, environments and multi-account | [M05](milestones/M05-terraform.md) wk 3–5 | 5–6 | L3 |
| 4 | Terraform in the pipeline: tests, scans, policy, cost | [M07](milestones/M07-cicd.md), [M09](milestones/M09-eks.md) | 7–10 | L3→L4 |
| 5 | Two clouds and operating it over time: drift, import, refactor, upgrades, recovery | [M12](milestones/M12-capacity-ha-dr.md)–[M15](milestones/M15-security-governance.md) | 13–17 | L4 |
| 6 | Terraform as a platform product: versioned modules, a contract, consumers you don't control | [M17](milestones/M17-golden-paths.md), [M18](milestones/M18-developer-platform.md) | 19–20 | L4→L5 |

Before Stage 1, you will have built the AWS environment **by hand** in [M04](milestones/M04-aws-core.md) and kept `docs/architecture/manual-build-log.md`. That log is the specification for Stage 1, and its length is the argument for IaC.

---

## 2. Stage by stage

### Stage 1 — The plan/apply model
**Concepts:** declarative vs imperative; providers and version constraints (`required_providers`, `~>`); resources, data sources; variables (types, validation, `sensitive`), locals, outputs; the dependency graph (implicit references vs `depends_on`); `plan` as a diff between desired state, recorded state and reality.
**Build:** the M04 VPC, subnets, route tables, SGs and the EC2 instance in one root module. Destroy the hand-built one.
**Must be able to:** read a plan and predict each `+`, `~`, `-/+` before applying.

### Stage 2 — State
**Concepts:** what state contains (resource IDs, attributes — **including secrets**, which is why it is encrypted and access-controlled); why concurrent applies need locking; remote state in **S3 with native locking (`use_lockfile = true`, Terraform ≥ 1.10 — DynamoDB locking is deprecated)**; bucket versioning as the recovery mechanism; `terraform_remote_state` vs data sources vs SSM parameters for cross-stack references (and why tight coupling through remote state hurts).
**Build:** S3 state bucket in `shared-services`, versioned, encrypted, Block Public Access, access limited to the Terraform roles.
**Must be able to:** explain three ways state becomes wrong (interrupted apply, out-of-band change, two writers without a lock).

### Stage 3 — Modules, environments, accounts
**Concepts:** module design as API design — minimal inputs, opinionated defaults, stable outputs, non-negotiables not exposed as variables; `count` vs **`for_each`** (and why removing the middle element of a `count` list destroys the wrong thing); `lifecycle` (`prevent_destroy`, `ignore_changes`, `create_before_destroy`, `replace_triggered_by`); **environment separation by directory and state, not workspaces** (ADR-005); provider aliases and cross-account `assume_role`; tagging via `default_tags`.
**Build:** `platform/modules/aws-network`, `aws-rds`, `aws-account`; `platform/live/aws/<account>/<env>/<component>` roots; the Organizations structure (ADR-003) in code; `make up` / `make down`.
**Must be able to:** defend directories-over-workspaces against a sceptical reviewer.

### Stage 4 — Terraform in the pipeline
**Concepts:** plan on PR, apply on merge, approval for prod; OIDC to AWS (and later Entra) — no keys; the **plan artefact** applied is the plan reviewed; static analysis (`fmt`, `validate`, **tflint**); config scanning (**trivy config** / **checkov**); policy on the plan JSON with **conftest (OPA)**; cost diff with **Infracost**; `terraform test` for module behaviour.
**Build:** the `tf-plan-apply` reusable workflow ([M07](milestones/M07-cicd.md)); the EKS module and its add-ons ([M09](milestones/M09-eks.md)); Kubernetes/Helm providers used *only* for cluster bootstrap (and why you stop there — see §5).
**Must be able to:** explain what each gate prevents; if you cannot, delete the gate.

### Stage 5 — Operating it over time
**Concepts:** **drift** — detect (nightly `plan -detailed-exitcode`), then decide: reconcile or adopt; `import` blocks for out-of-band resources; `moved` blocks to refactor without destruction; `removed` blocks to stop managing without destroying; provider major-version upgrades; state surgery (`state mv`, `state rm`) and when never to use it; `-target` as an emergency tool only; multi-cloud state: **Azure state in Azure Blob with lease locking** — state lives with its cloud (ADR-005).
**Build:** `drift-detect` workflow opening an issue with the diff; Azure landing zone, AKS and Key Vault modules ([M13](milestones/M13-azure-core.md), [M14](milestones/M14-aks-multicluster.md)); the pilot-light second region ([M12](milestones/M12-capacity-ha-dr.md)) as the *same* modules with different inputs; policy code (SCPs, Azure Policy) in Terraform ([M15](milestones/M15-security-governance.md)).
**Must be able to:** recover from each drill in §6 using only your runbook.

### Stage 6 — Terraform as a product
**Concepts:** semantic versioning as a promise; Git-tag module releases (`?ref=v2.1.0`) and why unpinned refs are an outage waiting to happen; changelogs and **migration guides**; deprecation windows; the `workload` module as the golden path's infrastructure contract behind `workload.yaml` (ADR-021); module version spread across consumers as a debt metric; account vending as a module invoked by a PR from the portal.
**Build:** `platform/modules/workload` v1 → a deliberate breaking v2 with a migration guide, consumers migrated ([M17](milestones/M17-golden-paths.md)); `aws-account` vending via portal-generated PRs ([M18](milestones/M18-developer-platform.md)).
**Must be able to:** release a breaking change without breaking a consumer.

---

## 3. Topic coverage (brief Step 8)

| Topic | Stage | Where | Evidence you keep |
|---|:--:|---|---|
| Providers | 1 | M05 | Pinned versions, a lock file committed |
| Resources | 1 | M05 | M04 estate in code |
| Variables | 1 | M05 | Typed, validated, documented |
| Outputs | 1 | M05 | Only what consumers need |
| State | 2 | M05 | Explanation in `docs/runbooks/terraform-state-recovery.md` |
| Remote state | 2 | M05 | S3 backend in `shared-services`; Azure Blob backend in M13 |
| Locking | 2 | M05 | Native S3 lockfile; a lock-contention drill (S05) |
| Modules | 3 | M05, M17 | Module READMEs with interface docs |
| Environments | 3 | M05 | ADR-005 |
| Dependency management | 1, 3 | M05 | Graph reading; no gratuitous `depends_on` |
| Secrets | 2, 4 | M05, M09 | No secrets in variables/state where avoidable; RDS-managed master password; ESO for runtime |
| Reusable modules | 3, 6 | M05, M17 | `workload` module, versioned |
| Testing | 4 | M07, M17 | `terraform test` files; see §4 |
| Linting | 4 | M07 | tflint in CI |
| Security scanning | 4 | M07, M15 | trivy/checkov with triaged exceptions |
| CI/CD integration | 4 | M07 | `tf-plan-apply` reusable workflow |
| Drift detection | 5 | M07, M12 | Nightly workflow; S06 records |
| Import | 5 | M05, M13 | `import` blocks for a console-created resource |
| Lifecycle | 3 | M05 | `prevent_destroy` on data stores; S07 |
| State recovery | 2, 5 | M05, M12 | Drill records; runbook |

---

## 4. The testing pyramid for IaC

```
                 ┌──────────────────────────┐
                 │  End-to-end: make up →   │  weekly / before release — real cloud, real cost
                 │  smoke test → make down  │
               ┌─┴──────────────────────────┴─┐
               │ Integration: terraform test   │  per module release — applies to a sandbox account,
               │ with `command = apply`        │  asserts outputs, destroys
             ┌─┴──────────────────────────────┴─┐
             │ Plan-level: conftest on plan JSON, │  every PR — tags present, no public IPs in
             │ Infracost diff, terraform test     │  workload subnets, no destroy of data stores
             │ with `command = plan`              │
           ┌─┴────────────────────────────────────┴─┐
           │ Static: fmt, validate, tflint,           │  every commit — seconds, free
           │ trivy/checkov config scan                │
           └──────────────────────────────────────────┘
```

Two rules: fast and free at the bottom, slow and paid at the top; and every level must have **failed at least once for a real reason** in your history — a gate that has never caught anything is unproven.

---

## 5. The final repository structure a platform team would maintain

```
platform/
├── modules/                           # versioned by Git tag; each has README, examples/, tests/
│   ├── aws-account/                   # Organizations account + baseline (vending)
│   ├── aws-network/                   # VPC, subnets, NAT, endpoints, flow logs
│   ├── aws-eks/                       # cluster, access entries, Pod Identity, Karpenter prerequisites
│   ├── aws-rds/
│   ├── azure-network/                 # hub-spoke, private DNS zones
│   ├── azure-aks/
│   ├── azure-keyvault/
│   ├── azure-openai/                  # deployment, private endpoint, diagnostics, budget
│   ├── k8s-baseline/                  # bootstrap only: Argo CD install + the app-of-apps root
│   ├── observability/                 # AMP workspace, buckets, alert routing
│   └── workload/                      # the golden-path contract (ADR-021)
├── live/                              # thin roots: provider config + module calls + inputs only
│   ├── aws/
│   │   ├── management/global/organization/
│   │   ├── shared-services/global/{state,ecr,observability}/
│   │   ├── workloads-dev/dev/{network,eks,rds,workloads}/
│   │   └── workloads-prod/prod/{network,eks,rds,workloads}/
│   └── azure/
│       ├── runway-platform/shared/{management-groups,hub-network,policy}/
│       └── runway-workloads/{dev,prod}/{spoke,aks,keyvault,openai}/
├── policy/                            # SCPs, Azure Policy, conftest rego, Kyverno (applied via GitOps)
└── tests/                             # cross-module end-to-end smoke tests
```

**Why it is shaped like this:**
- **Small states, small blast radius.** One component per state. A bad `eks` apply cannot touch `rds`.
- **Roots contain no logic.** If you need a `for_each` in a root, it belongs in a module.
- **Terraform stops at the cluster boundary.** Terraform creates the cluster and installs Argo CD; everything inside the cluster is GitOps ([M14](milestones/M14-aks-multicluster.md)). Two tools reconciling the same object is a guaranteed incident.
- **Order is explicit.** `organization → state → network → eks|rds → workloads` is documented in `platform/live/README.md` and encoded in `make up`.

Terragrunt, Terraform Stacks and HCP Terraform are legitimate alternatives. Know what each solves (DRY backends, orchestration, managed runs and policy) and record in ADR-005 why plain Terraform plus a `Makefile` is enough here.

---

## 6. State and failure drills

Run each at least once; the record goes in `docs/incidents/`, the procedure in `docs/runbooks/terraform-state-recovery.md`.

| Drill | How to cause it | What you must learn | Scenario |
|---|---|---|:--:|
| Stale lock | Kill an apply mid-run | Read the lock info; confirm nobody is running; `force-unlock` — and why the confirmation step matters | S05 |
| Corrupted state | Upload a truncated state object | Restore the previous S3 object version; verify with a no-op plan | S05 |
| Lost state | Delete the state object (in a copy of dev) | Rebuild state with `import` blocks from reality | S05 |
| Console drift | Change an SG rule in the console | Detect via nightly plan; ask *who and why* before reconciling | S06 |
| Destructive plan | Change a forces-new attribute on RDS | Spot `-/+` in review; `prevent_destroy`; a conftest rule that fails the PR | S07 |
| Refactor without destruction | Move a resource into a module | `moved` block; plan shows zero changes | — |
| Provider major upgrade | Bump AWS or AzureRM provider major | Read the upgrade guide; plan in dev; fix deprecations | — |
| Credential expiry mid-apply | Shorten the role session | Partial apply; re-plan; understand idempotency | S04 |

**Completion test for the whole track:** delete the dev environment, then rebuild it from an empty account state using only the repository and the runbooks, and a plan against the result shows **no changes**.

---

## 7. Checkpoints

| After | Checkpoint (without notes) |
|---|---|
| Stage 2 | Explain what is in your state file and who can read it; recover a corrupted state from bucket versioning. |
| Stage 3 | Walk a reviewer through a module interface and justify every variable; `make down && make up` rebuilds dev. |
| Stage 4 | A PR that would open a public IP on a workload subnet fails *before* apply, with a readable message. |
| Stage 5 | Reconcile drift after finding out why it happened; refactor with `moved` and a zero-change plan. |
| Stage 6 | Ship `workload` v2 with a breaking change, a migration guide, and every consumer migrated. |

**Terraform Associate** fits at R2 (~month 8), after Stage 4 begins — see [Section 21](21-certification-strategy.md).

---

## 8. Resources

1. ***Terraform: Up & Running*, 3rd ed (Brikman)** — ch. 3 (state), ch. 4 (modules), ch. 5 (loops/conditionals, and the `count` trap), ch. 8 (production-grade code), ch. 9 (testing).
2. **Terraform documentation** — "State" (purpose, locking, sensitive data), "S3 backend" (`use_lockfile`), "Refactoring" (`moved`), "Import blocks", "Tests" (`terraform test`), "Module development: composition".
3. **HashiCorp tutorials** — "Terraform in automation" and "Manage resource drift".
4. **terraform-best-practices.com (Babenko)** — "Key concepts" and "Code structure" for alternative layouts to compare with §5.
5. **Open Policy Agent — conftest docs** and **tflint rules** for your providers.

→ Next: [Section 14 — Docker & Kubernetes Learning Path](14-docker-kubernetes-learning-path.md)
