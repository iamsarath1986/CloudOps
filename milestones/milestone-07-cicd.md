# Milestone 7 — Continuous Delivery

*Phase 6 · ~3 weeks · Prerequisite: Milestones 4, 5, 6*

**Learning topics**
* The pipeline as a product with users, not a script
* **Build once, promote by digest** — the artifact tested in dev must be bit-identical to the one in prod
* Stage ordering: fast feedback first, slow and expensive last; caching, because a slow pipeline gets routed around
* Environments, protection rules, approvals; secrets in CI and why OIDC means there are almost none
* Deployment strategies — recreate, rolling, blue-green, canary — and their cost/risk trade-offs
* **Rollback as a first-class capability**, measured in seconds
* Migration ordering and expand-contract, because "deploy code before migration" and "migration breaks the running old version" are the two classic deployment outages
* Plan-on-PR / apply-on-merge; reusable workflows and composite actions

**Build**
Three reusable workflows: `build-and-push` (build, scan, SBOM, push by digest), `deploy` (deploy a digest, smoke test, roll back on failure), `tf-plan-apply` (plan on PR, apply on merge, approval on prod). A nightly `drift-detect` workflow that runs `terraform plan` and opens an issue if drift exists.

**Cloud resources**
No new resource types — but Terraform now runs from the pipeline, not your laptop, with environment-scoped identities.

**DevOps work**
`cargo-api` and `carrier-sim` consume the reusable workflows. Dev deploys on merge; prod requires approval. Implement expand-contract for a schema change: nullable column → dual-write → backfill → read new → drop old.

**Operations exercise**
Time a rollback: deploy a broken version, roll it back, record the seconds. That number belongs in your README. Then deliberately rebuild the image in the prod job with a floating dependency, watch dev and prod diverge, and fix it by promoting the digest.

**Security considerations**
Break OIDC trust and diagnose the authentication failure. Confirm the prod environment's identity cannot deploy to dev and vice versa. No long-lived cloud credential exists in GitHub.

**Expected outcome**
A merge to `main` results in a scanned, signed-off artifact running in dev automatically and in prod after approval, with an automatic rollback if the smoke test fails.

**Evidence of mastery**
Explain build-once-promote and what specifically breaks without it. State your rollback time and how you measured it. Draw expand-contract on a whiteboard. Justify every gate in your pipeline — if you cannot say what a gate prevents, delete it.
