# Milestone M07 — CI/CD and Release Engineering (ECS Fargate Target)

*Phase 07 · ~4 weeks · Months ~6.8–7.8 · Prerequisites: M05, M06 · Unlocks: M09 (EKS), M11 (change management), M17 (reusable workflows become the golden path) · Track: [Section 15](../15-cicd-learning-path.md)*

## Why this phase exists

The JD asks for *"strong experience with DevOps practices, including Git, CI/CD pipelines, automated testing, automated deployment"* and asks you to *"own and manage changes within the technology area"*. This phase turns Runway from "infrastructure I apply" into **a system that changes safely through a pipeline** — tested, scanned, signed, promoted by digest, rolled back in measured seconds, and leaving an auditable change record behind.

It comes after Terraform and containers because a pipeline needs both an artefact to build and infrastructure to deploy onto. The first container runtime is **ECS Fargate** — a deliberately simpler target than Kubernetes, so you learn release engineering without also learning an orchestrator. In M09 you migrate from ECS to EKS, which is itself a change you will manage.

## Concepts to learn

- **The pipeline as a product** with users (developers) and an SLA (time to feedback). A slow pipeline gets routed around.
- **Build once, promote by digest** — the single most important idea here: the artefact tested in dev is bit-identical to the one in prod. Never rebuild per environment.
- **Stage ordering:** fast and cheap first (format, lint, types, unit tests), slow and expensive later (integration, image build, scan, deploy, smoke, performance).
- **The automated testing pyramid for Runway:**
  - *Unit* — pure logic, milliseconds.
  - *Contract* — the OpenAPI document is the contract; generated schema must match committed `openapi.yaml`; consumer expectations tested (Schemathesis or similar property-based API testing).
  - *Integration* — testcontainers PostgreSQL and a local SQS emulator (or LocalStack) in CI.
  - *Infrastructure* — `terraform test`, tflint, trivy config, conftest (from M05).
  - *Smoke* — post-deploy, against the real environment (`runway-tools smoke`).
  - *Performance gate* — a short k6 run with a p95 threshold (full capacity work in M12).
  - Flaky tests: quarantine policy, never "retry until green".
- **OIDC federation from GitHub Actions to AWS:** the IAM OIDC provider, a role whose trust policy restricts `sub` (repo, branch, environment) and `aud`. No AWS keys in GitHub — ever.
- **Environments, protection rules and approvals;** secrets scoped per environment (there should be almost none).
- **Deployment strategies:** recreate, rolling, blue/green (ECS with CodeDeploy or ECS native blue/green), canary — risk, cost and rollback speed of each.
- **ECS concepts:** clusters, task definitions (immutable revisions), services, deployment circuit breaker with automatic rollback, minimum/maximum healthy percent, ALB target group health, task IAM role vs execution role.
- **Rollback as a capability:** redeploy previous task definition by digest; measure it. Roll forward vs roll back.
- **Database migrations in pipelines:** migration as a separate step before code; **expand → migrate → contract**; never a backward-incompatible migration in the same release as the code that needs it.
- **Plan-on-PR / apply-on-merge** for Terraform, with the plan posted as a PR comment and prod applies behind approval.
- **Reusable workflows and composite actions** — the mechanism the golden path is later built on.
- **Supply chain in CI:** cosign keyless signing with the workflow's OIDC identity, SBOM attestation, dependency updates via Renovate or Dependabot.
- **Change management groundwork:** a change is any modification to production. Standard (pre-approved, low risk, e.g. a routine deploy through the pipeline), normal (assessed and approved), emergency (expedited, reviewed afterwards). Every prod deployment produces a **change record** automatically: what, why, who approved, risk, the digest, test evidence, rollback plan, and outcome. DORA metrics: deployment frequency, lead time, change failure rate, time to restore. M11 builds the full process on top of this.

## Architecture decisions in this phase

### ADR-007 — Release model: build once/promote by digest; ECS Fargate as first runtime; rolling vs blue/green
- **Problem:** Runway needs a safe, repeatable path from merged commit to production, with a runtime simple enough to learn release engineering on.
- **Options (runtime):** EC2/ASG (from M05) with in-place deploys; ECS on EC2; **ECS Fargate**; EKS now.
- **Options (strategy):** rolling with deployment circuit breaker; blue/green with traffic shift; canary.
- **Decision:** ECS Fargate for `cargo-api`, `eta-worker` and `carrier-sim`; rolling deploys with the ECS deployment circuit breaker and automatic rollback in dev; **blue/green** for `cargo-api` in prod (fast rollback by switching target groups). Images promoted by digest from dev to prod. Migrations run as a one-off ECS task before the service update, with expand–contract enforced by review.
- **Trade-offs:** Fargate costs more per vCPU than EC2 and gives less control, but removes node management while you learn releases; EKS deferred to M09 so orchestration and release engineering are not learned at once (you will write ADR-008 on why you moved). Blue/green doubles capacity briefly (cost) in exchange for near-instant rollback.

## Hands-on labs

1. **Fail at every stage.** Make the pipeline fail by lint, type, unit, contract, integration, scan, smoke and perf threshold — one at a time. Each failure must stop the pipeline at the right stage with an actionable message.
2. **The rebuild trap.** Rebuild the image in the prod job with a floating dependency and watch dev and prod diverge. Fix by promoting the digest.
3. **OIDC.** Configure the GitHub OIDC provider and a deploy role. Then break the `sub` condition (wrong branch) and read the failure. Then try to use the role from a fork's PR and confirm it is refused.
4. **Time a rollback.** Deploy a broken version to dev; let the circuit breaker roll back automatically; then do a manual rollback in prod blue/green. Record seconds from decision to recovered. That number goes in the README.
5. **Expand–contract.** Add a nullable column, deploy code writing both, backfill, deploy code reading the new, drop the old — four releases. Then do the naive single-release version in dev and break it on purpose (S11).
6. **Green but down.** Deploy a version whose readiness depends on a dependency that is down; make the pipeline report success. Then fix the pipeline so it waits for service stability *and* runs the smoke test (S10).
7. **Terraform in the pipeline.** Plan on PR (plan as comment, Infracost delta, conftest results), apply on merge for dev, approval for prod. No applies from laptops from now on (break-glass documented).
8. **Sign and verify.** Sign images keylessly with cosign in CI; verify the signature and the SBOM attestation from your laptop against the workflow identity.
9. **Change record.** Generate a change record for a prod deploy automatically; then simulate a failed change and fill in the outcome and a short post-implementation review.

## Project implementation

- `platform/modules/aws-ecs-service` — Fargate service, task definition, task/execution roles, ALB target groups (two for blue/green), autoscaling, log group, deployment circuit breaker. Replaces `aws-ec2-service` for workloads.
- ECS cluster per environment in `workloads-dev` and `workloads-prod`; EC2/ASG deployment retired (record it in the EOL register as the first retirement).
- GitHub OIDC provider and roles per repo/environment via Terraform (`platform/live/aws/<account>/global/ci-oidc`).
- Reusable workflows in `.github/workflows/`:
  - `build-sign-push.yml` — test pyramid stages, build, Trivy scan, Syft SBOM, cosign sign + attest, push to ECR by digest, output the digest.
  - `deploy.yml` — inputs: service, environment, digest; runs migrations task, updates the ECS service, waits for stability, runs smoke test, rolls back on failure; emits deployment events.
  - `tf-plan-apply.yml` — fmt/validate/tflint/trivy/`terraform test`/conftest/Infracost on PR; apply on merge; prod behind environment approval.
  - `drift-detect.yml` — nightly `plan -detailed-exitcode` on all roots; opens or updates an issue labelled `drift` (S06 automated).
  - `change-record.yml` — creates `docs/changes/CHG-NNNN.md` (or an issue labelled `change`) for every prod deploy and apply: change type, risk, approver, digest/plan, test evidence links, rollback plan, outcome.
- Thin caller workflows for `cargo-api`, `eta-worker`, `carrier-sim` — each a few lines calling the reusable ones.
- Environments `dev` (auto-deploy on merge) and `prod` (required reviewer, deploy window noted).
- Renovate (or Dependabot) for base image digests, Python dependencies, GitHub Actions and Terraform providers.
- `docs/runbooks/failed-deployment.md`, `docs/runbooks/rollback.md`, `docs/runbooks/ci-break-glass.md`.
- `docs/operations/change-types.md` — standard / normal / emergency definitions for Runway (expanded in M11).
- `docs/adr/ADR-007-release-model.md`.
- README gains the first numbers: **rollback time**, **lead time for change**, **pipeline duration to prod**.

## Engineering problems to hit deliberately

| Problem | How to cause it | What you should learn |
|---|---|---|
| Deploy reports green, service down | No stability wait / smoke test | Pipelines must verify outcomes, not just actions |
| Dev and prod differ | Rebuild per environment | Build once, promote by digest |
| Migration breaks running version | Drop/rename in the same release | Expand–contract; migrations are changes too |
| OIDC trust failure | Wrong `sub` condition | Trust policy conditions are the security boundary |
| Flaky integration test | Time-dependent assertion | Quarantine and fix; never retry-until-green |
| Secret needed in CI | "Just add an access key" | OIDC and service-managed secrets remove the need |
| Pipeline too slow | No caching, serial stages | Speed is a feature; measure it |

## Troubleshooting exercises

- **S10 — "The pipeline says the release succeeded; users are getting 503s."** Diagnose with ALB target health, ECS events and the deploy logs; roll back; then change the pipeline so it cannot lie again. Full write-up: `docs/incidents/INC-005-green-pipeline-outage.md`.
- **S11 — "Errors spiked immediately after the release, then partially recovered."** A backward-incompatible migration ran while old tasks were still draining. Identify from timing that it was the migration, not the code.
- **S04 (pipeline edition) — "Deploys fail with AccessDenied since this morning."** The OIDC trust policy was tightened in a Terraform PR. Find the change record that caused it.
- **"Prod is running something nobody can identify."** A task definition references a tag, not a digest. Prove what is running and trace it back to a commit.

## Deliverables

- [ ] ECS Fargate services for all workloads in dev and prod, via Terraform
- [ ] Reusable workflows: `build-sign-push`, `deploy`, `tf-plan-apply`, `drift-detect`, `change-record`, consumed by three thin callers
- [ ] Test pyramid running in CI with the stage timings recorded
- [ ] Zero long-lived cloud credentials in GitHub (show the secrets list)
- [ ] Signed images with SBOM attestations, verifiable from outside CI
- [ ] Automatic change records for every prod change
- [ ] `docs/runbooks/failed-deployment.md`, `rollback.md`, `ci-break-glass.md`; `docs/operations/change-types.md`
- [ ] ADR-007
- [ ] `docs/incidents/INC-005-green-pipeline-outage.md`
- [ ] README numbers: rollback time, lead time, pipeline duration

## Completion criteria

- Merge to `main` reaches dev automatically and prod after approval, with the **same digest** in both (prove it).
- Measured rollback time in prod under 2 minutes from decision (measure yours; record it honestly).
- PR feedback (lint → unit → contract) in under 5 minutes; full path to dev in under 15.
- A backward-incompatible migration is caught by review/checklist before merge in a staged test, and the expand–contract version deploys with zero errors under a light k6 load.
- Every prod change in the last two weeks has a change record with outcome filled in.
- Explain: build-once-promote and what breaks without it; how the OIDC trust policy restricts who can assume the deploy role; why every gate exists (delete any you cannot justify); standard vs normal vs emergency change.

## Mentor checkpoints

- **Design review:** bring the pipeline stage diagram and ADR-007 before writing YAML. Expect: "What does each gate prevent?", "What is the worst thing a malicious PR from a fork could do?"
- **Code review:** the `deploy.yml` reusable workflow. The mentor checks inputs vs hard-coding, failure handling, rollback path, and whether it verifies outcomes.
- **The failure I'll inject after you think you're done:** a release that passes every gate and still hurts users. You will decide between rollback and roll forward and write the change's post-implementation review.

## Resources

- Jez Humble & David Farley, *Continuous Delivery* — ch. 5 (the deployment pipeline), ch. 10 (deploying and releasing applications), ch. 12 (managing data — migrations).
- Nicole Forsgren et al., *Accelerate* — ch. 2 (measuring performance: the four key metrics) and ch. 4 (technical practices).
- GitHub docs — "Security hardening your deployments: configuring OpenID Connect in Amazon Web Services", "Reusing workflows", "Using environments for deployment".
- Amazon ECS Developer Guide — "Deployment circuit breaker", "Blue/green deployments", "Task IAM role" vs "Task execution IAM role".
- Sigstore docs — "Keyless signing with GitHub Actions".

## Estimate vs actual

| | Hours | Cost |
|---|---|---|
| Planned | 40 | $25–45 |
| Actual | | |
| Variance note | | |
